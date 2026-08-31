# Celestron CG-5 OnStep

Documentation for a custom motorized conversion of a manual Celestron CG-5 German equatorial mount using OnStep.

## Overview

This repository documents the mechanical, electrical, and software setup of a manually operated Celestron CG-5 German equatorial mount converted to stepper motor control using the OnStep open-source GOTO controller.

## Hardware

### Controller

| Component | Model | Notes |
|---|---|---|
| Mount | Celestron CG-5 | Manual German equatorial — non-motorized original |
| GOTO Controller | OnStep | ESP32 WeMos D1 R32 (Espduino-32) |
| Motor Shield | Arduino CNC Shield V3 | LV8729 drivers |
| Host computer | Raspberry Pi 4 | Astroberry |

### Stepper Motors

| Parameter | Value |
|---|---|
| **Manufacturer** | StepperOnline |
| **Part Number** | 17HM19-2004S |
| **Type** | NEMA 17 Bipolar |
| **Step angle** | **0.9° (400 steps/rev)** |
| **Rated current** | 2.0 A/phase |
| **Phase resistance** | 1.45 Ω |
| **Inductance** | 4.0 mH |
| **Holding torque** | 46 Ncm (65.1 oz·in) |
| **Body length** | 48 mm |
| **Shaft diameter** | 5 mm |
| **Leads** | 4 |

### Motor Drivers

| Parameter | Value |
|---|---|
| **Driver** | LV8729 (confirmed — chip marking: LV8729 1TN0) |
| **Vref RA (Axis1 / socket X)** | 0.702 V |
| **Vref DEC (Axis2 / socket Y)** | 0.705 V |
| **Imax per driver** | ~1.41 A (70% of rated 2.0A) |
| **Microstepping** | 32x (M0/M1/M2 jumpers) |

### CNC Shield Socket Assignment

Verified from `src/pinmaps/Pins.CNC3.h` (STEP/DIR GPIO mapping):

| CNC Shield socket | GPIO STEP | GPIO DIR | OnStep Axis | Motor |
|---|---|---|---|---|
| **X** | 26 | 16 | Axis1 | RA (Right Ascension) |
| **Y** | 25 | 27 | Axis2 | DEC (Declination) |
| Z | 19 | 14 | Axis3 | Not used |
| A | 17 | 14 | Axis4 | Not used |

> Only sockets **X** and **Y** are used for the CG-5 conversion.

### Mechanical

| Parameter | Value |
|---|---|
| **Worm gear ratio RA** | 144:1 (CG-5 standard) |
| **Worm gear ratio DEC** | 144:1 (CG-5 standard) |
| **Motor pulley teeth** | 20 |
| **Axis pulley teeth** | 60 |
| **Pulley ratio** | 3:1 |
| **Belt type** | GT2 (156-2GT) |

### Steps per Degree Calculation

```
steps/degree = motor_steps × microsteps × pulley_ratio × worm_ratio / 360
             = 400 × 32 × 3 × 144 / 360
             = 15360 steps/degree  ✅ (matches Config.h)
```

## Firmware

| Parameter | Value |
|---|---|
| **OnStep version** | 4.24m |
| **PINMAP** | CNC3 |
| **Baud rate** | 9600 |
| **Bluetooth** | ON (ESP32) — device name "OnStep" |
| **AXIS1_STEPS_PER_DEGREE** | 15360.0 |
| **AXIS2_STEPS_PER_DEGREE** | 15360.0 |
| **AXIS1_STEPS_PER_WORMROT** | 38400 |
| **Driver model** | LV8729 |
| **Microsteps** | 32 |
| **Config.h** | See `firmware/Config.h` |

### Build environment

| Component | Version |
|---|---|
| arduino-cli | — |
| esp32:esp32 platform | 2.0.17 |
| FQBN | `esp32:esp32:d1_mini32` |
| BluetoothSerial | 2.0.0 |
| Rtc by Makuna | 2.5.0 |
| Adafruit BME280 | 2.3.0 |
| Adafruit BusIO | 1.17.4 |
| Adafruit Unified Sensor | 1.1.15 |

### Known build issue: `tone()` duplicate default argument

With `esp32:esp32` 2.0.17, compilation fails with:

```
src/HAL/ESP32/Analog.h:54: error: default argument given for parameter 3 of 'void tone(...)' [-fpermissive]
```

