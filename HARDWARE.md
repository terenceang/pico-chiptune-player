# Hardware Documentation - Pico Chiptune Player

**Title:** Pico Chiptune Player  
**Author:** Terence Ang <terenceang@mac.com> / MotionFxDesign  
**Company:** MotionFxDesign  
**Status:** Manufactured / Sent to Production (Schematic SSOT)  
**Version / Rev:** 0.4  
**Date:** 2026-08-15  
**License:** Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) — NOT FOR COMMERCIAL USE  


---

## 1. Overview

The **Pico Chiptune Player** is a dedicated hardware chiptune music player powered by a **Raspberry Pi Pico 2 (RP2350)** microcontroller driving a real **General Instrument AY-3-8910** (or **Yamaha YM2149F**) 3-voice Programmable Sound Generator (PSG).

### Key Features & Subsystems
- **Microcontroller:** Raspberry Pi Pico 2 (RP2350) microcontroller running the chiptune player firmware, file parser, and UI.
- **Sound Generator:** AY-3-8910 / YM2149F 40-pin DIP PSG interfaced via an 8-bit parallel bus (`DA0..DA7`), bus control lines (`BC1`, `BDIR`), active-low reset (`~RESET`), and a hardware master clock output with series damping.
- **Audio Summer & Filter:** 3-channel passive resistive summing network (`R4`, `R5`, `R7` = 1kΩ, `R8` = 1kΩ load) with high-cut low-pass filtering (`C4` = 100nF) and AC output coupling (`C3` = 1µF).
- **Audio Amplifier:** Onboard **Adafruit PAM8302A 2.5W Mono Class-D Audio Amplifier Module** (`U3`) header with hardware GPIO shutdown/mute control (`SD_Amp`).
- **Speaker Output:** Direct differential `OUT+` / `OUT-` speaker terminals on the PAM8302 module breakout.
- **Display:** 0.96" 128x64 I2C OLED display module (`U2`) with bulk and high-frequency decoupling (`C6` = 10µF, `C7` = 100nF).
- **Storage:** Full SPI microSD card socket (`J2`) with hardware card detect (`SD_Card_Detect`) and dedicated pull-up (`R1` = 10kΩ).
- **Navigation Controls:** 3 tactile pushbuttons (`SW1` Play/Pause, `SW2` Next Track, `SW3` Previous Track).
- **Clock Divider Select:** Solder jumper `JP1` to select YM2149 internal ÷2 clock division.

---

## 2. Block Diagram

```
                             +-------------------+
                             |  Raspberry Pi     |
                             | Pico 2 (RP2350)   |
                             +--+---+---+---+----+
                                |   |   |   |
          +---------------------+   |   |   +-----------------------+
          | (SPI: GPIO18..22)       |   | (I2C: GPIO16, 17)         | (GPIO13, 14, 15)
          v                         v   v                           v
+------------------+         +------------+                    +-----------+
| microSD Card     |         | 0.96" OLED |                    | Buttons   |
| Slot (J2)        |         | Display(U2)|                    | (SW1..3)  |
+------------------+         +------------+                    +-----------+
                                    |
            +-----------------------+
            | 8-Bit Parallel Data Bus: DA0..DA7 (GPIO0..7)
            | Bus Control: BC1 (GPIO8), BDIR (GPIO10)
            | Active-Low Reset: ~RESET (GPIO11) + 10kΩ R3 Pull-down
            | Master Clock: CLOCK (GPIO12) + 33Ω R2 Series Damping
            | Amp Shutdown Control: SD_Amp (GPIO9)
            v
  +--------------------+
  | AY-3-8910 / YM2149 | <--- JP1 (YM2149 /SEL ÷2 Clock Mode Jumper)
  +---------+----------+
            |
            | ANALOG_A, ANALOG_B, ANALOG_C
            v
  +--------------------+
  | Passive Mono Mixer | (R4=1k, R5=1k, R7=1k Summing + R8=1k Load + C4=100nF LPF)
  +---------+----------+
            |
            v
  +--------------------+
  | AC Output Coupling | (C3 = 1µF Polarized Electrolytic Cap)
  +---------+----------+
            |
            | Net-(U3-A+)
            v
  +--------------------+
  | Mono Audio Amp     | (U3: Adafruit PAM8302A 2.5W Class-D Module)
  | Breakout Header    | (VIN=+5V with C8=100nF, A-=GND, SD=GPIO9 SD_Amp)
  +---------+----------+
            |
            v
     Differential Speaker Terminals (OUT+ / OUT- on U3)
```

