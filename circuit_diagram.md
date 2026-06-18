# Circuit Diagram & Wiring Guide — ParkGrid Premium

## Overview

This guide covers wiring an 8-slot ParkGrid installation using an ESP32 and HC-SR04 ultrasonic sensors. The pattern is identical for fewer or more slots — just repeat the per-slot wiring and update `NUM_SLOTS` plus the pin arrays in `config.h`.

## Components per Slot

- 1× HC-SR04 Ultrasonic Sensor
- 1× Red LED + 1× Green LED (optional, occupied/available indicator)
- 2× 220Ω resistors (for LEDs, if used)
- 1× 1kΩ resistor + 1× 2kΩ resistor (voltage divider for ECHO pin — required)

## Shared Components

- 1× Buzzer (optional)
- 1× OLED Display, I2C, SSD1306 0.96" (optional but recommended)
- ESP32 Dev Board

## Wiring Table (Slot 1 example)

| Component    | Pin       | ESP32 Pin                      |
|--------------|-----------|----------------------------------|
| HC-SR04 #1   | VCC       | 5V                                |
| HC-SR04 #1   | GND       | GND                               |
| HC-SR04 #1   | TRIG      | GPIO 5                            |
| HC-SR04 #1   | ECHO      | GPIO 18 (via voltage divider)     |
| Red LED #1   | Anode (+) | (optional, set in config.h)       |
| Green LED #1 | Anode (+) | (optional, set in config.h)       |
| LEDs         | Cathode (-) | GND                              |

Default pin assignments for all 8 slots (from `config.h`):

```
trigPins = {5, 17, 16, 4, 2, 15, 0, 27}
echoPins = {18, 19, 21, 22, 23, 32, 33, 34}
```

> Note: GPIO 0 and 2 are strapping pins on most ESP32 boards — avoid connecting anything that pulls them low/high during boot if you run into upload issues, or remap them in `config.h`.

## Voltage Divider for ECHO Pin (required)

The HC-SR04's ECHO pin outputs 5V, but ESP32 GPIOs are rated for 3.3V. Use a simple resistive divider for every sensor's ECHO line:

```
ECHO (5V) ---[1kΩ]---+---[2kΩ]--- GND
                      |
                 ESP32 GPIO (≈3.3V)
```

## OLED Display (I2C, optional)

| OLED Pin | ESP32 Pin |
|----------|-----------|
| VCC      | 3.3V      |
| GND      | GND       |
| SDA      | GPIO 21*  |
| SCL      | GPIO 22*  |

\* If GPIO 21/22 are already used for ultrasonic sensors in your configuration, remap either the OLED or the conflicting sensor pins in `config.h`.

## Buzzer (optional)

| Buzzer Pin | ESP32 Pin           |
|------------|----------------------|
| +          | any free GPIO (set `BUZZER_PIN`) |
| -          | GND                  |

## Power Notes

- ESP32 can run off USB (5V) during development.
- For a permanent installation with 8 ultrasonic sensors plus LEDs, use a dedicated 5V/2A+ supply — USB power alone may brown out under load.
- Run a common ground rail across all sensors, LEDs, and the ESP32.
- Keep sensor wiring runs under ~3m where possible; longer runs increase noise on the ECHO line and may require shielded cable.

## Mounting Guidance

Mount each HC-SR04 facing downward above its slot (on a beam, pole, or canopy), at a height where a typical vehicle roof sits comfortably within the configured `SLOT_THRESHOLD_CM` once parked. Test and recalibrate the threshold after mounting — vehicle heights vary, and a threshold tuned for sedans may misread trucks or SUVs.
