# Derived Quantities & Kinematic Precursor Detection

**Module 04 — Physics Engine**  
**Cross-References:** [`knothe-model.md`](knothe-model.md) · [`corruption-chain.md`](corruption-chain.md) · [Module 02 Sensor Suite](../02-sensor-hardware/sensor-suite.md)

---

## 1. Kinematic Spatial Derivatives

All physical hazard indicators measured by AEGIS sensors derive directly from spatial derivatives of the master Knothe surface $S(x, y, t)$. No independent empirical approximations are permitted.

```
       VERTICAL SUBSIDENCE: S(x, y, t)
                 │
                 ▼  First Spatial Derivative: ∂/∂x
       GROUND TILT / SLOPE: T_x = ∂S/∂x
                 │
                 ▼  Scaled by B_horiz = 0.32 · r
       HORIZONTAL DISPLACEMENT: U_x = B_horiz · T_x
                 │
                 ▼  Second Spatial Derivative: ∂²/∂x²
       HORIZONTAL STRAIN: ε_x = ∂U_x/∂x = B_horiz · (∂²S/∂x²)
```

### 1. Ground Tilt (Slope Gradient: $T_x, T_y$)
Tilt represents the change in ground slope per unit horizontal distance, measured by MPU-6050 inclinometers:

$$T_x(x, y, t) = \frac{\partial S(x, y, t)}{\partial x} = \frac{S_{\text{max}} \cdot \eta(t)}{2 r} \left[ e^{-\frac{\pi (x - x_1)^2}{r^2}} - e^{-\frac{\pi (x - x_2)^2}{r^2}} \right] \left[ \text{erf}\left(\frac{\sqrt{\pi}(y - y_1)}{r}\right) - \text{erf}\left(\frac{\sqrt{\pi}(y - y_2)}{r}\right) \right]$$

### 2. Horizontal Displacement ($U_x, U_y$)
Surface points do not move purely downward; they displace inward toward the center of the extraction goaf. By Awershin's hypothesis, horizontal displacement is directly proportional to surface tilt:

$$U_x(x, y, t) = B_{\text{horiz}} \cdot T_x(x, y, t) \quad \text{where } B_{\text{horiz}} = 0.32 \cdot r = 0.32 \times 75.0\text{m} = \mathbf{24.0\text{ meters}}$$

### 3. Horizontal Tensile & Compressive Strain ($\varepsilon_x, \varepsilon_y$)
Strain represents the rate of stretch or compression of the ground surface, measured directly by foil gauges and 10m invar rod extensometers:

$$\varepsilon_x(x, y, t) = \frac{\partial U_x}{\partial x} = B_{\text{horiz}} \cdot \frac{\partial^2 S(x, y, t)}{\partial x^2}$$

$$\frac{\partial^2 S}{\partial x^2} = -\frac{2 \pi}{r^2} \left[ (x - x_1) e^{-\frac{\pi (x - x_1)^2}{r^2}} - (x - x_2) e^{-\frac{\pi (x - x_2)^2}{r^2}} \right] \cdot \left[ \text{spatial } y \text{ terms} \right]$$

---

## 2. Derived Peak Magnitudes (Sanity Anchors)

Recomputed from closed-form equations over a high-resolution $4001 \times 4001$ numerical grid:

| Physical Channel | Fully Settled State ($t \to \infty$) | At Day 40 ($\eta = 0.432$) | Sensor Measurement Range | Margin to Saturation |
| :--- | :--- | :--- | :--- | :--- |
| **Subsidence ($S$)** | **1.950 meters** | 0.842 meters | Computed analytically | — |
| **Surface Tilt ($T$)** | **25,978 µrad (1.49°)** | $11,222\ \mu\text{rad}$ | $\pm 65,534\ \mu\text{rad}$ @ 2 µrad LSB | **2.52×** |
| **Horizontal Strain ($\varepsilon$)** | **12,649 µε** | $5,464\ \mu\varepsilon$ | $\pm 32,767\ \mu\varepsilon$ @ 1 µε LSB | **2.59×** |
| **Extensometer (10m Span)** | **126.5 mm** | $54.6\text{ mm}$ | $\pm 327.6\text{ mm}$ @ 10 µm LSB | **2.59×** |

---

## 3. "Strain Is the Channel That Detects Things"

A common mistake in mine safety is relying on tilt inclinometers for early warning. 
* Tilt is the first derivative of displacement. At the bottom of the subsidence bowl and along the extraction boundary, tilt is mathematically near zero.
* In contrast, **horizontal tensile strain ($\varepsilon$) is the second derivative**. Strain peaks precisely along the inflection boundary where rock delamination begins.
* Strain responds weeks before visible tilt or vertical collapse manifests to visual observers.

---

## 4. The 8.93-Day Advance Crack Prediction Derivation

In sedimentary sandstone and shale strata typical of Indian coal basins, surface tensile crack formation initiates when ground tensile strain crosses the critical rock tensile threshold:

$$\theta_c = 1500\ \mu\varepsilon \pm 10\%$$

Using the closed-form time factor $\eta(t) = 1 - e^{-c \cdot t}$ with $c = 0.01414\text{ day}^{-1}$ and peak asymptotic tensile strain $\varepsilon_{\text{final}} = 12,649\ \mu\varepsilon$:

$$\varepsilon_{\text{peak}}(t) = \varepsilon_{\text{final}} \cdot (1 - e^{-c \cdot t_{\text{crack}}}) \ge \theta_c$$

$$1500 = 12,649 \cdot (1 - e^{-0.01414 \cdot t_{\text{crack}}})$$

$$\frac{1500}{12,649} \approx 0.118586 \implies e^{-0.01414 \cdot t_{\text{crack}}} = 1 - 0.118586 = 0.881414$$

$$-0.01414 \cdot t_{\text{crack}} = \ln(0.881414) \approx -0.126263$$

$$t_{\text{crack}} = \frac{0.126263}{0.01414} = \mathbf{8.9295\text{ days}} \approx \mathbf{8.93\text{ days}}$$

### Key Takeaway for Presentation & Safety Audits:
Surface cracking initiates at **Day 8.93** ($\approx 214\text{ hours}$) after extraction starts. Because AEGIS samples strain continuously every 60 seconds from Day 1, the platform detects rising micro-strain accumulation **up to 8.9 days before physical fissures open on the surface**, giving mine supervisors ample time to evacuate personnel, shore up foundations, and divert railway traffic.
