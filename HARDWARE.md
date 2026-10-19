# Hardware Documentation - Pico Chiptune Player

**Title:** Pico Chiptune Player  
**Author:** Terence Ang <terenceang@mac.com> / MotionFxDesign  
**Company:** MotionFxDesign  
**Status:** Schematic & PCB SSOT — Rev 0.8 (JLC Assembly Prep: corrected GPIO map, 0402/0805 JLC basic passives, LCSC part numbers; supersedes Rev 0.7)  
**Version / Rev:** 0.8  
**Date:** 2026-10-20  
**License:** Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) — NOT FOR COMMERCIAL USE  

---

## 1. Overview

The **Pico Chiptune Player** is a dedicated hardware chiptune music player powered by an on-board **Raspberry Pi RP2354A** microcontroller (`U5`, QFN-60 with 2 MB internal stacked QSPI NOR flash) driving **two** real **Yamaha YM2149F** 3-voice Programmable Sound Generators (PSGs), register-compatible with the General Instrument AY-3-8910 — **6 voices total**, routed through a **software-controlled 4-mode analog stereo/mono mixing matrix** (`2× 74HCT4053D`, `U8` & `U9`).

### Key Features & Subsystems
- **Microcontroller (`U5`):** Bare **Raspberry Pi RP2354A** (QFN-60, 7×7 mm, 0.4 mm pitch, 2 MB internal stacked QSPI NOR flash) exposing all **30 Bank-0 GPIOs (`GPIO0..GPIO29`)**, clocked by a 12 MHz SMD-3225 crystal (`Y1` + `C20`/`C21` = 15 pF + `R16` = 1 kΩ), with internal 1.1 V core buck regulator (`L1` = 3.3 µH + `C23` = 4.7 µF + `C24`/`C25` = 100 nF) and onboard `BOOTSEL` (`SW4` + `R20` = 1 kΩ) and `RESET` (`SW5` + `R19` = 10 kΩ) SMD tactile switches (`TS-1187A`, LCSC `C318884`).
- **USB-C 5V/3A Power & 3-Stage Hardware Power Sequencing (`J1`, `D1`, `U6`, `Q1`, `Q2`):**
  - 16-pin USB 2.0 Type-C receptacle (`J1`) with two independent 5.1 kΩ 1% pull-downs (`R21`, `R22`) on `CC1`/`CC2` advertising a fixed 5V/3A sink, `USBLC6-2SC6` (`D1`) ESD protection, and 27 Ω (`R17`, `R18`) USB D+/D− series termination.
  - **Stage 1 (`VBUS` → `+3.3V` & `+1V1`):** `AMS1117-3.3` LDO (`U6`, SOT-223) immediately powers `+3.3V` (`RP2354A` `IOVDD`, microSD `J2`, OLED `U2`) and `RP2354A`'s internal `+1V1` (`DVDD`) buck regulator (`L1`) on USB plug-in.
  - **Stage 2 (`+3.3V` RC-delayed → `+5V`):** RC delay network (`R23` = 47 kΩ, `C31` = 100 nF, $\tau \approx 4.7\,\text{ms}$) delays turn-on of N-MOSFET `Q2` (`2N7002`), which pulls down `PFET_GATE` (`R28` = 100 kΩ pull-up to `VBUS`, `C32` = 10 nF soft-start cap) to turn on high-side P-MOSFET `Q1` (`AO3401A`) **only after `+3.3V` (`IOVDD`) is stable**, preventing unpowered latch-up or back-bias on the 5 V `YM2149F` PSGs (`U1`, `U4`), `74HCT4053D` analog switches (`U8`, `U9`), and `PAM8403` amplifier (`U3`).
  - **Stage 3 (`SD_Amp` on `GPIO21` → `U3` `PAM8403`):** `R13` (10 kΩ pull-down) keeps `PAM8403` `~MUTE` and `~SHDN` LOW during Stage 1 & Stage 2 power-up until firmware drives `GPIO21` HIGH after AC coupling capacitors `C3`/`C10` settle.
- **Sound Generators (`U1`, `U4`):** 2× YM2149F 40-pin DIP PSG (socketed; a matched pair of AY-3-8910s is an alternative with `JP1` open, see §3.2). The 8-bit parallel bus (`DA0..DA7`), master clock (`CLOCK`), active-low reset (`~RESET`) and `BC2` are shared; `BDIR`/`BC1` are driven **separately per PSG** (`GPIO10`/`GPIO8` → `U1`, `GPIO13`/`GPIO12` → `U4`).
- **4-Mode Hardware Stereo/Mono Analog Mixing Matrix (`U8`, `U9`):**
  - Dual **74HCT4053D** triple 2:1 analog multiplexers (SOIC-16, powered from `+5V` with 3.3 V TTL-compatible inputs `V_IH = 2.0V`) controlled by `GPIO14` (`MUX_PAN`), `GPIO15` (`MUX_B_CEN`), `GPIO16` (`MUX_C_CEN`), and `GPIO17` (`MUX_MONO`) with 10 kΩ boot pull-downs (`R24..R27`) and 470 Ω (`R14`, `R15`) `-3 dB` center-channel compensation resistors.
  - Supports **Mode 0: Dual-PSG Split Stereo** (`U1`=Left, `U4`=Right), **Mode 1: ABC Stereo** (`A`=Left, `B`=Center, `C`=Right), **Mode 2: ACB Stereo** (`A`=Left, `C`=Center, `B`=Right), and **Mode 3: True Mono** (`U1+U4` summed equally to Left & Right).
- **Audio Summer, Filter & Amplifier (`U3`, `J3`, `J4`):**
  - Left (`MIX1`: `R8` = 1 kΩ load, `C4` = 100 nF LPF, `C3` = 1 µF AC coupling) and Right (`MIX2`: `R12` = 1 kΩ load, `C9` = 100 nF LPF, `C10` = 1 µF AC coupling) summing junctions feeding an on-board **PAM8403 3W+3W filterless Class-D stereo amplifier** (`U3`, SOIC-16) and JST-PH 2-pin speaker connectors (`J3` Left, `J4` Right).
- **Display (`U2`) & Storage (`J2`):** 0.96" 128×64 I2C OLED display (`U2`) and 4-bit SDIO microSD card socket (`J2`, Hirose `DM3AT-SF-PEJM5`, LCSC `C114218`).
- **Controls (`SW1..SW5`, `JP1`):** 3 track navigation buttons (`SW1` Play/Pause, `SW2` Next, `SW3` Prev), 2 MCU system buttons (`SW4` `BOOTSEL`, `SW5` `RESET`) — all five `TS-1187A` SMD tactiles (LCSC `C318884`), and solder jumper `JP1` (`/SEL` ÷2 clock divider).

