# The 6-Stage Sensor Corruption Chain

**Module 04 — Physics Engine**  
**Cross-References:** [`knothe-model.md`](knothe-model.md) · [`ground-truth-generation.md`](ground-truth-generation.md) · [Module 06 C7 Corrector](../06-backend-pipeline/c7-corrector.md)

---

## 1. Why Realistic Sensor Corruption Is Mandatory

A common failure mode in academic IoT simulations is feeding pristine, noise-free mathematical equations into AI models and threshold detectors. In real field deployments, raw sensor readings are heavily distorted by ambient thermodynamics, electronic aging, and wireless packet dropouts.

To ensure that backend calibration algorithms (`C7`) and detection engines (`C8`) are field-tested before physical installation, AEGIS passes pristine Knothe ground truth through an authoritative **6-Stage Physical Degradation Chain**.

```
[Pristine Ground Truth S, T, ε]
               │
               ▼  Stage 1: Thermodynamic Expansion & Piezoresistive Drift
[Thermal Drift Injection: k_T · (T_die - 25°C)]
               │
               ▼  Stage 2: ADC Quantization & Integer Rounding
[LSB Truncation: tilt 2 µrad, strain 1 µε, ext 10 µm]
               │
               ▼  Stage 3: Semiconductor Bias Random Walk
[Ornstein-Uhlenbeck Stochastic Process: τ = 6 hours]
               │
               ▼  Stage 4: Power Rail Variation
[Battery Voltage Sag Scaling: V_bat vs V_nom]
               │
               ▼  Stage 5: Non-Deterministic RF Fading
[Packet Dropouts: Missing epochs preserved as missing (NEVER ZEROED)]
               │
               ▼  Stage 6: Age-Based Uncertainty Inflation
[Temporal Staleness Penalty: f_gap scaling]
               │
               ▼
[Degraded Field Telemetry → nodes.csv]
```

---

## 2. Stage-by-Stage Mathematical Formulation

### Stage 1: Thermal Drift Injection
Silicon MEMS accelerometers, foil strain bridges, and invar extensometer wires expand and shift electronic offsets with temperature:

$$y_{\text{thermal}}(t) = y_{\text{true}}(t) + k_T \cdot (T_{\text{die}}(t) - T_{\text{ref}})$$

Where $T_{\text{ref}} = 25.0^\circ\text{C}$ and:
* Tilt: $k_T = 250\ \mu\text{rad/}^\circ\text{C}$
* Strain: $k_T = 5\ \mu\varepsilon/^\circ\text{C}$
* Extensometer: $k_T = 12\ \mu\text{m/}^\circ\text{C}$

### Stage 2: Quantization Noise (LSB Granularity)
Analogue voltages are digitized by integer ADCs, imposing finite quantization noise:
$$y_{\text{quant}} = \text{round}\left(\frac{y_{\text{thermal}}}{\text{LSB}}\right) \cdot \text{LSB}$$
Where LSB values match the 23-byte wire format: $2\ \mu\text{rad}$ (tilt), $1\ \mu\varepsilon$ (strain), $10\ \mu\text{m}$ (extensometer).

### Stage 3: Low-Frequency Bias Drift (Ornstein-Uhlenbeck Walk)
Semiconductor components undergo slow baseline drift governed by a mean-reverting Ornstein-Uhlenbeck stochastic process with correlation time $\tau = 21,600\text{ seconds}$ (6 hours):
$$db_t = -\frac{1}{\tau} b_t\, dt + \sigma_b \sqrt{\frac{2}{\tau}}\, dW_t$$
* Tilt drift: $\sigma_b = 3\ \mu\text{rad}$
* Strain drift: $\sigma_b = 0.5\ \mu\varepsilon$
* Extensometer drift: $\sigma_b = 5\ \mu\text{m}$

### Stage 4: Battery Supply Voltage Sag
As battery cell voltage declines from $4.2\text{V}$ down to $3.2\text{V}$, small reference voltage shifts scale the measured analogue strain signals:
$$y_{\text{sag}} = y_{\text{quant}} \cdot \left[ 1 + \alpha_{\text{sag}} \cdot \left( \frac{V_{\text{nom}} - V_{\text{bat}}}{V_{\text{nom}}} \right) \right]$$
Where $V_{\text{nom}} = 3.70\text{ V}$.

### Stage 5: Non-Deterministic Packet Loss
Wireless packets are subjected to log-normal RF shadowing dropouts.
* **The Inviolable Ingestion Rule:** If a frame is dropped, the telemetry cell is recorded as **missing / NaN**.
* **Zero-Filling Prohibition:** A missing telemetry reading is **never zero-filled**. Zero-filling generates artificial step-function spikes that falsely trip rate-of-change alarms and destroy PINN gradient convergence (verified by test `T28`).

### Stage 6: Age-Based Uncertainty Inflation
When data arrives after an outage, its dynamic uncertainty ($\sigma$) inflates based on the time elapsed since the previous confirmed epoch:
$$f_{\text{gap}} = \min\left(10.0, \, 1.0 + 0.1 \cdot \frac{\Delta t_{\text{gap}}}{3600\text{ s}}\right)$$
Capping $f_{\text{gap}}$ at 10 prevents numerical explosion during extended backhaul maintenance (verified by test `T45`).
