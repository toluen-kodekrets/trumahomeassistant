# trumahomeassistant
Adding the Truma with iNet X to Home Assistant

Readme for the Home Assistant addon:
Absolutely — here is a clean, well-structured **README.md** suitable for a public GitHub repository for your Home Assistant Trumanita add-on.

You can copy/paste this directly into `README.md`.
If you want badges, screenshots, installation GIFs, or extra sections, just tell me.

---

# Trumanita – Home Assistant Add-on

### Control your Truma Combi heater via LIN bus & MQTT

This repository contains a Home Assistant add-on that packages the excellent **Trumanita** project and exposes it natively inside Home Assistant.
It allows you to monitor and control a **Truma Combi** (e.g. Combi 4/6) via the LIN bus using a UART interface.

The add-on automatically publishes sensor data over MQTT, and receives setpoints and control commands from Home Assistant through MQTT topics.

---

## ✨ Features

✅ Reads live data from Truma Combi via LIN bus
✅ Publishes temperature/water/heater status to MQTT
✅ Receives control messages from Home Assistant (HVAC / boiler / fan / power modes)
✅ Fully configurable via add-on UI (no manual `.env` editing)
✅ Uses Debian base image for full binary compatibility
✅ Auto-generates `.env` on startup
✅ Includes custom logo for Home Assistant Add-on Store
✅ Supports all Home Assistant hardware types (Pi, ODROID, x86, etc.)

---

## 🧱 Architecture Overview

```
Home Assistant Add-on
        │
        ├── generates /config/.env
        ├── copies .env → /opt/trumanita/.env
        ├── runs LINsenddirect (from Trumanita)
        │
        └── MQTT topics:
                truma/control/...  (incoming commands)
                truma/report/...   (sensor readings)
```

The add-on communicates with a UART-to-LIN adapter connected to the Truma heater.

---

## 📦 Installation

1. Copy the `trumanita` folder (this repository) into:

```
/addons/
```

on your Home Assistant system (via Samba, SSH, or File Editor).

2. Restart Home Assistant (or just the Supervisor).

3. Go to:

```
Settings → Add-ons → Add-on Store → Local Add-ons
```

4. Install **Trumanita**.

5. Open the add-on **Configuration** tab and set:

* MQTT credentials
* UART device path (e.g. `/dev/ttyUSB0` or `/dev/serial/by-id/...`)
* Topics
* Publish interval
* Baudrate
* Sleep timing

6. Start the add-on and check the logs.

---

## ⚙️ Configuration Options

All configuration is done via the Add-on UI.

| Field                             | Description                                       |
| --------------------------------- | ------------------------------------------------- |
| `mqtt_host`                       | MQTT broker address (default: `core-mosquitto`)   |
| `mqtt_port`                       | MQTT port (default: 1883)                         |
| `mqtt_username` / `mqtt_password` | MQTT credentials                                  |
| `mqtt_topic`                      | Control topic (default: `truma/control`)          |
| `mqtt_topic_report`               | Report topic (default: `truma/report`)            |
| `sleep_duration_us`               | LIN loop delay in microseconds                    |
| `mqtt_publish_interval`           | How often to publish readings                     |
| `uart_device`                     | UART device (e.g. `/dev/ttyAMA0`, `/dev/ttyUSB0`) |
| `uart_baudrate`                   | UART baudrate (default: 9600)                     |

The add-on generates:

```
/config/.env
```

and copies it into:

```
/opt/trumanita/.env
```

which Trumanita reads directly.

---

## 🔌 UART Device Notes

You can find the correct UART path in Home Assistant:

```
Settings → System → Hardware → ... → All Hardware
```

Common values:

* `/dev/ttyAMA0`
* `/dev/ttyUSB0`
* `/dev/ttyACM0`
* `/dev/serial/by-id/...` (recommended)

If the wrong device is set, you will see:

```
[ERROR] Failed to open UART device: No such file or directory
```

---

## 🧪 MQTT Topics

### Publish (sensor data)

```
truma/report/room_temp
truma/report/water_temp
truma/report/heating_state
truma/report/boiler_mode
truma/report/error_state
truma/report/voltage
```

### Subscribe (control config)

```
truma/control/truma_config
```

Trumanita expects a full JSON payload representing the desired heater/boiler/fan/power configuration.

---

## 🛠️ Files in the Add-on

```
trumanita/
 ├── Dockerfile
 ├── run.sh
 ├── config.yaml
 ├── icon.png
 └── logo.png
```

### Dockerfile

Uses Debian base image for compatibility:

```
FROM ghcr.io/home-assistant/aarch64-base-debian:bookworm
```

### run.sh

Generates `.env` dynamically from add-on options, copies it, and starts Trumanita.

---

## 🖼️ Branding

The add-on includes:

* `icon.png` — shown in the Add-on Store list
* `logo.png` — shown on the add-on detail page

These can be replaced with your own artwork if desired.

---

## 👍 Credits

* **Trumanita** original project:
  [https://github.com/vincentgloss/trumanita](https://github.com/vincentgloss/trumanita)
* This add-on packaging, UI, and integration work was created to make Trumanita easy to install and use inside Home Assistant.

---
