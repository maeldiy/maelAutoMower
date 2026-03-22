# ArduRover Parameter Setup Guide

---
## 🔗 Quick Navigation

[INDEX](INDEX.md) | [README](README.md) | [QUICK_START](QUICK_START.md) | [ROVER_SYSTEM_OVERVIEW](ROVER_SYSTEM_OVERVIEW.md) | [WIRING_GUIDE](WIRING_GUIDE.md) | [RTK_SETUP_GUIDE](RTK_SETUP_GUIDE.md) | [TROUBLESHOOTING_GUIDE](TROUBLESHOOTING_GUIDE.md)

---

## Overview

This guide provides optimized ArduRover parameters for your skid-steer rover with Kakute H7 flight controller. Parameters address the "sketchy jump start" issue by tuning acceleration, deceleration, and steering response.

---

## Critical Parameters Explained

### 1. **Frame Configuration**

```
FRAME_CLASS = 1          # 1 = Rover
FRAME_TYPE = 0           # 0 = Skid steer (tank treads / differential wheels)
```

**Why**: Skid steering uses differential motor speed to turn (left slower = right turn).

---

### 2. **Speed Controller Tuning** (Fixes Jerky Movements)

| Parameter | Current | Recommended | Reason |
|-----------|---------|------------|--------|
| **ATC_ACCEL_MAX** | 0.3 m/s² | **1.5 m/s²** | Current value is too conservative; causes hesitation on startup |
| **ATC_DECEL_MAX** | 0 | **2.0 m/s²** | Unlimited deceleration can cause jerky braking |
| **ATC_SPEED_P** | 0.07 | **0.15** | Increase speed response proportional gain |
| **ATC_SPEED_I** | 0.05 | **0.08** | Increase integral to reduce steady-state error |
| **ATC_SPEED_IMAX** | 1 | **2** | Allow more integral term accumulation |

**Impact**: Smoother acceleration ramp, reduces "jump start" jerkiness.

---

### 3. **Steering Rate Control** (Fixes Oscillation)

| Parameter | Current | Recommended | Reason |
|-----------|---------|------------|--------|
| **ATC_STR_RAT_P** | 0.18 | **0.25** | More responsive steering without overshoot |
| **ATC_STR_RAT_I** | 0.156 | **0.20** | Better steering error correction |
| **ATC_STR_RAT_D** | 0 | **0.003** | Small damping to reduce oscillation |
| **ATC_STR_RAT_MAX** | 120 °/s | **90 °/s** | Reduce max steering rate for stability |

**Impact**: Smoother turning, less wandering off course.

---

### 4. **Angle Controller** (Direction Holding)

| Parameter | Current | Recommended | Purpose |
|-----------|---------|------------|---------|
| **ATC_STR_ANG_P** | 2 | **3.0** | Proportional gain for angle correction |
| **ATC_TURN_MAX_G** | 0.3 G | **0.5 G** | Max lateral acceleration during turns |

**Impact**: Better heading maintenance during autonomous missions.

---

## Complete Optimized Parameter File

Create a new file: `ROVER_OPTIMIZED.param` with these values:

