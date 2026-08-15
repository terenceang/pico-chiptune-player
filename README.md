# Pico Chiptune Player

**Version:** 0.4 (Manufactured Production Release)  
**Author:** Terence Ang <terenceang@mac.com>  
**Company:** MotionFxDesign  
**License:** [CC BY-NC-SA 4.0](LICENSE) (Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International) — **NOT FOR COMMERCIAL USE**  

---

## Overview

The **Pico Chiptune Player** is an open-hardware music player powered by the **Raspberry Pi Pico 2 (RP2350)** microcontroller driving an authentic **General Instrument AY-3-8910** (or **Yamaha YM2149F**) 3-voice Programmable Sound Generator (PSG).

It enables standalone playback of classic AY-3-8910 / YM2149 chiptune music formats (e.g., `.ym`, `.vgm`, `.psg`, `.pt3`) loaded directly from an onboard microSD card, with song and playback metadata displayed on an I2C OLED display.

---

## Hardware Highlights

- **Microcontroller:** Raspberry Pi Pico 2 (RP2350)
- **Sound Generator:** General Instrument AY-3-8910 / Yamaha YM2149F (40-pin DIP)
- **Audio Output:**
  - 3-channel passive summing mixer (`R4`, `R5`, `R7`, `R8` = 1kΩ)
  - Anti-aliasing / low-pass filter capacitor (`C4` = 100nF)
  - AC coupling capacitor (`C3` = 1µF)
  - Onboard Adafruit PAM8302A 2.5W Mono Class-D Audio Amplifier Module (`U3`) with hardware GPIO mute/shutdown control (`SD_Amp` on GPIO9)
  - Direct differential speaker terminals (`OUT+`, `OUT-`)
- **Display:** 0.96" SSD1306 128x64 I2C OLED Display (`U2`)
- **Storage:** MicroSD Card Slot via SPI mode with Card Detect (`J2`)
- **Controls:** 3 tactile pushbuttons (`SW1` Play/Pause, `SW2` Next Track, `SW3` Previous Track)
- **Clock Divider Select:** Solder jumper `JP1` for YM2149 ÷2 internal clock divider selection

---

## Pinout Summary (Raspberry Pi Pico 2)

| GPIO | Pico Pin | Net / Signal | Connected Subsystem |
| :--- | :--- | :--- | :--- |
| **`GPIO0..GPIO7`** | Pins 1, 2, 4..7, 9, 10 | `DA0..DA7` | AY-3-8910 8-Bit Data Bus (Pins 37..30) |
| **`GPIO8`** | Pin 11 | `BC1` | AY-3-8910 Bus Control 1 (Pin 29) |
| **`GPIO9`** | Pin 12 | `SD_Amp` | PAM8302 Audio Amp Shutdown / Mute Control (`U3` Pin 3) |
| **`GPIO10`** | Pin 14 | `BDIR` | AY-3-8910 Bus Direction Control (Pin 27) |
| **`GPIO11`** | Pin 15 | `~RESET` | AY-3-8910 Active-Low Reset (Pin 23) + 10kΩ R3 Pull-down |
| **`GPIO12`** | Pin 16 | `CLOCK` | AY-3-8910 Master Clock Input (Pin 22 via 33Ω series R2) |
| **`GPIO13`** | Pin 17 | `SW_NEXT` | Next Track Button (`SW2`) |
| **`GPIO14`** | Pin 19 | `SW_PLAY` | Play / Pause Button (`SW1`) |
| **`GPIO15`** | Pin 20 | `SW_PREV` | Previous Track Button (`SW3`) |
| **`GPIO16`** | Pin 21 | `OLED_SCL` | 0.96" OLED I2C SCL (`U2` Pin 3) |
| **`GPIO17`** | Pin 22 | `OLED_SDA` | 0.96" OLED I2C SDA (`U2` Pin 4) |
| **`GPIO18`** | Pin 24 | `SD_CS` | MicroSD SPI Chip Select (`J2` Pin 2 + 10kΩ R1 Pull-up) |
| **`GPIO19`** | Pin 25 | `SD_MOSI` | MicroSD SPI Data Input (`J2` Pin 3) |
| **`GPIO20`** | Pin 26 | `SD_SCK` | MicroSD SPI Clock (`J2` Pin 5) |
| **`GPIO21`** | Pin 27 | `SD_MISO` | MicroSD SPI Data Output (`J2` Pin 7) |
| **`GPIO22`** | Pin 29 | `SD_Card_Detect` | MicroSD Card Detect Switch (`J2` Pin 9) |

For the complete hardware design details, circuit topology, and full Bill of Materials with LCSC part numbers, refer to [HARDWARE.md](HARDWARE.md).

---

## License & Commercial Use Notice

Copyright (c) 2026 Terence Ang <terenceang@mac.com> / MotionFxDesign.

This project is licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.

> **NOT FOR COMMERCIAL USE**  
> This design, including schematics, PCB layouts, Gerber files, firmware, and documentation, is free for personal, hobby, educational, and non-commercial use.  
> **Commercial manufacture, distribution, sale, or bundling for profit is strictly prohibited** without prior written authorization from Terence Ang (<terenceang@mac.com>).

See the [LICENSE](LICENSE) file for the complete license terms.
