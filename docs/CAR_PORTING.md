# BYD Atto3 Porting Guide for DragonPilot

This document provides comprehensive guidelines for porting the BYD Atto3 electric vehicle to work with this dragonpilot fork.

## Prerequisites

### Hardware Requirements
- Comma Three device
- Vehicle-specific harness for BYD Atto3
- Access to BYD Atto3 OBD-II port
- CAN bus analysis tools (Panda device recommended)

### Knowledge Requirements
- CAN bus protocol understanding
- Python programming experience
- Automotive systems knowledge
- Understanding of vehicle control systems (steering, acceleration, braking)

## Phase 1: CAN Bus Analysis

### 1.1 Initial CAN Data Collection
```bash
# Use panda to collect CAN data
cd /data/openpilot/tools/joystick
python joystickd.py  # For manual control testing

# Collect CAN logs during various driving scenarios:
# - Steering wheel movements (left/right)
# - Acceleration/deceleration
# - Turn signal activation
# - Cruise control engagement
# - Lane keep assist activation (if available)
```

### 1.2 CAN Message Identification
Focus on identifying these critical CAN messages:
- **Steering wheel angle** (typically 0x25, 0x260, or manufacturer-specific)
- **Vehicle speed** (wheel speeds, GPS speed)
- **Steering torque** (EPS motor commands)
- **Accelerator/brake pedal positions**
- **Turn signals**
- **Gear position**
- **Cruise control status**
- **Safety systems status**

### 1.3 CAN Database Creation
Create a new DBC file: `opendbc/byd_atto3_generated.dbc`

Example structure:
```dbc
VERSION ""

NS_ : 
	NS_DESC_
	CM_
	BA_DEF_
	BA_
	VAL_
	CAT_DEF_
	CAT_
	FILTER
	BA_DEF_DEF_
	EV_DATA_
	ENVVAR_DATA_
	SGTYPE_
	SGTYPE_VAL_
	BA_DEF_SGTYPE_
	BA_SGTYPE_
	SIG_VALTYPE_
	SIGTYPE_VALTYPE_
	BO_TX_BU_
	BA_DEF_REL_
	BA_REL_
	BA_DEF_DEF_REL_
	BU_SG_REL_
	BU_EV_REL_
	BU_BO_REL_
	SG_MUL_VAL_

BS_:

BU_:

BO_ 548 STEER_ANGLE_SENSOR: 8 XXX
 SG_ STEER_ANGLE : 3|12@0+ (0.1,0) [0|0] "deg" XXX
 SG_ STEER_RATE : 35|12@0- (0.1,0) [0|0] "deg/s" XXX

BO_ 37 STEER_TORQUE_SENSOR: 8 XXX
 SG_ STEER_TORQUE_DRIVER : 7|16@0+ (1,0) [0|0] "" XXX
 SG_ STEER_TORQUE_EPS : 23|16@0- (1,0) [0|0] "" XXX
```

## Phase 2: Vehicle Interface Development

### 2.1 Create BYD Directory Structure
```bash
mkdir -p selfdrive/car/byd
touch selfdrive/car/byd/__init__.py
touch selfdrive/car/byd/values.py
touch selfdrive/car/byd/interface.py
touch selfdrive/car/byd/carstate.py
touch selfdrive/car/byd/carcontroller.py
touch selfdrive/car/byd/bydcan.py
touch selfdrive/car/byd/radar_interface.py
```

### 2.2 Vehicle Values Configuration (values.py)
```python
from dataclasses import dataclass, field
from enum import Enum, IntFlag
from typing import Dict, List, Union

from cereal import car
from selfdrive.car import dbc_dict
from selfdrive.car.docs_definitions import CarInfo, CarParts

Ecu = car.CarParams.Ecu

class CAR:
  BYD_ATTO3_2022 = "BYD ATTO3 2022"

@dataclass
class BydCarInfo(CarInfo):
  package: str = "All"

CAR_INFO: Dict[str, Union[BydCarInfo, List[BydCarInfo]]] = {
  CAR.BYD_ATTO3_2022: BydCarInfo("BYD Atto3 2022"),
}

class CarControllerParams:
  STEER_MAX = 1000  # Adjust based on testing
  STEER_DELTA_UP = 10
  STEER_DELTA_DOWN = 25
  STEER_ERROR_MAX = 350
  
  # Longitudinal limits
  ACCEL_MAX = 2.0  # m/s2
  ACCEL_MIN = -3.5  # m/s2

DBC = {
  CAR.BYD_ATTO3_2022: dbc_dict('byd_atto3_generated', None),
}

STEER_THRESHOLD = 150
```

