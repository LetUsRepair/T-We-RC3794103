# Reverse Engineering the T-We Remote Control (RC3794103 / RC3794104/01BR)

![T-We Remote Control PCB Teardown](IMG_0325.jpg)

## 📌 Overview

This repository documents the reverse engineering process, hardware teardown, pinout analysis, and protocol investigation for the **T-We Remote Control** (supplied with T-We set-top boxes / TV decoders by Telenor). 

While the plastic enclosure is labeled **RC3794103/01BR** (or referenced online as **RC3794014/01BR**), the PCB itself is silkscreened with **RC3794103**.

The main objective of this project is to:
1. Map out the hardware architecture and test pads.
2. Interface with the main System-on-Chip (SoC) via hardware debug/programming pins.
3. Dump and analyze the onboard firmware.
4. Decode both the legacy Infrared (IR) commands and the primary Radio Frequency (RF/BLE) transmission protocols.
5. Explore possibilities for custom firmware or integrating the remote into custom home automation systems (e.g., Home Assistant, ESP32 BLE gateways).

---

## 🛠️ Hardware & Board Specifications

### Key Hardware Components
* **Main Microcontroller / Wireless SoC:** **Telink TLSR8273**
  * **Core:** 32-bit proprietary RISC microcontroller running up to 48 MHz.
  * **Memory:** 512 KB internal Flash, 64 KB SRAM.
  * **Radio:** 2.4 GHz Multi-Standard RF (Bluetooth Low Energy 5.3, Zigbee, RF4CE, 2.4 GHz proprietary).
  * **Features:** Hardware AES-128, integrated power management, PWM channels for IR generation, 14-bit ADC, low-power sleep modes.
  * **Debug Interface:** Telink **SWS** (Single Wire Slave) debug/programming interface.
* **Infrared Emitter:** Front-mounted IR LED driven via transistor switch / PWM channel on the TLSR8273.
* **Power & Charging:** Micro-USB port at the top edge connected to onboard battery charging/regulation circuitry.
* **User Input & Indicators:**
  * Top status LED indicator around the `DECODER` / `TV Power` buttons.
  * Keypad matrix covering remote control buttons.
  * Onboard tactile switch (Reset/Pairing button located near debug header).

### PCB Markings & Markings
* **Silkscreen Identification:** `RC3794103`
* **Serial / Production Box:** `6660 000 0307` / `03071`
* **UL / PCB Manufacturer Markings:** `E319508`, `ZY-4 94V-0`, `P2C-00587A`
* **Date Code:** `4421` (Manufactured around Week 44, 2021)

---

## 📡 Wireless & Transmission Protocols

The remote control operates in a hybrid dual-mode configuration:

### 1. Infrared (IR) Mode
* **Protocol:** Standard **NEC** IR encoding (38 kHz modulation).
* **Usage:** Used before the remote is paired to a decoder, for controlling basic TV power/volume functions, or during factory setup modes.

### 2. Radio Frequency (RF / Bluetooth LE) Mode
* **Protocol:** Bluetooth Low Energy (BLE) / 2.4 GHz proprietary RF protocol.
* **Behavior:** Once paired with the set-top box, all keystrokes and navigation commands are transmitted over RF/BLE for non-line-of-sight operation.
* **Voice Capabilities:** (If equipped) Speech/audio stream compressed and transmitted over BLE GATT services.

---

## 🔌 Pinout & Debug Interface

Near the Micro-USB port on the right side of the PCB, there is an exposed test pad array suitable for probing and flashing.

```
          +--------------------------------------+
          | [USB] [TACT SWITCH]                  |
          |                                      |
          |  (o) (o) (o)                         |
          |  (o) (o) (o)  <- Test Pad Matrix     |
          +--------------------------------------+
```

### Identified / Target Test Points
| Pad / Signal | Functional Description | Telink Pin / Connection |
| :--- | :--- | :--- |
| **SWS** | Single Wire Slave (Telink Debug/Flash) | TLSR8273 SWS Pin |
| **TX** | UART Transmit (Debug Logs) | GPIO / UART TX |
| **RX** | UART Receive (Commands) | GPIO / UART RX |
| **VCC / 3V3** | Supply Voltage | 3.3V Power Rail |
| **GND** | Ground | Common Ground |
| **RST** | Hardware Reset | Reset Pin |

*(Note: Exact pad assignment and pin mapping currently under verification).*

---

## 🕹️ Operational & Pairing Reference Sequences

The factory firmware supports specific key combinations for reset and pairing:

* **RF Pairing Mode (Decoder):**
  * Hold **[MENU]** + **[OK]** simultaneously for ~3 seconds until the `DECODER` power LED flashes.
  * Release to enter Bluetooth pairing mode (lasts ~30 seconds).

* **Factory Reset / Unpair Remote:**
  * Hold **[1]** + **[6]** simultaneously for ~3 seconds until the `DECODER` button stays lit.
  * Press **[9] - [8] - [1]** (or **[9] - [9] - [6]**).
  * The LED flashes twice to confirm all pairing data and TV settings have been wiped.

* **TV Brand Programming Mode:**
  * Hold **[1]** + **[3]** for ~3 seconds until the `TV Power` button lights up.
  * Hold **[MUTE]** or **[TV Power]** until the TV responds, then release to lock in the code.

---

## 🔬 Reverse Engineering Roadmap & Work in Progress

- [x] Initial non-destructive disassembly and PCB visual inspection.
- [x] Identification of primary IC (Telink TLSR8273) and peripheral circuitry.
- [ ] Logic analyzer capture of SWS and UART pins during startup.
- [ ] Flash dump extraction using Telink SWS flasher tools (`tl_sws` / OpenOCD / ESP32 flasher).
- [ ] Memory map analysis & disassembling firmware in Ghidra / IDA Pro.
- [ ] Bluetooth LE packet sniffing (using Wireshark + NRF Sniffer) to reverse engineer GATT key codes.
- [ ] Custom firmware payload / alternative integration testing.

---

## 📄 License & Disclaimer

This project is intended strictly for educational, research, and interoperability purposes. All product names, trademarks, and registered trademarks belong to their respective owners.
