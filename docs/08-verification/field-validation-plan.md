# Field Validation & Coalfield Pilot Trial Plan

**Module 08 — Verification**  
**Cross-References:** [`test-register.md`](test-register.md) · [`build-order.md`](build-order.md) · [Module 09 Installation](../09-deployment-and-impact/installation-procedure.md)

---

### Question: Which underground coal mining sites are selected for AEGIS field trials, and what geological conditions make them definitive proving grounds?

**Answer:** To rigorously prove operational readiness under actual coal extraction conditions, field validation is targeted across two premier mechanized underground coal operations in India:

1. **Primary Candidate Site: Adriyala Longwall Project (SCCL, Godavari Valley Coalfield, Telangana)**
   * **Geotechnical Environment:** India’s flagship deep mechanized continuous longwall operation, working Seam No. 1 at an overburden depth $H = 375\text{ m}$ with an extraction thickness $m = 3.0\text{ m}$. Overlying strata consists of massive Barakar sandstones prone to periodic roof weighting and dynamic caving.
   * **Operational Dynamics:** Steady face advance rates between $3.5\text{ and }4.5\text{ m/day}$ produce a predictable, propagating subsidence wave. This continuous displacement front provides the dynamic ground movement necessary to rigorously test real-time strain accumulation and predictive time-to-crack models under high-stress conditions.
   * **Ground Truth Availability:** Extensive baseline borehole lithology logs, historical multi-point borehole extensometer (MPBX) data, and active daily geotechnical shift inspections.

2. **Secondary Candidate Site: Jhanjra Project (ECL, Raniganj Coalfield, West Bengal)**
   * **Geotechnical Environment:** Deep mechanized longwall face working the continuous R-VII seam with complex multi-aquifer overburden layering.
   * **Operational Advantage:** Proximity to premier research bodies (CSIR-CIMFR and IIT ISM Dhanbad), enabling independent third-party geotechnical audit oversight.

---

### Question: How is the physical deployment array scaled for field trials, and how does it transition from a pilot cluster to a full-panel array?

**Answer:** Rather than deploying an arbitrary static node count, AEGIS sizes sensor meshes dynamically using Knothe physical mechanics and empirical draw angles ($\beta$):

```
                       PILOT SUB-SLICE vs. FULL-PANEL SCALING
                       
 [ 600m × 200m Active Pilot Sub-Slice ]
   ├── 37 Scout Stations (17 Tilt, 14 Strain, 6 Extensometer)
   ├── 6 Anchor Relays (Cluster heads; max 5 children per anchor)
   └── 1 Master Gateway Hub (10m mast, SX1302 concentrator, 125 dB siren)
   
                           │   Algorithmic Expansion:
                           │   Spatial density scaled by Influence Radius:
                           ▼   r = H / tan(β), Grid Spacing: 15–25m
                           
 [ 1,800m × 300m Full-Scale Mechanized Longwall Panel ]
   ├── 327 Scout Stations (Distributed along traveling and cross-profiles)
   ├── 82 Anchor Relays (Multi-hop backbone mesh, fan-out ≤ 5)
   └── 2 Redundant Master Gateways (Alternating 125 kHz channels)
```

* **Pilot Sizing Rationale:** For the Adriyala 40-day trial (covering $\approx 160\text{ m}$ of face advance), an active sub-slice measuring $600\text{ m} \times 200\text{ m}$ centered over the active face inflection zone is instrumented with 37 Scout nodes (17 Tier 1A Tilt, 14 Tier 1B Strain, 6 Tier 1C Extensometers).
* **Backbone Infrastructure:** 6 Anchor Relays are distributed to maintain a minimum $+10\text{ dB}$ RF link margin under worst-case 90th percentile shadow fading, terminating at 1 Master Gateway mounted on a 10m pneumatic mast at the colliery surface substation.
* **Deterministic Scalability:** Transitioning from the 37-node pilot sub-slice to a 409-node full-panel array requires zero modifications to node firmware, wire serialization, or backend ingestion pipelines. Station spacing is mathematically dictated by the tensile zone boundaries:
  $$x_{\text{inflection}} = x_{\text{face}} \pm 0.5 \cdot r = x_{\text{face}} \pm \frac{H}{2 \tan\beta}$$
  Concentrating sensors along the inflection zone ($15\text{–}25\text{ m}$ spacing) while relaxing spacing to $50\text{ m}$ in stable far-field zones guarantees that any localized tensile crack ($\ge 1500\ \mu\varepsilon$) is captured by at least three adjacent nodes for Byzantine quorum consensus.

