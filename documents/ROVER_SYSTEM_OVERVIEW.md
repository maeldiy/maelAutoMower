# Autonomous Lawn Mower Rover - System Overview

---
## 🔗 Quick Navigation

[INDEX](INDEX.md) | [README](README.md) | [QUICK_START](QUICK_START.md) | [WIRING_GUIDE](WIRING_GUIDE.md) | [ARDUROVER_PARAMETERS](ARDUROVER_PARAMETERS.md) | [RTK_SETUP_GUIDE](RTK_SETUP_GUIDE.md) | [TROUBLESHOOTING_GUIDE](TROUBLESHOOTING_GUIDE.md)

---

## Project Summary
- **Type**: Wheeled skid-steer rover
- **Purpose**: Autonomous lawn mowing + manual RC control
- **Size**: ~1m × 0.5m
- **Weight**: 30 kg
- **Drivetrain**: 2 motor wheels (skid steering) + 1 free swivel caster
- **Status**: Basic RC control working, autonomous needs optimization

---

## Hardware Stack

### Power System
| Component | Specification |
|-----------|---------------|
| **Battery** | 36V (primary for wheel motors) |
| **Motors** | 2× DC motor wheels (skid steering) |
| **Mower Engine** | Gasoline (not managed by electronics) |

### Propulsion & Control
| Component | Model/Purpose |
|-----------|---------------|
| **Motor Wheels** | [Geared DC Motors](https://fr.aliexpress.com/item/1005004121086966.html) |
| **Motor Controller** | [Brushed DC Controller](https://fr.aliexpress.com/item/1005006305406654.html) |
| **Steering Type** | Differential/Skid steering (left/right wheel speed differential) |

### Flight Controller & Sensors
| Component | Model |
|-----------|-------|
| **Flight Controller** | Kakute H7 (from param file) |
| **GPS/Compass** | MATEKSYS M10Q-5883 (M10 with 5883 magnetometer) |
| **Telemetry/Comms** | 433MHz RF wireless relay receiver |

### Control Interfaces
| Method | Hardware |
|--------|----------|
| **Autonomous** | ArduRover + GNSS navigation |
| **Manual RC** | 433MHz RF wireless relay (2-channel) + PPM receiver |

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        MAIN POWER (36V)                          │
│                     Lithium/Lead-Acid Battery                    │
└───────┬───────────────────────────────────────────────────┬──────┘
        │                                                    │
        ├──────────────────────┬───────────────────────────┤
        │                      │                           │
   ┌─────────┐        ┌────────────────┐          ┌──────────────┐
   │  Motor1 │        │  Motor2        │          │ Gas Mower    │
   │ (Left)  │        │ (Right)        │          │ (Not elec.)  │
   └────┬────┘        └────────┬───────┘          └──────────────┘
        │                      │
        └──────────┬───────────┘
                   │
            ┌──────▼───────┐
            │ Motor Driver │ ◄─── PWM Control
            │  Controller  │      from Flight Controller
            └──────────────┘
                   
┌─────────────────────────────────────────────────────────────────┐
│                      FLIGHT CONTROLLER                           │
│                    Kakute H7 (ArduRover)                         │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐       │
│  │   GPS/Compass│  │  RC Receiver │  │  Motor Outputs  │       │
│  │ M10Q-5883    │  │  433MHz RF   │  │  (PWM Ch1, Ch2) │       │
│  └──────────────┘  └──────────────┘  └─────────────────┘       │
│                                                                  │
│  Navigation      Control            Propulsion Commands         │
│  (lat, lon, alt) (manual override)   (throttle, steering)       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Operating Modes

### 1. **Manual RC Mode**
- Operator controls rover via 433MHz transmitter
- RC signals override autonomous navigation
- Direct motor control via flight controller
- No GNSS required (but available)

### 2. **Autonomous Navigation Mode**
- Flight controller reads mission waypoints
- GNSS provides position/heading updates
- Motor speeds adjusted to follow planned path
- RC can interrupt/override at any time

### 3. **RTK-Enhanced Navigation** (Optional)
- Connects to local RTK base station (CentipedeRTK network)
- Sub-10cm positioning accuracy (vs 1-5m standard GNSS)
- **Base Station Details**: SMVD station near you
  - Coordinates: 5.182411°E, 43.729197°N (Southern France)
  - Platform: RTKBase + u-blox ZED-F9P
  - Data Format: RTCM3
  - Frequencies: L1-L2 bifréquence (GPS, GLONASS, Galileo, BeiDou)

---

## Communication Flow

```
Manual Control:
  [RC Transmitter] --433MHz--> [RF Receiver] --PPM--> [Flight Controller] --PWM--> [Motor Driver]

Autonomous:
  [GPS/Compass] --UART--> [Flight Controller] --PWM--> [Motor Driver]
  [Telemetry]   --UART--> [Ground Station] (monitoring)

RTK Enhancement:
  [RTK Base] --NTRIP/RTCM3--> [Flight Controller] --improves GPS--> [Navigation]
```

---

## Current Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| **Sketchy jump start** | PID tuning, acceleration limits | Tune ATC_ACCEL_MAX, ATC_DECEL_MAX |
| **Autonomous instability** | Steering command delay, noise | Apply rate limiting, increase filtering |
| **Inconsistent heading** | Compass calibration, local interference | Recalibrate compass, check mounting |

---

## Next Steps

1. ✅ Analyze existing hardware and parameters
2. ⚠️ Create detailed wiring diagram
3. ⚠️ Optimize ArduRover parameters
4. ⚠️ Configure RTK integration
5. ⚠️ Test and tune in field

---

## Key Parameters to Review

From your `20260803_kakuteH7_telem_RC_OK.param`:

```
Frame Class: 1 (Rover)
Frame Type: 0 (Skid)
Firmware: ArduRover

Motor Control:
- ATC_SPEED_P: 0.07 (Speed P gain)
- ATC_STR_RAT_P: 0.18 (Steering rate P gain)
- ATC_ACCEL_MAX: 0.3 (Acceleration limit - may be too low)

Compass:
- COMPASS_ORIENTATION: 2 (rotated)
- COMPASS_DIA_X/Y/Z: 1.138 / 0.996 / 0.990 (calibrated)

GNSS:
- EK3_SOURCE1_POSXY: 3 (GPS)
- GPS1_TYPE: 1 (u-blox)
```

These will be optimized in the detailed setup guide.