### 2.3 CAN Interface (bydcan.py)
```python
import struct
from selfdrive.config import Conversions as CV

def create_steer_command(packer, steer, enabled):
  values = {
    "STEER_TORQUE_CMD": steer,
    "STEER_REQUEST": enabled,
  }
  return packer.make_can_msg("STEERING_LKA", 0, values)

def create_accel_command(packer, accel, enabled):
  values = {
    "ACCEL_CMD": accel,
    "ACC_REQUEST": enabled,
  }
  return packer.make_can_msg("ACC_CONTROL", 0, values)
```

### 2.4 Car State Reader (carstate.py)
```python
from cereal import car
from selfdrive.car.interfaces import CarStateBase
from opendbc.can.parser import CANParser
from selfdrive.car.byd.values import DBC, STEER_THRESHOLD

class CarState(CarStateBase):
  def __init__(self, CP):
    super().__init__(CP)

  def update(self, cp, cp_cam):
    ret = car.CarState.new_message()
    
    # Update vehicle state from CAN
    ret.vEgo = cp.vl["SPEED"]["SPEED"] * CV.KPH_TO_MS
    ret.vEgoRaw = ret.vEgo
    ret.aEgo = cp.vl["ACCEL"]["ACCEL"]
    
    # Steering
    ret.steeringAngleDeg = cp.vl["STEER_ANGLE_SENSOR"]["STEER_ANGLE"]
    ret.steeringRateDeg = cp.vl["STEER_ANGLE_SENSOR"]["STEER_RATE"]
    ret.steeringTorque = cp.vl["STEER_TORQUE_SENSOR"]["STEER_TORQUE_DRIVER"]
    ret.steeringPressed = abs(ret.steeringTorque) > STEER_THRESHOLD
    
    # Gear and pedals
    ret.gearShifter = self.parse_gear_shifter(cp.vl["GEAR"]["GEAR"])
    ret.gas = cp.vl["PEDALS"]["GAS_PEDAL"] / 100.0
    ret.brake = cp.vl["PEDALS"]["BRAKE_PEDAL"] / 100.0
    
    return ret

  @staticmethod
  def get_can_parser(CP):
    signals = [
      ("SPEED", "SPEED", 0),
      ("ACCEL", "ACCEL", 0),
      ("STEER_ANGLE", "STEER_ANGLE_SENSOR", 0),
      ("STEER_RATE", "STEER_ANGLE_SENSOR", 0),
      ("STEER_TORQUE_DRIVER", "STEER_TORQUE_SENSOR", 0),
      ("GEAR", "GEAR", 0),
      ("GAS_PEDAL", "PEDALS", 0),
      ("BRAKE_PEDAL", "PEDALS", 0),
    ]
    
    checks = [
      ("SPEED", 50),
      ("STEER_ANGLE_SENSOR", 100),
      ("STEER_TORQUE_SENSOR", 100),
    ]
    
    return CANParser(DBC[CP.carFingerprint]["pt"], signals, checks, 0)
```

## Phase 3: Control Algorithm Tuning

### 3.1 Lateral Control Tuning
Start with conservative PID values and iterate:

```python
# In selfdrive/car/byd/interface.py
def _get_params(ret, candidate, fingerprint, car_fw, experimental_long, docs):
    ret.carName = "byd"
    ret.safetyConfigs = [get_safety_config(car.CarParams.SafetyModel.byd)]
    
    # Start with conservative lateral tuning
    ret.lateralTuning.init('pid')
    ret.lateralTuning.pid.kiBP = [0.]
    ret.lateralTuning.pid.kpBP = [0.]
    ret.lateralTuning.pid.kpV = [0.1]    # Start low
    ret.lateralTuning.pid.kiV = [0.01]   # Start low
    ret.lateralTuning.pid.kf = 0.00003   # Start low
    
    ret.steerActuatorDelay = 0.15
    ret.steerLimitTimer = 0.4
    ret.steerMaxBP = [0.]
    ret.steerMaxV = [1000]  # Adjust based on testing
```

