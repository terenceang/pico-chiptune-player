# Hardware Documentation - Pico Chiptune Player

**Title:** Pico Chiptune Player  
**Author:** Terence Ang / MotionFxDesign  
**Company:** MotionFxDesign  
**Revision:** 0.2  
**Date:** 2026-08-12  

---

## Overview

The **Pico Chiptune Player** is a hardware chiptune music player powered by a **Raspberry Pi Pico 2 (RP2350)** microcontroller driving a dedicated **General Instrument AY-3-8910** (or **Yamaha YM2149F**) 3-voice Programmable Sound Generator (PSG).

It features passive 3-channel mono audio summing, YM2149F clock division jumper selection, an SPI microSD card interface with hardware Card Detect, an I2C OLED display header (`U3`), and navigation pushbuttons.

---

## Block Diagram

```
                             +-------------------+
                             |  Raspberry Pi     |
                             | Pico 2 (RP2350)   |
                             +--+---+---+---+----+
                                |   |   |   |
          +---------------------+   |   |   +-----------------------+
          | (SPI)                   |   | (I2C)                     | (GPIO12..15)
          v                         v   v                           v
+------------------+         +------------+                    +-----------+
| microSD Card     |         | 0.96" OLED |                    | Buttons   |
| Slot (J1)        |         | Display(U3)|                    | (SW1..3)  |
+------------------+         +------------+                    +-----------+
                                    |
            +-----------------------+
            | 8-Bit Data Bus (GPIO0..7)
            | Control (BDIR, BC1, ~RESET)
            | Clock Out (GPIO11 + 33Ω R2 Series Damping)
            v
  +--------------------+
  | AY-3-8910 / YM2149 | <--- JP1 (YM2149 /SEL Mode Select)
  +---------+----------+
            |
            | ANALOG_A, B, C
            v
  +--------------------+
  | Passive Mono Mixer | (R4=1k, R5=1k, R6=1k equal weight summer)
  +---------+----------+
            |
            v
  +--------------------+
  | Low-Pass & Load    | (R7=1kΩ load, C2=10nF LPF)
  +---------+----------+
            |
            v
  +--------------------+
  | AC Output Coupling | (C3=1µF)
  +---------+----------+
            |
            v
     2-Pin Audio Header (J2)
```

---

## Complete Pinout & Wiring Table

### 1. Raspberry Pi Pico 2 (A1) Pin Connections

| Pico Pin | GPIO Pin | Function / Net Name | Connected To | Signal Description |
| :--- | :--- | :--- | :--- | :--- |
| **Pin 1** | **GPIO0** | `DA0` | AY-3-8910 Pin 37 | PSG Data Bus Bit 0 |
| **Pin 2** | **GPIO1** | `DA1` | AY-3-8910 Pin 36 | PSG Data Bus Bit 1 |
| **Pin 3** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 4** | **GPIO2** | `DA2` | AY-3-8910 Pin 35 | PSG Data Bus Bit 2 |
| **Pin 5** | **GPIO3** | `DA3` | AY-3-8910 Pin 34 | PSG Data Bus Bit 3 |
| **Pin 6** | **GPIO4** | `DA4` | AY-3-8910 Pin 33 | PSG Data Bus Bit 4 |
| **Pin 7** | **GPIO5** | `DA5` | AY-3-8910 Pin 32 | PSG Data Bus Bit 5 |
| **Pin 8** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 9** | **GPIO6** | `DA6` | AY-3-8910 Pin 31 | PSG Data Bus Bit 6 |
| **Pin 10** | **GPIO7** | `DA7` | AY-3-8910 Pin 30 | PSG Data Bus Bit 7 |
| **Pin 11** | **GPIO8** | `BDIR` | — | Bus Direction (reserved; U1 BDIR tied to +5V) |
| **Pin 12** | **GPIO9** | `BC1` | AY-3-8910 Pin 29 | Bus Control 1 |
| **Pin 13** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 14** | **GPIO10** | `~RESET` | AY-3-8910 Pin 23 | PSG Active Low Reset (10kΩ pull-down R1 to GND for boot sequencing) |
| **Pin 15** | **GPIO11** | `CLOCK` | AY-3-8910 Pin 22 | PSG Master Clock (1.789773 MHz / 2 MHz, via 33Ω R2 series resistor) |
| **Pin 16** | **GPIO12** | `SW_PREV` | Switch SW3 Pin 1 | Previous Track Button Input (Internal Pull-up) |
| **Pin 17** | **GPIO13** | `SW_PLAY` | Switch SW1 Pin 1 | Play / Pause Button Input (Internal Pull-up) |
| **Pin 18** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 19** | **GPIO14** | `N/C` | — | Not Connected |
| **Pin 20** | **GPIO15** | `SW_NEXT` | Switch SW2 Pin 1 | Next Track Button Input (Internal Pull-up) |
| **Pin 21** | **GPIO16** | `OLED_SCL` | OLED Display U3 Pin 3 | I2C Clock Line (Internal Pull-up) |
| **Pin 22** | **GPIO17** | `OLED_SDA` | OLED Display U3 Pin 4 | I2C Data Line (Internal Pull-up) |
| **Pin 23** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 24** | **GPIO18** | `SD_CS` | MicroSD J1 Pin 2 | SPI Chip Select (CS, Internal Pull-up) |
| **Pin 25** | **GPIO19** | `SD_MOSI` | MicroSD J1 Pin 3 | SPI Master Out Slave In (DI) |
| **Pin 26** | **GPIO20** | `SD_SCK` | MicroSD J1 Pin 5 | SPI Serial Clock (SCK) |
| **Pin 27** | **GPIO21** | `SD_MISO` | MicroSD J1 Pin 7 | SPI Master In Slave Out (DO, Internal Pull-up) |
| **Pin 28** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 29** | **GPIO22** | `SD_Card_Detect` | MicroSD J1 Pin 9 | MicroSD Card Detect (Internal Pull-up) |
| **Pin 36** | **3V3** | `+3V3` | MicroSD J1 Pin 4, OLED U3 Pin 1 | 3.3V Power Out |
| **Pin 39** | **VSYS** | `N/C` | — | Not Connected (external supply input, unused) |
| **Pin 40** | **VBUS** | `+5V` | AY-3-8910 Pin 40 (VCC), PAM8302 U2 Pin 6 (VDD), pull-ups | 5V rail — sourced directly from USB VBUS (board is USB-powered only) |

