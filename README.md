<div align="center">

# 📺 Remodaz
### 🎛️ DIY Touchscreen Universal IR Remote + Protocol Analyzer

**Built around the ESP32-WROOM-32U · V1**

<p>
  <img src="https://img.shields.io/badge/MCU-ESP32--WROOM--32U-blue?style=for-the-badge&logo=espressif" alt="ESP32">
  <img src="https://img.shields.io/badge/Display-ILI9341-orange?style=for-the-badge" alt="ILI9341">
  <img src="https://img.shields.io/badge/Touch-XPT2046-purple?style=for-the-badge" alt="XPT2046">
  <img src="https://img.shields.io/badge/IR-36kHz-red?style=for-the-badge" alt="IR 36kHz">
  <img src="https://img.shields.io/badge/Storage-MicroSD-green?style=for-the-badge" alt="MicroSD">
</p>

<p>
  <b>Learn.</b> <b>Analyze.</b> <b>Replay.</b> <b>Organize.</b>
</p>

</div>

> 🛠️ **DIY hardware project** — ESP32 based touchscreen remote that can learn raw IR signals, replay them with hold/repeat behavior, and analyze common IR protocols.

Version **V1** · Board: ESP32-WROOM-32U
<div align="center">
  
https://github.com/user-attachments/assets/8ca75e5a-cfd0-48c6-a7a4-24c1e1e44019

</div>

---

## ✨ At a Glance

| 🎯 Capability | 📌 What it does |
|---|---|
| 🎛️ **Universal Remote** | Learn and replay IR buttons from almost any IR remote |
| 🔬 **IR Analyzer** | Identify protocols, inspect raw timings, decode supported frames |
| 💾 **SD Database** | Store devices, key labels, raw captures and analyzer exports |
| 👆 **Touch UI** | XPT2046 resistive touchscreen with 4-point calibration |
| 🔁 **Hold / Repeat** | Reproduce real remote-style press-and-hold behavior |

| ⚙️ **Display Settings** | Brightness, timeout and 0°/180° rotation |

> 💡 **Core idea:** the remote stores the *raw IR timing waveform*, so replay does not depend on the Analyzer knowing the protocol.

---

## 🧩 System Overview

```text
                         ┌──────────────────────────┐
                         │      ESP32-WROOM-32U     │
                         │                          │
                         │  Touch UI + IR Engine    │
                         │  SD Database + Analyzer  │
                         └────────────┬─────────────┘
                                      │
             ┌────────────────────────┼────────────────────────┐
             │                        │                        │
             ▼                        ▼                        ▼
      ┌─────────────┐         ┌─────────────┐          ┌─────────────┐
      │  ILI9341    │         │  XPT2046    │          │  MicroSD    │
      │  2.4" TFT   │         │ Touch Panel │          │   FAT32     │
      │   240×320   │         │             │          │  Database   │
      └─────────────┘         └─────────────┘          └─────────────┘
             │                        │                        │
             └────────────────────────┼────────────────────────┘
                                      │
                           ┌──────────┴──────────┐
                           │                     │
                           ▼                     ▼
                    ┌─────────────┐       ┌─────────────┐
                    │  TSOP RX    │       │   IR LED TX │
                    │  GPIO 22    │       │   GPIO 27   │
                    └─────────────┘       └─────────────┘
```

---

## 🔌 Connection Quick Reference

### Display — ILI9341

| ESP32 | ILI9341 | Function |
|:---:|:---:|---|
| `GPIO 14` | SCK | SPI Clock |
| `GPIO 13` | MOSI | SPI Data Out |
| `GPIO 12` | MISO | SPI Data In |
| `GPIO 15` | CS | Chip Select |
| `GPIO 2` | DC | Data / Command |
| `GPIO 21` | BL | PWM Backlight |
| `3.3V` | VCC | Power |
| `GND` | GND | Ground |
> 💡 **Note:** LED backlight control is managed via a transistor NPN S8050 transistor, with the base controlled through a 1kΩ resistor  This pin is PWM-controlled for brightness adjustment.
### Touch — XPT2046
| ESP32 | XPT2046 | Function |
|:---:|:---:|---|
| `GPIO 25` | CLK | SPI Clock |
| `GPIO 32` | MOSI | SPI Data Out |
| `GPIO 39` | MISO | SPI Data In |
| `GPIO 33` | CS | Chip Select |
| `GPIO 36` | IRQ | Touch Interrupt |
| `3.3V` | VCC | Power |
| `GND` | GND | Ground |

### Storage — MicroSD

