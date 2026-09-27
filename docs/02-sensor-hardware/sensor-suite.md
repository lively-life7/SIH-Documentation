# The 7-Sensor Suite & Transducer Specifications

**Module 02 — Sensor Hardware**  
**Cross-References:** [`node-classification.md`](node-classification.md) · [`bill-of-materials.md`](bill-of-materials.md) · [`edge-intelligence.md`](edge-intelligence.md) · [Module 04 Physics Engine](../04-physics-engine/derived-quantities.md) · [Module 07 Data Pipeline](../07-verification-and-analysis/c7-data-pipeline.md)

---

## 1. Complete Transducer Specification and Geotechnical Roles

### Question: What are the physical measurement channels integrated into the AEGIS sensor suite, and what are their hardware transducers, wire formats, dynamic ranges, and geotechnical roles?

**Answer:** The AEGIS hardware platform captures seven on-board measurement channels, complemented by RF provenance telemetry recorded by the gateway receiver. Each channel maps directly to a physical failure indicator:

| # | Sensor Modality | Primary Hardware Part | Physical Input | Wire Field | Format & LSB Resolution | Dynamic Range | Geotechnical Measurement Role |
|:---|:---|:---|:---|:---|:---|:---|:---|
| **1** | **Tilt / Inclination** | InvenSense MPU-6050 | Static gravity vector projection ($\vec{g}$) relative to sensor PCB | `tilt_x`, `tilt_y` | int16 ($2\ \mu\text{rad/LSB}$) | $\pm 65,534\ \mu\text{rad}$ ($\approx \pm 3.75^\circ$) | **Secondary Spatial Indicator:** Measures ground rotation. High thermal sensitivity prevents tilt from triggering standalone safety trips. |
| **2** | **Die Temperature** | MPU-6050 Internal Thermal Diode | On-chip silicon junction temperature | `temp_dc` | int16 ($0.1^\circ\text{C/LSB}$) | $-40.0^\circ\text{C}\text{ to }+85.0^\circ\text{C}$ | **Metrological Correction Key:** Required to strip thermal expansion and bias drift from tilt and strain sensors. |
| **3** | **Horizontal Ground Strain** | Linear Slide Potentiometer / Foil Gauge + TI ADS1115 | Horizontal ground extension or compression over a 10m baseline | `strain_ue` | int16 ($1\ \mu\varepsilon\text{/LSB}$) | $\pm 32,767\ \mu\varepsilon$ | **Primary Early Warning:** Detects differential horizontal ground stretching weeks before visible surface cracking. |
| **4** | **Extensometer Displacement** | 10m/30m Invar Wire / Draw-Wire Potentiometer | Differential linear distance change between anchored ground pegs | `ext_delta_10um` | int16 ($10\ \mu\text{m/LSB}$) | $\pm 327.6\text{ mm}$ | **Primary Mechanical Indicator:** Satisfies DGMS statutory guidelines for tracking relative displacement across extraction boundaries. |
| **5** | **Crack Fissure Trip** | Conductive Break-Wire Substrate Trace | Surface rock mass tensile shear rupture | `status_flags` (bits 0–1) | uint8 (2 bits) | Level 0 to 3 (Intact to Sheared) | **Binary Physical Confirmation:** Irreversible hardware trigger confirming active surface rupture along known fault lines. |
| **6** | **Ground Vibration Burst** | 3-axis MEMS Accelerometer (400 Hz burst + FFT) | High-frequency ground oscillations | `vib_rms_x100`, `vib_peak_x100`, `vib_fdom_hz` | uint16 $\times 2$, uint8 ($0.01\text{ mm/s}$, $1\text{ Hz}$) | $0\text{ to }655\text{ mm/s}$, $0\text{ to }200\text{ Hz}$ | **Spectral Veto Discriminator:** Distinguishes heavy dumper trucks and production blasts from rock mass fracturing. |
| **7** | **Battery Voltage** | Precision 1% Thin-Film Resistor Divider + ADC | LiFePO4 cell terminal voltage | `vbat_mv` | uint16 ($1\text{ mV/LSB}$) | $2,500\text{ to }4,200\text{ mV}$ | **Correction Key & Node Health:** Eliminates ADC reference sag bias; provides operational state-of-charge monitoring. |
| — | **Radio Provenance** | Semtech SX1302 Concentrator PHY | Uplink RF channel physical characteristics | `rssi_dbm`, `snr_db`, `hops` | int16, f32, uint8 ($1\text{ dBm}$, $0.1\text{ dB}$) | $-140\text{ to }-40\text{ dBm}$ | **Measurement Uncertainty Weight:** Dynamically scales Kalman filter measurement noise covariance ($R$); never fed into physical mechanics models. |

---

## 2. "Sensors That Are Not Sensors": Temperature and Battery Correction Keys

### Question: Why are semiconductor die temperature (`temp_dc`) and battery terminal voltage (`vbat_mv`) classified as "Sensors That Are Not Sensors"?

**Answer:** Channels 2 (`temp_dc`) and 7 (`vbat_mv`) do not measure geotechnical ground movement. They exist solely to make the remaining five mechanical channels mathematically trustworthy by eliminating environmental and electrical bias:

