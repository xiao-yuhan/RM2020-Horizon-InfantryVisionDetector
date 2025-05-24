# RM2020-Horizon-InfantryVisionDetector Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [System Overview](#system-overview)
3. [Installation and Setup](#installation-and-setup)
4. [Architecture](#architecture)
5. [Core Components](#core-components)
6. [Configuration](#configuration)
7. [Communication Protocol](#communication-protocol)
8. [Usage Guide](#usage-guide)
9. [Development Guide](#development-guide)
10. [Future Improvements](#future-improvements)

## Introduction

RM2020-Horizon-InfantryVisionDetector is a computer vision system developed by the Horizon team at North China University of Science and Technology for the 2020 RoboMaster competition. The system is designed for infantry robots to detect and track armor plates, spinning tops, and energy mechanisms.

The software implements various computer vision algorithms for target detection, angle solving, motion prediction, and communication with the robot's control system. It's optimized for real-time performance in competition environments.

## System Overview

The system provides the following key functionalities:

1. **Armor Detection**: Detects enemy robot armor plates using color segmentation and contour analysis
2. **Spinning Top Detection**: Identifies and tracks spinning tops with specialized algorithms
3. **Motion Prediction**: Uses Kalman filtering to predict target movement for accurate shooting
4. **Energy Mechanism Recognition**: Detects and tracks the energy mechanism (big rune) for precise shooting
5. **Angle Solving**: Calculates the 3D position of targets and converts to pitch/yaw angles
6. **Serial Communication**: Communicates with the STM32 control system to send targeting data

The system is designed to run on an embedded platform with camera inputs, processing frames in real-time to provide targeting information to the robot's control system.

## Installation and Setup

### Dependencies

- Ubuntu 16.04
- Qt Community 5.8.0
- OpenCV 3.4.2
- GCC 5.4.0
- C++11 standard
- Eigen matrix computation library
- libsvm classifier
- Daheng camera SDK

### Installation Steps

1. **Install Eigen library**:
   ```bash
   # Follow the guide at: https://blog.csdn.net/weixin_42587961/article/details/94011659
   ```

2. **Install OpenCV**:
   ```bash
   # Follow the guide at: https://www.jianshu.com/p/f646448da265
   ```

3. **Install Daheng Imaging SDK**:
   Download from the official website: https://www.daheng-imaging.com/index.aspx

4. **Clone the repository**:
   ```bash
   git clone https://github.com/xiao-yuhan/RM2020-Horizon-InfantryVisionDetector.git
   ```

5. **Open the project in Qt**:
   ```bash
   # Open the .pro file in Qt Creator
   ```

## Architecture

The system is built around a multi-threaded architecture to maximize CPU utilization and processing speed:

1. **Image Production Thread**: Captures frames from cameras
2. **Image Consumption Thread**: Processes frames to detect targets
3. **Data Reception Thread**: Receives data from the STM32 control system
4. **Data Transmission Thread**: Sends targeting data to the STM32 control system

This producer-consumer pattern allows for asynchronous image capture and processing, improving overall performance.

### File Structure

```
InfantryVisionDetector/  
├─include                           // Header files
│ 
├─BuffDetector                      // Energy mechanism detection and shooting
│      BuffAngleSolver.cpp          // Energy mechanism angle solving and prediction
│      FindBuff.cpp                 // Energy mechanism detection
│        
├─Filter                            // Filtering tools
│      Filter.cpp  
│        
├─FindArmor                         // Armor detection, prediction, spinning top detection
│      AngleSolver.cpp              // Angle solving, motion prediction, spinning top shooting     
│      ImageProcess.cpp             // Image acquisition and processing thread control
│      LongFindArmor.cpp            // Image processing for armor information
│      ShootTuoluo.cpp              // Spinning top detection
│        
├─GetNum                            // Number recognition
│      GetNum.cpp  
│      svm.cpp  
│        
├─SendRecive                        // Communication with STM32 controller
│      CRC_Check.cpp                // CRC verification  
│      RemoteController.cpp         // Send/receive thread control
│      serial.cpp                   // Send/receive function construction
│        
├─video                             // Camera interfaces
│       DaHengCamera.cpp            // Daheng camera USB3.0 interface
│       VideoCapture.cpp            // USB camera interface
│  DrawCurve.cpp                    // Waveform drawing
│  InfantryVisionDetector.pro     
│  InfantryVisionDetector.pro.user  
│  main.cpp                         // Main function, thread creation
│  Readme.md    
│  Variables.cpp                    // Global variable definitions, macros, utility functions
```

## Core Components

### 1. Armor Detection

The armor detection module identifies enemy robot armor plates using the following approach:

1. Color segmentation using weighted RGB and HSV thresholding
2. Grayscale thresholding and morphological operations
3. Contour detection and ellipse fitting to find light bars
4. Pairing light bars to form potential armor plates
5. Filtering false positives using geometric constraints
6. Tracking armor plates across frames for consistent detection

The system can reliably detect armor plates up to 4 meters away, with the primary focus on targets within 3 meters.

### 2. Spinning Top Detection

The spinning top detection module identifies rotating armor plates using:

1. Estimation of the relative angle of the armor plate using the pinhole camera model
2. Analysis of angle changes over time to identify spinning patterns
3. Least squares fitting to calculate the center and radius of rotation
4. Kalman filtering to estimate rotation speed and predict future positions

The system can achieve approximately 60-70% hit rate on spinning targets.

### 3. Motion Prediction

The motion prediction system uses Kalman filtering to predict target movement:

1. Conversion of camera coordinates to absolute coordinates relative to the turret
2. Calculation of target velocity in x, y, and z axes
3. Kalman filtering to obtain optimal estimates of position and velocity
4. Prediction of future position based on estimated velocity
5. Conversion back to relative angles for the turret

The system achieves approximately 80% hit rate on moving targets at 3 meters.

### 4. Energy Mechanism Recognition

The energy mechanism (big rune) detection module:

1. Grayscale thresholding and morphological operations
2. Contour detection to find all potential fan blades
3. Ellipse fitting and feature filtering to identify fan blades
4. Analysis of contour hierarchy to find the target fan blade

### 5. Angle Solving

The angle solving module calculates the 3D position of targets and converts to pitch/yaw angles:

1. Uses PnP (Perspective-n-Point) pose estimation to calculate target position
2. Converts camera coordinates to turret coordinates
3. Calculates pitch and yaw angles for the turret
4. Adds prediction offsets for moving targets

## Configuration

The system uses XML configuration files to store parameters:

- `DebugParam.xml`: Contains parameters for image processing, thresholds, and camera settings

Key configurable parameters include:

- RGB and HSV thresholding values
- Camera exposure and gain settings
- Detection thresholds
- Filtering parameters

In debug mode, the system provides trackbars for real-time parameter adjustment.

## Communication Protocol

### STM32 to Vision System

| Byte0 | Byte1-4 | Byte5 | Byte6-9 | Byte10 | Byte11 | Byte12 | Byte13 |
|-------|---------|-------|---------|--------|--------|--------|--------|
| 0xaa (header) | Yaw angle data | Yaw angle symbol | Pitch angle data | Pitch angle symbol | Shoot speed grade | Is shoot buff | 0xbb (tail) |

### Vision System to STM32

| Byte0 | Byte1-4 | Byte5 | Byte6-9 | Byte10 | Byte11 | Byte12 | Byte13 | Byte14-15 |
|-------|---------|-------|---------|--------|--------|--------|--------|-----------|
| 0xaa (header) | Pitch angle data | Pitch angle symbol | Yaw angle data | Yaw angle symbol | Is have armor | Distance | Is shoot | CRC check |

## Usage Guide

### Running the System

1. Configure the camera settings in the XML file
2. Set the appropriate detection mode in `ImageProcess.cpp` by uncommenting the desired mode:
   - `USING_VIDEO`: Use video files for debugging
   - `USING_CAMERA_DAHENG`: Use Daheng camera
   - `USING_CAMERA_USB`: Use USB camera
   - `USING_BUFF_DETECTOR`: Enable energy mechanism detection

3. Build and run the project in Qt Creator

### Debug Mode

In debug mode, the system provides several windows for visualization and parameter adjustment:

- BGR window: Adjust RGB thresholding parameters
- R-HSV/B-HSV windows: Adjust HSV thresholding parameters
- Exposure window: Adjust camera exposure settings

## Development Guide

### Adding New Features

1. **New Detection Algorithm**:
   - Add header file in `include/`
   - Implement the algorithm in an appropriate directory
   - Integrate with the image processing pipeline in `ImageProcess.cpp`

2. **New Camera Support**:
   - Implement a camera interface class similar to `DaHengCamera.cpp` or `VideoCapture.cpp`
   - Add appropriate initialization in `ImageProcess::ImageProducter()`

3. **New Communication Protocol**:
   - Modify the protocol definitions in `RemoteController.cpp`
   - Update the CRC check if necessary

### Code Style

- Use camelCase for function names and variables
- Use PascalCase for class names
- Add appropriate comments for complex algorithms
- Document parameter meanings in header files

## Future Improvements

As mentioned in the original README, several improvements are planned:

1. **Recognition Algorithm Optimization**:
   - Improve number recognition accuracy in poor lighting conditions
   - Explore neural network approaches for more robust detection
   - Consider using CUDA acceleration on TX2

2. **Motion Prediction Enhancement**:
   - Develop more effective algorithms for variable-speed targets
   - Improve response time to acceleration changes

3. **Spinning Top Detection**:
   - Increase hit rate beyond the current 60-70%
   - Improve prediction accuracy for varying rotation speeds

4. **Camera Calibration**:
   - Develop a more efficient method for calibrating the camera relative to the turret
   - Reduce the complexity of the current 6-parameter adjustment process

5. **Parameter Tuning Interface**:
   - Create a more user-friendly interface for parameter adjustment
   - Implement parameter auto-tuning for different environments

---

## Contact Information

For questions or discussions about this code, please contact:
- Xie Jiapeng: QQ: 460857545, WeChat: xie__1024 (two underscores)
- Ye Hongqing: QQ: 958047328, WeChat: yhq38414942

**Disclaimer**: This software is only for use in RoboMaster competitions or for team exchanges. It cannot be used for commercial purposes. The Horizon team of North China University of Science and Technology reserves the right of final interpretation.

