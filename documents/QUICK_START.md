# Quick Start Checklist - Autonomous Lawn Mower Rover

---
## 🔗 Quick Navigation

[INDEX](INDEX.md) | [README](README.md) | [ROVER_SYSTEM_OVERVIEW](ROVER_SYSTEM_OVERVIEW.md) | [WIRING_GUIDE](WIRING_GUIDE.md) | [ARDUROVER_PARAMETERS](ARDUROVER_PARAMETERS.md) | [RTK_SETUP_GUIDE](RTK_SETUP_GUIDE.md) | [TROUBLESHOOTING_GUIDE](TROUBLESHOOTING_GUIDE.md)

---

## Pre-Assembly Verification

### **Hardware Check** (Before wiring)

- [ ] Kakute H7 Flight Controller: Inspect for physical damage, bent pins
- [ ] MATEKSYS M10Q-5883 GPS/Compass: Check connector pins not bent
- [ ] Motor Controller: Verify all pins present, no burnt components
- [ ] 36V Battery: Test voltage with multimeter (should be close to 36V)
- [ ] Motors (2×): Spin freely by hand, no grinding sounds
- [ ] Wheels: Roll freely, check tire pressure if pneumatic
- [ ] Power Module: Test 5.3V output with multimeter on test pins

---

## Wiring Assembly (Reference: WIRING_GUIDE.md)

### **Step 1: Power Distribution**
- [ ] Install 30-60A fuse on 36V battery positive
- [ ] Connect fused battery (+) to Motor Controller input
- [ ] Connect battery (-) to Motor Controller and Power Module ground
- [ ] Wire Power Module: 36V input → 5.3V output
- [ ] Verify Power Module is outputting 5.3V (CRITICAL!)

### **Step 2: Flight Controller Power**
- [ ] Connect Power Module 5V output to Kakute H7 5V pin
- [ ] Connect Power Module GND to Kakute H7 GND (multiple pins)
- [ ] Use short, thick wires (16 AWG minimum)
- [ ] Add strain relief to all connectors

### **Step 3: Motor Connections**
- [ ] Motor Controller PWM signal from Kakute CH1 (left motor)
- [ ] Motor Controller PWM signal from Kakute CH2 (right motor)
- [ ] Motor Controller GND connected to FC GND
- [ ] Motor 1 (Left): Connected to motor controller output A
- [ ] Motor 2 (Right): Connected to motor controller output B
- [ ] Test: Manually spin each motor (should be free)

### **Step 4: GPS/Compass** (MATEKSYS M10Q-5883)
- [ ] GPS VCC (5V) → Power Module 5V output
- [ ] GPS GND → Common ground (FC GND)
- [ ] GPS TX → Kakute H7 UART1 RX (RC IN pin area)
- [ ] GPS RX → Kakute H7 UART1 TX
- [ ] Mount antenna vertically on top of rover
- [ ] Keep > 10 cm away from compass

### **Step 5: RC Receiver** (433MHz RF)
- [ ] Receiver power: 5V from FC (if powered by receiver)
- [ ] Receiver PPM output → RC IN on Kakute H7 (usually pin labeled "RC IN")
- [ ] Receiver GND → Common ground
- [ ] Receiver should be away from motors/antennas

### **Step 6: Protection & Insulation**
- [ ] All exposed connectors: Heat shrink tubing
- [ ] High-current wires: Separate bundles from logic wires
- [ ] Flight controller: Protected in enclosure or foam
- [ ] Antenna: Protected from physical damage

---

## Initial Power-Up Test

### **BEFORE Connecting Battery**

1. [ ] Visual inspection: No loose wires, connectors seated
2. [ ] No smell test: No burning or chemical smells
3. [ ] Continuity check: No short circuits (use multimeter on battery terminals)

### **FIRST Power-Up** 

```
CRITICAL SAFETY: Remove all props/wheels before this!
```

