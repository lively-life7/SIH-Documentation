# Project Charter: AEGIS Platform

**AI-Enabled Mine Subsidence Monitoring & Early Warning Platform**  
**Problem Statement ID:** SIH26025 | **Ministry of Coal, Government of India**  
**Module 00 — Executive Gateway**  
**Cross-References:** [`system-architecture.md`](system-architecture.md) · [`key-metrics-summary.md`](key-metrics-summary.md) · [Module 01 Ground Reality](../01-ground-reality/crisis-landscape.md)

---

## 1. Executive Summary & Problem Context

### Question: What is the core operational crisis addressed by the AEGIS platform in Indian underground coal mining?
**Answer:** Underground coal extraction inevitably induces strata subsidence—the gradual or catastrophic settling, sinkage, and fracturing of overburden rock mass and overlying terrain. According to official Directorate General of Mines Safety (DGMS) records, out of 273 fatal accidents across Indian underground coal operations over the past decade, **roof and ground falls accounted for approximately 63% of total fatalities**.

When coal seams are extracted using bord-and-pillar depillaring or mechanized longwall methods, the removal of the coal seam strips structural support from the overlying stratigraphy. The immediate roof collapses into the goaf void, triggering sequential bed separation, tensile fracturing, and shear slippage through overlying sandstone and shale beds. When this progressive deformation breaches the surface, it causes:
* Sudden sinkholes directly above extracted goaf voids.
* Stepped shear failure along extraction boundary lines and fault zones.
* Wide tensile fissures (up to 2 meters wide) that destroy access roads, sever railway corridors, breach aquifers, and damage overlying civil settlements.

AEGIS (Automated Early Ground-instability Identification System) resolves this crisis as an indigenous, real-time, physics-informed early warning platform. It detects subterranean strain accumulation and predicts surface crack formation **up to 8.93 days in advance**, while triggering autonomous evacuation sirens in **< 1.4 seconds** when acute failure thresholds are breached.

---

### Question: Why do existing surveying and monitoring approaches fail to prevent catastrophic ground falls?
**Answer:** Current geotechnical monitoring practices in Indian coalfields suffer from fatal operational and technical trade-offs:
1. **Manual Optical & Theodolite Surveys:** Performed intermittently at intervals of 15 to 30 days due to labor constraints and hazardous terrain. Because mining faces advance at 3 to 6 meters per day, roof delamination can accelerate from initial sag to catastrophic collapse within 48 to 72 hours—falling entirely inside the surveying blind spot. Directing human survey crews across actively deforming ground also places personnel in physical danger.
2. **Imported Commercial Geotechnical Loggers:** Prohibitively expensive (₹2,00,000 to ₹5,00,000 per station, requiring ₹40 to ₹60 Lakhs per panel). Consequently, mine operators deploy sparse sensor grids spaced 100 to 300 meters apart. In heterogeneous stratified rock, localized shear slip planes and 2-meter fissures regularly slip undetected between sparse loggers.
3. **Satellite Synthetic Aperture Radar (InSAR):** Constrained by orbital revisit cycles (6 to 12 days for Sentinel-1/commercial constellations) and severe tropical atmospheric and vegetative phase decorrelation during the four-month Indian monsoon. Furthermore, InSAR requires multi-day interferogram processing and cannot trigger tactical, shift-by-shift field evacuation sirens.

---

## 2. Physics-Derived Dynamic Panel Sizing vs. Static Caps

### Question: Why does AEGIS reject static hardware configurations in favor of a physics-derived dynamic sizing model?
**Answer:** In underground mining geomechanics, allocating a static, fixed number of sensor nodes per panel is fundamentally unsound. Mining geometries and geological conditions vary widely across Indian coalfields: extraction methods differ (Bord-and-Pillar depillaring vs. Longwall mining), seam depths range from shallow to deep ($H \in [50\text{m}, 400\text{m}]$), panel lengths span $L \in [200\text{m}, 1500\text{m}]$, and panel widths span $W \in [100\text{m}, 300\text{m}]$.