---

## 3. Complete Pinout & Wiring Tables (SSOT)

### 3.1 Raspberry Pi Pico 2 (A1) Pin Connections

| Pico Pin | Pin Name / GPIO | Net Name | Connected To | Signal Description |
| :--- | :--- | :--- | :--- | :--- |
| **Pin 1** | **GPIO0** | `DA0` | AY-3-8910 (U1) Pin 37 | PSG 8-Bit Data Bus Bit 0 |
| **Pin 2** | **GPIO1** | `DA1` | AY-3-8910 (U1) Pin 36 | PSG 8-Bit Data Bus Bit 1 |
| **Pin 3** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 4** | **GPIO2** | `DA2` | AY-3-8910 (U1) Pin 35 | PSG 8-Bit Data Bus Bit 2 |
| **Pin 5** | **GPIO3** | `DA3` | AY-3-8910 (U1) Pin 34 | PSG 8-Bit Data Bus Bit 3 |
| **Pin 6** | **GPIO4** | `DA4` | AY-3-8910 (U1) Pin 33 | PSG 8-Bit Data Bus Bit 4 |
| **Pin 7** | **GPIO5** | `DA5` | AY-3-8910 (U1) Pin 32 | PSG 8-Bit Data Bus Bit 5 |
| **Pin 8** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 9** | **GPIO6** | `DA6` | AY-3-8910 (U1) Pin 31 | PSG 8-Bit Data Bus Bit 6 |
| **Pin 10** | **GPIO7** | `DA7` | AY-3-8910 (U1) Pin 30 | PSG 8-Bit Data Bus Bit 7 |
| **Pin 11** | **GPIO8** | `BC1` | AY-3-8910 (U1) Pin 29 | Bus Control 1 (Address Latch / Data Write) |
| **Pin 12** | **GPIO9** | `SD_Amp` | PAM8302 Module (U3) Pin 3 | Audio Amplifier Shutdown Control (Active High Enable / Low Mute) |
| **Pin 13** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 14** | **GPIO10** | `BDIR` | AY-3-8910 (U1) Pin 27 | Bus Direction Control |
| **Pin 15** | **GPIO11** | `~RESET` | AY-3-8910 (U1) Pin 23, R3 | PSG Active-Low Reset (with 10kΩ pull-down R3 to GND) |
| **Pin 16** | **GPIO12** | `Net-(A1-GPIO12)` | Resistor R2 Pin 2 (33Ω) | PSG Master Clock Output (series damped to U1 Pin 22 `CLOCK`) |
| **Pin 17** | **GPIO13** | `SW_NEXT` | Tact Switch SW2 Pin 1 | Next Track Button (Active Low, Internal Pull-up) |
| **Pin 18** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 19** | **GPIO14** | `SW_PLAY` | Tact Switch SW1 Pin 1 | Play / Pause Button (Active Low, Internal Pull-up) |
| **Pin 20** | **GPIO15** | `SW_PREV` | Tact Switch SW3 Pin 1 | Previous Track Button (Active Low, Internal Pull-up) |
| **Pin 21** | **GPIO16** | `OLED_SCL` | OLED Display (U2) Pin 3 | I2C Serial Clock (SCL, Internal / Display Pull-up) |
| **Pin 22** | **GPIO17** | `OLED_SDA` | OLED Display (U2) Pin 4 | I2C Serial Data (SDA, Internal / Display Pull-up) |
| **Pin 23** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 24** | **GPIO18** | `SD_CS` | MicroSD (J2) Pin 2, R1 | SPI Chip Select (CS, with 10kΩ pull-up R1 to +3.3V) |
| **Pin 25** | **GPIO19** | `SD_MOSI` | MicroSD (J2) Pin 3 | SPI Master Out Slave In (CMD / DI) |
| **Pin 26** | **GPIO20** | `SD_SCK` | MicroSD (J2) Pin 5 | SPI Serial Clock (CLK) |
| **Pin 27** | **GPIO21** | `SD_MISO` | MicroSD (J2) Pin 7 | SPI Master In Slave Out (DAT0 / DO) |
| **Pin 28** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 29** | **GPIO22** | `SD_Card_Detect` | MicroSD (J2) Pin 9 | MicroSD Card Detect (Active Low to GND when inserted) |
| **Pin 30** | **RUN** | `N/C` | — | Reset input (Unconnected) |
| **Pin 31** | **GPIO26_ADC0** | `N/C` | — | Analog input 0 (Unconnected) |
| **Pin 32** | **GPIO27_ADC1** | `N/C` | — | Analog input 1 (Unconnected) |
| **Pin 33** | **AGND** | `N/C` | — | Analog ground (Unconnected) |
| **Pin 34** | **GPIO28_ADC2** | `N/C` | — | Analog input 2 (Unconnected) |
| **Pin 35** | **ADC_VREF** | `N/C` | — | ADC reference (Unconnected) |
| **Pin 36** | **3V3** | `+3.3V` | MicroSD J2, OLED U2, R1, C5..C7 | 3.3V System Power Rail (Sourced from Pico onboard regulator) |
| **Pin 37** | **3V3_EN** | `N/C` | — | 3.3V regulator enable (Unconnected) |
| **Pin 38** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 39** | **VSYS** | `N/C` | — | System supply input (Unconnected) |
| **Pin 40** | **VBUS** | `+5V` | U1 Pin 40/25/28, U3 Pin 2, C1, C2, C8 | 5V Main Power Rail (Sourced from Pico USB VBUS) |

