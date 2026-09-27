# Node Classification & Hardware Hierarchy

**Module 02 — Sensor Hardware**  
**Cross-References:** [`sensor-suite.md`](sensor-suite.md) · [`bill-of-materials.md`](bill-of-materials.md) · [Module 03 Mesh Networking](../03-mesh-networking/routing-and-failover.md)

---

## 1. Physical Node Hierarchy

AEGIS deploys an asymmetric node hierarchy across the mining panel. Rather than deploying identical, over-specified sensor packages at every coordinate, hardware is classified into specialized tiers based on geotechnical role, sampling demand, and communication load.

```
+-----------------------------------------------------------------------------------+
|                            MASTER GATEWAY (Tier 3)                                |
|  - 10m mast, SX1302 concentrator, 4G/NB-IoT uplink, hardware siren relay          |
+-----------------------------------------+-----------------------------------------+
                                          |
                      +-------------------+-------------------+
                      |                                       |
                      v                                       v
+-----------------------------------------+   +-------------------------------------+
|    BEDROCK REFERENCE ANCHOR (Tier 2A)   |   |   DEEP BOREHOLE ANCHOR (Tier 2B)    |
|  - Zero-subsidence reference peg        |   |   Multi-point extensometer interface|
|  - SX1262 LoRa relay & time beacon sync |   |   Common-Mode Rejection (CMR) anchor|
+---------------------+-------------------+   +------------------+------------------+
                      |                                          |
          +-----------+-----------+                  +-----------+-----------+
          |                       |                  |                       |
          v                       v                  v                       v
+-------------------+   +-------------------+  +-------------------+   +--------------------+
|  SCOUT TIER 1A    |   |  SCOUT TIER 1B    |  |  SCOUT TIER 1C    |   |  CRACK TRIP SCOUT  |
|  MPU-6050 Tilt    |   |  10m Rod Strain   |  |  30m Extensometer |   |  Conductive trace  |
|  Trough bottom    |   |  Mid-slope shear  |  |  Inflection zone  |   |  Hinge-line break  |
+-------------------+   +-------------------+  +-------------------+   +--------------------+
```

---

## 2. Detailed Tier Specifications

### Tier 1: Scout Nodes (Dense Surface Coverage)
Scout nodes represent the dense field mesh deployed directly across the subsiding terrain. They are self-contained, solar-rechargeable units housed in IP67 polycarbonate enclosures mounted onto ground-driven rebar pegs.
* **Tier 1A (Tilt Inclinometer):**
  * *Transducer:* InvenSense MPU-6050 (3-axis MEMS accelerometer + gyroscopic temperature sensor).
  * *Placement:* Located across the central trough bottom and outer basin perimeter where slope changes are gradual.
  * *Role:* Provides secondary spatial gradient verification. It never trips alarms on its own due to thermal sensitivity.
* **Tier 1B (Horizontal Rod Strain):**
  * *Transducer:* 10-meter carbon-fiber/invar reference rod anchored at one end, coupled to a linear slide potentiometer with an ADS1115 16-bit ADC at the sensor head.
  * *Placement:* Deployed across the mid-slope inflection band where differential horizontal displacement ($\partial U_x / \partial x$) peaks.
  * *Role:* Primary early warning signal. Detects tensile stretching weeks before surface fissures open.
* **Tier 1C (Wire Extensometer):**
  * *Transducer:* 30-meter high-tensile invar wire rotary draw-wire extensometer.
  * *Placement:* Straddles the boundary between the unmined pillar and the active extraction face.
  * *Role:* Directly answers the statutory requirement for measuring relative distance changes between ground stations.

### Tier 2: Anchor & Relay Nodes (Reference & Backbone)
* **Tier 2A (Bedrock Reference Anchor):**
  * *Placement:* Located at least $1.5\times$ the Knothe influence radius ($r$) away from the extraction boundary, anchored directly into solid, undisturbed bedrock or non-mining ground.
  * *Role:* Physical ground truth reference. Because this ground cannot move from mining activity, any registered deflection represents pure common-mode thermal drift, soil moisture swelling, or electronic aging. The C7 pipeline subtracts this signal from all Scout nodes.
* **Tier 2B (Backbone Relays):**
  * *Hardware:* ESP32-WROOM-32 paired with a high-efficiency Semtech SX1262 LoRa transceiver, powered by a 1W monocrystalline solar panel and a 3.2V 1500mAh LiFePO4 battery.
  * *Role:* Aggregates 23-byte leaf frames from 5–6 nearby Scout nodes and transmits bundled frames across the SF8 backbone channel to the Master Gateway.

### Tier 3: Master Edge Gateway
* **Hardware:** Industrial Single-Board Computer / Raspberry Pi 4 paired with a Semtech SX1302 8-channel LoRa concentrator hat, backed by a 20W solar array, a 12V 12Ah LFP battery bank, and a 10-meter pneumatic mast.
* **Uplink:** Dual-SIM 4G LTE Cat-1 / NB-IoT modem with local fallback to ruggedized Ethernet.
* **Alarm Interfaces:** Direct dry-contact relay output triggering a 125 dB omnidirectional motor siren and industrial strobe light, operable even during complete cellular backhaul loss.

---

## 3. Dynamic Placement Logic Across a Panel

Node distribution is strictly governed by subsidence physics rather than arbitrary grid spacing:
1. **Inflection Zone Focus:** Spatial density is highest along the perimeter inflection contour where curvature $\partial^2 S / \partial x^2$ transitions from convex to concave, maximizing horizontal tension.
2. **Buffer Zones:** Peripheral buffer zones carry lower node density, reserving radio time slots for critical extraction sectors.
