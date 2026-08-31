# Celestron CG-5 OnStep

Documentation for a custom motorized conversion of a manual Celestron CG-5 German equatorial mount using OnStep.

## Overview

This repository documents the mechanical, electrical, and software setup of a manually operated Celestron CG-5 German equatorial mount converted to stepper motor control with the OnStep open-source GOTO controller.

## Hardware

| Component | Model | Notes |
|---|---|---|
| Mount | Celestron CG-5 | Manual German equatorial — non-motorized original |
| GOTO Controller | OnStep (custom build) | Arduino Mega 2560 |
| Host computer | Raspberry Pi 4 | Running Astroberry OS |

## Firmware

| Parameter | Value |
|---|---|
| **OnStep version** | 4.24m |
| **Build date** | Nov 22 2021 |
| **Serial port** | `/dev/ttyUSB0` |
| **Baud rate** | 9600 |
| **Protocol** | LX200-compatible |

## Software

| Software | Role |
|---|---|
| Astroberry | Raspberry Pi OS for astronomy |
| KStars + Ekos | Planetarium and imaging sequencer |
| INDI LX200 OnStep | Mount driver |

## Repository Structure

```
docs/
├── mount.md              # Mechanical specs, gear ratios, worm wheel
├── mechanics.md          # Motor specs, driver configuration
├── wiring.md             # Wiring diagram and pin mapping
├── astroberry-indi.md    # Astroberry, INDI and KStars/Ekos setup
└── commands.md           # Useful OnStep LX200 diagnostic commands

firmware/
├── Config.h              # OnStep configuration file (to be added)
└── notes.md              # Firmware notes and calibration parameters

images/
├── mount-overview.jpg    # Photos of the converted mount
├── controller-box.jpg    # OnStep controller box
└── wiring-diagram.png    # Wiring diagram

sessions/
└── commissioning-log.md  # Recovery, test and commissioning logs
```

## Status

🔧 Currently in commissioning phase — recovering setup after a pause.

## References

- [OnStep GitHub](https://github.com/hjd1964/OnStep)
- [OnStep Groups.io Wiki](https://onstep.groups.io/g/main/wiki)
- [INDI LX200 OnStep driver](https://indilib.org)
- [Astroberry](https://www.astroberry.io)
