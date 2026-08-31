# Celestron CG-5 OnStep

Documentation for a custom motorized conversion of a manual Celestron CG-5 German equatorial mount using OnStep.

## Overview

This repository documents the mechanical, electrical, and software setup of a manually operated Celestron CG-5 German equatorial mount converted to stepper motor control using the OnStepX open-source GOTO controller.

## Hardware

### Controller

| Component | Model | Notes |
|---|---|---|
| Mount | Celestron CG-5 | Manual German equatorial — non-motorized original |
| GOTO Controller | OnStepX | ESP32 WeMos D1 R32 (Espduino-32) |
| Motor Shield | Arduino CNC Shield V3 | DRV8825 drivers |
| Host computer | Raspberry Pi 4 | Freshly installed — Astroberry/KStars |

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
| **Driver** | DRV8825 |
| **Vref AR (Axis1 / socket Y)** | 0.702 V |
| **Vref DEC (Axis2 / socket A)** | 0.705 V |
| **Imax per driver** | ~1.41 A |
| **Microstepping** | TBD — depends on M0/M1/M2 jumpers |

### CNC Shield Socket Assignment

| CNC Shield socket | OnStep Axis | Motor |
|---|---|---|
| Y (fila trasera, derecha) | Axis1 | AR (Ascensión Recta) |
| A (fila delantera, derecha) | Axis2 | DEC (Declinación) |

### Mechanical (Pending confirmation)

| Parameter | Value |
|---|---|
| **Worm gear ratio AR** | 144:1 (CG-5 standard) |
| **Worm gear ratio DEC** | 144:1 (CG-5 standard) |
| **Motor pulley teeth** | TBD |
| **Axis pulley teeth** | TBD |
| **Belt type** | TBD |

## Firmware

| Parameter | Value |
|---|---|
| **OnStep version** | OnStepX (to be compiled) |
| **Config.h** | Pending — to be created from scratch |
| **Serial port** | TBD |
| **Baud rate** | TBD |

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
├── Config.h              # OnStepX configuration file (to be added)
└── notes.md              # Firmware notes and calibration parameters

images/
├── mount-overview.jpg    # Photos of the converted mount
├── controller-box.jpg    # OnStep controller box
└── wiring-diagram.png    # Wiring diagram

sessions/
└── commissioning-log.md  # Recovery, test and commissioning logs
```

## Status

🔧 Currently in commissioning phase — creating Config.h from scratch.

## References

- [OnStepX GitHub](https://github.com/hjd1964/OnStepX)
- [OnStep Groups.io Wiki](https://onstep.groups.io/g/main/wiki)
- [INDI OnStep driver](https://indilib.org)
- [StepperOnline 17HM19-2004S](https://www.stepperonline.com/nema-17-bipolar-0-9deg-46ncm-65-1oz-in-2a-42x42x48mm-4-wires-17hm19-2004s)
- [DRV8825 Datasheet](https://www.ti.com/lit/ds/symlink/drv8825.pdf)
