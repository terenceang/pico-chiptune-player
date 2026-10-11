# Hardware Documentation - Pico Chiptune Player

**Title:** Pico Chiptune Player  
**Author:** Terence Ang <terenceang@mac.com> / MotionFxDesign  
**Company:** MotionFxDesign  
**Status:** Schematic SSOT — Rev 0.6 (dual-PSG stereo variant; supersedes Rev 0.5)  
**Version / Rev:** 0.6  
**Date:** 2026-10-11  
**License:** Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) — NOT FOR COMMERCIAL USE  


---

## 1. Overview

The **Pico Chiptune Player** is a dedicated hardware chiptune music player powered by a **Raspberry Pi Pico 2 (RP2350)** microcontroller driving **two** real **General Instrument AY-3-8910** (or **Yamaha YM2149F**) 3-voice Programmable Sound Generators (PSGs) — **6 voices total**, mixed as **stereo** (one PSG per channel).

### Key Features & Subsystems
- **Microcontroller:** Raspberry Pi Pico 2 (RP2350) microcontroller running the chiptune player firmware, file parser, and UI.
- **Sound Generators:** 2× AY-3-8910 / YM2149F 40-pin DIP PSG (`U1`, `U4`). The 8-bit parallel bus (`DA0..DA7`), master clock (`CLOCK`), active-low reset (`~RESET`) and `BC2` are shared; `BDIR`/`BC1` are driven **separately per PSG** (`GPIO10`/`GPIO8` → U1, `GPIO26`/`GPIO27` → U4) so each PSG can be independently selected.
- **Audio Summer & Filter (per channel):** 3-channel passive resistive summing networks (`R4`,`R5`,`R7` = 1kΩ + `R8` = 1kΩ load for U1; `R9`,`R10`,`R11` = 1kΩ + `R12` = 1kΩ load for U4) with high-cut low-pass filtering (`C4`, `C9` = 100nF) and AC output coupling (`C3`, `C10` = 1µF).
- **Audio Amplifiers (stereo):** Two onboard **Adafruit PAM8302A 2.5W Mono Class-D Audio Amplifier Modules** (`U3` left, `U5` right) with shared hardware GPIO shutdown/mute control (`SD_Amp`).
- **Speaker Output:** Direct differential `OUT+` / `OUT-` speaker terminals on each PAM8302 module breakout.
- **Display:** 0.96" 128x64 I2C OLED display module (`U2`) with bulk and high-frequency decoupling (`C6` = 10µF, `C7` = 100nF).
- **Storage:** Full 4-bit SDIO microSD card socket (`J2`), driven by the RP2350 PIO-based `pico_sd_card` driver (Pico SDK / pico-extras). Card presence is detected **in software** (no card-detect switch). A single external pull-up `R1` = 10kΩ sits on `DAT3`; the firmware enables the RP2350 internal pull-ups on `CMD`/`DAT0`–`DAT3`.
- **Navigation Controls:** 3 tactile pushbuttons (`SW1` Play/Pause, `SW2` Next Track, `SW3` Previous Track).
- **Clock Divider Select:** Solder jumper `JP1` to select YM2149 internal ÷2 clock division (shared by both PSGs).

---

## 2. Block Diagram