---

### 2. AY-3-8910 / YM2149 Sound Generator (U1) Pinout

| Pin | Symbol | Net Name | Connected To | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **1** | VSS | `GND` | Ground Plane | Power Ground |
| **3** | ANALOG_B | `ANALOG_B` | Resistor R5 (1k) | Channel B Output |
| **4** | ANALOG_A | `ANALOG_A` | Resistor R4 (1k) | Channel A Output |
| **22** | CLOCK | `CLOCK` | Pico GPIO11 (via 33Ω R2 series) | Master Clock Input |
| **23** | ~RESET | `~RESET` | Pico GPIO10 | Active-low Reset (10kΩ pull-down R1 to GND for startup High-Z safety) |
| **24** | A9 | `GND` | Ground Plane | Fixed Address Decoding Bit (0) |
| **25** | A8 | `+5V` | +5V Rail | Fixed Address Decoding Bit (1) |
| **26** | TEST2 / /SEL | `TEST2_SEL` | Jumper JP1 Pin 2 | YM2149F Clock Select Jumper Header |
| **27** | BDIR | `+5V` | +5V Rail | Tied high permanently (always-write mode) |
| **28** | BC2 | `+5V` | +5V Rail | Fixed Bus Control Enable (1) |
| **29** | BC1 | `BC1` | Pico GPIO9 | Bus Control 1 |
| **30..37** | DA7..DA0 | `DA7..DA0` | Pico GPIO7..GPIO0 | 8-Bit Bi-directional Data Bus |
| **38** | ANALOG_C | `ANALOG_C` | Resistor R6 (1k) | Channel C Output |
| **40** | VCC | `+5V` | +5V Rail | Power Input (+5V, Decoupled by C1 10nF, C2 10nF) |
| **2, 5..20, 39** | NC / I/O | `NC` | Unconnected | General Purpose I/O Ports A/B (Unused) |

---

### 3. MicroSD Card Socket (J1) Pinout

| Pin | Card Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **1** | DAT2 | `NC` | Unconnected | Unused in SPI Mode |
| **2** | DAT3 / CD | `SD_CS` | Pico GPIO18 | SPI Chip Select (Internal Pull-up) |
| **3** | CMD | `SD_MOSI` | Pico GPIO19 | SPI Data Input (DI) |
| **4** | VDD | `+3V3` | Pico Pin 36 | 3.3V Power |
| **5** | CLK | `SD_SCK` | Pico GPIO20 | SPI Serial Clock |
| **6** | VSS | `GND` | Ground Plane | Power Ground |
| **7** | DAT0 | `SD_MISO` | Pico GPIO21 | SPI Data Output (DO, Internal Pull-up) |
| **8** | DAT1 | `NC` | Unconnected | Unused in SPI Mode |
| **9** | Card Detect | `SD_Card_Detect` | Pico GPIO22 | Active Low Card Detect Switch (Internal Pull-up) |
| **10** | Casing / Shield| `GND` | Ground Plane | Metal Shell Ground |

---

### 4. OLED Display Header (U3) & Buttons (SW1..SW3)

| Reference | Pin / Function | Net Name | Connected To |
| :--- | :--- | :--- | :--- |
| **U3 Pin 1** | Power | `+3V3` | Pico 3.3V Rail |
| **U3 Pin 2** | Ground | `GND` | Ground Plane |
| **U3 Pin 3** | SCL Clock | `OLED_SCL` | Pico GPIO16 (Pin 21, Internal Pull-up) |
| **U3 Pin 4** | SDA Data | `OLED_SDA` | Pico GPIO17 (Pin 22, Internal Pull-up) |
| **SW1 Pin 1** | Play / Pause | `SW_PLAY` | Pico GPIO13 (Pin 17, Internal Pull-up) |
| **SW2 Pin 1** | Next Track | `SW_NEXT` | Pico GPIO15 (Pin 20, Internal Pull-up) |
| **SW3 Pin 1** | Prev Track | `SW_PREV` | Pico GPIO12 (Pin 16, Internal Pull-up) |

