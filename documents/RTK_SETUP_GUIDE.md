# RTK Setup Guide - CentipedeRTK Integration

---
## 🔗 Quick Navigation

[INDEX](INDEX.md) | [README](README.md) | [QUICK_START](QUICK_START.md) | [ROVER_SYSTEM_OVERVIEW](ROVER_SYSTEM_OVERVIEW.md) | [WIRING_GUIDE](WIRING_GUIDE.md) | [ARDUROVER_PARAMETERS](ARDUROVER_PARAMETERS.md) | [TROUBLESHOOTING_GUIDE](TROUBLESHOOTING_GUIDE.md)

---

## Quick Overview

Your rover can achieve **2-5 cm positioning accuracy** (instead of 1-2 m) by connecting to the local **SMVD RTK base station** via the CentipedeRTK network.

### Base Station Details (from your data)

| Property | Value |
|----------|-------|
| **Station ID** | SMVD (A867) |
| **Location** | 5.182411° E, 43.729197° N |
| **Altitude** | 208.28 m (RGF93 datum) |
| **Status** | ✅ ACTIVE (validated 2026-03-20) |
| **Equipment** | RTKBase + u-blox ZED-F9P |
| **Data Format** | RTCM3 (Real Time Correction Messages) |
| **Systems** | GPS, GLONASS, Galileo, BeiDou (Multi-constellation) |
| **Network** | CentipedeRTK (Community RTK network) |
| **Frequencies** | L1-L2 (Dual frequency for accuracy) |
| **Receiver Type** | u-blox ZED-F9P (same as your rover!) |

---

## How RTK Works

```
┌─────────────────────────────────────────────────────────────┐
│                    GNSS SATELLITES                         │
│  (GPS, GLONASS, Galileo, BeiDou)                          │
└────────────┬────────────────────────────┬──────────────────┘
             │                            │
         Raw GNSS                      Raw GNSS
         Signals                       Signals
             │                            │
     ┌───────▼────────┐          ┌───────▼────────┐
     │  RTK Base      │          │  Rover Rover   │
     │  (SMVD Fixed)  │          │  (Mobile)      │
     │  Calculates    │          │                │
     │  Error = 2-5cm │          │  In 1-2m       │
     │                │          │                │
     │  ZED-F9P       │          │  ZED-F9P      │
     │  + u-blox      │          │  M10Q-5883    │
     └────────┬───────┘          └────────┬───────┘
              │                          │
          RTCM3 Error                    │
          Corrections                    │
          (Radio/Internet)               │
              │                          │
              └──────────────┬───────────┘
                             │
                      "Corrections" applied
                             │
                      ┌──────▼────────┐
                      │  Rover Result  │
                      │  2-5 cm err    │
                      │  (100x better!)│
                      └────────────────┘
```

---

## RTK Connection Methods for Your Rover

You have **three main options** to receive corrections from SMVD base:

### **Option 1: NTRIP over WiFi (Easiest)**
- **Requirement**: WiFi module on rover
- **Method**: Request corrections from CentipedeRTK NTRIP caster
- **Setup**: Configure u-blox GPS with connection details
- **Range**: Unlimited (internet-based)
- **Latency**: 1-5 seconds (acceptable for rover)

**Advantages**:
- ✅ Works anywhere with WiFi
- ✅ No extra radio equipment
- ✅ Highly accurate
- ✅ Free CentipedeRTK service

**Disadvantages**:
- ❌ Requires WiFi on rover
- ❌ Higher power consumption
- ❌ Dependency on internet

---

### **Option 2: 433MHz Radio Link (Your Current Hardware)**
- **Requirement**: 433MHz modem for corrections
- **Method**: Connect RTCM3 corrections via serial link
- **Range**: Typically 1-5 km line-of-sight
- **Latency**: < 100 ms (very fast)

**Advantages**:
- ✅ Works without internet
- ✅ Low latency (fast corrections)
- ✅ Can re-use existing 433MHz gear
- ✅ Autonomous in remote areas

