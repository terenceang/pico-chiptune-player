# Hardware Documentation - Pico Chiptune Player

**Title:** Pico Chiptune Player  
**Author:** Terence Ang / MotionFxDesign  
**Company:** MotionFxDesign  
**Revision:** 0.2  
**Date:** 2026-08-12  

---

## Overview

The **Pico Chiptune Player** is a hardware chiptune music player powered by a **Raspberry Pi Pico 2 (RP2350)** microcontroller driving a dedicated **General Instrument AY-3-8910** (or **Yamaha YM2149F**) 3-voice Programmable Sound Generator (PSG).

It features passive 3-channel mono audio summing, YM2149F clock division jumper selection, an SPI microSD card interface with hardware Card Detect, an I2C OLED display header (`U2`), an onboard **PAM8302A 2.5W Mono Class-D Audio Amplifier** (`U3`), and navigation pushbuttons.

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
| Slot (J2)        |         | Display(U2)|                    | (SW1..3)  |
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
  | Passive Mono Mixer | (R4=1k, R5=1k, R7=1k equal weight summer + R8=1k load)
  +---------+----------+
            |
            v
  +--------------------+
  | Volume Attenuator  | (RV1=10k Trimmer Potentiometer)
  +---------+----------+
            |
            v
  +--------------------+
  | AC Output Coupling | (C3=1µF AC Coupling Cap)
  +---------+----------+
            |
            v
  +--------------------+
  | Mono Audio Amp     | (U3: PAM8302A 2.5W Mono Class-D, C4=10nF AC Ref GND)
  +---------+----------+
            |
            v
     2-Pin Speaker Header (J1)
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
| **Pin 21** | **GPIO16** | `OLED_SCL` | OLED Display U2 Pin 3 | I2C Clock Line (Internal Pull-up) |
| **Pin 22** | **GPIO17** | `OLED_SDA` | OLED Display U2 Pin 4 | I2C Data Line (Internal Pull-up) |
| **Pin 23** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 24** | **GPIO18** | `SD_CS` | MicroSD J2 Pin 2 | SPI Chip Select (CS, Internal Pull-up) |
| **Pin 25** | **GPIO19** | `SD_MOSI` | MicroSD J2 Pin 3 | SPI Master Out Slave In (DI) |
| **Pin 26** | **GPIO20** | `SD_SCK` | MicroSD J2 Pin 5 | SPI Serial Clock (SCK) |
| **Pin 27** | **GPIO21** | `SD_MISO` | MicroSD J2 Pin 7 | SPI Master In Slave Out (DO, Internal Pull-up) |
| **Pin 28** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 29** | **GPIO22** | `SD_Card_Detect` | MicroSD J2 Pin 9 | MicroSD Card Detect (Internal Pull-up) |
| **Pin 36** | **3V3** | `+3V3` | MicroSD J2 Pin 4, OLED U2 Pin 1 | 3.3V Power Out |
| **Pin 39** | **VSYS** | `N/C` | — | Not Connected (external supply input, unused) |
| **Pin 40** | **VBUS** | `+5V` | AY-3-8910 Pin 40 (VCC), PAM8302 U3 Pin 6 (VDD), pull-ups | 5V rail — sourced directly from USB VBUS (board is USB-powered only) |

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
| **38** | ANALOG_C | `ANALOG_C` | Resistor R7 (1k) | Channel C Output |
| **40** | VCC | `+5V` | +5V Rail | Power Input (+5V, Decoupled by C1 10µF, C2 100nF) |
| **2, 5..20, 39** | NC / I/O | `NC` | Unconnected | General Purpose I/O Ports A/B (Unused) |

---

### 3. MicroSD Card Socket (J2) Pinout

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

### 4. OLED Display Header (U2), Audio Amp (U3), & Buttons (SW1..SW3)

| Reference | Pin / Function | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **U2 Pin 1** | Power | `+3V3` | Pico 3.3V Rail | Display Power |
| **U2 Pin 2** | Ground | `GND` | Ground Plane | Display Ground |
| **U2 Pin 3** | SCL Clock | `OLED_SCL` | Pico GPIO16 (Internal Pull-up) | I2C Clock Line |
| **U2 Pin 4** | SDA Data | `OLED_SDA` | Pico GPIO17 (Internal Pull-up) | I2C Data Line |
| **U3 Pin 1** | ~SD (Shutdown)| `Net-(R6-Pad2)` | +5V Rail via 10kΩ `R6` | Active-low Shutdown (Pulled HIGH to enable amp) |
| **U3 Pin 3** | IN+ | `Net-(C3-Pad2)`| `C3` (1µF) negative lead | AC-coupled audio input from `RV1` wiper |
| **U3 Pin 4** | IN- | `Net-(C4-Pad1)`| `C4` (10nF) MLCC to GND | AC reference ground (preserves 2.5V differential bias) |
| **U3 Pin 5** | OUT+ | `Net-(J1-Pin_2)`| Speaker Header `J1` Pin 2 | Positive BTL Speaker Output |
| **U3 Pin 6** | VDD | `+5V` | +5V Rail | Power Supply |
| **U3 Pin 7** | GND | `GND` | Ground Plane | Power Ground |
| **U3 Pin 8** | OUT- | `Net-(J1-Pin_1)`| Speaker Header `J1` Pin 1 | Negative BTL Speaker Output |
| **SW1 Pin 1** | Play / Pause | `SW_PLAY` | Pico GPIO13 (Internal Pull-up) | Navigation Button |
| **SW2 Pin 1** | Next Track | `SW_NEXT` | Pico GPIO15 (Internal Pull-up) | Navigation Button |
| **SW3 Pin 1** | Prev Track | `SW_PREV` | Pico GPIO12 (Internal Pull-up) | Navigation Button |

