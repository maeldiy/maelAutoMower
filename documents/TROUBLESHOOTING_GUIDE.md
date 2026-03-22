# Troubleshooting Guide - Autonomous Lawn Mower Rover

---
## 🔗 Quick Navigation

[INDEX](INDEX.md) | [README](README.md) | [QUICK_START](QUICK_START.md) | [ROVER_SYSTEM_OVERVIEW](ROVER_SYSTEM_OVERVIEW.md) | [WIRING_GUIDE](WIRING_GUIDE.md) | [ARDUROVER_PARAMETERS](ARDUROVER_PARAMETERS.md) | [RTK_SETUP_GUIDE](RTK_SETUP_GUIDE.md)

---

## Table of Contents

1. [Power & Hardware Issues](#power--hardware-issues)
2. [Motor & Steering Issues](#motor--steering-issues)
3. [RC Control Problems](#rc-control-problems)
4. [GPS & Autonomous Navigation](#gps--autonomous-navigation)
5. [RTK Specific Issues](#rtk-specific-issues)
6. [Field Mission Failures](#field-mission-failures)
7. [Emergency Procedures](#emergency-procedures)

---

## Power & Hardware Issues

### **Problem: FC won't boot (no lights/beep)**

**Symptoms**: Kakute H7 completely dead, no LED activity

**Diagnosis**:
```
1. Check battery voltage (should be ~36V)
   - Use multimeter on battery terminals
   
2. Check fuse (30-60A)
   - Should be unbroken, conduct continuity test
   
3. Check power module output
   - Input: ~36V ✓
   - Output: ~5.3V on 5V pin ✓
   - GND connected ✓
```

**Solutions** (in order):
```
Step 1: Inspect all connectors
- Wiggle power connectors
- Check for corrosion or loose pins
- Reseat connectors firmly

Step 2: Replace fuse
- Buy 40A automotive blade fuse
- Test original fuse (should show continuity with multimeter)

Step 3: Test power module independently
- Disconnect from FC
- Connect 36V battery directly
- Verify 5V output with multimeter
- If no output, power module is dead (replace)

Step 4: Check flight controller
- Connect via USB to computer
- Should appear in Device Manager
- If not, FC may be damaged (requires replacement)
```

**Prevention**:
- Use correct wire gauge (10-12 AWG for main power)
- Always use fusing rated for motor current (36V × 30A ≈ 1080W)
- Secure all connectors with strain relief

---

### **Problem: Intermittent power loss during operation**

**Symptoms**: Rover randomly goes unresponsive, then recovers

**Diagnosis**:
```
1. Check battery voltage under load
   - Stationary: ~36V ✓
   - During motor acceleration: Should stay >34V
   - If drops below 30V → Battery issue
   
2. Check for loose power connectors
   - Tugging test: wiggle each power wire
   - Should not move freely
   
3. Check current draw
   - Idle: <1A
   - Driving: 10-20A
   - Maximum: <30A
   - If exceeds: Motor jam or short circuit
```

**Solutions**:
```
Loose Connectors:
- Solder all battery power connections
- OR use XT60 connectors (industrial strength)
- Add heat shrink tubing over connections

Battery Issue:
- Test with different 36V battery (borrow if possible)
- If works fine, original battery needs replacement
- Check battery voltage under load with multimeter

Current Limiting:
- Check for motor jamming (can't spin freely)
- Verify motor controller not in protection mode
- May need to reduce ATC_ACCEL_MAX (current limiting)
```

---

## Motor & Steering Issues

### **Problem: Motors won't spin (even in RC manual mode)**

**Symptoms**: Throttle stick doesn't engage motors, FC boots fine

**Diagnosis**:
```
1. Verify motor controller power
   - 36V into motor controller ✓
   - GND connected ✓
   - No protection lights blinking
   
2. Check PWM signals
   - Connect oscilloscope/logic analyzer to motor controller PWM inputs
   - Should see 1000-2000 μs pulses (1.0-2.0 ms)
   - If PWM width is 1500 μs (neutral): motors inactive (correct)
   
3. Verify RC input to FC
   - Mission Planner: Initial Setup → Radio
   - Move throttle stick → should see "Throttle" bar move
   - If bar doesn't move, RC receiver problem
```

**Solutions**:
```
No PWM Signal:
- Check motor controller pin connections (CH1 and CH2)
- Use multimeter to measure voltage on PWM lines
- Should oscillate 0-3.3V
- If flat voltage, FC PWM output dead (firmware issue)

Firmware/Software:
- Reload ArduRover firmware
- Mission Planner: Install Firmware → Select ArduRover
- Reload optimized parameters (see ARDUROVER_PARAMETERS.md)

Motor Controller Unresponsive:
- Check for burnt components (visual inspection)
- Test with external PWM signal from signal generator
- If motor can be driven externally, controller needs tuning
```

---

### **Problem: Motors spin but rover doesn't move**

**Symptoms**: Motor shafts turning, wheels locked or not transmitting power

**Diagnosis**:
```
1. Manual spin test
   - Stop rover
   - Manually try to spin each wheel
   - Should rotate relatively freely
   - If hard to rotate, mechanical jam
   
2. Listen for motor stalling
   - High current draw (20-30A constantly)
   - Grinding, squealing sounds
   - Indicates mechanical resistance
   
3. Check transmission
   - Motor output → Gearbox → Wheel
   - Look for broken teeth, cracked gears
```

**Solutions**:
```
Mechanical Jam:
- Remove wheel, check for debris
- Spin shaft by hand (should be free)
- Check gearbox oil level (if applicable)
- Verify gearbox mounting bolts tight

Broken Transmission:
- Replace gearbox or motor assembly
- Align new motor/gearbox to drive train
- Test before reassembly

Flat/Damaged Wheels:
- Check for punctures (if pneumatic)
- Verify tread contact with ground
- Replace if worn badly
```

---

### **Problem: Rover turns in circles (won't go straight)**

**Symptoms**: Even with straight throttle inputs, rover curves left or right

**Diagnosis**:
```
1. Motor speed comparison
   - Arm rover quietly
   - Note: Left motor RPM vs Right motor RPM
   - Should be equal
   - Use prop on axle as tachometer
   
2. Check for mechanical imbalance
   - Left vs Right wheel diameter
   - Left vs Right gear ratios
   - Unequal tire pressure
   
3. Verify motor controller channels
   - Swap CH1/CH2 connections at flight controller
   - If drift reverses → motor controller issue
   - If drift continues → mechanical problem
```

**Solutions**:
```
Motor Speed Mismatch - Firmware Fix:
- Measure actual motor speeds
- Calculate imbalance ratio (e.g., Left 100 RPM, Right 95 RPM)
- Adjust PWM output: Set second motor slightly higher

In Mission Planner → Parameters:
ATC_SPEED_P = 0.15  (default)
- Increase slightly (0.18) if left/right speed vary

Left/Right Motor Output:
- Open CH1 and CH2 PWM signal inspector
- They should be equal (when going straight)
- If different, adjust trim

Mechanical Imbalance:
- Measure wheel diameter (should be identical)
- Check tire pressure (should be equal)
- Inspect gears for wear
- Realign motor mounting if tilted

Quick Trim Fix:
Mission Planner → Motion Planning → Servo Output
- When outputting CH1:1500, CH2:1500 (neutral)
- Command small forward -> check motor speeds
- If Left < Right: Increase LEFT motor trim slightly
  (Set CH1_TRIM upward or adjust controller potentiometer)
```

---

### **Problem: Steering is "mushy" or delayed**

**Symptoms**: Steering input has noticeable lag or jerky response

**Diagnosis**:
```
1. Check steering parameters
   - ATC_STR_RAT_P: Should be 0.25+ (if too low: mushy)
   - ATC_STR_RAT_D: Should be 0.003+ (if zero: no damping)
   
2. Verify RC transmitter
   - Check for noise/interference near rover
   - Change frequency channel if available
   - Test with antenna fully extended
   
3. Check heading hold
   - Does compass need recalibration?
   - Place rover far from metal/electronics
   - Test heading with magnetic compass app
```

**Solutions**:
```
Parameter Tuning (increase responsiveness):
ATC_STR_RAT_P=0.3       (was 0.25, increase response)
ATC_STR_RAT_I=0.25      (was 0.20, improve hold)
ATC_STR_RAT_D=0.005     (was 0.003, more damping)
ATC_STR_ANG_P=4.0       (was 3.0, better angle tracking)

After each change:
- Save parameters
- Reboot rover
- Test in manual mode
- Adjust slowly - too aggressive causes oscillation

Compass Calibration:
1. Mission Planner: Initial Setup → Compass
2. Click "Compass": Select "Calibration"
3. Rotate rover slowly through all axes (30 sec)
4. Red line should trace smooth sphere
5. If clumpy/distorted: magnetic interference nearby
   - Move away from power lines, metal structures
   - Relocate flight controller away from motors
```

---

## RC Control Problems

### **Problem: RC receiver not responding to transmitter**

**Symptoms**: No motor response when moving RC sticks, red failsafe warning

**Diagnosis**:
```
1. Check receiver power
   - Verify 5V connected to receiver
   - Receiver LED should be solid (not blinking)
   - If blinking: No signal reception
   
2. Check RC signal input to FC
   - Mission Planner: Data → Status → "rc_channels_raw"
   - Should show ~1500 (neutral) for all channels
   - If all showing 0: No signal input
   
3. Verify transmitter
   - Transmitter on and powered
   - Transmitter range switch to "far" or "high"
   - Move throttle stick: Does transmitter beep?
   - Check transmitter battery
```

**Solutions**:
```
Receiver-to-FC Connection:
1. Verify PPM wire connected to RC IN pin on Kakute H7
2. Test with oscilloscope:
   - Should show 1000-2000 μs pulses at ~50 Hz
3. If no signal: Resolder connections on receiver output

Receiver Power:
- Measure voltage on receiver VCC pin
- Should be 5V ± 0.2V
- If low: Check power module output or connections

Transmitter Issues:
- Check transmission frequency (433 MHz)
- Verify antenna not damaged
- Test pairing with receiver
  (usually press pairing button on receiver)

RC Calibration in Mission Planner:
1. Initial Setup → Radio Calibration
2. Plug in battery (DISARMED, props off!)
3. Move each stick through full range
4. Verify all channels respond
5. Save calibration
```

---

### **Problem: Motors spin during failsafe (should stop)**

**Symptoms**: Rover keeps moving when RC signal lost or transmitter off

**Diagnosis**:
```
1. Check failsafe configuration
   - FS_THR_ENABLE: Should be 1 (enabled)
   - FS_THR_VALUE: Should be 910 (brake position)
   
2. Check failsafe mode
   - FS_ACTION: Should be 2 (Brake)
   - Current setting might be 0 (No action) or 1 (RTB)
```

**Solutions**:
```
In Mission Planner → Parameters:

FS_THR_ENABLE = 1              (Enable failsafe)
FS_THR_VALUE = 910             (Brake/Stop position)
FS_ACTION = 2                  (Brake on failsafe)
FS_TIMEOUT = 1.5               (Trigger after 1.5s no signal)

After setting:
1. Save parameters
2. Reboot rover
3. Test: Turn off transmitter
   - Rover should stop within 2 seconds
   - Should not move forward
4. Turn transmitter back on
   - Verify recover control
```

---

## GPS & Autonomous Navigation

### **Problem: GPS won't get lock (no GNSS signal)**

**Symptoms**: No position data, autonomous missions won't start

**Diagnosis**:
```
1. Check GPS physical connection
   - UART1 RX/TX correctly wired
   - All 4 pins connected (VCC, GND, TX, RX)
   - No loose connectors
   
2. Verify GPS module power
   - 5V at GPS VCC pin (multimeter)
   - GND properly connected
   - Current draw 200-400 mA
   
3. Check antenna placement
   - Antenna on top of rover (not blocked)
   - 10 cm away from compass/electronics
   - Clear view of sky (go outdoors!)
   - Not inside building (signals blocked)
   
4. Monitor GPS data
   - Mission Planner: Data → Messages
   - Look for "AHRS: GPS OK"
   - Watch "gps_numsat" (should increase 0 → ... → 6+)
   - Watch "gps_status" (should go 0 → 1 → 2 → 3)
```

**Solutions** (in order of likelihood):

```
Step 1: Move Outdoors (Most Common)
- GPS needs clear sky view
- Move away from buildings/trees
- Wait 3-5 minutes for initial lock

Step 2: Verify Physical Connections
- Disconnect/reconnect UART1 cable
- Test with multimeter for continuity
- Try different UART cable

Step 3: Check GPS Module
- Power off rover
- Connect GPS module directly to laptop via USB adapter
  (if module has USB interface)
- Run u-center software
- Should show satellite constellation

Step 4: UART Configuration
- Mission Planner: Data → Messages
- Should see regular UBX messages
- If nothing: UART1 configuration issue
  Mission Planner → Initial Setup → Serial Ports:
  SERIAL1_PROTOCOL = 9 (GPS)
  SERIAL1_BAUD = 38400
  
Step 5: Replace GPS Module
- If all above fail, GPS module may be damaged
- Order replacement M10Q-5883
- Install following WIRING_GUIDE.md
```

**GPS Lock Timeline**:
```
Cold start (first time):
├─ 0-30 sec: Acquiring satellites (gps_status = 0)
├─ 30-60 sec: Processing data (gps_status = 1)
├─ 60-120 sec: Coarse fix (gps_status = 2)
└─ 120-180 sec: Good fix (gps_status = 3) ✓

Warm start (powered off <12 hours):
├─ 0-15 sec: Quick acquisition
└─ 15-30 sec: Good fix ✓

u-blox ZED-F9P (multi-constellation L1/L2):
- Typically locks in 30-60 seconds
- If taking > 3 minutes: antenna issue
```

---

### **Problem: GPS has lock but rover drifts in autonomous mode**

**Symptoms**: GPS shows position, but LOITER mode drifts 1-2 meters off

**Diagnosis**:
```
1. Check GPS accuracy
   - Mission Planner: Status → "gps_hdop"
   - Should be < 1.0 (good) or < 2.0 (acceptable)
   - If > 2.5: Accuracy is poor, wait for better lock
   
2. Verify EKF fusion
   - Mission Planner: Status → "ekf_flags"
   - Should show green (all fusion healthy)
   - If red: Sensor disagreement (compass? baro?)
   
3. Check compass health
   - Mission Planner: Status → "mag_field"
   - Should be consistent magnitude (~43000 nT at latitude 43°N)
   - If varies wildly: Compass interference/drift
```

**Solutions**:

```
Poor GPS Signal (HDOP > 2.0):
- Wait for more satellites to appear
- Move to location with clear sky
- Note: Multi-constellation GPS (GLONASS + Galileo) helps
- Usually improves after 5-10 minutes

Compass Interference:
1. Identify interference source:
   - Power lines above rover
   - Metal structures near rover
   - Electronic devices (WiFi, 433MHz transmitter)
   
2. Relocate rover > 10 meters away

3. Recalibrate compass:
   Mission Planner → Initial Setup → Compass
   - Select compass (usually #1)
   - Click "Calibration"
   - Slowly rotate rover through all angles (30 sec)
   - Save calibration

4. If calibration fails:
   - Move to completely clear location
   - Empty parking lot or field
   - Try again

Compass Mounting:
- Should be > 10 cm away from motors/power lines
- Mount on plastic/non-magnetic standoffs
- Verify not near compass offset from accelerometer
  Check: COMPASS_OFS_X/Y/Z parameters
```

---

### **Problem: Rover overshoots waypoints in MISSION mode**

**Symptoms**: Rover misses waypoint, circles back, overshoots again

**Diagnosis**:
```
1. Check navigation parameters
   - WP_RADIUS: Waypoint acceptance radius (meters)
   - WP_RADIUS too large: Sloppy navigation
   - WP_RADIUS too small: Never approaches waypoint
   
2. Check speed parameters
   - CRUISE_SPEED: Usually 2 m/s (reasonable)
   - Too fast for small rover: Overshooting
   
3. Check steering smoothing
   - ATC_STR_RAT_D: Damping (prevent oscillation)
   - ATC_STR_ANG_P: Angle tracking gain
```

**Solutions**:

```
Parameter Tuning:
In Mission Planner → Parameters:

WP_RADIUS = 1.0                (Or 0.5 for tighter missions)
CRUISE_SPEED = 1.5             (Reduce from 2.0 m/s)
ATC_STR_RAT_D = 0.005          (Add damping)
ATC_STR_RAT_P = 0.2            (Reduce from 0.25 if oscillating)

Test after each change! Mission planning with fewer waypoints.

Steering Smoothness:
ATC_STR_RATE_FLTD = 0.5        (Low-pass filter for smoother input)
ATC_STR_ANG_P = 2.5            (Reduce aggressive steering)

RTK Advantage:
With RTK (see RTK_SETUP_GUIDE.md), can reduce WP_RADIUS to 0.2 m
Much tighter autonomous following becomes possible!
```

---

## RTK Specific Issues

### **Problem: RTK corrections not being applied (gps2_status = 0)**

**Symptoms**: Two GPS modules connected, but RTK status shows no fix, only primary GPS

**Diagnosis**:
```
1. Check WiFi module connection
   - ESP8266 powered? (LED blinking)
   - Connected to WiFi? (check router)
   - Correct NTRIP caster credentials?
   
2. Verify UART connection
   - GPS2 on UART3 (or configured port)
   - Measure voltage on TX/RX pins
   - Should oscillate 0-3.3V at ~10 Hz (RTCM3 data)
   
3. Check RTCM3 stream
   - Can use software to monitor NTRIP stream
   - Verify RTCM messages: 1004, 1077, 1087 (standard)
   - If stream broken: Caster/WiFi issue
```

**Solutions**:

```
WiFi/NTRIP Connection:
1. Test WiFi module independently
   - USB to serial adapter → ESP8266
   - Open Serial Monitor (115200 baud)
   - Type "AT" → should respond "OK"
   - Type "AT+CWLAP" → should list WiFi networks
   
2. Verify NTRIP caster credentials
   - CentipedeRTK website → SMVD station
   - Download latest caster address and mount point
   - Update ESP8266 firmware with new credentials
   - Test: Open browser → rtk.centipede.fr (should respond)

3. Check latency
   - NTRIP stream may have 5-10 second delay
   - EKF needs time to process corrections
   - Wait 30-60 seconds after power-on for convergence

UART3 Configuration:
Mission Planner → Initial Setup → Serial Ports:
SERIAL3_PROTOCOL = 9              (GPS protocol)
SERIAL3_BAUD = 115200             (or check module speed)

Or use Mission Planner:
Initial Setup → uBlox GPS → Configure for RTCM3 input

Fallback: Use Single GPS
If RTK setup is too complex, simply use primary GPS:
- Still get 1-2 meter accuracy
- Much simpler wiring
- Can upgrade later
```

---

### **Problem: RTK signal occasionally drops out**

**Symptoms**: RTK Fixed (gps2_status = 5), then drops to Float (4) or No Fix (0)

**Diagnosis**:
```
1. Check WiFi stability
   - WiFi RSSI (signal strength): Should be > -70 dBm
   - WiFi scan for interference (other networks)
   - Dropouts correlate with WiFi glitches?
   
2. Check NTRIP caster uptime
   - Visit CentipedeRTK status page
   - SMVD base should show "Active"
   - Check for maintenance window
   
3. Check distance from base
   - Your site is ~5 km from SMVD base (ideal)
   - Distance > 20 km: Works but less accurate
```

**Solutions**:

```
WiFi Stability:
- Move rover / WiFi antenna for better signal
- Change WiFi channel (avoid 2.4 GHz congestion)
- Use dual-band 5GHz WiFi if available
- Add external antenna to ESP8266

Base Station Issues:
- Check CentipedeRTK status page regularly
- Contact network admin if frequent outages
- SMVD is well-maintained (your logs confirm this!)

Internet Connectivity:
- Ensure stable internet connection to WiFi router
- Monitor latency: NTRIP → WiFi → Rover should be < 2 seconds
- Can check with ping tool

Fallback Strategy:
- If RTK drops: Rover switches to primary GPS
- Navigation continues (less precise, but functional)
- When RTK recovers: Automatically re-engages
- This is normal behavior - not a failure!
```

---

## Field Mission Failures

### **Problem: Mission starts but rover doesn't follow waypoints**

**Symptoms**: Armed in MISSION mode, goes straight or turns randomly

**Diagnosis**:
```
1. Check mission upload
   - Mission Planner: Fly → Load from Vehicle
   - View mission: Should show waypoints as circles
   - If empty: Mission never uploaded to rover
   
2. Verify mode switch
   - Transmitter mode switch set to MISSION?
   - Check RC training mode (if available)
   - Flight mode should show "MISSION" in HUD
   
3. Confirm arming state
   - Flight controller armed? (LED solid green)
   - If blinking red: Failsafe active
   - Pre-arm checks passed? (all green)
```

**Solutions**:

```
Mission Upload:
1. Mission Planner: Fly panel → "List" tab
2. Create waypoints (right-click on map)
3. Click "Upload" (blue arrow icon)
4. Wait for "Uploaded Complete"
5. Verify in "Load from Vehicle" (should match)

Arming Issues:
Mission Planner → Initial Setup → Mandatory Hardware
- Accelerometer: Green
- Compass: Green (arrow pointing N)
- Radio: Green (all channels visible)
- Battery: Green (voltage showing)

If any RED: Fix before arming
Common fixes:
- Compass calibration
- Radio calibration  
- GPS lock required for autonomous

Flight Mode:
- Transmitter has mode switch (usually Ch5 or Ch6)
- 3 positions: Manual, Stabilize, Mission
- Move switch to "Mission" position
- HUD should show "MISSION" blue text
```

---

### **Problem: Autonomous mission works but accuracy is poor**

**Symptoms**: Mission completes but rover path zigzags, misses waypoints by 1-2 meters

**Diagnosis**:
```
1. Check GPS accuracy (HDOP)
   - Status → "gps_hdop": Should be < 1.5
   - If > 2.0: GPS solution is poor
   
2. Verify compass calibration
   - Heading should be consistent
   - Test: Point rover North → Status should show ~0°
   
3. Check waypoint spacing
   - Waypoints < 5 meters apart? Create cleaner route
   - Use RTK for better accuracy
```

**Solutions**:

```
Improve Baseline Accuracy:
Without RTK (standard GPS):
- Accuracy: ~1-2 meters (inherent limitation)
- Accept wobbling ±1m from desired path
- Use larger waypoint acceptance radius
  WP_RADIUS = 2.0 m (more forgiving)

With RTK (upgrade):
- Accuracy: ~0.05 meters (20x improvement!)
- WP_RADIUS = 0.2 m (very tight)
- Path becomes smooth and precise
- See RTK_SETUP_GUIDE.md for setup

Compass Calibration:
- Redo compass calibration in clear field
- Away from power lines, vehicles, metal
- Verify with compass app on smartphone
  Should match heading shown in Mission Planner
```

---

## Emergency Procedures

### **Rover won't stop (stuck in forward)**

**Immediate Action**:
```
1. Switch mode: Move transmitter mode switch away from MISSION
   → Should switch to STABILIZE (manual control)
   → Move throttle stick backward (pull toward you)
   
2. If stuck: Kill switch
   → Disconnect battery (either terminal)
   → Or trigger failsafe (turn off transmitter)
   
3. Locate emergency stop
   → Many motor controllers have E-stop input
   → Pull/disconnect that line
```

**Prevention**:
- FS_TIMEOUT = 1.5 (Failsafe after 1.5 sec no signal)
- Always test failsafe before field deployment
- Keep hand on transmitter (never let go)

---

### **Rover flipped over but still powered**

**Immediate Action**:
```
1. Failsafe triggered: Turn off transmitter
2. If motors still spinning: Disconnect battery
3. Right it over carefully
4. Wait 30 seconds before restarting
5. Check for bent components
```

**Inspection Checklist**:
- [ ] Frame not bent
- [ ] Wheels aligned properly
- [ ] Motors/gearboxes intact
- [ ] Antenna not broken
- [ ] Connectors still seated
- [ ] Battery connections tight

---

### **Lost GPS but need to recover rover**

**Technique**:
```
1. Switch to STABILIZE mode (manual control)
2. Use landmarks to navigate toward rover position
3. Transmitter range: Usually 100-500 m (line-of-sight)
4. Follow rover beeping (if audio locator installed)
5. If no signal: Walk antenna pattern (expand squares)
   to find weak signal area

Prevention:
- Install GPS logger (records position every 5 sec)
- Download logs from SD card to see path
- Use compass heading for recovery (North indicator)
- RTK provides ultra-precise position (cm-level)
```

---

## Diagnostic Logging & Analysis

### **Enable Full Logs** (for crashes/issues)

```
Mission Planner → Data → Logs
→ Select dataflash logs
→ Download latest .bin file
→ Analyze with LogAnalyzer tool
```

**Common Issues Visible in Logs**:
- **NaN values** = Sensor dropout (physical disconnect)
- **Clipping** = Sensor saturation (too much movement)
- **Oscillation** = PID tuning too aggressive
- **GPS jumps** = Bad GPS lock quality
- **Current spikes** = Motor jam or short

---

## Quick Reference: Common Settings

```
Parameter              Symptom           Adjustment
────────────────────────────────────────────────────────
ATC_ACCEL_MAX         Jumpy startup      ↓ (reduce jerk)
ATC_STR_RAT_P         Mushy steering     ↑ (increase response)
ATC_STR_RAT_D         Oscillating        ↑ (add damping)
GPS_AUTO_CONFIG       GPS issues         Set to 1
EK3_ENABLE            Navigation drift   Set to 1
WP_RADIUS             Overshoots         ↓ (tighter)
CRUISE_SPEED          Erratic motion     Check sanity (0-3 m/s)
FS_THR_ENABLE         No failsafe        Set to 1
FS_ACTION             Failsafe issues    Set to 2 (Brake)
COMPASS_AUTO_ROT      Heading wanders    Set to 2
```

---

## When to Seek Help

**Before posting for help, collect**:
1. Dataflash logs (.bin file from SD card)
2. Parameter file (.param backup)
3. Photos of rover and wiring
4. Video of symptom
5. Exact error messages (screenshot)
6. Detailed reproduction steps

**Resources**:
- ArduPilot Discuss: https://discuss.ardupilot.org/
- Your local robotics club
- CentipedeRTK Discord/forum
- This documentation (cross-reference sections)

