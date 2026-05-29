# Retro Moon-Graph Watch (Project ChronoLuna)

An ultra-low-power, wrist-wearable modern tribute to the legendary 1980s **Casio Moon Graph (GMW-15)**. This repository houses the entire open-source journey of designing, simulating, coding, and building a physical digital watch from scratch—featuring high-accuracy astronomical calculations, a modern Memory-in-Pixel (MiP) display, custom multi-layer PCB layout, and micro-amp power optimization.

---

## 📌 Project Architecture Overview

Project ChronoLuna is designed around three pillars: absolute deterministic execution, maximum power conservation, and faithful visual recreation of historical LCD iconography on modern high-contrast digital paper displays.

```text
       +-------------------------------------------------------+
       |                  ChronoLuna Firmware                  |
       +-------------------------------------------------------+
                                   |
         +-------------------------+-------------------------+
         |                         |                         |
         v                         v                         v
+-----------------+      +-------------------+      +-----------------+
|  Timekeeping &  |      | Moon Phase Engine |      | UI State Machine|
|  Internal RTC   |      |   (Synodic Mod)   |      |  (Menu & Graph) |
+-----------------+      +-------------------+      +-----------------+
         |                         |                         |
         +-------------------------+-------------------------+
                                   |
                                   v
                      +--------------------------+
                      | Hardware Abstraction /   |
                      | Sharp Memory LCD (SPI)   |
                      +--------------------------+

```

### Key Technical Specifications (Target)

* **MCU Core:** STM32L4 series (ARM Cortex-M4 ultra-low-power microcontrollers).
* **Display:** Sharp Memory-in-Pixel (MiP) LCD via low-overhead SPI (Ultra-low static power consumption ~15 µW).
* **Power Source:** CR2032 or CR2016 Lithium Coin Cell targeting a minimum 1.5 to 2-year lifespan.
* **Firmware Stack:** Bare-metal C/C++ leveraging strict low-power sleep state management (`STOP` and `STANDBY` modes with RTC wakeups).

---

## 🗺️ Productization Roadmap: Step-by-Step Execution Plan

To ensure a robust engineering approach, the project transitions strictly from pure algorithmic verification to UI asset mapping, followed by hardware prototyping, custom hardware design, and finally, mechanical housing execution.

### Phase 1: Software Logic & Algorithmic Simulation 🔄 (IN PROGRESS)

The foundation of the watch lies in precise timekeeping, modular menu flows, and accurate astronomical tracking without relying on external network sync (floating-point precision optimization on a microcontroller).

* [ ] **1.1 Chrono-Engine Implementation:**
* Build a deterministic internal time calculation structure managing seconds, minutes, hours, days, months, years, and leap-year rules.
* Implement an epoch-offset calculator from a fixed historical timestamp.


* [ ] **1.2 High-Fidelity Synodic Moon Phase Algorithm:**
* Define the exact Synodic month cycle (29.530588853 days).
* Establish a precise New Moon reference epoch (e.g., January 6, 2000, at 18:14 UTC).
* Design a calculation engine to compute the current "Moon Age" in days, mapping to 8 primary and 32 sub-phases.


* [ ] **1.3 Hierarchical Menu State Machine:**
* Model a finite state machine (FSM) in pure C/C++ handling asynchronous button interrupts.
* Define states: `TIME_DISPLAY`, `MOON_DATA_VIEW`, `CHRONOGRAPH`, `ALARM_SETTING`, and `TIME_ADJUST`.



### Phase 2: UI Map & Bit-Display Layout 🎨

Recreating the signature multi-window LCD layout of the Casio GMW-15 using a 1-bit modern pixel layout.

* [ ] **2.1 Visual Screen Segmentation:**
* Define the coordinate mapping for the Top Horizon Curve (graphic display representing moon trajectories/paths).
* Allocate sub-buffers for the circular Moon Matrix.
* Lay out alphanumeric segments for the lower time and calendar matrix.


* [ ] **2.2 Asset Generation & Bitmaps:**
* Construct custom 1-bit monochrome pixel array fonts replicating the iconic Casio segment typography.
* Generate the array matrices for all visual steps of the moon illumination cycle to ensure fluid movement across the screen.



