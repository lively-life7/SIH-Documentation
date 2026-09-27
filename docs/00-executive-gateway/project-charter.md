# Project Charter: AEGIS Platform
**AI-Enabled Mine Subsidence Monitoring & Early Warning Platform**  
**Problem Statement ID:** SIH26025 | **Ministry of Coal, Government of India**

---

## 1. Executive Summary

Underground coal mining extraction inevitably induces subsidence—the settling, sinkage, or cracking of overburden rock and surface terrain. In India, roof and ground falls account for approximately 63% of underground mine fatalities according to Directorate General of Mines Safety (DGMS) records. Concurrently, subsidence threatens key national infrastructure (railway lines, highways, aquifers, and overlying settlements), causing hundreds of crores in structural damage and disrupting operations.

**AEGIS** is an indigenous, real-time, physics-informed IoT early warning system designed to detect subsurface deformation and predict surface crack formation **up to 8.9 days in advance**, while triggering automated sirens in **< 1.4 seconds**. 

---

## 2. Problem Context & Existing Gaps

Current monitoring approaches in Indian coalfields suffer from fatal trade-offs:
1. **Manual Theodolite / Total Station Surveys:** Performed intermittently (every 15–30 days). By the time ground strain manifests visibly to survey crews, catastrophic failure is often already underway.
2. **Imported Geotechnical Loggers:** Prohibitively expensive (₹2–5 Lakhs per sensor unit, reaching ₹40–60 Lakhs per panel). Because of high capital cost, mines deploy sparse grids (100–300m spacing) that completely miss localized 2m shear fissures.
3. **Satellite InSAR:** Subject to orbital repeat cycles (6–12 days) and severe tropical cloud cover attenuation during monsoon months, rendering it incapable of providing tactical, shift-by-shift safety alerts.

---

## 3. Core Architectural Principles & Clarifications

### Dynamic Panel Sizing vs. Static Hardware Caps
> [!IMPORTANT]
> **Dynamic Scaling Architecture:**
> In real-world underground coal mining, **a static, fixed hardware count is technically incorrect**. Coal panels differ substantially based on extraction methods (Bord-and-Pillar depillaring vs. Longwall faces), seam depth ($H \in [50\text{m}, 400\text{m}]$), panel length ($L \in [200\text{m}, 1500\text{m}]$), and panel width ($W \in [100\text{m}, 300\text{m}]$).

AEGIS employs a **physics-derived dynamic sizing model**:
* **Influence Radius Calculation:** $r = \frac{H}{\tan(\beta)}$, where $\beta$ is the major angle of draw.
* **Nyquist Grid Spacing:** To resolve localized fissures before tensile crack initiation, spatial sampling must satisfy:
  $$\Delta \le \frac{r}{2.86} \approx 15\text{--}25\text{m}$$
* **Dynamic Node Sizing Formulation:**
  $$N_{\text{nodes}} \approx \left(\frac{L_{\text{panel}} + 2r}{\Delta}\right) \times \left(\frac{W_{\text{panel}} + 2r}{\Delta}\right) \times \rho_{\text{criticality}} + N_{\text{anchors}}$$
  where $\rho_{\text{criticality}}$ increases density over sensitive surface infrastructure (railways, pipelines, villages) and $N_{\text{anchors}}$ includes bedrock reference and borehole anchors.
* **Modular Cost Scaling:** Rather than an arbitrary fixed package cost, the Scout Node BOM is maintained at an ultra-low unit cost of **~₹1,050/node** using indigenous Commercial Off-The-Shelf (COTS) parts. The total panel investment scales strictly as a modular function of panel geometry and required density, remaining ~90% cheaper than imported alternatives.

---

## 4. Key Performance Targets

| Metric | Target | Rationale & Mechanism |
| :--- | :--- | :--- |
| **Scout Node BOM** | ~₹1,050 / node | 100% indigenous COTS parts (ESP32, MPU-6050, ADS1115, LoRa SX1262) |
| **Advance Crack Prediction** | 8.9 days | Derived from critical tensile strain threshold ($\theta_c = 1500\,\mu\varepsilon$) on Knothe time curve |
| **End-to-End Siren Latency** | < 1.4 seconds | Edge gateway hardware interrupt to high-decibel siren trigger |
| **Zero Data Loss Buffer** | 72 hours | On-node SPI flash store-and-forward ring buffer (4,320 epochs) |
| **False Alarm Rate** | Near-zero | 5-node Byzantine quorum gating ($\ge 3\sigma$) + 4-band FFT vibration discriminator |
| **Spectrum Compliance** | License-free | GSR 564(E) IN865 band (865–867 MHz), 125 kHz BW, $\le 1\%$ duty cycle |

---

## 5. Inviolable System Boundaries

1. **Safety Firewall (C8 vs. C9):**
   * **C9 PINN (Physics-Informed Neural Network):** Responsible **only** for 3D continuous surface reconstruction, spatial interpolation, and digital twin rendering. C9 has **zero alarm authority**.
   * **C8 Alarm Engine:** Exclusively owns threshold checking, Byzantine quorum gating, and emergency siren/SMS actuation using deterministic, auditable rules.
   * **Hard Firewall:** C9 output is never fed into C8 logic.
2. **Ground Truth Quarantine:**
   * Synthetic simulation datasets (`truth/`) have no import path from backend production modules (`backend/`), strictly verified by test T8.
3. **Edge Vibration Discrimination:**
   * 200 Hz on-node FFT filters non-hazardous vibrations (trucks at 8–20 Hz, conveyors at 50 Hz, blasting at 40–80 Hz checked against DGMS blast registers) while escalating genuine rock fracture frequencies (100–250 Hz).

---

## 6. Handbook Roadmap & Verification

This project documentation is organized modularly according to the **Format B Technical Handbook Blueprint**:
* **Module 00 — Executive Gateway:** Charter, system architecture, key scorecard.
* **Module 01 — Ground Reality:** Crisis landscape, failure of existing techniques, opportunity statement.
* **Module 02 — Sensor Hardware:** Node tiers, 7-sensor suite, BOM breakdown, power budget.
* **Module 03 — Mesh Networking:** LoRa TDMA superframe, 23-byte wire format, multi-frequency DAG.
* **Module 04 — Physics Engine:** Knothe subsidence formulations, strain derivatives, 6-stage corruption.
* **Module 05 — AI/ML Pipeline:** PINN architecture, loss formulations, training contract, safety firewall.
* **Module 06 — Backend Pipeline:** Ingestion schema, C7 8-step cleaner, C8 alarm engine.
* **Module 07 — Digital Twin & UI:** CesiumJS 3D visualization, operator dashboard, siren workflows.
* **Module 08 — Verification:** T1–T46 test register, 6-day build order, field validation plan.
* **Module 09 — Deployment & Impact:** Field installation, scalability, DGMS compliance, Atmanirbhar Bharat.
* **Appendices:** Glossary, constants reference, sigma formula, citation ledger, acronyms.

---
*Governed under Smart India Hackathon 2026 repository guidelines.*
