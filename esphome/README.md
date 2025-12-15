# HW Pump Controller – ESPHome (v4.0)

Smart ESPHome controller for **hot-water recirculation pumps** with $\Delta T$ logic, Boost mode, Legionella flush, runtime failsafes, and full LED status indication.
Designed for **ESP32-C3 (DFRobot Beetle ESP32-C3)** and integrates automatically with Home Assistant via ESPHome API.

Thanks to @pdw-mb (https://github.com/pdw-mb) for all the hard work put into this. What I did is just some minor modifications to best adapt the logic to my own use case.

---

## 🚀 Features

### Pump Control Logic
- $\Delta T$ = **Flow temperature − Return temperature**
- Pump turns **ON** when $\Delta T > \text{ON threshold}$
- Pump turns **OFF** when $\Delta T < \text{OFF threshold}$
- ON/OFF thresholds are adjustable from Home Assistant

### Operating Modes
- **Manual switch** from HA (`switch.pump`)
- **Boost mode**: 4 minutes
- **Legionella flush**: 6 minutes
- **Physical button**:
  - Short press (50ms–1s) $\rightarrow$ **Boost**
  - Long press (3s–10s) $\rightarrow$ **Legionella**

### Safety & Protection
- Boost & Legionella are **mutually exclusive** (starting one cancels the other).
- If either temperature sensor is invalid for $>60\text{s} \rightarrow$ **pump forced OFF**
- If pump runs continuously for $>10\text{ minutes} \rightarrow$ **failsafe OFF**
- Timers **persist across reboots** and resume automatically.

### Offline Operation
All logic (Boost, Legionella, $\Delta T$ control, failsafes) works **even if Wi-Fi or Home Assistant are offline**.

---

## 🔄 Startup Behavior

1. LEDs enter a low-power "sleep" mode.
2. The main control logic resets the pump state to OFF initially.
3. Any active timer is automatically resumed:
   - **Boost** $\rightarrow$ LED4 ON
   - **Legionella** $\rightarrow$ LED5 ON
4. Pump control logic is recalculated immediately.

If no timer is active and the manual switch is OFF $\rightarrow$ idle mode (LED2/LED3 sleeping).

---

## 🏡 Home Assistant Entities (via ESPHome API)

| Entity ID | Type | Description |
|:---|:---|:---|
| `switch.pump` | Switch | Main logical pump control (triggers control logic) |
| `button.boost_pump` | Button | Starts Boost (4 min) |
| `button.legionella_flush` | Button | Starts Legionella flush (6 min) |
| `binary_sensor.pump_state` | Binary Sensor | **Physical relay state** (ON/OFF) |
| `binary_sensor.pump_button` | Binary Sensor | Physical button input (for short/long press) |
| `sensor.boost_timer` | Sensor | Boost time remaining (**rounded minutes**) |
| `sensor.legionella_timer` | Sensor | Legionella time remaining (**rounded minutes**) |
| `sensor.flow_temperature` | Sensor | Flow/supply water temperature |
| `sensor.pump_temperature` | Sensor | Return temperature |
| `number.pump_delta_t_on_threshold` | Number | Pump activation threshold ($\Delta T$ in $\text{°C}$) |
| `number.pump_delta_t_off_threshold` | Number | Pump deactivation threshold ($\Delta T$ in $\text{°C}$) |

> **Note:** Internal LEDs (`led2–led5`) are hidden from Home Assistant.

---

## 💡 LED Behavior

The physical LEDs provide immediate feedback on the controller's status:

| LED | Color/Role | ON/OFF State | Effect/Meaning |
|:---|:---|:---|:---|
| **LED1** | Blue (Pump Status) | OFF | Pump is physically OFF. |
| | | **Fast Pulse** | **Sensor Failure!** (Sensors invalid for $>1\text{s}$) |
| | | Pulse (Slow Flash) | Pump is physically ON (Normal $\Delta T$ run). |
| **LED2/LED3**| Green (System Status) | Solid ON | **Logic Active** (Pump forced ON by Timer/Switch). |
| | | Slow Pulse (Sleep) | **Idle** (Waiting for $\Delta T$ trigger or user input). |
| **LED4** | Amber (Boost) | Solid ON | **Boost Mode Active** (4 minutes remaining). |
| **LED5** | Red (Legionella) | Solid ON | **Legionella Flush Active** (6 minutes remaining). |

---

## 🛡️ Safety Mechanics

* **Sensor Failsafe:** If flow or return temperature becomes invalid (`NaN`) for more than **60 seconds**, the internal pump control is immediately forced OFF to prevent damage. LED1 will flash rapidly to indicate the fault.
* **Maximum Runtime Failsafe:** The pump cannot run for more than **10 minutes (600 seconds)** continuously. If this limit is exceeded, the pump is forced OFF, and the run counter is reset. This prevents scenarios where the $\Delta T$ failsafe might not trigger correctly (e.g., if temperatures fail to equalize).
* **Timer Exclusivity:** Starting the Boost timer automatically calls the script to stop the Legionella timer, and vice-versa, ensuring the system only runs one special mode at a time.

---

## ⚙️ Notes

* All timers shown in Home Assistant are rounded to the nearest **minute**.
* $\Delta T$ thresholds are adjustable from Home Assistant via the `number` entities.
* The system operates fully autonomously (including $\Delta T$ logic and failsafes) without Home Assistant or Wi-Fi connectivity.
* The **ESP-IDF** framework is required for the ESP32-C3 Beetle board configuration.
* The internal switch for the pump is inverted, meaning the `_pump_control` relay turns ON when the ESP pin is LOW.
