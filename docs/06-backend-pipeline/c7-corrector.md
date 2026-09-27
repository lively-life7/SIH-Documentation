# C7 Corrector: The 8-Step Calibration & Cleaning Pipeline

**Module 06 — Backend Pipeline**  
**Cross-References:** [`data-architecture.md`](data-architecture.md) · [`c8-alarm-engine.md`](c8-alarm-engine.md) · [Module 04 Corruption Chain](../04-physics-engine/corruption-chain.md)

---

### Question: What is the primary role of the C7 Corrector pipeline in the AEGIS system architecture?
**Answer:** The C7 Corrector pipeline is the sole backend authority responsible for transforming raw, degraded, integer-quantized wireless telemetry into clean, calibrated physical engineering units (radians, microstrain, millimeters, and mm/s). Raw bitstreams received from field mesh transceivers suffer from multi-physics environmental corruption: analogue-to-digital converter (ADC) voltage reference sag due to battery discharge, silicon die temperature drift, regional atmospheric ground swelling, and packet drops over the wireless channel. Downstream algorithms—specifically the deterministic C8 Safety Alarm Engine and the C9 Physics-Informed Neural Network (PINN) digital twin—require mathematically consistent, noise-bounded physical inputs. C7 enforces an immutable 8-step cleaning pipeline to strip sensor-level and environmental corruption before safety checks occur.

```
[Raw 23-Byte Unpacked Telemetry Frame]
               │
               ▼  Step 1: Assemble Epoch Bucket
[Align temporal bucket; mark missing packets as NaN (NEVER ZERO-FILL)]
               │
               ▼  Step 2: Decode Diagnostic Flags
[Unpack status_flags into 7 operational booleans]
               │
               ▼  Step 3: Undo Battery Voltage Sag
[Compensate analogue ADC reference shift vs V_nom = 3.7V]
               │
               ▼  Step 4: Undo Die Thermal Drift
[Subtract k_T · (T_die - 25.0°C) from tilt and strain]
               │
               ▼  Step 5: Convert to Standard SI Units
[Scale integers to radians, microstrain, millimeters, mm/s]
               │
               ▼  Step 6: Common-Mode Rejection (CMR)
[Subtract undisturbed Bedrock Anchor baseline from active stations]
               │
               ▼  Step 7: Compute Dynamic Channel Uncertainty (σ)
[Scale σ_base by link margin, packet gaps, and diagnostic flags]
               │
               ▼  Step 8: Construct Validity Mask
[Emit N_nodes × 4 boolean mask gating downstream algorithms]
               │
               ▼
[Cleaned Calibrated Physical State → Handed to C8 Detector & C9 PINN]
```

---

### Question: How does Step 1 (Epoch Assembly) handle missing packets, and why is zero-filling strictly prohibited?
**Answer:** Step 1 synchronizes asynchronous packets arriving from distributed nodes into synchronized discrete time buckets (epochs) corresponding to the network TDMA schedule. When a node fails to report within its assigned epoch window due to RF fading, packet collision, or temporary radio shadow, the C7 pipeline populates the missing telemetry fields with IEEE-754 `NaN` (Not a Number). 

Zero-filling is strictly prohibited because physical ground displacement, strain, and tilt are continuous, cumulative state variables. Inserting a numerical value of zero into missing slots introduces artificial infinite-gradient step discontinuities:

$$\lim_{\Delta t \to 0} \frac{0 - y(t-\Delta t)}{\Delta t} = -\infty$$

Such artificial impulses falsely indicate sudden structural unloading or catastrophic collapse, causing catastrophic false alarm trips in C8 threshold detectors, destabilizing Kalman filter covariance matrices, and corrupting loss gradients in the C9 PINN digital twin. All downstream operations in C7 propagate `NaN` values safely through arithmetic operations, which are subsequently masked out in Step 8.

---

### Question: What diagnostic status flags are unpacked in Step 2, and what operational states do they govern?
**Answer:** In Step 2, C7 decodes the 8-bit `status_flags` field transmitted in the telemetry frame into seven independent operational booleans. These flags indicate real-time hardware health and trigger specific calibration adjustments or algorithmic disqualifications:

| Bit Index | Flag Identifier | Binary Value | Physical Condition & Algorithmic Impact |
| :--- | :--- | :--- | :--- |
| **Bit 0** | `selftest_ok` | `1` = Pass, `0` = Fail | Transducer self-test circuit status; if `0`, node is stripped of C8 voting rights. |
| **Bit 1** | `accel_sat` | `1` = Saturated | MEMS accelerometer clipping ($> \pm 2g$); disqualifies tilt calculation. |
| **Bit 2** | `gauge_open` | `1` = Open/Short | Wheatstone bridge lead wire severed or shorted; forces strain channel invalid. |
| **Bit 3** | `i2c_timeout` | `1` = Bus Error | Digital sensor bus lockup; triggers hardware watchdog reboot flag. |
| **Bit 4** | `solar_active`| `1` = Charging | Photovoltaic harvesting active; alerts thermal model to expect rapid solar heating. |
| **Bit 5** | `low_vbat` | `1` = Low Power | Battery $< 3.2\text{ V}$; enables aggressive power conservation and flags ADC risk. |
| **Bit 6** | `tamper_trip` | `1` = Enclosure Open | Mechanical enclosure lid switch triggered; alerts operator to physical tampering. |