```
# ============================================================
# ARDUROVER OPTIMIZED PARAMETERS - Kakute H7 Skid Steer Rover
# ============================================================
# 
# Optimized for:
# - Smooth acceleration (fixes jump start)
# - Stable autonomous navigation
# - Manual RC control responsiveness
# - RTK-ready GPS integration
#
# Date: 2026-03-20
# ============================================================

# === FRAME CONFIGURATION ===
FRAME_CLASS=1                    # Rover
FRAME_TYPE=0                     # Skid steer

# === SPEED CONTROLLER (Attack Jump Start) ===
ATC_ACCEL_MAX=1.5               # Smooth acceleration ramp (was 0.3)
ATC_DECEL_MAX=2.0               # Smooth deceleration 
ATC_SPEED_P=0.15                # Speed proportional gain (was 0.07)
ATC_SPEED_I=0.08                # Speed integral gain (was 0.05)
ATC_SPEED_IMAX=2                # Speed integrator limit (was 1)
ATC_SPEED_D=0.002               # Speed damping
ATC_SPEED_FF=0                  # No feedforward offset

# === STEERING RATE CONTROLLER ===
ATC_STR_RAT_P=0.25              # Steering rate P gain (was 0.18)
ATC_STR_RAT_I=0.20              # Steering rate I gain (was 0.156)
ATC_STR_RAT_D=0.003             # Steering rate D gain (was 0)
ATC_STR_RAT_IMAX=1              # Steering integrator limit
ATC_STR_RAT_MAX=90              # Max steering rate (was 120, reduce for stability)
ATC_STR_RAT_FLTD=0              # No D filter
ATC_STR_RAT_FLTE=10             # Filter cutoff

# === STEERING ANGLE CONTROLLER ===
ATC_STR_ANG_P=3.0               # Angle proportional gain (was 2)
ATC_STR_ACC_MAX=120             # Max steering acceleration

# === TURN BEHAVIOR ===
ATC_TURN_MAX_G=0.5              # Max lateral G during turns (was 0.3)

# === CRUISE SETTINGS ===
CRUISE_SPEED=2                  # Default cruise speed (m/s)
CRUISE_THROTTLE=30              # Default throttle (0-100)

# === FAILSAFE SETTINGS ===
FS_ACTION=2                      # Brake on failsafe
FS_THR_ENABLE=1                 # Enable throttle failsafe
FS_THR_VALUE=910                # Failsafe servo position
FS_TIMEOUT=1.5                  # Failsafe timeout (seconds)

# === BRAKING ===
ATC_BRAKE=1                     # Enable braking

# === AUTONOMOUS AVOIDANCE ===
AVOID_ENABLE=3                  # Proximity detection enabled
AVOID_ACCEL_MAX=3               # Avoidance acceleration limit
AVOID_MARGIN=2                  # Obstacle margin (meters)

# ============================================================
# === GPS / NAVIGATION ===
# ============================================================

GPS_AUTO_CONFIG=1               # Auto-configure u-blox GPS
GPS_AUTO_SWITCH=1               # Auto-switch between GPS sources
GPS_NAVFILTER=8                 # Kalman filter
GPS_PRIMARY=0                   # Primary GPS is first one
GPS_MIN_ELEV=-100               # Minimum elevation angle

GPS1_TYPE=1                      # u-blox (ZED-F9P)
GPS1_RATE_MS=200                # 5 Hz update rate
GPS1_DELAY_MS=0                 # No delay

# === COMPASS / MAGNETOMETER ===
COMPASS_AUTO_ROT=2              # Auto-rotation compensation
COMPASS_AUTODEC=1               # Auto-declination
COMPASS_ENABLE=1                # Enable compass
COMPASS_EXTERNAL=1              # External compass (M10Q-5883)
COMPASS_USE=1                   # Use for heading
COMPASS_ORIENT=2                # Rotation (check actual orientation)
COMPASS_SCALE=1.113329          # Calibration scale (from your params)

# Compass calibration (from your previous calibration)
COMPASS_OFS_X=27.87471
COMPASS_OFS_Y=80.75342
COMPASS_OFS_Z=-181.665
COMPASS_DIA_X=1.138066
COMPASS_DIA_Y=0.9963335
COMPASS_DIA_Z=0.9904658

# === ATTITUDE ESTIMATION (EKF3) ===
AHRS_EKF_TYPE=3                 # Extended Kalman Filter version 3
AHRS_GPS_USE=1                  # Use GPS for attitude
AHRS_GPS_GAIN=1                 # GPS measurement weighting
AHRS_GPS_MINSATS=6              # Minimum satellites for GPS
AHRS_COMP_BETA=0.1              # Yaw complementary filter beta

AHRS_TRIM_X=0.007379097         # IMU trim (keep from calibration)
AHRS_TRIM_Y=0.03800556

# ============================================================
# === EXTENDED KALMAN FILTER 3 (for RTK) ===
# ============================================================

EK3_ENABLE=1                    # Enable EKF3
EK3_PRIMARY=0                   # Use as primary estimator

# Source configuration (GPS as primary)
EK3_SRC1_POSXY=3                # GPS for horizontal position
EK3_SRC1_POSZ=1                 # Baro for vertical (or 3 for GPS, need testing)
EK3_SRC1_VELXY=3                # GPS for horizontal velocity
EK3_SRC1_VELZ=3                 # GPS for vertical velocity
EK3_SRC1_YAW=1                  # Compass for heading

# Filter tuning
EK3_HGT_DELAY=60                # GPS delay compensation (ms)
EK3_POS_I_GATE=500              # Position gate
EK3_VEL_I_GATE=500              # Velocity gate

# Measurement noise
EK3_POSNE_M_NSE=0.5             # GPS position noise
EK3_ACC_P_NSE=0.35              # Accelerometer noise
EK3_GYRO_P_NSE=0.015            # Gyro noise

# ============================================================
# === RTK CONFIGURATION (CRITICAL FOR YOUR USE CASE) ===
# ============================================================

# Note: RTK parameters are set via u-blox configuration
# These EKF settings prepare for RTK correction input

EK3_SRC_OPTIONS=1               # Blending of sources enabled
EK3_GPS_CHECK=31                # All GPS checks enabled

# RTK typically provides cm-level accuracy when:
# - Connected to local RTK base station
# - u-blox GPS configured with RTCM3 input
# - EKF is blending corrections

# ============================================================
# === ARMING / SAFETY ===
# ============================================================

ARMING_CHECK=1                  # Perform pre-arm checks
ARMING_REQUIRE=1                # Require arm switch
ARMING_RUDDER=2                 # Can arm with RC input

ARMING_ACCTHRESH=0.75           # Accel threshold for arm check
ARMING_MAGTHRESH=100            # Compass consistency threshold

# ============================================================
# === PHYSICAL SENSOR CALIBRATION ===
# ============================================================

# Accelerometer
INS_ACC_ID=3408154              # IMU sensor ID (keep as-is)
INS_ACC_BODYFIX=2               # Automatic body frame

# Barometer
BARO_PRIMARY=0                  # Primary baro
BARO1_DEVID=1472001             # Sensor ID

BATT_MONITOR=4                  # Battery monitoring type
BATT_VOLT_PIN=8                 # Voltage input pin
BATT_CURR_PIN=4                 # Current input pin
BATT_VOLT_MULT=18.18            # Voltage multiplier
BATT_AMP_PERVLT=36.6            # Current multiplier
BATT_CAPACITY=3300              # Battery Ah capacity

# ============================================================
# === TELEMETRY ===
# ============================================================

SERIAL1_PROTOCOL=9              # GPS protocol on UART1
SERIAL2_PROTOCOL=1              # Mavlink telemetry on UART2
SERIAL2_BAUD=57600              # Telemetry baud rate

# ============================================================
# === LOGGING ===
# ============================================================

LOG_BITMASK=176127              # Log most data for debugging
LOG_DISARMED=0                  # Don't log disarmed

# ============================================================
# === RESERVED / OPTIONAL ===
# ============================================================

# Sailboat mode (disabled for lawn mower)
ATC_SAIL_P=1
ATC_SAIL_I=0.1

# Balancing rover (disabled)
ATC_BAL_P=1.8

# ============================================================
# End of parameters
# ============================================================
```

