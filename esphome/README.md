# HW Pump Controller – YAML Summary (v2)

## 🔧 What this YAML does
- Controls your **hot-water recirculation pump** via an ESP32 (ESP-IDF).
- Pump activation methods:
  - **Manual switch** in Home Assistant (`Pump` / `pump_control`).
  - **30-minute Boost** (comfort mode).
  - **5-minute Legionella flush** (hygiene).
  - **Physical button**:
    - Short press (<1s) → Boost (30 min)
    - Long press (≥3s) → Legionella (5 min)
- **Exclusivity:** Boost and Legionella cannot run together (starting one cancels the other).
- **Smart ΔT control:**
  - ΔT > **ON threshold** → pump ON (LED1 pulses)
  - ΔT < **OFF threshold** → pump OFF
  - Thresholds adjustable from HA: `number.delta_on`, `number.delta_off`.
- **Failsafe:** if temperature sensors are invalid for >60s, the pump is forced **OFF**.
- **MQTT availability:** publishes `online` / `offline` to `hw_pump/status`. HA discovery is enabled.

Published entities (HA):
- Switches: `Pump` (logic master)
- Buttons: `Boost pump`, `Legionella flush`
- Timers (minutes): `Boost timer`, `Legionella timer`
- Temperatures: `Flow Temperature`, `Pump Temperature`
- Binary sensor: `Pump State` (device_class: running)
- Numbers (config): `Pump ΔT ON threshold`, `Pump ΔT OFF threshold`

---

## 💡 LED Behavior
**LED1 (Thermometer)**
- OFF → pump OFF  
- Solid ON → pump ON (manual/boost/legionella or ΔT stable)  
- **Pulse (smooth)** → pump just turned ON due to ΔT > threshold  

**LED2 + LED3 (Auto-adapt)**
- Both solid → logic active (`_run_pump = true`)  
- Both slow pulse → idle (`_run_pump = false`)  

**LED4 (100%)**
- Fast pulse (~0.25s) → Boost active  
- OFF → no Boost  

**LED5 (Sensor)**
- Solid ON → Legionella flush active  
- OFF → no Legionella  

---

## 📝 Notes
- Timers are shown in **minutes** for consistency.
- Boost/Legionella are **mutually exclusive**.
- ΔT thresholds are tunable in HA (no reflash).
- MQTT: set broker IP, `username`, and `password` in the `mqtt:` block (or via `!secret`).