---

## 2. Block Diagram

```
  +---------------------+     Stage 1 (Immediate)     +--------------------------+
  | USB-C 5V/3A Sink    | --------------------------> | U6: AMS1117-3.3 LDO      | ---> +3.3V Rail (IOVDD, J2, U2)
  | J1 + R21/R22 (5.1k) |                             +------------+-------------+
  | D1: USBLC6-2SC6 ESD |                                          | RC Delay (R23 47k + C31 100nF)
  +----------+----------+     Stage 2 (Delayed +5V)                v
             | VBUS           +------------------------------------+--------------------+
             +--------------> | Q2 (2N7002) + Q1 (AO3401A P-MOSFET) + C32 Soft-Start    | ---> +5V Rail (U1, U4, U8, U9, U3)
                              +---------------------------------------------------------+

                                        +-----------------------------------+
                                        | U5: Raspberry Pi RP2354A (QFN-60) |
                                        | 2MB Internal Flash | Y1 12MHz XTAL|
                                        | L1 3.3uH (+1V1)    | SW4/SW5 Btns |
                                        +----+--------+--------+--------+---+
                                             |        |        |        |
             +-------------------------------+        |        |        +----------------------------+
             | (SDIO: GPIO22..27)                     |        | (I2C0: GPIO28,29; Btns: GPIO18-20)  | (MUX: GPIO14-17)   
             v                                        |        v                                     |
  +--------------------+                              |  +------------+   +--------------+           |
  | microSD Slot (J2)  |                              |  | OLED (U2)  |   | SW1..SW3 Nav |           |
  +--------------------+                              |  +------------+   +--------------+           |
                                                      |                                              |
             +----------------------------------------+                                              |
             | DA0..DA7 (GPIO0..7), BC1/BDIR (GPIO8/10), BC1_2/BDIR2 (GPIO12/13)                     |
             | ~RESET (GPIO11 + R3 10k), CLOCK (GPIO13 + R2 33R), SD_Amp (GPIO21 + R13 10k)           |
             v                                                                                       v
  +------------------------+     R4,R5,R7 (U1) / R9,R10,R11 (U4)     +---------------------------------------+
  | 2x YM2149F (U1, U4)    | --------------------------------------> | U8, U9: 2x 74HCT4053D Stereo Matrix   |
  | + JP1 (/SEL Divider)   |     6 Voices: A1,B1,C1 & A2,B2,C2       | Modes: Dual-PSG | ABC | ACB | Mono    |
  +------------------------+                                         | + R14, R15 (470R -3dB Center Atten.)  |
                                                                     +--------+---------------------+--------+
                                                                              | MIX1                | MIX2
                                                                              | (R8 1k, C4 100nF)   | (R12 1k, C9 100nF)
                                                                              | C3 1uF -> AMP_INL   | C10 1uF -> AMP_INR
                                                                              v                     v
                                                                     +---------------------------------------+
                                                                     | U3: PAM8403 Stereo Class-D Amp        |
                                                                     | Stage 3 Enable: SD_Amp (GPIO21)        |
                                                                     +--------+---------------------+--------+
                                                                              |                     |
                                                                              v                     v
                                                                       J3 Speaker L          J4 Speaker R
```

---

## 3. Complete Pinout & Wiring Tables (SSOT)

### 3.1 Raspberry Pi RP2354A Microcontroller (`U5`, QFN-60) Pin Connections

`U5` is the `RP2354A` (`QFN-60-1EP_7x7mm_P0.4mm_EP3.4x3.4mm`) with 2 MB internal stacked QSPI NOR flash. All 30 Bank-0 GPIOs (`GPIO0..GPIO29`) are used.

