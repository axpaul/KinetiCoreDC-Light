# KinetiCoreDC Light

![MCU](https://img.shields.io/badge/MCU-STM32G070CBT6-03234B?style=flat-square&logo=stmicroelectronics)
![Driver](https://img.shields.io/badge/Driver-DRV8874PWPR-CC0000?style=flat-square&logo=texasinstruments)
![Variant](https://img.shields.io/badge/Variant-Light%20%2F%20Compact-2EA44F?style=flat-square)
![EDA](https://img.shields.io/badge/KiCad-10.0-314CB0?style=flat-square&logo=kicad)
![License](https://img.shields.io/badge/License-CERN--OHL--S--2.0-blue?style=flat-square)

🌐 **Langue / Language :** [🇫🇷 Français](README.md) | [🇬🇧 English](README.en.md)

---

**KinetiCoreDC Light** est la **version minimale et compacte** du contrôleur de moteur DC durci **[KinetiCoreDC](https://github.com/axpaul/KinetiCoreDC)**. Conçue pour réduire au maximum l'encombrement PCB et le coût de nomenclature (BOM), cette variante conserve l'étage de puissance protégé, le pont en H `DRV8874`, les entrées de commande opto-isolées et le retour d'encodeur incrémental, tout en supprimant les bus périphériques encombrants de la version complète. Développé par **Paul Miailhe** pour **Berry Rocket**.

© 2025/2026 - **Paul Miailhe** (*Berry Rocket*).  
This material is licensed under the **CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S)**. $\rightarrow$ [LICENSE](LICENSE)

---

## 1. Différences avec le dépôt officiel `KinetiCoreDC`

Cette édition **Light** cible les intégrations embarquées à fort contrainte de volume où un pilotage PWM/RC local avec retour incrémental suffit :

| Fonction / Bloc | [KinetiCoreDC (Officiel)](https://github.com/axpaul/KinetiCoreDC) | KinetiCoreDC Light (Ce dépôt) |
| :--- | :---: | :---: |
| **Protection alim (`LTC4365` + 2× `SiS412DN`) & LDO `3,3 V`** | ✅ Inclus | ✅ **Conservé** |
| **Driver moteur `DRV8874PWPR` ($6\text{ A}$ crête) + mesure `IPROPI`** | ✅ Inclus | ✅ **Conservé** |
| **Entrées opto-isolées PWM / GPIO (`2× TLP2310`)** | ✅ Inclus | ✅ **Conservé** (voie principale + redondante) |
| **Encodeur quadrature incrémental A/B (`TIM3` + Schmitt)** | ✅ Inclus | ✅ **Conservé** |
| **Fins de course (`2×` contact sec filtré + Schmitt)** | ✅ Inclus | ✅ **Conservé** |
| **Debug SWD + UART VCP (`Tag-Connect TC2070`)** | ✅ Inclus | ✅ **Conservé** |
| **IHM embarquée (LED RGB, buzzer piézo, LEDs sens moteur)** | ✅ Inclus | ✅ **Conservé** |
| **Sortie puissance moteur** | Connecteur `XT30PW-M` | 🔧 **Pads à souder directs** (`OUTA` / `OUTB`) |
| **Bus terrain différentiel RS-485 (`THVD1400D` + `SM712`)** | ✅ Inclus | ❌ **Retiré** (gain de surface PCB) |
| **Port capteur absolu SPI2 / I²C1 (`AS5047P` / `AS5600`)** | ✅ Inclus | ❌ **Retiré** (gain de surface PCB) |

---

## 2. Vue d'ensemble & Applications

Articulée autour d'un microcontrôleur **STM32G070CBT6** (ARM Cortex-M0+ @ $64\text{ MHz}$, horloge interne HSI) et d'un pont en H intégré **DRV8874PWPR**, la carte intègre l'essentiel de l'électronique de puissance, de protection et de conditionnement nécessaire au pilotage précis d'un actionneur DC :

- **Actionneur PWM / RC durci ou redondant :** Pilotage par consigne PWM ou impulsion RC ($1\text{–}2\text{ ms}$) via deux entrées opto-isolées (`TLP2310`) autorisant une voie de commande secondaire de secours.
- **Asservissement compact en boucle fermée :** Régulation PID en position, vitesse ou couple grâce au décodage quadrature matériel (`TIM3`), à la prise d'origine (*homing*) sur butées mécaniques et à la mesure de courant moteur (`IPROPI`).

---

## 3. Caractéristiques électriques

L'étage d'entrée est dimensionné pour une alimentation sur batterie **LiPo 2S ou 3S** (ou source DC régulée). Un contrôleur **LTC4365TS8** associé à deux MOSFETs N-Channel tête-bêche (**SiS412DN**) assure la coupure matérielle en cas de sous-tension, surtension ou inversion de polarité, tandis qu'une diode TVS **SMBJ15CA** protège le régulateur $+3{,}3\text{ V}$ contre les transitoires.

| Paramètre | Symbole | Min | Typ | Max | Unité | Conditions / Notes |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Tension d'entrée** | $V_{\text{IN}}$ | $5{,}90$ | $7{,}4\text{–}11{,}1$ | **$14{,}75$** | $\text{V}$ | Compatible LiPo 2S ($8{,}4\text{ V}$) et 3S ($12{,}6\text{ V}$) |
| **Tenue inversion polarité** | $V_{\text{REV}}$ | **$-40$** | — | — | $\text{V}$ | Blocage par N-MOSFETs tête-bêche |
| **Seuil sous-tension (coupure)** | $V_{\text{UVLO}}$ | — | $5{,}90$ | — | $\text{V}$ | Enclenchement à $6{,}20\text{ V}$ (config. défaut) |
| **Seuil surtension (coupure)** | $V_{\text{OVLO}}$ | — | **$14{,}75$** | — | $\text{V}$ | Réarmement à $14{,}01\text{ V}$ (TVS `SMBJ15CA`) |
| **Courant moteur régulé** | $I_{\text{TRIP}}$ | — | **$2{,}6$** | **$2{,}6$** | $\text{A}$ | $V_{\text{REF}} = 2{,}50\text{ V}$, $R_{\text{IPROPI}} = 2{,}49\text{ k}\Omega$ |
| **Courant moteur crête** | $I_{\text{PEAK}}$ | — | — | **$6{,}0$** | $\text{A}$ | Limite silicium `DRV8874` |
| **Puissance d'entrée** | $P_{\text{IN}}$ | — | $32{,}8$ | $38{,}4$ | $\text{W}$ | À $I_{\text{TRIP}} = 2{,}6\text{ A}$ (jusqu'à $88{,}5\text{ W}$ crête à $6\text{ A}$) |
| **Rail logique** | $V_{\text{3V3}}$ | $3{,}28$ | $3{,}30$ | $3{,}32$ | $\text{V}$ | Précision $\pm 0{,}4\text{ \%}$ (`MCP1703A`, boîtier SOT-223) |
| **Courant disponible $+3{,}3\text{ V}$** | $I_{\text{OUT(3V3)}}$ | — | — | **$250$** | $\text{mA}$ | Total carte ($\approx 50\text{ mA}$) + encodeur externe |
| **Courant de veille** | $I_{\text{SHDN}}$ | — | — | **$< 10$** | $\mu\text{A}$ | Broche `SHDN_EXT` tirée à `GND` |

> **Ajustement des seuils :** La fenêtre de protection UVLO/OVLO peut être adaptée à un profil batterie spécifique (LiPo 2S dédié $6{,}45\text{–}10{,}84\text{ V}$, LiPo 3S dédié $9{,}15\text{–}14{,}28\text{ V}$ ou pile 9V) via le pont diviseur du `LTC4365` sur la feuille `Power.kicad_sch`. Le seuil de courant $I_{\text{TRIP}}$ est également ajustable jusqu'à $6\text{ A}$ via `R_IPROPI`.

---

## 4. Interfaces & Connectique

Toutes les entrées/sorties externes conservées bénéficient des mêmes protections contre les décharges électrostatiques (ESD), les inversions de câblage et le bruit de découpage moteur ($32\text{ kHz}$). Le départ d'alimentation $+3{,}3\text{ V}$ vers l'encodeur (`J10`) est protégé par une résistance série de $22\ \Omega$, tandis que les signaux logiques sont filtrés matériellement puis remis en forme par des triggers de Schmitt (**SN74LVC2G14DBV**).

| Connecteur | Interface | Signal / Protocole | Conditionnement & Protection | Compatibilité |
| :--- | :--- | :--- | :--- | :--- |
| **`J2`** | `XT30PW-M` (2 pts) | Puissance entrée $V_{\text{IN}}$ | TVS `SMBJ15CA`, N-MOSFETs `SiS412DN` | Batterie LiPo 2S–3S ($15\text{ A}$ cont.) |
| **`J3` / `J4`** | Pads à souder (`1×1`) | Sorties moteur `OUTA` / `OUTB` | LEDs antiparallèles sens/PWM ($2\text{ k}\Omega$) | Moteur DC à balais (fils directs) |
| **`J7` / `J8`** | `JST-GH` (2 pts) | $2\times$ PWM / GPIO isolés ($3{,}3\text{–}5\text{ V}$) | Optocoupleurs `TLP2310`, clamp inverse `SMF6.0A` | Consigne PWM/RC, voie redondante |
| **`J10`** | `JST-GH` (4 pts) | Encodeur quadrature A/B ($3{,}3\text{–}5\text{ V}$) | `USBLC6-2SC6`, $+3{,}3\text{ V}$ ($22\ \Omega$), RC ($160\text{ kHz}$) + Schmitt | Encodeurs incrémentaux à effet Hall |
| **`J11` / `J5`** | `JST-GH` (2 pts) | $2\times$ Fins de course (`TRIG1` / `TRIG2`) | Pull-up $2{,}2\text{ k}\Omega$, RC ($160\text{ Hz}$) + Schmitt | Butées mécaniques Min / Max |
| **`J9`** | `JST-GH` (3 pts) | Activation carte (`SHDN_EXT`) | Pull-up $510\text{ k}\Omega$ vers $V_{\text{IN}}$ (`LTC4365`) | Interrupteur marche / veille |
| **`J6`** | `JST-GH` (2 pts) | Sortie défaut alim (`NFAULT_EXT`) | Drain ouvert actif bas + $22\ \Omega$ (`LTC4365`) | Supervision système hôte |
| **`J1`** | `TC2070-NL` (14 pts) | SWD + NRST + UART VCP (`USART3`) | Empreinte sans composants (STDC14) | ST-LINK/V3 (flash & debug) |

---

## 5. Brochage MCU (`STM32G070CBT6` — LQFP48)

| Bloc fonctionnel | Signal | Broche | Périphérique | Fonction |
| :--- | :--- | :---: | :--- | :--- |
| **Driver Moteur** | `PH/IN2_MCU` | `PA8` (28) | `TIM1_CH1` | Direction (`PH`) ou PWM secondaire |
| | `EN/IN1_MCU` | `PA9` (29) | `TIM1_CH2` | PWM principale moteur ($32\text{ kHz}$) |
| | `NSLEEP_MCU` | `PC6` (30) | `GPIO_Output` | Activation / Veille du pont en H |
| | `PMODE_MCU` | `PC7` (31) | `GPIO_Output` | Mode `PH/EN` (`0`) ou `PWM` (`1`) |
| | `NFAULT_MCU` | `PA10` (32) | `GPIO_EXTI10` | Interruption défaut driver (front descendant) |
| | `IPROPI_MCU` | `PB10` (22) | `ADC1_IN11` | Lecture courant moteur ($1{,}13\text{ V/A}$, filtre RC) |
| **Supervision** | `ADC_VM` | `PB2` (21) | `ADC1_IN10` | Mesure tension d'entrée $V_M$ |
| **Encodeur** | `TIM_ENC_A` / `B` | `PA6` (17), `PA7` (18) | `TIM3_CH1/CH2` | Décodage quadrature matériel X4 |
| **Optocoupleurs** | `OPTO1_MCU` / `OPTO2_MCU` | `PB14` (26), `PB15` (27) | `TIM15_CH1/CH2` | Input Capture PWM / Consigne redondante |
| **Fins de course** | `TRIG1_MCU` / `TRIG2_MCU` | `PD2` (40), `PD3` (41) | `GPIO_EXTI2/EXTI3` | Interruptions butées (front montant) |
| **IHM** | `BLUE` / `RED` / `GREEN` | `PC13`, `PC14`, `PC15` | `GPIO_Output` | LED RGB 8 couleurs (anode commune, actif bas) |
| | `BUZZER_MCU` | `PA4` (15) | `TIM14_CH1` | Signal piézo PWM $4\text{ kHz}$ |
| **Debug** | `SWDIO` / `SWCLK` | `PA13` (35), `PA14` (36) | `SYS_SWD` | Programmation et débogage |
| | `VPC_TX` / `VPC_RX` | `PA5` (16), `PB0` (19) | `USART3_TX/RX` | Console série virtuelle de diagnostic |

---

## 6. Architecture KiCad & Structure du dépôt

Le projet est conçu sous **KiCad 10.0** autour d'une feuille racine et de trois sous-feuilles hiérarchiques allégées, accompagnées du fichier de configuration **STM32CubeMX** (`KinetiCoreDC.ioc`) :

```text
KinetiCoreDC-Light/
├── KinetiCoreDC.kicad_pro       # Projet KiCad 10.0
├── KinetiCoreDC.kicad_sch       # Feuille 1/4 : MCU STM32G070CBT6 & hiérarchie
├── Interface.kicad_sch          # Feuille 2/4 : Optocoupleurs, encodeur quadrature, butées
├── Power.kicad_sch              # Feuille 3/4 : LTC4365, LDO 3.3V, DRV8874, RGB, buzzer
├── Connector.kicad_sch          # Feuille 4/4 : XT30 entrée, pads moteur, JST-GH, Tag-Connect
├── KinetiCoreDC.kicad_pcb       # Routage PCB compact
├── KinetiCoreDC.ioc             # Configuration matérielle STM32CubeMX
├── Datasheet/                   # Fiches techniques des composants
├── lib/                         # Symboles, empreintes et modèles 3D locaux
├── README.md                    # Documentation française
├── README.en.md                 # Documentation anglaise
└── LICENSE                      # Licence CERN-OHL-S-2.0