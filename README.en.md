# KinetiCoreDC Light

![MCU](https://img.shields.io/badge/MCU-STM32G070CBT6-03234B?style=flat-square&logo=stmicroelectronics)
![Driver](https://img.shields.io/badge/Driver-DRV8874PWPR-CC0000?style=flat-square&logo=texasinstruments)
![Variant](https://img.shields.io/badge/Variant-Light%20%2F%20Compact-2EA44F?style=flat-square)
![EDA](https://img.shields.io/badge/KiCad-10.0-314CB0?style=flat-square&logo=kicad)
![License](https://img.shields.io/badge/License-CERN--OHL--S--2.0-blue?style=flat-square)

🌐 **Language / Langue :** [🇫🇷 Français](README.md) | [🇬🇧 English](README.en.md)

---

**KinetiCoreDC Light** is the **minimal, compact variant** of the ruggedized closed-loop DC motor controller **[KinetiCoreDC](https://github.com/axpaul/KinetiCoreDC)**. Engineered to minimize PCB footprint and Bill of Materials (BOM) cost, this edition retains the full input power protection stage, `DRV8874` H-bridge driver, optoisolated control inputs, and incremental quadrature encoder interface, while stripping out bulky fieldbus and external sensor peripherals from the full version. Developed by **Paul Miailhe** for **Berry Rocket**.

© 2025/2026 - **Paul Miailhe** (*Berry Rocket*).  
This material is licensed under the **CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S)**. $\rightarrow$ [LICENSE](LICENSE)

---

## 1. Differences from the Official `KinetiCoreDC` Repository

This **Light** edition targets space-constrained embedded integrations where local PWM/RC control with incremental encoder feedback is sufficient:

| Feature / Functional Block | [KinetiCoreDC (Official)](https://github.com/axpaul/KinetiCoreDC) | KinetiCoreDC Light (This Repo) |
| :--- | :---: | :---: |
| **Power protection (`LTC4365` + 2× `SiS412DN`) & `3.3 V` LDO** | ✅ Included | ✅ **Retained** |
| **Motor driver `DRV8874PWPR` ($6\text{ A}$ peak) + `IPROPI` sensing** | ✅ Included | ✅ **Retained** |
| **Optoisolated PWM / GPIO inputs (`2× TLP2310`)** | ✅ Included | ✅ **Retained** (primary + redundant channel) |
| **Incremental quadrature A/B encoder (`TIM3` + Schmitt)** | ✅ Included | ✅ **Retained** |
| **Limit switches (`2×` RC-filtered dry contact + Schmitt)** | ✅ Included | ✅ **Retained** |
| **SWD debug + VCP UART (`Tag-Connect TC2070`)** | ✅ Included | ✅ **Retained** |
| **On-board HMI (RGB LED, piezo buzzer, motor polarity LEDs)** | ✅ Included | ✅ **Retained** |
| **Motor power output** | `XT30PW-M` connector | 🔧 **Direct solder pads** (`OUTA` / `OUTB`) |
| **Differential RS-485 fieldbus (`THVD1400D` + `SM712`)** | ✅ Included | ❌ **Removed** (PCB area reduction) |
| **Absolute encoder SPI2 / I²C1 port (`AS5047P` / `AS5600`)** | ✅ Included | ❌ **Removed** (PCB area reduction) |

---

## 2. Overview & Applications

Built around an **STM32G070CBT6** microcontroller (ARM Cortex-M0+ @ $64\text{ MHz}$, internal HSI clock) and an integrated **DRV8874PWPR** H-bridge driver, the board integrates the essential power, protection, and signal-conditioning electronics required for precise DC actuator control:

- **Ruggedized or Redundant PWM / RC Actuator:** Driven via PWM duty-cycle or standard RC servo pulses ($1\text{–}2\text{ ms}$) across two optoisolated inputs (`TLP2310`), supporting a secondary fail-safe command channel.
- **Compact Closed-Loop Servo Control:** Position, velocity, and torque PID regulation using hardware quadrature decoding (`TIM3`), mechanical end-stop homing, and cycle-by-cycle motor current sensing (`IPROPI`).

---

## 3. Electrical Characteristics

The input stage is rated for **2S or 3S LiPo** battery packs (or regulated DC supplies). An **LTC4365TS8** controller driving back-to-back N-Channel MOSFETs (**SiS412DN**) provides hardware disconnect during undervoltage, overvoltage, or reverse-polarity events, while an **SMBJ15CA** TVS diode clamps transients to protect the $+3.3\text{ V}$ regulator.

| Parameter | Symbol | Min | Typ | Max | Unit | Conditions / Notes |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Input supply voltage** | $V_{\text{IN}}$ | $5.90$ | $7.4\text{–}11.1$ | **$14.75$** | $\text{V}$ | Compatible with 2S ($8.4\text{ V}$) & 3S ($12.6\text{ V}$) LiPo |
| **Reverse polarity rating** | $V_{\text{REV}}$ | **$-40$** | — | — | $\text{V}$ | Blocked by back-to-back N-MOSFETs |
| **Undervoltage cutoff** | $V_{\text{UVLO}}$ | — | $5.90$ | — | $\text{V}$ | Turns on at $6.20\text{ V}$ (default resistor network) |
| **Overvoltage cutoff** | $V_{\text{OVLO}}$ | — | **$14.75$** | — | $\text{V}$ | Recovers at $14.01\text{ V}$ (`SMBJ15CA` TVS clamp) |
| **Regulated motor current** | $I_{\text{TRIP}}$ | — | **$2.6$** | **$2.6$** | $\text{A}$ | $V_{\text{REF}} = 2.50\text{ V}$, $R_{\text{IPROPI}} = 2.49\text{ k}\Omega$ |
| **Peak motor current** | $I_{\text{PEAK}}$ | — | — | **$6.0$** | $\text{A}$ | `DRV8874` silicon current limit |
| **Input power** | $P_{\text{IN}}$ | — | $32.8$ | $38.4$ | $\text{W}$ | At $I_{\text{TRIP}} = 2.6\text{ A}$ (up to $88.5\text{ W}$ peak at $6\text{ A}$) |
| **Logic rail voltage** | $V_{\text{3V3}}$ | $3.28$ | $3.30$ | $3.32$ | $\text{V}$ | $\pm 0.4\%$ accuracy (`MCP1703A`, SOT-223 package) |
| **Available $+3.3\text{ V}$ current** | $I_{\text{OUT(3V3)}}$ | — | — | **$250$** | $\text{mA}$ | Total board ($\approx 50\text{ mA}$) + external encoder |
| **Shutdown quiescent current** | $I_{\text{SHDN}}$ | — | — | **$< 10$** | $\mu\text{A}$ | `SHDN_EXT` pin pulled to `GND` |

> **Threshold Customization:** The UVLO/OVLO protection window can be tailored to a dedicated battery chemistry (2S LiPo $6.45\text{–}10.84\text{ V}$, 3S LiPo $9.15\text{–}14.28\text{ V}$, or 9V battery) via the `LTC4365` resistor divider on `Power.kicad_sch`. The current trip limit $I_{\text{TRIP}}$ is also adjustable up to $6\text{ A}$ via `R_IPROPI`.

---

## 4. Interfaces & Connectors

All retained external I/Os feature electrostatic discharge (ESD) protection, reverse-wiring tolerance, and filtering against motor PWM switching noise ($32\text{ kHz}$). The $+3.3\text{ V}$ sensor supply output (`J10`) is current-limited by a $22\ \Omega$ series resistor, while logic inputs pass through hardware RC low-pass filters followed by Schmitt-trigger buffers (**SN74LVC2G14DBV**).

| Connector | Interface | Signal / Protocol | Conditioning & Protection | Compatibility |
| :--- | :--- | :--- | :--- | :--- |
| **`J2`** | `XT30PW-M` (2-pin) | Main power input $V_{\text{IN}}$ | `SMBJ15CA` TVS, `SiS412DN` N-MOSFETs | 2S–3S LiPo battery ($15\text{ A}$ cont.) |
| **`J3` / `J4`** | Solder pads (`1×1`) | Motor outputs `OUTA` / `OUTB` | Antiparallel polarity/PWM LEDs ($2\text{ k}\Omega$) | Brushed DC motor (direct wire solder) |
| **`J7` / `J8`** | `JST-GH` (2-pin) | $2\times$ Isolated PWM / GPIO ($3.3\text{–}5\text{ V}$) | `TLP2310` optocouplers, `SMF6.0A` clamp | PWM/RC setpoint, redundant input |
| **`J10`** | `JST-GH` (4-pin) | Quadrature encoder A/B ($3.3\text{–}5\text{ V}$) | `USBLC6-2SC6`, $+3.3\text{ V}$ ($22\ \Omega$), RC ($160\text{ kHz}$) + Schmitt | Hall-effect incremental encoders |
| **`J11` / `J5`** | `JST-GH` (2-pin) | $2\times$ Limit switches (`TRIG1` / `TRIG2`) | $2.2\text{ k}\Omega$ pull-up, RC ($160\text{ Hz}$) + Schmitt | Min / Max mechanical end-stops |
| **`J9`** | `JST-GH` (3-pin) | Board enable (`SHDN_EXT`) | $510\text{ k}\Omega$ pull-up to $V_{\text{IN}}$ (`LTC4365`) | Hardware power / sleep switch |
| **`J6`** | `JST-GH` (2-pin) | Power fault output (`NFAULT_EXT`) | Open-drain active-LOW + $22\ \Omega$ (`LTC4365`) | Host system fault monitoring |
| **`J1`** | `TC2070-NL` (14-pin) | SWD + NRST + VCP UART (`USART3`) | No-legs spring-pin footprint (STDC14) | ST-LINK/V3 (flashing & debugging) |

---

## 5. MCU Pinout (`STM32G070CBT6` — LQFP48)

| Functional Block | Signal | Pin | Peripheral | Function |
| :--- | :--- | :---: | :--- | :--- |
| **Motor Driver** | `PH/IN2_MCU` | `PA8` (28) | `TIM1_CH1` | Direction (`PH`) or secondary PWM |
| | `EN/IN1_MCU` | `PA9` (29) | `TIM1_CH2` | Primary motor PWM ($32\text{ kHz}$) |
| | `NSLEEP_MCU` | `PC6` (30) | `GPIO_Output` | H-bridge enable / sleep control |
| | `PMODE_MCU` | `PC7` (31) | `GPIO_Output` | `PH/EN` (`0`) or `PWM` (`1`) mode select |
| | `NFAULT_MCU` | `PA10` (32) | `GPIO_EXTI10` | Driver fault interrupt (falling edge) |
| | `IPROPI_MCU` | `PB10` (22) | `ADC1_IN11` | Motor current sense ($1.13\text{ V/A}$, RC filtered) |
| **Supervision** | `ADC_VM` | `PB2` (21) | `ADC1_IN10` | Input supply voltage $V_M$ measurement |
| **Encoder** | `TIM_ENC_A` / `B` | `PA6` (17), `PA7` (18) | `TIM3_CH1/CH2` | Hardware X4 quadrature decoding |
| **Optocouplers** | `OPTO1_MCU` / `OPTO2_MCU` | `PB14` (26), `PB15` (27) | `TIM15_CH1/CH2` | PWM Input Capture / redundant command |
| **Limit Switches** | `TRIG1_MCU` / `TRIG2_MCU` | `PD2` (40), `PD3` (41) | `GPIO_EXTI2/EXTI3` | End-stop interrupts (rising edge) |
| **HMI** | `BLUE` / `RED` / `GREEN` | `PC13`, `PC14`, `PC15` | `GPIO_Output` | 8-color RGB LED (common anode, active-LOW) |
| | `BUZZER_MCU` | `PA4` (15) | `TIM14_CH1` | $4\text{ kHz}$ piezo buzzer PWM drive |
| **Debug** | `SWDIO` / `SWCLK` | `PA13` (35), `PA14` (36) | `SYS_SWD` | Programming and hardware debugging |
| | `VPC_TX` / `VPC_RX` | `PA5` (16), `PB0` (19) | `USART3_TX/RX` | Virtual COM port diagnostic serial console |

---

## 6. KiCad Architecture & Repository Structure

The hardware is designed in **KiCad 10.0** using a streamlined root sheet and three hierarchical sub-sheets, paired with the **STM32CubeMX** configuration file (`KinetiCoreDC.ioc`):

```text
KinetiCoreDC-Light/
├── KinetiCoreDC.kicad_pro       # KiCad 10.0 project file
├── KinetiCoreDC.kicad_sch       # Sheet 1/4: STM32G070CBT6 MCU & hierarchy
├── Interface.kicad_sch          # Sheet 2/4: Optocouplers, quadrature encoder, limit switches
├── Power.kicad_sch              # Sheet 3/4: LTC4365, 3.3V LDO, DRV8874, RGB LED, buzzer
├── Connector.kicad_sch          # Sheet 4/4: XT30 input, motor solder pads, JST-GH, Tag-Connect
├── KinetiCoreDC.kicad_pcb       # Compact PCB layout
├── KinetiCoreDC.ioc             # STM32CubeMX hardware pinout & peripheral config
├── Datasheet/                   # Component datasheets
├── lib/                         # Local symbols, footprints, and 3D STEP models
├── README.md                    # French documentation
├── README.en.md                 # English documentation
└── LICENSE                      # CERN-OHL-S-2.0 License