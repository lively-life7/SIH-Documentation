# The 6-Stage Sensor Corruption Chain

**Module 04 — Physics Engine**  
**Cross-References:** [`knothe-model.md`](knothe-model.md) · [`derived-quantities.md`](derived-quantities.md) · [`ground-truth-generation.md`](ground-truth-generation.md) · [Module 06 C7 Corrector](../06-backend-pipeline/c7-corrector.md) · [Module 08 Test Register](../08-verification/test-register.md)

---

## 1. Physical Motivation & Architectural Rationale

### Question: Why is an explicit 6-stage sensor corruption chain required instead of validating backend algorithms on pristine mathematical ground truth?

**Answer:** Evaluating geotechnical safety algorithms and neural networks on pristine, analytical ground truth represents an academic failure mode that guarantees field failure. In an active open-cast or underground coal mine, surface environmental conditions severely distort raw physical signals before they reach backend servers. Ambient diurnal temperatures fluctuate by over $25^\circ\text{C}$, electrochemical battery voltages sag during heavy load or depletion, semiconductor analog-to-digital converters (ADCs) introduce discrete truncation errors, and high-frequency RF packet dropouts occur due to heavy mining machinery obstruction and monsoon rain attenuation.

If calibration algorithms (such as the C7 Corrector) and alarm triggers (C8 Deterministic Detector) are tested solely against idealized Knothe curves, their error thresholds will fail when subjected to real-world industrial noise. The AEGIS 6-Stage Physical Degradation Chain systematically corrupts pristine continuous ground truth into realistic, degraded raw telemetry, guaranteeing that all backend filtering, Kalman estimators, and physics-informed neural networks (C9 PINN) are rigorously stress-tested against non-ideal field instrumentation.

```
[Pristine Knothe Ground Truth: S(x, y, t), T_x, T_y, ε_x, Ext]
                           │
                           ▼  Stage 1: Piezoresistive & MEMS Thermal Expansion
[Thermal Drift Injection: y_thermal = y_true + k_T · (T_die - 25°C)]
                           │
                           ▼  Stage 2: ADC Quantization & Integer Serialization
[LSB Truncation: round(y / LSB) · LSB]
                           │
                           ▼  Stage 3: Semiconductor Bias Random Walk
[Ornstein-Uhlenbeck Stochastic Process: τ = 6 hours]
                           │
                           ▼  Stage 4: Electrochemical Battery Sag
[Voltage Sag Multiplier: 1 + α_sag · ((V_nom - V_bat) / V_nom)]
                           │
                           ▼  Stage 5: Non-Deterministic RF Packet Erasure
[Log-Normal Fading: Preserved as NaN / Missing — NEVER ZERO-FILLED]
                           │
                           ▼  Stage 6: Dynamic Covariance Staleness Penalty
[Age-Based Inflation: f_gap = min(10.0, 1.0 + 0.1 · Δt_gap / 3600s)]
                           │
                           ▼
[Degraded Field Telemetry → Streamed to nodes.csv / C7 Corrector]
```

---

## 2. Mathematical Formulations of the Degradation Stages

### Question: How is thermal expansion and piezoresistive temperature drift modeled across different physical sensor transducers (Stage 1)?

**Answer:** Silicon MEMS accelerometers (inclinometers), foil strain gauge bridges, and vibrating-wire/invar extensometer rods expand and shift electronic bridge balance as ambient temperature changes. Stage 1 injects a deterministic temperature-dependent offset governed by the sensor's internal die temperature $T_{\text{die}}(t)$ relative to a laboratory reference datum $T_{\text{ref}} = 25.0^\circ\text{C}$:

$$y_{\text{thermal}}(t) = y_{\text{true}}(t) + k_T \cdot \left[ T_{\text{die}}(t) - T_{\text{ref}} \right]$$

Where the empirical thermal transfer coefficients $k_T$ are physically grounded in transducer specifications:
* **Biaxial Tilt (Inclinometer):** $k_T = 250.0\ \mu\text{rad}/^\circ\text{C}$ (originating from differential silicon MEMS thermal expansion).
* **Horizontal Strain (Foil Bridge):** $k_T = 5.0\ \mu\varepsilon/^\circ\text{C}$ (apparent strain caused by thermal mismatch between the constantan foil and sandstone mounting pad).
* **Displacement Extensometer:** $k_T = 12.0\ \mu\text{m}/^\circ\text{C}$ (linear thermal expansion coefficient of the mechanical invar rod assembly over a 10-meter baseline span).

The synthetic generator simulates diurnal thermal cycles using a sinusoidal model superimposed with stochastic micro-fluctuations:
$$T_{\text{die}}(t) = T_{\text{ambient\_mean}} + \Delta T_{\text{diurnal}} \sin\left(\frac{2\pi t}{86400} - \phi\right) + \mathcal{N}(0, \sigma_T^2)$$

