# GLUEDIA-NI (GLUEDIA 2) — Physical Sensor Probe Assembly & Interface Specification

This document details the **physical sensor assembly** that sits directly against human skin, collects the biological signals, and feeds them into the **GLUEDIA-NI ASIC microchip**.

---

## 1. System Architecture: On-Chip Silicon vs. Off-Chip Sensor Board

In a commercial medical device, the system is divided into two distinct hardware modules:

```
┌──────────────────────────────────────────────────┐         ┌──────────────────────────────────────────────────┐
│          OFF-CHIP SENSOR PROBE MODULE            │         │             GLUEDIA-NI ASIC MICROCHIP            │
│       (18 mm Flexible Polyimide Flex PCB)        │         │          (0.80 mm × 0.48 mm Silicon Die)         │
├──────────────────────────────────────────────────┤         ├──────────────────────────────────────────────────┤
│ • 940 nm & 1050 nm VCSEL Light Emitters          │         │ • Dual 10 MΩ Transimpedance Amplifiers (TIAs)   │
│ • Pulsed Mid-IR Laser Diode (Photoacoustic)      │  10-Pin │ • 8-Bit Current-Steering DC Baseline Subtraction │
│ • Dual InGaAs / Si-PIN Photodiode Detectors      │  Flex   │ • Folded-Cascode CSA + Pseudo-Resistor (>1.2 GΩ) │
│ • 200 kHz Piezoelectric Ultrasound Transducer    │ Ribbon  │ • 3-Bit PGA (20–40 dB) + 200 kHz Bandpass Filter │
│ • 4 Concentric Gold-Plated Bioimpedance Rings    │ ──────► │ • Multi-Channel 16-Bit ΔΣ Analog-to-Digital Conv │
│ • Skin Contact Temperature Sensor (TMP117)       │         │ • Digital Lock-In Demodulator (+52 dB SNR Gain)  │
│ • 3-Axis Accelerometer (Motion Gate)             │         │ • 8→32→16→1 Pipelined Neural Network (Q8.8)      │
│ • Sapphire Optical Window & Black Rubber Baffle  │         │ • Calibration LUT, Hypo/Hyper Alarms & SPI Bus   │
└──────────────────────────────────────────────────┘         └──────────────────────────────────────────────────┘
```

---

## 2. Mechanical Design & Probe Head Geometry

The sensor probe is laid out as an **18 mm concentric annular disk** mounted on the underside of a smartwatch, armband, or wearable adhesive patch:

```
                      CONCENTRIC SENSOR PROBE HEAD
                       (Bottom View Facing Skin)

      ┌─────────────────────────────────────────────────────────────┐
      │  (E4)  OUTER BIOIMPEDANCE CURRENT DRIVE ELECTRODE (+)       │
      │   ┌─────────────────────────────────────────────────────┐   │
      │   │  (E2)  INNER BIOIMPEDANCE VOLTAGE SENSE (+)         │   │
      │   │   ┌─────────────────────────────────────────────┐   │   │
      │   │   │  [PD 1] InGaAs Photodiode (940 nm Channel)  │   │   │
      │   │   │   ┌─────────────────────────────────────┐   │   │   │
      │   │   │   │   [LED 1]   [QCL LASER]   [LED 2]   │   │   │   │
      │   │   │   │   940 nm      Mid-IR      1050 nm   │   │   │   │
      │   │   │   │  (VCSEL)     (Pulsed)     (VCSEL)   │   │   │   │
      │   │   │   └─────────────────────────────────────┘   │   │   │
      │   │   │  [PD 2] InGaAs Photodiode (1050 nm Channel) │   │   │
      │   │   │  [PZT]  Piezoelectric Acoustic Ring Sensor │   │   │
      │   │   └─────────────────────────────────────────────┘   │   │
      │   │  (E1)  INNER BIOIMPEDANCE VOLTAGE SENSE (-)         │   │
      │   └─────────────────────────────────────────────────────┘   │
      │  (E3)  OUTER BIOIMPEDANCE CURRENT DRIVE ELECTRODE (-)       │
      └─────────────────────────────────────────────────────────────┘
      ◄────────────────────── Total Diameter: 18 mm ────────────────►
```

