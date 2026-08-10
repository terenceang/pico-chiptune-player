# Hardware Documentation - Pico Chiptune Player

**Title:** Pico Chiptune Player  
**Author:** Terence Ang / MotionFxDesign  
**Company:** MotionFxDesign  
**Revision:** 0.01  
**Date:** 2026-08-10  

---

## Overview

The **Pico Chiptune Player** is a hardware chiptune music player powered by a **Raspberry Pi Pico (RP2040)** microcontroller driving a dedicated **General Instrument AY-3-8910** (or **Yamaha YM2149F**) 3-voice Programmable Sound Generator (PSG).

It features passive 3-channel mono audio summing, YM2149F clock division jumper selection, an SPI microSD card interface with hardware Card Detect, an I2C OLED display header (`U2`), and navigation pushbuttons.

---

## Block Diagram

```
                             +-------------------+
                             |  Raspberry Pi     |
                             |  Pico (RP2040)    |
                             +--+---+---+---+----+
                                |   |   |   |
          +---------------------+   |   |   +-----------------------+
          | (SPI0)                  |   | (I2C1)                    | (GPIO12..14)
          v                         v   v                           v
+------------------+         +------------+                    +-----------+
| microSD Card     |         | 0.96" OLED |                    | Buttons   |
| Slot (J1)        |         | Display(U2)|                    | (SW1..3)  |
+------------------+         +------------+                    +-----------+
                                    |
            +-----------------------+
            | 8-Bit Data Bus (GPIO0..7)
            | Control (BDIR, BC1, ~RESET)
            | Clock Out (GPIO11 + 33Ω R7 Series Damping)
            v
  +--------------------+
  | AY-3-8910 / YM2149 | <--- JP1 (YM2149 /SEL Mode Select)
  +---------+----------+
            |
            | ANALOG_A, B, C
            v
  +--------------------+
  | Passive Mono Mixer | (R3=1k, R4=1k, R5=1k equal weight summer)
  +---------+----------+
            |
            v
  +--------------------+
  | Low-Pass & Load    | (R6=1kΩ load, C3=10nF LPF)
  +---------+----------+
            |
            v
  +--------------------+
  | AC Output Coupling | (C4=10µF)
  +---------+----------+
            |
            v
     2-Pin Audio Header (J2)
```

---

## Complete Pinout & Wiring Table

### 1. Raspberry Pi Pico (A1) Pin Connections

