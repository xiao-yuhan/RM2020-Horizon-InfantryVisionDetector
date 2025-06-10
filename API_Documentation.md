# RM2020-Horizon-InfantryVisionDetector API Documentation

This document provides detailed information about the key classes, functions, and interfaces in the RM2020-Horizon-InfantryVisionDetector system.

## Table of Contents
1. [Camera Interfaces](#camera-interfaces)
2. [Image Processing](#image-processing)
3. [Armor Detection](#armor-detection)
4. [Angle Solving](#angle-solving)
5. [Motion Prediction](#motion-prediction)
6. [Spinning Top Detection](#spinning-top-detection)
7. [Energy Mechanism Detection](#energy-mechanism-detection)
8. [Serial Communication](#serial-communication)
9. [Utility Functions](#utility-functions)

## Camera Interfaces

### DaHengCamera Class

Located in `video/DaHengCamera.cpp`

```cpp
class DaHengCamera {
public:
    // Initialize and connect to the camera
    void StartDevice(int DeviceIndex);
    
    // Set resolution (width, height)
    void SetResolution(int width_scale, int height_scale);
    
    // Start streaming
    void SetStreamOn();
    
    // Set exposure time in microseconds
    void SetExposureTime(int ExposureTime);
    
    // Set gain (channel, value)
    void SetGAIN(int channel, int value);
    
    // Set white balance mode (0 for manual, 1 for auto)
    void Set_BALANCE_AUTO(int value);
    
    // Get frame from camera
    bool GetMat(Mat &src);
};
```

### VideoCapture Class

Located in `video/VideoCapture.cpp`

```cpp
namespace myown {
class VideoCapture {
public:
    // Initialize with device path and buffer size
    VideoCapture(const char* device, unsigned int size);
    
    // Set exposure time
    void setExpousureTime(bool auto_exp, int t);
    
    // Set aperture
    void setApertureNumber(int a);
    
    // Set video format (width, height, format)
    void setVideoFormat(unsigned int width, unsigned int height, unsigned int format);
    
    // Start streaming
    void startStream();
    
    // Get frame
    bool grab(cv::Mat& frame);
    
    // Display camera information
    void info();
};
}
```

## Image Processing

### ImageProcess Class

Located in `FindArmor/ImageProcess.cpp`

```cpp
class ImageProcess {
public:
    ImageProcess();
    
    // Producer thread - captures frames from camera
    void ImageProducter();
    
    // Consumer thread - processes frames
    void ImageConsumer();
    
    // Draw feedback on image
    void DrawImage(Mat &src);
};
```

## Armor Detection

### LongFindArmor Class

Located in `FindArmor/LongFindArmor.cpp`

```cpp
class LongFindArmor {
public:
    // Process image to find armor plates
    void ImagePretreatment(Mat &src, ArmorBox &box, int &findState);
    
    // Detect light bars in the image
    void findLightBar(Mat &src, vector<LightBar> &lightBars);
    
    // Match light bars to form armor plates
    void matchLightBar(vector<LightBar> &lightBars, vector<ArmorBox> &armors);
    
    // Filter false positive armor detections
    void filterArmor(vector<ArmorBox> &armors, vector<ArmorBox> &finalArmors);
    
    // Recognize the number on the armor plate
    int getArmorNumber(Mat &src, ArmorBox &armor);
};
```

## Angle Solving

### AngleSolver Class

Located in `FindArmor/AngleSolver.cpp`

```cpp
class AngleSolver {
public:
    // Initialize with camera parameters
    void setCameraParam(const Mat& camera_matrix, const Mat& dist_coeff);
    
    // Set target size (width, height)
    void setTargetSize(double width, double height);
    
    // Calculate target position using PnP
    void getAngle(ArmorBox &box, double &yaw, double &pitch, double &distance);
    
    // Predict target position based on motion
    void getAnglePrediction(ArmorBox &box, double &yaw, double &pitch, double &distance);
};
```

## Motion Prediction

### Filter Class

Located in `Filter/Filter.cpp`

```cpp
class Filter {
public:
    // Initialize Kalman filter
    void initKalman();
    
    // Update filter with new measurement
    void setMeasurementMatrixH();
    
    // Predict next position
    void KalmanFilterPrediction(double &x, double &y, double &z);
    
    // Correct prediction with measurement
    void KalmanFilterCorrection(double x, double y, double z);
};
```

## Spinning Top Detection

### ShootTuoluo Class

Located in `FindArmor/ShootTuoluo.cpp`

```cpp
class ShootTuoluo {
public:
    // Detect if target is spinning
    bool isTuoluo(ArmorBox &box);
    
    // Calculate spinning center and radius
    void calculateCenter(vector<Point2f> &points);
    
    // Predict spinning target position
    void predictTuoluo(double &yaw, double &pitch, double &distance);
};
```

## Energy Mechanism Detection

### FindBuff Class

Located in `BuffDetector/FindBuff.cpp`

```cpp
class FindBuff {
public:
    // Process image to find energy mechanism
    void ImagePretreatment(Mat &src, BuffBox &box, int &findState);
    
    // Detect fan blades
    void findFanBlade(Mat &src, vector<FanBlade> &fanBlades);
    
    // Identify target fan blade
    void findTargetBlade(vector<FanBlade> &fanBlades, FanBlade &targetBlade);
};
```

### BuffAngleSolver Class

Located in `BuffDetector/BuffAngleSolver.cpp`

```cpp
class BuffAngleSolver {
public:
    // Calculate target position
    void getBuffAngle(BuffBox &box, double &yaw, double &pitch, double &distance);
    
    // Predict rotation and calculate shooting point
    void predictBuffAngle(BuffBox &box, double &yaw, double &pitch, double &distance);
};
```

## Serial Communication

### RemoteController Class

Located in `SendRecive/RemoteController.cpp`

```cpp
class RemoteController {
public:
    // Thread for receiving data from STM32
    static void paraReceiver(RemoteController &controller);
    
    // Thread for sending data to STM32
    static void paraGetCar(RemoteController &controller);
    
    // Parse received data
    void paraDecoder(unsigned char *buffer);
    
    // Encode data to send
    void paraEncoder(unsigned char *buffer);
};
```

### Serial Class

Located in `SendRecive/serial.cpp`

```cpp
class Serial {
public:
    // Open serial port
    int openPort(const char *dev);
    
    // Configure serial port
    int configurePort(int baud_rate, int data_bits, char parity, int stop_bits);
    
    // Send data
    int sendData(const unsigned char *data, int length);
    
    // Receive data
    int receiveData(unsigned char *data, int length);
};
```

## Utility Functions

Located in `Variables.cpp`

```cpp
// Calculate circle center from three points (2D)
Point2f GetCircle(Point2f point_one, Point2f point_two, Point2f point_three);

// Calculate circle center from three points (3D)
Point3f solveCenterPointOfCircle(vector<Point3f> pt);

// Calculate angle between three points (2D)
double GetAngle(Point2f a, Point2f b, Point2f c);

// Calculate angle between three points (3D)
double GetAngle(Point3f a, Point3f b, Point3f c);
```

## Data Structures

### ArmorBox Structure

```cpp
struct ArmorBox {
    vector<Point2f> armorVertices;  // Four corners of armor plate
    Point2f center;                 // Center point
    int armorNum;                   // Armor number
    double armorAngle;              // Armor angle
    LightBar leftLight;             // Left light bar
    LightBar rightLight;            // Right light bar
};
```

### BuffBox Structure

```cpp
struct BuffBox {
    vector<Point2f> fanVertices;    // Fan blade vertices
    Point2f center;                 // Center point
    double rotationSpeed;           // Rotation speed
    double radius;                  // Radius of rotation
    bool isClockwise;               // Rotation direction
};
```

### CarData Structure

```cpp
struct CarData {
    float yawAngle;                 // Current yaw angle
    float pitchAngle;               // Current pitch angle
    int shootSpeed;                 // Shooting speed grade
    bool isShootBuff;               // Whether to shoot energy mechanism
};
```

