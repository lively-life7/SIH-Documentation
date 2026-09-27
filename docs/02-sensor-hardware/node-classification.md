# Node Classification & Hardware Hierarchy

**Module 02 — Sensor Hardware**  
**Cross-References:** [`sensor-suite.md`](sensor-suite.md) · [`bill-of-materials.md`](bill-of-materials.md) · [`power-and-energy.md`](power-and-energy.md) · [Module 03 Mesh Networking](../03-mesh-networking/routing-and-failover.md) · [Module 04 Knothe Model](../04-physics-engine/knothe-model.md)

---

## 1. Asymmetric Node Hierarchy and Architectural Rationale

### Question: Why does AEGIS implement a multi-tiered, asymmetric node hierarchy instead of deploying identical, monolithic sensor packages across the mining panel?

**Answer:** Deploying identical, over-specified sensor packages uniformly across an underground mining panel is geotechnical and economic folly. Ground deformation over an extraction panel is fundamentally heterogeneous:
1. **Mechanical Differentiation:** The central trough bottom experiences large vertical settlement but near-zero horizontal tensile strain, whereas the perimeter inflection band undergoes severe horizontal tension and curvature with minimal initial settlement.
2. **Economic Optimization:** A monolithic node integrating multi-axis tilt, high-precision linear extensometers, multi-channel LoRa concentrators, and 4G modems costs upwards of ₹25,000 per station. Deploying 400 such nodes across a standard $1,800\text{m} \times 280\text{m}$ longwall panel would require over ₹1 Crore.
3. **RF and Energy Efficiency:** If all nodes functioned as full mesh repeaters or cellular transmitters, channel contention, RF collision, and battery drain would cause network collapse.

AEGIS separates functional responsibilities into three distinct tiers: high-density low-cost **Scout Nodes (Tier 1)** that monitor specific localized ground dynamics, elevated **Anchor Backbone Relays (Tier 2)** that aggregate local clusters, and a high-performance **Master Edge Gateway Hub (Tier 3)** that handles edge analytics, backhaul, and autonomous emergency alerting.

```
+-----------------------------------------------------------------------------------+
|                        AEGIS 3-TIER HARDWARE ARCHITECTURE                         |
|                                                                                   |
|                           [ MASTER EDGE GATEWAY ] (Tier 3)                        |
|                     SX1302 Multi-Channel LoRa Concentrator                        |
|                   Industrial SBC · Dual 4G LTE · 125 dB Siren                     |
|                                        │                                          |
|                 ┌──────────────────────┴──────────────────────┐                   |
|                 ▼                                             ▼                   |
|    [ BEDROCK REFERENCE ANCHOR ]                   [ BACKBONE RELAY ANCHOR ]       |
|             (Tier 2A)                                     (Tier 2B)               |
|      Stable Bedrock Datum                          ESP32 + SX1262 LoRa Relay      |
|      Common-Mode Noise Baseline                    TDMA Cluster Head (SF8)        |
|                 │                                             │                   |
|         ┌───────┴───────┐                             ┌───────┴───────┐           |
|         ▼               ▼                             ▼               ▼           |
|   [ SCOUT 1A ]    [ SCOUT 1B ]                  [ SCOUT 1C ]   [ CRACK TRIP ]     |
|   Tilt / Inclin.  Rod Strain                    Wire Extenso.  Conductive Trace   |
|   Trough Bottom   Inflection Band               Pillar Edge    Surface Fissure    |
+-----------------------------------------------------------------------------------+
```

---

## 2. Tier 1: Scout Nodes (Dense Surface Coverage)

### Question: What are the engineering specifications, mechanical mountings, and physical placement criteria for Tier 1 Scout Nodes?

**Answer:** Tier 1 Scout Nodes form the high-density surface monitoring grid deployed directly across the active subsidence area. Each node is self-contained in an IP67 UV-stabilized polycarbonate enclosure mounted on a stainless-steel ground-driven monument peg, powered by a 3.2V 1500mAh LiFePO4 battery and a 1W monocrystalline solar panel:

