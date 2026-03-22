# Detailed Wiring Diagram & Setup Guide

---
## 🔗 Quick Navigation

[INDEX](INDEX.md) | [README](README.md) | [QUICK_START](QUICK_START.md) | [ROVER_SYSTEM_OVERVIEW](ROVER_SYSTEM_OVERVIEW.md) | [ARDUROVER_PARAMETERS](ARDUROVER_PARAMETERS.md) | [RTK_SETUP_GUIDE](RTK_SETUP_GUIDE.md) | [TROUBLESHOOTING_GUIDE](TROUBLESHOOTING_GUIDE.md)

---

## Kakute H7 Flight Controller Pin Layout

The Kakute H7 has the following main connections:
- **UART Ports**: UART1 (Serial 1), UART2 (Serial 2), UART3 (Serial 3), UART4 (Serial 4)
- **PWM Outputs**: CH1-CH4 (Motor outputs)
- **CAN Bus**: CAN1, CAN2 (optional, for advanced sensors)
- **SPI**: SD card slot (logging)
- **ADC**: Battery voltage/current monitoring
- **Power**: 5V out, GND

---

## Complete Wiring Schematic

```
┌──────────────────────────────────────────────────────────────────────┐
│                           POWER DISTRIBUTION                          │
│                                                                       │
│  ┌─────────────┐                                                     │
│  │  36V Battery│                                                     │
│  │  (Primary)  │                                                     │
│  └──────┬──────┘                                                     │
│         │                                                             │
│    ┌────┴────┐                                                       │
│    │          │                                                       │
│    │ (FUSE    │                                                       │
│    │  30-60A) │                                                       │
│    │          │                                                       │
│    └────┬─────┘                                                       │
│         │                                                             │
│    ┌────┴──────────────────────────────────────────────────┐         │
│    │                                                        │         │
│    ▼                                                        ▼         │
│ ┌─────────────────┐                            ┌──────────────────┐│
│ │ Power Module ①  │                            │ Motor Controller ││
│ │ (BEC for FC)    │                            │ (H-Bridge/ESC)   ││
│ │ IN: 36V         │                            │ Motor1 (Left)    ││
│ │ OUT: 5.3V/3A    │                            │ Motor2 (Right)   ││
│ │ GND: Common     │                            │ IN: 36V Battery  ││
│ └────────┬────────┘                            └──────────────────┘│
│          │                                                          │
│          └─────────────────────────────────────────────────────────┘
│                           5V & GND
└──────────────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Wiring Instructions

### **STEP 1: Power Module Connection (Kakute H7)**

**From 36V Battery to Flight Controller:**

```
Battery +36V  ──[30-60A Fuse]──┬──► Motor Controller
                                │     (direct 36V)
                                │
                                └──► Power Module
                                      (5.3V output to FC)
                                
Battery GND ──────────────────────────► Common GND
```

| Connection | Pin Description | Wire Size |
|-----------|------------------|-----------|
| Battery + (36V) | Power Module IN+ | 10-12 AWG |
| Battery - (GND) | Power Module IN-, Common GND | 10-12 AWG |
| Power Module OUT+ (5.3V) | Kakute H7 5V pin | 16 AWG |
| Power Module OUT- (GND) | Kakute H7 GND | 16 AWG |

**Critical**: Use proper fusing (30-60A) between battery and power distribution.

---

### **STEP 2: Motor Connections**

**Motor Controller to Flight Controller:**

```
Kakute H7 PWM Outputs:
├─ CH1 (Main Out 1) ──► Motor Controller: LEFT Motor PWM signal
├─ CH2 (Main Out 2) ──► Motor Controller: RIGHT Motor PWM signal
├─ CH3 (Main Out 3) ──► [Reserved for future]
└─ CH4 (Main Out 4) ──► [Reserved for future]

