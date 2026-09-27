# Field Installation & Commissioning Procedures

**Module 09 — Deployment & Impact**  
**Cross-References:** [`scalability.md`](scalability.md) · [Module 02 Node Classification](../02-sensor-hardware/node-classification.md) · [Module 08 Field Validation](../08-verification/field-validation-plan.md)

---

### Question: What is the standardized 6-stage operational sequence for deploying an AEGIS sensor array in an active coalfield?

**Answer:** To eliminate ad-hoc field decisions and minimize worker exposure in active extraction zones, AEGIS mandates a structured 6-stage deployment workflow:

```
[Stage 1: Geotechnical Site Survey]
  │  Input: Seam depth H, panel coordinates, draw angle β, terrain DEM
  ▼
[Stage 2: Algorithmic Node Layout Generation]
  │  Output: nodes.json (Exact coordinates, TDMA slots, routing DAG)
  ▼
[Stage 3: Bedrock Anchor & Reference Station Fixation]
  │  Installed > 1.5r outside basin into undisturbed bedrock
  ▼
[Stage 4: Scout Node & Invar Wire Deployment]
  │  Ground-driven rebar pegs, solar panel alignment, crack-trace mounting
  ▼
[Stage 5: Gateway Mast Erection & TDMA Commissioning]
  │  10m mast, SX1302 sync beacon, RF link budget verification (≥10 dB)
  ▼
[Stage 6: Common-Mode Calibration & Go-Live]
  │  Initial zero-strain datum lock, siren relay test, DGMS handoff
```

This procedure guarantees that every physical node is anchored to stable geological strata and mapped to a collision-free TDMA slot before live telemetry ingestion commences.

---

### Question: How are mine geotechnical parameters ingested to deterministically generate sensor coordinates and radio schedules prior to field mobilization?

**Answer:** Rather than placing sensors heuristically, the engineering team extracts four core parameters from the official Colliery Working Plan:
* Seam extraction depth ($H$ in meters).
* Active extraction panel polygon coordinates: $(x_1, y_1)$ to $(x_2, y_2)$.
* Overburden stratigraphic draw angle ($\beta \implies \tan\beta$).
* High-resolution terrain elevation contours (CartoDEM or colliery drone survey).

These parameters are processed by the automated network generation utility (`scripts/size_network.py`):
1. **Influence Radius Calculation:** Derives the spatial extent of the subsidence trough:
   $$r = \frac{H}{\tan\beta}$$
2. **Traveling Profile Cross Geometry:** Generates sensor coordinates along longitudinal and transverse lines across the panel inflection zone ($x_{\text{inflection}} = x_{\text{face}} \pm 0.5 \cdot r$). Spacing is dynamically compressed to $15\text{–}25\text{ m}$ in the critical tensile zone and relaxed to $50\text{ m}$ in the stable far-field.
3. **Deterministic Manifest Output:** Generates `config/nodes.json`. This configuration file fixes the node identifier, exact GPS coordinates, primary Anchor Relay, secondary failover parent, and microsecond TDMA time slot for every station, freezing the network topology before technicians arrive on site.

---

### Question: What civil and mechanical engineering protocols govern the physical mounting of Bedrock Anchors, Scout Stations, and Invar Extensometers?

**Answer:** To guarantee that measured deformations represent true deep-seated ground strain rather than topsoil creep or wind flutter, physical installation adheres to rigorous mechanical standards:

1. **Bedrock Reference Anchors ($A_1, A_2$):**
   * Anchors must be surveyed at least $1.5 \cdot r$ outside the maximum extraction boundary to ensure zero subsidence influence ($|S| < 1.0\text{ mm}$, Test T15).
   * A $32\text{ mm}$ high-tensile steel rebar peg is driven $1.5\text{ meters}$ deep into consolidated sandstone bedrock or undisturbed ground using a pneumatic percussion drill.
   * A concrete collar is cast around the surface collar to eliminate loose soil heave. These anchors establish the absolute geodetic datum and common-mode rejection baseline for the entire panel.
