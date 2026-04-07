# 📡 Wi-Fi & Bluetooth Jammer — Wireless Security Research Device

> A hands-on hardware and cybersecurity project built to understand wireless communication, RF interference, and the real-world attack surface of modern wireless protocols — and how systems can be hardened against them.

---

## ⚠️ Legal Disclaimer

This device was built and tested **exclusively in isolated, controlled lab environments** with no third-party devices or networks affected at any time. Jamming wireless signals is **illegal in most jurisdictions** without explicit authorization. This project exists solely for **educational and defensive security research**. Do not replicate or operate this device outside of a properly shielded, authorized lab environment.

---

## ⚙️ Capabilities & Features

- **Full-spectrum interference** across Wi-Fi (2.4 GHz), Bluetooth Classic, BLE, Zigbee, and RC bands
- **Multi-protocol targeting** — switch between protocols via physical button or OLED menu
- **Passive scanning** — observe signal behavior, device density, and channel occupation patterns
- **Active scanning** — probe and enumerate nearby wireless environments
- **Device spoofing** — study trust models, authentication gaps, and protocol-level weaknesses
- **Attack simulation** — evaluate detection latency and defensive countermeasure effectiveness
- **Live OLED visualization** — system state, active mode, and wireless environment at a glance

---

## 🎯 Research Objectives

The core research question driving this project:

> *How does a defender detect and disrupt wireless attacks before they cause harm — and what gaps exist at the protocol and hardware level that make that harder than it should be?*

Key areas explored:

- **Detection latency** — How quickly do enterprise and consumer APs log and alert on deauth floods and jamming events? Results varied significantly by vendor and firmware version.
- **Protocol trust gaps** — Which 2.4 GHz protocols have the weakest authentication models and why?
- **Channel behavior** — How do devices respond to sudden channel saturation vs. selective interference?
- **Spoofing surface** — What information is exposed during passive beacon/advertisement scanning, and how is it exploited?

---

## 🔧 Hardware Architecture

| Component | Role |
|---|---|
| **ESP32** | Main microcontroller — native dual-mode Wi-Fi + Bluetooth |
| **NRF24L01+ × 3** | Parallel RF modules for multi-channel simultaneous coverage |
| **OLED Display (SH1106 128×64)** | Real-time feedback: active mode, channel status, scan results |
| **Physical Buttons (×5)** | On-device mode switching without serial dependency |
| **Custom Enclosure** | Compact, portable case designed for lab use |

Three NRF24L01+ modules run in parallel to cover multiple frequency channels simultaneously, enabling broad-spectrum signal analysis and generation without sequential scanning delays.

---

## 🔌 Complete Pin Reference

### SPI Bus (Shared across all NRF24L01+ modules)

| Signal | ESP32 GPIO |
|---|---|
| SCK | GPIO 18 |
| MISO | GPIO 19 |
| MOSI | GPIO 23 |

---

### OLED Display (SH1106) — I2C

| Signal | ESP32 GPIO |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | GPIO 22 |
| SDA | GPIO 21 |

---

### NRF24L01+ Module Assignments

Three modules are wired independently on CE/CSN while sharing the SPI bus.

| Module | CE Pin | CSN Pin | Used In |
|---|---|---|---|
| **Radio 1 / A** | GPIO 5 | GPIO 17 | Jammer, BLE Jammer, Blackout, Analyzer, Scanner |
| **Radio 2 / B** | GPIO 16 | GPIO 4 | Jammer, BLE Jammer, Blackout |
| **Radio 3 / C** | GPIO 15 | GPIO 2 | Jammer, BLE Jammer, Blackout |

> **Note:** The Analyzer and Scanner modules use only Radio 1 (GPIO 5 / GPIO 17) in receive mode. The multi-radio parallel setup is leveraged exclusively in active jamming modes.

---

### Button Pin Assignments by Module

Buttons serve different roles depending on the active firmware module. All buttons are configured as `INPUT_PULLUP` — active LOW on press.

#### Main Menu Navigation

These three buttons are always active on the main menu screen to navigate the module list.

| Button | GPIO | Function |
|---|---|---|
| BUTTON_UP | GPIO 26 | Scroll menu up |
| BUTTON_SELECT | GPIO 32 | Confirm / enter selected module |
| BUTTON_DOWN | GPIO 33 | Scroll menu down |

#### Wi-Fi Jammer (`jammer.cpp`)

