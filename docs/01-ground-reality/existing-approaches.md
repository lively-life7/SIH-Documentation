# Evaluation of Existing Monitoring Approaches

**Module 01 — Ground Reality**  
**Cross-References:** [`crisis-landscape.md`](crisis-landscape.md) · [`opportunity-statement.md`](opportunity-statement.md) · [Citation Ledger](../appendices/citation-ledger.md)

---

## 1. Overview of Contemporary Practices

Mining operators in India and internationally rely on four primary categories of ground monitoring. Each approach addresses a specific measurement dimension, but each breaks down when evaluated against the requirements of real-time life safety and rapid evacuation.

```
+-------------------------------------------------------------------------------+
|                       THE 4 CONVENTIONAL APPROACHES                           |
+----------------------+-------------------------+------------------------------+
| Approach             | Core Mechanism          | Primary Breakdown Mode       |
+----------------------+-------------------------+------------------------------+
| 1. Optical Surveys   | Theodolites / Total Stn | 15–30 day blind spot         |
| 2. Geotech Loggers   | Borehole Extensometers  | Prohibitive cost (₹2–5L/ea)  |
| 3. Satellite InSAR   | Spaceborne SAR (C-band) | Cloud decorrelation & revisit|
| 4. Post-Facto Audits | Visual fissure mapping  | Completely reactive          |
+----------------------+-------------------------+------------------------------+
```

---

## 2. Technical Evaluation & Failure Modes

### 1. Manual Optical & Theodolite Surveys
* **Methodology:** Survey personnel visit established grid stations with Total Stations, optical levels, or differential GPS rovers.
* **The Fatal Limitation (Temporal Blind Spot):** Surveys are scheduled at intervals of 15 to 30 days due to labor constraints and hazardous field terrain. Coal extraction advances continuously at 3 to 6 meters per day. Between survey rounds, overburden delamination can accelerate from initial roof sagging to total surface collapse in 48 to 72 hours.
* **Safety Hazard to Crews:** Directing human surveyors to walk across an actively deforming mining basin with active fracture propagation places field staff in direct physical peril.

### 2. Imported Commercial Geotechnical Loggers
* **Methodology:** Multi-point borehole extensometers (MPBX), vibrating wire piezometers, and automated tilt loggers sourced from international vendors (e.g., Campbell Scientific, RST Instruments, Sisgeo).
* **The Fatal Limitation (Prohibitive Unit Economics):** An individual certified telemetry station costs between ₹2,00,000 and ₹5,00,000. Equipping a single 600m × 200m coal panel with dense coverage would demand ₹40–60 Lakhs in capital expenditure.
* **Resulting Sparse Grid:** Because units are expensive, mine managers space them at 100m to 300m intervals. In heterogeneous stratified rock, localized shear slip planes and 2m fissures slip undetected between sensor locations.

### 3. Satellite Synthetic Aperture Radar (InSAR)
* **Methodology:** Phase difference interferometry using orbital sensors (Sentinel-1, TerraSAR-X, ALOS-2).
* **The Fatal Limitation (Atmospheric & Temporal Lag):**
  * *Revisit Latency:* Sentinel-1 provides a 12-day repeat orbit over India. If an acceleration occurs on Day 2, orbital radar cannot observe it until Day 12.
  * *Monsoon Phase Decorrelation:* Heavy precipitation, soil moisture swings, and dense seasonal vegetation destroy radar coherence across Indian mining fields (Jharkhand, Odisha, Chhattisgarh, Telangana) for four months each year.
  * *Tactical Inutility:* InSAR provides post-hoc deformation contour maps for mine planners; it cannot trip a field siren to evacuate underground miners or stop a train approaching a tilting track.

### 4. Post-Facto Visual Inspections
* **Methodology:** Field staff report surface cracks after they become visible to the eye.
* **The Fatal Limitation:** By the time a tensile crack breaches the surface soil layer, the sub-surface rock structure has already failed in shear. Warning time is zero.

---

## 3. Comparative Gap Analysis

| Critical Requirement | Manual Survey | Commercial Loggers | Satellite InSAR | Target System |
| :--- | :--- | :--- | :--- | :--- |
| **Real-Time Sampling (< 1 min)** | No (15–30 days) | Yes (10–60 min) | No (6–12 days) | **Yes (60 seconds)** |
| **Dense Spatial Grid (15–25m)** | No (50–100m) | No (100–300m) | Yes (15m pixel) | **Yes (15–25m)** |
| **Instant Siren Trigger (< 2s)** | No | No | No | **Yes (< 1.4s)** |
| **Algorithmic Low-Cost Scaling** | No (High OPEX) | No (₹40–60L CAPEX) | No (Commercial fees) | **Yes (~₹1,050–₹1,850/node)** |
| **Monsoon & Weather Immunity** | Poor | Good | Poor | **Good (IP67 + RF)** |
| **False Blast Discrimination** | N/A | Rare | None | **Yes (FFT + Blast Log)** |
