# Bill of Materials (BOM) & Modular Cost Model

**Module 02 — Sensor Hardware**  
**Cross-References:** [`node-classification.md`](node-classification.md) · [`sensor-suite.md`](sensor-suite.md) · [`power-and-energy.md`](power-and-energy.md) · [Module 09 Cost-Benefit Analysis](../09-deployment-and-impact/cost-benefit-analysis.md) · [Module 00 Key Metrics Summary](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Component Sourcing and Supply Chain Architecture

### Question: What is the hardware sourcing strategy for AEGIS sensor nodes, and how does it prevent supply chain bottlenecks or reliance on foreign proprietary instrumentation?

**Answer:** Every AEGIS node is engineered exclusively around Commercial Off-The-Shelf (COTS) components accessible through domestic Indian electronics distribution channels (e.g., Robu.in, ElectronicsComp, Semikart, and domestic PCB fabrication and injection-molding houses). By eliminating reliance on proprietary imported geotechnical instrumentation (such as RST Instruments, Campbell Scientific, or Sisgeo), AEGIS avoids 8- to 16-week customs delays, import tariff markups of 30% to 50%, and single-vendor maintenance lock-in. All compute, RF, sensing, power management, and enclosure components are multi-sourced from standard catalog footprints, ensuring that any damaged or failed node can be assembled, repaired, or replaced locally using domestic inventory.

### Question: What is the itemized Bill of Materials (BOM) and component unit cost for a Tier 1A Scout Node (Tilt / Inclinometer)?

**Answer:** A Tier 1A Scout Node provides continuous angular tilt and vibration monitoring. Its total unit manufacturing cost is approximately **₹1,450**:

| Component | Part Description / Model | Unit Cost (₹) | Source / Distributor | Technical Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Compute & Wireless** | ESP32-WROOM-32 (Dual Core, 240 MHz) + Semtech SX1262 LoRa module | ₹480 | Robu.in / Domestic distributors | Ultra-low deep-sleep current (5 µA), 865–867 MHz band support, +22 dBm link budget. |
| **Primary Inclinometer** | InvenSense MPU-6050 (3-axis MEMS accelerometer + gyro) | ₹160 | Domestic electronics distributor | 16-bit resolution, ± 2g scale, 0.05° angular tilt resolution, digital I²C interface. |
| **Energy Storage** | 3.2V 1500mAh LiFePO4 18650 cell + TP5000 CC/CV charge management | ₹240 | Robu.in / Semikart | Intrinsically stable lithium chemistry; zero thermal runaway risk up to 60°C; 2000+ lifecycle cycles. |
| **Solar Harvesting** | 1W 5V Monocrystalline Solar Panel (Epoxy sealed) | ₹180 | Domestic solar manufacturer | Delivers ≈ 180 mA peak current under peak sunlight; recharges daily node energy consumption in under 45 minutes. |
| **Enclosure & Mount** | IP67 Polycarbonate Enclosure + PG7 cable glands + rebar clamp | ₹270 | Local injection molding / hardware | Ingress protection against mine slurry, heavy monsoon downpours, and coal dust; UV-stabilized. |
| **Passives & PCB** | Double-sided FR4 PCB, TVS surge diodes, bypass caps, fasteners | ₹120 | Domestic assembly / JLCPCB | Transient voltage protection against electro-static discharge and induced surface lightning spikes. |
| **Total Unit Cost** | — | **₹1,450** | — | **Fully functional autonomous Tier 1A Scout Node.** |

### Question: What is the itemized Bill of Materials (BOM) for Tier 1B (Horizontal Strain) and Tier 1C (Borehole Extensometer) Scout Nodes?

**Answer:** Tier 1B and Tier 1C Scout Nodes use the identical compute, wireless, power, and enclosure base as Tier 1A, integrating specialized secondary transducer interfaces:

* **Tier 1B Scout Node (Horizontal Surface Strain) — Unit Cost: ₹1,850:**
  * Base Electronics Package (ESP32, SX1262, LiFePO4, 1W Solar, IP67 Enclosure, PCB): ₹1,270
  * Precision 16-bit ADC Interface (Texas Instruments ADS1115 with $I^2C$ bus and internal voltage reference): ₹180
  * 10-meter carbon-fiber/invar sliding potentiometer assembly with tensioned spring return: ₹400
  * **Total Unit Cost:** **₹1,850**

* **Tier 1C Scout Node (Multi-Point Borehole Extensometer) — Unit Cost: ₹2,200:**
  * Base Electronics Package (ESP32, SX1262, LiFePO4, 1W Solar, IP67 Enclosure, PCB): ₹1,270
  * Linear Variable Differential Transformer (LVDT) / Potentiometric Interface Board: ₹350
  * Stainless steel down-hole anchor wire clamp, protective collar, and spring guide: ₹580
  * **Total Unit Cost:** **₹2,200**

### Question: What is the itemized Bill of Materials for a Tier 2 Anchor Backbone Node and a Master Edge Gateway Hub?

**Answer:** Higher-tier aggregation and backbone nodes require expanded battery reserves, higher RF throughput, and cellular/satellite backhaul modules:

| Subsystem / Tier | Core Hardware Components | Unit Cost (₹) | Functional Role |
| :--- | :--- | :--- | :--- |
| **Tier 2 Anchor Backbone Node** | ESP32-S3 / ESP32 + Semtech SX1262 LoRa with +22 dBm PA; 3.2V 3000mAh LiFePO4 pack (dual 18650 parallel); 3W 5V solar panel; 3-meter elevated galvanized steel mounting mast. | **₹3,100** | Operates as TDMA cluster head; routes packets from up to 5 child scouts; maintains multi-hop backbone relay to gateway. |
| **Master Edge Gateway Hub** | Industrial Quad-Core ARM SBC (Raspberry Pi CM4 or Rockchip RK3568); SX1302/SX1303 8-channel multi-SF LoRa concentrator HAT; Quectel EC200U 4G LTE Cat-1 cellular modem; 12V 20Ah LiFePO4 battery pack; 20W monocrystalline solar panel + MPPT solar charge controller; 10-meter guyed lattice mast. | **₹12,500 – ₹15,000** | Ingests simultaneous packets from multiple RF channels; runs local physics validation and Kalman filters; syncs with cloud via MQTT over LTE. |

---

## 2. Dynamic Algorithmic Panel Sizing & Cost Formulation

### Question: How does AEGIS size the hardware requirements and total expenditure for an underground longwall panel without imposing arbitrary node count caps?

**Answer:** Hardware quantity is not determined by arbitrary budgetary limits or static baselines. Instead, node deployment is calculated as an algorithmic function of mine panel geometry, overburden depth, rock strata mechanical properties, and the spatial Nyquist criterion:

$$\text{Cost}_{\text{panel}} = N_{\text{scout}} \cdot \bar{C}_{\text{scout}} + N_{\text{anchor}} \cdot C_{\text{anchor}} + N_{\text{gateway}} \cdot C_{\text{gateway}} + C_{\text{mounting}}$$

Where:
1. **Influence Radius ($r$):**
   $$r = \frac{H}{\tan\beta}$$
   Where $H$ is the seam depth ($150\text{m}$ to $400\text{m}$) and $\beta$ is the angle of draw ($60^\circ$ to $65^\circ$).
2. **Spatial Grid Spacing ($\Delta$):**
   $$\Delta \le \frac{r}{2.86}$$
   Ensuring continuous spatial curvature reconstruction without aliasing.
3. **Scout Count ($N_{\text{scout}}$):**
   $$N_{\text{scout}} = \left\lceil \frac{L}{\Delta} \right\rceil \times \left\lceil \frac{W}{\Delta} \right\rceil$$
   Where $L$ and $W$ represent panel surface length and extraction width.
4. **Anchor Backbone Count ($N_{\text{anchor}}$):**
   $$N_{\text{anchor}} = \left\lceil \frac{N_{\text{scout}}}{4} \right\rceil$$
   Anchors are architected for a nominal fan-out ratio of 4:1 child scouts per anchor (`max_children_per_anchor = 5`), reserving 1 slot for dynamic mesh fail-over rerouting.
5. **Gateway Count ($N_{\text{gateway}}$):** Sized for spatial diversity: $\lceil N_{\text{anchor}} / 50 \rceil \ge 1$ (minimum 2 gateways for district panels spanning $>1\text{ km}$).
6. **Average Scout Cost ($\bar{C}_{\text{scout}}$):** Weighted average across the geotechnical sensor mix ($\approx ₹1,650$).

```
+-----------------------------------------------------------------------------------+
|                        DYNAMIC HARDWARE SCALING WORKFLOW                          |
|                                                                                   |
|  [ Panel Geometry (L, W) ] + [ Depth H, Angle of Draw β ]                         |
|                            │                                                      |
|                            ▼                                                      |
|         Calculate Knothe Influence Radius: r = H / tan(β)                         |
|                            │                                                      |
|                            ▼                                                      |
|         Derive Nyquist Spacing: Δ ≤ r / 2.86                                      |
|                            │                                                      |
|                            ▼                                                      |
|         Compute Scout Density: N_scout = ⌈L/Δ⌉ × ⌈W/Δ⌉                            |
|                            │                                                      |
|                            ▼                                                      |
|         Compute Anchor Count: N_anchor = ⌈N_scout / 4⌉ (Fan-out 4:1, max 5)       |
|                            │                                                      |
|                            ▼                                                      |
|         Total Dynamic Panel Cost: C_total = N_scout·C_s + N_anchor·C_a + ...      |
+-----------------------------------------------------------------------------------+
```

### Question: What are representative cost breakdowns for different operational panel dimensions derived using this dynamic scaling formula?

**Answer:** Applying the scaling model across two operational panel scales demonstrates its dynamic adaptation:

#### Scenario A: Localized Pilot Sector (600m × 200m, Overburden H = 150m, Grid Δ = 25m)
* Influence Radius: $r = 150 / \tan(63.4^\circ) = 75\text{ m}$.
* Nyquist Maximum Spacing: $\Delta \le 75 / 2.86 = 26.2\text{ m} \implies$ grid resolution configured to $25\text{ m}$.
* Active Dynamic Sizing: Sized to 37 Scout Nodes and 6 Anchor Relays to cover the critical tensile inflection sub-slice:

| System Component | Quantity | Unit Cost (₹) | Total Cost (₹) | Verification Gate / Specification |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1A Scout Nodes (Tilt)** | 17 | ₹1,450 | ₹24,650 | High-density inflection line monitoring |
| **Tier 1B Scout Nodes (Strain)** | 14 | ₹1,850 | ₹25,900 | Horizontal tension crack detection |
| **Tier 1C Scout Nodes (Extensometer)** | 6 | ₹2,200 | ₹13,200 | Subsurface bedrock anchoring points |
| **Tier 2 Anchor Backbone Nodes** | 6 | ₹3,100 | ₹18,600 | 4:1 nominal fan-out aggregation |
| **Master Edge Gateway Hub** | 1 | ₹8,500 | ₹8,500 | SX1302 8-channel LoRa + 4G LTE modem |
| **Ground Anchor Pegs & Fixtures** | 43 | ₹150 | ₹6,450 | Anti-heave stainless steel ground stakes |
| **Total Pilot Expenditure** | **43 stations** | — | **₹97,300** | **Comprehensive coverage under ₹1 Lakh** |

#### Scenario B: Full Commercial Longwall Panel (1,800m × 280m, Overburden H = 220m, Grid Δ = 35m)
* Influence Radius: $r = 220 / \tan(62^\circ) = 117\text{ m}$.
* Nyquist Maximum Spacing: $\Delta \le 117 / 2.86 = 40.9\text{ m} \implies$ configured to $\Delta = 35\text{ m}$.
* Sizing Output: 327 Scout Nodes, 82 Anchor Relays, 2 Master Edge Gateways:

| Component Tier | Count | Unit Cost (₹) | Subtotal (₹) | Sizing Basis |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1A/1B/1C Scout Nodes** | 327 | ₹1,650 avg | ₹5,39,550 | 100% surface subsidence bowl coverage |
| **Tier 2 Anchor Relays** | 82 | ₹3,500 | ₹2,87,000 | 4:1 fan-out with fail-over reserve |
| **Master Edge Gateway Hubs** | 2 | ₹15,000 | ₹30,000 | Redundant spatial diversity (dual mast) |
| **Ground Monuments & Brackets** | 411 | ₹150 | ₹61,650 | Galvanized rebar ground anchors |
| **Field Spares & Replacement Kit**| — | — | ₹1,10,800 | 10% on-site reserve modules |
| **Full Longwall Network Total** | **411 stations** | — | **₹10,29,000** | **Algorithmically Sized (~₹10.3 Lakhs)** |

---

## 3. Economic Comparison Against Commercial Imported Systems

### Question: How does the capital expenditure (CapEx) and spatial monitoring density of AEGIS compare against commercial geotechnical monitoring instrumentation?

**Answer:** Commercial mining instrumentation systems from imported suppliers (e.g., Campbell Scientific, RST Instruments, Sisgeo) rely on high-cost, low-density proprietary dataloggers and armored field cabling. As a result, coal operators can only afford sparse monitoring points, leaving catastrophic spatial blind spots:

| Metric / Dimension | Traditional Imported Geotechnical System | AEGIS Dynamic Mesh Architecture | Performance / Cost Advantage |
| :--- | :--- | :--- | :--- |
| **Total Capital Cost (CapEx)** | ₹50,00,000 – ₹80,00,000 per panel | **₹10,29,000** (Full scale) / **₹97,300** (Pilot) | **80% to 88% reduction in CapEx** |
| **Monitoring Station Density** | 3 to 6 isolated multi-point stations | **411 active spatial nodes** | **>70× higher spatial sensor density** |
| **Sensor Grid Spacing** | 250m to 500m gaps (aliased) | **25m to 35m physics-aligned grid** | Complies with spatial Nyquist theorem |
| **Trenching & Cabling Cost** | ₹12,00,000 – ₹20,00,000 (cables shear during subsidence) | **₹0 (100% wireless LoRa mesh)** | Zero trenching labor; immune to cable shear |
| **Spares & Maintenance Cost** | Imported proprietary spares (USD 1,000+ each; 8-week lead time) | Domestic COTS modules (₹1,450 unit cost; next-day delivery) | 90% cheaper spares; near-zero downtime |
| **Regulatory Compliance** | Point measurements only; cannot map dynamic surface trough | Continuous surface curvature and strain field mapping | Full DGMS Tech Circular No. 3 compliance |

```
+-----------------------------------------------------------------------------------+
|                        FULL LONGWALL MONITORING COST (INR)                        |
|                                                                                   |
|  Imported Commercial Geotechnical Array (RST / Campbell / Sisgeo)                  |
|  [============================================================] ₹50,00,000 – ₹80,00,000
|                                                                                   |
|  AEGIS Algorithmic Dynamic Mesh (327 Scouts + 82 Anchors)                         |
|  [========] ₹10,29,000                                                            |
|                                                                                   |
|  AEGIS Pilot Sub-Slice (37 Scouts + 6 Anchors + Gateway)                          |
|  [=] ₹97,300                                                                      |
+-----------------------------------------------------------------------------------+
```

---

## 4. Maintenance Architecture, Field Spares, and Rapid Replacement

### Question: How does AEGIS handle field maintenance, module replacement, and environmental survivability in severe open-cast or underground-overburden environments?

**Answer:** The mechanical and electrical packaging is engineered for extreme field survivability and rapid, tool-less maintenance by mine safety personnel:

1. **Modular Enclosure Architecture:**
   * Each scout node is enclosed in a high-impact, UV-stabilized polycarbonate enclosure rated to IP67.
   * Internal electronics are conformal coated with silicone elastomer to prevent moisture condensation and coal dust ingress.
   * The enclosure connects to the ground rebar monument using a quick-release stainless steel clamp, allowing mechanical swap-out in under 3 minutes.
2. **Zero-Soldering Modular Connectors:**
   * External transducers (strain gauge rods, borehole extensometers) interface via industrial M12 screw-locking waterproof connectors or internal spring-cage terminal blocks.
   * Field technicians do not need soldering irons, specialized calibration fixtures, or wire strippers on the subsidence field.
3. **10% On-Site Spare Inventory:**
   * Every operational deployment includes a pre-flashed, calibrated 10% reserve stock of Tier 1A, 1B, and Tier 2 Anchor nodes.
   * Replacement nodes automatically discover the local cluster upon power-up, authenticate via pre-shared cryptographic network keys, and acquire TDMA transmission slots without manual gateway reconfiguration.