| Button | GPIO | Function |
|---|---|---|
| BT1 | GPIO 33 | Cycle Wi-Fi channel (1–14) |
| BT2 | GPIO 26 | Toggle jamming ON / OFF |
| BT3 | GPIO 27 | Cycle data rate (250KBPS → 1MBPS → 2MBPS) |
| BT4 | GPIO 25 | Cycle PA level (MIN → LOW → HIGH → MAX) |

#### BLE / Bluetooth Jammer (`blejammer.cpp`)

| Button | GPIO | Function |
|---|---|---|
| MODE_BUTTON | GPIO 26 | Cycle mode: Deactive → BLE → Bluetooth |

#### Blackout Multi-Protocol Jammer (`blackout.cpp`)

| Button | GPIO | Function |
|---|---|---|
| MODE_BUTTON | GPIO 25 | Navigate protocol left (WiFi → NRF24 → Zigbee…) |
| MODE_BUTTON1 | GPIO 27 | Navigate protocol right |
| MODE_BUTTON2 | GPIO 26 | Toggle active / deactive jamming |

#### BLE Scanner (`blescan.cpp`)

| Button | GPIO | Function |
|---|---|---|
| BUTTON_PIN_UP | GPIO 26 | Scroll up through scanned device list |
| BUTTON_PIN_DOWN | GPIO 33 | Scroll down through scanned device list |
| BUTTON_PIN_SELECT | GPIO 27 | Open device detail view |
| BUTTON_PIN_BACK | GPIO 25 | Return to device list from detail view |

#### Wi-Fi Scanner (`wifiscan.cpp`)

| Button | GPIO | Function |
|---|---|---|
| BTN_PIN_UP | GPIO 26 | Scroll up through network list |
| BTN_PIN_DOWN | GPIO 33 | Scroll down through network list |
| BTN_PIN_SELECT | GPIO 27 | Open network detail view |
| BTN_PIN_BACK | GPIO 25 | Return to network list |

#### Apple Device Spoofer (`spoofer.cpp`)

| Button | GPIO | Function |
|---|---|---|
| deviceTypeNextPin | GPIO 27 | Next spoofed device type (AirPods, Beats, etc.) |
| deviceTypePrevPin | GPIO 25 | Previous spoofed device type |
| advTypeNextPin | GPIO 33 | Cycle BLE advertisement type (IND / SCAN / NONCONN) |
| advControlPin | GPIO 26 | Toggle advertising ON / OFF |

#### Settings Menu (`setting.cpp`)

| Button | GPIO | Function |
|---|---|---|
| BUTTON_UP | GPIO 26 | Navigate menu up |
| BUTTON_DOWN | GPIO 33 | Navigate menu down |
| BUTTON_SELECT | GPIO 27 | Toggle / apply selected setting |

---

### Quick GPIO Summary

| GPIO | Role |
|---|---|
| 2 | CSN — Radio 3 |
| 4 | CSN — Radio 2 |
| 5 | CE — Radio 1 |
| 15 | CE — Radio 3 |
| 16 | CE — Radio 2 |
| 17 | CSN — Radio 1 |
| 18 | SPI SCK |
| 19 | SPI MISO |
| 21 | I2C SDA — OLED |
| 22 | I2C SCL — OLED |
| 23 | SPI MOSI |
| 25 | Button: Back / Prev / PA Level |
| 26 | Button: Menu Up / Toggle / Control |
| 27 | Button: Select / Next / Data Rate |
| 32 | Button: Menu Select (main navigation) |
| 33 | Button: Menu Down / Channel / Adv Type |

---

## 📦 Firmware Modules

| Module | File | Description |
|---|---|---|
| **Wi-Fi Jammer** | `jammer.cpp` | 3-radio parallel Wi-Fi band jamming with adjustable channel, data rate, and PA level |
| **BLE / BT Jammer** | `blejammer.cpp` | Targets BLE (ch 2, 26, 80) and Bluetooth Classic (21 channels) |
| **Blackout** | `blackout.cpp` | Multi-protocol jammer — 8 selectable targets: WiFi, BLE, Bluetooth, Zigbee, RC, Video TX, USB Wireless, NRF24 |
| **NRF24 Analyzer** | `analyzer.cpp` | 2.4 GHz channel activity visualizer across 128 channels using carrier detect |
| **NRF24 Scanner** | `scanner.cpp` | Scrolling signal history graph with EEPROM persistence across power cycles |
| **BLE Scanner** | `blescan.cpp` | Scans and lists nearby BLE devices with RSSI, name, and MAC address |
| **Wi-Fi Scanner** | `wifiscan.cpp` | Scans and lists nearby 802.11 networks with SSID, BSSID, RSSI, and channel |
| **Apple Spoofer** | `spoofer.cpp` | Spoofs 17 Apple/Beats device types via randomized BLE advertisements |
| **Sour Apple** | `sourapple.cpp` | Generates rapid randomized Apple proximity pairing pop-up packets |
| **Settings** | `setting.cpp` | OLED-based settings menu: NeoPixel enable/disable, OLED brightness (EEPROM-persisted) |

