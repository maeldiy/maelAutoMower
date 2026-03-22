# System Architecture Diagram

## Rover System Overview (Mermaid)

```mermaid
graph TB
    subgraph "Power System"
        BATT["36V Battery<br/>Lithium/Lead-Acid"]
        FUSE["30-60A Fuse"]
        PWR_MOD["Power Module<br/>36V → 5.3V"]
        BATT -->|36V| FUSE
        FUSE -->|36V| PWR_MOD
        FUSE -->|36V| MOTOR_CTRL
    end

    subgraph "Flight Controller & Sensors"
        FC["Kakute H7<br/>Flight Controller<br/>ArduRover Firmware"]
        IMU["IMU<br/>Accelerometer<br/>Gyroscope"]
        BARO["Barometer<br/>Altitude sensing"]
        GPS["MATEKSYS M10Q-5883<br/>u-blox ZED-F9P<br/>GPS/GNSS + Compass"]
        
        IMU -->|Data| FC
        BARO -->|Data| FC
        GPS -->|UART1<br/>Position/Heading| FC
    end

    subgraph "Motor Drive System"
        MOTOR_CTRL["Motor Controller<br/>H-Bridge/ESC"]
        MOTOR_L["Motor 1<br/>Left Wheel"]
        MOTOR_R["Motor 2<br/>Right Wheel"]
        WHEEL_L["Left Wheel<br/>w/ Gearbox"]
        WHEEL_R["Right Wheel<br/>w/ Gearbox"]
        
        MOTOR_CTRL -->|Phase A+B| MOTOR_L
        MOTOR_CTRL -->|Phase A+B| MOTOR_R
        MOTOR_L --> WHEEL_L
        MOTOR_R --> WHEEL_R
        
        FC -->|PWM Ch1<br/>Left Speed| MOTOR_CTRL
        FC -->|PWM Ch2<br/>Right Speed| MOTOR_CTRL
    end

    subgraph "Control Interfaces"
        RC_RX["433MHz RF Receiver<br/>2-Channel Relay"]
        RC_TX["RC Transmitter<br/>Manual Control"]
        
        RC_TX -->|433MHz Signal| RC_RX
        RC_RX -->|PPM Signal| FC
    end

    subgraph "Telemetry & Monitoring"
        TEL_RX["Telemetry Module<br/>433MHz or USB"]
        GCS["Ground Control Station<br/>Mission Planner"]
        
        FC -->|UART2<br/>Mavlink| TEL_RX
        TEL_RX -->|Data Link| GCS
        GCS -->|Commands| FC
    end

    subgraph "RTK Enhancement"
        RTK_BASE["RTK Base Station<br/>SMVD - CentipedeRTK<br/>u-blox ZED-F9P"]
        WIFI_MOD["WiFi/NTRIP Module<br/>ESP8266"]
        RTC3["RTCM3<br/>Corrections"]
        
        RTK_BASE -->|RTCM3| RTC3
        RTC3 -->|WiFi NTRIP| WIFI_MOD
        WIFI_MOD -->|UART3<br/>Corrections| FC
    end

    subgraph "Navigation & Autonomy"
        EKF["EKF3<br/>Sensor Fusion<br/>Extended Kalman Filter"]
        NAV["Navigation Engine<br/>Path Planning<br/>Waypoint Following"]
        
        FC -->|Sensor Data| EKF
        EKF -->|Fused Position<br/>Velocity, Heading| NAV
        NAV -->|Motor Commands| FC
    end

    PWR_MOD -->|5V Power| FC
    PWR_MOD -->|5V Power| GPS
    PWR_MOD -->|5V Power| RC_RX
    PWR_MOD -->|5V Power| WIFI_MOD
    
    style BATT fill:#ff9999
    style FC fill:#99ccff
    style GPS fill:#99ff99
    style RTK_BASE fill:#ffcc99
    style EKF fill:#ff99ff
```

## Operating Modes

```mermaid
graph LR
    MANUAL["Manual RC Mode<br/>Direct Motor Control"]
    STAB["Stabilize Mode<br/>Rate Limiting<br/>Failsafe Enabled"]
    LOITER["LOITER Mode<br/>Autonomous Hold Position<br/>GPS Required"]
    MISSION["MISSION Mode<br/>Waypoint Navigation<br/>Autonomous"]
    RTK_MODE["RTK Enhanced<br/>Centimeter Accuracy<br/>WiFi + NTRIP"]
    
    MANUAL --> STAB
    STAB --> LOITER
    LOITER --> MISSION
    MISSION --> RTK_MODE
    
    style MANUAL fill:#ffcccc
    style STAB fill:#ffffcc
    style LOITER fill:#ccffcc
    style MISSION fill:#ccffff
    style RTK_MODE fill:#ffccff
```

## Power Distribution Flow

```mermaid
graph TD
    BAT["36V Battery<br/>Primary Power Source"]
    
    BAT -->|Through Fuse| FUSE["30-60A Fuse<br/>Protection"]
    
    FUSE -->|36V| MC["Motor Controller<br/>Input Power"]
    FUSE -->|36V| PM["Power Module<br/>Voltage Regulator"]
    
    PM -->|5.3V| FC["Kakute H7"]
    PM -->|5.3V| GPS["GPS Module<br/>M10Q-5883"]
    PM -->|5.3V| RC["RC Receiver"]
    PM -->|5.3V| WIFI["WiFi Module<br/>ESP8266"]
    PM -->|5.3V| TEL["Telemetry<br/>Radio"]
    
    MC -->|36V Variable| M1["Motor 1<br/>Left"]
    MC -->|36V Variable| M2["Motor 2<br/>Right"]
    
    style BAT fill:#ff6666
    style FUSE fill:#ff9999
    style PM fill:#99ff99
    style MC fill:#ffcc99
```