| Pico Pin | GPIO Pin | Function / Net Name | Connected To | Signal Description |
| :--- | :--- | :--- | :--- | :--- |
| **Pin 1** | **GPIO0** | `DA0` | AY-3-8910 Pin 37 | PSG Data Bus Bit 0 |
| **Pin 2** | **GPIO1** | `DA1` | AY-3-8910 Pin 36 | PSG Data Bus Bit 1 |
| **Pin 3** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 4** | **GPIO2** | `OLED_SDA` | OLED Display U2 Pin 4 | I2C Data Line (Internal Pull-up) |
| **Pin 5** | **GPIO3** | `OLED_SCL` | OLED Display U2 Pin 3 | I2C Clock Line (Internal Pull-up) |
| **Pin 6** | **GPIO4** | `DA4` | AY-3-8910 Pin 33 | PSG Data Bus Bit 4 |
| **Pin 7** | **GPIO5** | `DA5` | AY-3-8910 Pin 32 | PSG Data Bus Bit 5 |
| **Pin 8** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 9** | **GPIO6** | `DA6` | AY-3-8910 Pin 31 | PSG Data Bus Bit 6 |
| **Pin 10** | **GPIO7** | `DA7` | AY-3-8910 Pin 30 | PSG Data Bus Bit 7 |
| **Pin 11** | **GPIO8** | `BDIR` | AY-3-8910 Pin 27 | Bus Direction Control |
| **Pin 12** | **GPIO9** | `BC1` | AY-3-8910 Pin 29 | Bus Control 1 |
| **Pin 13** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 14** | **GPIO10** | `~RESET` | AY-3-8910 Pin 23 | PSG Active Low Reset (10kΩ pull-down R1 to GND for boot sequencing) |
| **Pin 15** | **GPIO11** | `CLOCK` | AY-3-8910 Pin 22 | PSG Master Clock (1.789773 MHz / 2 MHz, via 33Ω R7 series resistor) |
| **Pin 16** | **GPIO12** | `SW_PLAY` | Switch SW1 Pin 1 | Play / Pause Button Input (Internal Pull-up) |
| **Pin 17** | **GPIO13** | `SW_NEXT` | Switch SW2 Pin 1 | Next Track Button Input (Internal Pull-up) |
| **Pin 18** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 19** | **GPIO14** | `SW_PREV` | Switch SW3 Pin 1 | Previous Track Button Input (Internal Pull-up) |
| **Pin 20** | **GPIO15** | `SD_CD` | MicroSD J1 Pin 9 | MicroSD Card Detect (Internal Pull-up) |
| **Pin 21** | **GPIO16** | `SD_MISO` | MicroSD J1 Pin 7 | SPI0 Master In Slave Out (DO, Internal Pull-up) |
| **Pin 22** | **GPIO17** | `SD_CS` | MicroSD J1 Pin 2 | SPI0 Chip Select (CS, Internal Pull-up) |
| **Pin 23** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 24** | **GPIO18** | `SD_SCK` | MicroSD J1 Pin 5 | SPI0 Serial Clock (SCK) |
| **Pin 25** | **GPIO19** | `SD_MOSI` | MicroSD J1 Pin 3 | SPI0 Master Out Slave In (DI) |
| **Pin 36** | **3V3** | `+3V3` | MicroSD J1 Pin 4, OLED U2 Pin 1 | 3.3V Power Out |
| **Pin 40** | **VBUS** | USB 5V In | AY-3-8910 Pin 40 (+5V) | USB Power Supply Input |

---

### 2. AY-3-8910 / YM2149 Sound Generator (U1) Pinout

| Pin | Symbol | Net Name | Connected To | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **1** | VSS | `GND` | Ground Plane | Power Ground |
| **3** | ANALOG_B | `ANALOG_B` | Resistor R4 (1k) | Channel B Output |
| **4** | ANALOG_A | `ANALOG_A` | Resistor R3 (1k) | Channel A Output |
| **22** | CLOCK | `CLOCK` | Pico GPIO11 (via 33Ω R7 series) | Master Clock Input |
| **23** | ~RESET | `~RESET` | Pico GPIO10 | Active-low Reset (10kΩ pull-down R1 to GND for startup High-Z safety) |
| **24** | A9 | `GND` | Ground Plane | Fixed Address Decoding Bit (0) |
| **25** | A8 | `+5V` | +5V Rail | Fixed Address Decoding Bit (1) |
| **26** | TEST2 / /SEL | `TEST2_SEL` | Jumper JP1 Pin 2 | YM2149F Clock Select Jumper Header |
| **27** | BDIR | `BDIR` | Pico GPIO8 | Bus Direction Control |
| **28** | BC2 | `+5V` | +5V Rail | Fixed Bus Control Enable (1) |
| **29** | BC1 | `BC1` | Pico GPIO9 | Bus Control 1 |
| **30..37** | DA7..DA0 | `DA7..DA0` | Pico GPIO7..GPIO0 | 8-Bit Bi-directional Data Bus |
| **38** | ANALOG_C | `ANALOG_C` | Resistor R5 (1k) | Channel C Output |
| **40** | VCC | `+5V` | +5V Rail | Power Input (+5V, Decoupled by C1 10µF, C2 10nF) |
| **2, 5..20, 39** | NC / I/O | `NC` | Unconnected | General Purpose I/O Ports A/B (Unused) |

---

### 3. MicroSD Card Socket (J1) Pinout