**Disadvantages**:
- ❌ Limited range (~5 km)
- ❌ Requires base station radio
- ❌ Line-of-sight dependent

---

### **Option 3: 4G/LTE Modem (Future-Proof)**
- **Requirement**: 4G/LTE module on rover
- **Method**: NTRIP via cellular connection
- **Range**: Unlimited (cellular coverage)
- **Latency**: 1-5 seconds

**Advantages**:
- ✅ Works anywhere with 4G coverage
- ✅ Unlimited range
- ✅ Always-on corrections

**Disadvantages**:
- ❌ Requires 4G subscription
- ❌ Higher power consumption
- ❌ Module cost (~€80-150)

---

## Recommended Setup: NTRIP over WiFi (Quick Start)

Since you're starting, I recommend **Option 1: NTRIP WiFi** for testing accuracy gains.

### Step 1: Get CentipedeRTK Account

1. Visit: **https://centipederover.fr/** (or equivalent)
2. Find "SMVD" base station in map
3. Note the **NTRIP caster address** (usually: `rtk.centipede.fr` or similar)
4. Get your **credentials** (username/password)
5. Find the **mount point** for SMVD (typically: `SMVD` or `SMVD0`)

**Example credentials:**
```
Caster: rtk.centipede.fr:2101
Mount Point: SMVD
Username: [your_login]
Password: [your_password]
```

---

### Step 2: Connect WiFi Module to Kakute H7

**Hardware**:
- ESP8266 or ESP32 WiFi module (€5-15)
- Or use existing telemetry radio with NTRIP bridge

**Wiring**:
```
ESP8266/ESP32:
├─ VCC -> 5V (from power module)
├─ GND -> GND
├─ TX -> Kakute H7 UART3 or UART4 RX
└─ RX -> Kakute H7 UART3 or UART4 TX
```

**ArduPilot Configuration**:
```
SERIAL3_PROTOCOL = 9     # GPS protocol
GPS2_TYPE = 1            # u-blox (for corrections)
GPS2_COM_PORT = 3        # Use UART3
```

---

### Step 3: Configure u-blox GPS for RTK

The u-blox ZED-F9P on your rover supports RTCM3 corrections via **any input port**.

**Option A: Use Mission Planner**
1. Connect rover via USB
2. Go: **Initial Setup → Optional Hardware → uBlox GPS**
3. Select: **RTCM3** input mode
4. Enter NTRIP caster details
5. Reboot GPS module