---

### 3.2 AY-3-8910 / YM2149 Sound Generator (U1) Pinout

| Pin | Symbol | Net Name | Connected To | Notes / Function |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `VSS` | `GND` | Ground Plane | Power Ground |
| **2** | `NC` | `NC` | Unconnected | No Connect |
| **3** | `ANALOG_B` | `ANALOG_B` | Resistor R5 (1kΩ) | Channel B Analog Audio Output |
| **4** | `ANALOG_A` | `ANALOG_A` | Resistor R4 (1kΩ) | Channel A Analog Audio Output |
| **5** | `NC` | `NC` | Unconnected | No Connect |
| **6..13** | `IOB7..IOB0` | `NC` | Unconnected | General Purpose I/O Port B (Unused) |
| **14..21**| `IOA7..IOA0` | `NC` | Unconnected | General Purpose I/O Port A (Unused) |
| **22** | `CLOCK` | `CLOCK` | Resistor R2 Pin 1 (33Ω) | Master Clock Input (Driven from Pico GPIO12 via 33Ω series R2) |
| **23** | `~RESET` | `~RESET` | Pico GPIO11, R3 (10kΩ to GND) | Active-Low Hardware Reset (Boot pull-down holds PSG in reset) |
| **24** | `A9` | `GND` | Ground Plane | Fixed Address Decoding Bit 9 (Tied to GND = 0) |
| **25** | `A8` | `+5V` | +5V Rail | Fixed Address Decoding Bit 8 (Tied to +5V = 1) |
| **26** | `~{CS/Test_2}` | `NET-(JP1-A)` | Solder Jumper JP1 Pin 1 | YM2149 Clock Select (/SEL): Open = 1x Clock; Bridged to GND = ÷2 Clock |
| **27** | `BDIR` | `BDIR` | Pico GPIO10 | Bus Direction Control Line |
| **28** | `BC2` | `+5V` | +5V Rail | Bus Control 2 (Tied permanently HIGH to +5V) |
| **29** | `BC1` | `BC1` | Pico GPIO8 | Bus Control 1 Line |
| **30** | `DA7` | `DA7` | Pico GPIO7 | 8-Bit Bidirectional Data Bus Bit 7 |
| **31** | `DA6` | `DA6` | Pico GPIO6 | 8-Bit Bidirectional Data Bus Bit 6 |
| **32** | `DA5` | `DA5` | Pico GPIO5 | 8-Bit Bidirectional Data Bus Bit 5 |
| **33** | `DA4` | `DA4` | Pico GPIO4 | 8-Bit Bidirectional Data Bus Bit 4 |
| **34** | `DA3` | `DA3` | Pico GPIO3 | 8-Bit Bidirectional Data Bus Bit 3 |
| **35** | `DA2` | `DA2` | Pico GPIO2 | 8-Bit Bidirectional Data Bus Bit 2 |
| **36** | `DA1` | `DA1` | Pico GPIO1 | 8-Bit Bidirectional Data Bus Bit 1 |
| **37** | `DA0` | `DA0` | Pico GPIO0 | 8-Bit Bidirectional Data Bus Bit 0 |
| **38** | `ANALOG_C` | `ANALOG_C` | Resistor R7 (1kΩ) | Channel C Analog Audio Output |
| **39** | `TEST1` | `NC` | Unconnected | Test pin (Unconnected) |
| **40** | `VCC` | `+5V` | +5V Rail | Main +5V Power Supply (Decoupled by C1 10µF bulk + C2 100nF MLCC) |