| QFN Pin | Symbol | Net Name | Connected To | Signal Description |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `IOVDD` | `+3.3V` | +3.3V Plane, `C26` (100nF) | Digital I/O Power Supply (+3.3V) |
| **2** | `GPIO0` | `/DA0` | `U1` Pin 37, `U4` Pin 37 | Shared PSG 8-Bit Data Bus Bit 0 |
| **3** | `GPIO1` | `/DA1` | `U1` Pin 36, `U4` Pin 36 | Shared PSG 8-Bit Data Bus Bit 1 |
| **4** | `GPIO2` | `/DA2` | `U1` Pin 35, `U4` Pin 35 | Shared PSG 8-Bit Data Bus Bit 2 |
| **5** | `GPIO3` | `/DA3` | `U1` Pin 34, `U4` Pin 34 | Shared PSG 8-Bit Data Bus Bit 3 |
| **6** | `DVDD` | `+1V1` | Internal +1.1V Core Rail, `C25` (100nF) | Digital Core Power Supply (+1.1V) |
| **7** | `GPIO4` | `/DA4` | `U1` Pin 33, `U4` Pin 33 | Shared PSG 8-Bit Data Bus Bit 4 |
| **8** | `GPIO5` | `/DA5` | `U1` Pin 32, `U4` Pin 32 | Shared PSG 8-Bit Data Bus Bit 5 |
| **9** | `GPIO6` | `/DA6` | `U1` Pin 31, `U4` Pin 31 | Shared PSG 8-Bit Data Bus Bit 6 |
| **10** | `GPIO7` | `/DA7` | `U1` Pin 30, `U4` Pin 30 | Shared PSG 8-Bit Data Bus Bit 7 |
| **11** | `IOVDD` | `+3.3V` | +3.3V Plane, `C27` (100nF) | Digital I/O Power Supply (+3.3V) |
| **12** | `GPIO8` | `/BC1` | `U1` Pin 29 | PSG1 (`U1`) Bus Control 1 (Address Latch / Data Write) |
| **13** | `GPIO9` | `/MCU_CLK` | `R2` Pin 2 (33Ω → `/CLOCK`) | Shared PSG Master Clock Output (series damped via 33Ω `R2` to `U1`/`U4` Pin 22) |
| **14** | `GPIO10` | `/BDIR` | `U1` Pin 27 | PSG1 (`U1`) Bus Direction Control |
| **15** | `GPIO11` | `/~{RESET}` | `U1` Pin 23, `U4` Pin 23, `R3` (10kΩ) | Shared PSG Active-Low Reset (10kΩ `R3` pull-down holds PSGs in reset at boot) |
| **16** | `GPIO12` | `/BC1_2` | `U4` Pin 29 | PSG2 (`U4`) Bus Control 1 (Address Latch / Data Write) |
| **17** | `GPIO13` | `/BDIR2` | `U4` Pin 27 | PSG2 (`U4`) Bus Direction Control |
| **18** | `GPIO14` | `/MUX_PAN` | `U9` Pins 9, 10, 11, `R24` (10kΩ to `GND`) | Stereo Matrix Right/Left Pan Control (`0` = Dual-PSG/ACB, `1` = ABC) |
| **19** | `GPIO15` | `/MUX_B_CEN` | `U8` Pin 11, `R25` (10kΩ to `GND`) | Stereo Matrix Voice B Center Enable (`1` = Route `B1+B2` to Center `-3 dB`) |
| **20** | `IOVDD` | `+3.3V` | +3.3V Plane, `C28` (100nF) | Digital I/O Power Supply (+3.3V) |
| **21** | `XIN` | `/XIN` | `Y1` Pin 1 (12MHz), `C20` (15pF) | 12 MHz Crystal Oscillator Input |
| **22** | `XOUT` | `/XOUT_MCU` | `R16` Pin 1 (1kΩ → `/XOUT`, `Y1` Pin 3, `C21` 15pF) | 12 MHz Crystal Oscillator Output (1kΩ series damping `R16`) |
| **23** | `DVDD` | `+1V1` | Internal +1.1V Core Rail | Digital Core Power Supply (+1.1V) |
| **24** | `SWCLK` | `NC` | Unconnected | Serial Wire Debug Clock (Unused) |
| **25** | `SWDIO` | `NC` | Unconnected | Serial Wire Debug Data (Unused) |
| **26** | `RUN` | `/MCU_RUN` | `SW5` Pin 1 (`RESET`), `R19` (10kΩ to `+3.3V`) | Active-Low MCU Hardware Reset |
| **27** | `GPIO16` | `/MUX_C_CEN` | `U8` Pin 10, `R26` (10kΩ to `GND`) | Stereo Matrix Voice C Center Enable (`1` = Route `C1+C2` to Center `-3 dB`) |
| **28** | `GPIO17` | `/MUX_MONO` | `U8` Pin 9, `R27` (10kΩ to `GND`) | Stereo Matrix True Mono Bridge Enable (`1` = Tie `MIX1` & `MIX2` together) |
| **29** | `GPIO18` | `/SW_NEXT` | `SW2` Pin 1 | Next Track Button (Active Low, Internal Pull-up) |
| **30** | `IOVDD` | `+3.3V` | +3.3V Plane, `C29` (100nF) | Digital I/O Power Supply (+3.3V) |
| **31** | `GPIO19` | `/SW_PLAY` | `SW1` Pin 1 | Play / Pause Button (Active Low, Internal Pull-up) |
| **32** | `GPIO20` | `/SW_PREV` | `SW3` Pin 1 | Previous Track Button (Active Low, Internal Pull-up) |
| **33** | `GPIO21` | `/SD_Amp` | `U3` Pins 5 (`~MUTE`) & 12 (`~SHDN`), `R13` (10kΩ) | Stage 3 Audio Amplifier Enable (High = Play, Low = Mute/Shutdown) |
| **34** | `GPIO22` | `/SD_DAT0` | `J2` Pin 7 | MicroSD SDIO Data 0 (`DAT0`) |
| **35** | `GPIO23` | `/SD_DAT1` | `J2` Pin 8 | MicroSD SDIO Data 1 (`DAT1`) |
| **36** | `GPIO24` | `/SD_DAT2` | `J2` Pin 1 | MicroSD SDIO Data 2 (`DAT2`) |
| **37** | `GPIO25` | `/SD_DAT3` | `J2` Pin 2, `R1` (10kΩ to `+3.3V`) | MicroSD SDIO Data 3 (`DAT3`) with 10kΩ pull-up `R1` |
| **38** | `IOVDD` | `+3.3V` | +3.3V Plane, `C30` (100nF) | Digital I/O Power Supply (+3.3V) |
| **39** | `DVDD` | `+1V1` | Internal +1.1V Core Rail | Digital Core Power Supply (+1.1V) |
| **40** | `GPIO26_ADC0` | `/SD_CLK` | `J2` Pin 5 | MicroSD SDIO Clock (`CLK`) |
| **41** | `GPIO27_ADC1` | `/SD_CMD` | `J2` Pin 3 | MicroSD SDIO Command (`CMD`) |
| **42** | `GPIO28_ADC2` | `/OLED_SDA` | `U2` Pin 4 | Native `I2C0 SDA` — OLED Serial Data |
| **43** | `GPIO29_ADC3` | `/OLED_SCL` | `U2` Pin 3 | Native `I2C0 SCL` — OLED Serial Clock |
| **44** | `ADC_AVDD` | `+3.3V` | +3.3V Plane | ADC Analog Supply (+3.3V) |
| **45** | `IOVDD` | `+3.3V` | +3.3V Plane | Digital I/O Power Supply (+3.3V) |
| **46** | `VREG_VIN` | `+3.3V` | +3.3V Plane, `C22` (4.7µF) | Internal 1.1V Buck Regulator Input (+3.3V) |
| **47** | `VREG_PGND` | `GND` | Ground Plane | Internal 1.1V Buck Regulator Power Ground |
| **48** | `VREG_LX` | `/VREG_LX` | `L1` Pin 1 (3.3µH → `+1V1`) | Internal 1.1V Buck Regulator Switching Node |
| **49** | `USB_OTP_VDD` | `+3.3V` | +3.3V Plane | USB PHY & OTP Power Supply (+3.3V) |
| **50** | `DVDD` | `+1V1` | `L1` Pin 2, `C23` (4.7µF), `C24` (100nF) | Digital Core Power Supply (+1.1V) |
| **51** | `USB_DM` | `/MCU_USB_DM` | `R17` Pin 2 (27Ω → `/USB_D-`) | USB 2.0 Differential Data Minus (via 27Ω series `R17` & `D1` ESD) |
| **52** | `USB_DP` | `/MCU_USB_DP` | `R18` Pin 2 (27Ω → `/USB_D+`) | USB 2.0 Differential Data Plus (via 27Ω series `R18` & `D1` ESD) |
| **53** | `IOVDD` | `+3.3V` | +3.3V Plane | Digital I/O Power Supply (+3.3V) |
| **54** | `QSPI_IOVDD` | `+3.3V` | +3.3V Plane | Internal Stacked QSPI Flash Power Supply (+3.3V) |
| **55..59**| `QSPI_SD3..1, SCLK` | `NC` | Unconnected | Internal Stacked 2MB Flash Bus (Unconnected externally) |
| **60** | `~{QSPI_SS}` | `/QSPI_SS` | `R20` Pin 1 (1kΩ → `/BOOT_BTN`, `SW4` Pin 1) | USB UF2 Bootloader Select (`BOOTSEL` button `SW4` via 1kΩ `R20` to `GND`) |
| **61** | `GND` (EP) | `GND` | Ground Plane (Thermal Via) | Exposed Center Thermal / Ground Pad |