2. **Scout Stations (Tier 1A, 1B, 1C):**
   * Field technicians drive a $25\text{ mm}$ high-tensile steel rebar stake $1.0\text{ meter}$ into the ground at the pre-calculated GPS coordinate.
   * The IP67 polycarbonate electronics enclosure is clamped to the rebar peg exactly $300\text{ mm}$ above the ground surface. This elevation prevents water splashback and mud accumulation during intense monsoon cloudbursts while keeping the center of mass low to prevent wind vibration.
   * The 1W monocrystalline solar panel is oriented South-facing with a fixed $22^\circ$ inclination angle, mathematically optimized for central Indian latitudes to maximize winter solar insolation.
3. **Invar Rod and Wire Extensometers (Tier 1B & Tier 1C):**
   * For horizontal strain stations, a 10-meter invar rod or a 30-meter high-tensile invar wire is spanned between adjacent ground pegs across the predicted crack trajectory.
   * The line is terminated at a rotary potentiometer or linear optical encoder with an internal calibrated pre-tension spring ($15\text{ N}$) to eliminate thermal catenary sag.

---

### Question: How is the Master Gateway erected, and how is the wireless network verified against statutory link margin requirements?

**Answer:** Master Gateway commissioning follows a strict RF validation checklist:

1. **Physical Mast Erection:** A 10-meter pneumatic telescopic mast is erected at the surface colliery substation or outside the mine safety control cabin. The mast is secured with four high-tensile steel guy wires anchored at $90^\circ$ radial offsets.
2. **Antenna and Concentrator Installation:** A high-gain ($8.5\text{ dBi}$) omnidirectional collinear fiberglass antenna tuned to the 865–867 MHz band is mounted at the mast apex, coupled via low-loss RG-213 coaxial cable to the SX1302 8-channel LoRa concentrator.
3. **Power Subsystem:** The gateway is powered by a 20W monocrystalline solar panel paired with a 12V 24Ah $\text{LiFePO}_4$ battery pack, guaranteeing 7 continuous days of operation in complete darkness.
4. **RF Link Budget Verification (Test T21):**
   The gateway transmits an initial network synchronization beacon. Each Scout and Anchor node measures the downlink signal-to-noise ratio (SNR) and received signal strength indicator (RSSI). The gateway verifies that every primary and secondary backup parent link satisfies the minimum fade margin under 90th percentile log-normal shadow fading:
   $$\text{Fade Margin} = \text{RSSI}_{\text{measured}} - \text{Sensitivity}_{\text{SF7/SF8}} \ge 10.0\text{ dB}$$
   Any node failing the $10\text{ dB}$ margin is dynamically re-assigned to an intermediate Anchor Relay.

---

### Question: What mandatory calibration procedures and safety checks must be satisfied before achieving statutory operational sign-off?

**Answer:** Before the network transitions to live production status, two mandatory protocols are executed:

1. **24-Hour Baseline Calibration Lock:**
   The entire sensor network runs continuously for 24 hours without extracting coal. The C7 pipeline records the baseline diurnal thermal oscillation for every sensor ($k_T \cdot \Delta T_{\text{die}}$), verifies that common-mode topsoil swelling is correctly canceled by the Bedrock Anchors, and asserts that baseline drift residuals remain within the white-noise band ($\sigma_w \le 1.2\ \mu\varepsilon$, $\sigma_w \le 8.0\ \mu\text{rad}$).
2. **End-to-End Siren Relay Trip Test:**
   Technicians manually inject a simulated critical threshold trigger from a remote Scout node. The C8 Safety Detector must process the event, confirm spatial quorum, and energize the hardware dry-contact relay driving the 125 dB physical surface siren within **1.4 seconds**.
3. **Statutory Handover:**
   Upon successful siren verification, the Colliery Manager, Safety Officer, and Chief Geotechnical Surveyor sign the formal DGMS Commissioning Certificate, archiving the frozen `config/nodes.json` manifest into the colliery statutory safety register.
