<div align="center">

# 🌱 Soil Moisture Monitoring System

### IoT-Ready Water Level Indication using NodeMCU (ESP8266)

![Platform](https://img.shields.io/badge/Platform-NodeMCU%20ESP8266-blue?style=for-the-badge&logo=espressif&logoColor=white)
![IDE](https://img.shields.io/badge/IDE-Arduino%20IDE-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![Sensor](https://img.shields.io/badge/Sensor-LM393%20Soil%20Probe-green?style=for-the-badge)
![Baud](https://img.shields.io/badge/UART-115200%20baud-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Tested%20%26%20Working-success?style=for-the-badge)

*Is the soil thirsty? Green says OK, red says water me!* 💧🟢🔴

</div>

---

## 📖 Overview

Much of the water used in agriculture is wasted because irrigation runs on habit, not on actual soil condition. This project is a low-cost sensing node that tells you, continuously, whether the soil around a plant's roots is **dry or adequately wet**.

A resistive soil probe (LM393 sensor module) feeds the NodeMCU's 10-bit ADC. Once per second the firmware compares the reading against a threshold of **512** and lights a **green LED (WATER OK)** or a **red LED (WATER LOW)**. Every raw reading and its status is also streamed over UART at 115200 baud for monitoring and calibration.

Because the ESP8266 has built-in Wi-Fi, upgrading to a cloud-connected node later is a **firmware-only change**.

## ✨ Features

- 🌡️ **Analog soil moisture sensing** (0–1023) on A0
- 🚦 **Dual-LED status**: green = WATER OK, red = WATER LOW (mutually exclusive)
- 🖥️ **Live serial telemetry** with raw value + status on every line
- 🎯 **Calibratable threshold** (default 512, the ADC midpoint)
- 🔌 **USB-powered**, with no external supply or libraries needed
- 📡 **Wi-Fi ready** hardware for future MQTT/HTTP upgrade

## 🔄 System Block Diagram

![Block Diagram](images/block-diagram.jpg)

Soil probe → LM393 sensor module → NodeMCU ADC → decision → **green LED / red LED / serial port**

## 🧰 Hardware Required

| Component | Specification | Purpose |
|---|---|---|
| NodeMCU ESP8266 | ESP-12E, 80 MHz, 3.3 V logic | Main controller |
| Soil Moisture Sensor | Resistive 2-prong probe + LM393 module | Converts soil conductivity to analog voltage |
| Green LED | 5 mm | WATER OK indicator |
| Red LED | 5 mm | WATER LOW indicator |
| Resistors | 220 Ω – 330 Ω, ¼ W × 2 | Limit LED current |
| Jumper Wires | M-M / M-F | Interconnects |
| Micro-USB Cable | Data-capable | Power, programming, serial link |

> 💡 **Resistor maths:** at 3.3 V with a ~2.0 V green LED, 220 Ω gives ≈ 6 mA. That is bright enough to see and well within the ESP8266's per-pin limit.

## 🔌 Pin Configuration

| Signal | NodeMCU Pin | GPIO | Direction | Notes |
|---|---|---|---|---|
| Soil sensor analog out | `A0` | ADC0 | Input | Only ADC channel |
| Green status LED | `D5` | GPIO14 | Output | Active HIGH |
| Red status LED | `D6` | GPIO12 | Output | Active HIGH |
| Sensor supply | `3V3` | — | Power | 3.3 V rail |
| Common ground | `GND` | — | Reference | Shared by all |

### 🔧 Circuit Diagram

![Circuit Diagram](images/circuit-diagram.jpg)

> ⚠️ **ADC note:** The bare ESP8266 chip accepts only 0–1.0 V on its ADC. The **NodeMCU board** has an on-board divider (≈ 0–3.3 V), so the sensor can connect directly to A0. Never do this on a bare ESP-12E module without an external divider.
>
> 💡 **Why D5/D6?** They carry no boot-strapping restrictions and have no alternate-function conflicts.

## ⚙️ How It Works

Wet soil conducts well (low resistance), so the ADC reads **low**. Dry soil is highly resistive, so the ADC reads **high**. A higher number means drier soil.

Each loop iteration (every 1 second):

1. 📥 **Sample**: read A0 (0–1023)
2. 📤 **Report**: print the raw value over serial
3. 🧠 **Decide**: compare with threshold `512`
4. 💡 **Actuate**: set the LEDs and append `WATER LOW` or `WATER OK`

![Firmware Flowchart](images/firmware-flowchart.jpg)

## 🎯 Threshold Logic

| ADC Range | Approx. Voltage | Soil | Green | Red | Serial |
|---|---|---|---|---|---|
| 0 – 512 | 0 – 1.65 V | Wet / sufficient | ON | OFF | `WATER OK` |
| 513 – 1023 | 1.66 – 3.3 V | Dry / insufficient | OFF | ON | `WATER LOW` |

### 🧪 Calibrating for Your Soil

1. Put the probe in thoroughly **dry** soil and note the steady reading (dry reference).
2. Water to the ideal level, wait a few minutes, and note the reading (wet reference).
3. Set the threshold to the **midpoint** of the two, update the constant, and re-flash.
4. Cycle between both conditions to confirm clean switching.

> Soil type, fertiliser, temperature, probe depth and supply voltage all shift readings, so always recalibrate for a new setup.

## 💻 Software Setup

| Item | Setting |
|---|---|
| IDE | Arduino IDE + ESP8266 core (Boards Manager) |
| Board | NodeMCU 1.0 (ESP-12E Module) |
| CPU / Flash | 80 MHz / 4 MB |
| Upload & Serial speed | 115200 baud |
| Libraries | None (core Arduino functions only) |

### 🚀 Getting Started

1. Clone the repo
   ```bash
   git clone https://github.com/shoaib725/<your-repo-name>.git
   ```
2. Install the **ESP8266 board package** in Arduino IDE.
3. Wire the circuit as shown above.
4. Open the sketch, select **NodeMCU 1.0 (ESP-12E Module)** and your COM port.
5. Upload, then open the Serial Monitor at **115200 baud**.
6. Move the probe between dry and moist soil and watch the LEDs.

> If the monitor shows garbage characters, the baud rates don't match.

## 🧪 Test Results

| Condition | ADC Range | Margin vs 512 | LED | Serial |
|---|---|---|---|---|
| Dry soil / air | 590 – 593 | +78 to +81 | 🔴 Red ON | `WATER LOW` |
| Moist soil | 378 – 403 | −109 to −134 | 🟢 Green ON | `WATER OK` |

**🔴 Dry test**

![Dry Test](images/test-dry.jpg)

**🟢 Moist test**

![Moist Test](images/test-wet.jpg)

**Observations**
- ✅ Readings stable within about ±3 counts with no software filtering
- ✅ ~190-count gap between the two states (nearly 1/5 of the ADC range), so it tolerates calibration error
- ✅ State changes on the very next sample, so the only latency is the 1 s delay
- ✅ Only one LED lit at a time, and the serial status always matched the LED

## ⚖️ Advantages & Limitations

| 👍 Advantages | 👎 Limitations |
|---|---|
| Very low cost and component count | Resistive probes corrode over time |
| Instant local indication, no network needed | No hysteresis, so it may flicker at the boundary |
| Serial logging for easy calibration | Single ADC channel (one probe) |
| Wi-Fi hardware already on board | Relative verdict, not volumetric water % |
| USB powered, no external libraries | No data persistence |

## 🔮 Future Enhancements

- [ ] Two-threshold **hysteresis** to stop boundary flicker
- [ ] Average several samples to reduce noise
- [ ] **Relay/MOSFET + pump** for automatic irrigation
- [ ] **MQTT / HTTP** to ThingSpeak or Blynk for remote monitoring
- [ ] Power the probe from a GPIO only while measuring, to reduce corrosion
- [ ] **DHT22** for temperature and humidity correlation
- [ ] Move to a **capacitive** probe

## 🧠 Skills Demonstrated

`ADC Acquisition` · `Threshold-Based Decision Logic` · `GPIO Control` · `UART Telemetry` · `ESP8266 / NodeMCU` · `Sensor Calibration` · `Circuit Design`

## 👨‍💻 Author

**Pathan Mohammad Shoaib Khan**
B.Tech, Electronics & Communication Engineering
Seshadri Rao Gudlavalleru Engineering College

[![GitHub](https://img.shields.io/badge/GitHub-shoaib725-181717?style=for-the-badge&logo=github)](https://github.com/shoaib725)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-shoaibkhan725-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/shoaibkhan725)

---

<div align="center">

⭐ If you found this project useful, consider giving it a star! ⭐

</div>