### Question: How are finite ADC resolution and telemetry wire packing simulated (Stage 2)?

**Answer:** Field sensor microcontrollers do not transmit 64-bit IEEE floating-point values over LoRa wireless links. Instead, analogue voltages are digitized by successive-approximation register (SAR) ADCs and packed into signed 16-bit integers (`int16`). Stage 2 applies non-linear quantization truncation to reflect the true Least Significant Bit (LSB) granularity of the hardware:

$$y_{\text{quant}} = \text{round}\left( \frac{y_{\text{thermal}}}{\text{LSB}} \right) \cdot \text{LSB}$$

The LSB resolutions correspond exactly to the AEGIS 23-byte binary wire serialization protocol:
* **Tilt ($T_x, T_y$):** $\text{LSB} = 2.0\ \mu\text{rad}$ per count (dynamic span $\pm 65,534\ \mu\text{rad} \approx \pm 3.75^\circ$).
* **Strain ($\varepsilon_x, \varepsilon_y$):** $\text{LSB} = 1.0\ \mu\varepsilon$ per count (dynamic span $\pm 32,767\ \mu\varepsilon$).
* **Extensometer Displacement:** $\text{LSB} = 10.0\ \mu\text{m}$ per count (dynamic span $\pm 327.6\text{ mm}$).

Any continuous movement occurring below these LSB thresholds is trapped as quantization noise, compelling backend filters to maintain sub-LSB tracking via temporal state estimation.

### Question: How is long-term electronic baseline drift modeled, and why is an Ornstein-Uhlenbeck process chosen over an unbounded random walk (Stage 3)?

**Answer:** Analogue front-end amplifiers, operational amplifier input offset currents, and piezoresistive bridge bonds experience low-frequency $1/f$ flicker noise and physical relaxation over weeks of field exposure. If modeled as a pure Gaussian random walk (Brownian motion), the simulated sensor bias would integrate infinitely ($b_t \to \pm \infty$), which is physically impossible for passive silicon components.

AEGIS implements a mean-reverting **Ornstein-Uhlenbeck (OU) stochastic differential equation** with an empirical relaxation time constant $\tau = 21,600\text{ seconds}$ (6.0 hours):

$$db_t = -\frac{1}{\tau} b_t\, dt + \sigma_b \sqrt{\frac{2}{\tau}}\, dW_t$$

Where $dW_t$ is a standard Wiener process increment ($dW_t \sim \mathcal{N}(0, dt)$), and $\sigma_b$ represents the stationary asymptotic standard deviation of the physical sensor bias:
* **Tilt Inclinometer Drift:** $\sigma_b = 3.0\ \mu\text{rad}$
* **Horizontal Strain Drift:** $\sigma_b = 0.5\ \mu\varepsilon$
* **Extensometer Displacement Drift:** $\sigma_b = 5.0\ \mu\text{m}$

The mean-reverting term $-\frac{1}{\tau} b_t\, dt$ pulls the baseline back toward zero, perfectly reproducing the bounded physical drift characteristics of precision semiconductor instrumentation under continuous thermal cycling.

### Question: How does electrochemical battery discharge distort analog measurement bridges (Stage 4)?

**Answer:** Solar-recharged lithium iron phosphate ($\text{LiFePO}_4$) and lithium-ion cells exhibit battery voltage sag from $4.20\text{V}$ (fully charged peak) down to $3.20\text{V}$ (depleted plateau under low insolation or load). While the digital microcontroller runs behind a low-dropout (LDO) regulator, micro-fluctuations in the analog reference voltage rail ($V_{\text{ref}}$) scale the excitation bridge voltage of strain sensors.

Stage 4 simulates this ratiometric scaling using the instantaneous battery voltage $V_{\text{bat}}$ relative to nominal voltage $V_{\text{nom}} = 3.70\text{V}$:

$$y_{\text{sag}} = y_{\text{quant}} \cdot \left[ 1 + \alpha_{\text{sag}} \cdot \left( \frac{V_{\text{nom}} - V_{\text{bat}}}{V_{\text{nom}}} \right) \right]$$

Where:
* $V_{\text{nom}} = 3.70\text{ V}$ (nominal battery rail).
* $\alpha_{\text{sag}} = 0.015$ (empirical gain sensitivity coefficient).

When battery voltage drops to $3.20\text{V}$, an uncalibrated strain signal experiences an artificial negative scaling of $\approx 0.2\%$, which the C7 Corrector must actively compensate for using battery telemetry prior to downstream processing.

---

## 3. Telemetry Loss & Uncertainty Propagation Mechanics

### Question: Why is zero-filling of missing wireless telemetry strictly prohibited, and what failure does it cause in downstream AI and alarm systems (Stage 5, Test T28)?

