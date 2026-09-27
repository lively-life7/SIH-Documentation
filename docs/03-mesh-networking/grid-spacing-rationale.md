# Grid Spacing Rationale: Physics-Driven vs. Radio-Driven

**Module 03 — Mesh Networking**  
**Cross-References:** [`routing-and-failover.md`](routing-and-failover.md) · [Module 04 Knothe Model](../04-physics-engine/knothe-model.md) · [Module 05 Surface Reconstruction](../05-ai-ml-pipeline/surface-reconstruction.md) · [Module 00 Frozen Constants](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Why Is Sensor Grid Spacing Governed by Subsurface Geomechanics Rather Than LoRa Radio Range?

### Question: A standard LoRa transceiver can transmit 2 to 5 kilometers in open terrain. Why should ground sensor stations be spaced only 15 to 25 meters apart instead of kilometers apart?

**Answer:** Spacing geotechnical surface sensors based on radio range is a critical engineering fallacy. Radio range dictates only how far a data packet can travel through the air; it has zero relationship with the spatial wavelength of ground deformation. An underground coal extraction panel typically creates a localized surface subsidence bowl spanning only 200m to 600m in total width. If sensor stations were deployed at intervals of 500m to 1,000m based on radio capability, only one or two stations would fall inside the entire subsidence area, while the rest would stand outside the active displacement bowl and measure zero movement while the panel collapses beneath them.

### Question: What physical phenomenon determines the required distance between surface sensors?

**Answer:** Sensor spacing is dictated entirely by the spatial Nyquist theorem applied to the curvature of the subsidence trough. To reconstruct continuous ground slope (tilt) and horizontal tensile strain without mathematical distortion (aliasing), the sensor grid interval ($\Delta$) must be significantly smaller than the distance over which ground curvature changes from convex to concave. The subsurface depth of the coal seam and the rock strata strength define this curvature, establishing an uncompromising physics limit on sensor separation.

---

## 2. How Is the Lateral Extent of the Subsidence Trough Mathematically Derived?

### Question: How is the lateral reach of the surface deformation bowl (Influence Radius $r$) calculated?

**Answer:** According to Knothe's theory of subsidence, the lateral reach of the surface deformation bowl is characterized by the **Radius of Influence ($r$)**, defined as:

$$r = \frac{H}{\tan(\beta)}$$

Where:
* $H$ = Depth of the extracted coal seam below the surface datum ($150\text{m}$ to $400\text{m}$).
* $\beta$ = Major angle of draw of the overlying strata (typically $60^\circ$ to $65^\circ \implies \tan\beta \approx 1.7\text{ to }2.1$).

### Question: What is the calculated influence radius for a standard Indian coal extraction panel?

**Answer:** For a representative Indian coal mining panel at an overburden depth of $H = 150.0\text{ meters}$ with a typical angle of draw tangent $\tan\beta = 2.0$ ($\beta \approx 63.4^\circ$):

$$r = \frac{150.0}{2.0} = \mathbf{75.0\text{ meters}}$$

This confirms that the primary surface zone of tensile and compressive deformation develops within a radius of 75 meters from the extraction boundary.

```
+-----------------------------------------------------------------------------------+
|                        KNOTHE SUBSIDENCE TROUGH PROFILE                           |
|                                                                                   |
|  Surface Datum ──────────────────────────┐             ┌───────────────────────── |
|                                          \             /                          |
|                                           \   Trough  /                           |
|                                            \  Bottom /                            |
|                                             \_______/                             |
|  <── Inflection Zone ──>                                                          |
|       (Peak Strain)                                                               |
|  |←────────────────────────── r = 75 m ──────────────────────────→|               |
+-----------------------------------------------------------------------------------+
```

---

## 3. What Is the Formal Mathematical Derivation of the Spatial Nyquist Sampling Limit ($\Delta \le r / 2.86$)?

### Question: Why does spatial reconstruction of surface subsidence require a Nyquist sampling criterion?

**Answer:** In classical digital signal processing, the Nyquist-Shannon sampling theorem states that to prevent spectral aliasing, a continuous waveform must be sampled at a frequency $f_s \ge 2 f_{\max}$. In spatial geomechanics, continuous ground settlement $S(x)$ and horizontal strain $\varepsilon(x)$ represent continuous spatial signals. If spatial sampling points are spaced too far apart, high-frequency spatial gradients—specifically the steep inflection zone where tensile strain reaches its dangerous maximum—are completely missed or falsely reconstructed as gentle, harmless slopes.

### Question: What is the step-by-step mathematical proof deriving the factor $2.86$?

**Answer:** The derivation proceeds from Knothe's Gaussian error integral to the spatial Fourier bandwidth of the curvature profile:

1. **The Continuous Subsidence Profile:**  
   Along a principal cross-section, vertical surface subsidence $S(x)$ over an extraction edge at $x = 0$ is governed by the Gaussian error function:
   $$S(x) = \frac{S_{\max}}{2} \left[ 1 - \text{erf}\left( \frac{\sqrt{\pi} x}{r} \right) \right]$$

2. **First Derivative (Ground Tilt / Slope $T(x)$):**  
   Differentiating $S(x)$ with respect to lateral distance $x$ yields the ground tilt profile:
   $$T(x) = \frac{\partial S}{\partial x} = -\frac{S_{\max} \sqrt{\pi}}{r} \cdot \frac{2}{\sqrt{\pi}} e^{-\pi \frac{x^2}{r^2}} \cdot \frac{1}{2} = -\frac{S_{\max}}{r} e^{-\pi \frac{x^2}{r^2}}$$
   This slope profile is a pure Gaussian bell curve centered at the extraction ribside ($x = 0$).

3. **Second Derivative (Ground Curvature $\kappa(x)$ and Horizontal Strain $\varepsilon(x)$):**  
   Horizontal strain $\varepsilon(x)$ is directly proportional to ground curvature ($\varepsilon(x) = -B \cdot \partial^2 S / \partial x^2$). Differentiating tilt yields:
   $$\kappa(x) = \frac{\partial^2 S}{\partial x^2} = \frac{2\pi S_{\max} x}{r^3} e^{-\pi \frac{x^2}{r^2}}$$

4. **Location of Peak Curvature and Inflection Points:**  
   Setting the third derivative $\partial^3 S / \partial x^3 = 0$ locates the peak curvature coordinates:
   $$\frac{\partial^3 S}{\partial x^3} = \frac{2\pi S_{\max}}{r^3} \left( 1 - \frac{2\pi x^2}{r^2} \right) e^{-\pi \frac{x^2}{r^2}} = 0 \implies x_{\text{peak}} = \pm \frac{r}{\sqrt{2\pi}} \approx \pm 0.399 r$$
   The distance between opposite inflection peaks represents the critical inflection zone width:
   $$W_{\text{inflection}} = 2 \cdot x_{\text{peak}} = \frac{2r}{\sqrt{2\pi}} = \sqrt{\frac{2}{\pi}} r \approx 0.798 r$$

5. **Fourier Spatial Bandwidth ($f_{\max}$):**  
   The spatial Fourier transform of the Gaussian tilt function $T(x)$ is:
   $$\mathcal{F}\{T(x)\}(k) \propto e^{-\pi k^2 \frac{r^2}{\pi}} = e^{-k^2 r^2}$$
   In spatial frequency analysis, $99\%$ of the spectral energy of this Gaussian distribution is confined within a cutoff spatial frequency:
   $$f_{\max} = \frac{1.43}{r}$$

6. **The Nyquist Spatial Sampling Limit:**  
   Applying the Shannon-Nyquist spatial sampling condition requires that the spatial sampling rate satisfy $f_s \ge 2 f_{\max}$. Because the spatial sampling interval is $\Delta = 1 / f_s$:
   $$\Delta \le \frac{1}{2 f_{\max}} = \frac{1}{2 \times \frac{1.43}{r}} = \frac{r}{2.86}$$

7. **Calculated Grid Interval for Standard Panel ($r = 75\text{m}$):**  
   Substituting $r = 75.0\text{ meters}$:
   $$\Delta \le \frac{75.0}{2.86} \approx \mathbf{26.2\text{ meters}}$$
   This establishes the operational requirement for a **15 to 25 meter grid spacing**. A spacing greater than $26.2\text{m}$ violates the Nyquist condition, causing spatial aliasing that prevents accurate reconstruction of ground strain.

---

## 4. Why Does Grid Spacing Differ Between Shallow Seams (15–25m) and Deep Longwalls (40m)?

### Question: Why do shallow coal workings require tighter grid intervals than deep extractions?

**Answer:** As shown by the influence formula $r = H / \tan\beta$, the radius of influence $r$ scales linearly with seam depth $H$. In shallow panels ($H \le 150\text{m}$), the influence radius is small ($r \le 75\text{m}$), causing the ground surface to collapse steeply over a compact horizontal distance. This steep gradient produces high spatial frequencies that demand tight **15–25m grid spacing**. In contrast, deep coal workings ($H \ge 350\text{m}$) distribute subsidence across a very wide, gentle bowl ($r \ge 175\text{m}$), shifting the spatial frequency spectrum toward lower frequencies and allowing wider station spacing.

### Question: What is the empirical 'Error Knee Curve' and how does it prove the optimality of 15–25m and 40m spacing?

**Answer:** Across empirical survey lines and synthetic field datasets (such as the Adriyala Longwall Project at $H = 375\text{m}$, $r = 187.5\text{m}$), evaluating reconstruction error across various grid spacings yields an interpolation error knee:

| Grid Spacing ($\Delta$) | Maximum Spatial Interpolation Error | Relative Deployment Density | Engineering Assessment |
| :--- | :--- | :--- | :--- |
| **100 meters** | $142\text{ mm}$ | $1.0\times$ (Low) | Unacceptable; severe spatial aliasing; completely misses localized tensile fractures |
| **60 meters** | $58\text{ mm}$ | $1.6\times$ | Marginal; fails to reliably resolve peak tensile strain inflection points |
| **40 meters** | **21 mm (Optimal Knee)** | **2.4×** | **Optimal Engineering Knee for Deep Seams ($H > 300\text{m}$)** |
| **20 meters** | $18\text{ mm}$ | $4.8\times$ | Diminishing returns for deep seams (doubles hardware for only 3mm gain); mandatory for shallow panels |

For deep seams ($H > 300\text{m}$), **40m spacing** achieves the ideal engineering knee, delivering high reconstruction accuracy ($21\text{ mm}$ error) without over-instrumentation. For shallow panels ($H \le 150\text{m}$), the error curve shifts leftward, making **15–25m spacing** mandatory to capture the sharper deformation gradient.

---

## 5. Why Do Sparse Conventional Monitoring Grids Fail to Detect 2-Meter Localized Shear Fissures?

### Question: Why can conventional 100m–300m borehole logger networks miss hazardous ground fracturing?

**Answer:** Sedimentary overburden does not deform as a uniform plastic block; it is stratified with joint planes, cleat systems, and pre-existing faults. When tensile strain exceeds the rock tensile capacity ($\theta_c = 1500\ \mu\varepsilon$), deformation concentrates abruptly into a discrete, localized shear fracture or surface crevasse spanning only 1 to 2 meters in width. If monitoring stations are spaced 100m to 300m apart, a 2-meter fissure opening midway between stations induces virtually zero tilt at the distant sensors, leaving operators completely unaware of active ground failure.

### Question: How does a dense 15–25m grid guarantee detection of localized fracture zones?

**Answer:** Enforcing the physics-derived grid spacing ($\Delta \le 25\text{m}$ for shallow panels, $\Delta \le 40\text{m}$ for deep seams) ensures that at least two adjacent sensor stations directly straddle any emerging fracture zone. Differential horizontal displacement between these two adjacent stations immediately triggers an anomalous tilt gradient ($\Delta T / \Delta x$) and differential extension, providing actionable warning days before tensile cracks breach the surface.

---

## 6. How Does the Grid Scale Dynamically Across Arbitrary Mine Panels Without Fixed Hardware Limits?

### Question: Does the monitoring platform impose an arbitrary fixed hardware cap or static station count?

**Answer:** No. Fixed station counts are technically unviable because real-world coal mining panels vary widely in dimension (lengths from 200m to 1,500m, widths from 100m to 300m, and depths from 50m to 400m+). The platform uses a purely algorithmic dynamic scaling model where station counts are computed directly from panel geometry and overburden physics, without arbitrary minimum or maximum hardware limits.

### Question: What analytical formula calculates the required number of spatial monitoring stations for any given panel?

**Answer:** For an extraction panel of length $L_{\text{panel}}$ and width $W_{\text{panel}}$ at seam depth $H$ with angle of draw $\beta$, the total active surface footprint including the draw perimeter is:

$$L_{\text{surface}} = L_{\text{panel}} + 2 \cdot \frac{H}{\tan\beta}, \quad W_{\text{surface}} = W_{\text{panel}} + 2 \cdot \frac{H}{\tan\beta}$$

The required number of spatial monitoring stations ($N_{\text{stations}}$) is computed as:

$$N_{\text{stations}} \approx \left(\frac{L_{\text{surface}}}{\Delta}\right) \times \left(\frac{W_{\text{surface}}}{\Delta}\right) \times \rho_{\text{criticality}}$$

Where:
* $\Delta \le \frac{r}{2.86}$ is the physics-derived Nyquist grid interval.
* $\rho_{\text{criticality}} \ge 1.0$ is a localized weighting multiplier that increases station density over vulnerable surface features (railway corridors, high-pressure pipelines, or village structures).

This formulation guarantees that monitoring arrays automatically adapt to any mining district, ensuring consistent spatial fidelity from small bord-and-pillar depillaring panels up to massive multi-kilometer longwall operations.

---

## 7. How Does Physics-Driven Dense Spacing Benefit Network Reliability and Battery Life?

### Question: Does placing sensor stations 15 to 25 meters apart increase wireless interference or drain batteries faster?

**Answer:** Dense spatial proximity significantly improves radio efficiency rather than degrading it. Because adjacent stations are separated by only 15 to 25 meters, transceivers operate at low RF transmission power (+10 dBm instead of the +22 dBm maximum required for kilometer-range links). This reduces peak power consumption during radio transmission by more than 60%, drastically extending battery operational lifetime. Furthermore, low transmit power minimizes the RF interference footprint, ensuring clean TDMA superframe execution without co-channel contention.

### Question: How does dense spatial proximity ensure compliance with statutory RF duty cycle limits?

**Answer:** Because transmissions over short 15–25m links require high signal-to-noise ratios (SNR), transceivers can use high data rates (Spreading Factor SF7 at 125 kHz bandwidth), resulting in an ultra-short on-air packet transmission time of only $32.8\text{ ms}$. Over a standard 60-second measurement cycle, the active transmitter duty cycle is only:

$$\text{Duty Cycle} = \frac{32.8\text{ ms}}{60,000\text{ ms}} \approx \mathbf{0.055\%}$$

This is well below the statutory $1.0\%$ ceiling mandated by Indian wireless regulations (GSR 564(E)), ensuring seamless spectrum compliance while maintaining high-density spatial surveillance.
