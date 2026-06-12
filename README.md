Fairland Pool Heat Pump Local Controller (ESP32 + RS485 + MQTT)

A complete, 100% local integration (no Tuya cloud) for **Fairland** pool heat pumps (and clones such as Poolex, Madimack, Aquark, Inverpad). It utilizes an **ESP32** microcontroller, **ESPHome**, and the **Modbus RTU (RS485)** protocol.

This project enables full control of the heat pump from Smart Home systems (OpenHAB, Home Assistant) **while keeping the original factory touch screen fully functional.**

## 🌟 Key Features & "The 25°C Bug" Fix
Many users integrating these heat pumps encounter a frustrating issue: the Smart Home system ignores the sent target temperature, and the pump stubbornly reverts to a default 25°C, conflicting with the physical panel.

**The Solution:** This configuration eliminates this bug. Fairland heat pumps do not use a single universal temperature register. Instead, they use **three separate holding registers** depending on the active operating mode:
* `Register 2`: Target Temperature for **AUTO** mode
* `Register 3`: Target Temperature for **HEATING** mode
* `Register 4`: Target Temperature for **COOLING** mode

By mapping these three registers independently, we completely eliminate the "conflict of masters" between the Smart Home system and the physical heat pump panel.

### Additional Features:
* **Live Diagnostics:** Monitor Inlet/Outlet/Ambient temperatures and compressor speed (%).
* **Power Calculation (W):** Automatically calculates real-time power consumption based on compressor amperage (A) and internal inverter bus voltage (DC).
* **Full Control:** Toggle Power, change Operating Modes (Auto/Heat/Cool), and Power Modes (Smart/Silence/Turbo).
* **Error Codes:** Binary reading of all E-series and F-series errors (perfect for PUSH notifications).
* **Collision-Free:** Uses a 60-second polling interval to co-exist peacefully on the same RS485 bus as the factory screen.

---

## 🛠️ Hardware Requirements

1. **ESP32 Board** (e.g., ESP32-WROOM-32 DevKit).
2. **RS485 to TTL Module (Auto Flow Control version)**. It only has VCC, GND, TXD, and RXD pins. No need to solder or manage DE/RE pins.
3. **DC-DC Step-Down Converter (12V to 5V)**. Crucial for dropping the 12V output from the heat pump's motherboard to a safe 5V for the ESP32.
4. **4-Core Stranded Cable** (e.g., a high-quality USB cable). Due to pump vibrations, avoid solid-core wires to prevent metal fatigue and snapping.
5. **JST-XH 4-pin Connector (2.54mm pitch)** or female Dupont jumper wires.
6. **3D Printed Enclosure** (PETG material is recommended due to outdoor/operational temperatures).

---

## 🔌 Wiring Guide

Locate the empty 4-pin socket on the heat pump's mainboard labeled `CN12` (often marked as `B A G +12V`). Leave the factory cable from the touch screen plugged into its original location.

| Heat Pump Board (CN12) | DC-DC Converter | ESP32 | RS485 Module |
| :---        | :---      | :---       | :---                  |
| **+12V**    | IN+       | -          | -                     |
| **G (GND)** | IN-       | GND        | GND *(Common Ground)* |
| -           | OUT+ (5V) | VIN / 5V   | VCC (5V)              |
| -           | -         | **GPIO17** | DI / TXD              |
| -           | -         | **GPIO16** | RO / RXD              |
| **A**       | -         | -          | A (D+)                |
| **B**       | -         | -          | B (D-)                |

---

## 🚀 Compiling and Flashing (via Podman/Docker)

If you prefer not to install ESPHome natively on your system, you can easily compile and upload the firmware using **Podman** (or Docker). 

Place your YAML configuration file (e.g., `iphcr26.yaml`) in your current directory and run the following command to compile and flash Over-The-Air (OTA) directly to the ESP32's IP address:

```bash
podman run --rm -it -v "$PWD":/config ghcr.io/esphome/esphome run iphcr26.yaml --device 192.168.x.x
```

🏡 OpenHAB UI Integration Example

An example showcasing how to use dynamic visibility in OpenHAB to seamlessly swap the temperature sliders based on the currently selected operating mode, keeping the UI clean.

.sitemap file:

Frame label="Fairland Pool Heat Pump" {
    Switch item=FairlandPower label="Main Power" icon="switch"
    Switch item=FairlandOperatingMode label="Operating Mode" mappings=["Auto"="Auto", "Heating"="Heating", "Cooling"="Cooling"]

    // Visibility magic - only displays the slider for the active mode
    Setpoint item=FairlandTargetTempAuto label="Target Temp (AUTO) [%.1f °C]" minValue=18 maxValue=40 step=1 visibility=[FairlandOperatingMode=="Auto"]
    Setpoint item=FairlandTargetTempHeat label="Target Temp (HEATING) [%.1f °C]" minValue=18 maxValue=40 step=1 visibility=[FairlandOperatingMode=="Heating"]
    Setpoint item=FairlandTargetTempCool label="Target Temp (COOLING) [%.1f °C]" minValue=18 maxValue=40 step=1 visibility=[FairlandOperatingMode=="Cooling"]
    
    Selection item=FairlandPowerMode label="Power Mode" mappings=["Smart"="Smart", "Silence"="Silence", "Super Silence"="Super Silence", "Turbo"="Turbo"]

    Text item=FairlandTempInlet label="Inlet Temp [%.1f °C]" icon="water"
    Text item=FairlandPowerW label="Current Power [%d W]" icon="energy"
}

⚠️ Disclaimer
Modifying devices operating with mains voltage is done entirely at your own risk. Always verify that the main circuit breaker powering the heat pump is completely turned OFF before opening the casing or touching any internal components.