Motor Controller Signal GND: Connected to FC GND
```

| Connection | Kakute Pin | Motor Controller Pin | Signal |
|-----------|-----------|----------------------|--------|
| Left Motor PWM | CH1 Signal | PWM_LEFT | 3.3V PWM |
| Right Motor PWM | CH2 Signal | PWM_RIGHT | 3.3V PWM |
| Motor Controller GND | GND | GND | Common Ground |
| Motor1 Phase | Motor1+ | Motor1A | 36V variable |
| Motor1 Phase | Motor1- | Motor1B | 36V variable |
| Motor2 Phase | Motor2+ | Motor2A | 36V variable |
| Motor2 Phase | Motor2- | Motor2B | 36V variable |

**Motor Wiring Note**: 
- Connect motorcycle/lawn mower DC motors directly to controller
- Verify motor rotation direction in firmware (can be reversed)
- Both motors should spin the same direction for straight movement

---

### **STEP 3: GPS/Compass Module (MATEKSYS M10Q-5883)**

**UART1 Connection (Recommended for GPS):**

```
MATEKSYS M10Q-5883:
├─ VCC (5V) ────────► Kakute H7 5V output
├─ GND ─────────────► Kakute H7 GND
├─ TX ──────────────► Kakute H7 UART1 RX
└─ RX ──────────────► Kakute H7 UART1 TX
```

| Connection | Kakute H7 Pin | M10Q-5883 Pin | Notes |
|-----------|--------------|---------------|-------|
| 5V Power | +5V | VCC | Use regulated 5V from power module |
| GND | GND | GND | Common ground |
| TX (GNSS→FC) | UART1 RX | TX | 3.3V level (built-in tolerant) |
| RX (FC→GNSS) | UART1 TX | RX | 3.3V to 5V (pull-up compatible) |

**GPS Placement**:
- Mount on top of rover, away from motors
- Orientation: Default (no rotation needed initially)
- Antenna facing upward
- Distance from compass: > 10cm, away from metal

---

### **STEP 4: RC Receiver (433MHz RF Wireless Relay)**

**PPM Signal Connection:**

```
433MHz RF Receiver Output:
└─ PPM Signal ──────► Kakute H7 RC Input (RCIN)
└─ GND ─────────────► Kakute H7 GND
└─ +5V (if needed) ──► Kakute H7 5V output
```

| Connection | Kakute H7 Pin | RF Receiver Pin | Signal Type |
|-----------|--------------|-----------------|-------------|
| PPM Signal | RC IN | PPM_OUT | 3.3V logic |
| GND | GND | GND | Common ground |
| Power | +5V | VCC (optional) | If receiver needs power |

**RC Receiver Configuration**:
- 2-channel relay can be mapped to:
  - **Ch1**: Forward/Reverse throttle
  - **Ch2**: Left/Right steering
- Configure in Mission Planner: Initial Setup → Radio Calibration

---

### **STEP 5: Telemetry (Optional, for monitoring)**

If using telemetry (e.g., 433MHz radio module):

```
Telemetry Module:
├─ VCC ─────────────► 5V output
├─ GND ─────────────► GND
├─ TX ──────────────► Kakute H7 UART2 RX (Telem1)
└─ RX ──────────────► Kakute H7 UART2 TX
```

---

## Complete Pin Assignment Summary

### **Kakute H7 "Top-Down" View**

```
UART Assignments:
┌────────────────────────────────────────┐
│ UART1: GPS/Compass (MATEKSYS M10Q)     │
│ UART2: Telemetry (if available)        │
│ UART3: [Available]                     │
│ UART4: [Available]                     │
└────────────────────────────────────────┘

PWM Outputs (Motor Control):
┌────────────────────────────────────────┐
│ CH1: Left Motor PWM (Throttle)         │
│ CH2: Right Motor PWM (Steering)        │
│ CH3: [Auxiliary]                       │
│ CH4: [Auxiliary]                       │
└────────────────────────────────────────┘

