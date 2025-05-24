# Quick Start Guide

This guide will help you quickly set up and run the RM2020-Horizon-InfantryVisionDetector system.

## Prerequisites

Before starting, ensure you have the following installed:
- Ubuntu 16.04
- Qt Community 5.8.0 or higher
- OpenCV 3.4.2
- GCC 5.4.0 or higher
- Eigen matrix library
- libsvm

## Installation Steps

### 1. Clone the Repository

```bash
git clone https://github.com/xiao-yuhan/RM2020-Horizon-InfantryVisionDetector.git
cd RM2020-Horizon-InfantryVisionDetector
```

### 2. Install Dependencies

#### Install Eigen Library

```bash
sudo apt-get update
sudo apt-get install libeigen3-dev
```

#### Install OpenCV 3.4.2

```bash
# Install dependencies
sudo apt-get install build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

# Download and extract OpenCV
wget -O opencv-3.4.2.zip https://github.com/opencv/opencv/archive/3.4.2.zip
unzip opencv-3.4.2.zip
cd opencv-3.4.2

# Create build directory
mkdir build
cd build

# Configure
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..

# Build and install
make -j$(nproc)
sudo make install
```

#### Install Daheng Camera SDK

1. Download the SDK from [Daheng Imaging Official Website](https://www.daheng-imaging.com/index.aspx)
2. Follow the installation instructions provided with the SDK

### 3. Configure the Project

1. Open Qt Creator
2. Open the project file: `RM2020-Horizon-InfantryVisionDetector.pro`
3. Configure the build settings
4. Build the project

## Configuration

### Camera Setup

#### USB Camera

1. Connect your USB camera
2. Find the device path (usually `/dev/video0`)
3. Update the device path in `Variables.cpp`:
   ```cpp
   string USBDevicPath = "/dev/video0";
   ```

#### Daheng Camera

1. Connect your Daheng camera
2. Install the Daheng camera SDK
3. Configure the camera parameters in `ImageProcess.cpp`

### Detection Mode Configuration

Open `FindArmor/ImageProcess.cpp` and configure the detection mode by uncommenting/commenting the appropriate macros:

```cpp
// For USB camera mode
#define USING_CAMERA_USB

// For Daheng camera mode
//#define USING_CAMERA_DAHENG

// For video debugging mode
//#define USING_VIDEO

// For energy mechanism detection
//#define USING_BUFF_DETECTOR
```

### Serial Communication Setup

1. Connect your STM32 board to the computer
2. Find the serial port (usually `/dev/ttyUSB0`)
3. Update the serial port in `RemoteController.cpp`

## Running the System

### 1. Build the Project

```bash
cd RM2020-Horizon-InfantryVisionDetector
qmake
make
```

### 2. Run the Executable

```bash
./RM2020-Horizon-InfantryVisionDetector
```

## Basic Usage

### Keyboard Controls

- Press `s` to save current parameters to the XML configuration file
- Press `Esc` to exit the program

### Parameter Adjustment

In debug mode, you can adjust parameters using trackbars:
- RGB thresholds
- HSV color ranges
- Exposure settings
- Binary thresholds

### Viewing Output

The system displays the processed image with detection results:
- Green boxes: Tracked armor plates
- Purple boxes: Newly detected armor plates
- Red boxes: Buffered armor plates (during frame drops)
- Text overlay: Position and angle information

## Troubleshooting

### Common Issues

#### Camera Not Detected

1. Check if the camera is properly connected
2. Verify the device path in `Variables.cpp`
3. Ensure you have the correct permissions:
   ```bash
   sudo chmod 777 /dev/video0
   ```

#### Build Errors

1. Ensure all dependencies are installed
2. Check Qt and OpenCV versions
3. Verify include paths in the project file

#### Detection Issues

1. Adjust lighting conditions
2. Modify HSV thresholds in the XML configuration file
3. Check camera exposure settings

## Next Steps

After getting the basic system running, you can:

1. Calibrate the camera for your specific setup
2. Adjust detection parameters for your environment
3. Integrate with your robot control system
4. Experiment with different detection modes

For more detailed information, refer to the [Technical Documentation](technical_details.md).

