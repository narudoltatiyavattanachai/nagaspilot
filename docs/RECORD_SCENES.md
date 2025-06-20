# CAN Data Collection Scenes and Conditions

This document provides a comprehensive list of scenarios and conditions for collecting CAN bus data during vehicle porting and analysis. Each scenario is designed to capture specific vehicle systems and their CAN message patterns.

## Recording Setup Requirements

### Hardware Setup
- Panda device or compatible CAN interface
- Comma Three device (optional but recommended)
- OBD-II connection to vehicle
- Laptop with recording software
- Storage device with sufficient capacity (minimum 32GB for extended recording)

### Software Setup
```bash
# Install required tools
cd /data/openpilot/tools
python -m pip install cantools pandas matplotlib

# Start CAN recording
python /data/openpilot/tools/lib/logreader.py --can --output /data/can_logs/
```

### Pre-Recording Checklist
- [ ] Vehicle in good mechanical condition
- [ ] All safety systems functional
- [ ] Professional driver or test environment
- [ ] Emergency procedures established
- [ ] Recording equipment tested and configured
- [ ] Sufficient storage space available

## Priority 1: Critical ADAS Systems

### 1.1 Steering System Analysis
**Objective**: Identify steering wheel angle, torque, and EPS control messages

#### Scenario 1.1.1: Steering Wheel Movement Test
**Location**: Parking lot or safe closed area
**Duration**: 10-15 minutes
**Conditions**:
- Vehicle stationary, engine on
- Manual steering wheel movements:
  - Slow left turn (full lock)
  - Slow right turn (full lock)  
  - Center position hold
  - Quick left-right movements
  - Various turning speeds
  - Step inputs (sudden movements)
  - Sinusoidal movements

#### Scenario 1.1.2: On-Road Steering Analysis
**Location**: Various road types
**Duration**: 30-45 minutes
**Conditions**:
- Straight highway driving
- Gentle curve navigation
- Sharp turn execution  
- Lane change maneuvers
- Parking lot navigation
- Low speed maneuvering
- Different road surfaces

#### Scenario 1.1.3: EPS System States
**Location**: Safe test area
**Duration**: 15-20 minutes
**Conditions**:
- EPS system ON/OFF transitions
- Engine start/stop with steering
- Various EPS assist levels (if available)
- Heavy steering load conditions
- Light steering load conditions
- Power steering fault simulation (if safe)

### 1.2 Vehicle Speed and Motion
**Objective**: Identify speed sensors, wheel speeds, and motion-related messages

#### Scenario 1.2.1: Speed Sensor Analysis
**Location**: Controlled test track or empty road
**Duration**: 20-30 minutes
**Conditions**:
- Vehicle stationary (0 km/h)
- Slow acceleration from stop
- Constant speeds: 10, 20, 30, 40, 50, 60, 80, 100 km/h
- Gradual deceleration
- Emergency braking (safe conditions)
- Reverse driving
- Different gear positions

#### Scenario 1.2.2: Wheel Speed Differential
**Location**: Controlled environment
**Duration**: 15-20 minutes
**Conditions**:
- Straight line driving
- Left/right turning (different wheel speeds)
- Parking lot circles
- Figure-8 patterns
- One wheel on different surface (safely)
- ABS activation (safe conditions)
- Hill driving (uphill/downhill)

### 1.3 Brake System Analysis
**Objective**: Identify brake pedal, brake pressure, and brake system messages

#### Scenario 1.3.1: Brake Pedal Inputs
**Location**: Safe test area
**Duration**: 15-20 minutes
**Conditions**:
- Vehicle stationary, engine on
- Light brake pedal pressure
- Medium brake pressure
- Heavy brake pressure
- Rapid pedal press/release
- Gradual pressure increase/decrease
- Brake hold (if available)
- Electronic parking brake operation

#### Scenario 1.3.2: Brake System Operation
**Location**: Controlled driving area
**Duration**: 25-30 minutes
**Conditions**:
- Normal braking scenarios
- Progressive braking
- Emergency braking
- ABS activation
- ESP/ESC activation
- Hill hold operation
- Brake assist activation
- Regenerative braking (EVs/Hybrids)

### 1.4 Accelerator/Throttle System
**Objective**: Identify accelerator pedal and throttle control messages

