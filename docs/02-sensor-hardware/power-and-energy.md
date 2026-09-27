# Power Architecture & Energy Budget

**Module 02 — Sensor Hardware**  
**Cross-References:** [`node-classification.md`](node-classification.md) · [`sensor-suite.md`](sensor-suite.md) · [Module 03 TDMA Scheduling](../03-mesh-networking/tdma-scheduling.md)

---

## 1. Battery Chemistry & Solar Harvesting Rationale

Field nodes must survive multi-year outdoor deployments in extreme Indian opencast and subsidence environments (ambient temperatures ranging from $4^\circ\text{C}$ in winter to $48^\circ\text{C}$ in summer, with high solar irradiance and coal dust accumulation).

### Lithium Iron Phosphate ($\text{LiFePO}_4$) Selection
Standard Lithium-Cobalt ($LiCoO_2$) 18650 cells common in consumer electronics are rejected due to thermal runaway risk above $55^\circ\text{C}$ and rapid capacity degradation after 500 charge cycles. AEGIS selects **$\text{LiFePO}_4$ (3.2V 1500mAh 18650 cylindrical cells)**:
* **Thermal Stability:** Stable up to $+70^\circ\text{C}$ without thermal runaway.
* **Cycle Durability:** Over 2,000 full charge-discharge cycles to 80% capacity retention (equivalent to >5 years of continuous field operation).
* **Flat Voltage Plateau:** Operates between $3.2\text{V}$ and $3.1\text{V}$ across 80% of its discharge curve, minimizing reference rail drift for analogue sensor circuitry.

### Solar Harvesting Sizing
* **Solar Panel:** 1 Watt monocrystalline panel ($5.5\text{V}$, $180\text{ mA}$ peak current) encased in UV-stabilized epoxy with an IP67 mounting bracket.
* **Charge Controller:** TP5000 dedicated $\text{LiFePO}_4$ switch-mode charger configured with a $3.65\text{V}$ float voltage cutoff.
* **Dust Margin:** Oversized by $3.5\times$ relative to daily consumption, ensuring that even under severe coal dust shading (reducing solar output by up to 70%), the cell achieves full recharge during 4 hours of daylight.

---

## 2. 60-Second Epoch Power Cycle & Duty Budget

A Scout node spends over 98% of its operational life in ultra-low-power deep sleep. It wakes synchronously on a 60-second hardware timer derived from the network TDMA schedule.

```
+-----------------------------------------------------------------------------------+
|                           60-SECOND SCOUT OPERATING CYCLE                         |
|                                                                                   |
|  [ WAKE & READ ]  [ 200 Hz FFT ]  [ PACK 23B ]  [ LORA TX ]  [ DEEP SLEEP ...... ] |
|     3.0 ms           640 ms          0.05 ms      90.4 ms        59.26 seconds    |
|    (15 mA)          (22 mA)          (15 mA)      (110 mA)        (12 µA)         |
+-----------------------------------------------------------------------------------+
```

### Energy Consumption Breakdown Per 60s Epoch

| Operating Phase | Duration | Current Draw | Energy per Epoch |
| :--- | :--- | :--- | :--- |
| **Sensor I²C / SPI Read** | 3.0 ms | 15 mA | $0.045\text{ mA}\cdot\text{s}$ |
| **Vibration Sampling Burst (256 pts)** | 640 ms | 22 mA | $14.08\text{ mA}\cdot\text{s}$ |
| **Edge FFT Computation** | 1.5 ms | 45 mA | $0.068\text{ mA}\cdot\text{s}$ |
| **Binary 23-byte Packing** | 0.05 ms | 15 mA | $0.001\text{ mA}\cdot\text{s}$ |
| **LoRa Uplink Transmission (SF7/125kHz)** | 90.4 ms | 110 mA | $9.944\text{ mA}\cdot\text{s}$ |
| **Downlink ACK Window (SX1262 RX)** | 40.0 ms | 11 mA | $0.440\text{ mA}\cdot\text{s}$ |
| **Deep Sleep (RTC Timer Active)** | 59,225 ms | 0.012 mA | $0.711\text{ mA}\cdot\text{s}$ |
| **Total Energy Per 60-Second Cycle** | **60,000 ms** | **—** | **$25.29\text{ mA}\cdot\text{s}$** |

$$\text{Average Current Draw} = \frac{25.29\text{ mA}\cdot\text{s}}{60\text{ s}} \approx 0.422\text{ mA}$$
$$\text{Average Power Consumption} = 0.422\text{ mA} \times 3.2\text{ V} \approx 1.35\text{ mW}$$

---

## 3. Autonomy & Battery Life Without Sun

In the event of total solar panel failure or continuous monsoon cloud cover:
$$\text{Operational Autonomy} = \frac{1500\text{ mAh} \times 0.85\ (\text{usable depth})}{0.422\text{ mA}} \approx 3,021\text{ hours} \approx 125.8\text{ days}$$

A fully charged Scout node operates autonomously for **over 4 months** in total darkness without dropping a single 60-second telemetry packet.

---

## 4. ESP32 Compute Budget: Why Edge MIPS Are Plentiful

The embedded compute budget on the ESP32 (dual-core Xtensa LX6 @ 240 MHz) has two orders of magnitude of spare capacity:

| Compute Routine | Execution Time | RAM Consumption | Headroom Factor |
| :--- | :--- | :--- | :--- |
| **7-Channel Sensor Polling** | 3.0 ms | 64 Bytes | $> 20,000\times$ |
| **256-Point Real FFT** | 1.5 ms | 2.0 KB | $> 40,000\times$ |
| **Wire Pack & CRC-16 Checksum** | 45 µs | 23 Bytes | $> 1,000,000\times$ |
| **72-Hour SPI Flash Ring Buffer** | Persistent | 99 KB (of 4MB flash) | $> 40\times$ |

The binding constraint on field IoT hardware is **RF airtime and milliamp-hours, not processor clock cycles**. Compute is virtually free; all physical design pressures stem from radio transmission limitations.