---

### Question: Why must Battery Voltage Sag compensation (Step 3) strictly precede Die Thermal Drift compensation (Step 4)? What physical failure mode occurs if this sequence is inverted (Test T7)?
**Answer:** The execution sequence of Step 3 (sag compensation) before Step 4 (thermal compensation) is mathematically and physically non-commutative. 

The physical mechanism is governed by the microcontroller's internal Analogue-to-Digital Converter (ADC). The ADC measures input voltages relative to an analogue supply reference voltage ($V_{\text{ref}}$). As the station's lithium battery discharges from its nominal $V_{\text{nom}} = 3.7\text{ V}$ down to $3.2\text{ V}$, $V_{\text{ref}}$ sags proportionally. This reference sag uniformly expands the digital counts for all connected analog transducers:

$$\text{ADC}_{\text{count}} \propto \frac{V_{\text{sensor}}}{V_{\text{ref}}(V_{\text{bat}})}$$

Crucially, the silicon die temperature diode is itself an internal analogue transducer measured by this same sagging reference. If thermal drift compensation were executed first using the uncorrected raw temperature reading, the algorithm would compute thermal subtraction based on a fictitious, sag-distorted temperature value ($T_{\text{distorted}}$):

$$T_{\text{distorted}} = \frac{T_{\text{raw}} \cdot 0.1}{1.0 + \alpha_{\text{sag}} \left(\frac{V_{\text{nom}} - V_{\text{bat}}}{V_{\text{nom}}}\right)} \ne T_{\text{true}}$$

This error propagates directly into the strain correction:

$$\varepsilon_{\text{erroneous}} = \varepsilon_{\text{raw}} - k_T \cdot (T_{\text{distorted}} - T_{\text{ref}})$$

To guarantee physical fidelity, Step 3 must execute first to rescale all ADC counts back to nominal $V_{\text{nom}}$ linearity. Once linear counts are restored, Step 4 calculates true die temperature and subtracts pure thermal drift:

```python
# C7 Calibration Execution Block (Python / NumPy)
def clean_epoch_telemetry(raw_row, node_cfg):
    # Step 3: Undo Battery Voltage Sag FIRST
    vbat_factor = 1.0 + node_cfg["alpha_sag"] * ((V_NOM - raw_row["vbat_mv"]) / V_NOM)
    linear_strain_counts = raw_row["raw_strain"] / vbat_factor
    linear_temp_c = (raw_row["raw_temp_dc"] * 0.1) / vbat_factor

    # Step 4: Undo Thermal Drift SECOND
    delta_t = linear_temp_c - T_REF
    calibrated_strain_ue = linear_strain_counts - (node_cfg["k_T_strain"] * delta_t)
    
    return calibrated_strain_ue
```

Continuous integration test `T7` enforces this execution invariant. In `T7`, synthetic raw frames degraded by simultaneous $500\text{ mV}$ battery drops and $35^\circ\text{C}$ temperature swings are passed into C7; the calibrated output must recover the ground-truth physical displacement within $\pm 1.2\ \mu\varepsilon$. Inverting the function calls fails `T7` and halts the deployment pipeline.

---

### Question: How does Step 5 convert raw ADC integer counts into standardized SI engineering units?
**Answer:** Step 5 maps dimensionless 12-bit, 16-bit, and 24-bit integer registers into standardized SI physical units using factory calibration coefficients specified in `nodes.json`. Each physical sensor channel uses a dedicated transfer function:

| Channel | Transducer Hardware | Raw Representation | Transfer Function to SI Engineering Units | SI Output Unit |
| :--- | :--- | :--- | :--- | :--- |
| **Biaxial Tilt ($X, Y$)** | MEMS Dual-Axis Inclinometer | `int16` $(-32768 \dots 32767)$ | $\theta = \arcsin\left(\frac{\text{raw} \cdot g_{\text{scale}}}{g}\right) - \theta_{\text{offset}}$ | Radians ($\text{rad}$) / $\text{mm/m}$ |
| **Tensile Strain ($\varepsilon$)** | Vibrating Wire / Foil Gauge | `int24` ADC raw counts | $\varepsilon = \left(\frac{\text{raw} - \text{raw}_{\text{zero}}}{\text{GF} \cdot V_{\text{bridge}}}\right) \times 10^6$ | Microstrain ($\mu\varepsilon$) |
| **Displacement ($\Delta L$)** | Multi-Point Extensometer | `int16` LVDT counts | $\Delta L = \text{raw} \cdot S_{\text{LVDT}} \cdot L_{\text{rod}}$ | Millimeters ($\text{mm}$) |
| **Vibration (PPV)** | 3-Axis Geophone / Accelerometer | `int16` Peak amplitude | $\text{PPV} = \frac{\text{raw} \cdot V_{\text{LSB}}}{\text{Sensitivity}_{\text{geo}}}$ | Millimeters/second ($\text{mm/s}$) |

