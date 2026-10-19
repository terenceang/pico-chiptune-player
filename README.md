# Pico Chiptune Player

**Version:** 0.8 (JLC Assembly Prep — corrected GPIO map, 0402/0805 passives, LCSC part numbers)  
**Author:** Terence Ang <terenceang@mac.com>  
**Company:** MotionFxDesign  
**License:** [CC BY-NC-SA 4.0](LICENSE) (Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International) — **NOT FOR COMMERCIAL USE**  

---

## Overview

The **Pico Chiptune Player** is an open-hardware music player powered by an on-board **Raspberry Pi RP2354A** microcontroller (QFN-60 with 2 MB internal stacked QSPI NOR flash) driving two authentic **Yamaha YM2149F** Programmable Sound Generators (PSGs), register-compatible with the General Instrument AY-3-8910. Each PSG provides 3 voices, giving **6 independent voices** total, routed through a **software-controlled 4-mode analog stereo/mono mixing matrix** (`2× 74HCT4053D`).

It enables standalone playback of classic AY-3-8910 / YM2149 chiptune music formats (e.g., `.ym`, `.vgm`, `.psg`, `.pt3`, `.sndh`) loaded directly from an onboard microSD card, with song and playback metadata displayed on an I2C OLED display.

---

## Hardware Highlights (Rev 0.8)

- **Microcontroller (`U5`):** Bare **Raspberry Pi RP2354A** (QFN-60, 7×7 mm, 0.4 mm pitch) with **2 MB internal stacked QSPI NOR flash** and all **30 Bank-0 GPIOs (`GPIO0..GPIO29`)** exposed, clocked by a 12 MHz SMD-3225 crystal (`Y1`, `C20`/`C21` = 15 pF, `R16` = 1 kΩ) with internal 1.1 V core buck regulator (`L1` = 3.3 µH) and onboard `BOOTSEL` (`SW4`) and `RESET` (`SW5`) SMD tactile buttons (`TS-1187A`, LCSC `C318884`).
- **USB-C 5V/3A Power & 3-Stage Sequencing (`J1`, `D1`, `U6`, `Q1`, `Q2`):**
  - 16-pin USB 2.0 Type-C receptacle (`J1`) with independent 5.1 kΩ 1% pull-downs (`R21`, `R22`) on `CC1`/`CC2` advertising a fixed 5V/3A sink, plus `USBLC6-2SC6` (`D1`) ESD protection and 27 Ω (`R17`, `R18`) USB series termination.
  - **Stage 1 (`VBUS` → `+3.3V` & `+1V1`):** `AMS1117-3.3` (`U6`) powers `+3.3V` (`RP2354A` `IOVDD`, microSD `J2`, OLED `U2`) and the `RP2354A` internal `+1V1` core regulator immediately on USB plug-in so `RP2354A` I/O pins initialize cleanly with `~RESET` held LOW (`R3` = 10 kΩ).
  - **Stage 2 (`+3.3V` RC-delayed → `+5V`):** RC delay (`R23` = 47 kΩ, `C31` = 100 nF) turns on N-MOSFET `Q2` (`2N7002`), pulling down the gate of high-side P-MOSFET `Q1` (`AO3401A`, with `R28` = 100 kΩ pull-up and `C32` = 10 nF soft-start cap) to switch `VBUS` onto the `+5V` rail **only after `+3.3V` is stable**, preventing unpowered latch-up or back-bias on the 5 V `YM2149F` PSGs (`U1`, `U4`), `74HCT4053D` analog switches (`U8`, `U9`), and `PAM8403` amplifier (`U3`).
  - **Stage 3 (`SD_Amp` on `GPIO21` → `U3` `PAM8403`):** `R13` (10 kΩ pull-down) keeps `PAM8403` `~MUTE` and `~SHDN` LOW during Stage 1 & Stage 2 power-up until firmware drives `GPIO21` HIGH after AC coupling capacitors `C3`/`C10` settle, eliminating power-on pop.
