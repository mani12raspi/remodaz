
## 🌡️❄️🌬️ Remodaz AC

**Built with ESP32-WROOM-32U · Firmware build `V1.6.4`**

> 🛠️ **DIY hardware project** — a companion firmware to [Remodaz](https://github.com/mani12raspi/remodaz), built for the same ESP32-WROOM-32U + 2.4" ILI9341 + XPT2046 hardware, but dedicated entirely to **air conditioner** remotes instead of generic IR devices. It replaces raw-timing IR learning with [IRremoteESP8266](https://github.com/crankyoldgit/IRremoteESP8266)'s `IRac` protocol engine, so instead of "learning" a button it *speaks* the AC's native protocol directly — full state control (power, temp, mode, fan, swing, and more) from a library of 77 built-in AC protocol entries, with no original remote required.

<img width="300" height="400" alt="pg1" src="https://github.com/user-attachments/assets/ce5dfa84-82fc-4458-9172-3a719f01c190" />
<img width="300" height="400" alt="pg2" src="https://github.com/user-attachments/assets/df071c40-2bfc-4c3d-8f86-942accbc2526" />
<img width="300" height="400" alt="pg3" src="https://github.com/user-attachments/assets/be39fc62-5d0d-4b10-be45-2305277bb105" />

## At a Glance

| Capability | What it does |
|---|---|
| 🔎 **Auto Detect** | Point the original AC remote at the receiver and press a button — the protocol and current AC state are decoded automatically |
| 📚 **AC Library** | 77 built-in protocol entries (Daikin, Gree, Mitsubishi, Samsung, LG, Panasonic, Voltas, and many more) — browse and select manually if you don't have the original remote |
| 🎛️ **Full State Control** | Power, Temp ±, Mode, Fan speed, Swing V/H, Turbo, Quiet, Eco, Light, Filter, Clean, Beep — each button only appears if the selected protocol actually supports it |
| 🔁 **Stateful Sends** | Every button press re-sends the AC's complete state (not just one keycode), matching how real AC remotes work |
| ↕️ **Arrange Keys** | Reorder which capability buttons appear and in what order, persisted to flash |
| 👆 **Touch UI** | XPT2046 resistive touchscreen with 4-point calibration, auto-run on first boot |
| ⚙️ **Display Settings** | Brightness, screen timeout, 0°/180° rotation |

---

## Table of Contents

- [Hardware & Wiring](#hardware--wiring)
- [Menu Tree](#menu-tree)
- [Using the AC Remote](#using-the-ac-remote)
- [Supported AC Protocols](#supported-ac-protocols)
- [Display Settings & Touch Calibration](#display-settings--touch-calibration)
- [Troubleshooting](#troubleshooting)
- [Version](#version)

---

## Hardware & Wiring

Same physical build as the base Remodaz hardware built with ESP32-WROOM-32U t to a 2.4" ILI9341 display, an XPT2046 touch panel, a microSD module and an IR receiver/transmitter pair.

| Component | Part | Notes |
|---|---|---|
| MCU | ESP32-WROOM-32U | Bare module |
| Display | ILI9341, 2.4", 240×320 | Driven by TFT_eSPI — pins |
| Touch | XPT2046, resistive | Shares the SD card's SPI (VSPI) peripheral |
| Storage | MicroSD | Used for touch calibration |
| IR Receiver | TSOP 1838 | `GPIO 22` |
| IR Transmitter | IR LED (36kHz carrier) | `GPIO 27` |
| Feedback LED | Status LED | `GPIO 26` |


## Menu Tree

```
REMODAZ AC
│
├── AUTO DETECT                Listen for the original AC remote, decode protocol + live state
│
├── ALL AC REMOTES             Browse the 77-entry protocol library, select manually
│
├── CURRENT REMOTE             Jump back into the active protocol's control screen
│   ├── Status block           Setpoint, ON/OFF, mode, fan, swing — live
│   ├── Capability buttons     Only buttons the protocol actually supports are shown
│   └── PREV / NEXT / BACK
│
├── ARRANGE KEYS                Reorder which capability buttons appear, and in what order
│
└── DISPLAY (Settings)
    ├── Brightness / Timeout / Rotation
    └── Calibrate Touch         4-point calibration
```

---

## Using the AC Remote

**Auto Detect** — point the original AC remote at the IR receiver (`GPIO 22`) and press a button (the on-screen prompt suggests TEMP). The firmware decodes both the protocol *and* the AC's current state (power, mode, temp, fan, etc.) from that single frame, then opens the matching control screen already synced to that state.

**All AC Remotes** — browse the full 77-entry protocol library if you don't have the original remote. Protocols not compiled into the current `IRac` build are shown greyed out and marked `*`. Selecting one initializes a fresh default state (Cool, 24°C, fan Auto, display light on).

**Current Remote** — the control screen for whichever protocol is active. Every button press re-sends the AC's *entire* state via `irac.sendAc()` (matching how real AC remotes work, rather than sending isolated keycodes). Non-POWER buttons are dimmed and disabled while the AC is off.

**Capabilities per protocol** — each AC protocol only exposes the buttons it actually supports (e.g. some protocols have no Swing H, Quiet, or Beep). This is looked up from a built-in capability table per protocol rather than assumed.

**Arrange Keys** — reorder the capability buttons (Power, Temp −/+, Mode, Fan, Swing V/H, Quiet, Turbo, Eco, Light, Filter, Clean, Beep); the order is shared across all protocols and persists across reboots.

---

## Supported AC Protocols

77 protocol entries are built in, spanning brands including Airton, Airwell, Amcor, Argo, Bosch, Carrier, Coolix, Daikin (10 protocol variants), Delonghi, Electra, Fujitsu, Gree, Haier, Hitachi, Kelon, Kelvinator, LG, Midea, Mirage, Mitsubishi / Mitsubishi Heavy, Neoclima, Panasonic, Samsung, Sanyo, Sharp, TCL, Teco, Toshiba, Transcold, Trotec, Truma, Vestel, Voltas, Whirlpool, and York.

Which buttons appear for each protocol (temp control is universal; swing, turbo, quiet, eco, light, filter, clean, and beep vary by protocol) is determined by a built-in per-protocol capability table, viewable in-app via **ALL AC REMOTES**.

> The exact set of protocols available at runtime depends on which decoders are compiled into the `IRremoteESP8266` build; unsupported entries are marked `*` in the library list rather than hidden.

---

## Display Settings & Touch Calibration

**DISPLAY** page: Backlight slider, screen timeout (10s / 20s / 30s / 60s / NEVER — default NEVER), rotation (Normal / 180°, both keeping the same 240×320 portrait layout), and **CALIBRATE TOUCH**. **SAVE** commits to SD; **BACK** discards.

**Calibration** is a 4-point corner-crosshair routine, extrapolated to the true screen edges, `/touchcal.cfg` on SD. It's captured once at the baseline rotation — the 180° mode mirrors both axes in software from the same calibration data, so no separate recalibration is needed per rotation. It also runs automatically on the very first boot if no calibration is stored yet.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Touch feels offset | **DISPLAY → CALIBRATE TOUCH** |
| "Received `<PROTOCOL>`" but no state change on Auto Detect | The frame decoded but not into a usable AC state — try again, or select the protocol manually via **ALL AC REMOTES** |
| A protocol shows `*` and won't select | Not compiled into the current `IRremoteESP8266`/`IRac` build — see [Supported AC Protocols](#supported-ac-protocols) |
| Button press does nothing | Check the AC is powered ON in the app (non-POWER buttons are disabled while off) |
| "`<ACTION>` UNSUPPORTED" status | The selected protocol's `IRac` sender rejected that state change — inspect wiring/LED polarity and retry |
| Screen goes black unexpectedly | Screen Timeout is active — set to **NEVER** on the DISPLAY page if unwanted |
| Devices/library missing after flashing | Expected — no SD device database is used in this build; the AC library is compiled in, not loaded from SD |

---

## Version 

**Build `V1.6.4`** — Auto Detect (protocol + state decode), 77-entry AC protocol library, full stateful control (power/temp/mode/fan/swing/turbo/quiet/eco/light/filter/clean/beep) gated per-protocol capability, reorderable key layout, touch calibration, display settings.

Companion firmware to [Remodaz](https://github.com/mani12raspi/remodaz) (universal IR remote + protocol analyzer) — same hardware platform, dedicated to AC control.

This is a personal DIY project, not a commercial product.