---

### Question: How is AEGIS scientifically validated against statutory survey practices during side-by-side benchmarking?

**Answer:** AEGIS operates simultaneously alongside the colliery’s official DGMS-mandated ground control survey team to provide direct, incontrovertible empirical comparison:

```
+-----------------------------------------------------------------------------------+
|                        SIDE-BY-SIDE FIELD BENCHMARKING                            |
|                                                                                   |
|  [ PROPAGATING SUBSIDENCE WAVE ]                                                  |
|         │                                                                         |
|         ├───> [ AEGIS Autonomous Wireless Mesh ]                                  |
|         │     - Continuous 60-second automated sampling                           |
|         │     - Micro-strain (1 µε LSB) & tilt (2 µrad LSB) derivative tracking    |
|         │     - Dynamic multi-hop LoRa mesh with store-and-forward persistence    |
|         │     - Deterministic C8 Safety Quorum with <1.4s automated siren relay   |
|         │                                                                         |
|         └───> [ Colliery Total Station Survey Team (Statutory Standard) ]         |
|               - Manual weekly leveling runs across control pegs                   |
|               - Optical theodolite angle measurements                             |
|               - 7-day data latency; static post-hoc spreadsheet plotting          |
|               - Zero warning capability between manual survey intervals           |
+-----------------------------------------------------------------------------------+
```

1. **Physical Mechanical Coupling:** Each Tier 1B (Horizontal Strain) and Tier 1C (Extensometer) station is anchored directly to an official DGMS brass survey monument driven $1.5\text{ m}$ into the subsoil. This ensures both AEGIS transducers and optical theodolite prisms measure identical physical ground displacement.
2. **Weekly Correlation Audits:** At the conclusion of each weekly manual survey run, the Colliery Survey Officer exports raw leveling elevations ($z$). These benchmarks are correlated against the AEGIS continuous time-series (`nodes.csv`) using linear regression to verify absolute displacement tracking accuracy.
3. **Temporal Advantage Capture:** The benchmark tracks the exact time delta between AEGIS detecting tensile strain inflection ($\varepsilon \ge 1500\ \mu\varepsilon$) versus the manual team discovering surface fissures days later during visual line inspections.

---

### Question: What quantitative acceptance criteria must AEGIS achieve to secure statutory DGMS pilot sign-off?

**Answer:** Statutory approval from the Directorate General of Mines Safety (DGMS) requires satisfying strict, non-negotiable performance thresholds across safety, geotechnical accuracy, and radio resilience over the 40-day operational trial:

| Performance Metric | Minimum Statutory Threshold | Verification & Measurement Protocol |
| :--- | :--- | :--- |
| **Displacement Correlation (R²)** | **R² ≥ 0.90** vs. manual leveling | Linear regression of daily median node subsidence against weekly optical Total Station survey elevations. |
| **Advance Crack Warning** | **≥ 7.0 days** prior to surface rupture | Time interval between AEGIS flagging critical tensile strain (ε > 1500 µε) and visual ground cracking verified in the field shift book. |
| **False Blast Alarms** | **Zero false evacuation sirens** | Cross-referencing all seismic trigger vetoes against the statutory DGMS colliery blasting log (`events.csv`). 100% blast veto accuracy required. |
| **Telemetry Availability (PDR)** | **≥ 99.5% packet delivery** | Ratio of persisted frames in `nodes.csv` to scheduled TDMA transmission slots across 40 continuous operating days (57,600 superframes). |
| **Emergency Trigger Latency** | **< 1.4 seconds** | Wall-clock latency measured from mechanical trip switch injection at a remote Scout to hardware relay closure of the 125 dB surface siren. |
| **Byzantine Resilience** | **Zero single-node false trips** | Controlled electrical shorting or mechanical destruction of individual sensors must produce diagnostic warnings (**F4**), never an evacuation siren. |
| **Environmental Enclosure Protection** | **IP67 Verification** | Physical teardown of all 37 pilot Scout enclosures at Day 40; zero internal dust accumulation or moisture condensation. |
| **Power Autonomy** | **V_bat ≥ 3.3 V continuous** | Battery voltage telemetry must demonstrate zero net discharge over consecutive cloudy or monsoon operational periods. |