| ESP32 | MicroSD | Function |
|:---:|:---:|---|
| `GPIO 18` | SCK | SPI Clock |
| `GPIO 23` | MOSI | SPI Data Out |
| `GPIO 19` | MISO | SPI Data In |
| `GPIO 5` | CS | Chip Select |
| `3.3V` | VCC | Power |
| `GND` | GND | Ground |

### IR + Feedback

| ESP32 | Connection | Function |
|:---:|:---|---|
| `GPIO 27` | IR LED / driver | 📡 36 kHz IR transmit |
| `GPIO 22` | TSOP OUT | 👂 IR receive / learning |
| `GPIO 26` | Status LED | 💡 Key transmit feedback |
| `GPIO 0` | Optional LED | 👁️ Capture/listening indicator |

> ⚠️ **Hardware note:** GPIO 0 is an ESP32 boot-strapping pin. The firmware uses it after boot as an optional indicator. Leave it unwired if you only have one LED.

---


## What Is This

Remodax is a touchscreen universal remote built usingan **ESP32-WROOM-32U**: a **2.4" ILI9341 TFT,  XPT2046** resistive touch panel, a **microSD** card, a **TSOP IR** receiver, an **IR LED**, and a status LED.


.**It's a general-purpose universal remote and IR tool** — buttons from other remotes can be learned, organized into named devices, replayed with press-and-hold behavior, and inspected with the built-in IR Analyzer.

This is a personal/DIY build. Every peripheral is individually wired to the ESP32; there is no single custom PCB or pre-built touchscreen development board involved.

---

## Why Raw IR Capture Instead of Protocol Encoding

The universal remote stores and replays the **exact mark/space timings** captured from the original remote rather than converting every signal into a protocol-specific representation.

This means:

- It can learn a remote even when its protocol is not implemented by the Analyzer.
- Replay is timing-based rather than reconstructed from decoded address/command values.
- The same mechanism can replay signals from different IR protocol families.

The trade-off is storage: each learned key stores its complete raw pulse train, up to the firmware's supported pulse count, instead of only a few encoded bytes.

The **IR Analyzer** is therefore complementary to the universal remote. It identifies the protocol family and decodes address/command information where supported.

---

## ✨ Feature Highlights

### 🎛️ Universal Remote

- **Learn any button** — point the original remote at the TSOP receiver and capture its raw mark/space timings.
- **Press-and-hold repeat** — held keys retransmit after a ~220 ms initial delay at ~110 ms repeat intervals.
- **Reorder devices and keys** — ordering is persisted to the SD card.
- **Test before save** — a newly learned key can be transmitted immediately before committing it to the database.


### 🔬 IR Analyzer

- **Protocol identification** using leader-burst timing signatures.
- Identifies **NEC, NEC repeat, Samsung, Sony SIRC, RC5, RC6, Panasonic/Kaseikyo, and JVC**.
- **Address/command decoding** for available protocols.
- **Checksum verification** where the protocol provides redundant/inverted bytes.
- **Raw timing viewer** with scrolling.
- **Five-entry in capture history**.
- **SD export** of protocol, decoded information, pulse count, and raw timings.
- **Raw replay** of the currently displayed capture, even when the protocol is not fully decoded.

### 👆 Touch & Display

- Four-point touch calibration with edge extrapolation.
- Calibration stored primarily in ESP32 , with an SD compatibility/backup file.
- PWM backlight control from 5–255.
- Screen timeout: 10 s, 20 s, 30 s, 60 s, or NEVER.
- Wake-on-touch behavior.
- Live 0° / 180° display rotation with the same calibration data.
- Settings use live preview with **SAVE** / **BACK** behavior.

---

## 🧰 Hardware Platform

| Component | Part | Notes |
|---|---|---|
| MCU | ESP32-WROOM-32U | Wired point-to-point on protoboard/perfboard |
| Display | ILI9341, 2.4", 240×320 | SPI |
| Touch | XPT2046, resistive | Separate pin configuration from the display |
| Storage | MicroSD (FAT32) | Device/key database and IR Analyzer exports |
| IR Receiver | TSOP IR receiver module | Demodulated, active-low output |
| IR Transmitter | IR LED | Driven using the ESP32 LEDC 36 kHz carrier |
| Feedback LED | Status/feedback LED | GPIO 26 for key feedback; GPIO 0 is used as an optional capture/listening indicator |

Unlike an off-the-shelf touchscreen development board, this project has:

- No onboard RGB LED
- No onboard speaker/buzzer
- No onboard LDR/light sensor
- No integrated display/touch/SD breakout board

---

## 🛒 Required Parts