| Tier | Transducer / Interface | Measurement Parameter | Dynamic Spatial Placement | Geotechnical Role & Safety Logic |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1A (Tilt Inclinometer)** | InvenSense MPU-6050 (3-axis MEMS accelerometer + gyro) | Angular tilt (θ_x, θ_y) in 0.05° resolution; 3-axis vibration | Central trough basin and outer subsidence boundary | Maps continuous spatial slope across the subsidence bowl. Tilt alone does not trip emergency alarms due to diurnal thermal tilt artifacts. |
| **Tier 1B (Horizontal Rod Strain)** | 10m carbon-fiber/invar reference rod + linear slide pot + TI ADS1115 16-bit ADC | Differential horizontal displacement (Δ U_x) in 0.1 mm resolution; ground strain (ε) | Perimeter inflection zone (± 0.4 r from extraction boundary) | Primary early-warning indicator. Measures horizontal ground stretching weeks before visible surface fissures appear. |
| **Tier 1C (Wire Extensometer)** | 30m high-tensile invar wire rotary draw-wire optical/potentiometric encoder | Long-baseline surface elongation in 0.5 mm resolution | Straddling the unmined barrier pillar and panel edge | Directly satisfies DGMS technical requirements for monitoring absolute distance changes between ground stations across major shear boundaries. |
| **Crack Trip Scout** | Continuous conductive silver-palladium break-wire trace or microswitch | Binary physical fissure rupture (0 = intact, 1 = sheared) | Known geological fault outcrops and panel hinge lines | Hardware interrupt-driven instantaneous trigger. Upon physical rupture, awakens from deep sleep and immediately broadcasts an emergency priority frame. |

---

## 3. Tier 2: Anchor & Relay Backbone Nodes

### Question: What is the function of Tier 2A Bedrock Reference Anchors, and how do they establish common-mode rejection (CMR)?

**Answer:** A primary failure mode of geotechnical surface monitoring is false alarms induced by seasonal soil swelling, surface moisture variations, and thermal expansion of ground monuments. Tier 2A Bedrock Reference Anchors solve this:
1. **Physical Anchoring:** Installed outside the active mining panel at a distance $D \ge 1.5 \times r$ (where $r$ is the Knothe influence radius), anchored deep into solid, non-yielding bedrock.
2. **True Geotechnical Datum:** Because this station sits outside the angle of draw, it experiences zero mining-induced displacement.
3. **Common-Mode Noise Rejection (CMR):** Any registered tilt or displacement at Tier 2A represents pure environmental noise (e.g., thermal diurnal cycles, monsoonal clay swelling, or electronic aging). The C7 backend filtering pipeline subtracts this reference noise vector from all active Tier 1 Scout signals, isolating genuine subsurface subsidence.

### Question: What are the hardware specifications and networking duties of Tier 2B Backbone Relay Nodes?

**Answer:** Tier 2B Backbone Relay Nodes form the resilient mesh transport infrastructure connecting dense Scout clusters to the Gateway Hub:
* **Processing & Radio:** Espressif ESP32-S3 microcontroller coupled to a Semtech SX1262 LoRa transceiver equipped with a $+22\text{ dBm}$ power amplifier and an elevated $+5\text{ dBi}$ omnidirectional collinear antenna.
* **Cluster Head Aggregation:** Each Tier 2B Anchor acts as a TDMA cluster head managing a local cell of up to 5 Tier 1 Scout nodes (`max_children_per_anchor = 5`, operating with 4 nominal child nodes and 1 dynamically reserved failover slot).
* **Dual-Channel Protocol Operation:** Ingests 23-byte leaf packets from child Scouts over short-range, low-power SF7 links, bundles them into aggregated backbone frames, and relays them across longer distances to the Master Gateway over the SF8 backbone channel.
* **Power Subsystem:** 3.2V 3000mAh LiFePO4 battery pack (dual parallel 18650 cells) charged by an elevated 3W monocrystalline solar panel, providing 14 days of continuous operation under total zero-sunlight conditions.