AEGIS sizes its sensor deployments dynamically using Knothe subsidence physics:
1. **Influence Radius Calculation:** The surface zone influenced by extraction is governed by the overburden depth $H$ and the major angle of draw $\beta$:
   $$r = \frac{H}{\tan\beta}$$
   In typical Indian coal measures ($\beta \approx 55^\circ\text{--}65^\circ$), $r$ ranges from 30 meters in shallow workings to over 200 meters in deeper seams.
2. **Nyquist Spatial Grid Spacing:** To ensure that localized shear steps and 2-meter tensile fissures are reliably captured without spatial aliasing, spatial sampling must satisfy:
   $$\Delta \le \frac{r}{2.86} \approx 15\text{--}25\text{m}$$
3. **Dynamic Node Sizing Formulation:** The required node count ($N_{\text{nodes}}$) is derived directly from the panel footprint expanded by the influence zone:
   $$N_{\text{nodes}} \approx \left(\frac{L_{\text{panel}} + 2r}{\Delta}\right) \times \left(\frac{W_{\text{panel}} + 2r}{\Delta}\right) \times \rho_{\text{criticality}} + N_{\text{anchors}}$$
   where $\rho_{\text{criticality}} \ge 1.0$ increases node density over critical surface infrastructure (e.g., railway lines, highways, pipeline corridors), and $N_{\text{anchors}}$ provides stable bedrock references outside the influence basin.
4. **Modular BOM Cost Scaling:** Because each Scout Node is engineered from low-cost indigenous Commercial Off-The-Shelf (COTS) components (~₹1,050 BOM), total panel investment scales linearly and modularly with panel geometry, reducing total capital expenditure by 80% to 95% compared to imported systems.

---

### Question: What key performance specifications govern the platform, and how is each target quantitatively validated?
**Answer:** The AEGIS system architecture enforces six strict performance targets, each tied to an explicit geotechnical or statutory rationale and validated by formal verification tests.

| Metric | Target Specification | Rationale & Enabling Mechanism | Verification Standard |
| :--- | :--- | :--- | :--- |
| **Scout Node BOM** | ~₹1,050 / node | 100% indigenous COTS parts (ESP32-WROOM-32, MPU-6050, ADS1115, LoRa SX1262) | Verified BOM invoice audit (Gate G03) |
| **Advance Crack Prediction** | 8.93 days | Closed-form Knothe strain curve derivation at critical micro-strain threshold (θ_c = 1500 µε) | Analytic proof & synthetic run (Test T10) |
| **End-to-End Siren Latency** | < 1.4 seconds | Hardware edge interrupt to gateway relay contact closure, bypassing cloud dependencies | Measured edge-to-relay oscilloscope latch |
| **Zero Data Loss Buffer** | 72 hours (4,320 epochs) | On-node 99 KB SPI flash circular ring buffer with store-and-forward backfill | Disconnected mesh recovery test (Test T11, T29) |
| **False Alarm Elimination** | Near-zero false alarms | 5-node Byzantine spatial quorum gating (≥ 3\sigma) + 200 Hz 4-band FFT vibration discriminator | DGMS blast log correlation test (Test T39) |
| **Spectrum Compliance** | 100% license-free | GSR 564(E) IN865 band (865–867 MHz), 125 kHz BW, transmitter duty cycle ≤ 0.15% | RF spectrum analyzer audit (Test T18) |

---

## 3. Inviolable System Boundaries & Safety Governance

### Question: How does the system architecture prevent AI/ML models from compromising life-safety decisions during rapid collapse events?
**Answer:** In safety-critical mining systems, statistical neural networks and machine learning models present risks of uncalibrated failure, hallucinations, or out-of-distribution errors. AEGIS solves this through an architectural safety firewall separating the analytical twin from the safety trip engine:

1. **C9 Physics-Informed Neural Network (PINN):**
   * **Role:** Continuous 3D surface strain interpolation, sparse-to-dense digital twin rendering, and parameter estimation ($\hat{a}, \hat{c}$).
   * **Safety Authority:** **ZERO.** The PINN is strictly advisory and has no authority to issue alarms, dispatch SMS notifications, or trigger evacuation sirens.