**Option B: Use u-center Software** (Advanced)
1. Download [u-center](https://www.u-blox.com/en/product/u-center) from u-blox
2. Connect via USB/serial
3. Configure RTCM3 input on port you're using
4. Save configuration to GPS

---

### Step 4: Verify RTK Lock

**In Mission Planner**:
1. Go: **Data → Status**
2. Look for `gps2_status` = **5** (RTK Fixed - best) or **4** (RTK Float - ok)
   - 0-3 = No RTK
   - 4 = RTK Float (decimeter level)
   - 5 = RTK Fixed (centimeter level) ✅

**Field Verification**:
```
Before RTK:
- GPS accuracy circle: 1-2 meter radius
- Rover drifts significantly

After RTK (Fixed):
- GPS accuracy circle: 2-5 cm radius
- Rover position very stable
- Autonomous missions much more accurate
```

---

## Detailed Configuration: WiFi + NTRIP

Here's the complete setup using free CentipedeRTK network:

### Hardware Setup

```
Kakute H7 (Flight Controller)
├── UART1: GPS/Compass (MATEKSYS M10Q-5883)
├── UART2: Telemetry (433MHz or other)
├── UART3: ⬅️ Second GPS (ESP8266 - WiFi/NTRIP)
│         └─ Receives RTCM3 corrections
└── UART4: [Available]

                    NTRIP Stream
ESP8266 WiFi  <──────────────────── CentipedeRTK Caster
  Module      RTCM3 Corrections      (rtk.centipede.fr:2101)
     │
     │ UART
     ▼
  Kakute H7 ──► Routes to GPS ──► u-blox applies corrections
```

### ESP8266 Firmware (Simple RTCMv3 Bridge)

Option: Use **ArduSimple RTK2Go board** (pre-configured):
- Comes with WiFi built-in
- Costs ~€120 but saves configuration time
- Ready-to-use with CentipedeRTK

OR: Use bare ESP8266 with custom firmware:
```cpp
// Simplified example - full code in repository
#include <WiFi.h>

void setup() {
  Serial.begin(115200); // Connected to Kakute UART3
  WiFi.begin("SSID", "password");
  
  // Connect to NTRIP caster
  client.connect("rtk.centipede.fr", 2101);
  client.println("GET /SMVD HTTP/1.0");
  client.println("Authorization: Basic [base64_credentials]");
}

void loop() {
  if (client.available()) {
    byte b = client.read();
    Serial.write(b); // Forward RTCM3 to flight controller
  }
}
```

---

## ArduRover Parameters for RTK

Add/modify these in Mission Planner → Initial Setup → Parameters:

```
# ============================================================
# RTK Configuration
# ============================================================

# Primary GPS (onboard M10Q-5883)
GPS1_TYPE=1                           # u-blox
GPS1_COM_PORT=1                       # UART1
GPS1_RATE_MS=200                      # 5 Hz

# Secondary GPS (WiFi module for corrections)
GPS2_TYPE=1                           # u-blox
GPS2_COM_PORT=3                       # UART3 (ESP8266)
GPS2_RATE_MS=200                      # 5 Hz
GPS_BLEND_MASK=5                      # Blend both sources (0b101)

# EKF3 will automatically blend both GPS sources
# When RTK lock is achieved, position accuracy jumps to 2-5 cm

# Extended Kalman Filter (optimized for RTK)
EK3_ENABLE=1                          # Use EKF3
EK3_SRC1_POSXY=3                      # GPS for position
EK3_SRC1_VELXY=3                      # GPS for velocity
EK3_HGT_DELAY=60                      # RTK correction latency compensation
EK3_POS_I_GATE=500                    # Tight gate for RTK
EK3_VEL_I_GATE=500                    # Trust RTK velocity

# Navigation tuning for RTK accuracy
WP_RADIUS=0.5                         # Waypoint acceptance radius (m)
                                      # Can be tighter with RTK
```

---

## Testing RTK Step-by-Step

### **Test 1: Verify RTCM3 Stream**

```
1. Power on rover
2. Connect to telemetry (Mission Planner)
3. Go: Data → Status
4. Watch messages flow in:
   - gps_numsat: Increases when satellite lock achieved
   - gps2_status: Changes from 0→1→2→3→4→5 over 30-60 seconds
5. When gps2_status = 5: RTK FIXED ✅
```

### **Test 2: Accuracy Comparison (Stationary)**

```
Without RTK:
1. Park rover outdoors (clear sky)
2. Wait 5 minutes
3. In Mission Planner, note position (Status → Location)
4. Move rover 10 meters away
5. Return to original spot
6. Note position drift: typically 1-3 meters ❌

With RTK:
Repeat above steps:
Position drift: typically 2-5 cm ✅ (100x improvement!)
```

### **Test 3: Autonomous Mission (with RTK)**

1. Create mission in Mission Planner with 4+ waypoints
2. Ensure GPS2_STATUS = 5 (RTK Fixed) before arm
3. Arm rover → Select MISSION mode → Start mission
4. Monitor path:
   - Without RTK: Wobbles ±1m off planned path
   - With RTK: Follows ±5cm of planned path ✅

---

## Troubleshooting RTK Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| **No RTK lock (gps2_status = 0-3)** | No NTRIP connection | Check WiFi, caster credentials, ESP8266 firmware |
| **RTK Float but not Fixed** | Weak satellite signal | Move to clearer sky, dual frequency may help |
| **RTK drops occasionally** | Intermittent connection | Check WiFi strength, antenna placement |
| **Rover drifts despite RTK** | EKF not trusting corrections | Check EK3_POS_I_GATE, verify correction latency < 2s |
| **High latency (>5s)** | Network congestion | Use 433MHz radio link instead of WiFi |

---

## Advanced: 433MHz Radio RTK Link

If you want RTK without WiFi (e.g., for long autonomous missions):

### Equipment
- 433MHz modem with high throughput (e.g., RFD900x)
- RTCM3 encoder at base station
- Linux/Raspberry Pi at SMVD base to handle NTRIP→RTCM3→433MHz

### Wiring
```
SMVD Base Station:
CentipedeRTK → NTRIP → Raspi → 433MHz Transmitter
                                     │
                                     │ RF Signal
                                     ▼
Rover:
433MHz Receiver → Kakute H7 UART3 (as GPS correction input)
```

---

## CentipedeRTK Network Details

**Network**: CentipedeRTK (Community-driven French RTK network)

**How it Works**:
1. SMVD base station broadcasts corrections
2. NTRIP caster aggregates multiple bases
3. Your rover requests corrections for your location
4. Network sends tailored RTCM3 messages

**Free Tier**: 
- ✅ Access to all public base stations
- ✅ Standard RTCM3 messages
- ✅ Unlimited connections
- ✅ Commercial use allowed

**Coverage**:
- France: Excellent coverage (especially south-east)
- Europe: Growing network of bases
- Global: Can request reciprocal access

**More Info**: 
- https://centipede.fr (official site)
- https://centipedeRover.fr (community platform)

---

## Cost Analysis: RTK Implementation

| Option | Equipment Cost | Annual Cost | Accuracy | Range |
|--------|---|---|---|---|
| **No RTK** | $0 | $0 | 1-2m ❌ | N/A |
| **WiFi NTRIP** | ~€30 (ESP8266) | €0 (free CentipedeRTK) | 2-5cm ✅ | Unlimited |
| **433MHz Radio** | ~€150 (RFD900) | €50-80 (base station) | 2-5cm ✅ | 5km |
| **4G LTE** | ~€100 (module) | €40/mo (subscription) | 2-5cm ✅ | Unlimited |

**Recommendation**: **Start with WiFi NTRIP** (cheapest, easiest setup)

---

## Verification Checklist

Before field testing autonomous missions with RTK:

- [ ] ESP8266 WiFi module wired to UART3
- [ ] WiFi credentials loaded in module
- [ ] NTRIP caster address correct: `rtk.centipede.fr:2101`
- [ ] Credentials verified with CentipedeRTK
- [ ] Mount point correct: `SMVD` or similar
- [ ] GPS1_TYPE = 1, GPS2_TYPE = 1 (both u-blox)
- [ ] EK3_ENABLE = 1 (fusion enabled)
- [ ] Rover powers on with clear sky view
- [ ] Wait 2 minutes for RTK lock
- [ ] Mission Planner: Status → gps2_status = 5 (RTK Fixed)
- [ ] Test mission created with 4+ waypoints
- [ ] Autonomous navigation: Rover follows ±5cm of path ✅

---

## Next Steps

1. **Immediate**: Test WiFi NTRIP with borrowed ESP8266
2. **Short-term**: Optimize autonomous parameters (see ARDUROVER_PARAMETERS.md)
3. **Medium-term**: Create field missions and test accuracy
4. **Long-term**: Integrate RTK into production workflow

---

## Resources

- **CentipedeRTK**: https://centipede.fr
- **ArduRover RTK Docs**: https://ardupilot.org/rover/common-gps-for-fast-position-updates.html
- **u-blox ZED-F9P**: https://www.u-blox.com/en/product/zed-f9p-module
- **ArduPilot GPS/EKF**: https://ardupilot.org/rover/parameters.html#gps

---

## Final Note: Local Advantage

You're extremely fortunate to be near a validated CentipedeRTK base station (SMVD). Most remote robotics projects require setting up their own base station (expensive!). Your location gives you:

✅ Sub-5cm positioning accuracy
✅ Free RTK corrections
✅ Community support
✅ Multi-constellation corrections (GPS, GLONASS, Galileo, BeiDou)

Take advantage of this for best autonomous navigation performance!

