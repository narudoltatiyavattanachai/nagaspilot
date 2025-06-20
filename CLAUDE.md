# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a dragonpilot fork of openpilot - an open source driver assistance system that provides Adaptive Cruise Control (ACC), Automated Lane Centering (ALC), Forward Collision Warning (FCW), and Lane Departure Warning (LDW). The dragonpilot fork adds additional features and customizations.

## Key Architecture Components

### Core System Architecture
- **selfdrive/**: Main application code for vehicle control and UI
  - **controls/**: Planning and vehicle control logic (controlsd.py, plannerd.py, radard.py)
  - **car/**: Vehicle-specific interfaces for different manufacturers (honda/, toyota/, hyundai/, etc.)
  - **manager/**: Process management and orchestration (manager.py)
  - **locationd/**: Localization and sensor fusion
  - **modeld/**: AI model runners for driving and monitoring
  - **ui/**: User interface components
  - **dragonpilot/**: Dragon-specific enhancements and features

### Supporting Systems
- **cereal/**: Message schema and serialization for inter-process communication
- **common/**: Shared utilities and helper functions
- **system/**: Hardware abstraction and system-level services
- **panda/**: CAN bus communication interface
- **opendbc/**: Vehicle CAN message definitions

## Development Commands

### Running the System
- `./launch_openpilot.sh` - Launch openpilot in active mode (PASSIVE=0)
- `./launch_chffrplus.sh` - Launch with configuration from launch_env.sh
- Environment variables set in `launch_env.sh`

### Testing
- Python test files follow naming pattern `test_*.py` 
- Test directories: `*/test/`, `*/tests/`
- Key test files:
  - `common/kalman/tests/test_simple_kalman.py`
  - `tools/lib/tests/test_*.py`
  - `selfdrive/*/test/test_*.py`

### Process Management
- Main entry point: `selfdrive/manager/manager.py`
- Process configuration: `selfdrive/manager/process_config.py`
- All processes are managed by the manager daemon

## Important Development Notes

### Dragon-specific Features
- This fork includes dragonpilot customizations in `selfdrive/dragonpilot/`
- Configuration system uses `common/dp_conf.py` for dragon-specific parameters
- Additional UI templates in `selfdrive/dragonpilot/tpl/`

### Code Organization
- Vehicle interfaces follow consistent pattern: carcontroller.py, carstate.py, interface.py, values.py
- CAN message handling in *can.py files (e.g., hondacan.py, toyotacan.py)
- Model inference handled by separate daemon processes (modeld, dmonitoringmodeld)
- Control algorithms split between lateral (steering) and longitudinal (speed) planners

### Key Dependencies
- Uses custom messaging system (cereal) for IPC
- EKF/Kalman filtering via rednose library
- ACADOS for model predictive control
- Qt for UI components
- Hardware abstraction supports both PC and embedded (TICI) platforms

## Common File Patterns
- `*d.py` files are typically daemon processes
- `interface.py` files provide standardized car interfaces
- `values.py` files contain vehicle-specific constants and parameters
- Generated code often in `c_generated_code/` directories

## Vehicle Porting and Performance Tuning

### BYD Atto3 Porting
For porting the BYD Atto3 electric vehicle, see the comprehensive guide at:
- `docs/BYD_ATTO3_PORTING.md` - Complete porting methodology including CAN analysis, interface development, and safety protocols

### Performance Tuning
For optimizing vehicle performance and control parameters:
- `docs/VEHICLE_PERFORMANCE_TUNING.md` - Comprehensive tuning guide covering lateral/longitudinal control, PID tuning, torque calibration, and DragonPilot-specific features

### Key Tuning Files
- `selfdrive/car/tunes.py` - Lateral control parameter definitions (PID, INDI)
- `selfdrive/car/torque_data/params.yaml` - Vehicle-specific torque characteristics
- `common/dp_conf.py` - DragonPilot configuration parameters
- `selfdrive/car/*/values.py` - Vehicle-specific control limits and characteristics

### Tuning Process Overview
1. **CAN Analysis**: Identify and decode vehicle CAN messages
2. **Interface Development**: Create vehicle-specific interface classes
3. **Safety Calibration**: Set conservative control limits and safety parameters  
4. **Performance Optimization**: Tune PID/INDI gains for optimal control
5. **Real-world Testing**: Validate performance across various driving conditions