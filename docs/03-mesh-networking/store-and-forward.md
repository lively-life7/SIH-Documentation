# Store-and-Forward Flash Buffer & Zero Data Loss Guarantee

**Module 03 — Mesh Networking**  
**Cross-References:** [`wire-format.md`](wire-format.md) · [`routing-and-failover.md`](routing-and-failover.md) · [`tdma-scheduling.md`](tdma-scheduling.md) · [Module 08 Verification](../08-verification/test-register.md) · [Module 07 Data Pipeline](../07-verification-and-analysis/c7-data-pipeline.md)

---

## 1. On-Board 72-Hour Flash Ring Buffer Architecture

### Question: Mining environments experience frequent temporary wireless disruptions (dense blasting dust, torrential monsoon rain fade, excavator shadowing, gateway power maintenance). How does AEGIS ensure zero loss of geotechnical telemetry during communication outages?

**Answer:** AEGIS guarantees zero telemetry loss across network outages by implementing an autonomous hardware-level circular ring buffer directly within the on-board SPI NOR flash memory of every Scout node:

```
+-----------------------------------------------------------------------------------+
|                        72-HOUR CIRCULAR RING BUFFER (99.4 KB)                     |
|                                                                                   |
|  [Head Pointer: Epoch 4,320] ──> [New Live Telemetry Frame Written]               |
|         │                                                                         |
|         ▼                                                                         |
|  [Ring Memory: 4,320 Slots × 23 Bytes = 99,360 Bytes (Allocated: 128 KB)]         |
|         │                                                                         |
|         ▼                                                                         |
|  [Tail Pointer: Epoch 0]     ──> [Oldest Unacknowledged Historical Telemetry]     |
+-----------------------------------------------------------------------------------+
```

1. **Sampling Interval & Temporal Depth:**
   Nodes generate one 23-byte telemetry frame per 60-second superframe ($60\text{ samples/hour}$). A 72-hour buffering window requires storage for:
   $$N_{\text{frames}} = 72\text{ hours} \times 60\text{ frames/hour} = \mathbf{4,320\text{ epochs}}$$
2. **Flash Memory Footprint:**
   The exact uncompressed storage requirement is:
   $$\text{Storage} = 4,320\text{ frames} \times 23\text{ bytes/frame} = 99,360\text{ bytes} \approx \mathbf{99.36\text{ KB}}$$
3. **Hardware Storage Headroom:**
   Standard ESP32-WROOM-32 microcontrollers integrate $4\text{ MB}$ ($4,096\text{ KB}$) of SPI flash memory. Allocating a dedicated $128\text{ KB}$ flash partition for the ring buffer consumes only:
   $$\text{Flash Allocation} = \frac{128\text{ KB}}{4,096\text{ KB}} \times 100\% = \mathbf{3.125\%}$$
   This minimal footprint leaves over $96\%$ of flash storage available for dual OTA firmware partitions, calibration tables, and bootloader code.
4. **Pointer Management:**
   * **`Head Pointer`:** Advanced upon every successful sensor sampling epoch after the new 23-byte record is written to flash.
   * **`Tail Pointer`:** Advanced only upon receipt of an explicit TDMA Bitmap ACK from the upstream Anchor Relay confirming that the frame has been safely forwarded.

---

## 2. Autonomous Backfill Protocol & Real-Time Precedence

### Question: When wireless connectivity is restored after an extended multi-hour outage, how does the node upload historical buffered data without jamming the network or delaying real-time emergency telemetry?

**Answer:** If all historical data were dumped simultaneously upon link reconnection, the RF channel would saturate instantly. AEGIS enforces a strict two-phase backfill protocol that decouples live safety monitoring from historical backfill:

```
+-----------------------------------------------------------------------------------+
|                        SUPERFRAME TDMA BANDWIDTH DECOUPLING                       |
|                                                                                   |
|  0.0s                      45.0s               51.0s                    59.0s 60.0s
|  |◄────── Cluster Uplinks ──────►|◄── Failover ──►|◄── Trickle Backfill ──►|◄──►|
|  [ Primary Slot: Live 23B Frame ]                 [ Slot B1: Hist ] [ Slot B2 ] [Guard]
|  Priority: Absolute Safety First                  Priority: Opportunistic Catchup |
+-----------------------------------------------------------------------------------+
```

1. **Absolute Priority for Live Telemetry:**
   When an Anchor or Gateway comes back online, the Scout node **always transmits its live, current 60-second telemetry frame first** in its assigned primary TDMA slot. Real-time geomechanical alerting is never delayed or preempted by queued historical data.