#### AY-3-8910 Bus State Decoding Table (`BC2` Tied HIGH to +5V)

| `BDIR` (GPIO10) | `BC1` (GPIO8) | Bus State | Operation Description |
| :---: | :---: | :--- | :--- |
| **0** | **0** | **INACTIVE** | PSG data bus is High-Z / Inactive. |
| **0** | **1** | **READ** | Pico reads register contents from PSG data bus `DA[7:0]`. |
| **1** | **0** | **WRITE** | Pico writes data on `DA[7:0]` into currently addressed register. |
| **1** | **1** | **LATCH ADDRESS** | Pico selects target PSG register address (0x00..0x0F) on `DA[7:0]`. |

---

### 3.3 MicroSD Card Socket (J2) Pinout

| Pin | Card Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `DAT2` | `NC` | Unconnected | Unused in SPI Mode |
| **2** | `DAT3 / CD` | `SD_CS` | Pico GPIO18 (Pin 24), R1 | SPI Chip Select (CS) with 10kΩ pull-up R1 to +3.3V |
| **3** | `CMD` | `SD_MOSI` | Pico GPIO19 (Pin 25) | SPI Data Input (DI / MOSI) |
| **4** | `VDD` | `+3.3V` | +3.3V Rail | +3.3V Power Supply (Decoupled by C5 100nF MLCC) |
| **5** | `CLK` | `SD_SCK` | Pico GPIO20 (Pin 26) | SPI Serial Clock (SCK) |
| **6** | `VSS` | `GND` | Ground Plane | Power Ground |
| **7** | `DAT0` | `SD_MISO` | Pico GPIO21 (Pin 27) | SPI Data Output (DO / MISO) |
| **8** | `DAT1` | `NC` | Unconnected | Unused in SPI Mode |
| **9** | `Card Detect`| `SD_Card_Detect`| Pico GPIO22 (Pin 29) | Card Detect Switch (Shorts to GND when card is inserted) |
| **10** | `Casing` | `GND` | Ground Plane | Metal Shield / Housing Ground |

---

### 3.4 Peripherals: OLED (U2), Audio Amp Module (U3), Buttons (SW1..SW3), Jumper (JP1)