### Critical Mechanical Features:
1. **Central Optical Cavity (Sapphire Window):** Houses the 940 nm LED, 1050 nm LED, and Mid-IR laser. Protected by a scratch-resistant 0.3 mm sapphire glass window.
2. **Optical Crosstalk Barrier (Baffle):** A 1.0 mm high black neoprene/silicone rubber ring surrounds the emitters. This blocks light from shining horizontally directly from the LEDs into the photodiodes. **Photons are forced to travel through the dermis before reaching the detectors.**
3. **Acoustic Matching Pad:** A 0.2 mm medical-grade silicone pad bridges the skin surface to the PZT piezoelectric transducer, matching acoustic impedance ($Z \approx 1.5\text{ MRayl}$) to prevent ultrasound reflection at the skin boundary.
4. **Concentric Gold Electrodes:** Immersion gold (ENIG) over copper rings provide galvanic skin contact without skin irritation or chemical gel requirements.

---

## 3. Bill of Materials (BOM) — Off-the-Shelf Sensor Components

Every component on the sensor probe assembly can be purchased off-the-shelf and assembled onto a standard 4-layer polyimide flex PCB:

| Subsystem | Component Description | Manufacturer & Part Number | Package / Footprint | Key Specifications | Connection to GLUEDIA-NI ASIC |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Optical Emitter 1** | 940 nm NIR VCSEL LED | Vishay VSMY2940G / Osram SFH 4715AS | 0805 SMD ($2.0 \times 1.25\text{ mm}$) | $\lambda = 940\text{ nm}$, $I_f = 20\text{ mA}$, $15\text{ mW/sr}$ | Driven by low-side NMOS from **Pad 12 (`led_940_en`)** |
| **Optical Emitter 2** | 1050 nm Glucose LED | Marktech MTE1050W-C / Thorlabs LED1050E | 0805 SMD ($2.0 \times 1.25\text{ mm}$) | $\lambda = 1050\text{ nm}$, $I_f = 20\text{ mA}$, C-H overtone | Driven by low-side NMOS from **Pad 13 (`led_1050_en`)** |
| **Optical Detectors** | Dual InGaAs Photodiodes | Hamamatsu S1223-01 / Osram BPW 34 S | SMD Ceramic ($1.0\text{ mm}^2$ active) | Responsivity $\mathcal{R} = 0.85\text{ A/W}$ @ 940–1050nm | Wired to **Pad 1 (`IN_OPT1_PD`)** & **Pad 2 (`IN_OPT2_PD`)** |
| **Acoustic Laser** | Pulsed Mid-IR Laser | Hamamatsu L12004 / Alpes Lasers QCL | Micro-DFB TO-Can ($5.6\text{ mm}$) | $\lambda = 8.5\text{–}10.5\ \mu\text{m}$, 50 ns pulse @ 1 kHz | Triggered by **Pad 14 (`pa_ref_clk`)** |
| **Acoustic Transducer**| 200 kHz PZT Piezo Sensor | TE Connectivity 28 µm PVDF / PZT Disc | Annular Disc ($8.0\text{ mm}$ OD, $5.5\text{ mm}$ ID) | $f_0 = 200\text{ kHz}$, $50\ \mu\text{V}$ response | Wired to **Pad 3 (`IN_PA_PIEZO`)** ➔ Folded CSA |
| **BioZ Electrodes** | 4-Point Galvanic Array | Custom ENIG Gold on Flex PCB | 4 Concentric Rings ($0.5\text{ mm}$ width) | Dry skin contact, 10 kHz–1 MHz swept | Outer rings: Drive | Inner: **Pad 4 & 5 (`IN_BIOZ_P/N`)** |
| **Skin Temperature** | Digital Temp Sensor | Texas Instruments TMP117 | WLCSP-6 ($1.5 \times 1.0\text{ mm}$) | $\pm 0.1^\circ\text{C}$ clinical medical accuracy | Wired to **Pad 18 (`temp_raw`)** via internal ADC |
| **Motion Sensor** | 3-Axis Accelerometer | STMicroelectronics LIS2DH12 | LGA-12 ($2.0 \times 2.0\text{ mm}$) | Ultra-low-power ($2\ \mu\text{A}$), motion gate | Wired to **Pad 19 (`accel_mag`)** via I²C bus |
| **LED Drivers** | Dual N-Channel MOSFETs | Diodes Inc. DMN2075U / BSS138 | SOT-23 / SOT-363 | $V_{DS} = 20\text{ V}$, $R_{DS(\text{on})} = 0.07\ \Omega$ | Switched by ASIC sequencer pins |
| **Substrate Board** | Flex PCB Substrate | OSH Park / PCBWay 4-Layer Flex | 18 mm Circular Disk, 0.2 mm Polyimide | Matte black solder mask (reduces stray light) | Terminates in 10-pin 0.5 mm ZIF FPC connector |

