# ParkGrid Cloud Relay 

This is a tiny optional relay server that lets the ParkGrid dashboard be viewed from **anywhere**, not just on the same Wi-Fi network as the ESP32.

## Why you'd want this

By default, the ESP32 serves `/status` only to devices on its local network. That's fine for an on-site display, but if you want to check your parking lot from your phone over mobile data, or share a live link with someone else, you need a relay that's reachable on the public internet.

## How it works

```
ESP32  --(POST /ingest every few seconds)-->  Cloud Relay  <--(GET /status)--  Dashboard (anywhere)
```

## Setup

1. Deploy this folder to any Node.js host (Render, Railway, Fly.io, a VPS, or even a Raspberry Pi with a reverse proxy).
2. Install dependencies and start:
   ```bash
   npm install
   npm start
   ```
3. (Optional) Set a shared secret so only your ESP32 can push data:
   ```bash
   export RELAY_SECRET="some-long-random-string"
   ```
4. On your ESP32, add a small HTTP POST to `/ingest` with the same JSON shape as `/status` (see `firmware/parkgrid_premium.ino`'s `buildStatusJson()`), including header `x-relay-secret` if you set one.
5. Point the dashboard's "Device address" field at your relay's public URL instead of the ESP32's local IP.

## Endpoints

| Method | Path       | Description                          |
|--------|------------|---------------------------------------|
| POST   | `/ingest`  | ESP32 pushes the latest slot snapshot |
| GET    | `/status`  | Dashboard reads the latest snapshot   |
| GET    | `/history` | Rolling buffer of recent snapshots    |

This relay is intentionally minimal — no database, just an in-memory snapshot plus a short rolling history. For long-term analytics, swap in a real database (e.g. SQLite, Postgres, or a time-series store).
