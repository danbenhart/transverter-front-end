# transverter-front-end
![License](https://img.shields.io/badge/license-CERN--OHL--S%20v2-blue)
[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/danbenhart/transverter-front-end)
![Status](https://img.shields.io/badge/status-pre--fab-orange)
Project introduction

This project is a modular HF transmitter front-end and control system designed around a software-defined radio (SDR) with an external RF signal chain.
The system includes:

	band-selectable RF filtering (LPF/BPF)
    relay-based signal routing
    digitally controlled attenuation
    local oscillator (LO) control
    optional forward/reflected power measurement
    microcontroller-based control and user interface
    The design is intended to:
    support clean HF transmission (QRP ~10 W class)
    provide flexibility for experimentation and expansion
    maintain good RF layout and shielding practices
    separate RF, control, and power domains into modular boards

Project function

The system provides the following functionality:

    Band selection and filtering
        Relay-switched filter banks (QRP Labs BPF/LPF modules)
        Ensures proper harmonic suppression and band operation
    RF signal chain control
        Preamp + attenuator + PA drive control
        Fixed gain stages with digitally selectable attenuation
    LO control
        Voltage-controlled LO (DAC-driven)
        Optional frequency trim, defaults to using a 2.5V reference
    Relay control system
        ULN2803-based relay drivers
        Separate control and power domains
    Optional RF metering (future expansion)
        Forward and reflected power measurement via directional coupler + detectors
        Enables SWR calculation and system diagnostics
    System monitoring
        Input supply voltage monitoring (ADC)
        Reserved ADC channels for RF power monitoring
    User interface (planned/partial)
        OLED display via I2C (Qwiic)
        Potential for power, SWR, and system status display

Project Parameters

    Frequency range: HF bands (with ~100 MHz IF considerations)
    Output power: ~10 W (QRP class)
    Control MCU: Arduino Nano R4 (5V logic, built-in DAC + ADC)
    Supply rails:
        12 V (PA and primary power)
        5 V / 5.3 V (control and logic)
    Relay control:
        ULN2803 Darlington arrays
        Individual flyback diodes at relays
    Attenuation control:
        Digital step attenuator (SPI controlled)
    ADC usage:
        Forward power (reserved)
        Reflected power (reserved)
        Supply voltage monitoring
    Modular/Separate boards for:
        Control
        LO/frequency conversion
        power distribution
        Filtering
        Optional daughterboard expansion

Principle analysis (Hardware description)

The system is divided into several functional blocks:

    Power subsystem
        12 V input with protection (TVS, filtering)
        Local decoupling for PA:
            1 µF + 100 nF + planned bulk (≈47 µF)
        Separate 5 V / 5.3 V rails for logic and control
        Star-style distribution from bulk capacitor
    RF signal path
        Tx
            SDR → IF-BPF → Mixer → BPF → preamp → attenuator → PA → LPF → antenna
        Rx
            Antenna → LPF → BPF → preamp → Mixer → IF-BPF → SDR
        Filter modules use QRP Labs 2 RF + 2 GND header interface
        Emphasis on:
            short RF paths
            controlled impedance where practical
            minimal parasitic coupling
    Relay switching
        Controlled via ULN2803 arrays
        Individual relay coil decoupling (100 nF per relay)
        Flyback diodes located at relays (primary)
        ULN2803 COM optionally unused (external diodes preferred)
    Control and monitoring
        Arduino Nano R4:
            ADC for voltage and future RF power measurement
            DAC for LO control
            GPIO
                relay logic
                Sequencing
                Tx/Rx footswitch input
            USB for digital Tx/Rx switching and debug
            Qwiic connector for OLED and navigation buttons
    RF shielding and layout
        Ground plane continuity emphasized
        Optional PCB-based shielding partitions (grounded “dummy PCBs”)
        LO section enclosed in grounded shield can
        Care taken to:
            minimize loop area
            separate noisy digital/control from RF paths
    Expandability
        Dedicated headers for:
            filter/daughterboard integration
            RF power measurement modules
        ADC pin routing allows dual use:
            GPIO or analog measurement

Software code

TBD
Announcements

    RF layout optimized for HF performance, not microwave frequencies
    Emphasis placed on:
        grounding strategy
        shielding between functional blocks
        minimizing coupling between relay/control and RF paths
    Forward/reflected power measurement is designed but not populated
    ADC filtering implemented via:
        series resistor near detector
        capacitor near MCU
    ULN2803 unused inputs tied to GND; outputs left floating
    External flyback diodes used at relay level
    Board-to-board interconnect strategy includes:
        multiple ground pins
        optional IDC connector for additional grounding and signal routing

Assembling processes

TBD
Finished product display

TBD
Looking for feedback on:

    RF layout and grounding strategy
    Relay control + noise isolation
    Power distribution and decoupling
    Shielding approach (PCB partitions + LO can)
    Connector strategy between boards
    Any obvious risks before fabrication
