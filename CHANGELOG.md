# Changelog — VIGO fork

## v2.3.8-vigo.1 (2026-04-24)

### Fixed
- ESP32-S3 support for GPIO > 33 (issue #131 and #141 upstream).
  OneWire_direct_gpio.h now uses the correct GPIO register pair
  (GPIO.in / GPIO.in1.val) based on pin number, enabling DS18B20
  sensors on GPIO 34-48 of ESP32-S3.

### Context
- Forked from upstream v2.3.8.
- Upstream issues #131 (2023) and #141 (2024) unresolved at time of
  fork. Maintained by Denk Vooruit BV for VIGO & Wilco platform.
