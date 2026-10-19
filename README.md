# Pico Chiptune Player

**Version:** 0.6 (Dual-PSG Stereo Variant — 2× YM2149F, 6 voices)  
**Author:** Terence Ang <terenceang@mac.com>  
**Company:** MotionFxDesign  
**License:** [CC BY-NC-SA 4.0](LICENSE) (Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International) — **NOT FOR COMMERCIAL USE**  

---

## Overview

The **Pico Chiptune Player** is an open-hardware music player powered by the **Raspberry Pi Pico 2 (RP2350)** microcontroller driving two authentic **Yamaha YM2149F** Programmable Sound Generators (PSGs), register-compatible with the General Instrument AY-3-8910. Each PSG provides 3 voices, giving **6 independent voices** total, split into **stereo** (one PSG per channel).

It enables standalone playback of classic AY-3-8910 / YM2149 chiptune music formats (e.g., `.ym`, `.vgm`, `.psg`, `.pt3`) loaded directly from an onboard microSD card, with song and playback metadata displayed on an I2C OLED display.

---

## Hardware Highlights

- **Microcontroller:** Raspberry Pi Pico 2 (RP2350)
- **Sound Generators:** 2× Yamaha YM2149F (40-pin DIP, socketed), 6 voices total; a matched pair of AY-3-8910s also fits (see the PSG fit note below)
- **Storage:** 4-bit SDIO microSD card socket (`J2`), driven by the Pico SDK / pico-extras PIO-based `pico_sd_card` driver; card presence detected in software (`R1` 10kΩ pull-up on `DAT3`)
- **Audio Output (stereo):**
  - Two independent 3-channel passive summing mixers (`R4`,`R5`,`R7`,`R8` for the left/U1 PSG; `R9`,`R10`,`R11`,`R12` for the right/U4 PSG), each with a low-pass filter cap (`C4`, `C9` = 100nF) and AC coupling cap (`C3`, `C10` = 1µF X7R MLCC)
  - One on-board PAM8403 3W+3W filterless Class-D stereo amplifier (`U3`, SOIC-16) with hardware mute/shutdown on `SD_Amp` (GPIO9). A 10kΩ `R13` pull-down keeps it muted at boot; drive GPIO9 HIGH to play
  - JST-PH 2-pin speaker connectors `J3` (left) / `J4` (right) with bridge-tied outputs for 4–8Ω speakers
- **Display:** 0.96" SSD1306 128x64 I2C OLED Display (`U2`)
- **Storage:** MicroSD Card Slot via SPI mode with Card Detect (`J2`)
- **Controls:** 3 tactile pushbuttons (`SW1` Play/Pause, `SW2` Next Track, `SW3` Previous Track)
- **Clock Divider Select:** Solder jumper `JP1` for the YM2149F ÷2 internal clock divider (`/SEL`, pin 26), shared by both PSGs

---

## Pinout Summary (Raspberry Pi Pico 2)

| GPIO | Pico Pin | Net / Signal | Connected Subsystem |
| :--- | :--- | :--- | :--- |
| **`GPIO0..GPIO7`** | Pins 1, 2, 4..7, 9, 10 | `DA0..DA7` | YM2149F 8-Bit Data Bus, shared by U1/U4 (Pins 37..30) |
| **`GPIO8`** | Pin 11 | `BC1` | PSG1 (U1) Bus Control 1 (Pin 29) |
| **`GPIO9`** | Pin 12 | `SD_Amp` | PAM8403 Audio Amp Enable / Mute (`U3` Pins 5 `~MUTE` & 12 `~SHDN`, `R13` pull-down) |
| **`GPIO10`** | Pin 14 | `BDIR` | PSG1 (U1) Bus Direction Control (Pin 27) |
| **`GPIO11`** | Pin 15 | `~RESET` | YM2149F Active-Low Reset, shared by U1/U4 (Pin 23) + 10kΩ R3 Pull-down |
| **`GPIO12`** | Pin 16 | `CLOCK` | YM2149F Master Clock Input, shared by U1/U4 (Pin 22 via 33Ω series R2) |
| **`GPIO13`** | Pin 17 | `SW_NEXT` | Next Track Button (`SW2`) |
| **`GPIO14`** | Pin 19 | `SW_PLAY` | Play / Pause Button (`SW1`) |
| **`GPIO15`** | Pin 20 | `SW_PREV` | Previous Track Button (`SW3`) |
| **`GPIO16`** | Pin 21 | `OLED_SDA` | 0.96" OLED I2C SDA (`U2` Pin 4) |
| **`GPIO17`** | Pin 22 | `OLED_SCL` | 0.96" OLED I2C SCL (`U2` Pin 3) |
| **`GPIO18`** | Pin 24 | `SD_DAT0` | MicroSD SDIO Data 0 (`J2` Pin 7) |
| **`GPIO19`** | Pin 25 | `SD_DAT1` | MicroSD SDIO Data 1 (`J2` Pin 8) |
| **`GPIO20`** | Pin 26 | `SD_DAT2` | MicroSD SDIO Data 2 (`J2` Pin 1) |
| **`GPIO21`** | Pin 27 | `SD_DAT3` | MicroSD SDIO Data 3 (`J2` Pin 2 + 10kΩ R1 Pull-up) |
| **`GPIO22`** | Pin 29 | `SD_CLK` | MicroSD SDIO Clock (`J2` Pin 5) |
| **`GPIO26`** | Pin 31 | `BDIR2` | PSG2 (U4) Bus Direction Control (Pin 27) |
| **`GPIO27`** | Pin 32 | `BC1_2` | PSG2 (U4) Bus Control 1 (Pin 29) |
| **`GPIO28`** | Pin 34 | `SD_CMD` | MicroSD SDIO Command (`J2` Pin 3) |