---

## Power Supply & Audio Stage Specification

### 1. Power Supply & Decoupling
* **Power source**: The `+5V` rail is sourced from the Raspberry Pi Pico 2 **VBUS pin (Pin 40)** — i.e. USB bus voltage. **VSYS (Pin 39) is Not Connected**; the board is designed to be USB-powered only. Do **not** feed an external 5 V into the `+5V` rail while USB is connected, as it will backfeed into VBUS.
* **`C1`**: 10nF MLCC ceramic (0805) decoupling capacitor on `+5V` (adjacent to AY-3-8910 VCC Pin 40).
* **`C2`**: 10nF MLCC ceramic (0805) decoupling capacitor on the audio mixer output / LPF node.
* **Pull-up resistors**:
  * **`R1`** = 10kΩ pull-down to GND on `~RESET` (AY-3-8910 Pin 23), for boot-time high-Z safety before the Pico drives the pin.
  * **`R3`** = 10kΩ pull-up to `+3.3V` on `SD_CS` (Pico GPIO18 / MicroSD J1 Pin 2), ensures SD card sees CS high before Pico configures the GPIO (avoids the card entering SD-mode on power-up).
  * **`R8`** = 10kΩ pull-up to `+5V` on PAM8302 `~SD` (U2 Pin 1), keeps the amplifier enabled by default. Can optionally be driven low by an RP2350 GPIO for pop-free mute.

### 2. YM2149F Jumper Header (`JP1`)
* **Pin 1**: `+5V`
* **Pin 2**: AY-3-8910 Pin 26 (`TEST2` / `/SEL`)
* **Pin 3**: `GND`
  * *Position 1–2:* Internal clock divided by 1 (1:1 mode for YM2149F).
  * *Position 2–3:* Internal clock divided by 2 (1:2 mode for YM2149F).
  * *Open:* Floating for standard AY-3-8910.

### 3. Audio Stage & PAM8302A Mono Class-D Amplifier (`U2`)
* **Passive Mono Summer**: `ANALOG_A` (R4 = 1kΩ) + `ANALOG_B` (R5 = 1kΩ) + `ANALOG_C` (R6 = 1kΩ)
* **Passive Low-Pass Filter**: Resistor load `R7` = 1kΩ to GND, Capacitor `C2` = 10nF MLCC ceramic (0805) LPF to GND.
* **Volume Attenuator**: Potentiometer `RV1` (Bourns 3224W, 10kΩ) as a resistor divider before amplifier input.
* **Amplifier (PAM8302AAS - 2.5W Mono Class-D, SOIC-8 footprint)** — pin mapping per KiCad `Amplifier_Audio:PAM8302AAS` library symbol:
  * **Pin 1 (`~SD`)**: Shutdown control (active low). Pulled up to `+5V` via 10kΩ `R8`. Can optionally be driven by an RP2350 GPIO for pop-free mute.
  * **Pin 2 (`NC`)**: Not connected (hidden in symbol).
  * **Pin 3 (`IN+`)**: AC-coupled audio input via `C3` = 1µF polarized aluminum electrolytic (`CP_Elec_3x5.4` footprint) — positive terminal to signal side.
  * **Pin 4 (`IN-`)**: AC-coupled to GND via `C5` = 10nF MLCC ceramic (0805). Note: matching PAM8302 datasheet recommends this cap equal `C3` (1µF) for a flat audio HPF; the 10nF here creates a bass roll-off (intentional or accidental — verify).
  * **Pin 5 (`OUT+`)**: Positive Bridge-Tied Load (BTL) speaker output to `J2` Pin 1.
  * **Pin 6 (`VDD`)**: +5V power supply, decoupled by `C4` = 10µF MLCC ceramic (0805, `C_0805` footprint).
  * **Pin 7 (`GND`)**: System ground.
  * **Pin 8 (`OUT-`)**: Negative Bridge-Tied Load (BTL) speaker output to `J2` Pin 2 (**do NOT ground!**).
* **Speaker Output Header (`J2`)**: 2-pin 1.27 mm pin socket for 4Ω or 8Ω mono speaker (`OUT+` on Pin 1, `OUT-` on Pin 2).

---

## Revision History

| Rev  | Date       | Notes |
| :--- | :--------- | :---- |
| 0.2  | 2026-08-12 | Corrected `+5V` source from Pico Pin 39 (VSYS) to Pin 40 (VBUS); documented `R8` pull-up on PAM8302 `~SD`; corrected `R3` role to `SD_CS` pull-up; re-mapped PAM8302 pinout to match KiCad `Amplifier_Audio:PAM8302AAS` library symbol; updated capacitor types (`C3` polarized electrolytic, `C4` MLCC ceramic) to match schematic footprints. Schematic-side: added `no_connect` markers on unused Pico + AY-3-8910 pins; set `C1` footprint. Schematic is the single source of truth. |
| 0.1  | 2026-08-08 | Initial hardware documentation release. |
