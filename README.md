# Celestron CG-5 with OnStep

Documented configuration for a Celestron CG-5 German equatorial mount used for solar observation and imaging with a Lunt LS60MT Dual Filter telescope.

## Confirmed configuration

- Mount: Celestron CG-5 German equatorial mount (GEM)
- Mount firmware: OnStep 4.24s
- Controller: WeMos D1 R32 (ESP32)
- Motor shield: CNC Shield V3
- Right ascension and declination drivers: LV8729
- Active network interface: ESP32 integrated Wi-Fi
- Current Wi-Fi SSID: `ONSTEP`
- Network operating mode: Station mode
- Integrated web interface: Smart Web Server 1.0a
- Observed web interface latency: present but below 1 second

## Solar imaging setup

- Solar telescope: Lunt LS60MT Dual Filter
- Observing band: H-alpha
- Etalon configuration: dual H-alpha etalon
- Blocking filter: B1200
- Main camera: ZWO ASI174MM
- Guiding: not used

This mount configuration is dedicated to the solar setup above. It is not associated with the separate night-imaging C8 setup.

## Wi-Fi architecture

The operational network interface is the integrated ESP32 Wi-Fi running with OnStep and advertising the `ONSTEP` SSID. It is configured in Station mode so the mount joins the local observatory or home network.

An ESP-01/ESP8266 running SmartWebServer 2.10h may also advertise `OnStep-SWS`. It is not part of the currently selected network path. Do not use both web servers concurrently as mount-control channels.

## Firmware configuration

The OnStep build configuration is stored in [firmware/Config.h](firmware/Config.h). Before changing firmware or attempting an OnStepX migration, preserve a known-working copy of this file and bench-test RA/DEC motion, axis direction, travel limits, tracking, and network connectivity.

## Operating notes

- The integrated web interface has sub-second latency and is suitable for general mount control.
- Use one mount-control interface at a time during testing and operation.
- Station mode requires locating the IP address assigned by the router; a DHCP reservation is recommended for a stable address.
- Do not use the web interface as a substitute for precision guiding; this setup does not use guiding.
