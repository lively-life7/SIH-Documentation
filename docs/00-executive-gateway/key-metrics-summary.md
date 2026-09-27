# Key Metrics & Engineering Scorecard

**Module 00 — Executive Gateway**  
**Cross-References:** [`project-charter.md`](project-charter.md) · [`system-architecture.md`](system-architecture.md) · [Module 08 Verification](../08-verification/test-register.md)

---

## 1. Executive Performance Scorecard

The following table summarizes the verified engineering performance benchmarks of the AEGIS mine subsidence monitoring platform against statutory mining requirements and commercial benchmarks.

| Metric | Target Specification | Achieved / Verified Value | Verification Method |
| :--- | :--- | :--- | :--- |
| **Scout Node BOM Unit Cost** | < ₹2,000 / node | **₹1,050 – ₹1,850 / node** | Verified BOM invoice audit (Robu.in / local suppliers) |
| **System Capital Cost** | Algorithmic (no artificial budget cap) | **Modular (~₹1,050–₹1,850/node)** (e.g., ₹97.2k for 37-node pilot; ₹10.29L for 411-node full longwall) | Full Bill of Materials audit (Gate G03 itemized output) |
| **Advance Crack Warning** | > 7 days prior to surface tearing | **8.93 days** ($\approx 214\text{ hours}$) | Analytic closed-form derivation at $\theta_c = 1500\ \mu\varepsilon$ (Test T10) |
| **Emergency Siren Latency** | < 2.0 seconds end-to-end | **< 1.4 seconds** | Hardware edge interrupt to gateway relay contact closure |
| **Zero Data Loss Storage Buffer** | ≥ 48 hours | **72 hours (4,320 epochs)** | On-board 99 KB SPI flash ring buffer backfill (Test T11, T29) |
| **RF Spectrum Statutory Cap** | ≤ 200 kHz carrier bandwidth | **125 kHz (IN865 Band)** | GSR 564(E) RF spectrum compliance test (Test T18) |
| **Transmitter Duty Cycle** | ≤ 1.0% (ETSI/LoRa convention) | **0.10% (Scouts), 0.56% (Anchors)** | Semtech airtime calculation & 40-day logged run (Test T12, G08) |
| **False Alarm Elimination** | Zero false evacuations from blasting | **99.4% veto rate** | DGMS Circular 7/1997 shift blast log correlation (Test T39) |
| **Surface Spatial Resolution** | Resolve 2m localized fissures | **$\Delta \le 15\text{--}25\text{m}$ (Nyquist grid)** | Knothe influence radius derivation ($r = 75\text{m} \to \Delta \le 25\text{m}$) |
| **Unmonitored Blind Spot ($d_{\text{committed}}$)** | Documented and published | **358 meters** | Largest Empty Circle (LEC) spatial evaluation (Test T20) |
| **Verification Suite Coverage** | Complete functional coverage | **46 / 46 Tests Passing (T1–T46)** | Automated test runner in continuous integration |

---

## 2. Comparison Against Conventional Approaches

| Capability | Manual Theodolite Surveys | Imported Geotechnical Loggers | Satellite InSAR (Radar) | AEGIS Platform |
| :--- | :--- | :--- | :--- | :--- |
| **Unit Capital Cost** | High labor / recurring OPEX | ₹2,00,000 – ₹5,00,000 per unit | Free Sentinel data; ₹10L+ commercial | **₹1,050 / Scout Node** |
| **Panel Deployment Cost** | ₹3,00,000 – ₹5,00,000 / year | ₹40,00,000 – ₹60,00,000 | ₹12,00,000 / year processing | **Algorithmic modular scaling (80–95% lower CAPEX)** |
| **Sampling Frequency** | Every 15 to 30 days | Hourly / Daily logging | 6 to 12-day orbital repeat | **Continuous (60s TDMA superframe)** |
| **Warning Latency** | Weeks (Post-collapse observation) | Hours (Manual offload) | 3 to 7 days processing delay | **< 1.4 seconds (Autonomous siren)** |
| **Weather / Cloud Sensitivity** | Suspended in rain/monsoon | Weatherproof | Severe cloud & monsoon decorrelation | **All-weather IP67 field enclosures** |
| **Spatial Resolution** | Sparse survey pegs (50–100m) | Very sparse (5–10 per panel) | 10–20m pixel (phase noise in brush) | **Dense adaptive physics grid (15–25m)** |
| **False Alarm Discrimination** | N/A (Human visual check) | None (Threshold triggers on blast) | None | **4-band FFT + DGMS blast veto** |
| **Statutory Alignment** | Manual logs | Ad-hoc | Research only | **DGMS Circular 7/1997 & CMR 2017** |

---

## 3. Dynamic Scaling Rules vs. Static Allocations

1. **Node Density Scaling:**
   Total nodes ($N$) scale with panel length ($L$), panel width ($W$), and seam depth ($H$). As overburden depth increases, the influence radius $r = H / \tan\beta$ expands, dictating the maximum permissible grid spacing $\Delta \le r / 2.86$.
2. **Cost Elasticity:**
   Because the individual Scout Node is built from low-cost COTS components (~₹1,050 BOM), expanding coverage to deeper or wider panels incurs small linear hardware additions rather than capital-intensive logger expenditures.
3. **Bandwidth Headroom:**
   The 60-second TDMA superframe accommodates up to 60 leaf nodes per gateway cluster while preserving a channel utilization rate below 8% (well below the 18% ALOHA collapse threshold).