---

## 4. Tier 3: Master Edge Gateway Hub

### Question: What are the compute, backhaul, and autonomous fail-safe capabilities of the Tier 3 Master Edge Gateway Hub?

**Answer:** The Tier 3 Master Edge Gateway Hub serves as the central command node for the local panel deployment:
* **High-Throughput RF Concentrator:** Features a Semtech SX1302/SX1303 8-channel LoRa concentrator HAT capable of demodulating up to 8 simultaneous packets across different spreading factors and frequency channels.
* **Edge Compute Engine:** Powered by an industrial quad-core ARM Cortex-A72 Single-Board Computer (Raspberry Pi CM4 or Rockchip RK3568) with 4GB RAM and 32GB industrial eMMC storage.
* **Local Safety Autonomy:** Runs the local Knothe forward kinematics validation and Kalman state estimation directly on-site. If cellular backhaul is severed during a catastrophic slope failure, the gateway operates completely autonomously.
* **Audible & Visual Alert Hardware:** Directly drives a heavy-duty industrial dry-contact relay wired to an omnidirectional 125 dB motor-driven siren and a 360-degree flashing strobe beacon mounted on its 10-meter guyed pneumatic mast.
* **Redundant Dual Backhaul:** Integrates a Quectel EC200U dual-SIM 4G LTE Cat-1 cellular modem with automatic carrier fallback, supplemented by a local ruggedized M12 Ethernet port for connection to the mine’s fiber backbone.

---

## 5. Dynamic Placement Logic Across a Panel

### Question: How does AEGIS determine the spatial placement of different node tiers across an active extraction panel based on Knothe subsidence physics?

**Answer:** Sensor nodes are not deployed in a rigid, blind Cartesian grid. Spatial placement is dynamically governed by the rock mass curvature profile derived from Knothe theory:

```
+-----------------------------------------------------------------------------------+
|                        CROSS-SECTIONAL PLACEMENT STRATEGY                         |
|                                                                                   |
|  Exterior Zone       Perimeter Inflection Zone     Central Trough Basin           |
|  (x > 1.5 r)         (-0.4 r ≤ x ≤ +0.4 r)         (Bottom of Trough)             |
|                                                                                   |
|  [Tier 2A Bedrock]   [Tier 1B Rod Strain Nodes]    [Tier 1A Tilt Nodes]           |
|  [Reference Anchor]  [Tier 1C Wire Extensometers]  [Spatial Inclinometers]        |
|  Zero Subsidence     Maximum Tensile Curvature     Maximum Settlement, Zero Tilt  |
|  Datum               Crack Trip Interrupters                                      |
+-----------------------------------------------------------------------------------+
```

1. **Perimeter Inflection Zone (Peak Curvature $\partial^2 S / \partial x^2$):**
   Located at $\pm r / \sqrt{2\pi} \approx \pm 0.4 r$ from the extraction boundary. This band experiences peak horizontal tensile strain and differential tilt. Sensor density is maximized here, concentrating Tier 1B horizontal strain rods, Tier 1C wire extensometers, and Crack Trip break-wires at close Nyquist intervals ($\Delta \le r / 2.86$).
2. **Central Trough Basin (Flat Settlement Zone):**
   Over the extracted center, vertical subsidence is maximal ($S → S_{\max}$) but curvature and horizontal strain approach zero. Tier 1A tilt inclinometers are spaced at wider intervals to track bulk elevation settlement and block rotation without unnecessary hardware over-allocation.
3. **Exterior Reference Zone ($x \ge 1.5 r$):**
   Outside the strata angle of draw, Tier 2A reference anchors provide the zero-displacement geotechnical baseline.
