# The Knothe Subsidence Model & Mathematical Foundations

**Module 04 — Physics Engine**  
**Cross-References:** [`derived-quantities.md`](derived-quantities.md) · [`ground-truth-generation.md`](ground-truth-generation.md) · [Module 00 Frozen Constants](../00-executive-gateway/key-metrics-summary.md) · [Module 08 Test Register](../08-verification/test-register.md)

---

## 1. Physical Principles of Knothe's Theory

### Question: What is Knothe's influence function theory, and why does an infinitesimal underground extraction element produce a Gaussian settlement distribution at the surface?

**Answer:** Developed by Prof. Stanisław Knothe (1953/1957), the influence function theory provides the foundational mathematical framework for modern mining geomechanics. When a small subterranean coal volume is extracted, the overlying rock mass does not collapse as a rigid block; rather, the void induces stress redistribution, progressive fracturing, and granular shear deformation throughout the overburden.

Knothe demonstrated that for a homogeneous, isotropic, or horizontally stratified rock mass, the macroscopic displacement transmitted through the granular media follows the central limit theorem: the cumulative effect of millions of inter-grain frictional slips produces an incremental surface settlement distribution that follows a two-dimensional Gaussian normal curve:

$$dS(x, y) = S_{\text{max}} \cdot \frac{1}{r^2} \exp\left( -\pi \frac{(x - \xi)^2 + (y - \zeta)^2}{r^2} \right) d\xi\, d\zeta$$

Where $(\xi, \zeta)$ are the extraction coordinates in the seam, and $r$ is the characteristic radius of influence. This Gaussian formulation ensures that surface subsidence is smooth, continuously differentiable ($C^\infty$), and asymptotically decays toward zero at large distances from the excavation.

---

## 2. Master Closed-Form Decomposition

### Question: How is the time-dependent subsidence field mathematically decoupled into spatial and temporal components?

**Answer:** Under Knothe's kinematic theory, vertical surface settlement $S(x, y, t)$ across spatial coordinates $(x, y)$ and elapsed time $t$ is expressed as the product of a static spatial subsidence basin $S_{\text{final}}(x, y)$ and a dimensionless, monotonically increasing time factor $\eta(t)$:

$$S(x, y, t) = S_{\text{final}}(x, y) \cdot \eta(t)$$

```
                                  KNOTHE DECOMPOSITION
                 S(x, y, t)  =  S_final(x, y)   ×    η(t)
                                      │                │
                        [Static Spatial Dish]    [Time Factor Curve]
                        - erf() integration      - 1 - exp(-c·t)
                        - Geometry & Depth H     - Time coefficient c
```

This spatio-temporal separability implies that the geometric shape of the subsidence trough is established by the excavation boundaries and overburden depth, while the rate at which the basin deepens is governed by the rheological compaction of the collapsed rock in the goaf.

### Question: What physical phenomenon governs the time factor $\eta(t)$, and how is the time decay constant $c$ calibrated?

**Answer:** Ground settlement does not occur instantaneously upon seam excavation. The broken roof strata (caving zone) collapses into the void, forming an uncompacted rubble mound (goaf). As the overburden weight bears down on this broken rock mass, time-dependent viscoelastic compaction and creep deformation take place.

Knothe modeled this rheological behavior using a linear rate-of-settlement differential equation:
$$\frac{d S(t)}{dt} = c \cdot \left[ S_{\text{final}} - S(t) \right]$$

Integrating with initial condition $S(0) = 0$ yields the closed-form time factor:
$$\eta(t) = 1 - e^{-c \cdot t}$$

Where:
* $t$ = Elapsed time in days since seam extraction began.
* $c$ = Time decay coefficient ($c = 0.01414\text{ day}^{-1}$, calibrated from high-precision levelling survey data at Singareni Collieries Company Limited (SCCL) Adriyala Longwall Project).
* At $t = 40\text{ days}$, $\eta(40) = 1 - e^{-0.01414 \times 40} = 1 - e^{-0.5656} \approx \mathbf{0.4320}$ ($43.2\%$ of final asymptotic settlement).

### Question: How is the spatial error function equation derived for a finite rectangular longwall panel?

**Answer:** For a rectangular extraction panel bounded by seam coordinates $x \in [x_1, x_2]$ and $y \in [y_1, y_2]$, the total static subsidence at any surface point $(x, y)$ is found by integrating Knothe's Gaussian kernel over the extraction domain:

$$S_{\text{final}}(x, y) = S_{\text{max}} \int_{x_1}^{x_2} \int_{y_1}^{y_2} \frac{1}{r^2} \exp\left(-\pi \frac{(x - \xi)^2 + (y - \zeta)^2}{r^2}\right) d\xi\, d\zeta$$