1. [ ] Connect battery slowly (via fused connector or switch)
2. [ ] Kakute H7 LED should light up (green steady)
3. [ ] Power Module should output 5.3V (verify with multimeter)
4. [ ] NO beeping, NO smoke, NO burning smell
5. [ ] Wait 5 seconds, then disconnect
6. [ ] **If anything is wrong or smells bad: STOP immediately**

### **Second Power-Up (with telemetry connected)**

1. [ ] Connect via USB to laptop running Mission Planner
2. [ ] Select COM port (Kakute H7 should enumerate as USB device)
3. [ ] Go: Setup → Install Firmware
4. [ ] Select "ArduRover" → Let it download and install (2-3 minutes)
5. [ ] Firmware installation successful message → Proceed

---

## Firmware & Parameter Configuration

### **Load ArduRover Firmware**

1. [ ] Mission Planner: Initial Setup → Install Firmware
2. [ ] Select "ArduRover" (correct board: Kakute H7)
3. [ ] Allow 3-5 minutes for flashing
4. [ ] Reboot flight controller via USB disconnect/reconnect

### **Load Optimized Parameters**

1. [ ] Download: `ROVER_OPTIMIZED.param` (from ARDUROVER_PARAMETERS.md)
2. [ ] Mission Planner: Initial Setup → Load from File
3. [ ] Select the `.param` file
4. [ ] Click "Load" → Wait for green success message
5. [ ] Click "Write Params" (all parameters will be written)
6. [ ] Reboot flight controller

### **Verify Parameters Loaded**

1. [ ] Mission Planner: Initial Setup → Parameter List
2. [ ] Search for "ATC_ACCEL_MAX" → Should be **1.5**
3. [ ] Search for "FRAME_CLASS" → Should be **1**
4. [ ] Search for "FRAME_TYPE" → Should be **0** (skid)

---

## Hardware Calibration (Must Do Before Flying)

### **Accelerometer Calibration**

1. [ ] Mission Planner: Initial Setup → Accelerometer Calibration
2. [ ] Click "Calibrate Accel"
3. [ ] Place rover on level surface (table/floor)
4. [ ] When prompted, press continue → Wait 5 seconds
5. [ ] Rover should be absolutely still during this
6. [ ] Success message should appear

### **Compass Calibration**

1. [ ] Move rover to **completely clear outdoor area** (away from buildings, power lines, metal)
2. [ ] Mission Planner: Initial Setup → Compass → Calibration (Onboard Mag Calibration)
3. [ ] Click "Start"
4. [ ] **Slowly rotate the rover** through complete 360° **AT LEAST 3 TIMES**
5. [ ] Rotate in pitch, roll, and yaw (all axes)
6. [ ] Keep rotating smoothly for 30-60 seconds
7. [ ] Success message + sphere diagram should appear
8. [ ] If sphere is "lumpy": Move further from interference, try again
9. [ ] Document scale: `COMPASS_SCALE` should be ~1.1

### **Radio Calibration**

1. [ ] Plug in battery (but rover is DISARMED)
2. [ ] Mission Planner: Initial Setup → Radio Calibration
3. [ ] Turn on RC transmitter
4. [ ] **For each stick & switch**, move it through FULL range:
   - Throttle: Down (1000 μs) → Up (2000 μs)
   - Steering: Left (1000 μs) → Right (2000 μs)
   - Mode switch: Through all positions
5. [ ] All channels should show green values 1000-2000
6. [ ] Click "Calibrate" when complete
7. [ ] SUCCESS message and green lights on all channels

---

## Pre-Flight Checklist

### **Before First Autonomous Test**

**Hardware**:
- [ ] All wires secure, no loose connectors
- [ ] Battery fully charged (>95%)
- [ ] Motors spin freely by hand
- [ ] GPS antenna has clear sky view
- [ ] Compass is far from interference (check with compass app)

**Software**:
- [ ] Firmware: ArduRover (correct version)
- [ ] Parameters: All loaded and verified
- [ ] Accelerometer: Calibrated
- [ ] Compass: Calibrated (green arrow pointing North in HUD)
- [ ] Radio: All channels responding (green bars in Mission Planner)
- [ ] GPS: Lock acquired (watch Status → messages for "GPS OK")

