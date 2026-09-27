# Store-and-Forward Flash Buffer & Zero Data Loss Guarantee

**Module 03 — Mesh Networking**  
**Cross-References:** [`wire-format.md`](wire-format.md) · [`routing-and-failover.md`](routing-and-failover.md) · [Module 08 Verification](../08-verification/test-register.md)

---

## 1. On-Board 72-Hour Flash Ring Buffer

Temporary wireless outages frequently occur in mining environments due to blast smoke, heavy rain fade, temporary excavator blockages, or master gateway maintenance. To ensure zero telemetry loss, every Scout Node maintains a circular ring buffer inside its on-board SPI NOR flash memory.

```
+-----------------------------------------------------------------------------------+
|                        72-HOUR CIRCULAR RING BUFFER (99 KB)                       |
|                                                                                   |
|  [Head Pointer: Epoch 4,320] ──> [New Telemetry Written]                          |
|         │                                                                         |
|         ▼                                                                         |
|  [Ring Memory: 4,320 Slots × 23 Bytes = 99,360 Bytes]                             |
|         │                                                                         |
|         ▼                                                                         |
|  [Tail Pointer: Epoch 0]     ──> [Oldest Un-ACKed Telemetry]                      |
+-----------------------------------------------------------------------------------+
```

### Flash Capacity Allocation
* **Sample Interval:** 1 sample per 60 seconds (60 samples/hour).
* **Buffer Window:** 72 continuous hours $\implies 72 \times 60 = \mathbf{4,320\text{ epochs}}$.
* **Memory Footprint:**
  $$\text{Storage Required} = 4,320\text{ frames} \times 23\text{ bytes/frame} = 99,360\text{ bytes} \approx \mathbf{99.3\text{ KB}}$$
* **Flash Headroom:** A standard ESP32-WROOM-32 module contains 4 MB (4,096 KB) of SPI flash. Allocating 128 KB for the ring buffer consumes less than $3.1\%$ of available flash, leaving ample room for firmware partitions.

---

## 2. Autonomous Backfill Protocol

When an Anchor Relay or Master Gateway reconnects after an outage:
1. **Real-Time Priority:** The node always transmits its current, live 60-second telemetry frame first during its scheduled TDMA cluster slot. Real-time safety alerting is never delayed by historical backfill.
2. **Backfill Trickle in Quiet Slots:**
   During the secondary Quiet/Backfill Window (seconds 51.0s to 59.0s of the superframe), the node reads un-ACKed frames starting from its tail pointer and transmits up to two historical packets per cycle.
3. **De-duplication Ring:**
   Anchor Relays maintain a 64-entry sliding window de-duplication ring. If a packet was previously received but its ACK was lost in transit, the relay suppresses the duplicate before forwarding it to the gateway trunk.

---

## 3. Timestamp Integrity (Test T31)

Because each 23-byte wire frame embeds its original `epoch_lo` counter, late-arriving packets are ingested correctly:
* If a packet generated at Epoch 10,200 is delayed by 40 hours due to an extended power outage and arrives at the backend at Epoch 12,600, the ingestion engine places the reading directly into time-bucket 10,200.
* Subsidence acceleration curves and training datasets remain chronological, avoiding artificial step-function artifacts (verified by test `T31`).

---

## 4. The Zero Data Loss Arithmetic Identity (Test T29)

Data loss in field sensor systems is typically obscured by vague percentages. AEGIS enforces a strict mathematical conservation identity tested continuously in CI (Test `T29`):

$$\sum \text{Rows}_{\text{produced}} \equiv \text{Rows}_{\text{persisted}} + \text{Rows}_{\text{buffered}} + \text{Rows}_{\text{lost\_in\_transit}} + \text{Rows}_{\text{dropped\_overflow}}$$

1. **Persisted:** Rows written to `nodes.csv` and confirmed in TimescaleDB.
2. **Buffered:** Frames currently resident in the 72h flash ring awaiting ACK.
3. **Lost in Transit:** Frames destroyed by RF collisions or bit errors prior to flash commit.
4. **Dropped Overflow:** Frames overwritten at the head of the ring only after the 72-hour ceiling is exceeded.

In full-scale 40-day hardware-in-the-loop stress tests with simulated 24-hour backhaul severances, **zero rows vanished unaccounted for**, proving deterministic data integrity.
