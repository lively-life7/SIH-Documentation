# TDMA Superframe Scheduling & Collision Avoidance

**Module 03 — Mesh Networking**  
**Cross-References:** [`wire-format.md`](wire-format.md) · [`routing-and-failover.md`](routing-and-failover.md) · [`spectrum-compliance.md`](spectrum-compliance.md)

---

## 1. The 60-Second Superframe Architecture

Unslotted ALOHA channel access suffers from severe collision collapse when channel utilization exceeds 18%. To ensure 100% deterministic packet delivery and sub-second emergency response, AEGIS enforces a strict **60-Second TDMA Superframe** synchronized to the Master Gateway's clock.

```
|--- 0.0s ---|--- 1.0s to 37.0s ---|--- 38.0s to 45.0s ---|--- 46.0s to 50.0s ---|--- 51.0s to 60.0s ---|
[Beacon Sync] [Scout Clusters S1..S6] [Anchor Backbone TX]   [Emergency Slots E1..E2] [Deep Sleep / Quiet]
  (75.0 ms)      (1.9s per block)         (SF8 Bundles)          (Collision-free)         (Channel Clear)
```

### Superframe Phase Allocation

| Phase Window | Duration | Protocol Function | PHY Configuration | Airtime per Transmission |
| :--- | :--- | :--- | :--- | :--- |
| **0.00s – 0.10s** | 100 ms | **Master Sync Beacon** broadcast by Gateway | SF7 / BW125 | 75.0 ms (13 bytes) |
| **1.00s – 37.00s** | 36.0 s | **Scout Cluster Uplinks** (6 clusters, 6 slots each) | SF7 / BW125 | 90.4 ms (23 bytes) |
| **38.00s – 45.00s** | 7.0 s | **Anchor Backbone Relays** to Master Gateway | SF8 / BW125 | 406.0 – 457.2 ms (bundled) |
| **46.00s – 50.00s** | 4.0 s | **Emergency Contention-Free Slots** (16 subslots) | SF7 / BW125 | 69.9 ms (8-byte alert) |
| **51.00s – 60.00s** | 9.0 s | **Store-and-Forward Backfill & Quiet Window** | SF7 / BW125 | Dynamic backfill |

---

## 2. Cluster Slot Structure & Bitmap Acknowledgments

Each cluster block (lasting $1.9\text{ seconds}$) coordinates up to 6 leaf Scout nodes reporting to a designated Anchor Relay:
* **Slots 1 to 6 (250 ms each):** Leaf Scout nodes transmit their 23-byte telemetry packet in their assigned sub-slot.
* **Bitmap ACK Window (150 ms):** The Anchor Relay transmits a single compact **Bitmap ACK packet (4 bytes payload, 20 bytes on-air)**. Each bit in the mask corresponds to a child node ID:
  $$\text{Bitmap} = \sum_{i=1}^6 b_i \cdot 2^{i-1}$$
  If bit $b_i = 1$, node $i$ marks its packet as delivered and clears its immediate retransmit buffer. This single broadcast replaces 6 individual downlink packets, cutting downlink airtime by **$87\%$** and keeping gateway duty cycle well under statutory limits.
* **Retry Sub-slot (250 ms):** Any node whose bit was 0 attempts an immediate retransmission before the cluster window closes.

---

## 3. Collision-Free Emergency Slots (Gate G10 & G11)

When a node experiences sudden mechanical shock, rapid tensile strain acceleration ($> 5\times$ baseline gradient), or a conductive crack-wire break:
1. It does not wait for its routine 60-second telemetry slot.
2. It transitions immediately into priority alert mode and transmits in the dedicated **Emergency Window (46.0s – 50.0s)**.
3. **Deterministic Subslot Indexing (Gate G10):**
   To prevent multiple simultaneous alarms from colliding on-air, the emergency window is divided into 16 deterministic subslots ($250\text{ ms}$ each). A node selects its emergency slot using a deterministic modulo function of its hardware ID:
   $$\text{Slot}_{\text{emergency}} = (\text{Node\_ID} \pmod{16})$$
4. **Zero Collision Guarantee (Gate G11):** Even if 20 nodes trip simultaneously due to a sudden roof shear event, their transmissions map into non-overlapping subslots, guaranteeing that the critical trip packet reaches the gateway in **$< 1.4\text{ seconds}$**.

---

## 4. Clock Synchronization & Drift Control

Clock drift is controlled through hardware-level synchronization:
* **Gateway Master Clock:** Disciplined by an onboard GPS receiver (pulse-per-second PPS accuracy $< 1\ \mu\text{s}$) or NTP server when network-connected.
* **Hardware Timestamping:** The Gateway transmits the 13-byte sync beacon at exact second boundaries ($t = 0.000\text{s}$). Scout microcontrollers capture the Semtech SX1262 `DIO1` packet-received hardware interrupt to reset their internal microsecond timers.
* **Drift Margin:** Uncompensated ESP32 crystal oscillators drift by up to $\pm 30\text{ ppm}$ ($\approx 1.8\text{ ms}$ over 60 seconds). Because TDMA guard bands are sized to $50\text{ ms}$, oscillator drift cannot cause slot overlap.
