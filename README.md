# Pico Chiptune Player (v0.01)

A standalone hardware chiptune music player powered by the **Raspberry Pi Pico (RP2040)** microcontroller, driving a physical **General Instrument AY-3-8910** or **Yamaha YM2149F** 3-voice Programmable Sound Generator (PSG).

**Revision:** `0.01` | **Author:** Terence Ang / MotionFxDesign

---

## Features

- **Microcontroller:** Raspberry Pi Pico (RP2040) controlling an 8-bit parallel bus.
- **Sound Generator:** Compatible with AY-3-8910 or YM2149F retro 3-voice PSG.
- **Storage:** SPI MicroSD Card interface with dedicated hardware Card Detect (`SD_Card_Detect`).
- **Display:** 4-pin I2C OLED header (0.96" SSD1306/SH1106).
- **Controls:** 3 onboard pushbuttons (Play/Pause, Next Track, Previous Track).
- **Clock Mode Selector:** Hardware jumper (`JP1`) for YM2149F clock division mode selection (1:1 or 1:2 clock divider).
- **Audio Output:** Passive 3-channel (Channels A, B, C) mono summer, low-pass filter, and **PAM8302A 2.5W Mono Class-D Audio Amplifier** driving speaker output header (`J2`).

---

## Repository Contents

| File / Folder | Description |
| :--- | :--- |
| [`HARDWARE.md`](HARDWARE.md) | Complete hardware documentation, pin mapping table, block diagram, and signal specs |
| [`pico chiptune player.kicad_sch`](pico%20chiptune%20player.kicad_sch) | KiCad 8 Schematic |
| [`pico chiptune player.kicad_pcb`](pico%20chiptune%20player.kicad_pcb) | KiCad 8 PCB Layout |
| [`bom/ibom.html`](bom/ibom.html) | Interactive HTML Bill of Materials |
| [`production/`](production/) | Gerber ZIP archives, BOM CSV, and component position files for SMT assembly |
| [`Greber/`](Greber/) | Gerber & Drill manufacturing files |

---

## Hardware Overview & Pinout

For full pinout tables, block diagrams, and hardware specifications, refer to [**`HARDWARE.md`**](HARDWARE.md).

### Quick Summary

- **PSG Bus:** Data bits `DA0..DA7` mapped to Pico `GPIO0..GPIO7`.
- **Bus Control:** `BDIR` on `GPIO8` (reserved; U1 BDIR tied to +5V), `BC1` on `GPIO9`, `~RESET` on `GPIO10`.
- **Clock Output:** Pico `GPIO11` providing master clock (1.789773 MHz / 2.0 MHz).
- **Display:** I2C bus on `GPIO16` (SCL) and `GPIO17` (SDA).
- **User Input:** Navigation buttons on `GPIO12` (Prev), `GPIO13` (Play), and `GPIO15` (Next).
- **SD Interface:** SPI bus on `GPIO18` (CS), `GPIO19` (MOSI), `GPIO20` (SCK), `GPIO21` (MISO), Card Detect on `GPIO22`.

---

## Author

Created by **Terence Ang / MotionFxDesign**

---

## License

This project is licensed under the **Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0)**.

### You are free to:
- **Share** — Copy and redistribute the material in any medium or format.
- **Adapt** — Remix, transform, and build upon the material.

### Under the following terms:
- **Attribution** — You must give appropriate credit to **Terence Ang / MotionFxDesign**, provide a link to the license, and indicate if changes were made.
- **NonCommercial** — You may not use the material for commercial purposes.

To view a copy of this license, visit [Creative Commons CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).
