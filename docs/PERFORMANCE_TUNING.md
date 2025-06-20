# Vehicle Performance Tuning Guide for DragonPilot

This guide provides comprehensive instructions for tuning vehicle performance parameters in this dragonpilot fork to optimize ADAS functionality.

## Overview

Vehicle performance tuning involves adjusting control parameters to achieve optimal:
- Lateral control (steering) responsiveness and stability
- Longitudinal control (speed/following) behavior  
- Safety system integration
- Driver experience and comfort

## Tuning Architecture

### Control Systems Hierarchy
```
Vehicle Interface (interface.py)
    ↓
Lateral/Longitudinal Planners (controls/lib/)
    ↓  
Vehicle Controllers (carcontroller.py)
    ↓
CAN Commands (carcan.py)
    ↓
Vehicle ECUs
```

## Lateral Control Tuning

### 1. PID Controller Tuning

The most common lateral control method uses PID (Proportional-Integral-Derivative) control.

#### 1.1 Basic PID Parameters
Located in `selfdrive/car/tunes.py`:

```python
# Example PID configuration
tune.pid.kpV = [0.2]    # Proportional gain
tune.pid.kiV = [0.05]   # Integral gain  
tune.pid.kf = 0.00006   # Feed-forward gain
```

#### 1.2 PID Tuning Process

**Step 1: Start Conservative**
```python
tune.pid.kpV = [0.1]     # Low proportional gain
tune.pid.kiV = [0.01]    # Low integral gain
tune.pid.kf = 0.00003    # Low feed-forward
```

**Step 2: Tune Proportional Gain (kpV)**
- Increase until system responds well to lane changes
- Too high = oscillation/ping-ponging
- Too low = poor lane tracking, wide turns

**Step 3: Tune Integral Gain (kiV)**  
- Eliminates steady-state error
- Too high = overshooting/instability
- Too low = persistent offset from lane center

**Step 4: Tune Feed-forward (kf)**
- Improves response to curvature
- Helps with smooth curve handling
- Adjust based on steering ratio and vehicle response

#### 1.3 Speed-Dependent Tuning
```python
# Different gains for different speeds
tune.pid.kpBP = [0., 20., 40.]         # Speed breakpoints (m/s)
tune.pid.kpV = [0.15, 0.20, 0.25]     # Proportional gains
tune.pid.kiBP = [0., 20., 40.]         # Speed breakpoints  
tune.pid.kiV = [0.01, 0.03, 0.05]     # Integral gains
```

### 2. INDI (Incremental Nonlinear Dynamic Inversion)

Alternative to PID for vehicles with good torque control.

```python
# INDI parameters in tunes.py
tune.indi.innerLoopGainV = [15]        # Inner loop gain
tune.indi.outerLoopGainV = [17]        # Outer loop gain  
tune.indi.timeConstantV = [4.5]        # Time constant
tune.indi.actuatorEffectivenessV = [15] # Actuator effectiveness
```

### 3. Vehicle-Specific Parameters

#### 3.1 Steering Characteristics
```python
# In values.py CarControllerParams
STEER_MAX = 1500           # Maximum steering torque
STEER_DELTA_UP = 15        # Torque ramp-up rate
STEER_DELTA_DOWN = 25      # Torque ramp-down rate  
STEER_ERROR_MAX = 350      # Maximum allowed error
```

#### 3.2 Physical Vehicle Parameters
```python
# In interface.py _get_params()
ret.wheelbase = 2.7           # Vehicle wheelbase (meters)
ret.steerRatio = 15.3         # Steering ratio (wheel:road)
ret.mass = 1650 + STD_CARGO_KG # Vehicle mass (kg)
```

## Longitudinal Control Tuning

### 1. Acceleration Limits
```python
# In values.py CarControllerParams  
ACCEL_MAX = 1.5    # Maximum acceleration (m/s²)
ACCEL_MIN = -3.5   # Maximum deceleration (m/s²)
```