- **Sound Generators (`U1`, `U4`):** 2× Yamaha YM2149F (40-pin DIP, socketed), 6 voices total; a matched pair of AY-3-8910s also fits with `JP1` open.
- **4-Mode Hardware Stereo/Mono Analog Mixing Matrix (`U8`, `U9`):**
  - Dual `74HCT4053D` triple 2:1 analog multiplexers powered from `+5V` (`VCC = +5V`, `VEE = VSS = INH = GND`, 3.3 V TTL-compatible control inputs) controlled by `GPIO14` (`MUX_PAN`), `GPIO15` (`MUX_B_CEN`), `GPIO16` (`MUX_C_CEN`), and `GPIO17` (`MUX_MONO`) with 470 Ω (`R14`, `R15`) `-3 dB` center-channel compensation resistors.
  - Supports **Dual-PSG Split Stereo** (`U1`=Left, `U4`=Right), **ABC Stereo** (`A`=Left, `B`=Center, `C`=Right across both PSGs), **ACB Stereo** (`A`=Left, `C`=Center, `B`=Right across both PSGs), and **True Mono** (`A+B+C` of both PSGs summed equally to Left & Right).
- **Audio Amplifier & Output (`U3`, `J3`, `J4`):**
  - Two passive summing/filtering networks (`R4`,`R5`,`R7`,`R8`,`C4`,`C3` for Left; `R9`,`R10`,`R11`,`R12`,`C9`,`C10` for Right) feeding an on-board **PAM8403 3W+3W filterless Class-D stereo amplifier** (`U3`, SOIC-16).
  - JST-PH 2-pin speaker connectors `J3` (left) / `J4` (right) with bridge-tied outputs for 4–8 Ω speakers.
- **Storage (`J2`):** 4-bit SDIO microSD card socket (Hirose `DM3AT-SF-PEJM5`, LCSC `C114218`) driven by the RP2350 PIO-based `pico_sd_card` driver; card presence detected in software (`R1` 10 kΩ pull-up on `DAT3`).
- **Display (`U2`) & Controls (`SW1..SW5`):** 0.96" SSD1306 128×64 I2C OLED display (`U2`), 3 navigation pushbuttons (`SW1` Play/Pause, `SW2` Next, `SW3` Prev), plus `SW4` (`BOOTSEL`) and `SW5` (`RESET`) — all `TS-1187A` SMD tactiles (LCSC `C318884`).
- **Clock Divider Select (`JP1`):** Solder jumper for the YM2149F ÷2 internal clock divider (`/SEL`, pin 26), shared by both PSGs.

---

## Pinout Summary (`U5` — RP2354A QFN-60)

| GPIO | QFN-60 Pin | Net / Signal | Connected Subsystem |
| :--- | :---: | :--- | :--- |
| **`GPIO0..GPIO3`** | Pins 2..5 | `DA0..DA3` | YM2149F 8-Bit Data Bus Bits 0..3, shared by `U1`/`U4` (Pins 37..34) |
| **`GPIO4..GPIO7`** | Pins 7..10 | `DA4..DA7` | YM2149F 8-Bit Data Bus Bits 4..7, shared by `U1`/`U4` (Pins 33..30) |
| **`GPIO8`** | Pin 12 | `BC1` | PSG1 (`U1`) Bus Control 1 (Pin 29) |
| **`GPIO9`** | Pin 13 | `MCU_CLK` | YM2149F Master Clock Input, shared by `U1`/`U4` (Pin 22 via 33 Ω series `R2`) |
| **`GPIO10`** | Pin 14 | `BDIR` | PSG1 (`U1`) Bus Direction Control (Pin 27) |
| **`GPIO11`** | Pin 15 | `~RESET` | YM2149F Active-Low Reset, shared by `U1`/`U4` (Pin 23) + 10 kΩ `R3` pull-down |
| **`GPIO12`** | Pin 16 | `BC1_2` | PSG2 (`U4`) Bus Control 1 (Pin 29) |
| **`GPIO13`** | Pin 17 | `BDIR2` | PSG2 (`U4`) Bus Direction Control (Pin 27) |
| **`GPIO14`** | Pin 18 | `MUX_PAN` | Stereo Matrix Right/Left Voice Pan (`U9` Pins 9, 10, 11 + 10 kΩ `R24` pull-down) |
| **`GPIO15`** | Pin 19 | `MUX_B_CEN` | Stereo Matrix Voice B Center Enable (`U8` Pin 11 + 10 kΩ `R25` pull-down) |
| **`GPIO16`** | Pin 27 | `MUX_C_CEN` | Stereo Matrix Voice C Center Enable (`U8` Pin 10 + 10 kΩ `R26` pull-down) |
| **`GPIO17`** | Pin 28 | `MUX_MONO` | Stereo Matrix True Mono Bridge Enable (`U8` Pin 9 + 10 kΩ `R27` pull-down) |
| **`GPIO18`** | Pin 29 | `SW_NEXT` | Next Track Button (`SW2`) |
| **`GPIO19`** | Pin 31 | `SW_PLAY` | Play / Pause Button (`SW1`) |
| **`GPIO20`** | Pin 32 | `SW_PREV` | Previous Track Button (`SW3`) |
| **`GPIO21`** | Pin 33 | `SD_Amp` | PAM8403 Audio Amp Enable / Mute (`U3` Pins 5 `~MUTE` & 12 `~SHDN`, `R13` pull-down) |
| **`GPIO22`** | Pin 34 | `SD_DAT0` | MicroSD SDIO Data 0 (`J2` Pin 7) |
| **`GPIO23`** | Pin 35 | `SD_DAT1` | MicroSD SDIO Data 1 (`J2` Pin 8) |
| **`GPIO24`** | Pin 36 | `SD_DAT2` | MicroSD SDIO Data 2 (`J2` Pin 1) |
| **`GPIO25`** | Pin 37 | `SD_DAT3` | MicroSD SDIO Data 3 (`J2` Pin 2 + 10 kΩ `R1` pull-up) |
| **`GPIO26`** | Pin 40 | `SD_CLK` | MicroSD SDIO Clock (`J2` Pin 5) |
| **`GPIO27`** | Pin 41 | `SD_CMD` | MicroSD SDIO Command (`J2` Pin 3) |
| **`GPIO28`** | Pin 42 | `OLED_SDA` | 0.96" OLED I2C0 SDA (`U2` Pin 4) |
| **`GPIO29`** | Pin 43 | `OLED_SCL` | 0.96" OLED I2C0 SCL (`U2` Pin 3) |

