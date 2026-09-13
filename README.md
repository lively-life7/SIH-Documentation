# SIH-Documentation
AEGIS: AI-Enabled Mine Subsidence Monitoring &amp; Early Warning Platform (SIH26025)
# AEGIS — Format B Blueprint: Modular Technical Handbook (Multi-Document)

> Separate files per module in a `docs/` folder. Each file is self-contained but cross-linked. A reviewer can enter at any module and get full context.

---

## Folder Structure

```
docs/
├── 00-executive-gateway/
│   ├── project-charter.md
│   ├── system-architecture.md
│   └── key-metrics-summary.md
├── 01-ground-reality/
│   ├── crisis-landscape.md
│   ├── existing-approaches.md
│   └── opportunity-statement.md
├── 02-sensor-hardware/
│   ├── node-classification.md
│   ├── sensor-suite.md
│   ├── bill-of-materials.md
│   ├── power-and-energy.md
│   └── edge-intelligence.md
├── 03-mesh-networking/
│   ├── protocol-selection.md
│   ├── tdma-scheduling.md
│   ├── wire-format.md
│   ├── routing-and-failover.md
│   ├── store-and-forward.md
│   ├── grid-spacing-rationale.md
│   └── spectrum-compliance.md
├── 04-physics-engine/
│   ├── knothe-model.md
│   ├── derived-quantities.md
│   ├── corruption-chain.md
│   └── ground-truth-generation.md
├── 05-ai-ml-pipeline/
│   ├── pinn-architecture.md
│   ├── loss-formulation.md
│   ├── training-contract.md
│   ├── surface-reconstruction.md
│   └── safety-boundary.md
├── 06-backend-pipeline/
│   ├── data-architecture.md
│   ├── c7-corrector.md
│   ├── c8-alarm-engine.md
│   └── pipeline-flow.md
├── 07-digital-twin-and-ui/
│   ├── 3d-visualization.md
│   ├── operator-dashboard.md
│   ├── alert-system.md
│   └── web-mobile-platform.md
├── 08-verification/
│   ├── test-register.md
│   ├── build-order.md
│   └── field-validation-plan.md
├── 09-deployment-and-impact/
│   ├── installation-procedure.md
│   ├── scalability.md
│   ├── regulatory-compliance.md
│   ├── cost-benefit-analysis.md
│   └── national-alignment.md
└── appendices/
    ├── glossary.md
    ├── constants-reference.md
    ├── sigma-formula.md
    ├── citation-ledger.md
    └── acronyms.md
```

---

## Module-by-Module Blueprint

---

### Module 00 — Executive Gateway

> The entry point. A reviewer reads this first and decides which modules to deep-dive into.

#### `project-charter.md`
<!-- Why this project exists
     - Problem statement origin (SIH26025, Ministry of Coal)
     - Team members and roles
     - Repository governance pointer
     - Project timeline and current status -->

#### `system-architecture.md`
<!-- End-to-end system topology
     - Full block diagram: Sensors → Mesh → Gateway → C7 → C8 → C9 → Twin → Alerts
     - Five inviolable system boundaries
     - Component ownership matrix (subsystem | owns | does NOT own)
     - Data flow with byte counts and latencies at each stage -->

#### `key-metrics-summary.md`
<!-- One-page scorecard
     - ₹1,050 per Scout Node
     - 8.9-day advance crack prediction
     - <1.4s end-to-end siren latency
     - 31-node full panel coverage for <₹1.5 Lakhs
     - 72h zero data loss buffer
     - T1–T46 verification register
     - Zero false alarm (Byzantine quorum) -->

---

### Module 01 — Ground Reality

> Context and motivation. Why the problem is real, why existing solutions fail, and what gap AEGIS fills.

#### `crisis-landscape.md`
<!-- DGMS fatality statistics (273 deaths, 63% roof/ground fall)
     India's underground expansion (26 MT → 100 MT by 2030)
     Infrastructure damage (₹313 Cr railway, aquifer destruction)
     Jharia Master Plan (₹5,940 Cr, 110-year fire) -->

#### `existing-approaches.md`
<!-- Manual theodolite surveys — what they do, 15–30 day blind spot
     Imported geotechnical loggers — ₹2–5L per unit, sparse grids
     Satellite InSAR — temporal lag, cloud interference
     Post-facto damage assessments — reactive, not predictive
     Why each fails on its own terms -->

#### `opportunity-statement.md`
<!-- The gap: no indigenous, real-time, affordable, scalable, intelligent solution exists
     Frame as engineering opportunity, not checklist response
     Position AEGIS without naming the problem statement -->

---

### Module 02 — Sensor Hardware

> Everything that physically sits on a node: sensors, compute, power, enclosure, and on-node processing.