**Field Setup**:
- [ ] Clear test area: 50m radius minimum
- [ ] No obstacles, hazards, or people nearby
- [ ] Clear line-of-sight to rover (for RC recovery)
- [ ] Transmitter range checked (can control at distance)

---

## First Test Sequence

### **Test 1: Manual RC Mode** (Verify basic control)

1. [ ] Arm rover: Switch mode to STABILIZE, ensure all safety checks pass
2. [ ] Move throttle stick forward → Left & right motors should spin equally (go straight)
3. [ ] Move throttle backward → Motors reverse, rover backs up
4. [ ] Move steering left/right → Motors should respond (differential speed)
5. [ ] **If reversed**: Set reverse flag for that motor in parameters
6. [ ] Disarm rover: Switch mode back to MANUAL

**Expected behavior**: Smooth, responsive control with no jerking

### **Test 2: GPS Lock** (Verify navigation capability)

1. [ ] Power on rover outdoors with clear sky view
2. [ ] Mission Planner: Data → Status
3. [ ] Watch "gps_numsat" (number of satellites)
   - Should increase: 0 → 3 → 6 → 12+
4. [ ] Watch "gps_status"
   - 0 = No signal
   - 1 = GPS searching
   - 2 = Coarse fix
   - **3 = Good fix** ✅ (can proceed)
5. [ ] **If no lock after 2 minutes**:
   - Check antenna direction (should be up)
   - Move to different location (reduce radio interference)
   - Check UART1 connection to GPS

### **Test 3: LOITER Mode** (Autonomous hold position)

1. [ ] Ensure GPS lock (status = 3)
2. [ ] Switch to LOITER mode (via transmitter)
3. [ ] **Don't move transmitter sticks**
4. [ ] Rover should stay in place (with ±1-2m accuracy)
5. [ ] Move rover manually 5 meters → Release
6. [ ] Rover should navigate back to original position
7. [ ] Observe behavior:
   - Smooth return? → Good tuning ✓
   - Overshoots waypoint? → May need PID tuning
   - Wobbles a lot? → Compass interference (recalibrate)

### **Test 4: Simple Mission** (First autonomous waypoint mission)

1. [ ] GPS lock confirmed (status = 3)
2. [ ] Mission Planner: Fly → Create simple 2-waypoint mission
   - Waypoint 1: Rover's current position
   - Waypoint 2: 10 meters forward
   - Waypoint 3: Return to start
3. [ ] Upload mission to rover
4. [ ] Arm rover manually (switch to MISSION mode)
5. [ ] **Stay ready with transmitter to abort if needed**
6. [ ] Rover should autonomously navigate to waypoints
7. [ ] Observe rover:
   - Follows straightish path? → Good ✓
   - Overshoots significantly? → Reduce CRUISE_SPEED (1.0 m/s)
   - Drifts off path by >2m? → Check compass calibration

---

## Performance Tuning (After First Successful Test)

### **If Rover is Too Jerky on Startup**

```
Parameter                          Current       Try
─────────────────────────────────────────────────────
ATC_ACCEL_MAX                      1.5 m/s²      1.0 or 0.8
ATC_SPEED_P                        0.15          0.10
```

Then test with small throttle inputs.

### **If Rover Overshoots Waypoints**

```
Parameter                          Current       Try
─────────────────────────────────────────────────────
CRUISE_SPEED                       2.0 m/s       1.5 m/s
ATC_STR_RAT_P                      0.25          0.18
WP_RADIUS                          1.0 m         1.5 m
```

### **If Rover Oscillates/Wanders**

```
Parameter                          Current       Try
─────────────────────────────────────────────────────
ATC_STR_RAT_D                      0.003         0.005
ATC_STR_ANG_P                      3.0           2.5
```

---

## RTK Setup (Optional but Highly Recommended!)

See: **RTK_SETUP_GUIDE.md** for detailed setup

