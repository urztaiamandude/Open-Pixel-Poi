# Firmware Build & Flash (PlatformIO)

## Links
- [User Manual](../user-manual.md)
- [Firmware Flashy Thing](https://mitchlol.github.io/#openpixelpoi)
- [Firmware source folder](../../../Firmware/open_pixel_poi_firmware_platformio/)

## Overview
This firmware is an Arduino project designed to be built in PlatformIO.

## Dependencies and board settings
PlatformIO should get all the correct dependancies automatically based on
`platformio.ini`, but in case it doesn't:
- It uses the neopixelbus library.
- It requires the ESP32 boards package; I reccomend v2.0.12 as newer versions
  cause LED glitches.
- When building, use the XIAO_ESP32C3 board so the GPIO config lines up. All
  the other board settings can be default.

## Compile and upload
1. Connect your ESP32 PCB to your computer via USB. On Windows it should show
   up as "USB Serial Device (COM#)" with the "#" being a random number.
2. Open the firmware project folder in VSCode with the PlatformIO plugin
   installed.
3. Hit the -> arrow button on the bottom bar to compile and upload the
   firmware to your PCB.

All the library dependencies and board config are contained in
`platformio.ini`.

## Note for self: Export a compiled firmware to the web-based flasher
1. Hit the -> arrow button on the bottom bar to compile and upload the firmware
   to your PCB.
2. Copy `.pio/build/seeed_xiao_esp32c3/firmware.bin` to the `opp_firmware`
   folder in the `mitchlol.github.io` project, replacing the old one.
3. Update the `manifest.json` in that same folder with the current date to
   have some minimal tracking.
