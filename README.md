# trumahomeassistant
Adding the Truma with iNet X to Home Assistant

Readme for the Home Assistant addon:


  [Wiring](https://github.com/danielfett/inetbox.py)


Issues:
When using iNet X panel, You first must have the iNet X panel connected at the same time as the raspberri pi, turn on ventilation mode on the iNet X panel, then unplug the panel and then it can be controlled by this setup with the raspberri pi.
Future plans:
Make this auto discovrable by the MQTT add on in Home Assistant
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


This method is useful when:

* Home Assistant OS prevents you from editing `/boot/config.txt` directly
* You cannot use `ha os shell`
* You want to fix UART before booting
* You want a clean and guaranteed method

---

# Enabling UART on Raspberry Pi 5 Using the SD Card

### Works for Home Assistant OS (supervised / appliance image)

This guide explains how to enable the hardware UART for use with the **Trumanita** add-on *by editing the SD card directly on another computer*.

This is currently the **most reliable** method for Raspberry Pi 5 with Home Assistant OS.

---

# 🧰 What You Need

* Your Raspberry Pi 5 SD card
* A computer with an SD card reader
* A text editor (Notepad++, VSCode, nano, anything)

---

# 📌 Step 1 — Shut down Home Assistant

From the UI:

```
Settings → System → Hardware → Power → Shutdown
```

Or via Terminal:

```bash
ha host shutdown
```

Wait until LEDs stop blinking.

Remove the SD card.

---

# 📌 Step 2 — Insert the SD card into your computer

Your PC will show a partition named:

```
hassos-boot
```

This is the Pi boot partition.

Inside, you will find:

* `config.txt`
* `firmware`
* `cmdline.txt`
* `overlays/`

---

# 📌 Step 3 — Edit `config.txt`

Open:

```
config.txt
```

Add the following lines **at the very bottom** (or anywhere under `[all]`):

```txt
# Enable UART for Trumanita / LIN interface
enable_uart=1
dtoverlay=uart0
```

### Why?

* `enable_uart=1`
  enables the PL011 UART in the Pi 5 SoC

* `dtoverlay=uart0`
  exposes it on the GPIO header as `/dev/ttyAMA0`

This ensures the right hardware UART is active *before* Home Assistant boots.

---

# 📌 Step 4 — Ensure serial console is disabled (required)

Still in the `hassos-boot` partition, open:

```
cmdline.txt
```

Remove any of these if present:

```
console=ttyAMA0,115200
console=serial0,115200
```

The result should be **one single line**.
Do **NOT** add line breaks.

Example final line:

```txt
root=LABEL=hassos-data rd.systemd.unit=haos-init.service
```

---

# 📌 Step 5 — Save and eject the SD card

Make sure to:

* Save `config.txt`
* Save `cmdline.txt`
* Properly eject the SD card to avoid corruption

---

# 📌 Step 6 — Reinsert SD card & boot Pi 5

Insert the card back into your Raspberry Pi 5 and power it on.

Wait for Home Assistant to boot.

---

# 📌 Step 7 — Verify UART is working

Open:

```
Settings → System → Hardware → All Hardware
```

You should now see devices like:

```
/dev/ttyAMA0
/dev/ttyAMA10
/dev/ttyS0
```

The correct one for GPIO UART is usually:

```
/dev/ttyAMA0
```

---

# 📌 Step 8 — Configure Trumanita Add-on

In the add-on configuration:

```yaml
uart_device: /dev/ttyAMA0
uart_baudrate: 9600
```

Restart the add-on.

Check logs for:

```
[CONFIG] UART Device: /dev/ttyAMA0
[CONFIG] UART Baudrate: 9600
```

If you see:

```
[ERROR] Failed to open UART device
```

it usually means:

* Wiring issue
* Wrong UART port
* Adapter not detected

But the Pi UART is now correctly enabled.

---

# ✔ UART Enabled Successfully

You have now enabled the UART interface using the SD card — the safest and most reliable method for Raspberry Pi 5 + Home Assistant OS.

---



````markdown
# Home Assistant Configuration – Truma Combi via Trumanita

This file contains the **Home Assistant YAML configuration** for controlling a **Truma Combi** heater using the [Trumanita](https://github.com/vincentgloss/trumanita) LIN → MQTT bridge.

It provides:

- Input selects for **Power Mode** and **Boiler Mode**
- A full-featured **MQTT climate** entity
- Rich Jinja2 templates that turn UI state into the JSON config Trumanita expects
- A set of **MQTT sensors** for temperatures, voltage, and status

All communication uses a single config topic:

```text
truma/control/truma_config
````

and several report topics:

```text
truma/report/room_temp
truma/report/water_temp
truma/report/heating_state
truma/report/boiler_mode
truma/report/error_state
truma/report/voltage
```

---

## 🔧 Input Selects

These are helpers used to control **energy mix / power mode** and **boiler (water heater) mode**.

```yaml
input_select:
  truma_power_mode:
    name: Truma Power Mode
    options:
      - "Gas only"
      - "Electric - low power"
      - "Electric - high power"
      - "Mix - low power"
      - "Mix - high power"
    initial: "Gas only"

  truma_boiler_mode:
    name: Truma Boiler Mode
    options:
      - "Boiler off"
      - "Eco"
      - "Hot"
      - "Boost"
    initial: "Eco"
```

---

## 🌡️ MQTT Climate Entity

The climate entity:

* Controls **heating** vs **fan-only** vs **off**
* Has presets: `heat` vs `heat+water`
* Supports fan modes: `off / low / medium / high / max`
* Reads current temp from `truma/report/room_temp`
* Sends **full JSON config** to `truma/control/truma_config` on every change

```yaml
mqtt:
  climate:
    - name: "Truma Combi"
      unique_id: truma_combi

      modes:
        - "off"
        - "heat"
        - "fan_only"     # vent-only

      # Simple presets: heat only vs heat+water
      preset_modes:
        - "heat"
        - "heat+water"

      min_temp: 5
      max_temp: 29
      temp_step: 1.0
      precision: 1.0
      temperature_unit: "C"

      # Current room temp from Trumanita
      current_temperature_topic: "truma/report/room_temp"
      current_temperature_template: "{{ value | float | round(1) }}"

      fan_modes:
        - "off"
        - "low"
        - "medium"
        - "high"
        - "max"

      # All commands via config JSON
      mode_command_topic: "truma/control/truma_config"
      temperature_command_topic: "truma/control/truma_config"
      preset_mode_command_topic: "truma/control/truma_config"
      fan_mode_command_topic: "truma/control/truma_config"

      optimistic: true
```

---

## 🧠 Command Templates (Jinja2)

All four templates (`mode`, `temperature`, `preset`, `fan`) work the same way:

1. Read the current climate state (mode, target temp, fan, preset).
2. Read the selected **power mode** and **boiler mode** from `input_select` entities.
3. Map user-friendly labels → internal Truma values:

   * `energymix`
   * `mode`
   * `mode2`
   * `heater`
   * `boiler`
   * `fan`
4. Emit a **complete JSON object** as payload to `truma/control/truma_config`.

To keep this README tidy, each template is inside a collapsible section.

---

### 🔘 Mode Change Template

<details>
<summary><strong>Click to expand mode_command_template</strong></summary>
<br>

```yaml
      mode_command_template: >
        {%- set target_temp = state_attr('climate.truma_combi', 'temperature') | float(20) -%}
        {%- set preset = state_attr('climate.truma_combi', 'preset_mode') or 'heat' -%}
        {%- set hvac = value -%}
        {%- set fan_mode = state_attr('climate.truma_combi', 'fan_mode') or 'medium' -%}

        {# ----- POWER MODE (pretty label -> internal key) ----- #}
        {%- set power_state = states('input_select.truma_power_mode') %}
        {%- set power_key_map = {
          'Gas only': 'gas',
          'Electric - low power': 'electric_lowpower',
          'Electric - high power': 'electric_highpower',
          'Mix - low power': 'mix_lowpower',
          'Mix - high power': 'mix_highpower'
        } -%}
        {%- set power = power_key_map.get(power_state, 'gas') -%}

        {%- set energymix_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode_map = {
          'gas': 'gas',
          'electric_lowpower': 'mix1',
          'electric_highpower': 'mix2',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode2_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix',
          'mix_highpower': 'mix'
        } -%}
        {%- set emix = energymix_map.get(power, 'gas') -%}
        {%- set truma_mode = mode_map.get(power, 'gas') -%}
        {%- set truma_mode2 = mode2_map.get(power, 'gas') -%}

        {# ----- BOILER MODE (pretty label -> internal key) ----- #}
        {%- set boiler_state = states('input_select.truma_boiler_mode') %}
        {%- set boiler_key_map = {
          'Boiler off': 'off',
          'Eco': 'eco',
          'Hot': 'hot',
          'Boost': 'boost'
        } -%}
        {%- set boiler_selected = boiler_key_map.get(boiler_state, 'eco') -%}

        {%- if hvac in ['off', 'fan_only'] %}
          {%- set heater = 'off' -%}
          {%- set boiler = 'off' -%}
        {%- else %}
          {%- set heater = 'on' -%}
          {%- if preset == 'heat+water' %}
            {%- set boiler = boiler_selected -%}
          {%- else %}
            {%- set boiler = 'off' -%}
          {%- endif %}
        {%- endif %}

        {# ---- Fan mapping ---- #}
        {%- set fan_map = {
          'off': 'off',
          'low': 'low',
          'medium': 'medium',
          'high': 'high',
          'max': 'max'
        } -%}
        {%- set truma_fan = fan_map.get(fan_mode, 'medium') -%}

        {
          "heater": "{{ heater }}",
          "boiler": "{{ boiler }}",
          "temp": {{ target_temp | int }},
          "fan": "{{ truma_fan }}",
          "energymix": "{{ emix }}",
          "mode": "{{ truma_mode }}",
          "mode2": "{{ truma_mode2 }}"
        }
```

</details>

---

### 🌡️ Temperature Change Template

<details>
<summary><strong>Click to expand temperature_command_template</strong></summary>
<br>

```yaml
      temperature_command_template: >
        {%- set new_temp = value | float | round(0) -%}
        {%- set preset = state_attr('climate.truma_combi', 'preset_mode') or 'heat' -%}
        {%- set hvac = states('climate.truma_combi') -%}
        {%- set fan_mode = state_attr('climate.truma_combi', 'fan_mode') or 'medium' -%}

        {%- set power_state = states('input_select.truma_power_mode') %}
        {%- set power_key_map = {
          'Gas only': 'gas',
          'Electric - low power': 'electric_lowpower',
          'Electric - high power': 'electric_highpower',
          'Mix - low power': 'mix_lowpower',
          'Mix - high power': 'mix_highpower'
        } -%}
        {%- set power = power_key_map.get(power_state, 'gas') -%}

        {%- set energymix_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode_map = {
          'gas': 'gas',
          'electric_lowpower': 'mix1',
          'electric_highpower': 'mix2',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode2_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix',
          'mix_highpower': 'mix'
        } -%}
        {%- set emix = energymix_map.get(power, 'gas') -%}
        {%- set truma_mode = mode_map.get(power, 'gas') -%}
        {%- set truma_mode2 = mode2_map.get(power, 'gas') -%}

        {%- set boiler_state = states('input_select.truma_boiler_mode') %}
        {%- set boiler_key_map = {
          'Boiler off': 'off',
          'Eco': 'eco',
          'Hot': 'hot',
          'Boost': 'boost'
        } -%}
        {%- set boiler_selected = boiler_key_map.get(boiler_state, 'eco') -%}

        {%- if hvac in ['off', 'fan_only'] %}
          {%- set heater = 'off' -%}
          {%- set boiler = 'off' -%}
        {%- else %}
          {%- set heater = 'on' -%}
          {%- if preset == 'heat+water' %}
            {%- set boiler = boiler_selected -%}
          {%- else %}
            {%- set boiler = 'off' -%}
          {%- endif %}
        {%- endif %}

        {%- set fan_map = {
          'off': 'off',
          'low': 'low',
          'medium': 'medium',
          'high': 'high',
          'max': 'max'
        } -%}
        {%- set truma_fan = fan_map.get(fan_mode, 'medium') -%}

        {
          "heater": "{{ heater }}",
          "boiler": "{{ boiler }}",
          "temp": {{ new_temp | int }},
          "fan": "{{ truma_fan }}",
          "energymix": "{{ emix }}",
          "mode": "{{ truma_mode }}",
          "mode2": "{{ truma_mode2 }}"
        }
```

</details>

---

### 💧 Preset Change Template (heat vs heat+water)

<details>
<summary><strong>Click to expand preset_mode_command_template</strong></summary>
<br>

```yaml
      preset_mode_command_template: >
        {%- set preset = value -%}
        {%- set target_temp = state_attr('climate.truma_combi', 'temperature') | float(20) -%}
        {%- set hvac = states('climate.truma_combi') -%}
        {%- set fan_mode = state_attr('climate.truma_combi', 'fan_mode') or 'medium' -%}

        {%- set power_state = states('input_select.truma_power_mode') %}
        {%- set power_key_map = {
          'Gas only': 'gas',
          'Electric - low power': 'electric_lowpower',
          'Electric - high power': 'electric_highpower',
          'Mix - low power': 'mix_lowpower',
          'Mix - high power': 'mix_highpower'
        } -%}
        {%- set power = power_key_map.get(power_state, 'gas') -%}

        {%- set energymix_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode_map = {
          'gas': 'gas',
          'electric_lowpower': 'mix1',
          'electric_highpower': 'mix2',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode2_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix',
          'mix_highpower': 'mix'
        } -%}
        {%- set emix = energymix_map.get(power, 'gas') -%}
        {%- set truma_mode = mode_map.get(power, 'gas') -%}
        {%- set truma_mode2 = mode2_map.get(power, 'gas') -%}

        {%- set boiler_state = states('input_select.truma_boiler_mode') %}
        {%- set boiler_key_map = {
          'Boiler off': 'off',
          'Eco': 'eco',
          'Hot': 'hot',
          'Boost': 'boost'
        } -%}
        {%- set boiler_selected = boiler_key_map.get(boiler_state, 'eco') -%}

        {%- if hvac in ['off', 'fan_only'] %}
          {%- set heater = 'off' -%}
          {%- set boiler = 'off' -%}
        {%- else %}
          {%- set heater = 'on' -%}
          {%- if preset == 'heat+water' %}
            {%- set boiler = boiler_selected -%}
          {%- else %}
            {%- set boiler = 'off' -%}
          {%- endif %}
        {%- endif %}

        {%- set fan_map = {
          'off': 'off',
          'low': 'low',
          'medium': 'medium',
          'high': 'high',
          'max': 'max'
        } -%}
        {%- set truma_fan = fan_map.get(fan_mode, 'medium') -%}

        {
          "heater": "{{ heater }}",
          "boiler": "{{ boiler }}",
          "temp": {{ target_temp | int }},
          "fan": "{{ truma_fan }}",
          "energymix": "{{ emix }}",
          "mode": "{{ truma_mode }}",
          "mode2": "{{ truma_mode2 }}"
        }
```

</details>

---

### 🌀 Fan Mode Change Template

<details>
<summary><strong>Click to expand fan_mode_command_template</strong></summary>
<br>

```yaml
      fan_mode_command_template: >
        {%- set new_fan_mode = value -%}
        {%- set preset = state_attr('climate.truma_combi', 'preset_mode') or 'heat' -%}
        {%- set hvac = states('climate.truma_combi') -%}
        {%- set target_temp = state_attr('climate.truma_combi', 'temperature') | float(20) -%}

        {%- set power_state = states('input_select.truma_power_mode') %}
        {%- set power_key_map = {
          'Gas only': 'gas',
          'Electric - low power': 'electric_lowpower',
          'Electric - high power': 'electric_highpower',
          'Mix - low power': 'mix_lowpower',
          'Mix - high power': 'mix_highpower'
        } -%}
        {%- set power = power_key_map.get(power_state, 'gas') -%}

        {%- set energymix_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode_map = {
          'gas': 'gas',
          'electric_lowpower': 'mix1',
          'electric_highpower': 'mix2',
          'mix_lowpower': 'mix1',
          'mix_highpower': 'mix2'
        } -%}
        {%- set mode2_map = {
          'gas': 'gas',
          'electric_lowpower': 'electric',
          'electric_highpower': 'electric',
          'mix_lowpower': 'mix',
          'mix_highpower': 'mix'
        } -%}
        {%- set emix = energymix_map.get(power, 'gas') -%}
        {%- set truma_mode = mode_map.get(power, 'gas') -%}
        {%- set truma_mode2 = mode2_map.get(power, 'gas') -%}

        {%- set boiler_state = states('input_select.truma_boiler_mode') %}
        {%- set boiler_key_map = {
          'Boiler off': 'off',
          'Eco': 'eco',
          'Hot': 'hot',
          'Boost': 'boost'
        } -%}
        {%- set boiler_selected = boiler_key_map.get(boiler_state, 'eco') -%}

        {%- if hvac in ['off', 'fan_only'] %}
          {%- set heater = 'off' -%}
          {%- set boiler = 'off' -%}
        {%- else %}
          {%- set heater = 'on' -%}
          {%- if preset == 'heat+water' %}
            {%- set boiler = boiler_selected -%}
          {%- else %}
            {%- set boiler = 'off' -%}
          {%- endif %}
        {%- endif %}

        {%- set fan_map = {
          'off': 'off',
          'low': 'low',
          'medium': 'medium',
          'high': 'high',
          'max': 'max'
        } -%}
        {%- set truma_fan = fan_map.get(new_fan_mode, 'medium') -%}

        {
          "heater": "{{ heater }}",
          "boiler": "{{ boiler }}",
          "temp": {{ target_temp | int }},
          "fan": "{{ truma_fan }}",
          "energymix": "{{ emix }}",
          "mode": "{{ truma_mode }}",
          "mode2": "{{ truma_mode2 }}"
        }
```

</details>

---

## 📟 MQTT Sensors

Finally, the sensors that expose the Truma data in Home Assistant:

```yaml
  sensor:
    - name: "Truma Room Temperature"
      state_topic: "truma/report/room_temp"
      unit_of_measurement: "°C"
      device_class: temperature

    - name: "Truma Water Temperature"
      state_topic: "truma/report/water_temp"
      unit_of_measurement: "°C"
      device_class: temperature

    - name: "Truma Error State"
      state_topic: "truma/report/error_state"

    - name: "Voltage repported by Truma"
      state_topic: "truma/report/voltage"
      unit_of_measurement: "V"
      device_class: voltage

    - name: "Truma heating state"
      state_topic: "truma/report/heating_state"

    - name: "Truma boiler mode"
      state_topic: "truma/report/boiler_mode"
```

---

## ✅ Usage Summary

1. Make sure the Trumanita add-on is running and using the same MQTT topics.
2. Add this YAML to your `configuration.yaml` (or a package).
3. Restart Home Assistant.
4. Use:

   * `climate.truma_combi` as your main thermostat entity
   * `input_select.truma_power_mode` to choose gas/electric/mix
   * `input_select.truma_boiler_mode` to choose boiler off/eco/hot/boost

Every change in the UI generates a **full JSON config** and publishes it to:

```text
truma/control/truma_config
```

which Trumanita then sends to the Truma Combi over LIN.

---

```

