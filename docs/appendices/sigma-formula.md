# Dynamic Measurement Uncertainty ($\sigma$) Derivations

**Appendices**  
**Cross-References:** [`constants-reference.md`](constants-reference.md) · [Module 06 C7 Corrector](../06-backend-pipeline/c7-corrector.md) · [Module 08 Test T45](../08-verification/test-register.md)

---

## 1. The Need for Dynamic Uncertainty

Static confidence intervals fail in outdoor wireless IoT networks. A measurement taken with a fresh battery over a direct $+10\text{ dB}$ SNR wireless link is far more reliable than a reading arriving after a 12-hour transmission outage over a fading $-12\text{ dB}$ RF link.

To ensure that the C8 Byzantine Quorum Detector and C9 PINN model weight observations correctly, the C7 pipeline calculates an explicit **Dynamic Standard Deviation ($\sigma_{\text{final}}$)** for every channel at every epoch.

---

## 2. The 3-Stage Mathematical Derivation

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

### Stage 1: Base Sensor Noise ($\sigma_{\text{base}}$)
The empirical white-noise floor of the physical transducers operating at $25.0^\circ\text{C}$:
* Horizontal Strain ($\varepsilon$): $\sigma_{\text{base}} = 1.2\ \mu\varepsilon$
* Ground Tilt ($T_x, T_y$): $\sigma_{\text{base}} = 8.0\ \mu\text{rad}$
* Wire Extensometer: $\sigma_{\text{base}} = 15.0\ \mu\text{m}$

### Stage 2: Common-Mode Rejection Variance Injection ($\sigma_{\text{cmr}}$)
In Step 6 of the C7 pipeline, the average baseline of two Bedrock Anchors ($A_1, A_2$) is subtracted from the Scout reading to eliminate regional topsoil swelling. 
* By the rules of linear variance propagation, subtracting an independent random variable adds its variance:
  $$\sigma_{\text{cmr}} = \sqrt{\sigma_{\text{base}}^2 + \frac{\sigma_{\text{anchor}}^2}{N_{\text{anchors}}}} = \sqrt{\sigma_{\text{base}}^2 + \frac{\sigma_{\text{base}}^2}{2}} = \sigma_{\text{base}} \cdot \sqrt{1.5} \approx \mathbf{1.225 \cdot \sigma_{\text{base}}}$$

### Stage 3: Multiplicative Penalty Factors
The post-CMR uncertainty is scaled by three operational penalty multipliers:

$$\sigma_{\text{final}} = \sigma_{\text{cmr}} \cdot f_{\text{link}} \cdot f_{\text{gap}} \cdot f_{\text{flags}}$$

---

## 3. Penalty Multiplier Formulations

### 1. RF Link Margin Multiplier ($f_{\text{link}}$)
Evaluates uplink reception quality recorded at the Master Gateway:

$$f_{\text{link}} = \begin{cases} 
1.0 & \text{if } \text{SNR} \ge 0\text{ dB (Strong link)} \\ 
1.0 + 0.05 \cdot |\text{SNR}| & \text{if } -10\text{ dB} \le \text{SNR} < 0\text{ dB} \\ 
1.5 & \text{if } \text{SNR} < -10\text{ dB (Fading edge)} 
\end{cases}$$

### 2. Temporal Age Gap Multiplier ($f_{\text{gap}}$ — Test T45)
If wireless packets were lost over preceding epochs, ground movement uncertainty increases with the elapsed duration ($\Delta t_{\text{gap}}$):

$$f_{\text{gap}} = \min\left(10.0, \, 1.0 + 0.1 \cdot \left(\frac{\Delta t_{\text{gap}}}{3600\text{ seconds}}\right)\right)$$

> [!NOTE]
> **The 10.0 Ceiling (Test T45):**
> Uncapped uncertainty formulas cause $\sigma$ to inflate by over $1,000\times$ during a multi-day network severance, causing division-by-zero numerical overflow in downstream matrix inversions. Capping $f_{\text{gap}}$ at **10.0** guarantees numerical stability.

### 3. Hardware Diagnostic Multiplier ($f_{\text{flags}}$)
Evaluates bit flags from byte 17 of the wire packet:

$$f_{\text{flags}} = 1.0 + 0.5 \cdot (\text{ext\_overflow}) + 0.3 \cdot (\text{vbat\_critical})$$

If the physical extensometer has reached the mechanical end of its stroke or battery voltage drops below $3.1\text{V}$, uncertainty inflates to discount the reading during spatial quorum voting.