- ESP32-WROOM-32U module
- ILI9341 2.4" TFT panel
- XPT2046 resistive touch panel
- MicroSD card + slot, FAT32 formatted
- TSOP IR receiver module
- IR LED + current-limiting resistor
- Driver transistor **S8050** used if higher IR LED current/range is required
- Status LED(s) + current-limiting resistor
- An original IR remote for learning buttons from

### IR Path Notes

The IR transmitter uses the ESP32 LEDC peripheral to generate a gated **36 kHz carrier**. An external IR driver IC is not required for a basic build, although a transistor driver is recommended when higher LED current/range is desired.

The TSOP receiver is a demodulated, active-low digital output. The firmware captures its edges directly rather than measuring the 36 kHz carrier itself.

GPIO 0 is an ESP32 boot-strapping pin. The firmware uses it as a secondary LED output after boot; if no LED is connected there, it simply remains unwired.

---

## 🔌 Wiring

Every peripheral is individually wired to the ESP32-WROOM-32U.

### ILI9341 TFT Display

| ESP32 GPIO | TFT Pin | Description |
|---|---|---|
| GPIO 14 | SCK | SPI clock |
| GPIO 13 | MOSI | SPI data |
| GPIO 12 | MISO | SPI data |
| GPIO 15 | CS | Chip select |
| GPIO 2 | DC | Data / command |
| Board / TFT_eSPI | RST | Display reset |
| GPIO 21 | BL | PWM backlight |
| 3.3V | VCC | Power |
| GND | GND | Ground |

### XPT2046 Touch

| ESP32 GPIO | XPT2046 Pin | Description |
|---|---|---|
| GPIO 25 | CLK | SPI clock |
| GPIO 32 | MOSI | SPI data |
| GPIO 39 | MISO | SPI data |
| GPIO 33 | CS | Chip select |
| GPIO 36 | IRQ | Touch interrupt / pen detect |
| 3.3V | VCC | Power |
| GND | GND | Ground |

### MicroSD

| ESP32 GPIO | SD Pin | Description |
|---|---|---|
| GPIO 18 | SCK / CLK | SPI clock |
| GPIO 23 | MOSI / DI | SPI data |
| GPIO 19 | MISO / DO | SPI data |
| GPIO 5 | CS | Chip select |
| 3.3V | VCC | Power |
| GND | GND | Ground |

### IR Receiver

| ESP32 GPIO | TSOP Pin | Description |
|---|---|---|
| GPIO 22 | OUT | Demodulated IR signal, active-low |
| 3.3V | VCC | Power |
| GND | GND | Ground |

### IR Transmitter

| ESP32 GPIO | IR LED Circuit | Description |
|---|---|---|
| GPIO 27 | LED anode via resistor / transistor driver | 36 kHz LEDC carrier |

S8050 NPN transistor is used, GPIO 27 drives the transistor through an 1K base resistor rather than sourcing the IR LED current directly.

### Feedback LEDs

| ESP32 GPIO | Function | Description |
|---|---|---|
| GPIO 26 | Press feedback | Flashes on key transmission |
| GPIO 0 | Listening / capture indicator | Optional second LED |

### SPI Bus Note

The TFT uses GPIO 12/13/14. SD uses GPIO 18/19/23. Touch has its own CLK/MOSI/MISO/CS pin assignment but shares the VSPI peripheral with the SD interface.

---

## 🗂️ Menu Tree

```text
REMODAZ 
│
├── Devices                    Home screen — saved remotes
│   ├── + ADD                  Create a new device
│   ├── EDIT                   Reorder / rename / delete devices
│   └── <device>
│       ├── Key grid           10 keys/page, tap to transmit
│       │                       hold to repeat
│       ├── + KEY              Learn a new button
│       └── EDIT
│           ├── ARRANGE        Reorder keys
│           ├── LEARN          Re-learn selected key
│           ├── RENAME         Rename selected key
│           └── DELETE         Delete selected key
│
├── IR Analyzer                Standalone capture + protocol decode
│   ├── CAPTURE                Listen for a raw IR frame
│   ├── PREV / NEXT            Browse last 5 captures
│   ├── TEST                   Replay current capture
│   ├── SAVE                   Export capture + decode to SD
│   └── CLEAR                  Wipe capture history
│
└── Display (Settings)
    ├── Backlight              PWM brightness
    ├── Screen Timeout         10s / 20s / 30s / 60s / Never
    ├── Rotation               Normal / 180°
    └── Calibrate Touch        4-point calibration
```

---
<div align="center">

### 📺 Learn IR. Analyze IR. Control Everything.

</div>


 
