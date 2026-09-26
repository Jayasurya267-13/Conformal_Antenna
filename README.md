# Conformal_Antenna

# Helmet-Mounted Conformal Antenna for Tactical Communications

Low-profile, helmet-integrated conformal antenna system for reliable wearable RF
communication in dense urban (CQB) environments — designed, calculated, and
simulated end-to-end, from RF theory through a working CST Studio Suite model.

> Smart India Hackathon 2026 Project

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [Our Solution](#our-solution)
- [System Architecture](#system-architecture)
- [Antenna Design Specifications](#antenna-design-specifications)
- [Why Two Antennas — Diversity](#why-two-antennas--diversity)
- [Working Procedure](#working-procedure)
- [Repository Structure](#repository-structure)
- [Getting Started with the CST Model](#getting-started-with-the-cst-model)
- [Current Status](#current-status)
- [Key Parameters Evaluated](#key-parameters-evaluated)
- [Roadmap](#roadmap)
- [Potential Users](#potential-users)
- [Team](#team)

---

## Problem Statement

In dense urban environments, radio communication becomes unreliable because
buildings, concrete walls, vehicles, and other obstacles attenuate, reflect, and
scatter RF signals — producing unstable received signal strength and multipath
fading. Conventional external/whip antennas compound this: they protrude from
equipment, snag on obstacles, and are impractical for compact wearable
integration.

**This project proposes a helmet-integrated conformal antenna system, engineered
and validated for reliable wearable RF communication in these conditions.**

---

## Our Solution

A **433.5 MHz helmet-mounted conformal antenna system** using two low-profile
meandered PIFA (Planar Inverted-F Antenna) elements mounted on the rear-left and
rear-right of the helmet, instead of a single protruding external antenna.

- **Conformal design** — antenna follows the helmet's curved surface
- **Dual-antenna diversity** — spatially separated RF paths, so one obstructed
  side doesn't kill the link
- **Adaptive antenna selection** — driven by real-time RSSI, SNR, and
  packet-success monitoring
- **ESP32 + LoRa** transceiver chain for the digital communication link

---

## System Architecture

```
USER (text/data)
       │
       ▼
     ESP32  ──── packetize, control
       │
       ▼
 LoRa Transceiver ──── 433.5 MHz modulation
       │
       ▼
 Matching Network ──── 50 Ω impedance match
       │
       ▼
┌─────────────────────────────┐
│  Helmet Antenna Diversity    │
│  Antenna A        Antenna B  │
│  (Rear-Left)     (Rear-Right)│
└─────────────────────────────┘
       │
       ▼
  Urban RF Channel ──── reflection, diffraction, multipath
       │
       ▼
  Receiver Stack ──── LoRa + ESP32, reversed
       │
       ▼
  RSSI / SNR / Packet-Success Monitoring
       │
       ▼
    Message Display
```

The antenna itself does not generate data — the LoRa transceiver converts
digital information into a 433.5 MHz modulated RF signal, and the dual conformal
antennas radiate and receive that energy. The ESP32 continuously monitors link
quality on both antennas and selects whichever is delivering the more reliable
connection.

---

## Antenna Design Specifications

| Parameter | Specification |
|---|---|
| Design frequency | ≈ 433.5 MHz |
| Antenna type | Meandered conformal microstrip patch / PIFA |
| Feed impedance | 50 Ω |
| Substrate | Flexible PCB — Kapton / PET / FR-4 (early bench testing) |
| Radiator | Copper foil / PCB copper layer |
| Ground plane | Copper layer behind the substrate |
| Target S11 | < −10 dB near 433 MHz |
| Target VSWR | < 2:1 |
| Evaluation | S11, bandwidth, gain, radiation pattern, RSSI, SNR, packet-success rate |

### Design Calculations (Summary)

| Quantity | Formula | Value |
|---|---|---|
| Free-space wavelength (λ₀) | c / f₀ | 692.1 mm |
| Quarter-wave electrical length | λ₀ / 4 | 173.0 mm |
| Physical conductor length (εeff ≈ 1.3) | L_elec / √εeff | ≈ 151.7 mm |

Full derivation (effective dielectric constant, PIFA/meander length equations,
feed-position/input-resistance relationship, S11–Γ–VSWR relationships, and
L-network matching formulas) is documented in
[`docs/Antenna_Design_Calculation_Note_433MHz.md`](docs/Antenna_Design_Calculation_Note_433MHz.md).

---

## Why Two Antennas — Diversity

A single antenna facing an obstacle loses link quality with no fallback. Two
conformal antennas — rear-left and rear-right — give the helmet spatially
separated RF paths:

1. **Obstacle blocks one path** — a wall, turn, or body orientation weakens RF
   on one side of the helmet.
2. **Both links are monitored** — the ESP32 reads RSSI, SNR, and packet-success
   rate from Antenna A and Antenna B continuously.
3. **Better link is selected** — the transceiver switches to whichever antenna
   currently gives the more reliable connection.

---

## Working Procedure

1. User enters a test message, e.g. `"HELLO SIH"`
2. ESP32 packages the digital data
3. LoRa transceiver modulates to 433.5 MHz
4. Signal passes through the 50 Ω matching network
5. Active conformal antenna (A or B) radiates it
6. Signal propagates through the urban test environment
7. Receiving antenna captures the signal
8. LoRa receiver demodulates & decodes
9. ESP32 reconstructs the packet
10. Receiver displays `"HELLO SIH"`

> **Current scope:** digital message transmission over LoRa. A microphone/audio
> path for real-time voice is a planned future extension, not yet implemented
> or tested on this prototype.

---

## Repository Structure

```
.
├── README.md
├── docs/
│   └── Antenna_Design_Calculation_Note_433MHz.md   # full RF calculation note
├── cst/
│   ├── Build_PIFA_433MHz_v5.bas                    # current CST build macro
│   └── build_433.5.zip                             # saved CST project
├── firmware/                                        # ESP32 / LoRa firmware (TBD)
└── hardware/                                         # PCB / mechanical files (TBD)
```

---

## Getting Started with the CST Model

The antenna geometry is fully parametrized and auto-built via a CST Studio
Suite macro, so it can be reproduced or re-tuned without manual redrawing.

1. Open CST Studio Suite 2025 (new blank project, or the provided project)
2. **Macros → Macro Editor** → paste in `cst/Build_PIFA_433MHz_v5.bas` → **Run (F5)**
3. **Simulation → Mesh → Global Properties** → set Cells per wavelength = 20–30
4. **Simulation → Start Simulation** (Time Domain solver, 300–600 MHz range is
   pre-configured)
5. Check **1D Results → S-Parameters (S1,1)** and **VSWR**

All key dimensions (`segLen`, `feedX`, `feedY`, `shortX`, `shortY`, substrate
properties, etc.) are exposed as named parameters in **Modeling → Parameter
List**, so re-tuning is a value edit + re-run, not a geometry rebuild.

---

## Current Status

| Milestone | Status |
|---|---|
| Design calculations (wavelength, PIFA length, feed position) | ✅ Done |
| Parametrized CST model built | ✅ Done |
| Resonant frequency tuned to target | ✅ 433.85 MHz achieved (target 433.5 MHz) |
| Impedance match (VSWR < 2:1) | 🔄 In progress — iterating feed position from measured Z-parameter data |
| Physical fabrication | ⏳ Not yet started |
| VNA measurement & validation | ⏳ Pending fabrication |
| Helmet curvature + body-loading model | ⏳ Pending |
| Dual-antenna diversity firmware (ESP32 RSSI/SNR selection) | ⏳ Pending |

This project is at the **simulation and RF tuning stage**. Numbers above reflect
CST simulation results, not yet VNA-measured hardware.

---

## Key Parameters Evaluated

**Antenna parameters:** resonant frequency, S11/return loss, VSWR, bandwidth,
radiation pattern, gain & efficiency

**Communication parameters:** RSSI (per antenna), SNR (per antenna),
packet-success rate, link stability, effect of orientation & obstacles,
adaptive antenna-selection accuracy

---

## Roadmap

- [ ] Close impedance match to VSWR < 2:1 (feed-position fine-tuning)
- [ ] Add helmet curvature and representative body-loading model in CST, re-tune
- [ ] Fabricate prototype (flexible substrate) with a trim-tab margin
- [ ] VNA characterization, compare to simulation
- [ ] Microphone / audio path for voice communication
- [ ] Ruggedization and field testing
- [ ] Multi-band / MIMO antenna designs
- [ ] Security and encryption for the communication link
- [ ] Miniaturization and power optimization

---

## Potential Users

Defence personnel · Police & emergency-response teams · Firefighters ·
Search-and-rescue teams · Disaster-response personnel · Industrial safety teams

---

## Team

_Murali Shri rengan R
 Ajey adith A
 Deepan S K
 Abirami S
 Sairam J ._

---

