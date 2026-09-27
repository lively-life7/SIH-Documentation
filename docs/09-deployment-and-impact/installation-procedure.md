# Field Installation & Commissioning Procedures

**Module 09 — Deployment & Impact**  
**Cross-References:** [`scalability.md`](scalability.md) · [Module 02 Node Classification](../02-sensor-hardware/node-classification.md) · [Module 08 Field Validation](../08-verification/field-validation-plan.md)

---

## 1. Six-Stage Deployment Workflow

Field installation of the AEGIS platform follows a disciplined 6-stage operational protocol designed to minimize field labor while guaranteeing physics-compliant sensor placement.

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
  │  Initial zero-strain datum lock, siren relay test, SCADA handoff
```

---

## 2. Step-by-Step Installation Procedures

### Step 1: Geotechnical Data Ingestion
Before dispatching field crews, the engineering team extracts four parameters from the Colliery Working Plan:
* Seam extraction depth ($H$).
* Active panel boundary polygon coordinates: $(x_1, y_1)$ to $(x_2, y_2)$.
* Overburden stratigraphic draw angle ($\beta \implies \tan\beta$).
* High-resolution terrain elevation contours (CartoDEM / drone survey).

### Step 2: Algorithmic Sizing & Slot Assignment
The site parameters are ingested by the network sizing engine (`scripts/size_network.py`):
* Calculates influence radius $r = H / \tan\beta$.
* Generates the traveling profile cross geometry across the inflection zone.
* Emits the frozen configuration manifest: `config/nodes.json`, assigning deterministic TDMA slots, primary parent anchors, and backup routing paths for every station.

### Step 3: Bedrock Reference Anchor Installation
* Anchor stations ($A_1, A_2$) are surveyed at least $1.5\times r$ away from the extraction boundary.
* A $32\text{ mm}$ steel rebar peg is driven $1.5\text{ meters}$ deep into firm sandstone bedrock or compacted non-mining ground using a rotary percussion hammer drill.
* Anchors establish the undisturbed absolute elevation and common-mode rejection (CMR) baseline.

### Step 4: Scout Node & Sensor Mounting
1. **Rebar Peg Installation:** Field technicians drive $25\text{ mm}$ high-tensile steel rebar stakes into the ground at pre-computed GPS coordinates.
2. **Enclosure Attachment:** The IP67 polycarbonate enclosure is clamped to the peg $300\text{ mm}$ above surface level to avoid monsoon splashback.
3. **Solar Orientation:** Monocrystalline solar panels are oriented South-facing with a fixed $22^\circ$ tilt angle (optimized for central Indian latitudes).
4. **Strain & Extensometer Linking:** For Tier 1B and 1C stations, the 10m invar rod or 30m wire extensometer is tensioned between adjacent pegs with a calibrated spring pre-load.

### Step 5: Gateway Mast & Mesh Commissioning
* Erect a 10-meter telescopic pneumatic mast at the colliery substation or surface safety cabin.
* Power the Master Gateway via its 20W solar array and 12V LiFePO4 battery pack.
* Trigger a network survey beacon: The gateway cycles through all assigned TDMA slots, recording RSSI and SNR for each node. Any link with $< 10\text{ dB}$ shadow margin is re-routed to an alternative anchor.

### Step 6: Baseline Calibration Lock & Siren Check
* System runs continuously for 24 hours in calibration lock mode to record diurnal thermal baselines and lock the common-mode rejection anchor baseline.
* Technicians perform a controlled test trip of the 125 dB physical siren relay.
* Geotechnical Officer and Colliery Manager sign the DGMS commissioning handover certificate.
