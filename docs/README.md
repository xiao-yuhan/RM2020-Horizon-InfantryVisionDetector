# RM2020-Horizon-InfantryVisionDetector Documentation

## Overview

This repository contains the vision detection system developed by the Horizon team from North China University of Technology for the RoboMaster 2020 competition. The system is designed for infantry robots and includes various detection and tracking algorithms for armor plates, spinning tops, and energy mechanisms.

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Installation](#installation)
3. [Project Structure](#project-structure)
4. [Core Modules](#core-modules)
   - [Armor Detection](#armor-detection)
   - [Angle Solving](#angle-solving)
   - [Motion Prediction](#motion-prediction)
   - [Spinning Top Detection](#spinning-top-detection)
   - [Energy Mechanism Detection](#energy-mechanism-detection)
5. [Communication Protocol](#communication-protocol)
6. [Configuration](#configuration)
7. [Usage](#usage)
8. [Future Development](#future-development)

## System Requirements

- Ubuntu 16.04
- Qt Community 5.8.0
- OpenCV 3.4.2
- GCC 5.4.0
- C++11 standard
- Eigen matrix computation library
- libsvm classifier
- Daheng camera SDK

## Installation

### Installing Dependencies

1. **Install Eigen Library**:
   Follow the instructions at [Eigen Installation Guide](https://blog.csdn.net/weixin_42587961/article/details/94011659)

2. **Install OpenCV**:
   Follow the instructions at [OpenCV Installation Guide](https://www.jianshu.com/p/f646448da265)

3. **Install Daheng Camera SDK**:
   Download from [Daheng Imaging Official Website](https://www.daheng-imaging.com/index.aspx)

### Building the Project

1. Open the project file `RM2020-Horizon-InfantryVisionDetector.pro` with Qt Creator
2. Configure the build settings
3. Build the project

## Project Structure

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
├─SendRecive                        // Communication with STM32
│      CRC_Check.cpp                // CRC verification  
│      RemoteController.cpp         // Send/receive thread control
│      serial.cpp                   // Send/receive function construction
│        
├─video                             // Camera interfaces
│       DaHengCamera.cpp            // Daheng camera Galaxy series USB3.0 interface
│       VideoCapture.cpp            // USB camera interface
│  DrawCurve.cpp                    // Waveform drawing
│  RM2020-Horizon-InfantryVisionDetector.pro     
│  main.cpp                         // Main function, thread creation
│  Variables.cpp                    // Global variable definitions, macros, utility functions
```

## Core Modules

### Armor Detection

The armor detection module uses color-based segmentation and morphological operations to identify potential armor plates on enemy robots.

**Process Flow**:
1. Color segmentation using weighted RGB channels and HSV thresholding
2. Grayscale conversion and binary thresholding
3. Morphological operations to clean up the binary image
4. Contour detection and ellipse fitting to find light bars
5. Pairing light bars to form potential armor plates
6. Secondary filtering to remove connected armor plates
7. Tracking armor plates between frames for continuous detection

![Armor Detection Process](Image/装甲检测流程图.png)

### Angle Solving

The angle solving module calculates the 3D position of detected armor plates relative to the camera and determines the pitch and yaw angles needed for targeting.

**Features**:
- Uses PnP (Perspective-n-Point) algorithm for pose estimation
- Converts camera coordinates to gimbal-relative coordinates
- Calculates shooting angles with gravity compensation

### Motion Prediction

The motion prediction module uses Kalman filtering to predict the future position of moving targets.

**Process Flow**:
1. Calculate target velocity based on position changes between frames
2. Apply Kalman filtering to estimate optimal position and velocity
3. Predict future position based on estimated velocity
4. Convert predicted position to gimbal angles

![Motion Prediction Process](Image/移动预测流程图.png)

### Spinning Top Detection

The spinning top detection module identifies spinning armor plates and predicts their rotation.

**Process Flow**:
1. Estimate the relative angle of the armor plate using the pinhole camera model
2. Determine left/right relationship based on light bar positions
3. Analyze angle and position changes to identify spinning motion
4. Use least squares method to calculate rotation center and radius
5. Apply Kalman filtering to estimate rotation speed
6. Predict future position for targeting

![Spinning Top Detection Process](Image/陀螺检测流程图.png)

### Energy Mechanism Detection

The energy mechanism detection module identifies and targets the energy mechanism (windmill) in the competition.

**Process Flow**:
1. Grayscale conversion and binary thresholding
2. Contour detection
3. Ellipse fitting and feature-based filtering to find fan blades
4. Contour hierarchy analysis to identify target fan blade
5. Single-point distance measurement to get relative coordinates
6. Three-point fitting to find the center of rotation
7. Calculate the normal vector of the windmill plane
8. Apply Kalman filtering to estimate rotation speed
9. Predict target position for shooting

![Energy Mechanism Detection Process](Image/能量机关检测流程图.png)

## Communication Protocol

### Receiving from STM32

| Byte0 | Byte1-4 | Byte5 | Byte6-9 | Byte10 | Byte11 | Byte12 | Byte13 |
|-------|---------|-------|---------|--------|--------|--------|--------|
| 0xaa (header) | YawAngleData | YawAngleSymbol | PitchAngleData | PitchAngleSymbol | ShootSpeedGrade | IsShootBuff | 0xbb (footer) |

- **YawAngleData**: Current yaw angle of the robot's gimbal (float)
- **YawAngleSymbol**: Sign of the yaw angle (0 = negative, 1 = positive)
- **PitchAngleData**: Current pitch angle of the robot's gimbal (float)
- **PitchAngleSymbol**: Sign of the pitch angle (0 = negative, 1 = positive)
- **ShootSpeedGrade**: Current shooting speed level
- **IsShootBuff**: Mode selection (0 = normal mode, 1 = energy mechanism mode)

### Sending to STM32

| Byte0 | Byte1-4 | Byte5 | Byte6-9 | Byte10 | Byte11 | Byte12 | Byte13-14 | Byte15 |
|-------|---------|-------|---------|--------|--------|--------|-----------|--------|
| 0xaa (header) | PitchAngleData | PitchAngleSymbol | YawAngleData | YawAngleSymbol | IsHaveArmor | distance | IsShoot | CRC_Check |

- **PitchAngleData**: Target pitch angle (float)
- **PitchAngleSymbol**: Sign of the pitch angle (0 = negative, 1 = positive)
- **YawAngleData**: Target yaw angle (float)
- **YawAngleSymbol**: Sign of the yaw angle (0 = negative, 1 = positive)
- **IsHaveArmor**: Whether a target is detected (0 = no, 1 = yes)
- **distance**: Distance to the target in decimeters (integer)
- **IsShoot**: Whether to shoot (0 = no, 1 = yes)
- **CRC_Check**: CRC verification bits

## Configuration

The system uses an XML configuration file to store parameters. The file path can be modified in `Variables.cpp`.

Key configuration parameters include:
- RGB thresholding values
- HSV color filtering ranges
- Camera exposure settings
- Detection thresholds

Parameters can be saved during runtime by pressing 's' when the display window is active.

## Usage

### Running the Program

1. Ensure all dependencies are installed
2. Build the project using Qt Creator
3. Run the executable

### Switching Modes

The system supports different operating modes that can be configured in `FindArmor/ImageProcess.cpp`:
- Video mode (using pre-recorded videos)
- USB camera mode
- Daheng camera mode
- Combined USB and Daheng camera mode

To switch between modes, uncomment/comment the corresponding macro definitions:
```cpp
//#define CAMERA_DEBUG                // Enable camera debugging
//#define USING_BUFF_DETECTOR         // Enable energy mechanism detection
//#define USING_VIDEO_BUFF            // Use video for energy mechanism debugging
//#define USING_VIDEO                 // Use video for debugging
//#define USING_CAMERA_DAHENG         // Use Daheng camera
#define USING_CAMERA_USB              // Use USB camera
```

### Debugging

In debug mode, the system provides trackbars for adjusting parameters in real-time:
- Exposure settings
- RGB weights
- HSV thresholds
- Binary thresholds

## Future Development

1. **Improved Recognition Algorithms**:
   - Enhance armor number recognition accuracy in low-light conditions
   - Explore neural network approaches for more robust detection

2. **CUDA Acceleration**:
   - Implement GPU acceleration using CUDA on TX2 platform

3. **Enhanced Motion Prediction**:
   - Develop more effective algorithms for variable-speed targets

4. **Improved Spinning Top Detection**:
   - Increase hit rate for spinning targets

5. **Camera Calibration**:
   - Develop more efficient methods for camera-to-gimbal offset calibration

6. **Parameter Tuning Interface**:
   - Create a more user-friendly interface for parameter adjustment

## Contributors

- Jia Peng Xie (QQ: 460857545, WeChat: xie__1024)
- Hong Qing Ye (QQ: 958047328, WeChat: yhq38414942)

## License

This software is only for use in RoboMaster competitions or for team exchanges. It may not be used for commercial purposes. The Horizon team of North China University of Technology reserves the right of final interpretation.

