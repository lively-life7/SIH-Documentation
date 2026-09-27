# Power Architecture & Energy Budget

**Module 02 — Sensor Hardware**  
**Cross-References:** [`node-classification.md`](node-classification.md) · [`sensor-suite.md`](sensor-suite.md) · [`bill-of-materials.md`](bill-of-materials.md) · [Module 03 TDMA Scheduling](../03-mesh-networking/tdma-scheduling.md) · [Module 00 Key Metrics Summary](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Battery Chemistry & Environmental Durability

### Question: Why does AEGIS specify Lithium Iron Phosphate ($\text{LiFePO}_4$) chemistry rather than standard Lithium-Ion/Cobalt ($LiCoO_2$) or NMC cells for mining panel deployments?

**Answer:** Standard consumer Lithium-Ion ($LiCoO_2$) and Nickel Manganese Cobalt (NMC) chemistries represent unacceptable safety hazards in Indian open-cast and subsidence monitoring environments:
1. **Thermal Stability and Explosion Immunity:** In Indian mining sectors (e.g., Godavari Valley, Singrauli, Jharia), summer ambient temperatures regularly reach $+48^\circ\text{C}$. Inside sealed, UV-exposed IP67 enclosures, internal temperatures can exceed $+65^\circ\text{C}$. Standard lithium chemistries undergo exothermic electrolyte breakdown and catastrophic thermal runaway above $+55^\circ\text{C}$. In contrast, $\text{LiFePO}_4$ possesses exceptional chemical stability, remaining completely safe up to $+70^\circ\text{C}$ with zero risk of explosion or fire.
2. **Industrial Cycle Longevity:** Standard Li-ion cells degrade after 300 to 500 charge-discharge cycles, requiring node battery replacement within 18 months. $\text{LiFePO}_4$ delivers **$>2,000$ full cycles** to $80\%$ capacity retention—guaranteeing over 5 years of continuous, unattended operational life in the field.
3. **Flat Voltage Discharge Plateau:** $\text{LiFePO}_4$ maintains a nearly flat discharge profile between $3.2\text{V}$ and $3.1\text{V}$ across $80\%$ of its nominal capacity. This flat rail ensures reference voltage stability for analog-to-digital converters (ADCs), preventing measurement skew and drift that plague conventional batteries as they discharge.

### Question: How are the solar harvesting and battery charging circuits engineered to withstand severe coal dust accumulation and monsoon shading?

**Answer:** Solar power harvesting is systematically oversized to guarantee autonomy under extreme environmental degradation:
* **Solar Collector:** A 1-Watt monocrystalline photovoltaic panel ($5.5\text{V}$ open circuit, $180\text{ mA}$ peak short-circuit current) encased in UV-stabilized optical epoxy with an IP67 anodized aluminum mounting bracket.
* **Charge Management Circuitry:** Utilizes a TP5000 dedicated switch-mode $\text{LiFePO}_4$ charger configured for a strict $3.65\text{V}$ float voltage cutoff with reverse-current leakage blocking ($<1\ \mu\text{A}$) and over-temperature thermal throttling.
* **Coal Dust Derating Margin:** Coal dust settling on the solar face can attenuate incident sunlight by up to $70\%$. Even under $70\%$ occlusion, the panel delivers $\approx 54\text{ mA}$ of charging current during peak solar hours ($180\text{ mA} \times 0.30$). Since a Scout node consumes only $10.13\text{ mAh}$ per day, just **12 minutes of direct sunlight** (or 40 minutes of diffuse overcast daylight) fully restores the daily energy expenditure.

---

## 2. 60-Second Epoch Power Budget & Energy Calculations

### Question: What is the step-by-step energy breakdown and current consumption across the 60-second operational superframe cycle?

**Answer:** A Scout node spends over $98.7\%$ of its operational life in ultra-low-power deep sleep, waking synchronously once every 60 seconds according to its assigned TDMA slot:

```
+-----------------------------------------------------------------------------------+
|                           60-SECOND SCOUT OPERATING CYCLE                         |
|                                                                                   |
|  [ WAKE & READ ]  [ 200 Hz FFT ]  [ PACK 23B ]  [ LORA TX ]  [ DEEP SLEEP ...... ] |
|     3.0 ms           640 ms          0.05 ms      90.4 ms        59.26 seconds    |
|    (15 mA)          (22 mA)          (15 mA)      (110 mA)        (12 µA)         |
+-----------------------------------------------------------------------------------+
```

The precise electrical charge consumption per 60-second epoch is itemized below:

| Operational Phase | Duration ($t$) | Current ($I$) | Electrical Charge ($I \times t$) | Energy ($V_{cc} = 3.2\text{V}$) |
| :--- | :--- | :--- | :--- | :--- |
| **Sensor I²C / SPI Read** | $3.0\text{ ms}$ | $15.0\text{ mA}$ | $0.0450\text{ mA}\cdot\text{s}$ | $0.144\text{ mJ}$ |
| **Vibration Sampling Burst (256 pts @ 400Hz)** | $640.0\text{ ms}$ | $22.0\text{ mA}$ | $14.0800\text{ mA}\cdot\text{s}$ | $45.056\text{ mJ}$ |
| **Edge FFT Feature Extraction** | $1.5\text{ ms}$ | $45.0\text{ mA}$ | $0.0675\text{ mA}\cdot\text{s}$ | $0.216\text{ mJ}$ |
| **Wire Payload Serialization & CRC-16** | $0.05\text{ ms}$ | $15.0\text{ mA}$ | $0.0008\text{ mA}\cdot\text{s}$ | $0.003\text{ mJ}$ |
| **LoRa Uplink Transmission (SF7 / 125kHz)** | $90.4\text{ ms}$ | $110.0\text{ mA}$ | $9.9440\text{ mA}\cdot\text{s}$ | $31.821\text{ mJ}$ |
| **Downlink ACK Window (SX1262 RX)** | $40.0\text{ ms}$ | $11.0\text{ mA}$ | $0.4400\text{ mA}\cdot\text{s}$ | $1.408\text{ mJ}$ |
| **Ultra-Low Power Deep Sleep** | $59,225.45\text{ ms}$ | $0.012\text{ mA}$ | $0.7107\text{ mA}\cdot\text{s}$ | $2.274\text{ mJ}$ |
| **Total Per 60-Second Superframe** | **$60,000.0\text{ ms}$** | — | **$25.288\text{ mA}\cdot\text{s}$** | **$80.922\text{ mJ}$** |

### Question: What are the resulting average current, average power consumption, and daily milliamp-hour requirements?

**Answer:** Converting the cumulative charge per epoch yields the operational power metrics:

1. **Average Operating Current:**
   $$I_{\text{avg}} = \frac{Q_{\text{epoch}}}{T_{\text{epoch}}} = \frac{25.288\text{ mA}\cdot\text{s}}{60.0\text{ s}} \approx \mathbf{0.4215\text{ mA}} \approx 421.5\ \mu\text{A}$$
2. **Average Power Dissipation:**
   $$P_{\text{avg}} = I_{\text{avg}} \times V_{\text{nominal}} = 0.4215\text{ mA} \times 3.2\text{ V} \approx \mathbf{1.349\text{ mW}}$$
3. **Daily Energy Consumption:**
   $$\text{Consumption}_{\text{day}} = I_{\text{avg}} \times 24\text{ hours} = 0.4215\text{ mA} \times 24\text{ h} = \mathbf{10.12\text{ mAh/day}}$$
   $$(10.12\text{ mAh/day} \times 3.2\text{ V} = 32.38\text{ mWh/day} \approx 116.6\text{ Joules/day})$$

---

## 3. Autonomous Battery Longevity Under Solar Failure

### Question: How long can a field Scout node operate continuously under complete solar panel failure or prolonged monsoon cloud cover?

**Answer:** If a solar panel is destroyed by rockfall or completely occluded by thick monsoon cloud cover, the node relies solely on its internal $1500\text{ mAh}$ cell:

$$\text{Usable Capacity} = C_{\text{nom}} \times \text{DoD}_{\text{safe}} = 1500\text{ mAh} \times 0.85 = 1,275\text{ mAh}$$

$$\text{Autonomous Lifespan} = \frac{\text{Usable Capacity}}{I_{\text{avg}}} = \frac{1,275\text{ mAh}}{0.4215\text{ mA}} \approx 3,025\text{ hours}$$

$$\text{Autonomous Lifespan (Days)} = \frac{3,025\text{ hours}}{24\text{ hours/day}} \approx \mathbf{126.0\text{ days}}\ (\approx 4.2\text{ months})$$

A Scout node survives uninterrupted for **over four months in total darkness**, fully bridging the longest Indian monsoon season (June to September) without dropping a single 60-second telemetry packet.

---

## 4. Microcontroller Compute Budget and Headroom

### Question: Does running on-node edge FFT and digital signal processing compromise microcontroller longevity or energy efficiency?

**Answer:** No. On modern embedded microcontrollers, processor clock cycles are virtually free, whereas radio transmission airtime is the dominant energy constraint.

The ESP32’s dual-core Xtensa LX6 processor operating at $240\text{ MHz}$ provides 600 MIPS (Million Instructions Per Second) of computing capacity. The computational headroom of all node routines is summarized below:

| Computational Task | Execution Time | RAM Footprint | CPU Headroom Factor | Energy per Epoch |
| :--- | :--- | :--- | :--- | :--- |
| **7-Channel Sensor Polling ($I^2C$/SPI)** | $3.0\text{ ms}$ | $64\text{ Bytes}$ | $> 20,000\times$ | $0.144\text{ mJ}$ ($0.18\%$) |
| **256-Point Real FFT Feature Extraction** | $1.5\text{ ms}$ | $2.0\text{ KB}$ | $> 40,000\times$ | $0.216\text{ mJ}$ ($0.27\%$) |
| **Wire Framing & CRC-16 Calculation** | $0.045\text{ ms}$ | $23\text{ Bytes}$ | $> 1,000,000\times$ | $0.003\text{ mJ}$ ($0.004\%$) |
| **Flash Log Append (SPI Ring Buffer)** | $0.8\text{ ms}$ | $128\text{ Bytes}$ | $> 75,000\times$ | $0.038\text{ mJ}$ ($0.05\%$) |
| **LoRa Uplink Transmission (RF Output)** | $90.4\text{ ms}$ | — | — | **$31.821\text{ mJ}$ ($39.3\%$)** |
| **Deep Sleep Standby (RTC Active)** | $59,225\text{ ms}$ | — | — | **$2.274\text{ mJ}$ ($2.8\%$)** |

As demonstrated, the 256-point FFT consumes only $0.216\text{ mJ}$—less than **$0.27\%$** of the superframe energy budget. In contrast, the LoRa radio transmission consumes $31.82\text{ mJ}$ ($39.3\%$ of total energy). Executing local edge analytics to compress $144\text{ KB}$ of raw data into 5 bytes reduces total RF energy consumption by over $99\%$, proving that edge compute directly optimizes system battery life.