#### Scenario 1.4.1: Accelerator Pedal Analysis
**Location**: Safe test area
**Duration**: 15-20 minutes
**Conditions**:
- Vehicle stationary, engine on
- Slow pedal depression
- Rapid pedal inputs
- Partial throttle positions (25%, 50%, 75%, 100%)
- Pedal release patterns
- Eco/Normal/Sport mode changes
- Kick-down activation (automatic transmission)

## Priority 2: Driver Interface and Controls

### 2.1 Turn Signal and Indicator Systems
**Location**: Parking lot or safe area
**Duration**: 10-15 minutes
**Conditions**:
- Left turn signal activation
- Right turn signal activation
- Hazard lights activation
- Turn signal cancellation
- Multi-function stalk positions
- Lane change signals (brief activation)
- Emergency flasher sequences

### 2.2 Gear Selection and Transmission
**Location**: Safe test area
**Duration**: 15-20 minutes
**Conditions**:
- Park position
- Reverse gear selection
- Neutral position  
- Drive gear selection
- Manual mode (if available)
- Sport mode (if available)
- Gear change sequences
- Hill descent control
- Paddle shifter inputs (if available)

### 2.3 Cruise Control and ADAS Controls
**Location**: Highway or controlled road
**Duration**: 25-35 minutes
**Conditions**:
- Cruise control activation/deactivation
- Speed set/resume operations
- Following distance adjustments
- Speed increase/decrease commands
- Cancel operations
- Adaptive cruise control modes
- Lane keeping assist activation
- Driver assistance system toggles

### 2.4 Driver Input Controls
**Location**: Various
**Duration**: 20-25 minutes
**Conditions**:
- Steering wheel button presses
- Multi-function display controls
- Audio system controls
- Climate control adjustments
- Seat adjustment controls
- Mirror adjustment controls
- Window controls
- Horn activation

## Priority 3: Safety and Monitoring Systems

### 3.1 Door and Safety Status
**Location**: Parking area
**Duration**: 10-15 minutes
**Conditions**:
- All doors closed
- Individual door opening/closing
- Hood opening/closing
- Trunk/tailgate operation
- Fuel door operation (if applicable)
- Charge port operation (EVs)

### 3.2 Seatbelt and Occupancy
**Location**: Stationary vehicle
**Duration**: 10-12 minutes
**Conditions**:
- Driver seatbelt fastened/unfastened
- Passenger seatbelt states
- Driver seat occupied/unoccupied
- Passenger seat occupancy changes
- Rear seat occupancy (if monitored)

### 3.3 Lighting Systems
**Location**: Parking area (day/night)
**Duration**: 15-20 minutes
**Conditions**:
- Headlight OFF/AUTO/ON positions
- High beam activation
- Fog light operation
- Interior lighting
- Brake light activation
- Reverse light operation
- Daytime running lights
- Automatic headlight operation

## Priority 4: Powertrain and Engine Systems

### 4.1 Engine/Motor Status (ICE/EV/Hybrid)
**Location**: Various
**Duration**: 20-30 minutes
**Conditions**:
- Engine start/stop sequences
- Idle conditions
- Various RPM levels
- Load conditions
- Temperature variations
- Hybrid mode transitions (if applicable)
- Electric mode operation (EVs/PHEVs)
- Regenerative braking states

### 4.2 Fuel/Battery Systems
**Location**: Various
**Duration**: 15-20 minutes
**Conditions**:
- Various fuel/charge levels
- Fuel consumption data
- Battery status messages (EVs)
- Charging state messages (EVs/PHEVs)
- Range calculations
- Efficiency displays

### 4.3 Climate Control Systems
**Location**: Stationary and driving
**Duration**: 20-25 minutes
**Conditions**:
- HVAC system ON/OFF
- Temperature adjustments
- Fan speed changes
- Air distribution mode changes
- A/C compressor operation
- Defrost/defog operations
- Automatic climate control
- Seat heating/cooling

## Priority 5: Advanced Driver Assistance Systems (ADAS)

### 5.1 Lane Keeping and Lane Departure
**Location**: Highway with clear lane markings
**Duration**: 30-40 minutes
**Conditions**:
- Lane keeping assist activation
- Lane departure warnings
- Lane change assistance
- Blind spot monitoring
- Lane centering operation
- Road edge detection
- Various lane marking conditions

### 5.2 Adaptive Cruise Control (ACC)
**Location**: Highway or controlled road
**Duration**: 35-45 minutes
**Conditions**:
- ACC activation with no lead vehicle
- Following a lead vehicle at various distances
- Cut-in scenarios (safe conditions)
- Speed limit adaptation
- Traffic jam assist
- Stop and go operation
- ACC deactivation scenarios

