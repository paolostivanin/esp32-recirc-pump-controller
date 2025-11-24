# HW Pump Controller – ESPHome (v4.0)

Smart ESPHome controller for **hot-water recirculation pumps** with ΔT logic, Boost mode, Legionella flush, runtime failsafes, and full LED status indication.  
Designed for **ESP32-C3 (DFRobot Beetle ESP32-C3)** and integrates automatically with Home Assistant via ESPHome API.

Thanks to @pdw-mb (https://github.com/pdw-mb) for the extensive work on this project and for providing the PCB. I only applied a few small adjustments to better fit my specific use case.

---

## Features

### Pump Control Logic
- ΔT = **Flow temperature − Return temperature**
- Pump turns **ON** when ΔT > ON threshold  
- Pump turns **OFF** when ΔT < OFF threshold  
- ON/OFF thresholds are adjustable from Home Assistant

### Operating Modes
- **Manual switch** from HA (`switch.pump`)
- **Boost mode**: 4 minutes  
- **Legionella flush**: 6 minutes  
- **Physical button**:
  - Short press → Boost  
  - Long press (≥3s) → Legionella  

### Safety & Protection
- Boost & Legionella are **mutually exclusive**
- If either temperature sensor is invalid for >60s → **pump forced OFF**
- If pump runs continuously for >10 minutes → **failsafe OFF**
- Timers **persist across reboots** and resume automatically

### Offline Operation
All logic (Boost, Legionella, ΔT control, failsafes) works **even if Wi-Fi or Home Assistant are offline**.

---

## Startup Behavior

1. LEDs enter sleep mode  
2. Pump state resets to OFF  
3. Any active timer is automatically resumed:
   - **Boost** → LED4 ON  
   - **Legionella** → LED5 ON  
4. Pump logic is recalculated immediately

If no timer is active → idle mode (LED2/LED3 sleeping).

---

## Home Assistant Entities (via ESPHome API)

| Entity ID | Type | Description |
|-----------|------|-------------|
| `switch.pump` | Switch | Main logical pump control |
| `button.boost_pump` | Button | Start Boost (4 min) |
| `button.legionella_flush` | Button | Start Legionella flush (6 min) |
| `binary_sensor.pump_state` | Binary Sensor | Physical relay state |
| `binary_sensor.pump_button` | Binary Sensor | Physical button |
| `sensor.boost_timer` | Sensor | Boost time remaining (minutes) |
| `sensor.legionella_timer` | Sensor | Legionella time remaining (minutes) |
| `sensor.flow_temperature` | Sensor | Flow/supply water temperature |
| `sensor.pump_temperature` | Sensor | Return temperature |
| `sensor.delta_temp` | Sensor | Flow − Pump ΔT |
| `number.pump_delta_t_on_threshold` | Number | Activation threshold |
| `number.pump_delta_t_off_threshold` | Number | Deactivation threshold |

Internal LEDs (`led2–led5`) are hidden from HA.

---

## LED Behavior

### **LED1 — Blue (Thermometer)**
- OFF → Pump OFF  
- Solid ON → Pump ON  
- Pulse → Pump just activated (ΔT trigger)

### **LED2 & LED3 — Green (Auto-Adapt)**
- Solid ON → Logic active  
- Slow pulse → Idle (waiting for ΔT or timer)

### **LED4 — Amber (Boost)**
- Fast pulse → Boost active  
- OFF → No boost

### **LED5 — Red (Legionella)**
- Solid ON → Legionella active  
- OFF → Not active

---

## Safety Mechanics

- **Sensor failure**  
  - If flow or return temperature becomes NaN for >60s → pump OFF  

- **Maximum runtime**  
  - Pump cannot run >10 minutes continuously unless ΔT keeps it cycling normally  
  - Failsafe OFF triggers if exceeded  

- **Timer exclusivity**  
  - Starting Boost automatically stops Legionella and vice-versa  

---

## Notes

- All timers shown in **minutes**.
- ΔT thresholds adjustable from HA.
- System works even without Home Assistant.
- ESP-IDF is required for ESP32-C3 Beetle