**Quick start**:
1. [ ] Get ESP8266 WiFi module (~€10)
2. [ ] Wire to Kakute H7 UART3
3. [ ] Load ArduPilot NTRIP client firmware on ESP8266
4. [ ] Configure with CentipedeRTK credentials
5. [ ] In Mission Planner: Set GPS2_TYPE = 1, GPS2_COM_PORT = 3
6. [ ] Power on rover, wait 30 seconds
7. [ ] Check Status: gps2_status should change 0 → 4 → **5** (RTK Fixed!)
8. [ ] With RTK: Accuracy improves 1-2m → **2-5 cm** (100x better!)

---

## Troubleshooting Quick Reference

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| **No lights on FC** | No power | Check fuse, power module output |
| **Motors won't spin** | No PWM signal | Check UART connections, reload firmware |
| **Rover goes in circles** | Motor speed mismatch | Adjust motor trim or swap motor connections |
| **Bad GPS lock** | Antenna blocked or indoors | Move outdoors, clear line-of-sight to sky |
| **Rover overshoots waypoints** | Tuning too aggressive | Reduce CRUISE_SPEED or increase WP_RADIUS |
| **Compass heading erratic** | Interference | Recalibrate compass away from interference |

See **TROUBLESHOOTING_GUIDE.md** for full diagnostic procedures.

---

## Emergency Abort Procedure

**If rover is out of control or dangerous**:

1. **Immediate action**: Switch transmitter mode away from autonomous
2. **If that doesn't work**: **Turn off transmitter** (triggers failsafe)
3. **If motors still spinning**: **Disconnect battery negative terminal**
4. **Never**: Chase a moving rover or put hands in danger

---

## Important Files Reference

| Document | Purpose |
|----------|---------|
| `ROVER_SYSTEM_OVERVIEW.md` | High-level architecture and component list |
| `WIRING_GUIDE.md` | Detailed wiring instructions with diagrams |
| `ARDUROVER_PARAMETERS.md` | Parameter explanations and loading instructions |
| `RTK_SETUP_GUIDE.md` | How to set up RTK for cm-level accuracy |
| `TROUBLESHOOTING_GUIDE.md` | Diagnostic procedures for all issues |
| `SYSTEM_ARCHITECTURE.drawio` | Visual diagram (open in diagrams.net) |

---

## Next Steps (After successful first mission)

1. **Optimize parameters** based on observed behavior
2. **Create real-world missions** for lawn mowing use case
3. **Implement RTK** for centimeter-level accuracy
4. **Plan sensor upgrades** (blade control, obstacle detection)
5. **Document any modifications** for future maintenance

---

## Support Resources

- **ArduPilot Docs**: https://ardupilot.org/rover/
- **CentipedeRTK Network**: https://centipede.fr
- **Community Forum**: https://discuss.ardupilot.org/
- **u-blox Documentation**: https://www.u-blox.com/
- **Local Robotics Groups** (check Facebook, meetup.com)

---

## Safety Reminders

🚨 **CRITICAL SAFETY RULES**

1. **Always remove props/wheels when testing indoors**
2. **Never put hands/fingers near moving motors**
3. **Battery disconnection is final failsafe** (keep battery accessible)
4. **Transmitter control ready at all times** (don't trust autonomy yet)
5. **Test in empty field first** (away from people, animals, obstacles)
6. **Keep emergency procedures memorized**

---

## Success Indicators

When you've successfully completed this setup:

✅ Kakute H7 boots reliably
✅ All motors respond to RC control
✅ GPS lock achieved in < 2 minutes
✅ Compass calibrated (green arrow in HUD)
✅ LOITER mode holds position ±2m
✅ Simple 2-waypoint mission completes successfully
✅ Rover follows planned path within 1-2m accuracy

**Next milestone**: Setup RTK for 5cm accuracy, then full autonomous lawn mowing missions.

---

**Good luck with your project! The documentation above covers everything needed for a professional-grade autonomous rover.**

**Questions? Check TROUBLESHOOTING_GUIDE.md or reference specific section in your setup docs.**

