# transverter-front-end
![License](https://img.shields.io/badge/license-CERN--OHL--S%20v2-blue)
[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/danbenhart/transverter-front-end)
[![Type](https://img.shields.io/badge/type-HF%20RF%20hardware-green.svg)]()
![Status](https://img.shields.io/badge/status-pre--fab-orange)

## Project Introduction

This project is a modular HF transmitter front-end and control system designed around a software-defined radio (SDR) with an external RF signal chain.

The system includes:

- Band-selectable RF filtering (LPF/BPF)
- Relay-based signal routing
- Digitally controlled attenuation
- Local oscillator (LO) control
- Optional forward/reflected power measurement
- Microcontroller-based control and user interface

The design is intended to:

- Support clean HF transmission (QRP ~10 W class)
- Provide flexibility for experimentation and expansion
- Maintain good RF layout and shielding practices
- Separate RF, control, and power domains into modular boards

---

## Project Function

The system provides the following functionality:

- **Band selection and filtering**
  - Relay-switched filter banks (QRP Labs BPF/LPF modules)
  - Ensures proper harmonic suppression and band operation

- **RF signal chain control**
  - Preamp + attenuator + PA drive control
  - Fixed gain stages with digitally selectable attenuation

- **LO control**
  - Voltage-controlled LO (DAC-driven)
  - Optional frequency trim (defaults to 2.5V reference)

- **Relay control system**
  - ULN2803-based relay drivers
  - Separate control and power domains

- **Optional RF metering (future expansion)**
  - Forward and reflected power measurement via directional coupler + detectors
  - Enables SWR calculation and system diagnostics

- **System monitoring**
  - Input supply voltage monitoring (ADC)
  - Reserved ADC channels for RF power monitoring

- **User interface (planned/partial)**
  - OLED display via I2C (Qwiic)
  - Potential for power, SWR, and system status display

---

## Project Parameters

- **Frequency range:** HF bands (with ~100 MHz IF considerations)
- **Output power:** ~10 W (QRP class)
- **Control MCU:** Arduino Nano R4 (5V logic, built-in DAC + ADC)

- **Supply rails:**
  - 12 V (PA and primary power)
  - 5 V / 5.3 V (control and logic)

- **Relay control:**
  - ULN2803 Darlington arrays
  - Individual flyback diodes at relays

- **Attenuation control:**
  - Digital step attenuator (SPI controlled)

- **ADC usage:**
  - Forward power (reserved)
  - Reflected power (reserved)
  - Supply voltage monitoring

- **Modular boards:**
  - Control
  - LO / frequency conversion
  - Power distribution
  - Filtering
  - Optional daughterboard expansion

---

## Principle Analysis (Hardware Description)

The system is divided into several functional blocks:

### Power Subsystem
- 12 V input with protection (TVS, filtering)
- Local decoupling for PA:
  - 1 µF + 100 nF + planned bulk (~47 µF)
- Separate 5 V / 5.3 V rails for logic and control
- Star-style distribution from bulk capacitor

### RF Signal Path

- **Tx path:**
  - SDR → IF-BPF → Mixer → BPF → Preamp → Attenuator → PA → LPF → Antenna

- **Rx path:**
  - Antenna → LPF → BPF → Preamp → Mixer → IF-BPF → SDR

- Filter modules use QRP Labs 2 RF + 2 GND header interface

- Design emphasis:
  - Short RF paths
  - Controlled impedance (where practical)
  - Minimal parasitic coupling

### Relay Switching
- Controlled via ULN2803 arrays
- Individual relay coil decoupling (100 nF per relay)
- Flyback diodes located at relays (primary)
- ULN2803 COM optionally unused (external diodes preferred)

### Control and Monitoring
- **Arduino Nano R4:**
  - ADC for voltage and future RF power measurement
  - DAC for LO control
  - GPIO:
    - Relay logic
    - Sequencing
    - TX/RX footswitch input
  - USB for digital TX/RX switching and debugging
  - Qwiic connector for OLED and navigation buttons

### RF Shielding and Layout
- Ground plane continuity emphasized
- Optional PCB-based shielding partitions (grounded “dummy PCBs”)
- LO section enclosed in grounded shield can

- Design considerations:
  - Minimize loop area
  - Separate noisy digital/control from RF paths

### Expandability
- Dedicated headers for:
  - Filter/daughterboard integration
  - RF power measurement modules

- ADC pin routing allows dual use:
  - GPIO or analog measurement
---
## Design Tradeoffs / Component Selection Notes

Several components in this design were intentionally chosen despite higher cost or complexity. The goal was to prioritize RF performance, modularity, and reliability over minimum BOM cost.

### Oscillator (VCOCXO @ ~100 MHz)

- **Choice:** Voltage-controlled crystal oscillator (VCOCXO) with sine wave output

- **Key parameter choices:**

  - **~100 MHz frequency**
    - Chosen to be compatible with a wide range of SDR platforms
    - Falls within a commonly supported tuning range for many SDRs and transverters
    - Allows flexibility in IF planning and future system integration

  - **Sine wave output**
    - Reduces harmonic content compared to CMOS/square-wave oscillators
    - Minimizes broadband noise from fast edges
    - Simplifies downstream filtering requirements
    - Improves spectral cleanliness of the LO signal

  - **Voltage control (VCXO behavior)**
    - Allows fine frequency trimming via DAC
    - Enables compensation for:
      - component tolerances
      - temperature drift (within limits)
      - system-level frequency alignment
    - Provides flexibility without requiring a full PLL-based solution

- **Reasoning:**
  - Prioritizes **spectral purity and RF cleanliness** over simplicity
  - Avoids the need for aggressive filtering typically required with CMOS oscillators
  - Integrates well with the DAC available on the control MCU

- **Tradeoffs:**
  - Higher cost and fewer available options compared to TCXO modules
  - Larger footprint
  - Slightly more complex bias/control requirements


### Bandpass Filter (Mini-Circuits BPF-A120+)

- **Choice:** Mini-Circuits BPF-A120+
- **Reasoning:**
  - Provides a **well-characterized, stable IF filter stage**
  - Eliminates the need to design and tune a custom filter
  - Predictable insertion loss and rejection performance
  - Reduces development risk in the IF chain

- **Tradeoff:**
  - Higher cost compared to discrete LC filters
  - Less flexibility than a custom-designed filter


### Relay Selection

- **Choice:** General-purpose signal relays with external flyback diodes
- **Reasoning:**
  - Reliable and well-understood switching behavior
  - Good isolation between signal paths
  - Easy to integrate with ULN2803 drivers
  - Supports modular filter bank switching

- **Tradeoff:**
  - Larger physical size compared to solid-state switching options
  - Mechanical wear over time
  - Slower switching speeds (not critical for this application)


### Power Amplifier (QRP Labs 10W PA)

- **Choice:** QRP Labs 10W PA
- **Reasoning:**
  - Proven, well-documented design
  - Eliminates the need to design a PA from scratch
  - Known performance characteristics
  - Good integration with existing QRP Labs ecosystem

- **Tradeoff:**
  - Less flexibility than a custom PA design
  - Requires integration with external filtering and control


### Filter Modules (QRP Labs LPF/BPF Boards)

- **Choice:** QRP Labs plug-in filter modules
- **Reasoning:**
  - Modular, interchangeable design
  - Simplifies band-specific filtering
  - Avoids tuning and validation of multiple discrete filters
  - Standardized interface (2 RF + 2 GND)

- **Tradeoff:**
  - Connector-based RF interface introduces some parasitics
  - Larger footprint compared to fully integrated designs

---

### Overall Design Philosophy

This project prioritizes:

- RF performance over minimum cost
- Modular construction over maximum integration
- Predictable, known-good building blocks over fully custom circuits

Where possible, commercially available RF components were used to:
- reduce development time
- minimize tuning complexity
- improve repeatability

The result is a system intended to be:
- easier to debug
- easier to modify and expand
- more robust in practical use

---

## Software Code

TBD

---

## Announcements / Design Notes

- RF layout optimized for HF performance (not microwave frequencies)

- Emphasis on:
  - Grounding strategy
  - Shielding between functional blocks
  - Minimizing coupling between relay/control and RF paths

- Forward/reflected power measurement is designed but not populated

- ADC filtering implemented via:
  - Series resistor near detector
  - Capacitor near MCU

- ULN2803 unused inputs tied to GND; outputs left floating
- External flyback diodes used at relay level

- Board-to-board interconnect strategy includes:
  - Multiple ground pins
  - Optional IDC connector for additional grounding and signal routing

---

## Assembling Process

TBD

---

## Finished Product Display

TBD

---

## Looking for Feedback On

- RF layout and grounding strategy
- Relay control and noise isolation
- Power distribution and decoupling
- Shielding approach (PCB partitions + LO can)
- Connector strategy between boards
- Any obvious risks before fabrication
