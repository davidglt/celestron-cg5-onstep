# OnStep 4.24s to OnStepX Migration Guide

This guide describes a cautious, reversible migration path for the Celestron CG-5 configuration documented in this repository. The goal is to test OnStepX without losing the currently working OnStep 4.24s configuration.

## Scope

Current known hardware:

- Mount: Celestron CG-5 German equatorial mount (GEM)
- Controller: WeMos D1 R32 (ESP32)
- Motor shield: CNC Shield V3
- RA and DEC drivers: LV8729
- Current mount firmware: OnStep 4.24s
- Operational network path: integrated ESP32 Wi-Fi in Station mode
- Solar payload: Lunt LS60MT Dual Filter, dual H-alpha etalon, B1200 blocking filter, ZWO ASI174MM
- Guiding: not used

This document is a migration and test plan. It does not claim that an OnStep 4.24 configuration file can be copied unchanged into OnStepX.

## Safety principles

1. Keep the working OnStep 4.24s firmware and configuration recoverable before changing anything.
2. Do not test a newly flashed controller with the telescope near a pier, tripod, or hard mechanical stop.
3. Start at the slowest manual slew rate with the payload removed or secured.
4. Verify axis direction before enabling tracking, GoTo, homing, or parking.
5. Do not use the integrated ESP32 web interface and the external ESP-01/ESP8266 web server as simultaneous mount-control paths.
6. Treat every change to pin mapping, driver mode, microstepping, or steps-per-degree as unvalidated until tested.

## Phase 1: Preserve the working baseline

Before installing OnStepX, record and preserve the current state.

1. Keep the current repository commit and retain a local copy of `firmware/Config.h`.
2. Record the current OnStep version shown by the web interface: `4.24s`.
3. Save screenshots or notes for mount settings, date/time source, park position, home position, and network settings. Do not commit sensitive values.
4. Verify the current installation can connect, report status, and move RA and DEC correctly.
5. Record the exact behavior of the integrated Wi-Fi interface, including the observed sub-second web latency.
6. Keep a known-good firmware binary if one is available from the successful 4.24s build.

### Baseline test checklist

- [ ] Controller boots normally.
- [ ] Station mode connects to the intended local network.
- [ ] Web interface responds with latency below one second.
- [ ] RA manual motion is correct.
- [ ] DEC manual motion is correct.
- [ ] Tracking can be enabled and disabled.
- [ ] Mechanical limits are known and clear.
- [ ] Recovery path to OnStep 4.24s is available.

## Phase 2: Prepare an isolated OnStepX build

1. Download an official OnStepX release or clone the official OnStepX repository into a separate working directory.
2. Do not overwrite the OnStep 4.24 source tree or `firmware/Config.h`.
3. Use an ESP32-compatible board definition matching the WeMos D1 R32.
4. Start from the OnStepX default configuration and select the closest supported ESP32 pin map.
5. Create a dedicated OnStepX configuration file and document every deviation from the defaults.
6. Configure the mount explicitly as a German equatorial mount.
7. Configure both axes for the LV8729 Step/Dir driver arrangement only after checking the current OnStepX driver definitions.

## Phase 3: Translate settings deliberately

Transfer values one by one from the known-working OnStep 4.24 configuration. Do not assume option names or semantics are identical.

| Category | Verification required |
| --- | --- |
| Board and pin map | Confirm all STEP, DIR, ENABLE, limit, serial, and accessory pin assignments for the WeMos D1 R32 plus CNC Shield V3 wiring |
| Mount type | Explicitly configure GEM behavior |
| Motor drivers | Confirm LV8729 compatibility, mode, and Step/Dir output configuration in OnStepX |
| Microstepping | Match the physical MS-pin wiring and driver settings |
| Steps per degree | Copy only after checking units and formulas used by OnStepX |
| Motor direction | Verify independently for RA and DEC at slow speed |
| Slew, acceleration, and current | Begin conservatively; increase only after stable bench tests |
| Limits, home, and park | Re-enter and validate all positions; do not assume stored settings carry over |
| Time and network | Reconfigure after first boot without committing sensitive values |

## Phase 4: First flash and bench test

1. Power down the mount and disconnect the telescope payload or place it in a mechanically safe position.
2. Upload the OnStepX build to the ESP32.
3. Power-cycle the controller and inspect the boot behavior.
4. Connect using one interface only. USB/serial is preferred for first communication tests; integrated Wi-Fi may be configured after the controller responds reliably.
5. Confirm the controller reports the expected OnStepX version.
6. At the lowest manual slew rate, test RA for a brief movement. Stop immediately if the direction, speed, or sound is unexpected.
7. Test DEC in the same manner.
8. Verify that stopping motion works immediately.
9. Test tracking only after RA direction and basic motion are confirmed.

### Minimum acceptance criteria

- [ ] Controller boots reliably after multiple power cycles.
- [ ] RA and DEC respond independently.
- [ ] Axis directions are correct.
- [ ] Motors do not stall, overheat, or emit abnormal noise.
- [ ] Stop commands work reliably.
- [ ] Tracking direction is correct.
- [ ] No collision with a mechanical stop occurs.

## Phase 5: Network setup

After successful motion tests, configure the ESP32 network interface.

1. Start with one network path only.
2. Configure Station mode using local network credentials that are not stored in this repository.
3. Use DHCP initially and identify the assigned IP address locally.
4. Create a DHCP reservation locally if a stable address is needed.
5. Confirm web access and command-channel connectivity from the intended control computer.
6. Measure latency with the controller and client on the same local network.
7. Leave the ESP-01/ESP8266 SmartWebServer disconnected or unused while testing the integrated ESP32 network path.

## Phase 6: Mount functions

Validate advanced functions in this order:

1. Date, time, and time source.
2. Tracking rate selection, including solar tracking if used.
3. Manual motion at low, medium, and maximum approved rates.
4. Meridian and travel-limit behavior.
5. Home position, if hardware and configuration support it.
6. Park position and recovery from Park.
7. Alignment and GoTo using conservative targets well away from limits.
8. Reboot behavior and persistence of verified settings.

For the Lunt LS60MT solar setup, perform initial pointing and tracking tests in daylight only with all solar-observing safety procedures in place. The mount migration does not alter the need for correctly installed solar optical components.

## Rollback to OnStep 4.24s

If any test fails or behavior is uncertain:

1. Stop motion and power down the controller.
2. Restore the known-good OnStep 4.24s firmware.
3. Restore the preserved `firmware/Config.h` configuration.
4. Reboot and repeat the baseline RA/DEC motion checks.
5. Record the failed OnStepX configuration change before attempting another migration build.

Do not continue OnStepX testing if the mount cannot be stopped reliably, an axis direction is uncertain, or a mechanical limit cannot be protected.

## Migration record

Record each test build in this table.

| Date | OnStepX version/commit | Configuration change | RA result | DEC result | Tracking | Network | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  |  |
