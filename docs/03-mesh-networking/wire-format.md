# Binary Wire Format & Packet Structure

**Module 03 — Mesh Networking**  
**Cross-References:** [`tdma-scheduling.md`](tdma-scheduling.md) · [`store-and-forward.md`](store-and-forward.md) · [Module 06 Data Architecture](../06-backend-pipeline/data-architecture.md)

---

## 1. Byte-by-Byte Wire Map (23 Bytes)

To minimize RF airtime, eliminate serialization overhead, and comply with Indian statutory bandwidth limits, all sensor telemetry is packed into a compact **23-byte fixed-width binary struct**.

```
 0                   1                   2                   
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    tilt_x     |    tilt_y     |   strain_ue   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| ext_delta_10um|  vib_rms_x100 | vib_peak_x100 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|vib_f| temp_dc |    vbat_mv    |flags| epoch_lo|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  nid  |     crc16     |
+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Detailed Field Specification

| Offset | Field Name | Data Type | Units / Scaling | Valid Range | Physical Meaning |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0 – 1** | `tilt_x` | int16 (LE) | 2 µrad / LSB | $\pm 65,534\ \mu\text{rad}$ | Transverse ground inclination angle |
| **2 – 3** | `tilt_y` | int16 (LE) | 2 µrad / LSB | $\pm 65,534\ \mu\text{rad}$ | Longitudinal ground inclination angle |
| **4 – 5** | `strain_ue` | int16 (LE) | $1\ \mu\varepsilon$ | $\pm 32,767\ \mu\varepsilon$ | Horizontal ground strain across 10m base |
| **6 – 7** | `ext_delta_10um`| int16 (LE) | $10\ \mu\text{m}$ / LSB | $\pm 327.6\text{ mm}$ | 3D relative peg displacement |
| **8 – 9** | `vib_rms_x100` | uint16 (LE)| $0.01\text{ mm/s}$ | $0.00 \to 655.35\text{ mm/s}$ | RMS vibration intensity during burst |
| **10 – 11** | `vib_peak_x100`| uint16 (LE)| $0.01\text{ mm/s}$ | $0.00 \to 655.35\text{ mm/s}$ | Peak Particle Velocity (PPV) |
| **12** | `vib_fdom_hz` | uint8 | $1\text{ Hz}$ | $0 \to 255\text{ Hz}$ | Dominant FFT spectral frequency |
| **13 – 14** | `temp_dc` | int16 (LE) | $0.1^\circ\text{C}$ | $-40.0 \to +85.0^\circ\text{C}$ | Internal sensor die temperature |
| **15 – 16** | `vbat_mv` | uint16 (LE)| $1\text{ mV}$ | $2500 \to 4500\text{ mV}$ | Battery supply voltage |
| **17** | `status_flags` | uint8 | Bitmask | 8 discrete bits | Diagnostic and operational state |
| **18 – 19** | `epoch_lo` | uint16 (LE)| 1 epoch (60s) | $0 \to 65,535$ | Lower 16 bits of 60-second sample clock |
| **20** | `node_id` | uint8 | Integer ID | $1 \to 255$ | Physical hardware identifier |
| **21 – 22** | `crc16` | uint16 (LE)| Checksum | Standard CCITT | CRC-16-CCITT ($x^{16} + x^{12} + x^5 + 1$) |

---

## 2. Bitfield Specification: `status_flags` (Byte 17)

The `status_flags` byte provides single-bit diagnostic flags evaluated at the instant of packet assembly:

| Bit | Name | Logic 0 | Logic 1 | Significance |
| :--- | :--- | :--- | :--- | :--- |
| **0 – 1** | `crack_level` | 00: Intact | 01: Micro, 10: Mod, 11: Severe | Conductive trip wire fracture state |
| **2** | `selftest_ok` | Sensor Hardware Fault | Transducers Healthy | Excludes faulty nodes from quorum (Test T43) |
| **3** | `trip_active` | Routine Sampling | Priority Emergency Trip | Marks frame as emergency transmission |
| **4** | `solar_chg` | Solar Panel Inactive | Panel Charging Battery | Power subsystem diagnostic |
| **5** | `failover` | Primary Parent Active | Backup Parent Active | Indicates upstream mesh re-route |
| **6** | `ext_ovf` | Extensometer Normal | Stroke Limit Reached | Warns of mechanical stroke saturation |
| **7** | `reserved` | 0 | 0 | Reserved for future firmware expansion |

---

## 3. The 21-Byte to 23-Byte Upgrade Rationale

Early design iterations (v1.4) used a 21-byte packet lacking the `epoch_lo` timestamp field:
* **The Failure:** When a node lost radio connection and replayed stored frames 40 hours later via the store-and-forward ring buffer, the backend assigned the *arrival time* rather than the *measurement time*, corrupting subsidence time-series data.
* **The Semtech Airtime Identity:**
  Under Semtech LoRa modulation math at Spreading Factor 7 (SF7), Bandwidth 125 kHz, Coding Rate 4/5, and explicit header mode, packet airtime is determined by:
  $$N_{\text{symbols}} = 8 + \max\left(\left\lceil \frac{8 \cdot PL - 4 \cdot SF + 28 + 16 - 20 H}{4 \cdot (SF - 2 DE)} \right\rceil \cdot (CR + 4), 0\right)$$
  Substituting $PL = 21\text{ bytes}$ yields $88\text{ symbols} \implies \mathbf{90.4\text{ ms}}$.  
  Substituting $PL = 23\text{ bytes}$ yields the **exact same 88 symbols** $\implies \mathbf{90.4\text{ ms}}$.
* **Conclusion:** Adding 2 bytes for the essential `epoch_lo` timestamp incurred **zero airtime penalty** because the additional 16 bits did not cross a 4-bit LoRa symbol block boundary. This provided permanent store-and-forward correctness at zero airtime cost.
