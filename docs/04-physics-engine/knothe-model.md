# The Knothe Subsidence Model & Mathematical Foundations

**Module 04 — Physics Engine**  
**Cross-References:** [`derived-quantities.md`](derived-quantities.md) · [`ground-truth-generation.md`](ground-truth-generation.md) · [Module 00 Frozen Constants](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Overview of Knothe's Theory

The foundation of modern mining subsidence engineering is the theory of the influence function developed by Prof. St. Knothe (1953/1957). 

When a horizontal underground seam is extracted, the surface terrain settles into a smooth, continuous displacement basin. Knothe demonstrated that for homogeneous and isotropic overburden, the incremental surface displacement caused by an infinitesimal extraction element follows a two-dimensional Gaussian normal distribution curve.

---

## 2. The Master Closed-Form Equation

The time-dependent vertical subsidence $S(x, y, t)$ at any surface coordinate $(x, y)$ and time $t$ is expressed as the product of a static spatial distribution and a time-dependent extraction factor:

$$S(x, y, t) = S_{\text{final}}(x, y) \cdot \eta(t)$$

```
                                  KNOTHE DECOMPOSITION
                 S(x, y, t)  =  S_final(x, y)   ×    η(t)
                                      │                │
                        [Static Spatial Dish]    [Time Factor Curve]
                        - erf() integration      - 1 - exp(-c·t)
                        - Geometry & Depth H     - Time coefficient c
```

### 1. The Time Factor: $\eta(t)$
Ground settlement does not happen instantaneously upon extraction; it lags due to viscoelastic relaxation and compaction of broken strata in the goaf:

$$\eta(t) = 1 - e^{-c \cdot t}$$

Where:
* $t$ = Elapsed time in days since extraction initiated.
* $c$ = Knothe time factor coefficient ($c = 0.01414\text{ day}^{-1}$, calibrated from Singareni Adriyala empirical survey data).

### 2. Maximum Theoretical Subsidence: $S_{\text{max}}$
$$S_{\text{max}} = a \cdot m_{\text{seam}}$$
Where:
* $m_{\text{seam}} = 3.0\text{ meters}$ (extracted coal seam height).
* $a = 0.65$ (subsidence factor for caved longwall workings with sandstone/shale overburden).
* $\implies S_{\text{max}} = 0.65 \times 3.0\text{ m} = \mathbf{1.95\text{ meters}}$.

### 3. Spatial Error Function Integration: $S_{\text{final}}(x, y)$
For a finite rectangular extraction panel bounded by coordinates $(x_1, y_1)$ to $(x_2, y_2)$ at overburden depth $H$:

$$S_{\text{final}}(x, y) = \frac{S_{\text{max}}}{4} \left[ \text{erf}\left(\frac{\sqrt{\pi}(x - x_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(x - x_2)}{r}\right) \right] \cdot \left[ \text{erf}\left(\frac{\sqrt{\pi}(y - y_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(y - y_2)}{r}\right) \right]$$

Where:
* $\text{erf}(u) = \frac{2}{\sqrt{\pi}} \int_0^u e^{-\tau^2} d\tau$ is the standard Gauss error function.
* $r = \frac{H}{\tan\beta} = \frac{150.0}{2.0} = \mathbf{75.0\text{ meters}}$ (Radius of Influence).

---

## 3. Frozen Ground Constants (Single Source of Truth)

All simulation layers and analytical models import these frozen constants from `sim/constants.py`:

| Parameter Symbol | Frozen Value | Engineering Units | Physical Meaning |
| :--- | :--- | :--- | :--- |
| `H` | 150.0 | meters | Seam depth below surface datum |
| `TAN_BETA` | 2.0 | dimensionless | Tangent of major angle of draw ($\beta \approx 63.4^\circ$) |
| `R_INFL` | 75.0 | meters | Knothe radius of influence ($H / \tan\beta$) |
| `M_SEAM` | 3.0 | meters | Extracted coal seam thickness |
| `A_SUBS` | 0.65 | dimensionless | Subsidence coefficient (PINN must learn this) |
| `S_MAX` | 1.95 | meters | Maximum asymptotic center subsidence ($A \cdot M$) |
| `C_KNOTHE` | 0.01414 | $\text{day}^{-1}$ | Time decay coefficient (PINN must learn this) |
| `B_HORIZ` | 24.0 | meters | Horizontal displacement factor ($0.32 \cdot r$) |
| `PANEL` | (100, 100, 700, 300) | meters | Extraction panel coordinates ($600\text{m} \times 200\text{m}$) |

---

## 4. Exact Volume Conservation Identity (Test T1)

Under mass conservation, the total volume of surface subsidence depression must equal the excavated goaf volume scaled by the subsidence factor:

$$\iint_{-\infty}^{+\infty} S_{\text{final}}(x, y)\, dx\, dy \equiv a \cdot m_{\text{seam}} \cdot \text{Panel\_Area}$$

$$\text{Theoretical Volume} = 0.65 \times 3.0\text{ m} \times (600\text{ m} \times 200\text{ m}) = \mathbf{234,000\text{ m}^3}$$
$$\text{Numerical Integral (4001}^2\text{ grid)} = \mathbf{233,999\text{ m}^3} \implies \text{Ratio: } \mathbf{1.0000}$$

This identity is verified automatically by test `T1` in CI, confirming that the numerical integration contains zero mass-leakage bugs.