**Fix** — remove the duplicate default argument from `Analog.h`:

```bash
python3 -c "
with open('/home/astroberry/OnStep/src/HAL/ESP32/Analog.h', 'r') as f:
    content = f.read()
content = content.replace(
    'void tone(uint8_t pin, unsigned int frequency, unsigned long duration = 0)',
    'void tone(uint8_t pin, unsigned int frequency, unsigned long duration)'
)
with open('/home/astroberry/OnStep/src/HAL/ESP32/Analog.h', 'w') as f:
    f.write(content)
print('Done')
"
```

Verify:
```bash
grep "weak.*tone" ~/OnStep/src/HAL/ESP32/Analog.h
# Must show the line WITHOUT '= 0'
```

### Compile & flash

```bash
# Compile
arduino-cli compile --fqbn esp32:esp32:d1_mini32 ~/OnStep

# Flash
arduino-cli upload --fqbn esp32:esp32:d1_mini32 --port /dev/ttyUSB0 ~/OnStep
```

Expected output:
```
Sketch uses 1226173 bytes (93%) of program storage space.
Global variables use 45428 bytes (13%) of dynamic memory.
Wrote 1232752 bytes (783950 compressed) at 0x00010000 in 11.3 seconds
Hash of data verified.
```

### Verify communication (LX200)

OnStep does not stream text — it responds to LX200 commands terminated with `#`.

```bash
echo -e ":GVP#" | socat - /dev/ttyUSB0,b9600,raw,echo=0,crnl
# Expected response: On-Step
```

Useful diagnostic commands:

| Command | Response |
|---|---|
| `:GVP#` | Product name (`On-Step`) |
| `:GU#` | System status flags |
| `:GC#` | Current date |
| `:GL#` | Local time |
| `:Gg#` | Longitude |
| `:Gt#` | Latitude |
| `:Te#` | Enable motors |
| `:RC2#:Mw#` | Move RA at 64x sidereal |
| `:RS#:Mw#` | Move RA at slew speed (west) |
| `:RS#:Mn#` | Move DEC at slew speed (north) |
| `:Q#` | Stop all |

`:GU#` status flags:

| Flag | Meaning |
|---|---|
| `N` | Motors enabled |
| `n` | Motors disabled |
| `H` | At home |
| `P` | Parking |
| `I` | Not initialized |
| `G` | GoTo in progress |

## Software

| Software | Role |
|---|---|
| Astroberry (Raspberry Pi OS) | Host OS |
| KStars + Ekos | Planetarium and imaging sequencer |
| INDI OnStep (`indi_lx200_OnStep`) | Mount driver — port `/dev/ttyUSB0`, 9600 baud |

## Repository Structure

```
docs/
├── mount.md              # Mechanical specs, gear ratios, worm wheel
├── mechanics.md          # Motor specs, driver configuration
├── wiring.md             # Wiring diagram and pin mapping
├── astroberry-indi.md    # RPi, INDI and KStars/Ekos setup
└── commands.md           # Useful OnStep LX200 diagnostic commands

firmware/
├── Config.h              # OnStep configuration file
└── notes.md              # Firmware notes and calibration parameters

images/
├── mount-overview.jpg    # Photos of the converted mount
├── controller-box.jpg    # OnStep controller box
└── wiring-diagram.png    # Wiring diagram

sessions/
└── commissioning-log.md  # Recovery, test and commissioning logs
```

## Status

✅ Firmware compiled and flashed (2026-08-31) — OnStep responding on `/dev/ttyUSB0` at 9600 baud.  
✅ Both axes (RA + DEC) moving correctly at all speeds (2026-08-31).  
🔧 INDI commissioning and star alignment pending.

## References

- [OnStep GitHub](https://github.com/hjd1964/OnStep)
- [OnStep Groups.io Wiki](https://onstep.groups.io/g/main/wiki)
- [INDI OnStep driver](https://indilib.org)
- [StepperOnline 17HM19-2004S](https://www.stepperonline.com/nema-17-bipolar-0-9deg-46ncm-65-1oz-in-2a-42x42x48mm-4-wires-17hm19-2004s)
- [LV8729 Datasheet](https://www.lcsc.com/datasheet/lcsc_datasheet_2401121536_LVCHIP-Micro-LV8729_C2890789.pdf)