---

### Blackout Protocol Channel Map

The Blackout module targets specific 2.4 GHz channels per protocol:

| Protocol | Channels Targeted |
|---|---|
| Wi-Fi | 1–12 |
| Bluetooth Classic | 0–2, 4, 6, 8, 22–30, 32–34, 46–52, 74–80 |
| BLE | 2, 26, 80 |
| USB Wireless | 40, 50, 60 |
| Video Transmitter | 70, 75, 80 |
| RC | 1, 3, 5, 7 |
| Zigbee | 11, 15, 20, 25 |
| NRF24 | 76, 78, 79 |

---

### Scanner EEPROM Layout

| Address | Data |
|---|---|
| 0 | NeoPixel enabled flag |
| 1 | OLED brightness value (0–255) |
| 2–129 | NRF24 scanner graph history (128 bytes) |

The scanner saves its rolling signal history graph to EEPROM every **5 seconds**, so the waveform survives power cycles.

---

## 🛠️ Tech Stack & Concepts

```
Hardware       ESP32 · NRF24L01+ (×3) · OLED SH1106 128×64 · Custom PCB layout
Protocols      IEEE 802.11 (Wi-Fi) · Bluetooth Classic · BLE · Zigbee · 433/915 MHz RC
Firmware       Embedded C/C++ · Arduino framework · SPI bus management · EEPROM persistence
Concepts       RF Interference · Spread Spectrum · FHSS · Signal Spoofing · Carrier Detection
Security       Wireless Protocol Analysis · Deauth Attacks · Beacon Flooding · Defensive Countermeasures
```

---

## 🔬 Setup & Usage

> ⚠️ For authorized lab use only. Ensure your environment is RF-shielded or fully isolated.

### Prerequisites

- Arduino IDE or PlatformIO
- ESP32 board package installed
- Libraries: `U8g2`, `RF24`, `Adafruit_NeoPixel`, `WiFi.h`, `BLEDevice`

### Hardware Wiring

#### SPI Bus

| Signal | ESP32 GPIO |
|---|---|
| SCK | GPIO 18 |
| MISO | GPIO 19 |
| MOSI | GPIO 23 |

#### OLED Display (SH1106) — I2C

| Signal | ESP32 GPIO |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | GPIO 22 |
| SDA | GPIO 21 |

#### NRF24L01+ Modules

| Module | CE | CSN | VCC | GND |
|---|---|---|---|---|
| Radio 1 | GPIO 5 | GPIO 17 | 3.3V | GND |
| Radio 2 | GPIO 16 | GPIO 4 | 3.3V | GND |
| Radio 3 | GPIO 15 | GPIO 2 | 3.3V | GND |

#### Buttons

| GPIO | Function |
|---|---|
| GPIO 25 | Back / Prev / PA Level |
| GPIO 26 | Menu Up / Toggle / Control |
| GPIO 27 | Select / Next / Data Rate |
| GPIO 32 | Menu Select (main navigation) |
| GPIO 33 | Menu Down / Channel / Adv Type |

---

## 📸 Hardware

<img width="480" height="723" alt="1_g" src="https://github.com/user-attachments/assets/3b59fafc-c3af-43e5-854f-ce039c1b3a3c" />
<img width="480" height="723" alt="3_g" src="https://github.com/user-attachments/assets/4f6394e8-0bf5-4dcd-b61e-93911154b23c" />

---

## 📚 References & Further Reading

- [ESP32 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)
- [NRF24L01+ Product Specification](https://www.sparkfun.com/datasheets/Components/SMD/nRF24L01Pluss_Preliminary_Product_Specification_v1_0.pdf)
- IEEE 802.11 Standard — Wireless LAN MAC and PHY specifications
- [NIST SP 800-153 — Guidelines for Securing Wireless LANs](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-153.pdf)

---

## 👤 Author

Built by **Tunar Khankishiyev** — hardware hacker, embedded systems & wireless security enthusiast.

- GitHub: [@tunarkhankishiyev](https://github.com/tunarkhankishiyev)
- LinkedIn: [tunar-xankisiyev-154a74328](https://linkedin.com/in/tunar-xankisiyev-154a74328)

---

*Built for learning. Operated responsibly. Shared for the community.*