### 3.2 Safety Parameters
```python
# Critical safety limits
ret.wheelbase = 2.72  # BYD Atto3 wheelbase in meters
ret.steerRatio = 15.5  # Estimate, needs validation
ret.mass = 1750 + STD_CARGO_KG  # BYD Atto3 curb weight

# Conservative initial limits
ret.steerControlType = car.CarParams.SteerControlType.torque
ret.minSteerSpeed = 0.
ret.minEnableSpeed = -1.
```

## Phase 4: Testing and Validation

### 4.1 Bench Testing
1. Test CAN message parsing without vehicle movement
2. Verify steering wheel angle readings
3. Test basic CAN command generation

### 4.2 Vehicle Testing Protocol
⚠️ **SAFETY FIRST**: Always have a driver ready to take control

1. **Phase 1**: Passive monitoring only
   - Log all CAN data
   - Verify correct parsing of vehicle state
   - No control commands sent

2. **Phase 2**: Minimal steering assistance
   - Very low gain values
   - Low speed only (&lt;30 km/h)
   - Straight road testing only

3. **Phase 3**: Gradual tuning
   - Incrementally increase gains
   - Test in various conditions
   - Monitor for oscillations or instability

### 4.3 Tuning Checklist
- [ ] Steering wheel angle calibration
- [ ] Steering ratio validation  
- [ ] Maximum steering torque limits
- [ ] Lateral acceleration limits
- [ ] Emergency disengagement testing
- [ ] Turn signal integration
- [ ] Speed accuracy validation

## Phase 5: Integration and Optimization

### 5.1 Add to Main Car List
```python
# In selfdrive/car/__init__.py
from selfdrive.car.byd.values import CAR as BYD

# Add to car imports and fingerprinting
```

### 5.2 Performance Tuning
Use parameters from `selfdrive/car/torque_data/params.yaml` as reference:
- Adjust LAT_ACCEL_FACTOR based on real-world testing
- Calibrate MAX_LAT_ACCEL_MEASURED 
- Set appropriate FRICTION coefficient

### 5.3 DragonPilot Integration
Add BYD-specific parameters to `common/dp_conf.py`:
```python
# BYD specific configurations
{'name': 'dp_byd_regen_level', 'default': 1, 'type': 'UInt8', 'min': 0, 'max': 3, 'conf_type': ['param', 'struct']},
{'name': 'dp_byd_eco_mode', 'default': False, 'type': 'Bool', 'conf_type': ['param', 'struct']},
```

## Safety Considerations

### Critical Safety Requirements
1. **Steering Override**: Driver input must always override system
2. **Speed Limits**: Set conservative maximum speeds initially  
3. **Fault Detection**: Implement robust error detection
4. **Emergency Stop**: System must safely disengage on any error
5. **Manual Takeover**: Seamless transition to manual control

### Testing Environment
- Start with closed course testing
- Use professional test drivers
- Have backup safety systems
- Document all failure modes

## Common Issues and Solutions

### Issue: Steering Oscillation
**Solution**: Reduce proportional gain (kpV), increase damping

### Issue: Poor Lane Tracking  
**Solution**: Adjust camera calibration, verify steering ratio

### Issue: CAN Message Timing
**Solution**: Verify message frequencies, adjust timeout values

### Issue: Safety System Conflicts
**Solution**: Identify and disable conflicting ADAS features

## References and Resources

- [OpenPilot Car Porting Guide](https://blog.comma.ai/how-to-write-a-car-port-for-openpilot/)
- [DragonPilot Documentation](https://github.com/dragonpilot-community/dragonpilot)
- [CAN Bus Analysis Tools](https://github.com/commaai/panda)
- BYD Atto3 Technical Specifications
- Automotive CAN Standards (ISO 11898)

## Contributing

When your BYD Atto3 port is ready:
1. Submit CAN database to opendbc repository
2. Create pull request with vehicle interface code
3. Provide testing documentation and validation data
4. Update vehicle compatibility lists

Remember: Vehicle porting requires extensive testing and validation. Never compromise on safety for functionality.