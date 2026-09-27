# The 7-Sensor Suite & Transducer Specifications

**Module 02 — Sensor Hardware**  
**Cross-References:** [`node-classification.md`](node-classification.md) · [`bill-of-materials.md`](bill-of-materials.md) · [Module 04 Physics Engine](../04-physics-engine/derived-quantities.md)

---

## 1. Complete Transducer Specification

The physical sensor suite comprises seven on-board measurement channels, complemented by radio link provenance metrics logged at the gateway.

| # | Sensor Modality | Primary Hardware Part | Physical Input | Output Channels | Wire Format | Dynamic Range | Measurement Role |
|---|---|---|---|---|---|---|---|
| **1** | **Tilt / Inclination** | InvenSense MPU-6050 | Gravity vector relative to PCB plane | `tilt_x`, `tilt_y` | int16 (LSB: 2 µrad) | $\pm 65,534\ \mu\text{rad}$ ($\pm 3.75^\circ$) | **Secondary vote.** Subject to thermal drift; never trips alarms independently. |
| **2** | **Die Temperature** | MPU-6050 Internal Sensor | Semiconductor junction temperature | `temp_dc` | int16 ($\times 0.1^\circ\text{C}$) | $-40^\circ\text{C} \to +85^\circ\text{C}$ | **Correction key.** Required to remove temperature drift from tilt and strain channels. |
| **3** | **Horizontal Strain** | Linear Potentiometer / Foil + ADS1115 | Ground stretch / compression over 10m | `strain_ue` | int16 ($1\ \mu\varepsilon$) | $\pm 32,767\ \mu\varepsilon$ | **Primary detector.** Primary precursor signal for tensile cracking. |
| **4** | **Extensometer** | 10m/30m Invar Wire / Draw Wire | 3D relative displacement between pegs | `ext_delta_10um` | int16 ($\times 10\ \mu\text{m}$) | $\pm 327.6\text{ mm}$ | **Primary detector.** Directly tracks differential baseline displacement. |
| **5** | **Crack Detector** | Conductive substrate trace | Structural surface continuity break | `status_flags` (bits 0–1) | uint8 (2 bits) | Level 0–3 | **Binary confirmation.** Irreversible latch indicating physical fissure rupture. |
| **6** | **Ground Vibration** | 3-axis Accelerometer burst | High-frequency ground shaking | `vib_rms`, `vib_peak`, `vib_fdom` | uint16 $\times 2$, uint8 | 0–655 mm/s, 0–255 Hz | **Veto discriminator.** Differentiates blasts, trucks, and rock fracture. |
| **7** | **Battery Monitor** | Precision resistor divider + ADC | Cell terminal voltage | `vbat_mv` | uint16 ($1\text{ mV}$) | $3000\text{ to }4200\text{ mV}$ | **Correction key & health.** Removes voltage sag bias; tracks power state. |
| — | **Radio Provenance** | Semtech SX1302 PHY (Gateway) | Uplink RF path characteristics | `rssi_dbm`, `snr_db`, `hops` | int16, f32, uint8 | $-140 \to -40\text{ dBm}$ | **Dynamic $\sigma$ scaling.** Scales measurement uncertainty; never fed to PINN. |

---

## 2. "Two Sensors That Are Not Sensors"

Channels 2 (`temp_dc`) and 7 (`vbat_mv`) do not measure ground mechanics. They exist to make the remaining five mechanical channels mathematically trustworthy:

1. **Temperature Drift Removal:**
   Silicon MEMS accelerometers exhibit a documented thermal sensitivity of approximately $k_T = 250\ \mu\text{rad/}^\circ\text{C}$. Over an open-pit or subsidence panel where diurnal temperatures swing by $25^\circ\text{C}$, raw uncorrected thermal drift produces over $6,250\ \mu\text{rad}$ of apparent movement—swamping subtle early subsidence signals. Measuring exact internal die temperature enables the backend C7 pipeline to subtract thermal expansion and bias drift before evaluating safety thresholds.
2. **Battery Voltage Sag Compensation:**
   As cell voltage declines from $4.2\text{V}$ (fully charged) down to $3.3\text{V}$ (depleted), unbuffered analogue-to-digital references drop slightly. Measuring $V_{\text{bat}}$ allows the C7 pipeline to linearize sensor readings against battery discharge curves.

---

## 3. Preservation of Raw Field Data

A fundamental design constraint is that **nodes do not clean, calibrate, or filter their own data**:
* If an embedded microcontroller applies an onboard moving average, thermal offset, or noise filter, the raw physical measurement is permanently destroyed.
* A corrupted calibration coefficient on a remote microcontroller would silently falsify readings with no possibility of forensic audit.
* All raw bit-level observations are packed into the 23-byte wire format and transmitted directly. All 8 stages of cleaning, calibration, and temperature compensation occur inside the auditable, open-source backend pipeline (`C7`).