---

### 3.1a USB-C 5V/3A Receptacle (`J1`), ESD (`D1`), & 3-Stage Power Sequencer (`U6`, `Q1`, `Q2`)

| Designator | Pin / Part | Net Name | Connected To | Function |
| :--- | :--- | :--- | :--- | :--- |
| **`J1` Pins A4, A9, B4, B9** | `VBUS` | `VBUS` | `D1` Pin 5, `C17` (10µF), `C18` (100nF), `U6` Pin 3, `Q1` Pin 2, `R28` | Raw 5V USB Type-C Supply Input |
| **`J1` Pin A5** | `CC1` | `/USB_CC1` | `R21` Pin 1 (5.1kΩ 1% to `GND`) | USB-C Configuration Channel 1 (5.1kΩ Rd advertises 5V/3A sink) |
| **`J1` Pin B5** | `CC2` | `/USB_CC2` | `R22` Pin 1 (5.1kΩ 1% to `GND`) | USB-C Configuration Channel 2 (5.1kΩ Rd advertises 5V/3A sink) |
| **`J1` Pins A6, B6** | `D+` | `/USB_D+` | `D1` Pins 3 & 4, `R18` Pin 1 (27Ω) | USB 2.0 Data+ |
| **`J1` Pins A7, B7** | `D-` | `/USB_D-` | `D1` Pins 1 & 6, `R17` Pin 1 (27Ω) | USB 2.0 Data− |
| **`J1` Pins A1, A12, B1, B12, S1** | `GND` / `SHIELD` | `GND` | Ground Plane | Ground & Connector Shield |
| **`D1`** (`USBLC6-2SC6`) | Pins 1..6 | `/USB_D-`, `GND`, `/USB_D+`, `VBUS` | `J1`, `R17`, `R18` | Low-Capacitance Rail-to-Rail USB ESD Protection |
| **`U6`** (`AMS1117-3.3`) | Pin 1 (`GND`), Pin 2 (`VOUT`), Pin 3 (`VIN`) | `GND`, `+3.3V`, `VBUS` | `C17`/`C18` (`VBUS`), `C19` (10µF `+3.3V`), `R23` | **Stage 1:** Immediate `+3.3V` LDO Regulator (1A max) |
| **`R23` (47kΩ) + `C31` (100nF)** | RC Network | `/SEQ_GATE` | `U6` `+3.3V` → `Q2` Pin 1 (`G`) | **Stage 2 Delay:** $\tau = 4.7\,\text{ms}$ delay after `+3.3V` rises |
| **`Q2`** (`2N7002`, SOT-23) | Pin 1 (`G`), Pin 2 (`S`), Pin 3 (`D`) | `/SEQ_GATE`, `GND`, `/PFET_GATE` | `R23`/`C31`, `GND`, `R28`/`C32`/`Q1` Pin 1 | **Stage 2 Driver:** Pulls `PFET_GATE` low once `+3.3V` is stable |
| **`Q1`** (`AO3401A`, SOT-23) | Pin 1 (`G`), Pin 2 (`S`), Pin 3 (`D`) | `/PFET_GATE`, `VBUS`, `+5V` | `R28` (100kΩ to `VBUS`), `C32` (10nF to `+5V`) | **Stage 2 High-Side Switch:** 4A P-MOSFET with Miller soft-start (`C32`) supplying `+5V` |

---

### 3.2 YM2149F Sound Generators (`U1` & `U4`) Pinout

| Pin | Symbol | `U1` Net Name | `U4` Net Name | Connected To | Notes / Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `VSS` | `GND` | `GND` | Ground Plane | Power Ground |
| **2, 5, 39** | `NC` / `TEST1` | `NC` | `NC` | Unconnected | No Connect |
| **3** | `ANALOG_B` | `/ANALOG_B` | `/ANALOG_B2` | `R5` (1kΩ, `U1`) / `R10` (1kΩ, `U4`) | Channel B Analog Audio Output |
| **4** | `ANALOG_A` | `/ANALOG_A` | `/ANALOG_A2` | `R4` (1kΩ, `U1`) / `R9` (1kΩ, `U4`) | Channel A Analog Audio Output |
| **6..21** | `IOB7..IOA0` | `NC` | `NC` | Unconnected | Unused Parallel I/O Ports A & B |
| **22** | `CLOCK` | `/CLOCK` | `/CLOCK` | `R2` Pin 1 (33Ω from `U5` `GPIO13`) | Shared Master Clock Input |
| **23** | `~RESET` | `/~{RESET}` | `/~{RESET}` | `U5` `GPIO11`, `R3` (10kΩ to `GND`) | Shared Active-Low Hardware Reset |
| **24** | `A9` | `GND` | `GND` | Ground Plane | Fixed Address Decoding Bit 9 (`0`) |
| **25** | `A8` | `+5V` | `+5V` | Switched `+5V` Rail | Fixed Address Decoding Bit 8 (`1`) |
| **26** | `/SEL` | `/JP1_SEL` | `/JP1_SEL` | Solder Jumper `JP1` Pin 1 | YM2149F Clock Divider Select (`Open` = 1×, `Bridged` = ÷2) |
| **27** | `BDIR` | `/BDIR` | `/BDIR2` | `U5` `GPIO10` (`U1`) / `GPIO13` (`U4`) | Independent Bus Direction Control per PSG |
| **28** | `BC2` | `+5V` | `+5V` | Switched `+5V` Rail | Bus Control 2 (Tied HIGH to `+5V`) |
| **29** | `BC1` | `/BC1` | `/BC1_2` | `U5` `GPIO8` (`U1`) / `GPIO12` (`U4`) | Independent Bus Control 1 per PSG |
| **30..37**| `DA7..DA0` | `/DA7../DA0` | `/DA7../DA0` | `U5` `GPIO7..GPIO0` | Shared 8-Bit Bidirectional Data Bus |
| **38** | `ANALOG_C` | `/ANALOG_C` | `/ANALOG_C2` | `R7` (1kΩ, `U1`) / `R11` (1kΩ, `U4`) | Channel C Analog Audio Output |
| **40** | `VCC` | `+5V` | `+5V` | Switched `+5V` Rail | Main +5V Supply (`C1`+`C2` on `U1`, `C11`+`C12` on `U4`) |