---

## Bill of Materials & LCSC Part Mapping (SSOT)

| Designator | Value / Part | Package / Footprint | Quantity | LCSC Part # | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`A1`** | `RaspberryPi_Pico` | `Module_RaspberryPi_Pico` | 1 | `C22548` | Raspberry Pi Pico 2 (RP2350) Microcontroller |
| **`C1`** | `10uF` | `CP_Elec_3x5.4` | 1 | `C249450` | Aluminum Electrolytic Bulk Power Decoupling Cap |
| **`C2`** | `100nF` | `C_0805` | 1 | `C1711` | MLCC 0805 High-Frequency Decoupling Cap |
| **`C3`** | `1uF` | `CP_Elec_3x5.4` | 1 | `C249446` | Polarized Audio Input AC Coupling Cap |
| **`C4`** | `10nF` | `C_0805` | 1 | `C1710` | MLCC 0805 PAM8302 IN- AC Reference Cap |
| **`C5`** | `10nF` | `C_0805` | 1 | `C1710` | MLCC 0805 Audio Low-Pass Filter Cap |
| **`J1`** | `Spk Out` | `PinSocket_2x01_P1.27mm` | 1 | `C12437` | 2-Pin Speaker Output Header |
| **`J2`** | `MicroSD` | `MicroSD` | 1 | `C114217` | TF-1115 MicroSD Card Socket |
| **`JP1`**| `Clk Div` | `SolderJumper-2` | 1 | — | YM2149 Clock Divider Select Header |
| **`R1`** | `10K` | `R_0805` | 1 | `C1297` | AY-3-8910 ~RESET Boot Pull-Down Resistor |
| **`R2`** | `33` | `R_0805` | 1 | `C17631` | Master Clock Series Damping Resistor |
| **`R3`** | `10K` | `R_0805` | 1 | `C17414` | MicroSD SPI CS Boot Pull-Up Resistor |
| **`R4`** | `1k` | `R_0805` | 1 | `C17513` | Channel A Summer Resistor |
| **`R5`** | `1k` | `R_0805` | 1 | `C17513` | Channel B Summer Resistor |
| **`R6`** | `10K` | `R_0805` | 1 | `C1297` | PAM8302 ~SD Shutdown Pull-Up Resistor |
| **`R7`** | `1k` | `R_0805` | 1 | `C17513` | Channel C Summer Resistor |
| **`R8`** | `1K` | `R_0805` | 1 | `C17513` | Audio Mixer Load Resistor to GND |
| **`RV1`**| `3224W-1-103E` | `Bourns_3224W` | 1 | `C81348` | 10k Trimmer Volume Control Potentiometer |
| **`SW1..3`**| `SW_PUSH_6mm` | `SW_PUSH_6mm` | 3 | `C12740` | 6x6mm Pushbutton Tactile Switches |
| **`U1`** | `AY-3-8910` | `DIP-40_W15.24mm` | 1 | `C42878` | 3-Voice Programmable Sound Generator IC |
| **`U2`** | `0.96"OLED_I2C` | `0.96OLED I2C` | 1 | `C22548` | 4-Pin I2C OLED Display Header |
| **`U3`** | `PAM8302AAS` | `SOIC-8_3.9x4.9mm` | 1 | `C8302` | 2.5W Mono Class-D Audio Amplifier IC |

---

## Revision History

| Rev | Date | Notes |
| :--- | :--- | :--- |
| **0.2** | **2026-08-12** | **Single Source of Truth (SSOT) Release**: Updated all component designators (`U2` OLED, `U3` PAM8302AAS, `J1` Speaker Out, `J2` MicroSD, `R4/R5/R7` summer resistors, `R8` load resistor, `R6` shutdown pull-up). Integrated full LCSC part numbers and verified Adafruit-compliant PAM8302 audio circuit topology. |
| 0.1 | 2026-08-08 | Initial hardware documentation release. |
