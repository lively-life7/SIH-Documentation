# Field Validation & Coalfield Pilot Trial Plan

**Module 08 — Verification**  
**Cross-References:** [`test-register.md`](test-register.md) · [`build-order.md`](build-order.md) · [Module 09 Installation](../09-deployment-and-impact/installation-procedure.md)

---

## 1. Target Pilot Deployment Site

To validate AEGIS under active production conditions, a formal field trial plan is established in partnership with Coal India subsidiaries:

* **Primary Candidate Site:** **Adriyala Longwall Project**, Ramagundam-III Area, The Singareni Collieries Company Limited (SCCL), Godavari Valley Coalfield, Telangana.
  * *Geotechnical Profile:* Deep mechanized longwall face ($H = 375\text{m}$ overburden depth, $3.0\text{m}$ extraction height in Seam No. 1).
  * *Operational Advantage:* Established continuous extraction rates ($3.5\text{ to }4.5\text{ m/day}$) with extensive baseline geotechnical borehole records.
* **Secondary Candidate Site:** **Jhanjra Project**, Eastern Coalfields Limited (ECL), Raniganj Coalfield, West Bengal.

---

## 2. Trial Deployment Architecture

* **Pilot Hardware Array:** 37 Scout Nodes (17 Tier 1A Tilt, 14 Tier 1B Strain, 6 Tier 1C Extensometers) arranged along a traveling profile cross covering an active $600\text{m} \times 200\text{m}$ panel sub-slice. (Note: for full-panel deployments, node counts scale algorithmically, e.g., 327 Scouts + 82 Anchors for a 1.8 km face).
* **Backbone Infrastructure:** 6 Anchor Backbone Relays (sized dynamically for the pilot cluster at nominal fan-out 4, `max_children_per_anchor = 5`) + 1 Master Gateway Hub mounted on a 10m pneumatic mast situated at the surface colliery substation.
* **Trial Duration:** **40 continuous operating days**, corresponding to approximately $160\text{ meters}$ of continuous longwall face advance.

---

## 3. Side-by-Side Benchmarking Methodology

To scientifically validate AEGIS against statutory mining standards, an independent ground control survey will run concurrently on the same panel:

```
+-----------------------------------------------------------------------------------+
|                        SIDE-BY-SIDE FIELD BENCHMARKING                            |
|                                                                                   |
|  [ GROUND DISPLACEMENT EVENT ]                                                    |
|         │                                                                         |
|         ├───> [ AEGIS Autonomous Wireless Mesh ]                                  |
|         │     - Continuous 60-second sampling                                     |
|         │     - Micro-strain & tilt derivative tracking                           |
|         │     - Automated <1.4s siren response                                    |
|         │                                                                         |
|         └───> [ Colliery Total Station Survey Team ]                              |
|               - Manual weekly leveling runs across control pegs                  |
|               - Optical theodolite angle measurements                             |
|               - Static post-hoc spreadsheet plotting                              |
+-----------------------------------------------------------------------------------+
```

1. **Co-located Measurement Control Pegs:** Each Tier 1B Scout station is mechanically coupled to an official DGMS brass survey monument, allowing direct numerical comparison.
2. **Weekly Correlation Audits:** At the conclusion of each weekly manual survey round, the Colliery Survey Officer extracts leveling data to cross-correlate against AEGIS `nodes.csv` time-series.

---

## 4. Key Success Criteria for DGMS Pilot Sign-Off

| Metric | Minimum Acceptable Threshold | Measurement Method |
| :--- | :--- | :--- |
| **Displacement Correlation ($R^2$)** | **R² ≥ 0.90** vs. manual leveling | Linear regression against Total Station survey |
| **Advance Crack Prediction** | **≥ 7.0 days** prior to visible rupture | Time delta between $\varepsilon > 1500\ \mu\varepsilon$ and visual crack |
| **False Blast Alarms** | **Zero false evacuation alarms** | 100% correlation with DGMS shift blast register |
| **Network Availability / Uptime** | **≥ 99.5% packet delivery** | Total received frames vs. expected frames over 40 days |
| **Emergency Trigger Latency** | **< 1.4 seconds** | Hardware event injection to siren contact closure |
| **Hardware Enclosure Integrity** | Zero moisture or dust ingress | Post-trial inspection of IP67 enclosures and glands |