#### YM2149F / AY-3-8910 Bus State Decoding Table (`BC2` Tied HIGH to `+5V`)

| `BDIR` (`GPIO10` / `GPIO13`) | `BC1` (`GPIO8` / `GPIO12`) | Bus State | Operation Description |
| :---: | :---: | :--- | :--- |
| **0** | **0** | **INACTIVE** | PSG data bus is High-Z / Inactive. |
| **0** | **1** | **READ** | MCU reads register contents from PSG data bus `DA[7:0]`. |
| **1** | **0** | **WRITE** | MCU writes data on `DA[7:0]` into currently addressed register. |
| **1** | **1** | **LATCH ADDRESS** | MCU selects target PSG register address (`0x00..0x0F`) on `DA[7:0]`. |

---

### 3.3 MicroSD Card Socket (`J2`) Pinout

| Pin | Card Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `DAT2` | `/SD_DAT2` | `U5` `GPIO24` (Pin 36) | SDIO Data 2 (`DAT2`) |
| **2** | `DAT3 / CD` | `/SD_DAT3` | `U5` `GPIO25` (Pin 37), `R1` | SDIO Data 3 (`DAT3`), with 10kΩ pull-up `R1` to `+3.3V` |
| **3** | `CMD` | `/SD_CMD` | `U5` `GPIO27` (Pin 41) | SDIO Command (`CMD`) |
| **4** | `VDD` | `+3.3V` | `+3.3V` Rail | +3.3V Power Supply (Decoupled by `C5` 100nF MLCC) |
| **5** | `CLK` | `/SD_CLK` | `U5` `GPIO26` (Pin 40) | SDIO Clock (`CLK`) |
| **6** | `VSS` | `GND` | Ground Plane | Power Ground |
| **7** | `DAT0` | `/SD_DAT0` | `U5` `GPIO22` (Pin 34) | SDIO Data 0 (`DAT0`) |
| **8** | `DAT1` | `/SD_DAT1` | `U5` `GPIO23` (Pin 35) | SDIO Data 1 (`DAT1`) |
| **9** | `Card Detect`| `NC` | Unconnected | Card Detect switch unused (card presence detected in software) |
| **10** | `SH` (shield) | `GND` | Ground Plane | Shield / housing pads (`SH` symbol pin; Hirose DM3AT shield tabs) |

**Socket:** Hirose `DM3AT-SF-PEJM5` (LCSC `C114218`, footprint `Connector_Card:microSD_HC_Hirose_DM3AT-SF-PEJM5`), symbol shield pin `SH` (formerly pin `10`/`Casing`). The integrated card-detect switch is left unconnected (presence detected in software).

---

### 3.4 Peripherals: OLED (`U2`), Buttons (`SW1..SW5`), Jumper (`JP1`)

| Designator | Pin / Signal | Net Name | Connected To | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`U2` Pin 1** | `VCC` | `+3.3V` | `+3.3V` Rail | Display Power (Decoupled by `C6` 10µF + `C7` 100nF) |
| **`U2` Pin 2** | `GND` | `GND` | Ground Plane | Display Power Ground |
| **`U2` Pin 3** | `SCL` | `/OLED_SCL` | `U5` `GPIO29` (Pin 43) | Native `I2C0 SCL` Serial Clock Line |
| **`U2` Pin 4** | `SDA` | `/OLED_SDA` | `U5` `GPIO28` (Pin 42) | Native `I2C0 SDA` Serial Data Line |
| **`SW1` Pin 1 / 2** | `1` / `2` | `/SW_PLAY` / `GND` | `U5` `GPIO19` (Pin 31) / `GND` | Play / Pause Navigation Button (Internal Pull-Up) |
| **`SW2` Pin 1 / 2** | `1` / `2` | `/SW_NEXT` / `GND` | `U5` `GPIO18` (Pin 29) / `GND` | Next Track Navigation Button (Internal Pull-Up) |
| **`SW3` Pin 1 / 2** | `1` / `2` | `/SW_PREV` / `GND` | `U5` `GPIO20` (Pin 32) / `GND` | Previous Track Navigation Button (Internal Pull-Up) |
| **`SW4` Pin 1 / 2** | `1` / `2` | `/BOOT_BTN` / `GND` | `R20` (1kΩ to `U5` Pin 60 `~QSPI_SS`) / `GND` | USB UF2 `BOOTSEL` Button |
| **`SW5` Pin 1 / 2** | `1` / `2` | `/MCU_RUN` / `GND` | `U5` Pin 26 (`RUN`), `R19` (10kΩ to `+3.3V`) / `GND` | MCU Hardware `RESET` Button |
| **`JP1` Pin 1 / 2** | `A` / `B` | `/JP1_SEL` / `GND` | `U1` Pin 26 + `U4` Pin 26 / `GND` | YM2149F `/SEL` Clock Divider Select (both PSGs) |

Rev 0.8: `SW1..SW5` are SMD XKB `TS-1187A` tactiles (LCSC `C318884`, footprint `Button_Switch_SMD:SW_Push_1P1T_XKB_TS-1187A`).

---

### 3.4a PAM8403 Stereo Class-D Amplifier (`U3`) & Speaker Connectors (`J3`, `J4`)

