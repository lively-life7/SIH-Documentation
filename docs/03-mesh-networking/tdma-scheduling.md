# TDMA Superframe Scheduling & Collision Avoidance

**Module 03 — Mesh Networking**  
**Cross-References:** [`wire-format.md`](wire-format.md) · [`routing-and-failover.md`](routing-and-failover.md) · [`spectrum-compliance.md`](spectrum-compliance.md) · [Module 08 Verification](../08-verification/test-register.md) · [Module 00 Key Metrics Summary](../00-executive-gateway/key-metrics-summary.md)

---

## 1. The 60-Second TDMA Superframe Architecture

### Question: Why does AEGIS implement a rigid 60-second Time-Division Multiple Access (TDMA) superframe instead of asynchronous CSMA/CA or pure ALOHA channel access?

**Answer:** In dense industrial wireless deployments, unslotted channel access protocols suffer from severe packet collision collapse as traffic density increases:
1. **The ALOHA Throughput Collapse:**
   Under pure ALOHA (standard LoRaWAN access), channel throughput collapses when traffic load exceeds $18.4\%$ ($G > 0.5$). When dozens of geotechnical sensors wake simultaneously to report ground dynamics, packet collisions exceed $40\%$, producing persistent telemetry blind spots during critical slope instability.
2. **Failure of CSMA/CA with LoRa Physical Chirps:**
   Carrier Sense Multiple Access (CSMA/CA) relies on Clear Channel Assessment (CCA). Because LoRa chirp signals can be successfully demodulated up to $20\text{ dB}$ below the thermal noise floor, standard RSSI channel sensing fails to reliably detect active transmissions, resulting in frequent "hidden terminal" collisions.
3. **The TDMA Determinism Advantage:**
   AEGIS enforces a strictly synchronized **60-Second TDMA Superframe** governed by the Master Gateway. By allocating non-overlapping, microsecond-synchronized transmission windows to every active transmitter, channel contention is reduced to zero, packet delivery determinism reaches $100\%$, and leaf nodes safely sleep for $>98\%$ of each epoch at $12\ \mu\text{A}$ quiescent current.

```
+-----------------------------------------------------------------------------------+
|                        60-SECOND TDMA SUPERFRAME STRUCTURE                        |
|                                                                                   |
|  0.0s        1.0s                           38.0s           46.0s     51.0s       60.0s
|  ┌───┬────────┬───────────────────────────────┬───────────────┬─────────┬──────────┐
|  │SYN│ GUARD  │     SCOUT CLUSTER UPLINKS     │ BACKBONE RELAY│EMERGENCY│ BACKFILL │
|  │BCN│        │     (SF7 / 125 kHz)           │ (SF8 / 125kHz)│ SLOTS   │ & QUIET  │
|  └───┴────────┴───────────────────────────────┴───────────────┴─────────┴──────────┘
|  100ms 900ms            36.0 seconds               7.0 seconds  4.0s     9.0s      |
+-----------------------------------------------------------------------------------+
```

### Question: What is the exact temporal phase allocation across the 60-second superframe, and what are the radio parameters and airtimes for each phase?

**Answer:** The 60-second superframe is divided into five distinct operational phases engineered to isolate traffic classes and prevent cross-tier interference:

