# Derived Quantities & Kinematic Precursor Detection

**Module 04 — Physics Engine**  
**Cross-References:** [`knothe-model.md`](knothe-model.md) · [`corruption-chain.md`](corruption-chain.md) · [Module 02 Sensor Suite](../02-sensor-hardware/sensor-suite.md) · [Module 08 Test Register](../08-verification/test-register.md)

---

## 1. Kinematic Spatial Derivatives & Geotechnical Foundations

### Question: What is the rigorous physical relationship between vertical subsidence and surface hazard indicators such as tilt, horizontal displacement, and ground strain?

**Answer:** In mining subsidence engineering, surface hazard indicators are not independent empirical variables. Under continuum mechanics, all measurable ground deformation phenomena are strict spatial derivatives of the master continuous vertical subsidence surface $S(x, y, t)$.

AEGIS establishes an uncompromising derivative hierarchy where every sensor channel correlates directly to a kinematic derivative of the underlying Knothe displacement field:

```
                  VERTICAL SUBSIDENCE: S(x, y, t)
                               │
                               ▼  First Spatial Derivative: ∂/∂x
                     GROUND TILT / SLOPE: T_x = ∂S/∂x
                               │
                               ▼  Scaled by Awershin Parameter: B_horiz = 0.32 · r
                     HORIZONTAL DISPLACEMENT: U_x = B_horiz · T_x
                               │
                               ▼  Second Spatial Derivative: ∂²/∂x²
                     HORIZONTAL STRAIN: ε_x = ∂U_x/∂x = B_horiz · (∂²S/∂x²)
```

This mathematical hierarchy eliminates unphysical, disconnected sensor interpretations. An inclinometer measures the first spatial derivative, while foil strain gauges and extensometers measure the second spatial derivative.

### Question: How is lateral ground displacement derived from vertical subsidence without empirical ad-hoc guessing?

**Answer:** Surface points overlying an underground extraction goaf do not settle vertically downward; they displace inward toward the center of extraction. In classical strata mechanics, this phenomenon is governed by **Awershin's Hypothesis**, which establishes that horizontal displacement $U_x$ is directly proportional to surface tilt (the spatial gradient of vertical subsidence):

$$U_x(x, y, t) = B_{\text{horiz}} \cdot \frac{\partial S(x, y, t)}{\partial x} = B_{\text{horiz}} \cdot T_x(x, y, t)$$

Where the horizontal displacement coefficient $B_{\text{horiz}}$ is a geometric function of the strata influence radius $r$:
$$B_{\text{horiz}} = 0.32 \cdot r$$

For an extraction depth of $H = 150.0\text{ meters}$ and strata draw angle $\tan\beta = 2.0$ ($r = 75.0\text{ meters}$):
$$B_{\text{horiz}} = 0.32 \times 75.0\text{ m} = \mathbf{24.0\text{ meters}}$$

This formulation guarantees that horizontal ground displacements vanish at the exact center of the subsidence basin (where tilt is zero) and reach their maximum along the inflection perimeter.

### Question: What are the exact closed-form analytical equations for ground tilt and horizontal strain across a rectangular extraction panel?

**Answer:** Differentiating the master Knothe double-error-function equation yields exact, continuous analytical expressions for both first and second spatial derivatives.

