<div align="center">

# 📺 Remodaz
### Universal IR Remote + Protocol Analyzer 

**Built around the ESP32-WROOM-32U · Version 1.1.3**

<p>
  <img src="https://img.shields.io/badge/MCU-ESP32--WROOM--32U-blue?style=for-the-badge&logo=espressif" alt="ESP32">
  <img src="https://img.shields.io/badge/Display-ILI9341-orange?style=for-the-badge" alt="ILI9341">
  <img src="https://img.shields.io/badge/Touch-XPT2046-purple?style=for-the-badge" alt="XPT2046">
  <img src="https://img.shields.io/badge/IR-36kHz-red?style=for-the-badge" alt="IR 36kHz">
  <img src="https://img.shields.io/badge/Storage-MicroSD-green?style=for-the-badge" alt="MicroSD">
</p>

https://github.com/user-attachments/assets/01d12ce1-19a3-468a-ab67-a8e50f781e69

</div>

> 🛠️ **Project** — built by wiring an **ESP32-WROOM-32U** to a 2.4" ILI9341 (240 × 320 pixels) touchscreen display, an XPT2046 touch panel, a microSD card, and an IR receiver/transmitter. 


## At a Glance

| Capability | What it does |
|---|---|
| 🎛️ **Universal Remote** | Learn and replay IR buttons from almost any remote |
| 🔬 **IR Analyzer** | Identify protocols, inspect raw timings, decode supported frames |
| 💾 **SD Database** | Devices, key labels, raw captures, analyzer exports |
| 👆 **Touch UI** | XPT2046 resistive touchscreen with 4-point calibration |
| 🔁 **Hold / Repeat** | Real remote-style press-and-hold behavior |
| ⚙️ **Display Settings** | Brightness, timeout, 0°/180° rotation |

> 💡 The remote stores the **raw IR timing waveform**, so replay never depends on the Analyzer knowing the protocol.

---

## Table of Contents