```
                             +-------------------+
                             |  Raspberry Pi     |
                             | Pico 2 (RP2350)   |
                             +--+---+---+---+----+
                                |   |   |   |
          +---------------------+   |   |   +-----------------------+
          | (SDIO: GPIO18-22,28)    |   | (I2C: GPIO16, 17)         | (GPIO13, 14, 15)
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
| **Pin 21** | **GPIO16** | `OLED_SDA` | OLED Display (U2) Pin 4 | I2C Serial Data (SDA, Internal / Display Pull-up) |
| **Pin 22** | **GPIO17** | `OLED_SCL` | OLED Display (U2) Pin 3 | I2C Serial Clock (SCL, Internal / Display Pull-up) |
| **Pin 23** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 24** | **GPIO18** | `SD_DAT0` | MicroSD (J2) Pin 7 | SDIO Data 0 (DAT0) |
| **Pin 25** | **GPIO19** | `SD_DAT1` | MicroSD (J2) Pin 8 | SDIO Data 1 (DAT1) |
| **Pin 26** | **GPIO20** | `SD_DAT2` | MicroSD (J2) Pin 1 | SDIO Data 2 (DAT2) |
| **Pin 27** | **GPIO21** | `SD_DAT3` | MicroSD (J2) Pin 2, R1 | SDIO Data 3 (DAT3), with 10kΩ pull-up R1 to +3.3V |
| **Pin 28** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 29** | **GPIO22** | `SD_CLK` | MicroSD (J2) Pin 5 | SDIO Clock (CLK) |
| **Pin 30** | **RUN** | `N/C` | — | Reset input (Unconnected) |
| **Pin 31** | **GPIO26_ADC0** | `BDIR2` | AY-3-8910 (U4) Pin 27 | PSG2 Bus Direction Control |
| **Pin 32** | **GPIO27_ADC1** | `BC1_2` | AY-3-8910 (U4) Pin 29 | PSG2 Bus Control 1 (Address Latch / Data Write) |
| **Pin 33** | **AGND** | `N/C` | — | Analog ground (Unconnected) |
| **Pin 34** | **GPIO28_ADC2** | `SD_CMD` | MicroSD (J2) Pin 3 | SDIO Command (CMD) |
| **Pin 35** | **ADC_VREF** | `N/C` | — | ADC reference (Unconnected) |
| **Pin 36** | **3V3** | `+3.3V` | MicroSD J2, OLED U2, R1, C5..C7 | 3.3V System Power Rail (Sourced from Pico onboard regulator) |
| **Pin 37** | **3V3_EN** | `N/C` | — | 3.3V regulator enable (Unconnected) |
| **Pin 38** | **GND** | `GND` | Ground Plane | System Ground |
| **Pin 39** | **VSYS** | `N/C` | — | System supply input (Unconnected) |
| **Pin 40** | **VBUS** | `+5V` | U1 Pin 40/25/28, U4 Pin 40/25/28, U3 Pin 2, U5 Pin 2, C1, C2, C8, C11 | 5V Main Power Rail (Sourced from Pico USB VBUS) |

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

#### Duplicate PSG — Yamaha YM2149F (U4)

`U4` is the second PSG and is electrically identical to `U1` **except** that its `BDIR` and `BC1` are driven from a second GPIO pair. All other connections (`DA0..DA7`, `CLOCK`, `~RESET`, `A8`→+5V, `A9`→GND, `BC2`→+5V, `/SEL`→`JP1`, power/decoupling) are shared with `U1`.

| Pin | Symbol | Net Name | Connected To | Notes / Function |
| :--- | :--- | :--- | :--- | :--- |
| **3** | `ANALOG_B` | `ANALOG_B2` | Resistor R10 (1kΩ) | Channel B Analog Audio Output (right) |
| **4** | `ANALOG_A` | `ANALOG_A2` | Resistor R9 (1kΩ) | Channel A Analog Audio Output (right) |
| **26** | `~{CS/Test_2}` | `/JP1_SEL` | Solder Jumper JP1 Pin 1 | YM2149 Clock Select (/SEL), shared with U1 |
| **27** | `BDIR` | `/BDIR2` | Pico GPIO26 (Pin 31) | Bus Direction Control Line (PSG2) |
| **29** | `BC1` | `/BC1_2` | Pico GPIO27 (Pin 32) | Bus Control 1 Line (PSG2) |
| **30..37**| `DA7..DA0` | `DA7..DA0` | Pico GPIO7..GPIO0 | Shared 8-Bit Data Bus (with U1) |
| **38** | `ANALOG_C` | `ANALOG_C2` | Resistor R11 (1kΩ) | Channel C Analog Audio Output (right) |
| **40** | `VCC` | `+5V` | +5V Rail | Main +5V Supply (Decoupled by C11 10µF bulk + C12 100nF MLCC) |

> **Shared-bus note:** Only the currently selected PSG (the one whose `BDIR`/`BC1` pair is driven out of the inactive state) responds; the other PSG's bus stays High-Z. `U1` uses `GPIO10`/`GPIO8`; `U4` uses `GPIO26`/`GPIO27`.

---

### 3.3 MicroSD Card Socket (J2) Pinout

| Pin | Card Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `DAT2` | `SD_DAT2` | Pico GPIO20 (Pin 26) | SDIO Data 2 (DAT2) |
| **2** | `DAT3 / CD` | `SD_DAT3` | Pico GPIO21 (Pin 27), R1 | SDIO Data 3 (DAT3), with 10kΩ pull-up R1 to +3.3V |
| **3** | `CMD` | `SD_CMD` | Pico GPIO28 (Pin 34) | SDIO Command (CMD) |
| **4** | `VDD` | `+3.3V` | +3.3V Rail | +3.3V Power Supply (Decoupled by C5 100nF MLCC) |
| **5** | `CLK` | `SD_CLK` | Pico GPIO22 (Pin 29) | SDIO Clock (CLK) |
| **6** | `VSS` | `GND` | Ground Plane | Power Ground |
| **7** | `DAT0` | `SD_DAT0` | Pico GPIO18 (Pin 24) | SDIO Data 0 (DAT0) |
| **8** | `DAT1` | `SD_DAT1` | Pico GPIO19 (Pin 25) | SDIO Data 1 (DAT1) |
| **9** | `Card Detect`| `NC` | Unconnected | Card Detect switch unused (card presence detected in software) |
| **10** | `Casing` | `GND` | Ground Plane | Metal Shield / Housing Ground |

---

### 3.4 Peripherals: OLED (U2), Audio Amp Module (U3), Buttons (SW1..SW3), Jumper (JP1)

| Designator | Pin / Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`U2` Pin 1** | `VCC` | `+3.3V` | +3.3V Rail | Display Power (Decoupled by C6 10µF + C7 100nF) |
| **`U2` Pin 2** | `GND` | `GND` | Ground Plane | Display Power Ground |
| **`U2` Pin 3** | `SCL` | `OLED_SCL` | Pico GPIO17 (Pin 22) | I2C Serial Clock Line |
| **`U2` Pin 4** | `SDA` | `OLED_SDA` | Pico GPIO16 (Pin 21) | I2C Serial Data Line |
| **`U3` Pin 1** | `GND` | `GND` | Ground Plane | PAM8302 Power Ground |
| **`U3` Pin 2** | `VIN` | `+5V` | +5V Rail | PAM8302 Power Input (Decoupled by C8 100nF) |
| **`U3` Pin 3** | `SD` | `SD_Amp` | Pico GPIO9 (Pin 12) | Hardware Mute / Shutdown (HIGH = Enable, LOW = Mute) |
| **`U3` Pin 4** | `A-` | `GND` | Ground Plane | Differential Audio Negative Reference Input |
| **`U3` Pin 5** | `A+` | `NET-(U3-A+)` | C3 Pin 2 (Negative lead) | AC-Coupled Audio Input from Mixer Summing Junction |
| **`U3` Pin 6** | `OUT+` | `Net-(U3-OUT+-Pad6)` | Breakout Output Terminal | Positive Differential Class-D Speaker Output |
| **`U3` Pin 7** | `OUT-` | `Net-(U3-OUT--Pad7)` | Breakout Output Terminal | Negative Differential Class-D Speaker Output |
| **`U5` Pin 1** | `GND` | `GND` | Ground Plane | PAM8302 (right) Power Ground |
| **`U5` Pin 2** | `VIN` | `+5V` | +5V Rail | PAM8302 (right) Power Input (Decoupled by C13 100nF) |
| **`U5` Pin 3** | `SD` | `SD_Amp` | Pico GPIO9 (Pin 12) | Hardware Mute / Shutdown (shared with U3) |
| **`U5` Pin 4** | `A-` | `GND` | Ground Plane | Differential Audio Negative Reference Input |
| **`U5` Pin 5** | `A+` | `AMP2_IN` | C10 Pin 1 (Positive lead) | AC-Coupled Audio Input from right-channel Mixer |
| **`U5` Pin 6** | `OUT+` | `Net-(U5-OUT+-Pad6)` | Breakout Output Terminal | Positive Differential Class-D Speaker Output (right) |
| **`U5` Pin 7** | `OUT-` | `Net-(U5-OUT--Pad7)` | Breakout Output Terminal | Negative Differential Class-D Speaker Output (right) |
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

Each PSG feeds its own identical passive summing stage, giving a fully independent **left** (U1) and **right** (U4) channel:

**Left channel (U1 → U3):**

1. **Summing Resistors:**
   - Channel A (U1 Pin 4): `R4` (1kΩ, 0805)
   - Channel B (U1 Pin 3): `R5` (1kΩ, 0805)
   - Channel C (U1 Pin 38): `R7` (1kΩ, 0805)
2. **Summing Node (`Net-(C3-Pad1)`):**
   - **Mixer Load Resistor:** `R8` (1kΩ, 0805) to `GND`. Establishes the passive attenuator baseline impedance and DC ground reference.
   - **Low-Pass Filter Capacitor:** `C4` (100nF, 0805 MLCC) to `GND`. Forms a first-order low-pass filter with the equivalent summing impedance ($R_{eq} \approx 250\,\Omega$, cutoff frequency $f_c \approx 6.3\,\text{kHz}$) to eliminate high-frequency digital clock harmonics and PSG switching noise.
   - **AC Output Coupling Capacitor:** `C3` (1µF, 3x5.4mm Electrolytic). Blocks DC bias from the PSG output pins and passes the audio signal into the PAM8302 `A+` differential audio input (`U3` Pin 5).

**Right channel (U4 → U5):** identical topology using `R9`, `R10`, `R11` (1kΩ summing), `R12` (1kΩ load), `C9` (100nF LPF) and `C10` (1µF coupling) into the second PAM8302 `A+` input (`U5` Pin 5).

---

### 3.6 Rev 0.4 Native-Function Errata & Rev 0.5 Correction (SSOT)

The manufactured Rev 0.4 board used the pin assignment audited below, which predates the Rev 0.5 correction now shown in §3.1. A review against the **RP2350 Bank-0 GPIO functions** table shows that the pins chosen for the **OLED I²C** and the **microSD SPI** are not the RP2350's *native* peripheral (function-mux) pins. Because GPIO alternate functions are fixed per pin on RP2350, the on-chip **I2C0** and **SPI0** controllers cannot be routed to these signals as wired in Rev 0.4 — only **PIO** or software (bit-banged) I²C/SPI can drive them on an existing Rev 0.4 board.

#### Rev 0.4 pin audit

| Pico Pin | GPIO | Rev 0.4 Net | Native alt function of that GPIO | Compliant? |
| :--- | :--- | :--- | :--- | :--- |
| 21 | `GPIO16` | `OLED_SCL` | **I2C0 SDA** (SPI0 RX) | ❌ SDA/SCL swapped |
| 22 | `GPIO17` | `OLED_SDA` | **I2C0 SCL** (SPI0 CSn) | ❌ SDA/SCL swapped |
| 24 | `GPIO18` | `SD_CS` | **SPI0 SCK** | ❌ not a CS pin |
| 25 | `GPIO19` | `SD_MOSI` | **SPI0 TX (MOSI)** | ✅ correct |
| 26 | `GPIO20` | `SD_SCK` | **SPI0 RX (MISO)** | ❌ not SCK |
| 27 | `GPIO21` | `SD_MISO` | **SPI0 CSn** | ❌ not MISO |
| 29 | `GPIO22` | `SD_Card_Detect` | SPI0 SCK (used as GPIO) | ✅ (plain GPIO) |

#### Rev 0.5 correction (native-compliant, now the current SSOT)

The fix keeps every signal within the **same GPIO group (GP16–GP21)** and only re-assigns five Pico header connections — **no additional GPIOs are used**.

| Function | Rev 0.4 GPIO | **Rev 0.5 GPIO** | Native peripheral (RP2350) |
| :--- | :--- | :--- | :--- |
| `OLED_SDA` | `GPIO17` | **`GPIO16`** | I2C0 SDA |
| `OLED_SCL` | `GPIO16` | **`GPIO17`** | I2C0 SCL |
| `SD_SCK` | `GPIO20` | **`GPIO18`** | SPI0 SCK |
| `SD_MOSI` | `GPIO19` | `GPIO19` | SPI0 TX (MOSI) — unchanged |
| `SD_MISO` | `GPIO21` | **`GPIO20`** | SPI0 RX (MISO) |
| `SD_CS` | `GPIO18` | **`GPIO21`** | SPI0 CSn / GPIO |
| `SD_Card_Detect` | `GPIO22` | `GPIO22` | GPIO — unchanged |

#### Header-only change list (schematic nets, Rev 0.4 → Rev 0.5)

| Pico Pin | GPIO | Rev 0.4 Net | Rev 0.5 Net |
| :--- | :--- | :--- | :--- |
| 21 | `GPIO16` | `OLED_SCL` | `OLED_SDA` |
| 22 | `GPIO17` | `OLED_SDA` | `OLED_SCL` |
| 24 | `GPIO18` | `SD_CS` | `SD_SCK` |
| 25 | `GPIO19` | `SD_MOSI` | `SD_MOSI` (unchanged) |
| 26 | `GPIO20` | `SD_SCK` | `SD_MISO` |
| 27 | `GPIO21` | `SD_MISO` | `SD_CS` |

All other assignments — AY-3-8910 data bus (`DA0..DA7` on `GPIO0..7`), `BC1` (`GPIO8`), `SD_Amp` (`GPIO9`), `BDIR` (`GPIO10`), `~RESET` (`GPIO11`), `CLOCK` (`GPIO12`, PWM/PIO), and the three buttons (`GPIO13..15`) — are plain GPIO/PWM and remain unchanged.

#### Firmware note for existing Rev 0.4 boards

Rev 0.4 hardware works only if firmware does **not** rely on the RP2350 hardware I2C0/SPI0 peripherals:
- **microSD:** use PIO-based SPI (e.g., `pico-sdk` `hardware_pio` / `pio_spi`) or a bit-banged driver.
- **OLED:** use PIO/software I²C or a bit-banged I²C driver.
- The AY-3-8910 parallel bus is already GPIO bit-banged, so it is unaffected.

This Rev 0.5 pinout (the current SSOT, now applied in §3.1) restores native hardware `I2C0` and `SPI0` support.

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
| **`C9`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1710` | MLCC 0805 Right-Channel Audio Mixer Low-Pass Filter Cap |
| **`C10`** | `1uF` | `Capacitor_SMD:CP_Elec_3x5.4` | 1 | `C3348` | Aluminum Electrolytic Right-Channel Audio AC Coupling Cap |
| **`C11`** | `10uF` | `Capacitor_SMD:CP_Elec_4x5.4` | 1 | `C3343` | Aluminum Electrolytic Bulk Power Decoupling Cap (U4 +5V) |
| **`C12`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1711` | MLCC 0805 High-Frequency Decoupling Cap (U4 +5V) |
| **`C13`** | `100nF` | `Capacitor_SMD:C_0805` | 1 | `C1710` | MLCC 0805 PAM8302 (U5) +5V Power Decoupling Cap |
| **`J2`** | `MicroSD` | `Terence:MicroSD` | 1 | `C479742` | TF-1115 MicroSD Card Socket (4-bit SDIO) |
| **`JP1`** | `Clk Div` | `Jumper:SolderJumper-2_P1.3mm_Open` | 1 | — | 2-Pole Solder Jumper (YM2149 /SEL Clock Divider Mode) |
| **`R1`** | `10K` | `Resistor_SMD:R_0805` | 1 | `C17414` | MicroSD SDIO `DAT3` Pull-Up Resistor (+3.3V) |
| **`R2`** | `33` | `Resistor_SMD:R_0805` | 1 | `C17634` | PSG Master Clock (GPIO12) Series Damping Resistor |
| **`R3`** | `10K` | `Resistor_SMD:R_0805` | 1 | `C17414` | AY-3-8910 `~RESET` (GPIO11) Boot Pull-Down Resistor (GND) |
| **`R4`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Channel A Audio Mixer Summer Resistor |
| **`R5`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Channel B Audio Mixer Summer Resistor |
| **`R7`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Channel C Audio Mixer Summer Resistor |
| **`R8`** | `1K` | `Resistor_SMD:R_0805` | 1 | `C17513` | Left-Channel Mixer Summing Junction Load Resistor to GND |
| **`R9`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Right-Channel A Audio Mixer Summer Resistor |
| **`R10`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Right-Channel B Audio Mixer Summer Resistor |
| **`R11`** | `1k` | `Resistor_SMD:R_0805` | 1 | `C17513` | Right-Channel C Audio Mixer Summer Resistor |
| **`R12`** | `1K` | `Resistor_SMD:R_0805` | 1 | `C17513` | Right-Channel Mixer Summing Junction Load Resistor to GND |
| **`SW1`**| `SW_PUSH_6mm` | `Button_Switch_THT:SW_PUSH_6mm` | 1 | `C17513` | 6x6mm Pushbutton Tactile Switch (Play / Pause, GPIO14) |
| **`SW2`**| `SW_PUSH_6mm` | `Button_Switch_THT:SW_PUSH_6mm` | 1 | `C114217` | 6x6mm Pushbutton Tactile Switch (Next Track, GPIO13) |
| **`SW3`**| `SW_PUSH_6mm` | `Button_Switch_THT:SW_PUSH_6mm` | 1 | `C12740` | 6x6mm Pushbutton Tactile Switch (Previous Track, GPIO15) |
| **`U1`** | `AY-3-8910` | `Package_DIP:DIP-40_W15.24mm` | 1 | — | 3-Voice Programmable Sound Generator IC (DIP-40 Socket) |
| **`U2`** | `0.96"OLED_I2C` | `TModules:OLED 0.96 I2C` | 1 | — | 4-Pin 0.96" SSD1306 I2C OLED Display Module |
| **`U3`** | `PAM8302` | `TModules:PAM8302A_Module` | 1 | — | Adafruit PAM8302A 2.5W Mono Class-D Audio Amplifier Breakout (left) |
| **`U4`** | `AY-3-8910` | `Package_DIP:DIP-40_W15.24mm` | 1 | — | Second 3-Voice Programmable Sound Generator IC (DIP-40 Socket, right) |
| **`U5`** | `PAM8302` | `TModules:PAM8302A_Module` | 1 | — | Adafruit PAM8302A 2.5W Mono Class-D Audio Amplifier Breakout (right) |

