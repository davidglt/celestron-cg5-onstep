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
| Host computer | Raspberry Pi 4 | Freshly installed |

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
| **Vref AR (Axis1 / socket Y)** | 0.702 V |
| **Vref DEC (Axis2 / socket A)** | 0.705 V |
| **Imax per driver** | ~1.41 A (70% of rated 2.0A) |
| **Microstepping** | 32x (M0/M1/M2 jumpers) |

### CNC Shield Socket Assignment

| CNC Shield socket | OnStep Axis | Motor |
|---|---|---|
| Y (fila trasera, derecha) | Axis1 | AR (Ascensión Recta) |
| A (fila delantera, derecha) | Axis2 | DEC (Declinación) |

### Mechanical

| Parameter | Value |
|---|---|
| **Worm gear ratio AR** | 144:1 (CG-5 standard) |
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
| **OnStep version** | 4.24m (backup Config.h) |
| **PINMAP** | CNC3 |
| **Baud rate** | 9600 |
| **Bluetooth** | ON (ESP32) — device name "OnStep" |
| **AXIS1_STEPS_PER_DEGREE** | 15360.0 |
| **AXIS2_STEPS_PER_DEGREE** | 15360.0 |
| **AXIS1_STEPS_PER_WORMROT** | 38400 |
| **Driver model** | LV8729 |
| **Microsteps** | 32 |
| **Config.h** | See `firmware/Config.h` (to be moved) |

## Software

| Software | Role |
|---|---|
| Raspberry Pi OS | Host OS (freshly installed) |
| KStars + Ekos | Planetarium and imaging sequencer |
| INDI OnStep | Mount driver |

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

🔧 Currently in commissioning phase — firmware to be compiled and flashed.

## References

- [OnStep GitHub](https://github.com/hjd1964/OnStep)
- [OnStep Groups.io Wiki](https://onstep.groups.io/g/main/wiki)
- [INDI OnStep driver](https://indilib.org)
- [StepperOnline 17HM19-2004S](https://www.stepperonline.com/nema-17-bipolar-0-9deg-46ncm-65-1oz-in-2a-42x42x48mm-4-wires-17hm19-2004s)
- [LV8729 Datasheet](https://www.lcsc.com/datasheet/lcsc_datasheet_2401121536_LVCHIP-Micro-LV8729_C2890789.pdf)