> **Dual-PSG addressing (Rev 0.6):** The YM2149F's `A8`/`A9` lines are fixed the same on both chips, so they cannot be told apart by address; the 8-bit data bus (`DA0..DA7`), `CLOCK`, `~RESET` and `BC2` are **shared** between `U1` and `U4`, while `BDIR`/`BC1` are driven **separately per PSG** (`GPIO10`/`GPIO8` for U1, `GPIO26`/`GPIO27` for U4). Driving one PSG's `BDIR`/`BC1` to the inactive state (both low) puts its bus in High-Z, so the Pico can address either PSG by selecting the appropriate control pair.

> **PSG fit (Rev 0.6):** Both sockets (`U1`, `U4`) are specified as **Yamaha YM2149F**. `JP1` drives the `/SEL` pin of *both* chips, so with two YM2149F fitted it halves (bridged) or keeps (open) the clock on both channels together. A pair of AY-3-8910s is a drop-in alternative **only with `JP1` left open**; on an AY-3-8910, pin 26 is a test pin and the clock is never divided. Do not mix AY-3-8910 and YM2149F with `JP1` bridged: the two channels would play an octave apart.

> **Rev 0.5 native-function correction:** Rev 0.5 re-assigns the OLED I²C and microSD SPI to the RP2350's native I2C0/SPI0 pins (`OLED_SDA`→`GPIO16`, `OLED_SCL`→`GPIO17`; `SD_SCK`→`GPIO18`, `SD_MISO`→`GPIO20`, `SD_CS`→`GPIO21`; `SD_MOSI` stays `GPIO19`) so the on-chip peripherals drive them directly. No additional pins are used. The manufactured **Rev 0.4** board swapped/mis-mapped these signals and requires PIO or bit-banged I²C/SPI in firmware. Details in [HARDWARE.md §3.6](HARDWARE.md).

> **Rev 0.6 microSD SDIO:** The microSD interface is **4-bit SDIO** (not SPI), driven by RP2350 **PIO** via the Pico SDK / pico-extras `pico_sd_card` driver (`SD_DAT0..DAT3` = `GPIO18..GPIO21`, `SD_CLK` = `GPIO22`, `SD_CMD` = `GPIO28`). The dedicated card-detect switch is unused; card presence is detected in software. Because RP2350 has no hardware SDIO controller, firmware must use the PIO `pico_sd_card` driver and enable the internal pull-ups on `CMD`/`DAT0`–`DAT3` (only `DAT3` carries an external 10kΩ pull-up).

For the complete hardware design details, circuit topology, and full Bill of Materials with LCSC part numbers, refer to [HARDWARE.md](HARDWARE.md).

---

## License & Commercial Use Notice

Copyright (c) 2026 Terence Ang <terenceang@mac.com> / MotionFxDesign.

This project is licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.

> **NOT FOR COMMERCIAL USE**  
> This design, including schematics, PCB layouts, Gerber files, firmware, and documentation, is free for personal, hobby, educational, and non-commercial use.  
> **Commercial manufacture, distribution, sale, or bundling for profit is strictly prohibited** without prior written authorization from Terence Ang (<terenceang@mac.com>).

See the [LICENSE](LICENSE) file for the complete license terms.
