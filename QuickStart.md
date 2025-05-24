# RM2020-Horizon-InfantryVisionDetector Quick Start Guide

This guide provides a quick introduction to get you started with the RM2020-Horizon-InfantryVisionDetector system.

## Prerequisites

Before you begin, ensure you have the following installed:
- Ubuntu 16.04
- Qt Community 5.8.0
- OpenCV 3.4.2
- GCC 5.4.0
- Eigen matrix library
- libsvm

## Installation

1. **Install Eigen library**:
   ```bash
   sudo apt-get install libeigen3-dev
   ```

2. **Install OpenCV 3.4.2**:
   ```bash
   # Install dependencies
   sudo apt-get install build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
   
   # Download and extract OpenCV
   wget -O opencv-3.4.2.zip https://github.com/opencv/opencv/archive/3.4.2.zip
   unzip opencv-3.4.2.zip
   
   # Build and install
   cd opencv-3.4.2
   mkdir build && cd build
   cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
   make -j$(nproc)
   sudo make install
   ```

3. **Install Qt 5.8.0**:
   Download from the Qt website and follow the installation instructions.

4. **Clone the repository**:
   ```bash
   git clone https://github.com/xiao-yuhan/RM2020-Horizon-InfantryVisionDetector.git
   cd RM2020-Horizon-InfantryVisionDetector
   ```

## Configuration

1. **Update file paths**:
   Open `Variables.cpp` and update the following paths to match your system:
   
   ```cpp
   string paramFileName = "/path/to/your/DebugParam.xml";
   string VideoPath = "/path/to/your/video.avi";
   string BuffVideoPath = "/path/to/your/buff_video.avi";
   string USBDevicPath = "/dev/videoX";  // Replace X with your camera device number
   string BuffDevicPath = "/dev/videoY";  // Replace Y with your second camera device number
   ```

2. **Configure camera mode**:
   Open `FindArmor/ImageProcess.cpp` and uncomment the appropriate mode:
   
   ```cpp
   // Uncomment one of these modes
   //#define CAMERA_DEBUG                  // Debug camera
   //#define USING_BUFF_DETECTOR           // Enable energy mechanism detection
   //#define USING_VIDEO_BUFF              // Use video for energy mechanism debugging
   //#define USING_VIDEO                   // Use video for debugging
   //#define USING_CAMERA_DAHENG           // Use Daheng camera
   #define USING_CAMERA_USB                // Use USB camera
   ```

## Running the System

1. **Open the project in Qt Creator**:
   ```bash
   qtcreator InfantryVisionDetector.pro
   ```

2. **Build the project**:
   Click the build button in Qt Creator or use the keyboard shortcut (Ctrl+B).

3. **Run the application**:
   Click the run button in Qt Creator or use the keyboard shortcut (Ctrl+R).

## Testing with Video Files

If you don't have a camera available, you can test the system with video files:

1. Uncomment the `USING_VIDEO` definition in `FindArmor/ImageProcess.cpp`
2. Ensure the `VideoPath` in `Variables.cpp` points to a valid video file
3. Build and run the application

## Understanding the Output

When running, the system will display several windows:

- **Main window**: Shows the camera feed with detected armor plates highlighted
- **BGR window**: Shows the binary image after color thresholding
- **R-HSV/B-HSV windows**: Show the HSV thresholding results for red and blue colors

The system will also output information to the console, including:
- Detection status
- Target coordinates
- Angle calculations
- Serial communication data

## Troubleshooting

### Camera Not Found
- Check if the camera device path is correct in `Variables.cpp`
- Ensure the camera is properly connected
- Try running `ls /dev/video*` to list available camera devices

### Build Errors
- Ensure all dependencies are properly installed
- Check that the paths in the project file are correct
- Make sure you're using the correct versions of libraries

### Runtime Errors
- Check the console output for error messages
- Verify that the XML configuration file exists and is properly formatted
- Ensure the camera is working by testing with another application

## Next Steps

After getting the basic system running, you can:

1. **Adjust parameters**: Modify the XML configuration file to optimize detection for your environment
2. **Explore the code**: Understand the different components and how they interact
3. **Connect to hardware**: Set up the serial connection to your STM32 controller
4. **Customize algorithms**: Modify the detection and prediction algorithms for your specific needs

For more detailed information, refer to the full documentation in the `Documentation.md` file.

