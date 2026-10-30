# Cyber Clock: ESPHome firmware for a 64x64 HUB75 clock

Custom [ESPHome](https://esphome.io) firmware for the cheap ESP32 "Cyber Clock" 64x64 LED matrix driver
(an ESP32 DevKit on a "sunnydodo.top" adapter board) that ships with a Chinese-only web UI.

- Big stacked clock (20x28 px digits) over Matrix-style digital rain
- Automatic US daylight saving (real tz database, `America/Los_Angeles` by default; change `timezone:` in the YAML)
- Pencreus sunstone-eye boot splash
- Home Assistant controls: display mode, rain colour/speed/density/trail, clock colour, 24h/12h, brightness
- Auto-brightness from the on-board light sensor (with calibration numbers), button cycles display mode
- OTA updates

## Hardware (verified on a real unit)

ESP32-D0WD-V3, 4 MB flash, no PSRAM, CP2102 USB-UART, 64x64 1/32-scan HUB75 panel.

| signal | GPIO | | signal | GPIO |
|---|---|---|---|---|
| R1 | 25 | | C | 5 |
| G1 | 26 | | D | 17 |
| B1 | 27 | | **E** | **18** |
| R2 | 14 | | LAT | 4 |
| G2 | 12 | | OE | 15 |
| B2 | 13 | | CLK | 16 |
| A | 23 | | | |
| B | 19 | | | |

| function | GPIO |
|---|---|
| Button (pull-up, active low) | 32 |
| DHT11 header (not fitted on my board) | 33 |
| Light sensor (ADC1_CH7) | 35 |

Panel settings that work: `shift_driver: GENERIC`, `clock_speed: 10MHZ` (the ESP32 I2S maximum),
`bit_depth: 6` (keeps the DMA buffer ~24 KB so it fits without PSRAM), and **`double_buffer: true`**
(without it, redrawing every frame makes the whole panel pulse).

## Install

```bash
cd esphome
cp secrets.yaml.example secrets.yaml      # then fill it in
esphome run cyber-clock.yaml --device /dev/cu.usbserial-XXXX   # first flash over USB
esphome upload cyber-clock.yaml --device cyber-clock.local     # later updates over WiFi
```

Back up the stock firmware first if you want a way home:
`esptool read-flash 0 0x400000 backup.bin` at **230400 baud** (460800 fails on this CP2102).

## Notes

- `api: reboot_timeout: 0s` is deliberate. The ESPHome default (15 min) reboots the clock whenever no
  Home Assistant/API client is connected.
- Auto-brightness is off by default. Calibrate it from the "Light Level" sensor: higher voltage = brighter.
  On my unit a dark room reads ~0.27 V and a phone torch ~1.5 V; set "Light Dark Voltage" / "Light Bright Voltage"
  and the Auto Min/Max brightness numbers, then turn on "Auto Brightness".
- `cyber-clock-bringup.yaml` is the test-pattern firmware (colour bars, row/column scan, rainbow) used to prove the wiring.
- The board also has unused mic pads and an MPU6050 header; this firmware does not use them.
- Pins were recovered by reading the stock firmware, then confirmed on hardware.

MIT licensed.
