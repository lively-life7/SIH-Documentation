# Constants Reference Ledger (`sim/constants.py`)

**Appendices**  
**Cross-References:** [`glossary.md`](glossary.md) · [`sigma-formula.md`](sigma-formula.md) · [Module 04 Physics Engine](../04-physics-engine/knothe-model.md)

---

## 1. Single Source of Truth

All mathematical simulation models, embedded firmware profiles, backend calibration pipelines (`C7`), and detection engines (`C8`) import their numerical parameters directly from a single immutable Python module: `sim/constants.py`. If these values diverge between modules, training loss fails to converge.

---

## 2. Ground Mechanics & Panel Geometry

| Constant Name | Value | Engineering Units | Usage & Ownership |
| :--- | :--- | :--- | :--- |
| `H` | `150.0` | meters | Overburden seam depth below surface datum; surface & PINN (Fixed) |
| `TAN_BETA` | `2.0` | dimensionless | Tangent of major draw angle (β ≈ 63.4°); surface & PINN (Fixed) |
| `R_INFL` | `75.0` | meters | Radius of influence (H / \tanβ); surface & PINN (Fixed) |
| `M_SEAM` | `3.0` | meters | Total extracted coal seam thickness; surface & PINN (Fixed) |
| `A_SUBS` | `0.65` | dimensionless | True subsidence factor; **surface ONLY** (PINN must learn this) |
| `S_MAX` | `1.95` | meters | Maximum asymptotic center subsidence (A · M) |
| `C_KNOTHE` | `0.01414` | day⁻¹ | True time decay coefficient; **surface ONLY** (PINN must learn this) |
| `B_HORIZ` | `24.0` | meters | Awershin horizontal displacement ratio (0.32 · r) |
| `PANEL` | `(100, 100, 700, 300)` | meters | Rectangular extraction boundary coordinates (x_1, y_1, x_2, y_2) |
| `GRID` | `(25, 775, 25, 375, 64, 64)` | meters | Output evaluation mesh matrix (64 × 64) |

---

## 3. Sensor Noise, Drift & Corruption Parameters

| Channel | Thermal Sensitivity (k_T) | Random Drift (\sigma_b) | White Noise (\sigma_w) | LSB Resolution |
| :--- | :--- | :--- | :--- | :--- |
| **Tilt (T_x, T_y)** | 250 µrad/°C | 3 µrad | 8 µrad | **2 µrad** |
| **Horizontal Strain (ε)** | 5 µε/°C | 0.5 µε | 1.2 µε | **1 µε** |
| **Extensometer (Ext)** | 12 µm/°C | 5 µm | 15 µm | **10 µm** |
| **Die Temperature (T_die)** | — | — | 0.05°C | **0.1°C** |
| **Battery Voltage (V_bat)** | — | — | 5 mV | **1 mV** |

* Reference Temperature: `T_REF = 25.0 °C`
* Drift Correlation Time: `TAU_OU = 21600.0 s` (6.0 hours)
* Nominal Battery Voltage: `V_NOM = 3.70 V`

---

## 4. Vibration & Industrial Event Parameters

| Constant Name | Value | Physical Meaning |
| :--- | :--- | :--- |
| `PPV_K`, `PPV_EXP` | `1140.0, -1.6` | USBM explosive charge ground attenuation coefficients |
| `Q_EQ_TRUCK` | `0.14 kg` | Equivalent explosive blast charge simulating a loaded haul truck |
| `CONV_A0`, `CONV_D0` | `0.8 mm/s, 120.0 m` | Armored face conveyor baseline vibration and spatial decay |
| `THETA_C` | `1500.0 µε` | Critical tensile rock fracture initiation threshold (± 10%) |
| `F_DOM_BANDS` | `[8-20, 40-80, 50±0.5, 100-250]` | Dominant frequency classification bins (Truck, Blast, Conveyor, Fracture) |

---

## 5. Wireless Radio & TDMA Network Parameters

| Constant Name | Value | Regulatory / Technical Note |
| :--- | :--- | :--- |
| `RF_BAND` | `IN865 (865–867 MHz)` | Indian delicensed frequency band per GSR 564(E) |
| `CARRIER_BW` | `125.0 kHz` | Statutory compliance: Strictly ≤ 200 kHz (Test T18) |
| `TX_POWER_CONDUCTED`| `30.0 dBm (1.0 W)` | Maximum transmitter conducted power ceiling |
| `ERP_DESIGN` | `30.0 dBm (1.0 W)` | Design effective radiated power (Legal ceiling: 36 dBm) |
| `DUTY_CYCLE_LIMIT` | `1.0% (0.010)` | Self-imposed ETSI/LoRaWAN operational limit |
| `PRESET_LEAF` | `SF7 / BW125` | 90.4 ms packet duration; 0.10% measured duty cycle |
| `PRESET_RELAY` | `SF8 / BW125` | 406.0 ms bundle duration; 0.56% measured duty cycle |
| `WIRE_PACKET_BYTES` | `23 bytes` | Fixed binary payload format (includes `epoch_lo`) |
| `SUPERFRAME_S` | `60.0 seconds` | Master TDMA scheduling period |
| `LEAF_SLOT_MS` | `250.0 ms` | Individual cluster leaf uplink window |
| `RELAY_SLOT_MS` | `600.0 ms` | Trunk relay bundle transmission window |
| `BACKUP_SLOTS_N` | `16 slots` | Dedicated collision-free emergency slots (150 ms each) |
| `HOP_LIMIT` | `3 hops` | Maximum network forwarding diameter (2 nominal, 3 failover) |

---

## 6. Retention & Analytical Windows

| Constant Name | Value | Operational Function |
| :--- | :--- | :--- |
| `retention_h` | `36 hours` | Sliding window of live telemetry maintained in `nodes.csv` |
| `buffer_frames` | `4,320 frames` | 72-hour on-node SPI flash store-and-forward ring buffer |
| `dedup_ring_entries` | `64 entries` | Sliding memory window for suppressing duplicate LoRa frames |
| `baseline_window_h` | `24 hours` | Rolling median baseline window for C8 threshold checks |
| `pinn_window_h` | `24 hours` | Active training window for C9 neural surface reconstruction |
| `pinn_window_slices`| `48 slices` | Subsampled 30-minute time slices guaranteeing parameter identifiability |