| Pin | Symbol | Net Name | Connected To | Notes / Function |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `LOUT+` | `/SPK_L+` | `J3` Pin 1 | Left speaker output + (BTL) |
| **2** | `PGND` | `GND` | Ground Plane | Power ground |
| **3** | `LOUT-` | `/SPK_L-` | `J3` Pin 2 | Left speaker output − (BTL) |
| **4** | `PVDD` | `+5V` | `+5V` Plane | Power-stage supply (`C14` 1µF under `U3`, `C13` 1µF + `C15` 10µF bulk) |
| **5** | `~MUTE` | `/SD_Amp` | `U5` `GPIO21` (Pin 33), `R13` (10kΩ) | Low = mute (tied to `~SHDN`) |
| **6** | `VDD` | `+5V` | `+5V` Plane | Analog supply (`C8` 1µF under `U3`) |
| **7** | `INL` | `/AMP_INL` | `C3` Pin 2 | Left input, AC-coupled from `MIX1` |
| **8** | `VREF` | `/AMP_VREF` | `C16` (1µF to `GND`) | Internal bias reference bypass |
| **9** | `NC` | `NC` | Unconnected | No connect |
| **10** | `INR` | `/AMP_INR` | `C10` Pin 2 | Right input, AC-coupled from `MIX2` |
| **11** | `GND` | `GND` | Ground Plane | Analog ground |
| **12** | `~SHDN` | `/SD_Amp` | `U5` `GPIO21` (Pin 33), `R13` (10kΩ) | Low = shutdown (tied to `~MUTE`) |
| **13** | `PVDD` | `+5V` | `+5V` Plane | Power-stage supply |
| **14** | `ROUT-` | `/SPK_R-` | `J4` Pin 2 | Right speaker output − (BTL) |
| **15** | `PGND` | `GND` | Ground Plane | Power ground |
| **16** | `ROUT+` | `/SPK_R+` | `J4` Pin 1 | Right speaker output + (BTL) |

---

### 3.5 4-Mode Hardware Stereo/Mono Analog Mixing Matrix (`U8`, `U9` — 2× `74HCT4053D`)

To support all classic chiptune stereo/mono topologies in hardware without distortion or level mismatch, the 6 individual 1 kΩ voice resistors (`R4`, `R5`, `R7` from `U1`; `R9`, `R10`, `R11` from `U4`) feed a dual **74HCT4053D** (`U8`, `U9`, SOIC-16 on `B.Cu`) current-steering analog switch matrix operating into the low-impedance summing nodes `MIX1` (Left) and `MIX2` (Right):

- **Fixed Voices:**
  - `U1` Voice A (`R4` = 1 kΩ) connects directly to `MIX1` (Left summing junction).
  - `U4` Voice C (`R11` = 1 kΩ) connects directly to `C1_SUM` (`U9` switch `C`), which routes to `MIX2` (Right) in Modes 0/1/3 and to `MIX_C` in Mode 2 (`ACB`).
- **Steered Voices & Center `-3 dB` Compensation (`R14`, `R15` = 470 Ω):**
  - When a voice pair (`B1+B2` in `ABC` mode, or `C1+C2` in `ACB` mode) is steered to **Center**, both `U8` switches connect its intermediate node (`MIX_B` or `MIX_C`) simultaneously to `MIX1` via `R14` (470 Ω) and to `MIX2` via `R15` (470 Ω).
  - Combined with the 1 kΩ series voice resistors and the switch $R_{\text{ON}} \approx 50\,\Omega$, the parallel 470 Ω split yields an effective per-side impedance of $\approx 1.41\,\text{k}\Omega$ ($-3.0\,\text{dB}$), preserving constant acoustic power across the stereo field.

#### `U8` & `U9` (`74HCT4053D`) Pinout Table

| Pin | Symbol | `U8` Net Name | `U9` Net Name | Function |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `B1` | `/MIX_C` | `/NODE_B_PAN` | Switch B Throw 1 |
| **2** | `B0` | `/MIX2` | `/MIX1` | Switch B Throw 0 |
| **3** | `C1` | `/MIX2` | `/NODE_C_PAN` | Switch C Throw 1 |
| **4** | `C` (COM) | `/MIX1` | `/C1_SUM` | Switch C Common (`R7` + `R11` on `U9`; `MIX1` mono bridge on `U8`) |
| **5** | `C0` | `NC` | `/MIX1` | Switch C Throw 0 |
| **6, 7, 8**| `~E`, `VEE`, `GND` | `GND` | `GND` | Enable (Active Low) & Negative/Ground Supply |
| **9** | `S3` (`C` sel) | `/MUX_MONO` | `/MUX_PAN` | Switch C Control Input (`GPIO17` on `U8`, `GPIO14` on `U9`) |
| **10** | `S2` (`B` sel) | `/MUX_C_CEN` | `/MUX_PAN` | Switch B Control Input (`GPIO16` on `U8`, `GPIO14` on `U9`) |
| **11** | `S1` (`A` sel) | `/MUX_B_CEN` | `/MUX_PAN` | Switch A Control Input (`GPIO15` on `U8`, `GPIO14` on `U9`) |
| **12** | `A0` | `/MIX2` | `/MIX2` | Switch A Throw 0 |
| **13** | `A1` | `/MIX_B` | `/MIX1` | Switch A Throw 1 |
| **14** | `A` (COM) | `/NODE_B_PAN` | `/ANALOG_A2_R9` | Switch A Common (`R9` from `U4` Voice A2 on `U9`) |
| **15** | `B` (COM) | `/NODE_C_PAN` | `/B1_SUM` | Switch B Common (`R5` from `U1` Voice B1 on `U9`) |
| **16** | `VCC` | `+5V` (`C33` 100nF) | `+5V` (`C34` 100nF) | +5V Supply (enables 3.3V TTL logic compatibility on `S1..S3`) |

#### 4-Mode Stereo / Mono Mixing Matrix Truth Table

| Mode | `MUX_MONO` (`GPIO17`) | `MUX_B_CEN` (`GPIO15`) | `MUX_C_CEN` (`GPIO16`) | `MUX_PAN` (`GPIO14`) | Left Channel (`AMP_INL`) | Right Channel (`AMP_INR`) | Use Case |
| :--- | :---: | :---: | :---: | :---: | :--- | :--- | :--- |
| **0: Dual-PSG Split (Boot Default)** | `0` | `0` | `0` | `0` | `U1` (`A1 + B1 + C1`) | `U4` (`A2 + B2 + C2`) | 6-voice TurboSound / Dual-PSG tracks (`U1` Left, `U4` Right) |
| **1: ABC Stereo** | `0` | `1` | `0` | `1` | `A1 + A2` + `0.71×(B1 + B2)` | `C1 + C2` + `0.71×(B1 + B2)` | Western / Atari ST / ZX Spectrum ABC stereo (`A`=L, `B`=C, `C`=R) |
| **2: ACB Stereo** | `0` | `0` | `1` | `0` | `A1 + A2` + `0.71×(C1 + C2)` | `B1 + B2` + `0.71×(C1 + C2)` | Eastern European / Pentagon / PT3 ACB stereo (`A`=L, `C`=C, `B`=R) |
| **3: True Mono** | `1` | `0` | `0` | `0` | `0.5×(U1 + U4)` (`A+B+C`) | `0.5×(U1 + U4)` (`A+B+C`) | Single-PSG or dual-PSG mono tracks played equally on both speakers |

---