```
+-----------------------------------------------------------------------------------+
|                        METROLOGICAL COMPENSATION WORKFLOW                         |
|                                                                                   |
|  Raw Tilt Reading (θ_raw) ─────────┐                                              |
|                                    ▼                                              |
|  Die Temp Reading (temp_dc) ───> [ C7 Thermal Compensation ] ──> θ_corrected      |
|                                  (Subtracts 250 µrad / °C)                        |
|                                                                                   |
|  Raw ADC Channel (V_raw) ──────────┐                                              |
|                                    ▼                                              |
|  Battery Voltage (vbat_mv) ────> [ ADC Reference Ratiometric ] ─> Scaled Physical |
|                                  [ Linearization Engine      ]    Displacement    |
+-----------------------------------------------------------------------------------+
```

1. **Thermal Sensitivity Removal ($k_T$):**
   MEMS capacitive silicon accelerometers possess a significant, repeatable thermal coefficient of sensitivity:
   $$k_T \approx 250\ \mu\text{rad}/^\circ\text{C}$$
   In open Indian coalfields, surface ambient temperatures undergo diurnal swings exceeding $\Delta T = 25^\circ\text{C}$ between midday sun and pre-dawn chill. Uncompensated, this thermal gradient introduces an apparent tilt error of:
   $$\text{Error}_{\text{thermal}} = 25^\circ\text{C} \times 250\ \mu\text{rad}/^\circ\text{C} = 6,250\ \mu\text{rad} \approx 0.358^\circ$$
   This thermal artifact is more than $100\times$ larger than the genuine pre-failure ground creep rate ($50\ \mu\text{rad/day}$), which would trigger severe false evacuation alarms. By recording `temp_dc` at the exact instant of tilt capture, the backend C7 pipeline subtracts the thermal bias:
   $$\theta_{\text{corrected}} = \theta_{\text{raw}} - k_T (T_{\text{die}} - T_{\text{cal}})$$
2. **Battery Voltage Sag Compensation:**
   As a LiFePO4 battery discharges over months of operation, subtle changes in rail impedance can introduce minor ratiometric offsets into unbuffered analog transducers (e.g., potentiometer voltage dividers). By pairing every analog capture with an exact millivolt battery reading (`vbat_mv`), the C7 calibration engine normalizes analog readings against the measured excitation rail.

---

## 3. Preservation of Raw Field Data & Metrological Integrity

### Question: Why does AEGIS strictly prohibit edge sensor nodes from applying onboard calibration offsets, thermal compensation, or moving-average filters to their raw data?

**Answer:** Prohibiting edge filtering is an uncompromising principle of high-integrity forensic metrology and safety engineering:

1. **Immutability of Physical Observations:**
   If a remote microcontroller applies an onboard rolling average, high-pass filter, or thermal calibration formula, the true raw physical measurement is permanently destroyed. If a slope failure or subsidence incident occurs, regulatory bodies (such as DGMS) require an auditable forensic record of unaltered raw bit readings.
2. **Elimination of Silent Field Calibration Drift:**
   Over a multi-year deployment, onboard calibration constants stored in microcontroller flash memory can become corrupted due to power brownouts, electromagnetic interference, or accidental over-the-air parameter misconfigurations. If edge nodes applied local corrections, a corrupted coefficient would silently falsify ground deformation telemetry without the gateway detecting the corruption.
3. **Deterministic Separation of Responsibilities:**
   Sensor nodes operate strictly as unopinionated data acquisition units, serializing raw ADC integer counts into the 23-byte wire payload. All calibration curves, polynomial thermal corrections ($k_T$), baseline subtractions, and Kalman state estimation occur deterministically within the centralized, version-controlled **C7 Pipeline**. If an improved calibration curve or thermal model is developed, it can be retroactively applied across the entire historical raw dataset.

---

## 4. Radio Provenance and Uncertainty Quantification

### Question: How does the system utilize radio link metrics (RSSI, SNR, and hop count) without corrupting geotechnical deformation models?

**Answer:** Radio provenance metrics (`rssi_dbm`, `snr_db`, and `hops`) are logged at the physical RF gateway layer and kept strictly isolated from physical geomechanical models:
* **Dynamic Uncertainty Weighting ($\sigma^2$):** In the backend Extended Kalman Filter (EKF), radio provenance is used to dynamically adjust the observation noise covariance matrix $R$:
  $$R_{k} = R_0 \cdot f(\text{SNR}, \text{Hops}, \text{Latency})$$
  When a node transmits with degraded SNR (e.g., $<-10\text{ dB}$) or over multiple hops, packet delivery latency and jitter increase. The filter inflates the measurement variance $\sigma^2$ for that epoch, reducing the weight given to the observation without corrupting physical state estimates.
* **Strict Model Quarantine:** RF metadata is never fed into Knothe subsidence equations, boundary element models, or Physics-Informed Neural Networks (PINNs). Physical models ingest strictly geometric and mechanical variables (tilt, displacement, strain), ensuring geotechnical predictions remain independent of radio propagation anomalies.
