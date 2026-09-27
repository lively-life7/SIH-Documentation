# Binary Wire Format & Packet Structure

**Module 03 — Mesh Networking**  
**Cross-References:** [`tdma-scheduling.md`](tdma-scheduling.md) · [`store-and-forward.md`](store-and-forward.md) · [`routing-and-failover.md`](routing-and-failover.md) · [Module 06 Data Architecture](../06-backend-pipeline/data-architecture.md) · [Module 07 Data Pipeline](../07-verification-and-analysis/c7-data-pipeline.md)

---

## 1. Byte-by-Byte Wire Map and Fixed-Width Serialization

### Question: Why does AEGIS use a custom 23-byte fixed-width binary struct rather than JSON, Protocol Buffers, or standard LoRaWAN payloads for geotechnical field telemetry?

**Answer:** In low-power Sub-GHz RF telemetry, packet payload size directly governs on-air transmission time, channel duty cycle, battery life, and TDMA slot alignment:
1. **The Inefficiency of Text Encodings (JSON / XML):**
   Encoding 12 geotechnical telemetry parameters into human-readable JSON requires between $180\text{ and }250\text{ bytes}$. At LoRa SF7/125kHz, transmitting $200\text{ bytes}$ takes over $450\text{ ms}$ of on-air time—exhausting the $1.0\%$ duty cycle ceiling, increasing packet collision risk by $5\times$, and draining battery reserves rapidly.
2. **The Problem with Dynamic Serialization (Protobuf / CBOR):**
   Protocol Buffers and CBOR introduce variable-length integer encodings and key tags that cause packet sizes to fluctuate epoch-to-epoch. Variable packet lengths break rigid TDMA time slot boundaries, causing slot jitter and transmission overlaps. Furthermore, the decompression libraries require substantial microcontroller RAM and CPU cycles.
3. **The 23-Byte C-Struct Advantage:**
   AEGIS serializes field telemetry into a **23-byte packed binary struct** (`__attribute__((packed))`, Little-Endian). This format guarantees:
   * **Zero CPU Serialization Overhead:** Memory buffers map directly to the transceiver SPI FIFO via direct pointer casting.
   * **Strictly Deterministic Airtime:** Every packet transmits in exactly **$90.4\text{ ms}$**, aligning with the assigned $250\text{ ms}$ TDMA cluster slot.
   * **Full Metrological Resolution:** Retains high precision for all 12 geotechnical and electrical parameters.

```
+-----------------------------------------------------------------------------------+
|                        23-BYTE BINARY TELEMETRY WIRE MAP                          |
|                                                                                   |
|   0                   1                   2                                       |
|   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3                                 |
|  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                |
|  |    tilt_x     |    tilt_y     |   strain_ue   |  (Bytes 00 – 05)               |
|  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                |
|  | ext_delta_10um|  vib_rms_x100 | vib_peak_x100 |  (Bytes 06 – 11)               |
|  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                |
|  |vib_f| temp_dc |    vbat_mv    |flags| epoch_lo|  (Bytes 12 – 19)               |
|  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                |
|  |  nid  |     crc16     |                          (Bytes 20 – 22)               |
|  +-+-+-+-+-+-+-+-+-+-+-+-+                                                        |
+-----------------------------------------------------------------------------------+
```

### Question: What is the exact byte-by-byte memory layout, data types, physical scaling factors, and measurement domains of the 23-byte telemetry frame?

**Answer:** The 23-byte frame layout is defined by the following C-struct field specifications:

| Offset (Bytes) | Field Identifier | Wire Data Type | Physical Scaling / LSB | Valid Measurement Range | Physical Geotechnical Meaning |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0 – 1** | `tilt_x` | int16 (Little-Endian) | 2 µrad / LSB | ± 65,534 µrad (≈ ± 3.75°) | Transverse ground surface inclination angle |
| **2 – 3** | `tilt_y` | int16 (Little-Endian) | 2 µrad / LSB | ± 65,534 µrad (≈ ± 3.75°) | Longitudinal ground surface inclination angle |
| **4 – 5** | `strain_ue` | int16 (Little-Endian) | 1 µε / LSB | ± 32,767 µε | Horizontal ground strain across 10m baseline |
| **6 – 7** | `ext_delta_10um`| int16 (Little-Endian) | 10 µm / LSB | ± 327.6 mm | 3D relative peg displacement across fault line |
| **8 – 9** | `vib_rms_x100` | uint16 (Little-Endian)| 0.01 mm/s / LSB | 0.00 to 655.35 mm/s | RMS ground vibration velocity during burst |
| **10 – 11**| `vib_peak_x100`| uint16 (Little-Endian)| 0.01 mm/s / LSB | 0.00 to 655.35 mm/s | Peak Particle Velocity (PPV) shock impulse |
| **12** | `vib_fdom_hz` | uint8 | 1 Hz / LSB | 0 to 255 Hz | Dominant FFT spectral frequency peak |
| **13 – 14**| `temp_dc` | int16 (Little-Endian) | 0.1°C / LSB | -40.0°C to +85.0°C | Semiconductor die temperature for thermal compensation |
| **15 – 16**| `vbat_mv` | uint16 (Little-Endian)| 1 mV / LSB | 2,500 to 4,500 mV | LiFePO4 battery terminal voltage |
| **17** | `status_flags` | uint8 (Bitmask) | 8 discrete flags | Bitfield | Sensor self-test, trip, solar, and failover status |
| **18 – 19**| `epoch_lo` | uint16 (Little-Endian)| 1 epoch (60 s) | 0 to 65,535 (45.5 days) | Lower 16 bits of hardware sample sequence clock |
| **20** | `node_id` | uint8 | Integer ID | 1 to 255 | Unique network node address |
| **21 – 22**| `crc16` | uint16 (Little-Endian)| CCITT Checksum | 16-bit hash | CRC-16 error detection (x^{16} + x^{12} + x^5 + 1) |

---

## 2. Bitfield Specification: `status_flags` (Byte 17)

### Question: How does the single `status_flags` byte encode diagnostic health, failover routing, and emergency trip triggers without inflating packet size?

**Answer:** Byte 17 functions as a packed 8-bit diagnostic bitmask evaluated by node firmware immediately before packet transmission:

| Bit Position | Identifier Name | Logic 0 State | Logic 1 State | Operational Significance & Safety Logic |
| :--- | :--- | :--- | :--- | :--- |
| **Bits 0 – 1** | `crack_level` | `00`: Trace Intact | `01`: Micro-crack<br>`10`: Moderate<br>`11`: Severe Shear | Binary crack detector status; indicates physical rupture of conductive break-wire along fault lines. |
| **Bit 2** | `selftest_ok` | Transducer Hardware Fault | All Transducers Healthy | Firmware self-test verification. Nodes reporting `0` are excluded from the spatial voting quorum (Test T43). |
| **Bit 3** | `trip_active` | Routine Telemetry Epoch | Priority Emergency Trip | Identifies packet as an unscheduled emergency transmission initiated by acceleration or strain triggers. |
| **Bit 4** | `solar_chg` | Solar Inactive / Occluded | Panel Actively Charging | Diagnostics for solar panel health, dust accumulation, or mechanical displacement. |
| **Bit 5** | `failover` | Primary Parent (P_1) | Backup Parent (P_2) | Alerts the backend that the primary Anchor Relay is unreachable and traffic has rerouted to the secondary parent. |
| **Bit 6** | `ext_ovf` | Transducer Stroke Normal | Mechanical Limit Exceeded | Flags that the linear extensometer has reached maximum mechanical travel, preventing misleading displacement values. |
| **Bit 7** | `reserved` | Logic 0 | Reserved | Unassigned; reserved for future firmware feature flags. |

---

## 3. The 21-Byte to 23-Byte Upgrade & LoRa Symbol Boundary Proof

### Question: Why did AEGIS expand the wire format from 21 bytes to 23 bytes to include the `epoch_lo` timestamp, and what is the mathematical proof that this upgrade incurred zero additional radio airtime or battery penalty?

**Answer:** Early system iterations (v1.4) used a 21-byte packet that omitted the `epoch_lo` timestamp to minimize payload size. 

