# C7 Corrector: The 8-Step Calibration & Cleaning Pipeline

**Module 06 — Backend Pipeline**  
**Cross-References:** [`data-architecture.md`](data-architecture.md) · [`c8-alarm-engine.md`](c8-alarm-engine.md) · [Module 04 Corruption Chain](../04-physics-engine/corruption-chain.md)

---

## 1. The 8-Step Sequential Pipeline

The C7 pipeline is the sole entity responsible for transforming degraded, raw bitstream observations into clean, calibrated physical engineering units. 

To prevent cross-contamination of error modes, the eight steps must execute in a **strictly immutable sequence**:

```
[Raw 23-byte Unpacked Telemetry]
               │
               ▼  Step 1: Assemble Epoch Bucket
[Align time bucket; mark missing packets as NaN (NEVER ZERO-FILL)]
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
[Subtract undisturbed Bedrock Anchor baseline from active nodes]
               │
               ▼  Step 7: Compute Dynamic Channel Uncertainty (σ)
[Scale σ_base by link margin, packet gaps, and diagnostic flags]
               │
               ▼  Step 8: Construct Validity Mask
[Emit N × 4 boolean mask gating downstream algorithms]
               │
               ▼
[Cleaned Calibrated State → Handed to C8 Detector & C9 PINN]
```

---

## 2. Code Order Rule: Sag First, Then Thermal (Test T7)

A common bug in geotechnical calibration pipelines is inverting the order of battery sag and thermal compensation:
* **The Physical Mechanism:** Battery voltage sag alters the reference voltage ($V_{\text{ref}}$) of the analogue-to-digital converter, scaling the raw digital counts of all connected transducers uniformly (including the temperature diode).
* **The Consequence of Inversion:** If thermal compensation is computed first using uncorrected raw temperature counts, the thermal subtraction uses a distorted temperature estimate.
* **The Rule:** **Sag correction must execute first**, restoring true reference linearity, followed immediately by thermal drift subtraction.

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

Test `T7` asserts that applying this sequence to synthetic degraded data recovers true displacement within the baseline Gaussian sensor noise band. If `T7` fails, the build halts.

---

## 3. Common-Mode Rejection (CMR: Step 6)

Diurnal atmospheric pressure swings, seasonal moisture expansion of topsoil, and regional temperature variations affect all surface stations simultaneously.
* Bedrock Anchor nodes ($A_1, A_2$) are anchored into undisturbed ground outside the subsidence basin. They cannot move from mining activity.
* Step 6 computes the median baseline movement recorded by the Anchor nodes:
  $$\Delta_{\text{CMR}}(t) = \text{median}\left( y_{\text{anchor}, 1}(t), \, y_{\text{anchor}, 2}(t) \right)$$
* This common-mode signal is subtracted from all active Scout stations:
  $$y_{\text{scout}, i}^{\text{CMR}}(t) = y_{\text{scout}, i}(t) - \Delta_{\text{CMR}}(t)$$
This eliminates regional non-mining ground swelling, preventing seasonal false alarms.

---

## 4. Dynamic Uncertainty ($\sigma$) Formulation (Step 7)

Measurement uncertainty is not static. C7 calculates a dynamic standard deviation ($\sigma_{\text{final}}$) for every sensor channel at every epoch:

$$\sigma_{\text{final}} = \sigma_{\text{base}} \cdot f_{\text{link}} \cdot f_{\text{gap}} \cdot f_{\text{flag}}$$

1. **Base Hardware Noise ($\sigma_{\text{base}}$):** The baseline thermal noise of the sensor ($1.2\ \mu\varepsilon$ for strain, $8\ \mu\text{rad}$ for tilt).
2. **RF Link Margin Factor ($f_{\text{link}}$):** If uplink SNR drops below $-10\text{ dB}$, packet re-try probability rises, scaling uncertainty up to $1.5\times$.
3. **Age Gap Factor ($f_{\text{gap}}$):** If packets were dropped over previous epochs, uncertainty inflates according to time elapsed, capped at $10.0\times$ (Test `T45`).
4. **CMR Inflation Order:** Uncertainty inflation is applied **after** common-mode rejection, correctly accounting for the fact that subtracting anchor signals injects the anchor's own measurement noise into the result.