---

## 4-Mode Stereo / Mono Mixing Matrix Truth Table

| Mode | `MUX_MONO` (`GPIO17`) | `MUX_B_CEN` (`GPIO15`) | `MUX_C_CEN` (`GPIO16`) | `MUX_PAN` (`GPIO14`) | Left Channel (`AMP_INL`) | Right Channel (`AMP_INR`) | Use Case |
| :--- | :---: | :---: | :---: | :---: | :--- | :--- | :--- |
| **0: Dual-PSG Split (Default)** | `0` | `0` | `0` | `0` | `U1` (`A1 + B1 + C1`) | `U4` (`A2 + B2 + C2`) | 6-voice TurboSound / Dual-PSG tracks (chip 1 Left, chip 2 Right); default at boot via `R24..R27` pull-downs |
| **1: ABC Stereo** | `0` | `1` | `0` | `1` | `A1 + A2` + `0.71×(B1 + B2)` | `C1 + C2` + `0.71×(B1 + B2)` | Standard Western / Atari ST / ZX Spectrum ABC stereo (`A`=Left, `B`=Center `-3 dB`, `C`=Right) |
| **2: ACB Stereo** | `0` | `0` | `1` | `0` | `A1 + A2` + `0.71×(C1 + C2)` | `B1 + B2` + `0.71×(C1 + C2)` | Eastern European / Pentagon / PT3 ACB stereo (`A`=Left, `C`=Center `-3 dB`, `B`=Right) |
| **3: True Mono** | `1` | `0` | `0` | `0` | `0.5×(U1 + U4)` (`A+B+C`) | `0.5×(U1 + U4)` (`A+B+C`) | Classic single-PSG mono chiptunes played identically through both speakers |

**Rev 0.8** is prepared for JLC PCB assembly: every part carries an `LCSC Part #` in the schematic, passives are JLC basic 0402/0805 parts, and the microSD socket (`C114218`) and buttons (`C318884`) are SMD.

For the complete hardware design details, circuit topology, and full Bill of Materials with LCSC part numbers, refer to [HARDWARE.md](HARDWARE.md).

---

## License & Commercial Use Notice

Copyright (c) 2026 Terence Ang <terenceang@mac.com> / MotionFxDesign.

This project is licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.

> **NOT FOR COMMERCIAL USE**  
> This design, including schematics, PCB layouts, Gerber files, firmware, and documentation, is free for personal, hobby, educational, and non-commercial use.  
> **Commercial manufacture, distribution, sale, or bundling for profit is strictly prohibited** without prior written authorization from Terence Ang (<terenceang@mac.com>).

See the [LICENSE](LICENSE) file for the complete license terms.