### 3.6 Historical Rev 0.4 Native-Function Errata & Rev 0.5 Correction

The manufactured Rev 0.4 board predated the Rev 0.5–0.7 native pin assignment:
- `OLED_SDA` = `GPIO16` (`I2C0 SDA`), `OLED_SCL` = `GPIO17` (`I2C0 SCL`).
- `SD_DAT0..DAT3` = `GPIO18..GPIO21`, `SD_CLK` = `GPIO22`, `SD_CMD` = `GPIO28` (4-bit SDIO via RP2350 PIO `pico_sd_card`).

- **Superseded in Rev 0.8:** `SD_DAT0..DAT3` = `GPIO22..25`, `SD_CLK` = `GPIO26`, `SD_CMD` = `GPIO27`, `OLED_SDA` = `GPIO28`, `OLED_SCL` = `GPIO29`, navigation buttons = `GPIO18..20`, `SD_Amp` = `GPIO21`, MUX controls = `GPIO14..17`, PSG2 `BC1_2`/`BDIR2` = `GPIO12`/`GPIO13`, `MCU_CLK` = `GPIO13`.
---

## 4. Bill of Materials & LCSC Part Mapping (SSOT — Rev 0.8)

| Designator | Value / Part | Package / Footprint | Qty | LCSC Part # | Description |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **`C1`, `C6`, `C11`, `C15`, `C17`, `C19`** | `10uF` | `Capacitor_SMD:C_0805_2012Metric` | 6 | `C15850` | 10 µF 25V X5R 0805 JLC basic bulk capacitors (`+5V`, `VBUS`, `+3.3V`, `U2`) |
| **`C2`, `C4`, `C5`, `C7`, `C9`, `C12`, `C18`, `C24`..`C31`, `C33`, `C34`** | `100nF` | `Capacitor_SMD:C_0402_1005Metric` | 17 | `C307331` | MLCC 0402 high-frequency decoupling & RC-delay (`C31`) capacitors |
| **`C3`, `C8`, `C10`, `C13`, `C14`, `C16`** | `1uF` | `Capacitor_SMD:C_0402_1005Metric` | 6 | `C52923` | MLCC 0402 X7R AC coupling (`C3`, `C10`) & PAM8403 bypass (`C8`, `C13`, `C14`, `C16`) |
| **`C20`, `C21`** | `15pF` | `Capacitor_SMD:C_0402_1005Metric` | 2 | `C1548` | 15 pF 50V C0G 0402 crystal load capacitors |
| **`C22`, `C23`** | `4.7uF` | `Capacitor_SMD:C_0805_2012Metric` | 2 | `C1779` | 4.7 µF X5R 0805 buck regulator input/output capacitors |
| **`C32`** | `10nF` | `Capacitor_SMD:C_0402_1005Metric` | 1 | `C15195` | 10 nF 50V X7R 0402 Miller soft-start capacitor (`Q1`) |
| **`D1`** | `USBLC6-2SC6` | `Package_TO_SOT_SMD:SOT-23-6` | 1 | `C7519` | USB 2.0 low-capacitance ESD protection array |
| **`J1`** | `USB_C_Receptacle_USB2.0_16P` | `Connector_USB:USB_C_Receptacle_HRO_TYPE-C-31-M-12` | 1 | `C165948` | 16-pin USB 2.0 Type-C receptacle (5V/3A sink) |
| **`J2`** | `MicroSD` | `Connector_Card:microSD_HC_Hirose_DM3AT-SF-PEJM5` | 1 | `C114218` | Hirose DM3AT-SF-PEJM5 microSD socket (4-bit SDIO, card detect unused) |
| **`J3`, `J4`** | `Speaker_L`, `Speaker_R` | `Connector_JST:JST_PH_B2B-PH-K_1x02_P2.00mm_Vertical` | 2 | `C131337` | JST-PH 2-pin speaker connectors (left / right) |
| **`JP1`** | `Clk Div` | `Jumper:SolderJumper-2_P1.3mm_Open` | 1 | — | 2-pole solder jumper (YM2149F `/SEL` clock divider) |
| **`L1`** | `3.3uH` | `Inductor_SMD:L_Sunlord_MWSA0402S` | 1 | `C408336` | Sunlord MWSA0402S 3.3 µH shielded buck inductor |
| **`Q1`** | `AO3401A` | `Package_TO_SOT_SMD:SOT-23` | 1 | `C15127` | 30V 4A P-channel MOSFET (Stage 2 high-side `+5V` switch) |
| **`Q2`** | `2N7002` | `Package_TO_SOT_SMD:SOT-23` | 1 | `C8545` | 60V 300mA N-channel MOSFET (Stage 2 gate driver) |
| **`R1`, `R3`, `R13`, `R19`, `R24`..`R27`** | `10K` | `Resistor_SMD:R_0402_1005Metric` | 8 | `C25744` | 10 kΩ pull-up (`R1`, `R19`) and pull-down (`R3`, `R13`, `R24..R27`) |
| **`R2`** | `33` | `Resistor_SMD:R_0402_1005Metric` | 1 | `C25105` | 33 Ω PSG master clock (`GPIO13`) series damping |
| **`R4`, `R5`, `R7`..`R12`, `R16`, `R20`** | `1k` | `Resistor_SMD:R_0402_1005Metric` | 10 | `C11702` | 1 kΩ audio summing/load, crystal damping (`R16`), `BOOTSEL` (`R20`) |
| **`R14`, `R15`** | `470` | `Resistor_SMD:R_0402_1005Metric` | 2 | `C25117` | 470 Ω center-channel `-3 dB` attenuation |
| **`R17`, `R18`** | `27` | `Resistor_SMD:R_0402_1005Metric` | 2 | `C25100` | 27 Ω 1% USB 2.0 D−/D+ series termination |
| **`R21`, `R22`** | `5.1k` | `Resistor_SMD:R_0402_1005Metric` | 2 | `C25905` | 5.1 kΩ 1% USB-C `CC1`/`CC2` Rd pull-downs |
| **`R23`** | `47k` | `Resistor_SMD:R_0402_1005Metric` | 1 | `C25792` | 47 kΩ Stage 2 RC delay |
| **`R28`** | `100k` | `Resistor_SMD:R_0402_1005Metric` | 1 | `C25741` | 100 kΩ high-side P-MOSFET (`Q1`) gate pull-up |
| **`SW1`..`SW5`** | `TS-1187A` | `Button_Switch_SMD:SW_Push_1P1T_XKB_TS-1187A` | 5 | `C318884` | XKB TS-1187A SMD tactiles (`PLAY`, `NEXT`, `PREV`, `BOOTSEL`, `RESET`) |
| **`U1`, `U4`** | `YM2149F` | `Package_DIP:DIP-40_W15.24mm` | 2 | — | Yamaha 3-voice PSGs (DIP-40 socketed; hand-solder) |
| **`U2`** | `0.96"OLED_I2C` | `TModules:OLED 0.96 I2C` | 1 | — | 4-pin 0.96" SSD1306 I2C OLED module (hand-solder) |
| **`U3`** | `PAM8403` | `Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` | 1 | `C17337` | PAM8403 3W+3W filterless Class-D stereo amplifier |
| **`U5`** | `RP2354A` | `Package_DFN_QFN:QFN-60-1EP_7x7mm_P0.4mm_EP3.4x3.4mm` | 1 | `C41378174` | Raspberry Pi RP2354A (QFN-60, 2 MB internal flash) |
| **`U6`** | `AMS1117-3.3` | `Package_TO_SOT_SMD:SOT-223-3_TabPin2` | 1 | `C6186` | 3.3V 1A LDO (Stage 1 supply) |
| **`U8`, `U9`** | `74HCT4053D` | `Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` | 2 | `C87282` | Triple 2-channel analog multiplexer (3.3V TTL input compatible) |
| **`Y1`** | `12MHz` | `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm` | 1 | `C5160143` | 12.000 MHz SMD-3225 quartz crystal |
| **`H1`-`H4`** | `MountingHole_3mm` | `MountingHole:MountingHole_3mm` | 4 | — | 3 mm NPTH corner mounting holes (M3) |