---

## 4. 10-Pin Flex Ribbon Pinout & Interface to ASIC

The sensor probe connects to the main GLUEDIA-NI ASIC board through a standard **10-pin 0.5 mm pitch Flexible Printed Circuit (FPC) connector**:

```
 10-PIN FPC CONNECTOR                      GLUEDIA-NI ASIC (0.80 × 0.48 mm DIE)
┌──────────────────────┐                  ┌────────────────────────────────────────┐
│ Pin 1:  PD_940_CATH  │ ───────────────► │ Pad 1:  IN_OPT1_PD  (TIA 1 Summing In) │
│ Pin 2:  PD_1050_CATH │ ───────────────► │ Pad 2:  IN_OPT2_PD  (TIA 2 Summing In) │
│ Pin 3:  PZT_PIEZO_SIG│ ───────────────► │ Pad 3:  IN_PA_PIEZO (Acoustic CSA In)  │
│ Pin 4:  BIOZ_SENSE_P │ ───────────────► │ Pad 4:  IN_BIOZ_P   (BioZ INA In +)    │
│ Pin 5:  BIOZ_SENSE_N │ ───────────────► │ Pad 5:  IN_BIOZ_N   (BioZ INA In -)    │
│ Pin 6:  LED_940_GATE │ ◄─────────────── │ Pad 12: led_940_en  (TDM Phase 1 Trig) │
│ Pin 7:  LED_1050_GATE│ ◄─────────────── │ Pad 13: led_1050_en (TDM Phase 2 Trig) │
│ Pin 8:  LASER_TRIG   │ ◄─────────────── │ Pad 14: pa_ref_clk  (Laser Pulse Trig) │
│ Pin 9:  TEMP_SDA_SCL │ ◄──────────────► │ Pad 18: temp_raw    (Temperature Bus)  │
│ Pin 10: GND_SHIELD   │ ──────────────── │ Pad 32: VSS         (Analog Ground)    │
└──────────────────────┘                  └────────────────────────────────────────┘
```

---

## 5. Visual Assembly Diagram

A full-color, high-resolution diagram of this physical sensor probe assembly has been generated at:
> 🖼️ **[assets/gluedia_ni_sensor_probe_assembly.png](assets/gluedia_ni_sensor_probe_assembly.png)** *(4400 × 2600 px, 200 DPI)*

It illustrates:
1. The **concentric probe head layout** with exact ring diameters.
2. The **tissue cross-section** showing how the light and ultrasound waves travel through the dermis.
3. The **electrical schematic** connecting each sensor component through the 10-pin ribbon cable to the GLUEDIA-NI ASIC pads.
4. The **complete Bill of Materials table** with part numbers and ordering specifications.