---

## 5. Revision History

| Rev | Date | Notes |
| :--- | :--- | :--- |
| **0.6** | **2026-10-11** | **Dual-PSG stereo variant**: Added a second AY-3-8910/YM2149F (`U4`) sharing the `DA0..DA7`/`CLOCK`/`~RESET`/`BC2` bus, driven by a dedicated `BDIR`/`BC1` pair on `GPIO26`/`GPIO27` (6 voices total). Added an independent right-channel passive mixer (`R9`-`R12`, `C9`, `C10`) and a second PAM8302A amplifier module (`U5`) for true stereo, plus decoupling (`C11`, `C12`, `C13`). Board widened to 139.4 × 62.6 mm. ERC clean; routed PCB with 0 unconnected items. **microSD changed from SPI to 4-bit SDIO** (PIO-based `pico_sd_card`): `SD_DAT0..DAT3` → `GPIO18`-`GPIO21`, `SD_CLK` → `GPIO22`, `SD_CMD` → `GPIO28`; dedicated card-detect switch dropped (presence detected in software); `R1` 10kΩ now pulls up `DAT3`. |
| **0.5** | **2026-10-11** | **Native-function pin correction (new board; schematic amended, not yet manufactured)**: Re-mapped OLED I²C and microSD SPI to RP2350 native pins — `OLED_SDA`→`GPIO16`, `OLED_SCL`→`GPIO17` (SDA/SCL swap); `SD_SCK`→`GPIO18`, `SD_MISO`→`GPIO20`, `SD_CS`→`GPIO21`; `SD_MOSI` remains `GPIO19`. No additional GPIOs used. Supersedes Rev 0.4 (§3.6). |
| **0.4** | **2026-08-15** | **Manufactured Production Release (superseded by Rev 0.5, see §3.6)**: Fully aligned documentation with production schematic and manufactured PCB (`Pico_Chiptune_Player_0.4.zip` / `netlist.ipc`). Updated complete Pico GPIO assignments: `CLOCK` on GPIO12 via 33Ω R2, `BDIR` on GPIO10, `~RESET` on GPIO11 with R3 10kΩ pull-down, `BC1` on GPIO8, `SD_Amp` on GPIO9, `SW_PLAY` on GPIO14, `SW_NEXT` on GPIO13, and `SW_PREV` on GPIO15. Documented PAM8302A breakout module header topology (`U3`), 8-capacitor decoupling arrangement (`C1..C8`), and verified all LCSC component part numbers. |
| 0.2 | 2026-08-12 | Intermediate documentation revision. |
| 0.1 | 2026-08-08 | Initial hardware documentation release. |
 
---
 
## 6. License & Non-Commercial Notice
 
Copyright (c) 2026 Terence Ang <terenceang@mac.com> / MotionFxDesign.
 
This hardware project and documentation are licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.
 
> **NOT FOR COMMERCIAL USE**  
> You are free to share, copy, modify, and build upon this hardware design for personal, hobby, educational, and non-commercial purposes with attribution. Commercial manufacture, sale, or distribution for profit is strictly prohibited without prior written permission from the author.