All passives are JLC basic 0402/0805 parts and every `LCSC Part #` field is populated in the schematic for direct JLC assembly upload.

---

## 5. Revision History

| Rev | Date | Notes |
| :--- | :--- | :--- |
| **0.8** | **2026-10-20** | **JLC Assembly Prep**: Corrected GPIO map to the Rev 0.8 target — `MCU_CLK` → `GPIO13`; PSG2 `BC1_2`/`BDIR2` → `GPIO12`/`GPIO13`; MUX controls → `GPIO14`..`GPIO17`; nav buttons → `GPIO18`..`GPIO20`; `SD_Amp` → `GPIO21`; microSD 4-bit SDIO → `GPIO22`..`GPIO27`; OLED I²C → `GPIO28`/`GPIO29`. All passives converted to JLC basic 0402/0805 parts (`R*` 0402; 100 nF/1 µF/15 pF/10 nF 0402; 10 µF/4.7 µF 0805); `LCSC Part #` fields populated for every part (RP2354A `C41378174`, PAM8403 `C17337`, 74HCT4053D `C87282`, 12 MHz crystal `C5160143`, MWSA0402S `C408336`); microSD socket → Hirose `DM3AT-SF-PEJM5` (`C114218`) with `SH` shield pads; buttons → SMD `TS-1187A` (`C318884`). |
| **0.7** | **2026-10-19** | **Bare RP2354A + USB-C 5V/3A 3-Stage Power Sequencer + 4-Mode 74HCT4053D Stereo/Mono Matrix (`stereo-mod`)**: Replaced the `Raspberry Pi Pico 2` module (`A1`) with a bare **RP2354A** (`U5`, QFN-60 with 2 MB internal stacked QSPI NOR flash, exposing all 30 Bank-0 GPIOs `GPIO0..29`), 12 MHz crystal (`Y1`, `C20`, `C21`, `R16`), 3.3 µH internal 1.1V core buck inductor (`L1`, `C22`..`C30`), and `BOOTSEL` (`SW4`, `R20`) / `RESET` (`SW5`, `R19`) tactile switches. Added a 16-pin **USB-C 5V/3A receptacle** (`J1`) with independent 5.1 kΩ 1% pull-downs (`R21`, `R22`) on `CC1`/`CC2`, `USBLC6-2SC6` (`D1`) ESD protection, 27 Ω series resistors (`R17`, `R18`), and **3-stage power sequencing**: Stage 1 `AMS1117-3.3` (`U6`, `C17`..`C19`) immediately powers `+3.3V`/`+1V1`; Stage 2 RC delay (`R23` 47 kΩ + `C31` 100 nF) + `Q2` (`2N7002`) + `Q1` (`AO3401A` P-MOSFET with `R28` 100 kΩ & `C32` 10 nF soft-start) switches `+5V` on only after `+3.3V` is stable; Stage 3 `SD_Amp` (`GPIO9` + `R13` 10 kΩ pull-down) un-mutes `U3` (`PAM8403`) after coupling caps settle. Added a **4-mode hardware stereo/mono analog mixing matrix** (`U8`, `U9`: `2× 74HCT4053D` on `B.Cu`, `C33`, `C34`, `R14`, `R15` 470 Ω `-3 dB` center attenuators) driven by `GPIO23` (`MUX_PAN`), `GPIO24` (`MUX_B_CEN`), `GPIO25` (`MUX_C_CEN`), and `GPIO29` (`MUX_MONO`) (`R24`..`R27` 10 kΩ pull-downs), supporting Dual-PSG Split (`0000`), ABC Stereo (`0011`), ACB Stereo (`0101`), and True Mono (`1000`). |
| **0.6** | **2026-10-11** | **Dual-PSG stereo variant**: Added second YM2149F (`U4`) on `GPIO26`/`GPIO27`, 4-bit SDIO microSD (`J2`), and on-board PAM8403 stereo Class-D amplifier (`U3`, `J3`, `J4`). |
| **0.5** | **2026-10-11** | **Native-function pin correction**: Re-mapped OLED I²C (`GPIO16`/`17`) and microSD (`GPIO18`..`22`) to RP2350 native peripheral pins. |
| **0.4** | **2026-08-15** | **Manufactured Production Release (Single-PSG + Pico 2)**. |
| 0.2 | 2026-08-12 | Intermediate documentation revision. |
| 0.1 | 2026-08-08 | Initial hardware documentation release. |

---

## 6. License & Non-Commercial Notice

Copyright (c) 2026 Terence Ang <terenceang@mac.com> / MotionFxDesign.

This hardware project and documentation are licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC BY-NC-SA 4.0)**.

> **NOT FOR COMMERCIAL USE**  
> You are free to share, copy, modify, and build upon this hardware design for personal, hobby, educational, and non-commercial purposes with attribution. Commercial manufacture, sale, or distribution for profit is strictly prohibited without prior written permission from the author.