#### `node-classification.md`
<!-- Tier 1: Scout Node — form factor, specs, role, cost
     Tier 2A: Bedrock Reference Anchor — stable baseline, CMR
     Tier 2B: Deep Borehole Geotech Anchor — piezometers, extensometers
     Tier 3: Master Edge Gateway — SX1302, uplink, siren
     Diagram showing placement across a mine panel -->

#### `sensor-suite.md`
<!-- Full 7-sensor table:
     #1 Tilt (MPU-6050), #2 Die Temp, #3 Strain (ADS1115),
     #4 Extensometer (invar wire), #5 Crack Detector (conductive trace),
     #6 Vibration (accelerometer burst), #7 Battery Monitor
     For each: part, input, output channels, wire format, range, role
     "Two sensors that are not sensors" explanation (temp + battery as correction keys) -->

#### `bill-of-materials.md`
<!-- Full BOM table: part | quantity | unit cost | supplier
     Total per Scout: ~₹1,050
     Total per panel (31 nodes): <₹1.5 Lakhs
     Comparison table vs imported systems (₹2–5L per unit, ₹40–60L per panel) -->

#### `power-and-energy.md`
<!-- Solar panel sizing and harvesting
     Li-ion battery specs and charge/discharge curves
     Sleep/wake duty cycling strategy
     ESP32 compute budget table (read: 3ms, FFT: 1.5ms, burst: 640ms) -->

#### `edge-intelligence.md`
<!-- On-node 200 Hz FFT vibration classifier
     4-band discrimination:
       8–20 Hz → trucks (veto)
       50 Hz → conveyor harmonics (notch)
       40–80 Hz → blasting (cross-ref DGMS logs, veto)
       100–250 Hz → genuine rock fracture (retain + escalate)
     Why raw streaming would overwhelm LoRa bandwidth -->

---

### Module 03 — Mesh Networking

> How data moves from 31 field nodes to the gateway: protocol, scheduling, packet format, routing, resilience, and compliance.

#### `protocol-selection.md`
<!-- LoRa vs Zigbee vs Wi-Fi mesh vs NB-IoT
     Decision matrix: range, power, cost, license, duty cycle
     Why LoRa won for this application -->

#### `tdma-scheduling.md`
<!-- 60-second superframe architecture
     Beacon → cluster blocks → relay slots → backup slots → quiet period
     Slot assignment algorithm
     Collision avoidance mechanics
     Bitmap ACK strategy -->

#### `wire-format.md`
<!-- 23-byte binary packet: byte-by-byte map
     Fields: tilt_x, tilt_y, strain_ue, ext_delta, vib_rms, vib_peak, vib_fdom,
             temp_dc, vbat_mv, status_flags, epoch_lo
     Why 23 bytes: airtime budget derivation -->

#### `routing-and-failover.md`
<!-- Multi-frequency DAG topology (Ch₁–Ch₄ cluster, Ch₅–Ch₆ backbone)
     Gradient hop-count routing
     What happens when a relay is destroyed:
       ACK timeout detection → neighbor discovery → re-route → backfill
     FTSP time synchronization (DIO1 hardware interrupts + GPS beacons) -->

#### `store-and-forward.md`
<!-- On-board SPI flash ring buffer
     Capacity: 72 hours = 4,320 epochs
     Automatic backfill protocol when connectivity restores
     Zero data loss proof (math + test references: T11, T13, T29) -->

#### `grid-spacing-rationale.md`
<!-- Why 15–25m, not 100–300m
     Knothe influence radius r = 75m
     Nyquist sampling limit: Δ ≤ r/2.86 ≈ 25m
     What sparse grids miss: 2m localized fissures
     Grid spacing is physics-driven, NOT radio-range-driven -->

#### `spectrum-compliance.md`
<!-- Indian radio law: GSR 564(E)
     IN865 band: 865–867 MHz
     Carrier BW: 125 kHz (within 200 kHz statutory ceiling)
     Duty cycle: self-imposed 1% ceiling
     License-free operation: no WPC approval needed -->

---

### Module 04 — Physics Engine

> The Knothe subsidence model, derived quantities, corruption modeling, and synthetic ground-truth generation.

#### `knothe-model.md`
<!-- The master formula: S(x, y, t) = S_final(x,y) · (1 − e^(−ct))
     Physical meaning of each term
     Frozen constants table (H, TAN_BETA, R_INFL, M_SEAM, A_SUBS, S_MAX, C_KNOTHE, B_HORIZ)
     Panel geometry -->

#### `derived-quantities.md`
<!-- First derivative → tilt gradient (∇θ)
     Second derivative → horizontal strain (εyy = B · ∂²S/∂y²)
     "Strain is the channel that detects things"
     Critical strain threshold: θ_c = 1500 µε → t_crack = 8.93 days
     Invisible crack detection concept -->