| Phase Window | Duration | Protocol Function | PHY Configuration | Packet Type & On-Air Time |
| :--- | :--- | :--- | :--- | :--- |
| **0.00s – 0.10s** | $100\text{ ms}$ | **Master Sync Beacon** broadcast panel-wide by Master Gateway | LoRa SF7 / BW 125 kHz | 13-byte sync beacon ($75.0\text{ ms}$ on-air) |
| **0.10s – 1.00s** | $900\text{ ms}$ | **Network Propagation & Guard Window** | — | Channel idle; permits cluster heads to adjust phase timers |
| **1.00s – 37.00s** | $36.0\text{ s}$ | **Scout Cluster Uplinks** (Parallel orthogonal frequency channels) | LoRa SF7 / BW 125 kHz | 23-byte leaf telemetry ($90.4\text{ ms}$ on-air) |
| **37.00s – 38.00s**| $1.0\text{ s}$ | **Inter-Tier Channel Retuning Guard Window** | — | Anchor nodes finalize trunk frame serialization |
| **38.00s – 45.00s**| $7.0\text{ s}$ | **Anchor Backbone Relays** to Master Gateway Hub | LoRa SF8 / BW 125 kHz | Aggregated trunk bundles ($406.0\text{ to }457.2\text{ ms}$ on-air) |
| **45.00s – 46.00s**| $1.0\text{ s}$ | **Backbone Clearance Guard Window** | — | Master Gateway clears reception buffers |
| **46.00s – 50.00s**| $4.0\text{ s}$ | **Emergency Contention-Free Slots** (16 deterministic subslots) | LoRa SF7 / BW 125 kHz | 8-byte emergency trip frames ($69.9\text{ ms}$ on-air) |
| **50.00s – 51.00s**| $1.0\text{ s}$ | **Emergency Verification Guard Window** | — | Gateway validates emergency alarms |
| **51.00s – 60.00s**| $9.0\text{ s}$ | **Store-and-Forward Backfill & Quiet Window** | LoRa SF7 / BW 125 kHz | Opportunistic historical backfill ($90.4\text{ ms}$) |

---

## 2. Cluster Slot Coordination & Bitmap Downlink Acknowledgments

### Question: How does an Anchor Relay coordinate transmissions from child Scout nodes, and how does the Bitmap Acknowledgment protocol avoid downlink airtime exhaustion?

**Answer:** Each Anchor Relay manages a local cluster of up to 5 child Scout nodes (`max_children_per_anchor = 5`, operating with 4 nominal child nodes and 1 dynamically reserved failover slot). 

1. **The Downlink Airtime Bottleneck:**
   If an Anchor Relay transmitted an individual acknowledgment packet to each child Scout, it would require 5 separate downlink transmissions per cycle ($5 \times 70\text{ ms} = 350\text{ ms}$). This would rapidly deplete relay battery reserves and violate statutory duty-cycle ceilings.
2. **Compact Single-Frame Bitmap ACK:**
   AEGIS eliminates individual ACKs. After child nodes complete their scheduled transmission subslots, the Anchor Relay broadcasts a single **Bitmap ACK frame (1-byte bitmask, 20 bytes total on-air)** during a dedicated $150\text{ ms}$ window:
   $$\text{Bitmap} = \sum_{i=1}^5 b_i \cdot 2^{i-1}$$
   Where bit $b_i = 1$ indicates successful cyclic redundancy check (CRC-16) and ingestion of the telemetry packet from child slot $i$.
3. **Downlink Airtime Savings:**
   A single broadcast acknowledges all cluster children simultaneously, slashing downlink transmission time by **over $80\%$** and maintaining Anchor Relay duty cycle strictly at **$0.56\%$** (well below the $1.0\%$ ceiling).
4. **Immediate Local Retry:**
   Any child node whose corresponding bit is $0$ executes an immediate local retransmission in a designated retry subslot ($250\text{ ms}$) before the cluster block closes, preventing unacknowledged packets from spilling into adjacent cluster schedules.

---

## 3. Collision-Free Emergency Slots (Verification Gates G10 & G11)

### Question: When sudden rock mass shear or crack break-wire rupture occurs, how does AEGIS transmit emergency alarms within seconds without colliding on-air with routine telemetry?

**Answer:** In the event of sudden geotechnical failure (accelerating horizontal strain rate $> 5\times$ baseline or a conductive crack-wire break), waiting up to 60 seconds for a routine TDMA slot would endanger human lives. AEGIS provides dedicated, deterministic fast-path alerting:

```
+-----------------------------------------------------------------------------------+
|                        16-SUBSLOT EMERGENCY WINDOW (46.0s – 50.0s)                |
|                                                                                   |
|  46.0s                                                                      50.0s |
|  ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬┄┄┄┄┄┄┬──────┐        |
|  │Sub-00│Sub-01│Sub-02│Sub-03│Sub-04│Sub-05│Sub-06│Sub-07│      │Sub-15│ (250ms)  |
|  └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┴┄┄┄┄┄┄┴──────┘        |
|  Slot Index = (Node_ID mod 16) ──> Zero Collisions for Up to 16 Concurrent Alarms |
+-----------------------------------------------------------------------------------+
```

1. **Hardware Interrupt Trigger:**
   Physical fissure rupture triggers an instantaneous edge-triggered interrupt on the ESP32, immediately pulling the processor from deep sleep into active alert state.
2. **Emergency Window Execution (46.0s – 50.0s):**
   The node bypasses its routine 60-second telemetry queue and prepares an 8-byte prioritized trip payload (`packet_type = 0xAA`).
3. **Deterministic Subslot Indexing (Gate G10):**
   To prevent multiple concurrent alarms from colliding on the wireless channel, the 4-second Emergency Window is divided into 16 discrete subslots of $250\text{ ms}$ each. The node calculates its exact transmission slot via a deterministic hardware modulo function:
   $$\text{Slot}_{\text{emergency}} = (\text{Node\_ID} \pmod{16})$$
4. **Zero-Collision Mathematical Guarantee (Gate G11):**
   Because slot assignment is mutually exclusive across the 16 modulo indices, even if an entire longwall face experiences sudden roof fracturing causing up to 16 nodes to trip simultaneously, their transmissions land in separate, non-overlapping subslots. The emergency packet arrives at the Master Gateway within **$< 1.4\text{ seconds}$** of physical rupture, instantly triggering the 125 dB evacuation siren.

---

## 4. Clock Synchronization, Drift Margins, and Guard Bands

### Question: How does AEGIS maintain network-wide microsecond time synchronization without GPS on every low-cost scout node, and what ensures crystal oscillator drift does not cause slot overlap?

**Answer:** High-precision synchronization across hundreds of low-cost field nodes is achieved via hierarchical hardware timestamping:

1. **Master Gateway Discipline:**
   The Master Gateway Hub maintains an atomic time reference disciplined by an onboard industrial GPS receiver with a 1-Pulse-Per-Second (1-PPS) hardware signal providing absolute timing accuracy within $< 1\ \mu\text{s}$. If satellite signals are temporarily masked, the gateway maintains time via an oven-controlled crystal oscillator (OCXO) or local NTP.
2. **Hardware-Triggered Edge Synchronization:**
   At exact second zero of each superframe ($t = 0.000\text{s}$), the gateway broadcasts a 13-byte synchronization beacon. When a Scout node’s SX1262 receiver detects the sync preamble, the Semtech transceiver asserts a physical hardware interrupt on its `DIO1` pin. The ESP32 captures this edge via an internal timer capture register, resetting its local microsecond counter and eliminating all operating system software interrupt latency.
3. **Crystal Drift vs. Guard Band Mathematical Margin:**
   The uncompensated internal crystal oscillators of commercial ESP32 modules specify a maximum frequency tolerance of $\pm 30\text{ ppm}$ under extreme operational temperatures ($-20^\circ\text{C}\text{ to }+70^\circ\text{C}$). Over the full 60-second superframe epoch, the maximum accumulated timing drift is:
   $$\Delta t_{\text{drift}} = 60.0\text{ s} \times (\pm 30 \times 10^{-6}) = \mathbf{\pm 1.80\text{ milliseconds}}$$
   Each TDMA cluster subslot is allocated a $250\text{ ms}$ temporal window, whereas the physical 23-byte LoRa packet requires only $90.4\text{ ms}$ of on-air transmission time. This provides an effective guard band of:
   $$\text{Guard Margin} = 250\text{ ms} - 90.4\text{ ms} = \mathbf{159.6\text{ milliseconds}}$$
   Because the guard band ($159.6\text{ ms}$) exceeds maximum crystal drift ($1.8\text{ ms}$) by a safety factor of **$>88\times$**, clock drift can never cause packet overlap or inter-slot collisions between adjacent nodes.