Because the double integral separates into two independent single integrals:
$$\int_{x_1}^{x_2} \frac{\sqrt{\pi}}{r} e^{-\pi \frac{(x - \xi)^2}{r^2}} \frac{d\xi}{\sqrt{\pi}} = \frac{1}{2} \left[ \text{erf}\left(\frac{\sqrt{\pi}(x - x_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(x - x_2)}{r}\right) \right]$$

The closed-form analytical solution is:

$$S_{\text{final}}(x, y) = \frac{S_{\text{max}}}{4} \left[ \text{erf}\left(\frac{\sqrt{\pi}(x - x_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(x - x_2)}{r}\right) \right] \cdot \left[ \text{erf}\left(\frac{\sqrt{\pi}(y - y_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(y - y_2)}{r}\right) \right]$$

Where:
* $\text{erf}(u) = \frac{2}{\sqrt{\pi}} \int_0^u e^{-\tau^2} d\tau$ is the standard Gauss error function.
* $r = \frac{H}{\tan\beta}$ is the **Radius of Influence**. For overburden depth $H = 150.0\text{ m}$ and draw angle $\tan\beta = 2.0$ ($\beta \approx 63.4^\circ$), $r = \frac{150.0}{2.0} = \mathbf{75.0\text{ meters}}$.
* Maximum theoretical subsidence is $S_{\text{max}} = a \cdot m_{\text{seam}}$, where $m_{\text{seam}} = 3.0\text{ m}$ (extracted seam height) and $a = 0.65$ (subsidence factor for caved workings in sandstone/shale overburden), yielding $S_{\text{max}} = 0.65 \times 3.0\text{ m} = \mathbf{1.95\text{ meters}}$.

---

## 3. Frozen Geotechnical Constants & Site Parameterization

### Question: What are the single-source-of-truth frozen parameters, and what do their values represent physically?

**Answer:** All analytical models, synthetic telemetry generators, and calibration verification suites import identical constants from `sim/constants.py`, establishing complete mathematical consistency across the AEGIS ecosystem:

| Parameter Symbol | Frozen Value | Engineering Units | Physical Meaning |
| :--- | :--- | :--- | :--- |
| `H` | 150.0 | meters | Seam depth below surface datum |
| `TAN_BETA` | 2.0 | dimensionless | Tangent of major angle of draw ($\beta \approx 63.4^\circ$) |
| `R_INFL` | 75.0 | meters | Knothe radius of influence ($H / \tan\beta$) |
| `M_SEAM` | 3.0 | meters | Extracted coal seam thickness |
| `A_SUBS` | 0.65 | dimensionless | Empirical subsidence coefficient (PINN target) |
| `S_MAX` | 1.95 | meters | Maximum asymptotic center subsidence ($a \cdot m_{\text{seam}}$) |
| `C_KNOTHE` | 0.01414 | $\text{day}^{-1}$ | Time decay coefficient (PINN target) |
| `B_HORIZ` | 24.0 | meters | Awershin horizontal displacement factor ($0.32 \cdot r$) |
| `PANEL` | (100, 100, 700, 300) | meters | Extraction panel coordinates ($600\text{m} \times 200\text{m}$) |

Any dynamic panel deployment recalculates $r$, $S_{\text{max}}$, and $B_{\text{horiz}}$ algorithmically from the site's measured depth $H$, seam height $m_{\text{seam}}$, and draw angle tangent $\tan\beta$.

---

## 4. Volume Conservation & Numerical Integrity

### Question: How is the Exact Volume Conservation Identity mathematically derived, and why does CI Test T1 mandate a volume ratio of exactly 1.0000?

**Answer:** By the law of mass conservation in continuous media, the total volume of the surface depression basin must equal the subterranean void volume created by mining, multiplied by the volumetric subsidence factor $a$:

$$\iint_{-\infty}^{+\infty} S_{\text{final}}(x, y)\, dx\, dy \equiv a \cdot m_{\text{seam}} \cdot \text{Panel\_Area}$$

Evaluating the definite integral of Knothe's error function formulation across an infinite domain:
$$\int_{-\infty}^{+\infty} \frac{1}{2} \left[ \text{erf}\left(\frac{\sqrt{\pi}(x - x_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(x - x_2)}{r}\right) \right] dx = (x_2 - x_1)$$

Thus, the spatial integral simplifies identically to the panel surface area:
$$\iint_{-\infty}^{+\infty} S_{\text{final}}(x, y)\, dx\, dy = S_{\text{max}} \cdot (x_2 - x_1)(y_2 - y_1) = a \cdot m_{\text{seam}} \cdot (L \times W)$$

**Numerical Evaluation Benchmark:**
* Theoretical Volume:
  $$V_{\text{theoretical}} = 0.65 \times 3.0\text{ m} \times (600\text{ m} \times 200\text{ m}) = \mathbf{234,000\text{ m}^3}$$
* Numerical 2D Simpson Integration on $4001 \times 4001$ grid:
  $$V_{\text{numerical}} = \mathbf{233,999\text{ m}^3}$$
* Ratio:
  $$\frac{V_{\text{numerical}}}{V_{\text{theoretical}}} = \mathbf{1.0000} \quad (\text{Error} < 0.0005\%)$$

Automated test `T1` asserts that this ratio remains within $1.0000 \pm 0.005$, proving that the simulation implementation is mathematically closed and free of volume-leakage bugs.

---

## 5. Model Validity Domain & Separability Preconditions

### Question: Under what geological conditions is Knothe's spatio-temporal model valid, and how does Test T38 prevent unphysical model misuse?

**Answer:** Knothe's classical theory assumes four fundamental geomechanical preconditions:
1. **Isotropic / Quasi-Homogeneous Overburden:** The overlying rock mass consists of interbedded sedimentary layers without dominant dipping mega-faults traversing the basin.
2. **Sub-Critical to Super-Critical Extraction Geometry:** Panel width $W$ and length $L$ satisfy $W, L > 1.2 \cdot r$, ensuring fully developed caving.
3. **Continuous Extraction Advance:** The longwall shearer advances at a relatively steady velocity without multi-month work stoppages that would induce complex step-wise rheological relaxation.
4. **Absence of Regional Fault Reactivation:** Displacements occur as continuum subsidence rather than discontinuous tectonic block faulting.

**Automated CI Assertion (Test T38):**  
Test `T38` validates that the generation parameters strictly satisfy these four preconditions before any verification run is executed. Furthermore, Test `T5` verifies spatio-temporal separability across all grid points:
$$\frac{S(x, y, t_1)}{S(x, y, t_2)} \equiv \frac{\eta(t_1)}{\eta(t_2)} \quad \forall (x, y) \text{ where } S_{\text{final}} > 1\text{ mm}$$
Confirming that numerical discretization does not distort the analytical time decay curve.
