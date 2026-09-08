# Automatic Card Dealer PCB

<img width="540" height="540" alt="04c15ea1cc14ad1e33903ccc4fa5ff0a" src="https://github.com/user-attachments/assets/3cc5681c-dbcc-40f4-9250-92a4260bc8eb" />

## Overview

This project is a custom-designed printed circuit board (PCB) for an automatic card-dealing system. The PCB provides centralized control of the card-dealing mechanism using an **ESP32-S3** microcontroller, with dedicated motor drivers, user-interface components, card-detection inputs, and regulated power supplies.

The system is designed to coordinate two motor systems: a **DC motor driven by a DRV8833 motor driver** and a **5-wire stepper motor driven by a ULN2003 transistor array**. The PCB also provides an OLED display, rotary encoder interface, card-counting input, empty-card detection, and expansion GPIO for additional functionality.

The schematic is organized into four primary sections:

1. Power supply
2. ESP32-S3 controller and peripherals
3. ULN2003 stepper motor driver
4. DRV8833 DC motor driver

## Key Features

* ESP32-S3N8R2 main controller
* Battery-powered operation
* Integrated 5 V switching power supply
* DRV8833 dual H-bridge motor driver interface
* ULN2003-based 5-wire stepper motor interface
* I²C OLED display interface
* Rotary encoder with push-button input
* Card counter input
* Empty-card/deck sensor input
* Dedicated motor and peripheral connectors
* 10-pin expansion/reserve GPIO header
* On-board power indicator LED
* Bulk and high-frequency power filtering

## System Architecture

The overall hardware architecture is:

```text
                         Battery
                            │
                            ▼
                   ┌────────────────┐
                   │ Power Input &  │
                   │   Filtering    │
                   └───────┬────────┘
                           │
                       VDD_VIN
                           │
                           ▼
                   ┌────────────────┐
                   │   SY8120IABC   │
                   │  Buck Regulator│
                   └───────┬────────┘
                           │
                        VDD_5V
                           │
              ┌────────────┼─────────────┐
              │            │             │
              ▼            ▼             ▼
          ESP32-S3      ULN2003       DRV8833
              │            │             │
              │            ▼             ▼
              │         Stepper       DC Motor
              │          Motor
              │
      ┌───────┼───────────┬──────────────┐
      │       │           │              │
      ▼       ▼           ▼              ▼
    OLED   Encoder    Card Counter   Empty Sensor
```

The ESP32-S3 acts as the central controller and interfaces with the motor drivers, display, encoder, and card-detection circuitry.

## Hardware

### 1. Power Supply

The PCB accepts power through a dedicated battery connector and distributes the input as the `VDD_VIN` rail.

The power section includes bulk and bypass capacitors for supply filtering, as well as an LED power indicator. A **SY8120IABC switching regulator** generates the regulated `VDD_5V` supply used by the system.

Key components include:

* Battery connector
* SY8120IABC buck regulator
* 6.8 µH inductor
* 10 µF and 100 nF capacitors
* 220 µF bulk capacitor
* Power indicator LED
* Feedback resistors

The resulting power architecture provides the raw battery rail (`VDD_VIN`) and regulated 5 V rail (`VDD_5V`) required by the controller and peripheral circuitry.

### 2. ESP32-S3 Controller

The ESP32-S3N8R2 serves as the main microcontroller for the PCB.

It provides control signals for both motor systems and interfaces with the user-interface and sensing hardware.

The controller provides dedicated signals for:

* DRV8833 motor control
* ULN2003 stepper control
* OLED I²C communication
* Rotary encoder
* Card counter
* Empty-card sensor
* Expansion GPIO

The schematic provides multiple connectors around the ESP32-S3 to facilitate connection to the motors, sensors, display, encoder, and external circuitry.

### 3. DC Motor Control

A **DRV8833** motor driver provides the interface between the ESP32-S3 and an external DC motor.

The ESP32-S3 supplies:

* `DRV8833_AIN1`
* `DRV8833_AIN2`

to the driver, which produces:

* `DRV8833_OUT1`
* `DRV8833_OUT2`