### Phase 3: Hardware Prototyping & Desktop Testing 🔬

Validating firmware on physical development hardware before spinning custom silicon.

* [ ] **3.1 Breadboard Integration:**
* Source an STM32L4 Nucleo/Discovery development board.
* Wire up a Sharp Memory LCD breakout board over high-speed hardware SPI.
* Connect debounced hardware pushbuttons via external GPIO interrupt lines.


* [ ] **3.2 Power Profiling & Optimization:**
* Implement active current measurement using specialized power monitoring equipment.
* Optimize firmware loops to maximize the time spent in `Stop` mode.



### Phase 4: Custom PCB Design (EDA Layout) ⚡

Condensing the entire development platform onto a compact, multi-layer circular printed circuit board that fits perfectly on a human wrist.

* [ ] **4.1 Schematic Capture:**
* Design ultra-low quiescent current power regulation circuits.
* Embed high-accuracy external 32.768 kHz tuning-fork crystals.
* Integrate mechanical tactile switch interfaces with ESD protection networks.


* [ ] **4.2 PCB Layout & Routing:**
* Optimize layout for a circular form factor targeting a maximum diameter of 34-36 mm.
* Implement 0402/0603 size Surface Mount Devices (SMD) for maximum spatial efficiency.
* Incorporate a low-profile coin cell retention clip on the bottom layer.



### Phase 5: Mechanical Enclosure & Productization ⌚

Turning raw electronics into a durable, highly stylized, daily wearable object.

* [ ] **5.1 3D CAD Modeling:**
* Model the watch case shell using parametric CAD tools (e.g., Fusion 360).
* Design dedicated internal support structures to safely seat the display panel, PCB assembly, and custom button plungers.


* [ ] **5.2 Prototyping & Final Assembly:**
* Fabricate functional case mockups via high-resolution SLA/Resin 3D printing.
* Prototype waterproof/dustproof seal channels using custom elastomer O-rings.
* Mount standard 18mm or 20mm watch straps for field testing.



---

## 💻 Algorithmic Engine Preview

A conceptual code implementation of the Moon Phase Calculation written in standard embedded C/C++ compliance:

```cpp
#include <stdint.h>

#define SYNODIC_MONTH_DAYS 29.530588853f
#define SECONDS_PER_DAY    86400UL

// Epoch Reference: New Moon on Jan 6, 2000 18:14:00 UTC (947182440 Unix Timestamp)
#define NEW_MOON_EPOCH_S   947182440UL

typedef struct {
    float age_days;
    uint8_t phase_index; // 0 to 7 representing the core phases
} MoonState_t;

MoonState_t Calculate_Moon_Phase(uint32_t current_unix_timestamp) {
    MoonState_t state;
    
    if (current_unix_timestamp < NEW_MOON_EPOCH_S) {
        state.age_days = 0.0f;
        state.phase_index = 0;
        return state;
    }
    
    // Total seconds elapsed since the reference New Moon
    uint32_t delta_seconds = current_unix_timestamp - NEW_MOON_EPOCH_S;
    double delta_days = (double)delta_seconds / (double)SECONDS_PER_DAY;
    
    // Compute current position in the synodic cycle
    double remainder_days = delta_days - ((uint32_t)(delta_days / SYNODIC_MONTH_DAYS) * SYNODIC_MONTH_DAYS);
    state.age_days = (float)remainder_days;
    
    // Map the age to one of the 8 prominent segments
    state.phase_index = (uint8_t)(((remainder_days + (SYNODIC_MONTH_DAYS / 16.0)) / SYNODIC_MONTH_DAYS) * 8.0) % 8;
    
    return state;
}

```

---

## 🛠️ How to Contribute and Track

Progress is managed strictly through Github Issues and Projects. If you are experimenting with alternative low-power microcontrollers or have designed custom asset matrices for the 1-bit screen layout, feel free to open a detailed Pull Request.

*License: MIT Open Source Hardware & Software.*

```

```