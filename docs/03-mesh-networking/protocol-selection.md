# Protocol Selection & Wireless Decision Matrix

**Module 03 — Mesh Networking**  
**Cross-References:** [`tdma-scheduling.md`](tdma-scheduling.md) · [`wire-format.md`](wire-format.md) · [`spectrum-compliance.md`](spectrum-compliance.md)

---

## 1. Physical Environment & RF Propagation Challenges

Deploying a wireless monitoring network over an active Indian coal mining panel involves severe RF propagation obstacles:
* **Overburden Topography:** Deep depressions, steep spoil dumps, highwalls, and irregular ground subsidence cracks obstruct direct Line-of-Sight (LoS).
* **Atmospheric Attenuation:** Severe particulate coal dust, high humidity, and seasonal monsoon rainfall cause high scattering losses at 2.4 GHz and 5 GHz frequencies.
* **Large Spatial Extents:** Subsidence troughs span hundreds of meters (typically $600\text{m} \times 250\text{m}$ to $1,500\text{m} \times 400\text{m}$), requiring long-range links without high transmit power.
* **Energy Independence:** Nodes rely on small solar panels and batteries, demanding microwatt-level sleep power.

---

## 2. Wireless Protocol Decision Matrix

| Evaluation Criterion | LoRa Sub-GHz (IN865) | Zigbee / IEEE 802.15.4 | Wi-Fi Mesh (802.11s) | Cellular NB-IoT (Per Node) |
| :--- | :--- | :--- | :--- | :--- |
| **Operating Frequency** | **865 – 867 MHz (Sub-GHz)** | 2.4 GHz ISM | 2.4 GHz / 5.8 GHz | Licensed 700–900 MHz |
| **Non-LoS Foliage & Spoil Penetration** | **Superior (Low diffraction loss)** | Poor (High absorption) | Very Poor | Excellent |
| **Max Communication Range** | **2 – 5 km (Field verified)** | 50 – 100 m | 30 – 70 m | 5 – 15 km |
| **Deep Sleep Current** | **< 12 µA** | 20 – 50 µA | > 15 mA | > 15 µA |
| **Recurring SIM / Subscription Cost** | **₹0 (License-free ISM band)** | ₹0 | ₹0 | ₹600 – ₹1,200 / node / year |
| **Network Infrastructure Cost** | **Single Master Gateway** | Mesh repeaters every 50m | Mesh routers every 30m | Telecomm tower dependency |
| **Channel Contention Control** | **Deterministic TDMA Superframe** | CSMA/CA (High collisions) | CSMA/CA (High contention) | Carrier-scheduled |
| **Indian Spectrum Compliance** | **GSR 564(E) Compliant** | WPC ETA required | WPC ETA required | TRAI / Operator governed |

---

## 3. Architectural Rationale for LoRa IN865

AEGIS selects **LoRa Sub-GHz modulation (IN865 band: 865–867 MHz)** as the field communication standard for four technical reasons:

1. **Sub-GHz Diffraction & Earth Penetration:**
   Radio waves at 865 MHz diffract around spoil tips, vegetated berms, and undulating ground far more effectively than 2.4 GHz signals. Path loss measurements confirm a $12\text{ to }18\text{ dB}$ link budget advantage over 2.4 GHz technologies under equivalent transmit power.
2. **Airtime vs. Battery Optimization:**
   Operating at Spreading Factor 7 (SF7) and 125 kHz bandwidth allows a 23-byte sensor packet to be transmitted in just **90.4 ms**. This enables leaf nodes to operate at an ultra-low **0.10% duty cycle**, conserving battery power for over 120 days of solar-free autonomy.
3. **Decoupled Cellular Architecture:**
   Equipping 40+ individual nodes with cellular NB-IoT SIM cards introduces high annual recurring costs, recurring SIM deactivations, and total failure during underground telecommunication outages. AEGIS isolates all field communication to an autonomous, local LoRa mesh. Only the single Master Gateway carries a cellular uplink, lowering connectivity costs by over 95%.
4. **Deterministic TDMA over Raw LoRaWAN:**
   Standard LoRaWAN operates as an uncoordinated ALOHA protocol, which collapses into packet collision storms when dozens of nodes transmit concurrently. AEGIS implements a custom **Time-Division Multiple Access (TDMA) superframe** running directly on the raw LoRa physical layer, guaranteeing collision-free packet delivery and deterministic delivery latencies under 1.4 seconds.
