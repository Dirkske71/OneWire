# OneWire — VIGO fork

Fork of [PaulStoffregen/OneWire](https://github.com/PaulStoffregen/OneWire) v2.3.8, maintained by Denk Vooruit BV for the VIGO and Wilco platforms.

## Why this fork exists

Upstream v2.3.8 ships with `util/OneWire_direct_gpio.h` logic that restricts ESP32 GPIO register access with a `pin < 46` upper bound (and a partially-commented `pin <= 33` guard) in the `ARDUINO_ARCH_ESP32` branch. That assumption was correct for classic ESP32 (GPIO 34–39 are input-only) but is wrong for **ESP32-S3**, where GPIO 34–48 are fully bidirectional.

The result: DS18B20 sensors on GPIO > 33 silently fail to read/write on ESP32-S3 boards. We ran into this on the Wilco compressor-satellite hardware (GPIO 46 for `T_COND_WATER_UIT`).

Upstream tracking:
- [Issue #131 — ESP32-S3 GPIO > 33 broken (Sep 2023)](https://github.com/PaulStoffregen/OneWire/issues/131)
- [Issue #141 — KaoruTK's proposed fix (May 2024)](https://github.com/PaulStoffregen/OneWire/issues/141)

Both issues have been unanswered by upstream for 1–2.5 years, hence this fork.

## What this fork changes

Only `util/OneWire_direct_gpio.h`, only inside the `#elif defined(ARDUINO_ARCH_ESP32)` branch, only the plain-ESP32/S2/S3 sub-branch (not ESP32-C3/C6, not the IDF 3.x `rtc_gpio_desc` path):

- `directRead` / `directWriteLow` / `directWriteHigh`: the `else if ( pin < 46 )` upper-bound is dropped. The `else` branch now reaches `GPIO.in1.val` / `GPIO.out1_w1tc.val` / `GPIO.out1_w1ts.val` for every pin ≥ 32, so GPIO 34–48 work on ESP32-S3.
- `directModeOutput`: stale `pin <= 33` comments removed; the guard was already commented-out upstream but left confusing residue.

No other files touched. Library name stays `OneWire` so existing PlatformIO dependencies keep matching.

## Install via PlatformIO

In your project's `platformio.ini`:

```ini
lib_deps =
  https://github.com/Dirkske71/OneWire.git#v2.3.8-vigo.1
```

Pin the tag (`#v2.3.8-vigo.1`) so builds are reproducible.

## Quick test

On an ESP32-S3 board with a DS18B20 wired to a high GPIO (e.g. GPIO 46) with a 4.7 kΩ pull-up to 3V3:

```cpp
#include <OneWire.h>
#include <DallasTemperature.h>

OneWire oneWire(46);                // previously broken on upstream v2.3.8
DallasTemperature sensors(&oneWire);

void setup() {
  Serial.begin(115200);
  sensors.begin();
}

void loop() {
  sensors.requestTemperatures();
  Serial.println(sensors.getTempCByIndex(0));
  delay(1000);
}
```

Upstream v2.3.8: reads `-127.00` (device-disconnected error).
This fork: reads the actual temperature.

## Upstreaming

If upstream ever merges a compatible fix, this fork will be retired. Until then, track tags `v2.3.8-vigo.*` for fork-specific patches.