**1. Ground Tilt ($T_x = \partial S / \partial x$):**
$$T_x(x, y, t) = \frac{S_{\text{max}} \cdot \eta(t)}{2 r} \left[ e^{-\frac{\pi (x - x_1)^2}{r^2}} - e^{-\frac{\pi (x - x_2)^2}{r^2}} \right] \left[ \text{erf}\left(\frac{\sqrt{\pi}(y - y_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(y - y_2)}{r}\right) \right]$$

**2. Ground Curvature ($\kappa_x = \partial^2 S / \partial x^2$):**
$$\frac{\partial^2 S}{\partial x^2} = -\frac{\pi S_{\text{max}} \cdot \eta(t)}{r^3} \left[ (x - x_1) e^{-\frac{\pi (x - x_1)^2}{r^2}} - (x - x_2) e^{-\frac{\pi (x - x_2)^2}{r^2}} \right] \left[ \text{erf}\left(\frac{\sqrt{\pi}(y - y_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(y - y_2)}{r}\right) \right]$$

**3. Horizontal Tensile & Compressive Strain ($\varepsilon_x$):**
$$\varepsilon_x(x, y, t) = B_{\text{horiz}} \cdot \frac{\partial^2 S(x, y, t)}{\partial x^2}$$

By evaluating these analytical derivatives, AEGIS achieves exact ground truth benchmark calculations without the truncation errors inherent in numerical finite-difference grids.

---

## 2. Sensor Dynamic Range & Peak Magnitude Anchors

### Question: What are the theoretical peak magnitudes for subsidence, tilt, strain, and extensometer elongation, and how do they validate sensor dynamic range (Test T4)?

**Answer:** Using a high-resolution $4001 \times 4001$ numerical grid evaluated at fully settled equilibrium ($t \to \infty, \eta = 1.0$) and intermediate advance ($t = 40\text{ days}, \eta = 0.432$), the exact peak values are computed as follows:

| Physical Channel | Fully Settled State ($t \to \infty$) | At Day 40 ($\eta = 0.432$) | Hardware Measurement Range | Margin to Saturation |
| :--- | :--- | :--- | :--- | :--- |
| **Subsidence ($S$)** | **1.950 meters** | $0.842\text{ meters}$ | Analytical Benchmark | — |
| **Surface Tilt ($T$)** | **25,978 µrad ($1.49^°$)** | $11,222\ µrad$ | $± 65,534\ µrad$ @ $2\ µrad$ LSB | **2.52×** |
| **Horizontal Strain ($ε$)** | **12,649 µε** | $5,464\ µε$ | $± 32,767\ µε$ @ $1\ µε$ LSB | **2.59×** |
| **Extensometer (10m Span)** | **126.5 mm** | $54.6\text{ mm}$ | $± 327.6\text{ mm}$ @ $10\ µm$ LSB | **2.59×** |

**Engineering Validation:**
1. **Dynamic Headroom:** Every physical channel maintains a safety factor greater than $2.5\times$ above the maximum theoretical asymptotic deformation. Transducers will never saturate, even during extreme geological super-subsidence events.
2. **Quantization Precision:** With tilt LSB at $2\ \mu\text{rad}$ and strain LSB at $1\ \mu\varepsilon$, the initial micro-deformations occurring during the first 48 hours of mining advance are captured with $> 500$ discrete counts of resolution before visible ground fracturing occurs.
3. **CI Test Assertion (Test T4):** Automated test `T4` verifies that recomputed peak magnitudes match closed-form analytical peaks within $\pm 2.0\%$, ensuring no parameter drift across software updates.

---

## 3. Physical Precursor Mechanics: Why Strain Detects Collapse First

### Question: Why is horizontal tensile strain ($\varepsilon$) the definitive kinematic precursor for catastrophic ground failure rather than tilt or vertical settlement?

**Answer:** Relying primarily on tilt inclinometers or GNSS elevation pegs for early warning represents a dangerous misconception in mine monitoring:

1. **Spatial Invariant of the Extraction Boundary:**  
   Ground tilt is the first spatial derivative of subsidence ($\partial S / \partial x$). Directly above the extraction ribside and at the center of the goaf, ground tilt is mathematically zero or near-zero during early extraction stages. A tilt sensor positioned over the extraction boundary will report zero tilt while enormous tensile forces are tearing the rock apart.
2. **Second Derivative Geomechanical Precursor:**  
   Horizontal strain $\varepsilon$ is the second spatial derivative ($\partial^2 S / \partial x^2$). Strain reaches its absolute mathematical maximum directly along the inflection boundary of the subsidence trough ($x \approx \pm 0.399 r$). This inflection perimeter is precisely where the overlying rock strata transitions from bending to tensile delamination.
3. **Micro-Strain Precursor Accumulation:**  
   Brittle sedimentary rock formations (sandstone, shale, and coal measure rocks) possess high compressive strength but very low tensile strength ($\sigma_t \approx 0.1 \sigma_c$). Tensile micro-strain accumulates in the upper strata weeks before shearing occurs, producing measurable micro-strain increases long before any gross vertical displacement or tilt can be visually observed by mine personnel.

```
DEFORMATION CHANNEL TIMELINE:
Day 0: Seam extraction initiates.
Day 1–3: Horizontal strain rises from 0 → 500 µε. (Tilt & Settlement undetectable: < 1 cm).
Day 8.93: Horizontal strain breaches critical tensile threshold θ_c = 1,500 µε.
          MICRO-CRACKING INITIATES INTERNALLY.
Day 25–40: Macroscopic surface fissures widen; vertical settlement exceeds 80 cm.
```

---

## 4. Mathematical Derivation of the 8.93-Day Advance Crack Warning

### Question: What is the rigorous mathematical derivation proving that AEGIS provides up to 8.93 days of advance warning before surface cracks physically open (Test T10)?

**Answer:** In sedimentary coal basins (such as the Godavari Valley and Raniganj Coalfields), in-situ rock mechanics testing establishes that tensile fissure initiation in surface sandstone/topsoil occurs when horizontal tensile strain exceeds the critical rock failure threshold:

$$\theta_c = 1500\ \mu\varepsilon \pm 10\%$$

Using Knothe's time-dependent strain evolution equation along the maximum tensile inflection zone:
$$\varepsilon_{\text{peak}}(t) = \varepsilon_{\text{final}} \cdot \eta(t) = \varepsilon_{\text{final}} \cdot \left( 1 - e^{-c \cdot t} \right)$$

Substituting the frozen geotechnical site parameters:
* Asymptotic peak tensile strain: $\varepsilon_{\text{final}} = 12,649\ \mu\varepsilon$
* Time decay coefficient: $c = 0.01414\text{ day}^{-1}$
* Critical crack threshold: $\theta_c = 1,500\ \mu\varepsilon$

We solve for the exact elapsed time $t_{\text{crack}}$ at which peak strain crosses the threshold:

$$1500 = 12,649 \cdot \left( 1 - e^{-0.01414 \cdot t_{\text{crack}}} \right)$$

$$\frac{1500}{12,649} \approx 0.118586$$

$$1 - e^{-0.01414 \cdot t_{\text{crack}}} = 0.118586 \implies e^{-0.01414 \cdot t_{\text{crack}}} = 1 - 0.118586 = 0.881414$$

Taking the natural logarithm of both sides:
$$-0.01414 \cdot t_{\text{crack}} = \ln(0.881414) \approx -0.126263$$

$$t_{\text{crack}} = \frac{0.126263}{0.01414} = \mathbf{8.9295\text{ days}} \approx \mathbf{8.93\text{ days}} \quad (\approx 214.3\text{ hours})$$

### Regulatory & Statutory Significance:
Surface cracking initiates at **Day 8.93** after seam extraction begins. Because the AEGIS wireless sensor network samples strain continuously every 60 seconds starting on Day 0, the platform detects the accelerating micro-strain rate ($d\varepsilon/dt$) within the first 48 to 72 hours. 

This provides mine management and DGMS regulators with **up to 8.9 days of lead time** before physical tension cracks breach the surface. This operational window allows safe evacuation of heavy machinery, rerouting of surface transport corridors, and reinforcement of overlying civil infrastructure. Automated test `T10` verifies this trip time in CI simulation runs.

---

## 5. Derivative Numerical Integrity & Verification

### Question: How does the software ensure that analytical derivative formulations do not contain mathematical errors or sign flips (Test T3)?

**Answer:** Analytical expressions for high-order spatial derivatives of multivariate error functions are prone to algebraic sign errors. AEGIS enforces continuous numerical verification via automated test `T3`.

Across a dense spatial test matrix of $10,000$ points, the analytical gradient expressions are benchmarked against central finite-difference approximations with step size $h = 10^{-4}\text{ m}$:

$$T_x^{\text{numerical}}(x, y) = \frac{S(x + h, y) - S(x - h, y)}{2h}$$

$$\kappa_x^{\text{numerical}}(x, y) = \frac{S(x + h, y) - 2S(x, y) + S(x - h, y)}{h^2}$$

**Acceptance Criterion:**
$$\max\left| T_x^{\text{analytical}} - T_x^{\text{numerical}} \right| < 1 \times 10^{-7}\ \text{rad}$$
$$\max\left| \kappa_x^{\text{analytical}} - \kappa_x^{\text{numerical}} \right| < 1 \times 10^{-7}\ \text{m}^{-1}$$

If an algebraic error or sign flip is introduced during refactoring, Test `T3` halts the CI build immediately, guaranteeing total mathematical integrity.