---

## Installation Steps

### **Method 1: Using Mission Planner (Recommended)**

1. **Download Mission Planner**
   - Get from: [ardupilot.org](https://ardupilot.org/planner/)

2. **Connect Flight Controller**
   - Connect Kakute H7 via USB to computer
   - Select COM port in Mission Planner

3. **Load Firmware**
   - Go: `Initial Setup` → `Install Firmware`
   - Select: `ArduRover`
   - Wait for completion (2-3 minutes)

4. **Load Parameters**
   - Go: `Initial Setup` → `Load from File`
   - Select the optimized `.param` file above
   - Click `Load` → `Write Params`
   - Wait for message: "Parameters written successfully"

5. **Verify Installation**
   - Go: `Initial Setup` → `Mandatory Hardware`
   - Check: Compass, Accelerometer, Radio button green
   - All should show "Enable"

---

### **Method 2: Manual Parameter Entry**

If you don't have the `.param` file:

1. Go: `Initial Setup` → `Parameter List`
2. Search for parameter name (e.g., "ATC_ACCEL_MAX")
3. Click on value, change it
4. Press `Enter` to confirm
5. Repeat for all parameters

---

## Post-Installation Checklist

- [ ] Firmware loaded (ArduRover correct version)
- [ ] All parameters written
- [ ] Compass calibrated (in Mandatory Hardware)
- [ ] Accelerometer calibrated
- [ ] Radio configured (RC sticks mapped to Throttle/Steering)
- [ ] GPS lock achieved (Status panel shows fix)
- [ ] Failsafe set (throttle failsafe at 910)

---

## Fine-Tuning After First Drive

### **If rover is too jerky:**
```
Reduce: ATC_ACCEL_MAX (try 0.8)
Reduce: ATC_STR_RAT_P (try 0.15)
```

### **If rover overshoots turns:**
```
Increase: ATC_STR_RAT_D (try 0.01)
Reduce: ATC_STR_RAT_P (try 0.18)
```

### **If rover won't hold straight line:**
```
Increase: ATC_STR_RAT_I (try 0.3)
Increase: ATC_STR_ANG_P (try 4.0)
```

### **If response is sluggish:**
```
Increase: ATC_SPEED_P (try 0.25)
Increase: ATC_STR_RAT_P (try 0.4)
```

---

## RTK-Specific Parameters

These parameters prepare your rover for RTK corrections via CentipedeRTK:

```
# RTK Enhancement (when connected to base station):
EK3_SRC1_POSXY=3              # GPS for position (RTK provides better signal)
EK3_HGT_DELAY=60              # Account for RTK correction latency
EK3_VEL_I_GATE=500            # Tight gate for RTK velocity updates
EK3_POS_I_GATE=500            # Tight gate for RTK position updates
```

**Why these matter**:
- RTK provides 2-5 cm accuracy instead of 1-2 meters
- Tight gates mean EKF trusts corrections more
- Delay compensation accounts for latency in correction stream

See: `RTK_SETUP_GUIDE.md` for full RTK configuration.

---

## Next Steps

1. ✅ Load these parameters
2. ⚠️ Calibrate compass in field
3. ⚠️ Test in manual RC mode first
4. ⚠️ Test in LOITER mode (autonomous)
5. ⚠️ Enable RTK corrections
6. ⚠️ Create missions and test autonomous navigation

---

## Backup Your Current Parameters

Before loading new parameters, save your working ones:

```
Initial Setup → Backup/Restore → Backup
```

This way, if something goes wrong, you can restore and troubleshoot.

---

## Reference: Parameter Categories

All parameters explained by category:

- **ATC_*** = Attitude Control (steering, acceleration)
- **EK3_*** = Extended Kalman Filter (sensor fusion)
- **COMPASS_*** = Magnetometer calibration
- **GPS_*** = GNSS receiver configuration
- **AHRS_*** = Attitude & Heading Reference System
- **BATT_*** = Battery monitoring
- **FS_*** = Failsafe settings
- **SERIAL_*** = Communication port setup
- **LOG_*** = Data logging configuration

---

## Need Help?

- **ArduRover Docs**: https://ardupilot.org/rover/
- **Parameter Documentation**: https://ardupilot.org/rover/parameters.html
- **Community**: ArduPilot Discuss Forum