| Pin | Card Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **1** | DAT2 | `NC` | Unconnected | Unused in SPI Mode |
| **2** | DAT3 / CD | `SD_CS` | Pico GPIO17 | SPI Chip Select (Internal Pull-up) |
| **3** | CMD | `SD_MOSI` | Pico GPIO19 | SPI Data Input (DI) |
| **4** | VDD | `+3V3` | Pico Pin 36 | 3.3V Power |
| **5** | CLK | `SD_SCK` | Pico GPIO18 | SPI Serial Clock |
| **6** | VSS | `GND` | Ground Plane | Power Ground |
| **7** | DAT0 | `SD_MISO` | Pico GPIO16 | SPI Data Output (DO, Internal Pull-up) |
| **8** | DAT1 | `NC` | Unconnected | Unused in SPI Mode |
| **9** | Card Detect | `SD_CD` | Pico GPIO15 | Active Low Card Detect Switch (Internal Pull-up) |
| **10** | Casing / Shield| `GND` | Ground Plane | Metal Shell Ground |

---

### 4. OLED Display Header (U2) & Buttons (SW1..SW3)

| Reference | Pin / Function | Net Name | Connected To |
| :--- | :--- | :--- | :--- |
| **U2 Pin 1** | Power | `+3V3` | Pico 3.3V Rail |
| **U2 Pin 2** | Ground | `GND` | Ground Plane |
| **U2 Pin 3** | SCL Clock | `OLED_SCL` | Pico GPIO3 (Pin 5, Internal Pull-up) |
| **U2 Pin 4** | SDA Data | `OLED_SDA` | Pico GPIO2 (Pin 4, Internal Pull-up) |
| **SW1 Pin 1** | Play / Pause | `SW_PLAY` | Pico GPIO12 (Pin 16, Internal Pull-up) |
| **SW2 Pin 1** | Next Track | `SW_NEXT` | Pico GPIO13 (Pin 17, Internal Pull-up) |
| **SW3 Pin 1** | Prev Track | `SW_PREV` | Pico GPIO14 (Pin 19, Internal Pull-up) |

---

## Power Supply & Audio Stage Specification

### 1. Power Supply & Decoupling
* **`C1`**: 10µF bulk decoupling capacitor on `+5V`.
* **`C2`**: 10nF ceramic decoupling capacitor adjacent to AY-3-8910 VCC Pin 40.

### 2. YM2149F Jumper Header (`JP1`)
* **Pin 1**: `+5V`
* **Pin 2**: AY-3-8910 Pin 26 (`TEST2` / `/SEL`)
* **Pin 3**: `GND`
  * *Position 1–2:* Internal clock divided by 1 (1:1 mode for YM2149F).
  * *Position 2–3:* Internal clock divided by 2 (1:2 mode for YM2149F).
  * *Open:* Floating for standard AY-3-8910.

### 3. Audio Stage & PAM8302A Mono Class-D Amplifier
* **Passive Mono Summer**: `ANALOG_A` (R3 = 1kΩ) + `ANALOG_B` (R4 = 1kΩ) + `ANALOG_C` (R5 = 1kΩ)
* **Passive Low-Pass Filter**: Resistor load `R6` = 1kΩ to GND, Capacitor `C3` = 10nF LPF to GND.
* **Volume Attenuator**: 10kΩ Potentiometer / Resistor Divider before amplifier input (`IN+`).
* **Amplifier (PAM8302A - 2.5W Mono Class-D)**:
  * **Pin 1 (`IN+`)**: AC-coupled audio input via `C4` (1µF ceramic).
  * **Pin 2 (`~SD`)**: Shutdown Control (Active Low). Pulled up to `+5V` via 10kΩ resistor (or connected to RP2040 GPIO for pop-free mute control).
  * **Pin 3 (`IN-`)**: AC-coupled to GND via 1µF ceramic capacitor for differential noise rejection / high PSRR.
  * **Pin 4, 8 (`GND`)**: System Ground.
  * **Pin 5 (`VO+`)**: Positive Bridge-Tied Load (BTL) Speaker Output to `J2` Pin 1.
  * **Pin 6 (`VDD`)**: +5V Power Supply (Decoupled by 10µF electrolytic `C5` + 0.1µF ceramic `C6`).
  * **Pin 7 (`VO-`)**: Negative Bridge-Tied Load (BTL) Speaker Output to `J2` Pin 2 (Do NOT ground!).
* **Speaker Output Header (`J2`)**: 2-pin header for 4Ω or 8Ω mono speaker (`VO+` on Pin 1, `VO-` on Pin 2).