for the external motor.

This provides a dedicated DC motor drive channel suitable for the motorized portion of the card-dealing mechanism.

A **470 µF capacitor** is included on the motor supply to provide bulk filtering for motor current transients.

### 4. Stepper Motor Control

The PCB includes a **ULN2003AN** transistor-array driver for a 5-wire stepper motor.

Four control signals from the ESP32-S3 are provided:

* `ULN2003_IN1`
* `ULN2003_IN2`
* `ULN2003_IN3`
* `ULN2003_IN4`

These signals drive the corresponding ULN2003 channels, which interface with the external stepper motor through a 5-pin connector.

The ESP32-S3 is therefore responsible for generating the appropriate stepping sequence required to operate the motor.

### 5. OLED Display

A dedicated four-pin connector is provided for an I²C OLED display.

The interface contains:

* `OLED_SDA`
* `OLED_SCL`
* `VCC`
* `GND`

The display communicates with the ESP32-S3 through the I²C interface.

The OLED can be used by the system firmware to provide status and user feedback. The specific information displayed is determined by the software implementation.

### 6. Rotary Encoder

An **EC11 rotary encoder** provides a physical user-input interface.

The PCB exposes:

* `ENCODER_A`
* `ENCODER_B`
* `ENCODER_SW`
* `ESP32_3.3V`
* `VDD_5V`

The A and B channels provide quadrature rotation information, while the encoder switch provides a push-button input. This allows the encoder to be used for menu navigation, parameter selection, or other user-interface functions implemented in firmware.

### 7. Card Counter and Empty Sensor

The PCB provides dedicated inputs for card detection.

The schematic identifies:

* `COUNTER_D0`
* `EMPTY_SENSOR`

These interfaces allow external sensing hardware to communicate card-related events and empty-deck status to the ESP32-S3.

The specific sensing mechanism and detection algorithm are determined by the external sensors and firmware.

### 8. Expansion Interface

A dedicated 10-pin reserve header is included on the PCB.

The available signals are:

* `RESERVE_1`
* `RESERVE_2`
* `RESERVE_3`
* `RESERVE_4`
* `RESERVE_5`
* `RESERVE_6`
* `RESERVE_7`
* `RESERVE_8`
* `RESERVE_9`
* `RESERVE_10`

This provides additional flexibility for future sensors, controls, or other peripherals without requiring a complete PCB redesign.

## Functional Summary

The PCB integrates the major electrical functions required by the automatic card dealer into a single control board.

### Control

The ESP32-S3 serves as the central processing unit and coordinates the connected peripherals and motor drivers.

### Motion

Two independent motor interfaces are provided:

* **DRV8833 → DC motor**
* **ULN2003 → 5-wire stepper motor**

This allows the system to combine continuous motor motion with stepper-based positioning.

### Sensing

The controller can receive information from:

* Card counter
* Empty-card sensor
* Rotary encoder

These inputs provide the hardware required for monitoring the mechanism and accepting user commands.

### User Interface

The combination of an OLED and rotary encoder provides a compact local user interface.

### Power

The battery input is regulated through an onboard switching converter to provide the required 5 V supply while maintaining dedicated power rails for the system electronics.

## Repository Contents

The project repository contains the PCB design and manufacturing files, including:

```text
card_dealer_pcb/
├── SCH_Schematic2.pdf
├── Gerber_PCB2.zip
├── poker.epro2
└── README.md
```

### `SCH_Schematic2.pdf`

Complete electrical schematic for the PCB, including the power supply, ESP32-S3 controller, ULN2003 stepper driver, and DRV8833 motor driver.

### `Gerber_PCB2.zip`

Manufacturing files generated for PCB fabrication.

### `poker.epro2`

EasyEDA project file containing the PCB design project.


## Notes

This repository documents the **hardware design** of the automatic card-dealing system. The schematic defines the available electrical interfaces and hardware functionality; the exact card-dealing sequence, motor-control algorithms, OLED interface behavior, and sensor-processing logic depend on the firmware and mechanical implementation.
