# Technical Documentation for RM2020-Horizon-InfantryVisionDetector

This document provides detailed technical information about the implementation of the vision detection system for the RoboMaster 2020 competition.

## System Architecture

The system is built on a multi-threaded architecture to optimize CPU utilization and processing speed:

1. **Image Producer Thread**: Captures frames from cameras
2. **Image Consumer Thread**: Processes frames and detects targets
3. **Data Receiver Thread**: Receives data from STM32
4. **Data Sender Thread**: Sends processed data to STM32

The producer-consumer pattern is used for image processing, allowing asynchronous capture and processing of frames.

## Armor Detection Module

### Implementation Details

The armor detection module is implemented in `LongFindArmor.cpp` and follows these steps:

1. **Color Segmentation**:
   - RGB channel weighting for initial segmentation
   - HSV color space filtering for red/blue armor detection
   - Combination of RGB and HSV filtering for robust detection

2. **Binary Image Processing**:
   - Grayscale conversion with weighted RGB channels
   - Binary thresholding
   - Morphological operations (erosion, dilation) to clean up the image

3. **Light Bar Detection**:
   - Contour finding using OpenCV's `findContours`
   - Ellipse fitting for each contour
   - Filtering based on aspect ratio, area, and angle

4. **Armor Plate Formation**:
   - Pairing light bars based on geometric constraints
   - Calculating the angle between light bars
   - Filtering based on distance, angle, and size ratios

5. **Armor Plate Filtering**:
   - Secondary filtering to remove connected armor plates
   - Tracking between frames using center distance
   - Number recognition for target selection

### Key Parameters

- **Light Bar Filtering**:
  - Aspect ratio: 1.5 to 10.0
  - Minimum area: 20 pixels
  - Maximum angle deviation: 30 degrees

- **Armor Plate Filtering**:
  - Maximum angle between light bars: 30 degrees
  - Distance ratio: 0.5 to 3.5
  - Height ratio: 0.7 to 1.3

## Angle Solving Module

### Implementation Details

The angle solving module is implemented in `AngleSolver.cpp` and follows these steps:

1. **Camera Calibration**:
   - Uses pre-calibrated camera intrinsic parameters
   - Accounts for lens distortion

2. **3D Position Estimation**:
   - Uses PnP (Perspective-n-Point) algorithm
   - Maps 2D image points to 3D world coordinates
   - Calculates target position in camera coordinate system

3. **Coordinate Transformation**:
   - Transforms camera coordinates to gimbal coordinates
   - Accounts for camera-to-gimbal offset

4. **Angle Calculation**:
   - Calculates pitch and yaw angles for targeting
   - Compensates for gravity based on distance and projectile speed

### Mathematical Model

The PnP algorithm solves the following equation:
```
s * [u, v, 1]^T = K * [R|t] * [X, Y, Z, 1]^T
```
Where:
- `[u, v]` are the image coordinates
- `K` is the camera intrinsic matrix
- `[R|t]` is the rotation and translation matrix
- `[X, Y, Z]` are the world coordinates

## Motion Prediction Module

### Implementation Details

The motion prediction module uses Kalman filtering to predict target movement:

1. **State Representation**:
   - State vector: [x, y, z, vx, vy, vz]
   - Position and velocity in 3D space

2. **Kalman Filter Implementation**:
   - Process model: Constant velocity model
   - Measurement model: Position only
   - Process noise: Accounts for acceleration
   - Measurement noise: Accounts for detection errors

3. **Prediction**:
   - Estimates current position and velocity
   - Predicts future position based on velocity
   - Calculates time-to-target based on projectile speed

### Kalman Filter Parameters

- **Process Noise Covariance**:
  - Position variance: 0.01
  - Velocity variance: 0.1

- **Measurement Noise Covariance**:
  - Position variance: 0.1

## Spinning Top Detection Module

### Implementation Details

The spinning top detection module is implemented in `ShootTuoluo.cpp`:

1. **Rotation Detection**:
   - Tracks armor plate position and angle over time
   - Analyzes angle changes to identify rotation
   - Determines rotation direction (left/right)

2. **Rotation Center Estimation**:
   - Uses least squares method to fit a circle to observed positions
   - Calculates rotation center and radius

3. **Rotation Speed Estimation**:
   - Calculates angular velocity from angle changes
   - Applies Kalman filtering for stable estimation

4. **Prediction**:
   - Predicts future position based on rotation model
   - Calculates lead angle for targeting

### Detection Criteria

