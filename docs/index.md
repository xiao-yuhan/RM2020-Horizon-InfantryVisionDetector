# RM2020-Horizon-InfantryVisionDetector Documentation

Welcome to the documentation for the RM2020-Horizon-InfantryVisionDetector system, developed by the Horizon team from North China University of Technology for the RoboMaster 2020 competition.

## Documentation Contents

### Getting Started
- [README](README.md) - Overview and introduction to the system
- [Quick Start Guide](quick_start.md) - Get up and running quickly

### Technical Documentation
- [Technical Details](technical_details.md) - In-depth technical information
- [API Reference](api_reference.md) - Reference for classes, functions, and data structures

## System Overview

The RM2020-Horizon-InfantryVisionDetector is a vision-based detection and targeting system for infantry robots in the RoboMaster competition. It includes:

1. **Armor Detection** - Identifies enemy robot armor plates
2. **Angle Solving** - Calculates targeting angles
3. **Motion Prediction** - Predicts target movement
4. **Spinning Top Detection** - Identifies and targets spinning armor plates
5. **Energy Mechanism Detection** - Identifies and targets the energy mechanism (windmill)

## Key Features

- Multi-threaded architecture for efficient processing
- Support for both USB and Daheng cameras
- Real-time detection and tracking
- Kalman filtering for motion prediction
- Communication with STM32 control system
- Configurable parameters via XML

## System Requirements

- Ubuntu 16.04
- Qt Community 5.8.0
- OpenCV 3.4.2
- GCC 5.4.0
- C++11 standard
- Eigen matrix computation library
- libsvm classifier
- Daheng camera SDK

## Contact Information

For questions or support, contact the original developers:
- Jia Peng Xie (QQ: 460857545, WeChat: xie__1024)
- Hong Qing Ye (QQ: 958047328, WeChat: yhq38414942)

## License

This software is only for use in RoboMaster competitions or for team exchanges. It may not be used for commercial purposes. The Horizon team of North China University of Technology reserves the right of final interpretation.