## Communication Protocol Stack

```mermaid
graph LR
    AUTO["Autonomous<br/>Navigation Layer"]
    CTRL["Control Layer<br/>Rate & Attitude<br/>Control"]
    SERVO["Servo Output Layer<br/>PWM Signals"]
    HW["Hardware<br/>Motor Controller"]
    
    AUTO -->|Target Speed<br/>& Steering| CTRL
    CTRL -->|PWM Duty Cycle| SERVO
    SERVO -->|3.3V PWM<br/>CH1, CH2| HW
    HW -->|Variable 36V| MOTORS["Motors"]
    
    style AUTO fill:#ccffcc
    style CTRL fill:#ccffff
    style SERVO fill:#ffccff
    style HW fill:#ffcccc
```

---

## Hardware Connections Summary

| Component | Connection | Signal | Voltage |
|-----------|-----------|--------|---------|
| **Battery** | Power Module IN | 36V DC | 36V |
| **Motor 1 (Left)** | Motor Controller OUT | Phase A+B | 36V variable |
| **Motor 2 (Right)** | Motor Controller OUT | Phase A+B | 36V variable |
| **GPS/Compass** | FC UART1 | TX/RX | 3.3V |
| **RC Receiver** | FC RC IN | PPM | 3.3V |
| **WiFi Module** | FC UART3 | TX/RX | 3.3V |
| **Telemetry** | FC UART2 | TX/RX | 3.3V |
| **Motor Controller** | FC CH1, CH2 | PWM | 3.3V |
| **Power Module** | FC 5V Pin | 5V regulated | 5.3V |

---

## Signal Flow During Autonomous Mission

```
1. GPS Satellite Network
   ↓
2. M10Q-5883 Module (receives raw GNSS signals)
   ↓
3. Flight Controller (UART1 raw position data)
   ├─ EKF3 Sensor Fusion
   │  ├─ IMU (accelerometer, gyroscope)
   │  ├─ Barometer
   │  ├─ Compass (magnetometer)
   │  └─ GPS (position, velocity)
   └─ Fused State Estimate
   ↓
4. Navigation Engine
   ├─ Reads current position
   ├─ Calculates error from waypoint
   └─ Generates motor commands
   ↓
5. Control Loop (Attitude Controller)
   ├─ Speed controller (throttle)
   ├─ Steering controller (left/right differential)
   └─ Rate limiting (smooth acceleration)
   ↓
6. PWM Output (CH1, CH2)
   ↓
7. Motor Controller
   ├─ Interprets PWM duty cycle
   ├─ Generates phase signals
   └─ Drives motors with variable power
   ↓
8. Motors + Wheels
   └─ Rover moves toward waypoint
```

---

## RTK Signal Flow

```
1. RTK Base Station (SMVD - CentipedeRTK)
   ├─ Fixes position precisely
   └─ Calculates correction messages
   ↓
2. RTCM3 Stream (correction data)
   ↓
3. WiFi Module on Rover (ESP8266)
   ├─ Requests corrections via NTRIP
   └─ Forwards RTCM3 to flight controller
   ↓
4. Flight Controller (UART3)
   ├─ Receives RTCM3 messages
   └─ Applies corrections to GPS solution
   ↓
5. u-blox ZED-F9P Module
   ├─ Blends raw GPS + RTK corrections
   └─ Achieves 2-5cm accuracy (vs 1-2m standard GPS)
   ↓
6. EKF3 Fusion
   ├─ Trusts corrected position more (tighter gates)
   └─ Smoother, more accurate navigation
   ↓
7. Autonomous Mission
   └─ Follows within ±5cm of planned path (100x improvement!)
```

---

## Typical Mission Sequence

```
START: Battery connected, outdoors with clear sky

↓ (0 seconds)
Power-up: FC boots, all sensors initialize
↓
GPS Acquiring: Satellites appearing (3-6 found)
↓
IMU Calibration: Accelerometer/gyroscope baseline
↓ (30 seconds)
GPS Lock: Good position fix (gps_status = 3)
↓
Compass Calibrated: Ready for heading control
↓ (60 seconds)
Arm: Switch to MISSION mode, pre-arm checks pass
↓
Load Mission: Waypoints uploaded and verified
↓ (120 seconds)
Start Motor: First waypoint engagement
↓
Navigate: EKF fused sensors guide rover
↓
Heading Control: Compass maintains direction
↓
Speed Control: Smooth acceleration ramping
↓
Obstacle Avoidance: (if enabled) Detects and avoids
↓
Waypoint Reached: Circle tightens, moves to next
↓
[Repeat for all waypoints]
↓
Return to Home: Final waypoint to launch location
↓
Land: Throttle to zero, kill motors
↓
Complete: Mission finished, ready for next

TOTAL MISSION TIME: Variable (waypoint spacing dependent)
```

---

## See Also

- [ROVER_SYSTEM_OVERVIEW.md](ROVER_SYSTEM_OVERVIEW.md) - System architecture details
- [WIRING_GUIDE.md](../documents/WIRING_GUIDE.md) - Detailed wiring instructions
- [ARDUROVER_PARAMETERS.md](../documents/ARDUROVER_PARAMETERS.md) - Parameter tuning