- [Hardware & Wiring](#hardware--wiring)
- [Menu Tree](#menu-tree)
- [Installation](#installation)
- [Using the Universal Remote](#using-the-universal-remote)
- [IR Analyzer](#ir-analyzer)
- [Display Settings & Touch Calibration](#display-settings--touch-calibration)
- [SD Card Layout](#sd-card-layout)
- [Troubleshooting](#troubleshooting)
- [Project Phase & Credits](#project-phase--credits)

---
## Hardware & Wiring

It's assembled from individual parts — ESP32-WROOM-32U, a 2.4" ILI9341 display, an XPT2046 touch panel, a microSD module, and an IR receiver/transmitter — all wired to the ESP module.

| Component | Part | Notes |
|---|---|---|
| MCU | ESP32-WROOM-32U | Bare module, wired point-to-point on protoboard/perfboard |
| Display | ILI9341, 2.4", 240×320 | Individually wired over SPI |
| Touch | XPT2046, resistive | Own pin set, shares the SD's SPI peripheral |
| Storage | MicroSD (FAT32) | Individually wired module — device/key database + Analyzer exports |
| IR Receiver | TSOP module | Demodulated, active-low output |
| IR Transmitter | IR LED | Driven via ESP32 LEDC 36kHz carrier |
| Feedback LED | Status LED | GPIO 26 (press feedback) + optional GPIO 0 (listening/capture indicator) |


### Display — 2.4" ILI9341 → ESP32-WROOM-32U

| ESP32 Pin | ILI9341 Pin | Function |
|:---:|:---:|---|
| `GPIO 14` | SCK | SPI Clock |
| `GPIO 13` | MOSI | SPI Data Out |
| `GPIO 12` | MISO | SPI Data In |
| `GPIO 15` | CS | Chip Select |
| `GPIO 2` | DC | Data / Command |
| `GPIO 21` | BL | PWM Backlight |
| `3.3V` | VCC | Power |
| `GND` | GND | Ground |

> 💡 **Note:** LED backlight control is managed via an NPN **S8050** transistor, with the base controlled through a **1kΩ resistor**. This pin is PWM-controlled for brightness adjustment. IR Led Driver has the same wiring.

### Touch — XPT2046 → ESP32-WROOM-32U

| ESP32 Pin | XPT2046 Pin | Function |
|:---:|:---:|---|
| `GPIO 25` | CLK | SPI Clock |
| `GPIO 32` | MOSI | SPI Data Out |
| `GPIO 39` | MISO | SPI Data In |
| `GPIO 33` | CS | Chip Select |
| `GPIO 36` | IRQ | Touch Interrupt |
| `3.3V` | VCC | Power |
| `GND` | GND | Ground |

### Storage — MicroSD Module → ESP32-WROOM-32U

| ESP32 Pin | MicroSD Pin | Function |
|:---:|:---:|---|
| `GPIO 18` | SCK | SPI Clock |
| `GPIO 23` | MOSI | SPI Data Out |
| `GPIO 19` | MISO | SPI Data In |
| `GPIO 5` | CS | Chip Select |
| `3.3V` | VCC | Power |
| `GND` | GND | Ground |

### IR Receiver

| ESP32 GPIO | TSOP Pin | Function |
|---|---|---|
| GPIO 22 | OUT | 👂 IR receive / learning |
| 3.3V | VCC | Power |
| GND | GND | Ground |

### IR Transmitter

| ESP32 GPIO | IR LED Pin | Function |
|---|---|---|
| GPIO 27 | LED anode via resistor / transistor driver | 📡 36 kHz IR transmit |

S8050 NPN transistor is used, GPIO 27 drives the transistor through an 1K base resistor rather than sourcing the IR LED current directly.
**IR LED driver / LCD Backlight (PWM) GPIO 21** Shares Same Wiring.
> <img width="300" height="180" alt="image" src="https://github.com/user-attachments/assets/4093d861-0c2e-474a-9624-c7e168f4510c" />

### Feedback LEDs

| ESP32 GPIO | Function |
|---|---|
| GPIO 26 |💡 Key transmit feedback |
| GPIO 0 | 👁️ Capture/listening indicator |

> ⚠️ **Hardware note:** GPIO 0 is an ESP32 boot-strapping pin. The firmware uses it after boot as an optional indicator. Leave it unwired if you need only one LED.
> Use **330Ω** to protect led current.
<img width="300" height="160" alt="image" src="https://github.com/user-attachments/assets/8e13fe6e-3d97-4ccc-9f3d-283dfbd7b433" />

---

## Menu Tree

```text
REMODAZ
│
├── Devices                    Home screen — saved remotes
│   ├── + ADD / EDIT           Create, reorder, rename, delete devices
│   └── <device>
│       ├── Key grid           10 keys/page — tap to transmit, hold to repeat
│       ├── + KEY              Learn a new button
│       └── EDIT                ARRANGE / LEARN / RENAME / DELETE
│
├── IR Analyzer                Standalone capture + protocol decode
│   ├── CAPTURE / TEST / SAVE / CLEAR
│   └── PREV / NEXT            Browse last 5 captures
│
└── Display (Settings)
    ├── Backlight / Timeout / Rotation
    └── Calibrate Touch        4-point calibration
```

---

## Installation

Get the firmware onto your board — flashing a pre-built binary from the repository.

### Flash a Pre-Built Binary

1. Grab the latest `firmware.bin` from the [Releases](../../releases) page.
2. Install `esptool.py` (Espressif's official flashing tool, cross-platform):
   ```bash
   pip install esptool
   ```
3. Put the board into bootloader mode if it doesn't drop in automatically (hold **BOOT**, tap **EN/RST**, release **BOOT**).
4. Flash:
   ```bash
   esptool.py --chip esp32 --port <YOUR_PORT> --baud 460800 write_flash -z 0x0 firmware.bin
   ```
   Replace `<YOUR_PORT>` with your serial port (e.g. `COM5` on Windows, `/dev/cu.usbserial-XXXX` on macOS, `/dev/ttyUSB0` on Linux).

**Prefer a GUI? Any tool that flashes a single merged `.bin` at offset `0x0` works the same way — e.g. [ESP Web Tools](https://esphome.github.io/esp-web-tools/) (flash straight from Chrome, no install) or ESPHome-Flasher.**

> A pre-built binary only works correctly if your wiring matches the [pin map](#hardware--wiring) above exactly — it has no way to know your GPIOs differ.

> ⚠️ **NOTE** **SD card:** FAT32 formatted 8GB -16GB preferred — the firmware creates `/REMOTE/` and its contents automatically, nothing to pre-load.

**First boot:** shows "Loading... Preparing remotes...", runs touch calibration if none is stored yet (tap the 4 crosshairs), then opens the Devices screen.Tap **+ ADD** to create your first device and start learning keys. 
Recalibrate anytime via **Devices → DISPLAY → CALIBRATE TOUCH**.


---

## Using the Universal Remote

The firmware ships blank — no devices, no pre-loaded remote profile. Everything below starts from **+ ADD** on a fresh install.

**Devices list** — up to 5/page. **+ ADD** creates a device; **EDIT** reorders/renames/deletes; **DISPLAY** and **ANALYZ** open Settings and the Analyzer.

**Key grid** — up to 10 keys/page (5×2). Tap to transmit once, hold to repeat (~220ms delay, ~110ms interval). **+ KEY** learns a new button; **EDIT** enters key management (**ARRANGE** reorder, **LEARN** relearn, **RENAME**, **DELETE**).

**Learning a key:** name it on the on-screen keyboard (max 12 chars) → **Listening...** → point the original remote at the TSOP receiver and press once → **TEST** / **RETRY** / **SAVE** / **CANCEL**. A clean idle period is required before the receiver arms for a new capture.

**Replay** works by storing each key's raw mark/space timings and gating the IR LED's 36kHz carrier to match — any protocol, decoded or not.

---

## IR Analyzer

Standalone tool, separate from the device database — **Devices → ANALYZ**. **CAPTURE** listens for a frame; **PREV/NEXT** browse the last 5 in-RAM captures; **TEST** replays the shown capture; **SAVE** exports it to `/REMOTE/CAPTURES/` as text; **CLEAR** wipes history (history doesn't persist across reboot — only SAVE does).

### Protocol Support

| Protocol | Identify | Decode |
| ------------------ | -------- | ------ |
| NEC                | ✅        | ✅      |
| Samsung            | ✅        | ✅      |
| JVC                | ✅        | ✅      |
| Sony SIRC          | ✅        | ✅      |
| RC5                | ✅        | ✅      |
| RC6                | ✅        | ✅      |
| Panasonic/Kaseikyo | ✅        | ✅      |
| LG                 | ✅        | ✅      |
| Denon              | ✅        | ✅      |
| Sharp              | ✅        | ✅      |

- checksums — including Manchester-coded RC5/RC6
- A signal that doesn't match any named protocol gets a raw Addr/Cmd readout via a generic fallback.

---

## Display Settings & Touch Calibration

**Devices → DISPLAY:** backlight slider (PWM 5–255, live), screen timeout (10/20/30/60s or NEVER — IR capture counts as activity so it won't sleep mid-listen), rotation (Normal/180°, live preview), and **CALIBRATE TOUCH**. **SAVE** commits to NVS; **BACK** discards.

**Calibration** is a 4-point corner-crosshair routine, extrapolated to the true screen edges, stored primarily in ESP32 NVS (SD-independent) with `/touchcal.cfg` kept as a compatibility backup. It's always captured in Normal orientation — 180° mode mirrors both axes in software from the same data, so no separate recalibration is needed per rotation.

---

## SD Card Layout

```text
/REMOTE/
├── DEVICE_ORDER.TXT
├── D000/
│   ├── NAME.TXT
│   ├── KEY_ORDER.TXT
│   └── KEYS/
│       ├── K000.BIN     Raw pulse timings (uint16 count + pulses, µs)
│       └── K000.NAME    Key label
└── CAPTURES/            IR Analyzer exports
    ├── COUNTER.TXT
    └── NEC_0001.TXT
```

A missing/corrupt order file only affects display order — the firmware falls back to physical folder scanning and rebuilds it, so a real device or key can never be hidden by it. Limits: `MAX_DEVICES = 20`, `MAX_KEYS = 40` per device.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Touch feels offset | **Devices → DISPLAY → CALIBRATE TOUCH** |
| Devices list is empty right after flashing | Expected — the firmware ships blank. Tap **+ ADD** to create your first device |
| A device you created earlier is missing | Check SD is FAT32 Format and Wired Correctly, Check Serial Monitor for `SD begin FAILED` |
| "No valid IR" learning a key | Point remote directly at the TSOP (GPIO 22), wait for a clean idle period, check original remote's batteries |
| Key transmits but target ignores it | Inspect with IR Analyzer → CAPTURE; check IR LED polarity/resistor; test at close range |
| Screen goes black unexpectedly | That's Screen Timeout — set to NEVER if unwanted |
| Held button seems to "run away" | Shouldn't happen (release requires several consecutive confirmed readings) — suspect a flaky touch connection if it does |

---

## Project Phase & Credits

**V1.1.3 :** SD-backed device/key database, IR learning with test-before-save, hold/repeat replay, IR Analyzer with protocol ID + decode, touch calibration, display settings. Later Versions are planned for AC remote support.

Thanks to **mAi Digitech** for support building the hardware.

This is a personal DIY project, not a commercial product.

<div align="center">
<div align="center">

### 📺 Learn IR. Analyze IR. Control Everything.

</div>

**[⬆ Back to top](#remodaz)**

</div>
