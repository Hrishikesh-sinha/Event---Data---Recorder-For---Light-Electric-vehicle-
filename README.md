# 🔋 Event Data Recorder for Light Electric Vehicles

### IoT-Based Real-Time Environmental Monitoring System using ESP32 & Firebase

A low-cost breadboard prototype that monitors temperature, gas concentration, and voltage in real time, pushes readings to the cloud, and lays the groundwork for a **forensic-grade Event Data Recorder (EDR)** for LEV battery packs.

[Overview](#-overview) •
[Architecture](#️-system-architecture) •
[Hardware](#-hardware-components) •
[Getting Started](#-getting-started) •
[Results](#-performance-metrics) •
[Roadmap](#-future-work)

---

## 📖 Overview

Investigations into lithium-ion battery thermal runaway incidents in real-world LEV (electric two-wheeler) fleets consistently show that onboard BMS and telematics hardware is destroyed once thermal runaway propagates — resulting in **total loss of volatile event logs**. Existing solutions (gas-sensor research, patented cable-based detection circuits, commercial BMS/telematics) focus almost entirely on *prevention* and *local alerting*, with no mechanism to preserve forensic data during propagation.

This project is a first step toward closing that gap: a low-cost prototype that

- ⚡ Continuously monitors **temperature, gas concentration, and voltage**
- 🚨 Applies **threshold-based anomaly detection**
- ☁️ Immediately pushes sensor data and alert states to **Firebase**, so data "escapes" the device before hardware compromise
- 🖥️ Provides both **local (LCD + buzzer)** and **remote (web dashboard)** visibility

> Built as part of the *Embedded Systems (BCSE305L)* course project, VIT University — Winter Semester 2025–26.

---

## 🏗️ System Architecture

```
Sensors → ESP32 (ADC + Processing) → WiFi (TCP/IP, JSON) → Firebase Realtime Database → Web Dashboard / Mobile UI
```

| Layer | Function |
|---|---|
| **Sensing** | Temperature, gas, and voltage sensors generate analog/digital signals |
| **Processing** | ESP32 samples via ADC, applies filtering and threshold logic |
| **Communication** | Data serialized to JSON, transmitted over WiFi via HTTP/REST |
| **Cloud** | Firebase Realtime Database stores and syncs data instantly |
| **User Interface** | Web dashboard / LCD displays live values and alerts |

The system runs as a **soft real-time**, polling-based loop (no RTOS/interrupts): `Read → Process → Transmit → Display`, repeating every ~2 seconds.

<details>
<summary><b>Finite State Machine</b></summary>

```
Idle → Read Sensors → Send Data (Firebase) → Display Output → (loop)
```
- **Idle:** system waits for the next cycle
- **Read:** acquires temperature, gas, voltage
- **Send:** transmits JSON payload to Firebase
- **Display:** updates LCD and web UI

</details>

---

## 🔧 Hardware Components

| Component | Model | ESP32 Pin | Interface | Role |
|---|---|---|---|---|
| Microcontroller | ESP32 DevKit | — | WiFi + GPIO | Central processing & communication |
| Temperature Sensor | DS18B20 | GPIO4 | OneWire (Digital) | Temperature acquisition |
| Gas Sensor | MQ-x series | GPIO34 | Analog (ADC) | Gas concentration / off-gassing detection |
| Voltage Sensor | Voltage divider module | GPIO35 | Analog (ADC) | Supply voltage monitoring |
| Display | 16×2 LCD (I2C) | GPIO21 (SDA), GPIO22 (SCL) | I2C | Local real-time readout |
| Buzzer | Piezo buzzer | GPIO25 | Digital output | Audible alert |
| Power | 3.7V 18650 Li-ion / USB | 3.3V / 5V | — | System power |

> Built entirely on a solderless breadboard — no soldering, no real battery pack, no thermal runaway testing at this prototype stage.

---

## ⚙️ Software Stack

| Category | Details |
|---|---|
| Framework | Arduino core for ESP32 (Arduino IDE / PlatformIO) |
| Libraries | `WiFi.h`, `OneWire.h`, `DallasTemperature.h`, `LiquidCrystal_I2C.h`, Firebase ESP32 Client, `ArduinoJson` |
| Cloud Backend | Firebase Realtime Database (JSON store, REST + WebSocket sync) |
| Protocol | HTTP/REST over WiFi (TCP/IP), JSON payloads |

**Sample Firebase document:**

```json
{
  "temperature": 28.3,
  "gas": 118,
  "voltage": 3.91,
  "status": "NORMAL"
}
```

---

## 🧠 Algorithm

```
1. Initialize sensors, WiFi, and Firebase connection
2. Read temperature, gas, and voltage values
3. Convert raw ADC values into usable units
4. Compare readings against predefined thresholds
5. Set system status to NORMAL or ALERT
6. Activate buzzer/LED if an alert condition is detected
7. Push data to Firebase in JSON format
8. Update LCD / web UI
9. Repeat continuously (~2 second cycle)
```

**Alert thresholds:**

| Parameter | Normal Range | Alert Condition |
|---|---|---|
| Temperature | 20–50 °C | > 60 °C |
| Gas | 100–250 ppm | > 300 ppm |
| Voltage | 3.7–4.2 V | < 3.3 V |

---

## 🌐 Execution Model (Edge / Fog / Cloud)

| Layer | Platform | Function |
|---|---|---|
| **Edge** | ESP32 | Sensor reading, local threshold checks, alert generation |
| **Fog** | Local WiFi network | Buffering, LCD/local display |
| **Cloud** | Firebase | Long-term storage, remote monitoring, dashboard sync |

---

## 📊 Performance Metrics

| Metric | Measured | Target |
|---|---|---|
| Response Time | 1–2 s | < 3 s |
| Sampling Interval | 2 s | 2 s |
| Detection Accuracy | ~95% | ≥ 90% |
| Data Transmission Success | ~90% | ≥ 85% |
| System Uptime | ~99% | ≥ 95% |
| False Alerts | < 5% | < 10% |

<details>
<summary><b>✅ Test Case Summary</b></summary>

| TC# | Scenario | Expected Result | Status |
|---|---|---|---|
| TC01 | Normal operation | Status = NORMAL | ✅ PASS |
| TC02 | Gas > threshold | ALERT, buzzer ON | ✅ PASS |
| TC03 | Temperature > threshold | ALERT triggered | ✅ PASS |
| TC04 | Voltage drop | ALERT, warning shown | ✅ PASS |
| TC05 | WiFi disconnection | Data buffered, retry | ✅ PASS |
| TC06 | WiFi reconnect | Transmission resumes | ✅ PASS |
| TC07 | Continuous long-duration run | No crash, stable | ✅ PASS |
| TC08 | Display accuracy check | LCD matches sensor values | ✅ PASS |

</details>

---

## 📈 Comparative Analysis

| Feature | This Project (ESP32) | Arduino | Raspberry Pi | Commercial System |
|---|---|---|---|---|
| Cost | Low | Low | High | Very High |
| WiFi | Built-in | External module | Built-in | Built-in |
| Power Consumption | Low | Moderate | High | Optimized |
| Real-time Response | Fast | Moderate | Slower | Fast |
| Scalability | Good | Limited | High | High |
| Ease of Use | Simple | Moderate | Complex | Professional |

---

## 📂 Repository Structure

```
├── firmware/     # ESP32 Arduino/PlatformIO source code
├── docs/         # Block diagrams, circuit diagrams, FSM, DFG/CDFG
├── dashboard/    # Web UI (Firebase-connected dashboard)
└── README.md
```
> Adjust this to match your actual repo layout.

---

## 🚀 Getting Started

### Prerequisites
- ESP32 development board
- DS18B20 temperature sensor, MQ-x gas sensor, voltage divider module
- 16×2 I2C LCD, piezo buzzer, breadboard, jumper wires
- Arduino IDE or PlatformIO
- A Firebase project with Realtime Database enabled

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/Hrishikesh-sinha/Event---Data---Recorder-For---Light-Electric-vehicle-.git
cd Event---Data---Recorder-For---Light-Electric-vehicle-
```

1. Open the firmware project in Arduino IDE / PlatformIO.
2. Install required libraries: `WiFi.h`, `DallasTemperature.h`, `OneWire.h`, `LiquidCrystal_I2C.h`, Firebase ESP32 Client, `ArduinoJson`.
3. Update WiFi SSID/password and Firebase credentials (Database URL + Auth) in the firmware config section.
4. Wire the components according to the [pin table](#-hardware-components) above.
5. Flash the firmware to the ESP32 and open Serial Monitor to verify sensor readings.
6. Open the web dashboard (or Firebase console) to view live data.

---

## 🔭 Future Work

- [ ] Ultra-early precursor detection (off-gassing signatures) for earlier warning lead time
- [ ] Independent power buffering / thermal hardening for hardware survival during propagation
- [ ] Multi-sensor fusion (gas + impedance + pressure + acoustic) to reduce false positives
- [ ] Wireless burst transmission via ESP8266/ESP32 as add-on module for forensic payload escape
- [ ] Field validation under real-world LEV operating conditions

---

## 👥 Contributors

| Name | Reg. No. | Role |
|---|---|---|
| Hrishikesh Anand | 23BCT0124 | Author |
| Alok Singh | 23BCI0158 | Author |
| Yokesh Babu S | — | Faculty Guide |

School of Computer Science and Engineering, VIT University, Vellore, India

## 🔗 Links

- 📁 **Project Drive:** [Google Drive folder](https://drive.google.com/drive/folders/19CHtPH2I57WzOcKTYWXAbDedC10_Tpc5?usp=sharing)

## 📄 Course

Submitted for **Embedded Systems (BCSE305L)**, Winter Semester 2025–26, Vellore Institute of Technology.

## 📜 License

_No license specified yet — add one (e.g., MIT) if you want others to reuse this code._

Made with ⚡ at VIT University