### 2. Following Distance and Behavior
DragonPilot-specific parameters in `common/dp_conf.py`:

```python
# Cruise control behavior
'dp_toyota_cruise_override': True/False
'dp_toyota_cruise_override_speed': 30  # km/h

# Acceleration profiles  
'dp_accel_profile': 0-2  # 0=Eco, 1=Normal, 2=Sport
```

## Torque Data Calibration

### 1. Understanding Torque Parameters

Reference file: `selfdrive/car/torque_data/params.yaml`

```yaml
TOYOTA CAMRY 2018: [2.1172995371905015, 1.7156177222420887, 0.105192506]
# Format: [LAT_ACCEL_FACTOR, MAX_LAT_ACCEL_MEASURED, FRICTION]
```

#### Parameter Definitions:
- **LAT_ACCEL_FACTOR**: Converts steering torque to lateral acceleration
- **MAX_LAT_ACCEL_MEASURED**: Maximum observed lateral acceleration  
- **FRICTION**: Tire-road friction coefficient

### 2. Measuring Torque Parameters

#### 2.1 Data Collection Process
```bash
# 1. Record driving data with aggressive cornering
./launch_openpilot.sh

# 2. Analyze logs for maximum lateral acceleration
# Look for: lateralAcceleration, steeringTorque values

# 3. Calculate LAT_ACCEL_FACTOR
# LAT_ACCEL_FACTOR = lateral_accel / steering_torque
```

#### 2.2 Parameter Calculation
```python
# Example calculation script
import numpy as np

# From log analysis
steering_torques = [...]  # Steering torque values
lateral_accels = [...]    # Corresponding lateral accelerations

# Calculate factor
lat_accel_factor = np.mean(lateral_accels / steering_torques)
max_lat_accel = np.max(lateral_accels)
friction = 0.1  # Typical starting value, adjust based on testing
```

## DragonPilot-Specific Tuning

### 1. Lateral Control Enhancements
```python
# Lane change behavior
'dp_lateral_mode': 0-2
# 0=Stock, 1=Manual lane change, 2=Auto lane change

'dp_lc_min_mph': 15        # Minimum speed for lane changes
'dp_lc_auto_min_mph': 40   # Minimum speed for auto lane changes
'dp_lc_auto_delay': 3.0    # Delay before auto lane change

# Camera and path offsets
'dp_lateral_camera_offset': -6 to 100  # Camera position adjustment
'dp_lateral_path_offset': 0 to 100     # Path planning offset
```

### 2. UI and Behavior Customization
```python
# Quiet drive modes
'dp_quiet_drive': 0-2  # 0=Normal, 1=Quiet, 2=Silent

# Toyota-specific enhancements
'dp_toyota_sng': True/False           # Stop and go enhancement
'dp_toyota_auto_lock': True/False     # Auto door lock
'dp_toyota_auto_unlock': True/False   # Auto door unlock
```

## Testing and Validation Methodology

### 1. Systematic Testing Process

#### Phase 1: Bench Testing
- Verify parameter loading
- Check CAN message generation
- Test emergency disengagement

#### Phase 2: Controlled Environment
- Empty parking lot testing
- Low speed maneuvers (&lt;20 km/h)
- Basic lane keeping validation

#### Phase 3: Real-World Testing  
- Highway driving
- Various weather conditions
- Different road types and curvatures

### 2. Performance Metrics

#### Lateral Performance
- Lane keeping accuracy (RMS error from lane center)
- Steering smoothness (minimal oscillation)
- Response time to lane changes
- Curve handling capability

#### Safety Metrics
- Emergency disengagement response time
- Driver override responsiveness  
- Fault detection and recovery
- System availability percentage

### 3. Data Collection and Analysis

```python
# Key metrics to monitor
logs_to_analyze = [
    'lateralAcceleration',
    'steeringAngleDeg', 
    'steeringTorque',
    'pathPlannerError',
    'dPathPoints',
    'vEgo',
    'aEgo'
]
```

