# API Reference

This document provides a reference for the key classes, structures, and functions in the RM2020-Horizon-InfantryVisionDetector system.

## Table of Contents

- [Data Structures](#data-structures)
- [Classes](#classes)
- [Global Functions](#global-functions)
- [Configuration Parameters](#configuration-parameters)

## Data Structures

### `ImageDate`

Structure for storing captured images and associated data.

```cpp
struct ImageDate {
    Mat SrcImage;              // The captured image
    ImageGetMode mode;         // Image acquisition mode
    CarData ReciveStm32;       // Data received from STM32
};
```

### `CarData`

Structure for storing data received from the STM32.

```cpp
struct CarData {
    float pitch = 0;           // Current pitch angle
    float yaw = 0;             // Current yaw angle
    float ShootSpeed = 16;     // Current shooting speed
    bool IsBuffMode = false;   // Whether in energy mechanism mode
    double BeginToNowTime = 0; // Time since start
};
```

### `led`

Structure for storing light bar information.

```cpp
struct led {
    RotatedRect box;                   // Fitted ellipse
    Point2f led_on = Point2f(0,0);     // Top point of light bar
    Point2f led_down = Point2f(0,0);   // Bottom point of light bar
};
```

### `RM_ArmorDate`

Structure for storing detected armor plate information.

```cpp
struct RM_ArmorDate {
    float tx = 0;              // X-coordinate in camera frame
    float ty = 0;              // Y-coordinate in camera frame
    float tz = 0;              // Z-coordinate in camera frame
    float pitch = 0;           // Pitch angle for targeting
    float yaw = 0;             // Yaw angle for targeting
    bool IsSmall = true;       // Whether it's a small armor plate
    bool IsTuoluo = false;     // Whether it's in spinning state
    double get_waith = 0;      // Width of spinning top
    double distance = 0;       // Distance to target
    led leds[2];               // Light bars
    Point2f point[4];          // Four corners of armor plate
    bool IsShooting = true;    // Whether to shoot
};
```

### `RM_BuffData`

Structure for storing energy mechanism information.

```cpp
struct RM_BuffData {
    float tx = 0;              // X-coordinate in camera frame
    float ty = 0;              // Y-coordinate in camera frame
    float tz = 0;              // Z-coordinate in camera frame
    float pitch = 0;           // Pitch angle for targeting
    float yaw = 0;             // Yaw angle for targeting
    RotatedRect box;           // Fitted ellipse
    Point2f point[4];          // Four corners of fan blade
};
```

### `ArmorDate`

Structure for returning armor detection results.

```cpp
struct ArmorDate {
    RM_ArmorDate Armor;        // Armor information
    pattern status = stop;     // Detection status
};
```

### `TuoluoData`

Structure for storing spinning top information.

```cpp
struct TuoluoData {
    bool isTuoluo = false;                 // Whether it's a spinning top
    float R;                               // Radius of rotation
    float angle;                           // Current angle
    Point2f center;                        // Center of rotation
    TuoluoStatus status;                   // Current status
    TuoluoRunDirection runDirection;       // Rotation direction
    float spinSpeed;                       // Angular velocity
};
```

### `Angle_t`

Structure for returning targeting angles.

```cpp
typedef struct {
    float pitch;               // Pitch angle
    float yaw;                 // Yaw angle
    float t;                   // Time to target
    float angle;               // Lead angle
} Angle_t;
```

## Classes

### `ImageProcess`

Main class for image acquisition and processing.

#### Methods

```cpp
ImageProcess();                                    // Constructor
void ImageProducter();                             // Image producer thread function
void ImageConsumer();                              // Image consumer thread function
void DrawImage(Mat Src, ArmorDate BestArmorDate);  // Draw detection results
void SaveData();                                   // Save parameters to XML
```

### `LongFindArmor`

Class for armor detection.

#### Methods

```cpp
LongFindArmor();                                   // Constructor
ArmorDate GetArmorDate(Mat Src);                   // Main detection function
```

### `AngleSolver`

Class for angle solving and motion prediction.

#### Methods

```cpp
AngleSolver();                                                                  // Constructor
void GetArmorAngle(Mat Src, ArmorDate &BestArmor, Camera camera, CarData car);  // Calculate targeting angles
void BufferSetFilter(RM_ArmorDate armor, CarData car);                          // Handle frame drops
```

### `ShootTuoluo`

Class for spinning top detection.

#### Methods

```cpp
ShootTuoluo();                                                                  // Constructor
bool IsTuoluo(RM_ArmorDate &armor, CarData car);                                // Detect spinning motion
void GetTuoluoAngle(RM_ArmorDate &armor, CarData car);                          // Calculate targeting angles
```

### `FindBuff`

Class for energy mechanism detection.

#### Methods

```cpp
FindBuff();                                        // Constructor
RM_BuffData* BuffModeSwitch(Mat Src);              // Main detection function
```

### `BuffAngleSolver`

Class for energy mechanism angle solving.

#### Methods

```cpp
BuffAngleSolver();                                                              // Constructor
void GetBuffShootAngle(RM_BuffData* Buffs, BuffStatus status, CarData car);     // Calculate targeting angles
```

### `RemoteController`

Class for communication with STM32.

#### Methods

```cpp
RemoteController();                                // Constructor
void paraReceiver();                               // Data receiver thread function
void paraGetCar();                                 // Data sender thread function
void ArmorToData(ArmorDate armor, Camera camera);  // Convert armor data to transmission format
```

### `DaHengCamera`

Class for Daheng camera interface.

#### Methods

```cpp
DaHengCamera();                                    // Constructor
bool StartDevice(int DeviceIndex);                 // Initialize camera
void SetResolution(int width_scale, int height_scale); // Set resolution
void SetStreamOn();                                // Start streaming
void SetExposureTime(int ExposureTime);            // Set exposure time
void SetGAIN(int AutoGain, int GainValue);         // Set gain
void Set_BALANCE_AUTO(int value);                  // Set white balance
bool GetMat(Mat& src);                             // Get frame
```

### `myown::VideoCapture`

Class for USB camera interface.

#### Methods

```cpp
VideoCapture(const char* device, int channel);     // Constructor
bool isOpened();                                   // Check if camera is open
bool setVideoFormat(int width, int height, int format); // Set video format
bool setFPS(int fps);                              // Set frame rate
bool setExpousureTime(bool auto_exp, int t);       // Set exposure time
bool setApertureNumber(int value);                 // Set aperture
bool startStream();                                // Start streaming
bool stopStream();                                 // Stop streaming
bool operator>>(cv::Mat& src);                     // Get frame
void info();                                       // Print camera info
```

## Global Functions

### Geometric Utility Functions

```cpp
double GetDistance(Point2f a, Point2f b);                  // Calculate distance between 2D points
double GetDistance(Point3f a, Point3f b);                  // Calculate distance between 3D points
Point2f GetCircle(Point2f point_one, Point2f point_two, Point2f point_three); // Find circle from 3 points
Point3f solveCenterPointOfCircle(vector<Point3f> pt);      // Find 3D circle center
Point2f getArmorCenter(Point2f p[4]);                      // Calculate armor center
float getArmorAngle(Point2f p[4]);                         // Calculate armor angle
double GetAngle(Point2f a, Point2f b, Point2f c);          // Calculate angle between 2D points
double GetAngle(Point3f a, Point3f b, Point3f c);          // Calculate angle between 3D points
```

### System Functions

```cpp
void getDate();                                            // Load parameters from XML
```

## Configuration Parameters

### Color Thresholds

```cpp
// RGB thresholds
extern int GrayValue;          // Grayscale threshold
extern int RGrayWeightValue;   // Red channel weight
extern int BGrayWeightValue;   // Blue channel weight

// HSV thresholds for red armor
extern int RLowH, RHighH;      // Hue range
extern int RLowS, RHighS;      // Saturation range
extern int RLowV, RHighV;      // Value range

// HSV thresholds for blue armor
extern int BLowH, BHighH;      // Hue range
extern int BLowS, BHighS;      // Saturation range
extern int BLowV, BHighV;      // Value range

extern int V_ts;               // Value threshold
```

### Camera Parameters

```cpp
// USB camera
extern int UsbExpTime;         // Exposure time
extern int UsbAspTime;         // Aperture value

// Daheng camera
extern int GXExpTime;          // Exposure time
extern int GXGain;             // Gain value
```

### System Configuration

```cpp
// File paths
extern string VideoPath;       // Video path for debugging
extern string BuffVideoPath;   // Energy mechanism video path
extern string USBDevicPath;    // USB camera device path
extern string BuffDevicPath;   // Energy mechanism camera device path
extern string paramFileName;   // Parameter file path

// Target configuration
extern ArmorColor armor_color; // Target armor color
extern int ShootArmorNumber;   // Target armor number
```

### Enumerations

```cpp
// Shooting mode
enum pattern {
    FirstFind,                 // First detection
    Shoot,                     // Continuous shooting
    stop,                      // No target
    buffering                  // Frame drop handling
};

// Camera type
typedef enum {
    UsingLong,                 // Long-focus camera
    UsingGuang                 // Wide-angle camera
} Camera;

// Energy mechanism status
typedef enum {
    BUFF_FIRST_SHOOT,          // First detection
    BUFF_CONTINUE_SHOOT,       // Continuous shooting
    BUFF_BUFFER_SHOOT          // Frame drop handling
} BuffStatus;

// Image acquisition mode
typedef enum {
    USB,                       // USB camera
    DAHENG,                    // Daheng camera
    BUFF                       // Energy mechanism camera
} ImageGetMode;

// Armor color
enum ArmorColor {
    RED,                       // Red armor
    BLUE                       // Blue armor
};

// Spinning top status
enum TuoluoStatus {
    STILL,                     // Stationary
    SWITCH,                    // Switching
    RUN                        // Spinning
};

// Spinning direction
enum TuoluoRunDirection {
    LEFT,                      // Left rotation
    RIGHT                      // Right rotation
};
```