#### `corruption-chain.md`
<!-- 6-stage degradation model:
     Stage 1: Thermal drift (k_T per channel)
     Stage 2: Quantization noise (LSB granularity)
     Stage 3: Bias drift (σ_b per channel)
     Stage 4: Battery sag (voltage-dependent scaling)
     Stage 5: Packet loss (missing epochs)
     Stage 6: Stale data (age-based uncertainty inflation)
     Why modeling corruption is essential for realistic simulation -->

#### `ground-truth-generation.md`
<!-- truth.npz pipeline
     Quarantine: truth/ has no import path from backend/ (enforced by T8)
     Simulation ↔ reality mapping
     How we generate millions of correct rows from one formula -->

---

### Module 05 — AI/ML Pipeline

> Physics-Informed Neural Network (PINN): architecture, training, surface reconstruction, and the safety firewall.

#### `pinn-architecture.md`
<!-- Why PINN, not black-box ML
     The hallucination problem with standard models on noisy mining data
     How Knothe-constrained loss prevents non-physical predictions
     Neural network topology diagram -->

#### `loss-formulation.md`
<!-- PDE residual loss (Knothe differential equation)
     Boundary condition loss
     Data-fit loss
     Weighting strategy between the three -->

#### `training-contract.md`
<!-- 24-hour training window (48 slices)
     Why 48 slices: mathematical identifiability of ĉ
     What the PINN learns (a_subs, c_knothe) vs what is frozen (H, r, B_HORIZ)
     Retraining cadence -->

#### `surface-reconstruction.md`
<!-- Sparse-to-dense: 31 discrete points → continuous 3D surface
     Leave-One-Out (LOO) residual validation
     Largest Empty Circle (LEC) metric: d_committed = 358m
     Output: deformation heatmap for CesiumJS -->

#### `safety-boundary.md`
<!-- THE critical document
     C9 PINN: draws terrain ONLY — no alarm authority
     C8 detector: EXCLUSIVELY owns alarm decisions
     Hard architectural firewall between C8 and C9
     No neural network / LLM in the safety-critical loop
     Why this matters for DGMS regulatory compliance
     100% mathematical auditability -->

---

### Module 06 — Backend Pipeline

> Data ingestion, 8-step correction, deterministic alarm engine, and pipeline orchestration.

#### `data-architecture.md`
<!-- nodes.csv: 36h rolling window, ~8 MB fixed size
     Parquet cold archive for historical analysis
     nodes.json: frozen geometry + calibration manifest (31 entries)
     events.csv: DGMS blast register + lightning register
     How files relate to each other -->

#### `c7-corrector.md`
<!-- 8-step cleaning pipeline:
     1. Assemble epoch (missing = missing, never zero)
     2. Decode status_flags (7 booleans)
     3. Undo battery sag
     4. Undo thermal drift
     5. Convert to SI units
     6. Common-mode rejection (subtract anchor baseline)
     7. Compute dynamic σ per channel (full formula)
     8. Build valid mask (N × 4 boolean)
     Code order: sag first, then thermal (with code snippet) -->

#### `c8-alarm-engine.md`
<!-- Multi-level safety thresholds: advisory → warning → critical → emergency
     5-node Byzantine Quorum Gating (≥3σ across spatial neighbors)
     DGMS blast register cross-reference (veto scheduled detonations)
     4-source vibration discrimination (trucks/conveyors/blasts/genuine fracture)
     Staleness rule F9: old readings cannot trigger alarms
     F3/F4/F10 discriminators
     Actions: siren, SMS, dashboard — NOT stored in any data file -->

#### `pipeline-flow.md`
<!-- Mermaid diagram: nodes.csv + nodes.json → C7 → C8 → Actions
                                                   → C9 → Dashboard
                      events.csv → C8 (separate file)
                      C9 ——NEVER——→ C8
     Annotate with data types and latencies at each arrow -->

---

### Module 07 — Digital Twin & UI

> 3D visualization, operator dashboards, alert delivery, and web/mobile platform architecture.

#### `3d-visualization.md`
<!-- CesiumJS / Three.js terrain rendering
     Real-time deformation heatmap overlaid on satellite imagery
     Subsidence contour animation over time
     Infrastructure risk overlay: railways, highways, aquifers, settlements
     PINN surface mesh → CesiumJS rendering pipeline -->

#### `operator-dashboard.md`
<!-- Live telemetry panels: per-node health, sensor readings, battery status
     Multi-tier alert timeline (advisory/warning/critical/emergency)
     Historical trend analysis and playback
     Role-based views:
       Mine operator: real-time alerts + node status
       Planner: subsidence progression + extraction rate impact
       Regulator: compliance audit trail + alert history -->

