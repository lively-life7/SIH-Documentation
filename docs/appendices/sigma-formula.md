# Dynamic Measurement Uncertainty ($\sigma$) Derivations

**Appendices**  
**Cross-References:** [`constants-reference.md`](constants-reference.md) · [Module 06 C7 Corrector](../06-backend-pipeline/c7-corrector.md) · [Module 08 Test T45](../08-verification/test-register.md)

---

### Question: Why does AEGIS derive a dynamic measurement uncertainty ($\sigma_{\text{final}}$) for every channel at every epoch instead of using static sensor tolerances?

**Answer:** Static confidence intervals fail in outdoor wireless IoT environments. A geotechnical reading captured with a fresh battery over a direct line-of-sight RF link ($+10\text{ dB SNR}$) has vastly higher physical credibility than a reading arriving after a 12-hour store-and-forward transmission outage over a fading link ($-12\text{ dB SNR}$) with a decaying battery voltage.

If measurement uncertainty were treated as a static constant, downstream algorithms would either overreact to degraded readings (causing false alarms in the C8 Byzantine Quorum Detector) or assign excessive weight to stale observations during neural surface reconstruction (destabilizing the C9 PINN loss gradients).

To ensure that every observation is weighted by its true instantaneous statistical credibility, the C7 pipeline calculates an explicit **Dynamic Standard Deviation ($\sigma_{\text{final}}$)** for every transducer channel at every 60-second epoch:

```
[Base Hardware Thermal Noise: σ_base]
               │
               ▼  Stage 1: Common-Mode Rejection (CMR) Noise Addition
[Baseline Anchor Variance Injection: σ_cmr = √(σ_base² + σ_anchor²/2)]
               │
               ▼  Stage 2: Multiplicative Channel Penalties
[Scale by f_link (SNR Margin) × f_gap (Packet Age) × f_flags (Health)]
               │
               ▼
[Dynamic Channel Uncertainty: σ_final]
```

---

### Question: What is the exact mathematical derivation of $\sigma_{\text{final}}$, and how does error propagation propagate through common-mode rejection?

**Answer:** The dynamic uncertainty is derived across three rigorous mathematical stages:

#### Stage 1: Base Transducer Noise Floor ($\sigma_{\text{base}}$)
The empirical Gaussian white-noise floor of the physical transducers operating at the reference calibration temperature ($T_{\text{REF}} = 25.0^\circ\text{C}$):
* **Horizontal Strain ($\varepsilon$):** $\sigma_{\text{base}} = 1.2\ \mu\varepsilon$
* **Ground Tilt ($T_x, T_y$):** $\sigma_{\text{base}} = 8.0\ \mu\text{rad}$
* **Wire Extensometer ($Ext$):** $\sigma_{\text{base}} = 15.0\ \mu\text{m}$

#### Stage 2: Common-Mode Rejection Variance Injection ($\sigma_{\text{cmr}}$)
To eliminate regional topsoil swelling and seasonal barometric heaving, the C7 pipeline subtracts the average baseline displacement recorded by two Bedrock Reference Anchors ($A_1, A_2$) from the Scout reading:
$$S_{\text{corrected}} = S_{\text{scout}} - \frac{S_{A1} + S_{A2}}{2}$$

Under standard Gauss-Markov error propagation, subtracting an independent random variable adds its variance:
$$\sigma_{\text{cmr}}^2 = \sigma_{\text{base}}^2 + \operatorname{Var}\left(\frac{S_{A1} + S_{A2}}{2}\right) = \sigma_{\text{base}}^2 + \frac{\sigma_{A1}^2 + \sigma_{A2}^2}{4}$$

Given that reference anchors use identical instrumentation ($\sigma_{A1} = \sigma_{A2} = \sigma_{\text{base}}$):
$$\sigma_{\text{cmr}} = \sqrt{\sigma_{\text{base}}^2 + \frac{2\sigma_{\text{base}}^2}{4}} = \sqrt{\sigma_{\text{base}}^2 + \frac{\sigma_{\text{base}}^2}{2}} = \sigma_{\text{base}} \cdot \sqrt{1.5} \approx \mathbf{1.225 \cdot \sigma_{\text{base}}}$$

Common-mode rejection introduces a mathematically unavoidable $\approx 22.5\%$ increase in Gaussian variance, which is compensated for in the C8 alarm decision thresholds.

#### Stage 3: Multiplicative Penalty Formulation
The post-CMR uncertainty is scaled by three operational penalty multipliers:
$$\sigma_{\text{final}} = \sigma_{\text{cmr}} \cdot f_{\text{link}} \cdot f_{\text{gap}} \cdot f_{\text{flags}}$$

---

### Question: How are the RF link ($f_{\text{link}}$), temporal gap ($f_{\text{gap}}$), and hardware health ($f_{\text{flags}}$) penalty multipliers formulated?

**Answer:** The operational multipliers evaluate telemetry reception quality, data staleness, and on-node hardware diagnostics:

#### 1. RF Link Margin Multiplier ($f_{\text{link}}$)
Evaluates signal quality recorded by the Master Gateway receiver:
$$f_{\text{link}} = \begin{cases} 
1.0 & \text{if } \text{SNR} \ge 0\text{ dB (Strong, robust RF link)} \\ 
1.0 + 0.05 \cdot |\text{SNR}| & \text{if } -10\text{ dB} \le \text{SNR} < 0\text{ dB (Mild fading channel)} \\ 
1.5 & \text{if } \text{SNR} < -10\text{ dB (Fading edge; packet near sensitivity limit)} 
\end{cases}$$

#### 2. Temporal Age Gap Multiplier ($f_{\text{gap}}$ — Verified by Test T45)
When a Scout node's transmissions are temporarily severed by rockfall, machinery obstruction, or power dips, the elapsed time since the last valid sample ($\Delta t_{\text{gap}}$ in seconds) increases uncertainty regarding intervening ground movement:
$$f_{\text{gap}} = \min\left(10.0, \, 1.0 + 0.1 \cdot \left(\frac{\Delta t_{\text{gap}}}{3600\text{ seconds}}\right)\right)$$

**The Statutory 10.0 Ceiling (Test T45):**
Uncapped exponential or linear gap functions allow $\sigma$ to inflate by over $1,000\times$ following a 4-day communication partition. When ingested into downstream algorithms, such astronomical variances cause severe floating-point division-by-zero errors or numerical singularity ($NaN$) in inverse covariance matrix computations. Capping $f_{\text{gap}}$ at **10.0** mathematically suppresses stale data during spatial quorum consensus while guaranteeing numerical stability.

#### 3. Hardware Diagnostic Flag Multiplier ($f_{\text{flags}}$)
Evaluates byte 17 of the 23-byte wire frame for on-node hardware warnings:
$$f_{\text{flags}} = 1.0 + 0.5 \cdot (\text{ext\_overflow}) + 0.3 \cdot (\text{vbat\_critical})$$

* If the mechanical extensometer has reached the end of its stroke (`ext_overflow = 1`), uncertainty increases by $+50\%$.
* If battery voltage drops below the critical threshold ($V_{\text{bat}} < 3.1\text{ V}$, `vbat_critical = 1`), uncertainty increases by $+30\%$.

This dynamic inflation ensures that a dying or mechanically jammed sensor automatically receives minimal voting weight in the C8 Byzantine Quorum detector, preventing false alarms without human intervention.
