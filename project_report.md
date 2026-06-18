# ParkGrid — Project Report

## 1. Introduction

Urban parking congestion wastes driver time and increases unnecessary circling traffic. ParkGrid is an IoT-based smart parking system that detects per-slot occupancy in real time using ultrasonic sensors on an ESP32, and surfaces that data through a control-room-style live dashboard, with optional cloud access for off-network viewing.

This edition extends a basic slot-detection prototype into a more production-minded system: debounced sensor readings, a documented REST API, optional MQTT integration, NTP-synced timestamps, a health/diagnostics endpoint, and an optional cloud relay for remote access — alongside a deliberately designed dashboard rather than a generic status page.

## 2. Objectives

- Detect vehicle presence per slot reliably, filtering out sensor noise.
- Expose slot status over a documented JSON API.
- Provide a dashboard that's genuinely usable for monitoring a real lot, not just a demo.
- Support both local-network and remote (cloud-relayed) access.
- Make the system easy to demo without hardware, and easy to extend with more slots, MQTT, or analytics.

## 3. System Design

### 3.1 Hardware Architecture

Each slot has an HC-SR04 ultrasonic sensor mounted above it. The ESP32 polls every sensor on a fixed interval (default 1.5s), measuring distance to the nearest object. A slot is flagged occupied when distance falls below a configurable threshold.

### 3.2 Debouncing

Raw ultrasonic readings can be noisy — reflections, temperature drift, and edge-of-range readings can cause a slot's apparent state to flicker. ParkGrid requires `DEBOUNCE_SAMPLES` consecutive readings to agree before committing a state change, trading a small amount of latency for materially more stable readings.

### 3.3 Software Architecture

The firmware (`firmware/parkgrid_premium.ino`) is organized into:

1. **Sensor layer** — debounced distance readings per slot.
2. **Indicator layer** — LEDs and buzzer driven from committed slot states.
3. **Display layer** — optional OLED mirroring current status on-site.
4. **API layer** — `/status`, `/health`, `/config` JSON endpoints, with CORS enabled so a dashboard hosted elsewhere can call it directly.
5. **MQTT layer** (optional) — periodic publish of the same status payload to a broker, for integration with Home Assistant, Node-RED, or other automation tooling.

### 3.4 Dashboard

The dashboard (`web_dashboard/`) is deliberately styled as a control-room instrument rather than a generic admin panel: a dark palette derived from real parking-garage signage (amber/green/red), monospace data typography, an animated boom gate that opens when a slot becomes free, a live occupancy gauge, a per-slot grid, and a hand-drawn SVG history sparkline (no charting library dependency). A built-in demo mode generates simulated live traffic so the system can be shown convincingly with no hardware attached.

### 3.5 Cloud Relay

For access outside the ESP32's local network, an optional Node.js relay (`cloud/server.js`) accepts pushed status snapshots from the ESP32 (`POST /ingest`) and serves the latest snapshot publicly (`GET /status`), letting the same dashboard be pointed at a public URL instead of a local IP.

## 4. Data Flow

```
Sensor reading → debounce → committed slot state
   ├── LED / buzzer (immediate, local)
   ├── OLED display (immediate, local)
   ├── /status JSON (polled by local dashboard)
   ├── MQTT publish (optional, for automation)
   └── POST /ingest → cloud relay → /status (polled by remote dashboard)
```

## 5. Testing & Results

| Test Case | Expected Result | Observed Result |
|-----------|------------------|--------------------|
| Object placed under sensor at <10cm | Slot marked Occupied after `DEBOUNCE_SAMPLES` readings | ✅ Pass |
| Object removed | Slot marked Free after debounce window | ✅ Pass |
| Single noisy/false reading | State does not flip | ✅ Pass (debounce absorbs it) |
| All slots occupied | Buzzer sounds, MQTT alert published (if enabled) | ✅ Pass |
| Dashboard demo mode | Simulated slots update without hardware | ✅ Pass |
| Wi-Fi disconnect | Dashboard status pill shows Offline | ✅ Pass |
| Cloud relay round-trip | Status pushed by device appears via relay's `/status` | ✅ Pass |

## 6. Limitations

- Ultrasonic sensors remain sensitive to extreme weather (heavy rain, very high heat) and irregular vehicle shapes.
- The cloud relay is intentionally in-memory — a restart loses history; production deployments should add persistent storage.
- Per-slot LED/buzzer wiring scales linearly with slot count, which becomes a practical wiring challenge well beyond ~16–20 slots; larger deployments would likely move to a multiplexed I/O scheme or multiple ESP32 units per zone.

## 7. Future Work

See the Roadmap section in the main `README.md`: persistent analytics storage, ESP32-CAM license plate capture, servo-actuated physical gates, a native mobile app, multi-lot dashboards, and reservation/payment integration.

## 8. Conclusion

ParkGrid demonstrates that a low-cost ESP32 and ultrasonic sensors can support a genuinely usable, demo-ready, and extensible smart parking system — one with sensor-noise handling, a documented API, optional cloud reach, and a dashboard designed with the same care as the firmware behind it.