Analog Inputs:
┌────────────────────────────────────────┐
│ Voltage Input: Battery monitoring      │
│ Current Input: Current monitoring      │
│ (From power module)                    │
└────────────────────────────────────────┘

RC Input:
┌────────────────────────────────────────┐
│ RC IN (PPM): RC Receiver signal        │
└────────────────────────────────────────┘
```

---

## Physical Layout Recommendation

```
FRONT OF ROVER (Direction of travel):

    ┌─────────────────────────────────┐
    │   GPS Antenna (M10Q-5883)        │  ← Mount high, away from motors
    │   (Pointing UP)                  │
    └─────────────────────────────────┘
            
    ┌─────────────────────────────────┐
    │     Kakute H7 Flight Controller  │
    │     (centered, protected)        │  ← Vibration dampening recommended
    │     + Power Module               │
    └─────────────────────────────────┘

    ┌──────────┐         ┌──────────┐
    │  Motor 1 │         │  Motor 2 │  ← Connected to motor controller
    │  (LEFT)  │  FRONT  │  (RIGHT) │
    └──────────┘         └──────────┘
    
    ┌──────────────┐  ┌──────────────────┐
    │Motor Control │  │ 36V Battery +    │  ← Battery in center/low
    │   (H-bridge) │  │ Distribution Box │     Power module here
    └──────────────┘  └──────────────────┘
    
           BACK

    ┌─────────────────────────────────┐
    │  Swivel Caster Wheel            │  ← Free-rolling front wheel
    │  (Roulette gonflable pivotante) │
    └─────────────────────────────────┘
```

---

## Wiring Checklist

- [ ] Battery connected with proper fusing (30-60A)
- [ ] Power module outputs 5.3V (verify with multimeter)
- [ ] Motor wires properly twisted and insulated from FC logic wires
- [ ] GPS module connected to UART1 with proper pin order
- [ ] RC receiver connected to RC IN with correct signal type
- [ ] All GND connections properly made (star ground configuration)
- [ ] No loose connectors or exposed wires
- [ ] Kakute H7 mounted with vibration damping (foam pads)
- [ ] GPS antenna clear of obstacles and high enough
- [ ] Motor rotation directions verified (both forward = straight)

---

## Testing Before Flight

### 1. **Power-up Test**
```
- Connect battery slowly via distribution block
- Verify power module outputs 5.3V
- Check Kakute H7 LED comes on solid (not blinking)
- No smoke or burning smell!
```

### 2. **Motor Direction Test**
```
- Disarm rover (if armed via failsafe)
- In Mission Planner: Motion Planning → Servo Output
- Command CH1 forward → Left motor should spin forward
- Command CH2 forward → Right motor should spin forward
  (if reversed, swap motor leads or set REVERSED parameter)
```

### 3. **RC Signal Test**
```
- Arm rover (via RC switch if configured)
- Move RC stick forward → Should see throttle increase
- Move RC stick left/right → Should see steering change
```

### 4. **GPS Lock Test**
```
- Boot rover outdoors with clear sky view
- Wait 30-60 seconds for GPS lock (LED should change color)
- In Mission Planner: Data → Status → GPS Lock indicator
- Should show latitude/longitude/altitude numbers
```

---

## Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| **FC won't boot** | No power to 5V | Check power module output |
| **Motors lock up** | Reversed motor direction | Swap motor phase wires |
| **No GPS signal** | Bad connection or placement | Check UART1 pins, move antenna higher |
| **RC not responding** | PPM signal inverted | Check receiver polarity |
| **Jerky movements** | PID tuning too aggressive | Reduce ATC_STR_RAT_P parameter |

---

## Next: Mission Planner Configuration

Once wiring is verified, proceed to:
1. Load firmware via Mission Planner
2. Calibrate compass
3. Load parameters
4. Perform radio calibration
5. Test in manual mode before autonomous

See: `ARDUROVER_PARAMETERS.md` for detailed parameter setup.
