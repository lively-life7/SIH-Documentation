# Protocol Selection & Wireless Decision Matrix

**Module 03 — Mesh Networking**  
**Cross-References:** [`tdma-scheduling.md`](tdma-scheduling.md) · [`wire-format.md`](wire-format.md) · [`spectrum-compliance.md`](spectrum-compliance.md) · [`routing-and-failover.md`](routing-and-failover.md) · [Module 00 Key Metrics Summary](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Physical Environment & RF Propagation Challenges

### Question: What unique RF propagation hazards characterize active underground coal mining panels and open-cast highwalls, and why do conventional commercial wireless standards fail in this environment?

**Answer:** Geotechnical monitoring across active mining sectors imposes severe electromagnetic and environmental constraints that disqualify standard commercial wireless protocols:

1. **Non-Line-of-Sight (NLOS) Topography:**
   Overburden terrain over longwall panels and open-pit benches features severe ground undulations, active subsidence scarps ($0.5\text{m}$ to $2.0\text{m}$ vertical steps), continuous spoil dumps, and elevated perimeter bunds. Direct Line-of-Sight (LoS) is rarely maintained across hundreds of meters.
2. **Atmospheric and Particulate Attenuation:**
   Suspended coal dust, high particulate matter ($PM_{10}$ and $PM_{2.5}$ exceeding $500\ \mu\text{g/m}^3$), ambient humidity ($>85\%$), and seasonal monsoon downpours induce heavy scattering and absorption losses. Frequencies at $2.4\text{ GHz}$ and $5.8\text{ GHz}$ suffer severe signal attenuation and multipath fading under these conditions.
3. **Expansive Spatial Extents:**
   A standard Indian coal extraction panel spans lengths of $1,000\text{m}$ to $2,500\text{m}$ and extraction widths of $200\text{m}$ to $400\text{m}$. Covering this expanse requires long-range wireless links without exceeding legal radiated power limits.
4. **Energy Independence:**
   Field nodes cannot access grid power or trench miles of vulnerable cabling (which shear as the surface subsides). They must operate autonomously for over 5 years on compact solar-battery packages, restricting the radio layer to microwatt-level sleep power.

---

## 2. Wireless Protocol Decision Matrix

### Question: How does Sub-GHz LoRa (IN865) compare across physical and network layer parameters against Zigbee (IEEE 802.15.4), Wi-Fi Mesh (802.11s), and direct Cellular NB-IoT?

**Answer:** Evaluating candidate wireless standards against operational mining criteria demonstrates the decisive superiority of Sub-GHz LoRa:

| Evaluation Criterion | LoRa Sub-GHz (IN865) | Zigbee / IEEE 802.15.4 | Wi-Fi Mesh (802.11s) | Cellular NB-IoT (Per Node) |
| :--- | :--- | :--- | :--- | :--- |
| **RF Carrier Frequency** | **865 – 867 MHz (Sub-GHz)** | $2.4\text{ GHz}$ ISM | $2.4\text{ GHz} / 5.8\text{ GHz}$ | Licensed $700\text{–}900\text{ MHz}$ |
| **Diffraction & NLOS Penetration** | **Superior ($\lambda \approx 34.6\text{ cm}$)** | Poor ($\lambda \approx 12.5\text{ cm}$, high shadowing) | Very Poor (Severe multipath) | Excellent (High power cellular) |
| **Maximum Range (Field Verified)** | **2 to 5 km (Ground-to-ground)** | $50\text{ to }100\text{ m}$ | $30\text{ to }70\text{ m}$ | $5\text{ to }15\text{ km}$ (Requires cell tower) |
| **Deep Sleep Current** | **$< 12\ \mu\text{A}$ (SX1262 sleep)** | $20\text{ to }50\ \mu\text{A}$ | $> 15\text{ mA}$ (Constant listen) | $> 15\ \mu\text{A}$ (PSM mode) |
| **Active TX Current / Duration** | **110 mA for 90.4 ms** | $35\text{ mA}$ for $20\text{ ms}$ | $300\text{ mA}$ for $50\text{ ms}$ | $220\text{ mA}$ for $5\text{ to }25\text{ seconds}$ |
| **Recurring SIM / Subscription Cost** | **₹0 (License-free WPC ISM band)** | ₹0 | ₹0 | ₹600 – ₹1,200 / node / year |
| **Network Infrastructure Cost** | **Single Master Gateway Hub** | Dense repeaters every 60m | Dense routers every 40m | Dependent on telco tower uptime |
| **Channel Contention Control** | **Deterministic TDMA Superframe** | CSMA/CA (Packet storms) | CSMA/CA (High contention) | Telco network scheduled |
| **Indian Spectrum Compliance** | **GSR 564(E) Compliant** | WPC ETA required | WPC ETA required | TRAI / Telecom carrier governed |

---

## 3. Electromagnetic and Architectural Rationale for LoRa IN865

### Question: What is the physical basis for choosing the 865–867 MHz Sub-GHz band over 2.4 GHz ISM technologies?

**Answer:** The choice of the 865–867 MHz band is governed by foundational electromagnetic wave propagation physics:

1. **Free-Space Path Loss (FSPL) Advantage:**
   According to the Friis transmission formula, path loss increases with the square of the frequency:
   $$\text{FSPL} = 20\log_{10}(d) + 20\log_{10}(f) - 147.55\text{ dB}$$
   Comparing an 865 MHz carrier against a 2.4 GHz carrier over an identical distance $d$:
   $$\Delta \text{FSPL} = 20\log_{10}\left(\frac{2400\text{ MHz}}{865\text{ MHz}}\right) \approx 20\log_{10}(2.77) \approx \mathbf{8.87\text{ dB}}$$
   This means that simply by switching from 2.4 GHz to 865 MHz, the received signal strength improves by nearly $9\text{ dB}$—effectively doubling the operational communication range under identical transmitter power.
2. **Fresnel Zone and Knife-Edge Diffraction:**
   Over undulating ground and spoil tips, diffraction around obstacles is governed by the Fresnel-Kirchhoff diffraction parameter $v$:
   $$v = h \sqrt{\frac{2}{\lambda} \left(\frac{1}{d_1} + \frac{1}{d_2}\right)}$$
   Because wavelength $\lambda$ at 865 MHz ($\approx 0.346\text{ m}$) is nearly three times longer than at 2.4 GHz ($\approx 0.125\text{ m}$), the diffraction parameter $v$ is significantly smaller. Consequently, diffraction attenuation over terrain crests and mine bunds is $12\text{ to }18\text{ dB}$ lower, allowing signals to propagate over terrain irregularities that completely block 2.4 GHz signals.

```
+-----------------------------------------------------------------------------------+
|                        DIFFRACTION COMPARISON OVER MINE CREST                     |
|                                                                                   |
|                   Spoil Dump / Subsidence Scarp                                   |
|                             /\                                                    |
|                            /  \                                                   |
|                           /    \                                                  |
|  2.4 GHz Wave: ──────────>│ X  │ (Severe knife-edge shadow loss: > 35 dB)        |
|  (λ = 12.5 cm)            /    \                                                  |
|                          /      \                                                 |
|  865 MHz Wave: ─────────> \    / ───> [Rx Node] (Low diffraction loss: < 15 dB)   |
|  (λ = 34.6 cm)             \  /                                                   |
+-----------------------------------------------------------------------------------+
```

---

## 4. Local Mesh Isolation vs. Direct Cellular Per Node

### Question: Why does AEGIS reject equipping every field sensor node with a cellular NB-IoT / 4G SIM card, opting instead for a local mesh with a single gateway backhaul?

**Answer:** While cellular NB-IoT appears attractive for simplifying network topology, deploying individual cellular modems on every geotechnical node introduces three fatal engineering flaws:

1. **Vulnerability to Telecommunication Outages:**
   Remote coalfields in Jharkhand, Odisha, and Telangana frequently experience commercial telecommunication downtime due to power grid cuts, transmission tower fiber cuts, or severe monsoon thunderstorms. A system dependent on direct cellular connectivity per node loses all safety monitoring during an outage. In contrast, AEGIS maintains an autonomous local LoRa mesh; even if the gateway loses external cellular backhaul, it continues ingesting local sensor data and can trip its hardwired 125 dB on-site evacuation siren autonomously.
2. **Excessive Energy Consumption and Attach Latency:**
   Connecting to a cellular base station requires network synchronization, random access preamble negotiation, and authentication handshakes taking $5\text{ to }25\text{ seconds}$ during which the modem draws $150\text{ to }250\text{ mA}$. Under poor coverage, this attachment sequence exhausts node batteries in weeks. LoRa, by contrast, transmits an un-associated physical frame in **$90.4\text{ milliseconds}$** and immediately returns to deep sleep.
3. **Severe Recurring Operating Expenditure (OpEx):**
   Deploying 400 nodes across a commercial extraction panel with cellular SIMs costs ₹600 to ₹1,200 per SIM annually, creating an ongoing recurring liability of ₹2,40,000 to ₹4,80,000 per year, plus recurring logistical overhead for SIM deactivations and KYC renewals. AEGIS incurs **₹0 recurring RF cost** by operating on the license-free 865–867 MHz band, requiring only a single industrial SIM at the Master Gateway.

---

## 5. Custom TDMA MAC vs. Standard LoRaWAN ALOHA

### Question: Why does AEGIS implement a custom TDMA MAC protocol on top of the raw LoRa physical layer rather than using off-the-shelf LoRaWAN?

**Answer:** Standard LoRaWAN operates as an uncoordinated pure ALOHA protocol, where end nodes transmit whenever they have data. In high-density geotechnical monitoring arrays, pure ALOHA collapses:

1. **The ALOHA Collision Limit:**
   By the Poisson distribution, the maximum theoretical channel throughput of pure ALOHA is bounded at:
   $$S = G \cdot e^{-2G} \implies S_{\max} = \frac{1}{2e} \approx 18.4\%$$
   When 30 to 40 nodes wake at similar intervals to report geotechnical data, packet collision rates exceed $40\%$, causing severe packet loss and telemetry blind spots.
2. **Deterministic Time-Division Multiple Access (TDMA):**
   AEGIS implements a synchronized TDMA superframe over raw LoRa:
   * **Collision-Free Transmission:** Every Scout node is assigned an exclusive, non-overlapping $125\text{ ms}$ time slot synchronized via millisecond-accurate gateway beacons.
   * **Deterministic Delivery Latency:** Cluster telemetry is guaranteed to arrive at the backbone within a bounded window of $<1.4\text{ seconds}$ per superframe cycle.
   * **Sleep-Wake Energy Optimization:** Nodes wake only for their assigned micro-window, eliminating preamble sniffing and idle listening, reducing sleep-state current to $12\ \mu\text{A}$.
