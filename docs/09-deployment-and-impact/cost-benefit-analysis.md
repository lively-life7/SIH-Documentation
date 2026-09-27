# Cost-Benefit Analysis & Economic Impact

**Module 09 — Deployment & Impact**  
**Cross-References:** [`scalability.md`](scalability.md) · [Module 02 Bill of Materials](../02-sensor-hardware/bill-of-materials.md) · [Module 01 Crisis Landscape](../01-ground-reality/crisis-landscape.md)

---

### Question: How does the capital expenditure (CAPEX) of an AEGIS deployment compare against conventional imported geotechnical instrumentation?

**Answer:** The primary barrier preventing the widespread adoption of continuous geotechnical surveillance across Indian coal mines has been the prohibitive expense of imported instrumentation:

```
+-----------------------------------------------------------------------------------+
|                        CAPITAL EXPENDITURE PER PANEL (INR)                        |
|                                                                                   |
|  Commercial Imported Sensor Array (Campbell / RST / Sisgeo: 5–10 Stations)        |
|  [============================================================] ₹50,00,000 – ₹80,00,000
|                                                                                   |
|  AEGIS Algorithmic Dynamic Mesh (Full Longwall: 327 Scouts + 82 Anchors)          |
|  [========] ₹10,29,000                                                            |
|                                                                                   |
|  AEGIS Pilot Sub-Slice (37 Nodes + Gateway)                                       |
|  [=] ₹97,200                                                                      |
+-----------------------------------------------------------------------------------+
```

* **Commercial Systems:** Proprietary imported monitoring stations (e.g., Campbell Scientific, RST Instruments, Sisgeo) cost between ₹5,00,000 and ₹12,00,000 per station. A modest 5 to 10-station installation requires a capital expenditure of **₹50,00,000 to ₹80,00,000**, leaving massive spatial blind spots across the extraction basin.
* **AEGIS Unit Economics:** Built strictly from domestically available commercial off-the-shelf components, an individual AEGIS Tier 1 Scout Node costs approximately **₹1,050 to ₹1,400**. An Anchor Backbone Relay costs approximately **₹2,200**, and a fully equipped Master Gateway Hub (including a 10m pneumatic mast, solar array, and 125 dB siren) costs **₹18,500**.
* **Pilot Deployment Investment (37 Nodes):** Sized for an active $600\text{ m} \times 200\text{ m}$ traveling profile sub-slice, a 37-node pilot array (37 Scouts, 6 Anchors, 1 Gateway) costs **₹97,200** total.
* **Full-Panel Array Investment (409 Nodes):** Scaling across an entire $1,800\text{ m} \times 300\text{ m}$ continuous mechanized longwall panel requires 327 Scouts, 82 Anchors, and 2 Redundant Master Gateways, totaling **₹10,29,000**.
* **Capital Savings:** AEGIS achieves an **80% to 98% reduction in initial capital expenditure**. For the price of a single imported commercial borehole station, an operator can equip multiple entire longwall extraction districts with dense, sub-second wireless monitoring.

---

### Question: What recurring operational expenditure (OPEX) savings does AEGIS deliver over traditional mining survey workflows?

**Answer:** Conventional ground monitoring relies on manual surveying crews and commercial satellite radar tasking contracts, incurring substantial recurring operational costs:

| Operational Expense Category | Conventional Survey Practices | AEGIS Autonomous Platform | Net Annual Savings Per Colliery |
| :--- | :--- | :--- | :--- |
| **Manual Survey Crews** | 4-person surveying team, optical instruments, dedicated 4WD vehicle fuel and maintenance. | Autonomous 60-second wireless telemetry; automated cloud ingestion. | **₹14,00,000 / year** |
| **Commercial InSAR Radar Contracts** | Commercial satellite radar tasking, raw interferogram licensing, and external processing fees. | Local real-time physics-constrained ground mesh; zero orbital licensing dependencies. | **₹8,00,000 / year** |
| **Consumables & Battery Servicing** | Disposable primary lithium batteries requiring quarterly field replacement and disposal. | Integrated 1W monocrystalline solar panel + $\text{LiFePO}_4$ battery ($> 5\text{-year}$ continuous lifespan). | **₹1,50,000 / year** |
| **Total Recurring Annual Savings** | — | — | **₹23,50,000 / year / mine** |

Beyond direct line-item reductions, automating surface displacement logging frees colliery surveying engineers to focus on underground face alignment and statutory ventilation surveys.

---

### Question: How does AEGIS protect high-value national surface infrastructure and mitigate multi-crore industrial disasters?

**Answer:** While day-to-day OPEX savings justify the platform, the primary economic return is derived from averting catastrophic geotechnical failures:

1. **Railway Track Derailment & Realignment Mitigation:**
   Differential surface tilt exceeding $3.0\text{ mm/m}$ twists railway track geometry, inducing severe derailment hazards. Over the past decade, Indian Railways and Coal India subsidiaries have expended over **₹313 Crores** on emergency track realignments, ballast packing, and operational speed restrictions across subsiding panels in the Raniganj and Jharia coalfields. Providing up to 8.9 days of advance warning enables rail authorities to schedule controlled track tamping and dynamic ballast adjustments without halting rail traffic or risking derailments.
2. **Underground Face Inundation & Aquifer Breaches:**
   Tensile fracturing through confining shale aquitards permits surface water bodies and shallow aquifers to penetrate underground workings. Inundation of a mechanized longwall face damages continuous shearers, armored face conveyors, and powered roof supports, costing between **₹15 Crores and ₹50 Crores** in salvage operations and months of lost production. Detecting tensile strain inflection ($\varepsilon \ge 1500\ \mu\varepsilon$) days before fractures propagate to the surface allows colliery managers to regulate face advance rates and initiate proactive goaf backfilling.
3. **High-Voltage Transmission Pylon Protection:**
   Ground tilt beneath electrical grid transmission towers causes structural tower twisting and insulator flashovers. Underpinning or relocating a compromised high-voltage transmission tower costs between **₹40 Lakhs and ₹80 Lakhs**. Real-time angular tilt surveillance enables pre-emptive guy-wire re-tensioning.
4. **Avoidance of Unplanned Statutory Production Halts:**
   When DGMS inspectors identify unmonitored surface fissures near civil infrastructure, they issue statutory prohibition orders under Section 22 of the Mines Act, 1952. Unplanned production shutdowns cost deep mechanized longwalls between **₹30 Lakhs and ₹60 Lakhs per day** in lost revenue. Continuous digital compliance logs prevent arbitrary regulatory shutdowns.

---

### Question: What is the realistic Return on Investment (ROI) timeline for a colliery operator?

**Answer:** Because capital costs are minimized through indigenous hardware and recurring operational expenses are drastically lowered, the financial payback period is exceptionally rapid:

* **Pilot Network Deployment (37 Nodes, ₹97,200 CAPEX):** Payback achieved in **under 2 months** purely from operational labor and vehicle savings.
* **Full-Scale Longwall Deployment (409 Nodes, ₹10,29,000 CAPEX):** Payback achieved in **under 6 months** through operational savings alone ($\approx ₹1,95,800\text{ saved per month}$).
* **Catastrophic Event Payback:** Preventing a single railway slow order, transmission tower tilt incident, or longwall water inrush delivers an immediate **10× to 50× return on investment** on the day the incident is averted.