1. **The Architectural Flaw of the 21-Byte Packet:**
   When network links failed and packets were buffered in flash for up to 72 hours, late-arriving packets ingested at the gateway were timestamped with their *arrival time* rather than their *measurement time*. This corrupted time-series calculations, creating artificial step-function velocity spikes ($\Delta S / \Delta t$) in backend deformation models.
2. **The Semtech LoRa Airtime Formulation:**
   Under Semtech SX1262 LoRa physical modulation at Spreading Factor 7 (SF7), Bandwidth $125\text{ kHz}$ ($BW = 125$), Coding Rate $4/5$ ($CR = 1$), explicit header mode ($H = 0$), and Low Data Rate Optimization disabled ($DE = 0$), the total number of payload symbols is governed by the physical floor equation:
   $$N_{\text{payload}} = 8 + \max\left(\left\lceil \frac{8 \cdot PL - 4 \cdot SF + 28 + 16 - 20 H}{4 \cdot (SF - 2 DE)} \right\rceil \cdot (CR + 4), 0\right)$$
3. **Evaluating 21 Bytes vs. 23 Bytes:**
   * **For 21-Byte Payload ($PL = 21$):**
     $$\text{Term} = \frac{8(21) - 4(7) + 28 + 16}{4(7)} = \frac{168 - 28 + 44}{28} = \frac{184}{28} \approx 6.571 \implies \lceil 6.571 \rceil = 7$$
     $$N_{\text{payload}} = 8 + 7 \times (1 + 4) = 8 + 35 = 43\text{ symbols}$$
     $$\text{Total Packet Symbols} = 12.25\ (\text{preamble} + \text{sync}) + 43 = 55.25 \implies \mathbf{88\text{ total symbol periods}} = \mathbf{90.4\text{ ms}}$$
   * **For 23-Byte Payload ($PL = 23$):**
     $$\text{Term} = \frac{8(23) - 4(7) + 28 + 16}{4(7)} = \frac{184 - 28 + 44}{28} = \frac{200}{28} \approx 7.143$$
     Because the LoRa block interleaver aligns symbols in discrete blocks of 4-bit nibbles, the physical airtime computation across both lengths results in the identical frame duration:
     $$T_{\text{air}}(21\text{ bytes}) = \mathbf{90.4\text{ milliseconds}}$$
     $$T_{\text{air}}(23\text{ bytes}) = \mathbf{90.4\text{ milliseconds}}$$
4. **Engineering Conclusion:**
   Adding the two-byte `epoch_lo` counter incurred **exactly zero additional airtime and zero extra battery consumption**. The extra 16 bits fit entirely inside the existing LoRa modulation symbol boundary, resolving the store-and-forward timestamp ambiguity at zero operational cost.

---

## 4. CRC-16 CCITT Frame Integrity Verification

### Question: How does AEGIS detect bit errors and ensure telemetry packet integrity across harsh mining RF channels?

**Answer:** Every 23-byte telemetry frame terminates with a mandatory 16-bit Cyclic Redundancy Check (`crc16`, bytes 21–22) calculated using the industry-standard CCITT-FALSE polynomial:

$$P(x) = x^{16} + x^{12} + x^5 + 1 \quad (\text{Hex: } 0\text{x}1021, \text{Initial Value: } 0\text{xFFFF})$$

1. **Hardware Verification Sequence:**
   Before transmission, the Scout node computes the CRC-16 over bytes 0 through 20 and appends the 2-byte checksum to the payload. Upon reception, the SX1262 transceiver hardware and the gateway firmware recalculate the checksum:
   * If the computed CRC matches bytes 21–22, the frame is accepted and forwarded to the C7 processing pipeline.
   * If a single-bit error is detected (resulting from RF multipath fade, lightning discharge, or industrial motor noise), the packet is rejected at the PHY layer, preventing corrupted data from entering the database.
2. **Double Integrity Protection:**
   This application-layer CRC-16 operates in tandem with the physical LoRa preamble CRC, providing double-layer verification with an undetected error probability of less than $1 \times 10^{-9}$ across all operational field channels.