---

### Question: How does Common-Mode Rejection (Step 6) eliminate environmental false alarms from regional ground swelling?
**Answer:** Diurnal atmospheric pressure fluctuations, seasonal monsoon soil expansion, and regional ambient thermal cycles induce superficial ground displacements across an entire mining lease. Without compensation, a naive monitoring system would misinterpret regional soil swelling as active subsidence trough development.

To eliminate this, AEGIS establishes two Bedrock Anchor stations ($A_1, A_2$) anchored into competent, undisturbed strata located outside the active subsidence basin (beyond the Knothe angle of draw, distance $d > H / \tan\beta$). Step 6 computes the instantaneous common-mode regional disturbance vector as the median baseline reading of the anchor stations:

$$\Delta_{\text{CMR}}(t) = \text{median}\left( y_{\text{anchor}, 1}(t), \, y_{\text{anchor}, 2}(t) \right)$$

The median is chosen rather than the arithmetic mean to maintain robustness against a single anchor station suffering mechanical damage or localized disturbances. This baseline offset is then subtracted from all active monitoring stations across the mining panel:

$$y_{\text{station}, i}^{\text{CMR}}(t) = y_{\text{station}, i}(t) - \Delta_{\text{CMR}}(t)$$

By subtracting the regional baseline, non-mining diurnal expansion and seasonal moisture swelling are canceled out, leaving only genuine subsidence and mining-induced deformation.

---

### Question: How is dynamic channel uncertainty ($\sigma_{\text{final}}$, Step 7) calculated, and why must anchor noise inflation follow CMR?
**Answer:** Sensor uncertainty is not static. Operational conditions—such as degraded radio link margins or missed packets—degrade measurement confidence. C7 calculates a dynamic standard deviation ($\sigma_{\text{final}}$) for every sensor channel at every epoch using a multi-factor scaling model:

$$\sigma_{\text{final}} = \sqrt{\sigma_{\text{base}}^2 + \sigma_{\text{anchor}}^2} \cdot f_{\text{link}} \cdot f_{\text{gap}} \cdot f_{\text{flag}}$$

1. **Hardware Noise Baseline ($\sigma_{\text{base}}$):** The intrinsic thermal and quantization noise floor of the transducer ($1.2\ \mu\varepsilon$ for strain, $8\ \mu\text{rad}$ for tilt).
2. **Anchor Injected Noise ($\sigma_{\text{anchor}}$):** Because Step 6 subtracts the anchor baseline, error propagation dictates that the variance of the difference equals the sum of the variances:
   $$\sigma_{\text{CMR}}^2 = \sigma_{\text{station}}^2 + \sigma_{\text{anchor}}^2$$
   Applying uncertainty inflation **after** common-mode rejection accurately reflects this injected variance.
3. **RF Link Margin Factor ($f_{\text{link}}$):** If the uplink LoRa Signal-to-Noise Ratio (SNR) drops below $-10\text{ dB}$, packet re-try probability rises, scaling uncertainty up to $1.5\times$:
   $$f_{\text{link}} = 1.0 + 0.5 \cdot \max\left(0.0, \, \frac{-10 - \text{SNR}}{10}\right)$$
4. **Packet Age Gap Factor ($f_{\text{gap}}$):** When packets are dropped over previous epochs, uncertainty inflates exponentially with the time elapsed since the last valid reading ($\Delta t_{\text{elapsed}}$), capped at a maximum of $10.0\times$ (enforced by test `T45`):
   $$f_{\text{gap}} = \min\left(10.0, \, \exp\left(\frac{\Delta t_{\text{elapsed}}}{2 \cdot \tau_{\text{sampling}}}\right)\right)$$
5. **Diagnostic Degradation Factor ($f_{\text{flag}}$):** If minor hardware warning flags are asserted (e.g., low battery or solar charging active), $f_{\text{flag}}$ inflates uncertainty by $1.25\times$.

---

### Question: What is the structure of the Step 8 Validity Mask, and how does it protect downstream modules?
**Answer:** In Step 8, C7 constructs a binary validity tensor of dimension $N_{\text{nodes}} \times 4$, where $N_{\text{nodes}}$ represents the active station count across the panel grid and the 4 columns correspond to the physical channels: `[Tilt_X, Tilt_Y, Strain, Extensometer]`. 

A channel is assigned `True` (valid) if and only if:
1. The raw packet was received within the epoch window (`NaN == False`).
2. Transducer diagnostic flags confirm operational health (`selftest_ok == 1`, `gauge_open == 0`, `accel_sat == 0`).
3. The calculated dynamic uncertainty does not exceed statutory safety limits ($\sigma_{\text{final}} < 5 \cdot \sigma_{\text{base}}$).

This validity mask directly gates downstream processing:
* **In C8 Safety Engine:** Any station channel marked `False` is disqualified from participating in the 5-station Byzantine quorum voting pool, preventing corrupted or dead sensors from biasing spatial consensus.
* **In C9 PINN Digital Twin:** The mask is passed to the neural network loss function, zeroing out data-loss gradients at invalid coordinates and preventing corrupted inputs from distorting physics-informed terrain reconstruction.