## Troubleshooting Common Issues

### Issue: Oscillation/Ping-ponging
**Symptoms**: Vehicle weaves back and forth in lane
**Solutions**:
- Reduce proportional gain (kpV)
- Increase damping (steering delta down)
- Check steering ratio calibration
- Verify camera mounting and calibration

### Issue: Poor Curve Handling  
**Symptoms**: Cuts corners, late turn initiation
**Solutions**:
- Increase feed-forward gain (kf)
- Adjust path planning parameters
- Verify camera calibration
- Check LAT_ACCEL_FACTOR calibration

### Issue: Slow Response
**Symptoms**: Delayed reaction to lane changes
**Solutions**:
- Increase proportional gain (kpV)
- Reduce actuator delay values
- Check CAN timing and latency
- Verify steering ratio

### Issue: Steering Feels "Heavy"
**Symptoms**: High steering effort, driver fatigue
**Solutions**:
- Reduce STEER_MAX values
- Adjust STEER_DELTA rates  
- Check torque scaling factors
- Verify EPS system integration

## Advanced Tuning Techniques

### 1. Model Predictive Control (MPC)
For advanced users, lateral MPC tuning:

```python
# Located in selfdrive/controls/lib/lateral_mpc_lib/
# Adjust cost weights in acados_ocp_lat.json
"cost_weights": {
    "W_y": 1.0,      # Position error weight
    "W_y_e": 10.0,   # Terminal position weight  
    "W_u": 0.1       # Control effort weight
}
```

### 2. Vehicle Model Adaptation
```python
# In selfdrive/controls/lib/vehicle_model.py
# Adjust vehicle model parameters for specific dynamics
```

### 3. Custom Tune Selection
Add custom tunes to `selfdrive/car/tunes.py`:
```python
class LatTunes(Enum):
    # Add your custom tune
    PID_CUSTOM_BYD = 50

# Implement in set_lat_tune()
elif name == LatTunes.PID_CUSTOM_BYD:
    tune.pid.kpV = [0.25]
    tune.pid.kiV = [0.04]  
    tune.pid.kf = 0.00007
```

## Safety Guidelines

### Critical Safety Rules
1. **Always start with conservative parameters**
2. **Test incrementally with professional drivers**
3. **Have emergency procedures ready**
4. **Monitor system performance continuously**
5. **Document all changes and test results**

### Parameter Change Protocol
1. Make small incremental changes (10-20%)
2. Test in controlled environment first
3. Monitor for any adverse behavior
4. Revert immediately if issues arise
5. Document performance before/after

### Emergency Procedures
- Know how to quickly disable openpilot
- Practice manual takeover scenarios
- Have backup communication methods
- Understand system failure modes

## Performance Optimization Checklist

- [ ] Baseline performance measurement
- [ ] Vehicle-specific parameter calibration
- [ ] Torque data measurement and tuning
- [ ] PID/INDI gain optimization
- [ ] Speed-dependent parameter adjustment
- [ ] DragonPilot feature configuration
- [ ] Safety system integration testing
- [ ] Real-world validation testing
- [ ] Performance metric documentation
- [ ] Parameter backup and versioning

## Resources and References

- [OpenPilot Tuning Guide](https://github.com/commaai/openpilot/wiki)
- [DragonPilot Documentation](https://github.com/dragonpilot-community/dragonpilot)
- [Vehicle Dynamics and Control Theory](https://www.amazon.com/Vehicle-Dynamics-Control-Rajesh-Rajamani/dp/1461414032)
- [PID Controller Tuning](https://en.wikipedia.org/wiki/PID_controller#Tuning)
- [Automotive Control Systems](https://ieeexplore.ieee.org/document/8744636)

Remember: Vehicle tuning is an iterative process requiring patience, systematic testing, and prioritization of safety above all else.