| Designator | Pin / Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`U2` Pin 1** | `VCC` | `+3.3V` | +3.3V Rail | Display Power (Decoupled by C6 10µF + C7 100nF) |
| **`U2` Pin 2** | `GND` | `GND` | Ground Plane | Display Power Ground |
| **`U2` Pin 3** | `SCL` | `OLED_SCL` | Pico GPIO16 (Pin 21) | I2C Serial Clock Line |
| **`U2` Pin 4** | `SDA` | `OLED_SDA` | Pico GPIO17 (Pin 22) | I2C Serial Data Line |
| **`U3` Pin 1** | `GND` | `GND` | Ground Plane | PAM8302 Power Ground |
| **`U3` Pin 2** | `VIN` | `+5V` | +5V Rail | PAM8302 Power Input (Decoupled by C8 100nF) |
| **`U3` Pin 3** | `SD` | `SD_Amp` | Pico GPIO9 (Pin 12) | Hardware Mute / Shutdown (HIGH = Enable, LOW = Mute) |
| **`U3` Pin 4** | `A-` | `GND` | Ground Plane | Differential Audio Negative Reference Input |
| **`U3` Pin 5** | `A+` | `NET-(U3-A+)` | C3 Pin 2 (Negative lead) | AC-Coupled Audio Input from Mixer Summing Junction |
| **`U3` Pin 6** | `OUT+` | `Net-(U3-OUT+-Pad6)` | Breakout Output Terminal | Positive Differential Class-D Speaker Output |
| **`U3` Pin 7** | `OUT-` | `Net-(U3-OUT--Pad7)` | Breakout Output Terminal | Negative Differential Class-D Speaker Output |
| **`SW1` Pin 1** | `1` | `SW_PLAY` | Pico GPIO14 (Pin 19) | Play / Pause Navigation Button (Internal Pull-Up) |
| **`SW1` Pin 2** | `2` | `GND` | Ground Plane | Ground Return |
| **`SW2` Pin 1** | `1` | `SW_NEXT` | Pico GPIO13 (Pin 17) | Next Track Navigation Button (Internal Pull-Up) |
| **`SW2` Pin 2** | `2` | `GND` | Ground Plane | Ground Return |
| **`SW3` Pin 1** | `1` | `SW_PREV` | Pico GPIO15 (Pin 20) | Previous Track Navigation Button (Internal Pull-Up) |
| **`SW3` Pin 2** | `2` | `GND` | Ground Plane | Ground Return |
| **`JP1` Pin 1** | `A` | `NET-(JP1-A)` | AY-3-8910 (U1) Pin 26 | YM2149 /SEL Clock Divider Select Pin |
| **`JP1` Pin 2** | `B` | `GND` | Ground Plane | Ground Return (Bridge pads to enable ÷2 divider) |

---

### 3.5 Passive Audio Mixer & Filtering Topology

The analog audio outputs from the PSG are combined using a low-noise passive summing junction:

1. **Summing Resistors:**
   - Channel A (U1 Pin 4): `R4` (1kΩ, 0805)
   - Channel B (U1 Pin 3): `R5` (1kΩ, 0805)
   - Channel C (U1 Pin 38): `R7` (1kΩ, 0805)
2. **Summing Node (`Net-(C3-Pad1)`):**
   - **Mixer Load Resistor:** `R8` (1kΩ, 0805) to `GND`. Establishes the passive attenuator baseline impedance and DC ground reference.
   - **Low-Pass Filter Capacitor:** `C4` (100nF, 0805 MLCC) to `GND`. Forms a first-order low-pass filter with the equivalent summing impedance ($R_{eq} \approx 250\,\Omega$, cutoff frequency $f_c \approx 6.3\,\text{kHz}$) to eliminate high-frequency digital clock harmonics and PSG switching noise.
   - **AC Output Coupling Capacitor:** `C3` (1µF, 3x5.4mm Electrolytic). Blocks DC bias from the PSG output pins and passes the audio signal into the PAM8302 `A+` differential audio input (`U3` Pin 5).

---

## 4. Bill of Materials & LCSC Part Mapping (SSOT)