- **Rotation Confidence**:
  - Minimum angle change: 5 degrees
  - Minimum position change: 3 pixels
  - Consistent direction over multiple frames

- **Shooting Decision**:
  - Rotation speed stability: < 10% variation
  - Minimum tracking frames: 10

## Energy Mechanism Detection Module

### Implementation Details

The energy mechanism (windmill) detection module is implemented in `FindBuff.cpp`:

1. **Fan Blade Detection**:
   - Binary image processing
   - Contour detection
   - Ellipse fitting for fan blades

2. **Target Selection**:
   - Contour hierarchy analysis
   - Feature-based filtering
   - Identification of active fan blade

3. **3D Position Estimation**:
   - Single-point distance measurement
   - Three-point fitting for rotation center
   - Normal vector calculation for windmill plane

4. **Rotation Analysis**:
   - Rotation speed estimation
   - Kalman filtering for stable prediction
   - Phase prediction for targeting

### Fan Blade Filtering

- **Shape Criteria**:
  - Aspect ratio: 1.5 to 3.0
  - Minimum area: 100 pixels
  - Maximum angle deviation: 45 degrees

- **Hierarchy Criteria**:
  - Parent-child relationship
  - Nesting level: 2

## Communication Module

### Implementation Details

The communication module is implemented in `RemoteController.cpp` and `serial.cpp`:

1. **Serial Communication**:
   - Baud rate: 115200
   - Data bits: 8
   - Stop bits: 1
   - Parity: None

2. **Protocol Implementation**:
   - Packet framing with header/footer bytes
   - Data encoding/decoding
   - CRC error checking

3. **Thread Synchronization**:
   - Mutex for shared resource access
   - Timestamp synchronization between camera and gimbal

### Error Handling

- **Packet Validation**:
  - Header/footer verification
  - CRC checksum
  - Timeout handling

- **Recovery Mechanisms**:
  - Packet resending
  - Connection reset
  - Error logging

## Image Processing Pipeline

### Thread Interaction

The image processing pipeline uses a producer-consumer pattern:

1. **Producer Thread**:
   - Captures frames from camera
   - Stores frames in a circular buffer
   - Records timestamp and gimbal data

2. **Consumer Thread**:
   - Processes the most recent frame
   - Detects targets and calculates angles
   - Sends targeting data to STM32

### Buffer Management

- **Circular Buffer**:
  - Size: 5 frames
  - Overflow handling: Skip older frames
  - Synchronization: Producer/consumer indices

## Performance Optimization

### Processing Speed

- **Frame Rate**:
  - Target: 60 FPS
  - Current: ~150 FPS (limited by camera)

- **Processing Time**:
  - Armor detection: ~5ms
  - Angle solving: ~2ms
  - Motion prediction: ~1ms
  - Total processing: ~10ms per frame

### Memory Management

- **Image Buffer**:
  - Pre-allocated memory for frames
  - Reuse of processing buffers
  - Minimal copying of image data

### Computational Efficiency

- **Algorithm Optimization**:
  - Early rejection of non-target regions
  - Simplified calculations for real-time performance
  - Lookup tables for trigonometric functions

## Calibration Procedures

### Camera Calibration

1. **Intrinsic Parameters**:
   - Use standard checkerboard pattern
   - Capture multiple views
   - Calculate focal length and distortion coefficients

2. **Extrinsic Parameters**:
   - Measure camera position relative to gimbal
   - Calculate transformation matrix

### System Calibration

1. **Color Thresholds**:
   - Capture samples under different lighting conditions
   - Adjust HSV thresholds for robust detection

2. **Gimbal Offset**:
   - Shoot at stationary targets at known distances
   - Adjust pitch, yaw, and roll offsets
   - Fine-tune coordinate system transformation

## Testing Methodology

### Performance Metrics

- **Detection Rate**:
  - Armor detection: ~90% at 4m
  - Spinning top detection: ~70% at 3m
  - Energy mechanism: ~80% hit rate

- **Accuracy**:
  - Static targets: < 1° error
  - Moving targets: < 3° error
  - Spinning targets: < 5° error

### Test Scenarios

1. **Static Testing**:
   - Fixed targets at various distances
   - Different lighting conditions
   - Various armor types (small/large)

2. **Dynamic Testing**:
   - Moving targets at different speeds
   - Spinning targets at different rates
   - Combined movement patterns

3. **System Integration Testing**:
   - End-to-end testing with actual robot
   - Field testing in competition-like environments
   - Stress testing with multiple targets