#### `alert-system.md`
<!-- Siren activation: <1.4s end-to-end latency (derivation)
     SMS notifications (supervisor, control room)
     Email reports (shift summaries, threshold breaches)
     Mobile push notifications (acknowledge/escalate workflow)
     Escalation logic: if not acknowledged within X min → escalate to Y -->

#### `web-mobile-platform.md`
<!-- Responsive web application architecture
     Mobile companion app: alert acknowledgment, field inspection checklists
     Offline-first design: local cache + periodic cloud sync
     API contracts: WebSocket for real-time, REST for historical queries -->

---

### Module 08 — Verification

> Test catalogue, build sequence, and field validation planning.

#### `test-register.md`
<!-- T1–T46: complete catalogue
     For each: test ID | subsystem | description | acceptance criteria | what breaks if it fails
     Grouped by domain:
       Physics (T1–T10)
       Mesh/Radio (T11–T26)
       Pipeline/Backend (T27–T46) -->

#### `build-order.md`
<!-- 6-day milestone sequence:
     Day 1: constants.py, nodes.json (31 entries frozen), events.csv → V1–V12, E1–E8
     Day 2: Layers 0–2 (surface, derivatives, weather, vibration) → T1–T5, T9, T10
     Day 3: Layer 3 (6-stage corruption) → T6, T7 — IF THESE FAIL, STOP
     Day 4: Layer 4 (mesh, TDMA, PDR, aggregation, failover) → T11, T12, T16–T26
     Day 5: Output files (readings.parquet, nodes.csv, radio_report.json) → T8, T13, T27–T31
     Day 6: FREEZE — scenario tuning only → Full suite T1–T46 green -->

#### `field-validation-plan.md`
<!-- Target coalfield for pilot (which Coal India subsidiary)
     Number of nodes for trial deployment
     Duration and monitoring cadence
     Success metrics: detection accuracy, false alarm rate, uptime, latency
     Comparison methodology: AEGIS vs manual survey on same panel -->

---

### Module 09 — Deployment & Impact

> Installation procedures, scalability, regulatory compliance, cost-benefit analysis, and national alignment.

#### `installation-procedure.md`
<!-- Step-by-step:
     1. Site survey (panel geometry, terrain, obstructions)
     2. Node placement (physics-derived grid, not ad-hoc)
     3. Anchor installation (bedrock reference + borehole geotech)
     4. Mesh commissioning (TDMA slot assignment, link budget verification)
     5. Calibration (baseline readings, CMR anchor lock)
     6. Go-live (alert thresholds, dashboard configuration, stakeholder training) -->

#### `scalability.md`
<!-- Same architecture: 1 panel or 50 panels
     Linear cost scaling (₹1.5L per panel)
     Gateway density: 1 per panel or 1 per cluster of adjacent panels
     Cloud infrastructure scaling: horizontal backend, per-panel PINN instances
     Multi-mine centralized monitoring center concept -->

#### `regulatory-compliance.md`
<!-- DGMS statutory standards alignment
     Coal Mines Regulations (CMR) 2017
     GSR 564(E) radio spectrum compliance
     Tamper-proof digital audit trails
     Blast register cross-referencing per DGMS Circular 7/1997 -->

#### `cost-benefit-analysis.md`
<!-- Per-panel deployment: <₹1.5 Lakhs vs ₹40–60 Lakhs imported
     Annual opex savings: no manual survey crew
     Infrastructure damage prevention: ₹313 Cr railway, mine shutdowns
     ROI timeline for Coal India subsidiaries
     Comparison table: AEGIS vs imported vs manual vs InSAR -->

#### `national-alignment.md`
<!-- Atmanirbhar Bharat: 100% indigenous BOM, zero import dependency
     Student-prototype friendly: all parts available Amazon.in/Robu.in
     Open architecture: adaptable for academic research
     Sustainable mining alignment: post-mining land reclamation support
     Smart mining vision: digital transformation of Indian coalfields -->

---

### Appendices

#### `glossary.md`
<!-- All shared vocabulary — sync with main repo glossary.md -->

#### `constants-reference.md`
<!-- Full constants.py table: ground, noise, radio, timing parameters -->

#### `sigma-formula.md`
<!-- Full dynamic σ derivation: σ_base → σ_cmr → σ_final
     f_link, f_gap, f_flag multiplier formulas -->

#### `citation-ledger.md`
<!-- Every factual claim → official source → regulatory citation
     DGMS, CAG, PIB, ISPRS, CIMFR, Ministry of Coal, WPC -->

#### `acronyms.md`
<!-- AEGIS, PINN, TDMA, CMR, PDR, LEC, DGMS, FTSP, DAG, FFT, BOM, etc. -->