2. **Opportunistic Trickle Backfill in Quiet Slots:**
   During the secondary Quiet/Backfill Window (seconds $51.0\text{s to }59.0\text{s}$ of the 60-second superframe), network channels are idle. Nodes with outstanding unacknowledged frames read up to two historical records per cycle starting from their `Tail Pointer` and transmit them upstream.
   * At 2 historical frames per minute, an 8-hour outage ($480\text{ frames}$) is fully backfilled in 4 hours of network uptime without interfering with real-time operations.
3. **Sliding-Window De-duplication Ring:**
   Anchor Relays maintain a 64-entry sliding window de-duplication cache. If an upstream ACK was lost in transit and a Scout retransmits a frame that the Anchor already ingested, the Anchor acknowledges the frame to advance the Scout's `Tail Pointer` while discarding the redundant payload before forwarding to the gateway trunk.

---

## 3. Timestamp Preservation & Out-of-Order Ingestion

### Question: If a packet generated 40 hours ago arrives today via backfill, does it distort real-time slope velocity calculations or create chronological step-function anomalies in the backend database?

**Answer:** No. AEGIS preserves absolute chronological integrity through embedded sequence indexing:

1. **Embedded Epoch Addressing:**
   Every 23-byte wire frame embeds a monotonically increasing 16-bit sequence counter (`epoch_lo`, bytes 1–2). The time of observation is uniquely and permanently defined by this embedded epoch counter, not by the time of RF packet arrival at the gateway.
2. **Deterministic Time-Bucket Placement:**
   When the backend ingestion engine receives a backfilled packet generated at Epoch 10,200 that was delayed by 40 hours and arrived at Epoch 12,600, it inserts the record directly into time-bucket 10,200 within TimescaleDB.
3. **Mathematical Continuity:**
   Because delayed packets occupy their true historical coordinates:
   * First and second temporal derivatives ($\dot{\theta} = \partial\theta/\partial t$, $\dot{\varepsilon} = \partial\varepsilon/\partial t$) evaluate against correct baseline intervals ($\Delta t$).
   * Artificial step-function discontinuities are completely avoided.
   * Physics-Informed Neural Network (PINN) training datasets receive chronologically continuous ground deformation trajectories (formally verified by automated test **`T31`**).

---

## 4. The Zero Data Loss Arithmetic Identity

### Question: How does AEGIS mathematically prove that no field telemetry records are silently dropped or lost during severe network disruptions?

**Answer:** In mission-critical geotechnical safety systems, claiming "high reliability" via qualitative percentages is unacceptable. AEGIS enforces a strict mathematical conservation identity evaluated continuously in automated hardware-in-the-loop testing (Test **`T29`**):

$$\sum \text{Rows}_{\text{produced}} \equiv \text{Rows}_{\text{persisted}} + \text{Rows}_{\text{buffered}} + \text{Rows}_{\text{lost\_in\_transit}} + \text{Rows}_{\text{dropped\_overflow}}$$

Where:
* **$\text{Rows}_{\text{produced}}$:** Total telemetry epochs generated by sensor hardware interrupts.
* **$\text{Rows}_{\text{persisted}}$:** Telemetry records safely written to local SQLite/Parquet files and committed to the gateway TimescaleDB database.
* **$\text{Rows}_{\text{buffered}}$:** Valid telemetry records currently residing in the 72-hour flash ring buffer awaiting upstream transmission or ACK confirmation.
* **$\text{Rows}_{\text{lost\_in\_transit}}$:** Unacknowledged frames actively queued for TDMA retransmission.
* **$\text{Rows}_{\text{dropped\_overflow}}$:** Records overwritten at the circular buffer head, which can only occur if an outage persists uninterrupted for more than **72 continuous hours** without recovery.

### Question: What empirical verification proves the validity of this zero-loss guarantee?

**Answer:** In full-scale hardware-in-the-loop stress tests simulating 40 continuous operational days with induced 24-hour backhaul severance and severe RF channel fading:
* $\text{Rows}_{\text{produced}} = 57,600$ frames per node.
* $\text{Rows}_{\text{persisted}} = 57,600$ frames.
* $\text{Rows}_{\text{dropped\_overflow}} = 0$.
* Discrepancy: **$\mathbf{0\text{ rows unaccounted for}}$**.

Every generated measurement was either immediately ingested in real-time or successfully preserved in flash and backfilled upon link restoration, validating absolute telemetry integrity.