| Designator | Value / Part | Package / Footprint | Quantity | LCSC Part # | Description |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **`A1`** | `RaspberryPi_Pico 2` | `TMCu Module:RPi_Pico_SMD_TH` | 1 | — | Raspberry Pi Pico 2 (RP2350) Microcontroller Module |
| **`C1`** | `10uF` | `Capacitor_SMD:CP_Elec_4x5.4` | 1 | `C3343` | Aluminum Electrolytic Bulk Power Decoupling Cap (U1 +5V) |
| **`C2`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1711` | MLCC 0805 High-Frequency Decoupling Cap (U1 +5V) |
| **`C3`** | `1uF` | `Capacitor_SMD:CP_Elec_3x5.4` | 1 | `C3348` | Aluminum Electrolytic Audio Input AC Coupling Cap |
| **`C4`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1710` | MLCC 0805 Audio Mixer Low-Pass Filter Cap |
| **`C5`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1710` | MLCC 0805 MicroSD +3.3V Decoupling Cap |
| **`C6`** | `10uF` | `Capacitor_SMD:CP_Elec_4x5.4` | 1 | `C3343` | Aluminum Electrolytic Bulk Power Decoupling Cap (U2 +3.3V) |
| **`C7`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1711` | MLCC 0805 High-Frequency Decoupling Cap (U2 +3.3V) |
| **`C8`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1710` | MLCC 0805 PAM8302 +5V Power Decoupling Cap |
| **`J2`** | `MicroSD` | `Terence:MicroSD` | 1 | `C479742` | TF-1115 MicroSD Card Socket (SPI Mode with Card Detect) |
| **`JP1`** | `Clk Div` | `Jumper:SolderJumper-2_P1.3mm_Open` | 1 | — | 2-Pole Solder Jumper (YM2149 /SEL Clock Divider Mode) |
| **`R1`** | `10K` | `Resistor_SMD:R_0805` | 1 | `C17414` | MicroSD SPI `SD_CS` Pull-Up Resistor (+3.3V) |
| **`R2`** | `33` | `Resistor_SMD:R_0805` | 1 | `C17634` | PSG Master Clock (GPIO12) Series Damping Resistor |
| **`R3`** | `10K` | `Resistor_SMD:R_0805` | 1 | `C17414` | AY-3-8910 `~RESET` (GPIO11) Boot Pull-Down Resistor (GND) |
| **`R4`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Channel A Audio Mixer Summer Resistor |
| **`R5`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Channel B Audio Mixer Summer Resistor |
| **`R7`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Channel C Audio Mixer Summer Resistor |
| **`R8`** | `1K` | `Resistor_SMD:R_0805` | 1 | `C17513` | Mixer Summing Junction Load Resistor to GND |
| **`SW1`**| `SW_PUSH_6mm` | `Button_Switch_THT:SW_PUSH_6mm` | 1 | `C17513` | 6x6mm Pushbutton Tactile Switch (Play / Pause, GPIO14) |
| **`SW2`**| `SW_PUSH_6mm` | `Button_Switch_THT:SW_PUSH_6mm` | 1 | `C114217` | 6x6mm Pushbutton Tactile Switch (Next Track, GPIO13) |
| **`SW3`**| `SW_PUSH_6mm` | `Button_Switch_THT:SW_PUSH_6mm` | 1 | `C12740` | 6x6mm Pushbutton Tactile Switch (Previous Track, GPIO15) |
| **`U1`** | `AY-3-8910` | `Package_DIP:DIP-40_W15.24mm` | 1 | — | 3-Voice Programmable Sound Generator IC (DIP-40 Socket) |
| **`U2`** | `0.96"OLED_I2C` | `TModules:OLED 0.96 I2C` | 1 | — | 4-Pin 0.96" SSD1306 I2C OLED Display Module |
| **`U3`** | `PAM8302` | `TModules:PAM8302A_Module` | 1 | — | Adafruit PAM8302A 2.5W Mono Class-D Audio Amplifier Breakout |

---

## 5. Revision History

| Rev | Date | Notes |
| :--- | :--- | :--- |
| **0.4** | **2026-08-15** | **Manufactured Production Release (SSOT)**: Fully aligned documentation with production schematic and manufactured PCB (`Pico_Chiptune_Player_0.4.zip` / `netlist.ipc`). Updated complete Pico GPIO assignments: `CLOCK` on GPIO12 via 33Ω R2, `BDIR` on GPIO10, `~RESET` on GPIO11 with R3 10kΩ pull-down, `BC1` on GPIO8, `SD_Amp` on GPIO9, `SW_PLAY` on GPIO14, `SW_NEXT` on GPIO13, and `SW_PREV` on GPIO15. Documented PAM8302A breakout module header topology (`U3`), 8-capacitor decoupling arrangement (`C1..C8`), and verified all LCSC component part numbers. |
| 0.2 | 2026-08-12 | Intermediate documentation revision. |
| 0.1 | 2026-08-08 | Initial hardware documentation release. |
 
---
 
## 6. License & Non-Commercial Notice
 
Copyright (c) 2026 Terence Ang <terenceang@mac.com> / MotionFxDesign.
 
This hardware project and documentation are licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.
 
> **NOT FOR COMMERCIAL USE**  
> You are free to share, copy, modify, and build upon this hardware design for personal, hobby, educational, and non-commercial purposes with attribution. Commercial manufacture, sale, or distribution for profit is strictly prohibited without prior written permission from the author.

