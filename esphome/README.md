# HW Pump Controller – YAML Summary (v3.4)

Smart ESPHome controller for **hot-water recirculation pumps** with ΔT logic, Boost & Legionella modes.
Designed for **ESP32-C3 (DFRobot Beetle ESP32-C3)** - full MQTT discovery and debug-friendly logs.


## 🔧 What this YAML does
- Controls your **hot-water recirculation pump** via an ESP32 (ESP-IDF).
- Pump activation methods:
  - **Manual switch** in Home Assistant (`switch.pump` → *Pump*).
  - **30-minute Boost** (comfort mode).
  - **5-minute Legionella flush** (hygiene).
  - **Physical button**:
    - Short press (<1s) → Boost (30 min).
    - Long press (≥3s) → Legionella (5 min).
- **Exclusivity:** Boost and Legionella cannot run together (starting one cancels the other).
- **Smart ΔT control:**
  - ΔT = *Flow temperature − Pump/return temperature*.
  - If ΔT > **ON threshold** → pump ON (LED1 pulses).
  - If ΔT < **OFF threshold** → pump OFF.
  - Thresholds adjustable from HA: `number.pump_delta_t_on_threshold`, `number.pump_delta_t_off_threshold`.
- **Failsafe:** if either temperature sensor becomes invalid for >60s, the pump is forced **OFF**.
- **Symmetric ΔT updates:** reacts to changes from both flow and return temperature sensors.
- **Timers now persist across reboots** (`restore_value: yes`).
- **MQTT availability:** publishes `online` / `offline` to `hw_pump/status` and status events on `hw_pump/events`.
- **Home Assistant discovery:** all entities automatically created via MQTT discovery.

Timers and pump logic are fully autonomous — work even if HA or Wi-Fi is unavailable.

## ⚙️ Startup Behavior
At power-on:
1. LEDs initialize to sleep mode.
2. Pump state defaults to OFF.
3. Any running timers are resumed automatically:
  * Boost: LED4 pulses fast and countdown resumes.
  * Legionella: LED5 solid ON and countdown resumes.
4. Pump logic (update_pump_state) is re-evaluated immediately.

If no timer was running → system starts idle (LED2/3 sleep pulse).
  
---

## 📦 Published Entities in Home Assistant (via MQTT discovery)

| Entity ID                          | Friendly Name        | Type           | Device Class / Icon   | Description                           |
|------------------------------------|----------------------|----------------|-----------------------|---------------------------------------|
| `switch.pump`                      | Pump                 | Switch         | —                     | Master pump control (on/off)          |
| `button.boost_pump`                | Boost pump           | Button         | `mdi:flash`           | Trigger 30-min boost                  |
| `button.legionella_flush`          | Legionella flush     | Button         | `mdi:water-pump`      | Trigger 5-min legionella flush        |
| `binary_sensor.pump_state`         | Pump State           | Binary sensor  | `running`             | Physical pump relay state             |
| `sensor.boost_timer`               | Boost timer          | Sensor         | `mdi:clock-outline`   | Remaining boost timer (minutes)       |
| `sensor.legionella_timer`          | Legionella timer     | Sensor         | `mdi:clock-outline`   | Remaining legionella timer (minutes)  |
| `sensor.flow_temperature`          | Flow Temperature     | Sensor         | Temperature (°C)      | Supply/flow water temperature         |
| `sensor.pump_temperature`          | Pump Temperature     | Sensor         | Temperature (°C)      | Return/pump water temperature         |
| `sensor.delta_temp`                | ΔT Flow-Pump         | Sensor         | Temperature (°C)      | Flow − Pump temperature difference    |
| `number.pump_delta_t_on_threshold` | ΔT ON threshold      | Number         | —                     | ΔT threshold for pump activation (°C) |
| `number.pump_delta_t_off_threshold`| ΔT OFF threshold     | Number         | —                     | ΔT threshold for pump deactivation (°C)|

---

## 💡 LED Behavior

**LED1 (Thermometer)**
- OFF → pump OFF
- Solid ON → pump ON (manual/boost/legionella or ΔT stable)
- **Pulse (smooth)** → pump just turned ON due to ΔT > threshold

**LED2 + LED3 (Auto-adapt)**
- Both solid → logic active (`_run_pump = true`)
- Both slow pulse → idle (`_run_pump = false`)

**LED4 (Boost)**
- Fast pulse (~0.3, smooth transition) → Boost active
- OFF → no Boost

**LED5 (Legionella)**
- Solid ON → Legionella flush active
- OFF → no Legionella

---

## 📝 Notes
- Timers are shown in **minutes** for consistency.
- Boost/Legionella are **mutually exclusive**.
- ΔT thresholds are tunable in HA (no reflash).
- MQTT credentials: set broker IP, `username`, and `password` in the `mqtt:` block (or via `!secret`).

---

## 📡 MQTT Topics

- **Availability**
  - `hw_pump/status` → `online` / `offline`

- **Published (state)**
  - `hw_pump/switch/pump_control/state` → ON / OFF
  - `hw_pump/sensor/flow_temperature/state` → °C
  - `hw_pump/sensor/pump_temperature/state` → °C
  - `hw_pump/sensor/boost_timer_remaining/state` → minutes
  - `hw_pump/sensor/legionella_timer_remaining/state` → minutes
  - `hw_pump/sensor/delta_temp/state` → °C  
  - `hw_pump/binary_sensor/pump_state/state` → ON / OFF

- **Subscribed (commands)**  
  - `hw_pump/switch/pump_control/command` → ON / OFF
  - `hw_pump/button/boost/command` → trigger Boost
  - `hw_pump/button/legionella/command` → trigger Legionella

- **Events**
  - `hw_pump/events` → `Boost timer started`
  - `hw_pump/events` → `Legionella timer started`
  - `hw_pump/events` → `Boot completed`
  - `hw_pump/events` → `Resumed Boost timer after reboot`
  - `hw_pump/events` → `Resumed Legionella timer after reboot`