### 5.3 Collision Avoidance Systems
**Location**: Controlled test environment
**Duration**: 20-30 minutes
**Conditions**:
- Forward collision warnings
- Automatic emergency braking (simulated safely)
- Pedestrian detection alerts
- Cross traffic alerts
- Blind spot warnings
- Parking assist operations
- 360-degree camera systems

### 5.4 Parking and Maneuvering
**Location**: Parking areas
**Duration**: 20-25 minutes
**Conditions**:
- Parking sensor activation
- Reverse camera operation
- Automatic parking assist
- Parking brake operation
- Hill start assist
- Traction control activation
- Electronic stability control

## Special Conditions and Environmental Scenarios

### 6.1 Environmental Conditions
**Duration**: Record during various conditions
**Conditions**:
- Cold weather operation (-10°C to 0°C)
- Hot weather operation (30°C to 45°C)
- Rain/wet road conditions
- Night driving
- Tunnel driving
- Bridge/overpass driving
- Construction zone navigation

### 6.2 Edge Cases and Fault Conditions
**Location**: Controlled environment only
**Duration**: Variable
**Conditions**:
- Low battery/fuel conditions
- System fault simulations (where safe)
- Sensor obstruction scenarios
- GPS signal loss
- Communication errors
- Power supply variations
- Component disconnection (safely)

### 6.3 Performance Driving (Professional Only)
**Location**: Closed course/track
**Duration**: 20-30 minutes
**Conditions**:
- Aggressive acceleration
- Hard braking scenarios
- High-speed cornering
- Track driving modes
- Performance mode activation
- Stability system limits
- Maximum system capability testing

## Data Organization and Analysis

### Recording Session Structure
```
/data/can_logs/
├── vehicle_info.txt           # Vehicle details and VIN
├── session_YYYYMMDD_HHMMSS/   # Recording session folder
│   ├── scenario_1_1_1.log     # Steering wheel test
│   ├── scenario_1_2_1.log     # Speed sensor test
│   ├── scenario_notes.txt     # Test conditions and notes
│   └── video/                 # Optional video recordings
├── analysis/
│   ├── message_frequency.csv  # CAN message analysis
│   ├── signal_ranges.csv      # Signal value ranges
│   └── decoded_messages.dbc   # Preliminary DBC file
```

### Analysis Checklist
- [ ] Message frequency analysis
- [ ] Signal value range determination
- [ ] Message correlation identification
- [ ] Timing analysis
- [ ] Error/fault message cataloging
- [ ] State machine behavior mapping
- [ ] Control command validation

### Recording Quality Validation
- [ ] All scenarios completed successfully
- [ ] No data corruption or gaps
- [ ] Sufficient message samples for each scenario
- [ ] Clear correlation between actions and CAN messages
- [ ] Fault conditions properly documented
- [ ] Environmental conditions recorded

## Safety Considerations

### Critical Safety Rules
1. **Professional supervision required** for any advanced testing
2. **Closed course testing** for performance/limit scenarios
3. **Emergency procedures** established before testing
4. **Vehicle condition verification** before each session
5. **Weather and road condition assessment**
6. **Communication protocols** for test team
7. **Data backup procedures** to prevent loss

### Emergency Procedures
- Immediate test termination protocols
- Vehicle recovery procedures
- Data preservation methods
- Incident reporting requirements
- Equipment shutdown sequences

### Legal and Regulatory Compliance
- Obtain necessary permits for closed course testing
- Ensure compliance with local traffic laws
- Verify insurance coverage for testing activities
- Document professional driver qualifications
- Maintain safety equipment certifications

## Post-Recording Analysis Workflow

### 1. Data Validation
- Verify recording completeness
- Check data integrity
- Validate timing consistency
- Confirm scenario coverage

### 2. Message Identification
- Frequency analysis of CAN IDs
- Signal pattern recognition
- State correlation analysis
- Control command mapping

### 3. DBC File Development
- Create preliminary message definitions
- Define signal scaling and offsets
- Implement message validation
- Test message decoding accuracy

### 4. Integration Testing
- Implement basic parsing
- Validate real-time decoding
- Test control command generation
- Verify safety interlocks

This comprehensive recording plan ensures complete CAN bus analysis coverage for successful vehicle porting to the dragonpilot platform. Always prioritize safety and use professional drivers for advanced testing scenarios.