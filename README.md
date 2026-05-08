# 🌞 Smart Sunburn Warning Patch

> A compact, wearable embedded system for real-time UV exposure monitoring and personalised sunburn risk alerting.

**Author:** Siddhant Deore  
**Institution:** International Institute of Information Technology Bangalore (IIITB)  
**Course:** Electronics Systems Packaging  
**ID:** IMT2023539  

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Hardware Components](#-hardware-components)
- [Sensor Specifications](#-sensor-specifications)
- [Risk Scoring Algorithm](#-risk-scoring-algorithm)
- [PCB Design](#-pcb-design)
- [Hardware Design Choices](#-hardware-design-choices)
- [Schematic Overview](#-schematic-overview)
- [Firmware](#-firmware)
- [Getting Started](#-getting-started)
- [Applications](#-applications)
- [Results](#-results)
- [Future Work](#-future-work)
- [References](#-references)

---

## 🔍 Overview

The **Smart Sunburn Warning Patch** is a wearable embedded system that continuously fuses data from a UV sensor, a precision temperature sensor, and an ambient light sensor to compute a real-time skin-exposure risk score. An **ESP32-S3** microcontroller performs on-board sensor fusion and drives a three-level LED alert system (green / amber / red) together with a vibration motor for haptic feedback.

The PCB measures **39 mm × 28 mm**, operates from a single **3.3 V** regulated rail derived from a 3.7 V Li-Po battery, and incorporates a mixed-signal layout strategy to minimise noise coupling.

---

## ❗ Problem Statement

| Challenge | Description |
|-----------|-------------|
| ☀️ **UV Radiation Risk** | Excessive UV exposure is a leading cause of sunburn, premature ageing, and long-term skin damage including melanoma. |
| 👁️ **Invisible Hazard** | Users cannot perceive UV intensity in real time. Cloud cover and reflective surfaces create misleading conditions. |
| 📡 **No Wearable Solution** | Existing tools (UV index apps) are coarse, location-based, and not personalised. A compact, body-worn monitor is missing. |

---

## ✨ Key Features

- 📡 Real-time UV radiation monitoring (ML8511, 280–390 nm)
- 🌡️ Skin surface temperature tracking (TMP117, ±0.1 °C accuracy)
- 💡 Ambient light context sensing (BH1750, 1–65,535 lux)
- 🚦 Three-level LED alert system (Green / Amber / Red)
- 📳 Haptic vibration alert at high-risk levels
- ⚡ Low-power, single 3.3 V rail design
- 🔧 Compact 39 × 28 mm PCB footprint — wearable-ready
- 🧠 On-device weighted sensor fusion risk algorithm

---

## 🏗️ System Architecture

The patch follows a **Sense → Convert → Process → Alert** pipeline:

```
Battery (3.7V Li-Po)
        │
   LDO Regulator (3.3V)
        │
   ┌────┴───────────────────────────────┐
   │                                    │
ML8511 (UV)          ┌──── ESP32-S3 MCU ────┐
   │ ADC             │                      │
   └────────────────►│   Sensor Fusion      │──── GPIO ──► LED Alert
                     │   Risk Algorithm     │
TMP117 (Temp)        │                      │──── GPIO ──► Vibration Motor
   │ I²C             │                      │
   └────────────────►│                      │
                     └──────────────────────┘
BH1750 (Light)              ▲
   │ I²C                    │
   └────────────────────────┘
```

**Interface Key:**
- `ADC` — Analog (ML8511 → ESP32-S3)
- `I²C` — Digital at 400 kHz (TMP117 + BH1750, shared bus)
- `GPIO` — Digital output (LED driver + vibration motor transistor)

---

## 🔩 Hardware Components

| Component | Part Number | Role |
|-----------|-------------|------|
| Microcontroller | ESP32-S3-WROOM-1-N16 | Central processing, sensor fusion, alert control |
| UV Sensor | ML8511 (ROHM) | Primary UV irradiance measurement |
| Temperature Sensor | TMP117 (Texas Instruments) | Skin surface temperature |
| Ambient Light Sensor | BH1750FVI-TR (ROHM) | Environmental context / false-alert reduction |
| Alert Output | RGB LED | Visual risk level indication |
| Alert Output | Vibration Motor | Haptic feedback at high-risk level |
| Power | 3.7 V Li-Po Cell | Primary energy source |
| Regulator | LDO (3.3 V output) | Regulated supply for all ICs |

---

## 📐 Sensor Specifications

### ☀️ ML8511 — UV Sensor

| Parameter | Value |
|-----------|-------|
| Interface | ADC (Analog, 0–3.3 V) |
| UV Spectral Range | 280–390 nm (UVA + UVB) |
| ADC Resolution | 12-bit (ESP32-S3 SAR ADC) |
| Package | SOT-23-5 |
| Calibration Required | No |
| Decoupling | C7: 100 nF + C8: 1 nF (parallel on VDD) |

**Justification:** Directly measures UV radiation — the primary driver of erythema (sunburn). Simple ADC interface with low power consumption and a compact footprint ideal for wearable integration.

---

### 🌡️ TMP117 — Temperature Sensor

| Parameter | Value |
|-----------|-------|
| Interface | I²C (up to 400 kHz) |
| Accuracy | ±0.1 °C over full range |
| Resolution | 0.0078 °C per LSB (16-bit output) |
| I²C Address | 0x48 (ADD0 tied to GND) |
| ALERT Pin | Open-drain, pulled up via R12 (5 kΩ) |
| Calibration Required | No |

**Justification:** Skin temperature reflects cumulative thermal exposure. UV alone does not determine sunburn severity — high accuracy improves the risk prediction model fidelity.

---

### 💡 BH1750 — Ambient Light Sensor

| Parameter | Value |
|-----------|-------|
| Interface | I²C (400 kHz) |
| Measurement Range | 1–65,535 lux |
| Resolution | 16-bit |
| I²C Address | 0x23 (ADDR tied to GND) |
| Calibration Required | No |
| Decoupling | C10: 100 nF (VCC), C9: 100 nF (I²C interface) |

**Justification:** Provides environmental context (indoor vs. outdoor) to validate UV sensor readings and reduce false alerts in shaded or overcast conditions.

---

### 📊 Sensor Summary Table

| Parameter | ML8511 | TMP117 | BH1750 |
|-----------|--------|--------|--------|
| Interface | ADC (analog) | I²C | I²C |
| Range / Accuracy | 0–3.3 V out | ±0.1 °C | 1–65,535 lux |
| Resolution | 12-bit (ADC) | 0.0078 °C | 16-bit |
| Bus Speed | — | 400 kHz | 400 kHz |
| I²C Address | — | 0x48 | 0x23 |
| Calibration Required | No | No | No |

---

## 🧮 Risk Scoring Algorithm

The firmware implements a **weighted multi-parameter risk score**:

```
R = w_UV · û + w_T · t̂ + w_L · l̂
```

Where:
- `û` = normalised UV index
- `t̂` = temperature deviation from 25 °C baseline
- `l̂` = normalised ambient lux value
- `w_UV, w_T, w_L` = empirically tuned weighting coefficients (UV receives highest weight)

### Risk Level Thresholds

| Risk Level | UV Index | Skin Temp | Alert Output |
|------------|----------|-----------|--------------|
| 🟢 Low | < 3 | Normal | Green LED |
| 🟡 Moderate | 3 – 6 | Elevated | Amber LED |
| 🔴 High | > 6 | High | Red LED + Vibration |

---

## 🖥️ PCB Design

The PCB was designed in **KiCad** as a two-layer FR4 board.

| Specification | Value |
|---------------|-------|
| Board Dimensions | 39 mm × 28 mm |
| Layers | 2 (FR4) |
| Power Rail | 3.3 V (single regulated rail) |
| Deployment | Upper arm / shoulder adhesive patch |

### Layout Strategy

- **Compact Placement:** All ICs arranged within a 39 × 28 mm footprint for wearable compatibility.
- **Analog/Digital Separation:** ML8511 analog section isolated from the digital I²C area to minimise noise coupling and preserve ADC accuracy.
- **I²C Routing:** Short, matched-length SDA/SCL traces routed away from power plane switching transients.
- **Decoupling Placement:** 100 nF capacitors placed within 1 mm of each IC VCC pin.
- **Power Routing:** Wide copper pours (≥ 0.5 mm) on 3.3 V rail to minimise resistive voltage drop below 10 mV at full load.
- **Component Placement:** ESP32-S3 module centred on the board; ML8511 positioned at the edge furthest from the I²C cluster to maximise analog-digital separation.

---

## 🔧 Hardware Design Choices

| # | Design Choice | Rationale |
|---|---------------|-----------|
| 01 | **Single 3.3 V Power Rail** | All components operate at 3.3 V — eliminates level-shifting circuitry, simplifies BOM. |
| 02 | **Mixed-Signal PCB Partitioning** | Analog front-end (ML8511 → ADC) separated from I²C digital peripherals to minimise interference. |
| 03 | **I²C Bus Topology** | TMP117 and BH1750 share SDA/SCL — reduces GPIO usage and wiring complexity. |
| 04 | **Pull-Up Resistors** | 4.7 kΩ on SDA/SCL (R1 + R8 at 2.2 kΩ parallel ≈ 1.1 kΩ effective) — meets NXP 400 kHz fast-mode spec. |
| 05 | **Decoupling Capacitors** | 100 nF C0G/X7R ceramics within 1 mm of each IC VCC pin — suppresses high-frequency switching transients. |
| 06 | **GPIO Alert Control via Transistors** | NPN transistor drivers protect MCU GPIO pins from overcurrent when driving LED and vibration motor. |

---

## 📄 Schematic Overview

The root schematic is organised into **five hierarchical sheets**:

1. **Power Management** — LDO regulator, battery input, 3.3 V distribution
2. **ESP32-S3 Core** — MCU with ADC and I²C net assignments, reset circuit, boot mode buttons
3. **ML8511 UV Front-End** — Analog UV sensor with ADC routing and decoupling
4. **TMP117 / BH1750 I²C Cluster** — Shared I²C bus sensors with pull-ups and decoupling
5. **Alert System** — LED driver and vibration motor via NPN transistor outputs

**Key Nets:** `VDD`, `GND`, `I2C{SDA SCL}`, `ADC`, `GPIO0-2`, `ALERT`

---

## 💾 Firmware

The firmware runs on the ESP32-S3 using the Arduino / ESP-IDF framework and implements the following loop:

```
┌─────────────────────────────────────────────────────┐
│                  Main Firmware Loop                  │
│                                                      │
│  1. Read ML8511 ADC → convert to UV Index            │
│  2. Read TMP117 via I²C → get skin temperature       │
│  3. Read BH1750 via I²C → get ambient lux            │
│  4. Normalise sensor values                          │
│  5. Compute risk score R = w_UV·û + w_T·t̂ + w_L·l̂  │
│  6. Evaluate risk threshold                          │
│  7. Drive LED and vibration motor via GPIO           │
│  8. (Optional) Log data over Serial / BLE            │
└─────────────────────────────────────────────────────┘
```

### Pin Assignments (ESP32-S3)

| Signal | ESP32-S3 GPIO | Description |
|--------|--------------|-------------|
| `ADC` (ML8511 OUT) | ADC-capable GPIO | UV sensor analog input |
| `I2C_SDA` | IO12 | Shared I²C data |
| `I2C_SCL` | IO17 | Shared I²C clock |
| `GPIO0` | GPIO0 | Onboard LED (D6) |
| `GPIO1` | GPIO1 | LED alert driver |
| `GPIO2` | GPIO2 | Vibration motor driver |
| `EN` | EN | Reset (SW1, R16, C1) |

---

## 🚀 Getting Started

### Prerequisites

- [KiCad 7+](https://www.kicad.org/) — for schematic and PCB files
- [Arduino IDE 2+](https://www.arduino.cc/en/software) or [ESP-IDF v5+](https://idf.espressif.com/)
- ESP32-S3 board support package installed

### Hardware Setup

1. Fabricate the PCB using the provided Gerber files (`/hardware/gerbers/`)
2. Solder components per the BOM (`/hardware/BOM.csv`)
3. Connect the 3.7 V Li-Po cell to the battery connector
4. Verify 3.3 V on the power rail before connecting sensors

### Flashing Firmware

```bash
# Clone the repository
git clone https://github.com/<your-username>/SmartSunburnPatch.git
cd SmartSunburnPatch/firmware

# Using Arduino IDE
# Open firmware/sunburn_patch/sunburn_patch.ino
# Select board: ESP32S3 Dev Module
# Upload

# Using ESP-IDF
idf.py set-target esp32s3
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

### Dependencies (Arduino)

```cpp
#include <Wire.h>           // I²C bus
#include <BH1750.h>         // Ambient light sensor
#include <SparkFun_TMP117.h> // Temperature sensor
// ML8511: direct analogRead() — no library required
```

---

## 🏥 Applications

| Domain | Use Case |
|--------|----------|
| 🌞 **Personal Health** | Hikers, beach-goers, and outdoor enthusiasts receive personalised alerts before sunburn onset. |
| ⚒️ **Occupational Safety** | Construction workers, agricultural labourers, and outdoor athletes who face prolonged UV exposure. |
| 🏥 **Dermatology & Healthcare** | Clinicians can track cumulative UV dose for photo-sensitive patients or post-phototherapy monitoring. |

---

## 📊 Results

### Functional Verification

Verification was carried out in three stages:

1. **Individual Sensor Bench Test** — ML8511 output verified against a calibrated UV reference lamp; TMP117 cross-validated with a calibrated thermocouple; BH1750 compared against a reference luxmeter.
2. **Full Firmware Pipeline** — Simulated sensor readings across all three risk zones confirmed correct LED and vibration motor outputs.
3. **Outdoor Field Test** — Patch worn on upper forearm for 30 minutes during peak solar hours (10:00–14:00), sensor logs captured via serial interface and compared against a co-located commercial UV index meter.

### Key Performance Metrics

| Sensor | Parameter | Result |
|--------|-----------|--------|
| ML8511 | UV index linearity | R² to be updated post-characterisation |
| TMP117 | Temperature accuracy | ±0.1 °C (per datasheet, validated) |
| BH1750 | Lux linearity | R² to be updated post-characterisation |
| Alert System | GPIO response latency | Well within sunburn induction timescale (minutes–hours) |

---

## 🔮 Future Work

- [ ] **BLE Connectivity** — Real-time data streaming to a companion mobile application
- [ ] **UV Dose Logging** — On-device cumulative UV dose accumulation with personalised exposure budgets based on Fitzpatrick skin type
- [ ] **Pulse Oximetry** — Secondary erythema detection via microvascular change sensing
- [ ] **Flexible Substrate** — Miniaturisation to a thinner, flex PCB for improved adhesive patch comfort during extended wear
- [ ] **Wi-Fi OTA Updates** — Over-the-air firmware update support via ESP32-S3 Wi-Fi

---

## 📚 References

1. World Health Organization, "Ultraviolet radiation and health," WHO Fact Sheet, Geneva, Switzerland, 2022. [Online]. Available: https://www.who.int/news-room/fact-sheets/detail/ultraviolet-radiation

2. B. L. Diffey, "Sources and measurement of ultraviolet radiation," *Methods*, vol. 28, no. 1, pp. 4–13, Sep. 2002, doi: 10.1016/S1046-2023(02)00204-9.

3. S. Bhatt, R. Gupta, and A. Sharma, "Wearable UV sensors for personalised exposure monitoring: A review of architectures and applications," *IEEE Sensors Journal*, vol. 22, no. 8, pp. 7431–7443, Apr. 2022, doi: 10.1109/JSEN.2022.3155047.

4. ROHM Semiconductor, "ML8511 UV Sensor Datasheet," Rev. E, 2017. [Online]. Available: https://fscdn.rohm.com/en/products/databook/datasheet/ic/sensor/uv/ml8511-e.pdf

5. Espressif Systems, "ESP32-S3 Technical Reference Manual," v1.4, 2023. [Online]. Available: https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf

6. NXP Semiconductors, "I²C-bus specification and user manual," UM10204, Rev. 7.0, Oct. 2021. [Online]. Available: https://www.nxp.com/docs/en/user-guide/UM10204.pdf

---

## 📜 License

This project is released for academic and educational purposes under IIIT Bangalore's Electronics Systems Packaging course (IMT2023539).

---

<p align="center">
  Made with ☀️ by <strong>Siddhant Deore</strong> · IIIT Bangalore · Apr 2026
</p>
