# Bill of Materials (BOM) & Modular Cost Model

**Module 02 — Sensor Hardware**  
**Cross-References:** [`node-classification.md`](node-classification.md) · [`sensor-suite.md`](sensor-suite.md) · [Module 09 Cost-Benefit Analysis](../09-deployment-and-impact/cost-benefit-analysis.md)

---

## 1. Single-Node Unit Economics

Every node is designed exclusively around Commercial Off-The-Shelf (COTS) components sourced through Indian electronic distribution networks (e.g., Robu.in, ElectronicsComp, domestic fabricators), ensuring zero dependence on foreign specialty suppliers.

### Tier 1A Scout Node (Tilt Inclinometer) — Unit Cost: ~₹1,450

| Component | Part Description / Model | Unit Cost (₹) | Source / Distributor |
| :--- | :--- | :--- | :--- |
| **Compute & Wireless** | ESP32-WROOM-32 (Dual Core, 240 MHz) + SX1262 LoRa module | ₹480 | Robu.in / Waveshare India |
| **Primary Inclinometer** | InvenSense MPU-6050 (3-axis MEMS accelerometer + gyro) | ₹160 | Domestic electronics distributor |
| **Power Storage** | 3.2V 1500mAh LiFePO4 18650 cell + TP5000 charge board | ₹240 | Robu.in |
| **Solar Harvesting** | 1W 5V Monocrystalline Solar Panel (Epoxy sealed) | ₹180 | Domestic solar manufacturer |
| **Enclosure & Mount** | IP67 Polycarbonate Enclosure + PG7 cable glands + rebar clamp | ₹270 | Local injection molding / hardware |
| **Passives & Hardware** | Resistors, TVS transient diodes, decoupling capacitors, PCB | ₹120 | JLCPCB / Local assembly |
| **Total Unit Cost** | — | **₹1,450** | — |

### Tier 1B Scout Node (Horizontal Strain) — Unit Cost: ~₹1,850
* Base electronics package (ESP32, SX1262, LiFePO4, Solar, Enclosure): ₹1,270
* Precision 16-bit ADC (TI ADS1115 external module): ₹180
* 10-meter carbon-fiber/invar measurement rod with slide potentiometer assembly: ₹400
* **Total Unit Cost:** **₹1,850**

---

## 2. Panel-Scale BOM (Adriyala Coalfield Baseline: ₹97,200)

For the Singareni Collieries Company Limited (SCCL) Adriyala Longwall Project baseline ($600\text{m} \times 200\text{m}$ extraction panel at $375\text{m}$ depth), the dynamic sizing engine yields **37 Scout Nodes, 6 Anchor Backbone Relays, and 1 Master Gateway Hub**.

| System Component | Quantity | Unit Cost (₹) | Total Cost (₹) | Statutory & Verification Gate |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1A Scout Nodes (Tilt)** | 17 | ₹1,450 | ₹24,650 | Passes Gate G03 (Cap: ₹1,00,000) |
| **Tier 1B Scout Nodes (Strain)** | 14 | ₹1,850 | ₹25,900 | Passes Gate G03 |
| **Tier 1C Scout Nodes (Extensometer)** | 6 | ₹2,200 | ₹13,200 | Direct displacement compliance |
| **Tier 2 Anchor Backbone Nodes** | 6 | ₹3,100 | ₹18,600 | Includes 1W solar array & mast clamp |
| **Master Edge Gateway Hub** | 1 | ₹8,500 | ₹8,500 | 10m mast, SX1302, 4G modem, 20W solar |
| **Ground Anchor Pegs & Stainless Fixtures** | 43 | ₹150 | ₹6,450 | High-tensile ground mounting |
| **Total System Investment** | — | — | **₹97,200** | **Passed Gate G03 (< ₹1 Lakh ceiling)** |

---

## 3. Modular Cost Scaling Model

Total deployment cost is not a static bundle. It is an algorithmic function of mine panel geometry:

$$\text{Cost}_{\text{panel}} = N_{\text{scout}} \cdot \bar{C}_{\text{scout}} + N_{\text{anchor}} \cdot C_{\text{anchor}} + N_{\text{gateway}} \cdot C_{\text{gateway}} + C_{\text{mounting}}$$

Where:
* $N_{\text{scout}}$ scales dynamically with panel surface area ($L_{\text{panel}} \times W_{\text{panel}}$) and overburden depth ($H$).
* $\bar{C}_{\text{scout}} \approx ₹1,650$ (weighted average across Tier 1A, 1B, and 1C configurations).
* Anchor and gateway investments are amortized across adjacent panels, providing economies of scale as additional panels are developed.

---

## 4. Economic Comparison Against Imported Systems

```
+-----------------------------------------------------------------------------------+
|                        PANEL MONITORING COST COMPARISON                           |
|                                                                                   |
|  Imported Commercial Geotechnical Array (RST / Campbell / Sisgeo)                  |
|  [============================================================] ₹40,00,000 – ₹60,00,000
|                                                                                   |
|  AEGIS Indigenous Dynamic Mesh Platform (37 Nodes + Gateway)                      |
|  [=] ₹97,200                                                                      |
+-----------------------------------------------------------------------------------+
```

* **Cost Reduction:** **98.1% lower capital expenditure** than imported sensor systems.
* **Density Advantage:** At ₹97,200, AEGIS provides **37 active monitoring stations**, compared to the 3–5 sparse stations typically purchased with imported budgets.