**Answer:** Wireless packets transmitted across rugged mining terrain experience non-deterministic dropouts caused by log-normal multipath fading, deep fresnel zone obstruction by earthmoving dumpers, and atmospheric absorption. When a packet is lost, naive data engineering pipelines often replace missing telemetry rows with zero values (`0.0`).

AEGIS enforces the **Inviolable Ingestion Rule (verified by Test T28)**:
1. Missing telemetry readings are preserved strictly as **`NaN` / Missing Values**.
2. **Zero-filling is strictly prohibited across all layers of the architecture.**

```
PHYSICAL REALITY: Continuous ground movement at 5,000 µε
                  ┌──────────────────────────────────────────────┐
Packet Delivered: │ t = 100: ε = 5000 µε                         │
Packet Dropped:   │ t = 101: ε = NaN (Sensor reading missing)    │
Packet Delivered: │ t = 102: ε = 5020 µε                         │
                  └──────────────────────────────────────────────┘

CATASTROPHIC FAILURE OF ZERO-FILLING:
If t = 101 is zero-filled (ε = 0 µε):
- Artificial Step 1: dε/dt = (0 - 5000) / 60s = -83.3 µε/s
- Artificial Step 2: dε/dt = (5020 - 0) / 60s = +83.7 µε/s
CONSEQUENCE: Massive artificial rate-of-change spikes trigger false Class-A sirens
             and inject severe gradient singularities into the PINN optimizer.
```

By preserving missing values as `NaN`, the C7 Kalman filter executes a pure time update (state propagation without measurement update), maintaining mathematical continuity without tripping spurious rate-of-change alarms.

### Question: How does temporal staleness inflate observation covariance, and why must the staleness multiplier $f_{\text{gap}}$ be capped (Stage 6, Test T45)?

**Answer:** When communication link outages occur, the uncertainty associated with a sensor station's state estimate grows monotonically with elapsed time. When the node finally reconnects and retransmits telemetry, the measurement update must not be treated with the same statistical confidence as continuous, synchronized streaming data.

Stage 6 computes a dynamic covariance inflation multiplier $f_{\text{gap}}$ as a function of the time gap $\Delta t_{\text{gap}}$ since the last confirmed measurement epoch:

$$f_{\text{gap}} = \min\left(10.0, \, 1.0 + 0.1 \cdot \frac{\Delta t_{\text{gap}}}{3600\text{ seconds}}\right)$$

The observation variance $\sigma^2$ fed into downstream Kalman filtering and PINN loss weighting is dynamically scaled:
$$\sigma_{\text{adjusted}}^2 = f_{\text{gap}} \cdot \sigma_{\text{nominal}}^2$$

**The Mathematical Necessity of the Cap at 10.0 (Test T45):**  
If a remote node is isolated for several days or weeks during scheduled longwall maintenance, an uncapped linear inflation model would allow $f_{\text{gap}} \to \infty$. This would cause the observation covariance matrix $\mathbf{R}$ in the Kalman filter to become ill-conditioned, leading to numerical overflow, division-by-zero during Kalman gain computation ($\mathbf{K} = \mathbf{P} \mathbf{H}^T (\mathbf{H}\mathbf{P}\mathbf{H}^T + \mathbf{R})^{-1}$), and fatal system crashes. Capping $f_{\text{gap}}$ at $10.0$ bounds the condition number of the covariance matrix while accurately de-weighting stale telemetry.

---

## 4. Pipeline Order & Inversion Verification

### Question: Why must the corruption chain execute in a strictly enforced physical sequence, and how is this order validated in CI (Test T6 & Test T7)?

**Answer:** The degradation stages represent physical causal layers that occur in nature. They cannot commute mathematically:
1. Thermal expansion acts on the physical sensor crystal *before* digitization.
2. Quantization occurs *at* the ADC boundary, truncating the thermally shifted voltage.
3. Semiconductor drift acts on the front-end amplifier stage.
4. Battery sag scales the operational voltage applied to the bridge.
5. Packet loss acts on the serialized bitstream during airwave transit.
6. Uncertainty inflation evaluates the resulting time arrival gap at the gateway.

If Stage 2 (ADC quantization) were applied before Stage 1 (thermal expansion), small thermal expansions would be artificially quantized before reaching the bridge, violating semiconductor physics.

**CI Test Verification:**
* **Test T6 (Order Hash Assertion):** Asserts that synthetic generation scripts execute the corruption stages in the exact sequence `1 → 2 → 3 → 4 → 5 → 6`. Any alteration of execution order produces a cryptographic mismatch and immediately fails the build.
* **Test T7 (Inversion Residual Limit):** Asserts that the backend C7 Corrector, which runs the inverse transformations (reversing voltage sag, subtracting modeled thermal drift, and filtering bias drift), recovers ground truth subsidence within the white-noise measurement band ($\le 1.5\ \text{mm}$ residual). If Test T7 fails, deployment is halted immediately.
