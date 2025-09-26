# HW Pump Controller – YAML Summary

## 🔧 What this YAML does
- Controls your **hot-water recirculation pump** via an ESP32 board.  
- The pump can be activated by:
  1. **Manual switch** in Home Assistant (`pump_control`).  
  2. **30-minute Boost timer** (comfort mode).  
  3. **5-minute Legionella timer** (hygiene flush).  
  4. **Physical button on the pump**:
     - Short press (<1s) → Boost (30 min).  
     - Long press (≥3s) → Legionella flush (5 min).  
- Additionally, when the pump is running, it applies **smart control**:
  - Reads **flow temperature** (supply) and **pump temperature** (return).  
  - If ΔT > 15 °C → pump **turns ON** (LED1 flashes).  
  - If ΔT < 10 °C → pump **turns OFF**.  
  - Otherwise → pump state remains stable.  
- Publishes pump state and timers to Home Assistant.  
- Uses **5 LEDs on the front panel** to indicate status.

---

## 💡 LED Behavior

### **LED1 (Thermometer logo)**
- **OFF** → pump is OFF.  
- **ON (solid)** → pump is ON (manual, boost, legionella, or ΔT stable).  
- **Blinking (flash, 0.75 s)** → pump just turned ON because ΔT > 15 °C.  

### **LED2 + LED3 (Auto-adapt logos, 2 LEDs)**
- **Both solid ON** → pump logic active (`_run_pump = true`).  
- **Both pulsing (sleep effect, slow fade 10–40%)** → pump idle (`_run_pump = false`).  

### **LED4 (100% logo)**
- **Blinking fast (0.25 s pulse)** → 30-minute Boost timer active.  
- **OFF** → no boost active.  

### **LED5 (Sensor logo)**
- **Solid ON** → 5-minute Legionella flush active.  
- **OFF** → no legionella cycle.  

---

## 🎯 Summary at a glance
- **Pump running?** → Look at **LED1** (thermometer).  
- **System active vs idle?** → Look at **LED2 + LED3** (auto-adapt).  
- **Why is pump ON?**
  - LED4 flashing → **Boost mode**.  
  - LED5 solid → **Legionella flush**.  
  - LED1 flashing → **ΔT exceeded** (pipes cold, heating in progress).  
- **Front button:**
  - Short press → Boost (30m).  
  - Long press → Legionella flush (5m).  

👉 Panel logic:  
- **Thermometer = pump status**  
- **Auto-adapt = system active/idle**  
- **100% = boost**  
- **Sensor = legionella**