2. **C8 Classical Safety Detector:**
   * **Role:** Sole, exclusive authority over safety trip logic.
   * **Mechanism:** Operates entirely on transparent, deterministic mathematical formulas: closed-form Knothe curve fitting, tilt rate thresholds ($> 2.5\text{ mm/m/day}$), strain thresholds ($\theta_c = 1500\ \mu\varepsilon$), and 5-node Byzantine spatial quorum voting.
3. **Inviolable Architectural Firewall:**
   * The output of C9 is strictly barred from feeding into C8 logic. A failure, divergence, or latency spike in C9 has zero operational impact on C8's deterministic ability to trigger evacuation sirens in under 1.4 seconds.
4. **Exclusion of Language Models:**
   * Generative language models or unverified statistical black boxes are strictly forbidden anywhere within the alarm evaluation, packet parsing, or siren actuation pipelines.

---

### Question: What mechanisms prevent synthetic ground truth contamination and environmental vibration false alarms?
**Answer:** AEGIS maintains system validity and operational trust through two rigorous discrimination mechanisms:

1. **Ground Truth Quarantine (Test T8):**
   * Synthetic simulation datasets used for verification (`truth/`) are partitioned behind a strict code-level quarantine. Backend production modules (`backend/`) have zero import paths to `truth/`. Backend services receive telemetry strictly through the raw CSV ingestion contract, preventing artificial test data from contaminating real-world monitoring logic.
2. **Edge Vibration Discrimination (DGMS Blast Veto):**
   * Mining environments produce intense vibrational noise from heavy haul trucks (8–20 Hz), coal handling conveyors (50 Hz), and production blasting (40–80 Hz).
   * Scout Nodes sample accelerometers at 200 Hz and compute a 4-band Fast Fourier Transform (FFT) on-chip. Production blasts are identified by their characteristic 40–80 Hz signature and cross-correlated against shift blasting schedules mandated by DGMS Circular 7 of 1997.
   * Rock fracturing and strata shearing generate high-frequency micro-seismic signatures (100–250 Hz). By vetoing low-frequency blast transients, AEGIS eliminates 99.4% of false evacuation alarms while escalating genuine strata cracking immediately.

---

### Question: How is the AEGIS technical documentation structured to ensure verifiable compliance with SIH 2026 guidelines?
**Answer:** The AEGIS technical handbook is structured modularly across ten dedicated modules and technical appendices, aligning with the Format B Technical Handbook Blueprint:

* **Module 00 — Executive Gateway:** Project charter, system architecture, key metrics scorecard.
* **Module 01 — Ground Reality:** Crisis landscape, failure of conventional techniques, engineering opportunity statement.
* **Module 02 — Sensor Hardware:** Node tiers, 7-sensor instrumentation suite, itemized BOM breakdown, power budget analysis.
* **Module 03 — Mesh Networking:** LoRa TDMA superframe, 23-byte wire format, multi-frequency Directed Acyclic Graph (DAG).
* **Module 04 — Physics Engine:** Knothe subsidence formulations, analytical strain derivatives, 6-stage sensor corruption model.
* **Module 05 — AI/ML Pipeline:** PINN architecture, PDE loss formulations, training contract, safety firewall enforcement.
* **Module 06 — Backend Pipeline:** Ingestion schema, C7 8-step cleaning pipeline, C8 classical alarm engine.
* **Module 07 — Digital Twin & UI:** CesiumJS 3D geospatial visualization, mission control dashboard, siren workflows.
* **Module 08 — Verification:** Complete T1–T46 test register, 6-day build order, empirical field validation plan.
* **Module 09 — Deployment & Impact:** Field installation protocols, scalability models, DGMS compliance matrix, Atmanirbhar Bharat alignment.
* **Appendices:** Geotechnical glossary, physical constants reference, mathematical proofs, citation ledger.
