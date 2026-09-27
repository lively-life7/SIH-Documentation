# Grid Spacing Rationale: Physics-Driven vs. Radio-Driven

**Module 03 — Mesh Networking**  
**Cross-References:** [`routing-and-failover.md`](routing-and-failover.md) · [Module 04 Knothe Model](../04-physics-engine/knothe-model.md) · [Module 05 Surface Reconstruction](../05-ai-ml-pipeline/surface-reconstruction.md)

---

## 1. The Core Engineering Fallacy: Radio Range vs. Physics Resolution

A frequent design mistake in industrial IoT projects is spacing sensor nodes based on the maximum transmission range of the radio. 
* A modern LoRa transceiver (Semtech SX1262) operating at $+22\text{ dBm}$ easily achieves communication ranges of $2\text{ to }5\text{ km}$ across open terrain.
* However, spacing ground stations at $500\text{m}$ to $1,000\text{m}$ intervals over an underground coal extraction panel is scientifically useless. A subsidence basin typically spans only $200\text{m}$ to $600\text{m}$ in width. Nodes placed kilometers apart would stand outside the active displacement bowl and record zero movement while the panel collapses beneath them.

**Grid spacing is strictly dictated by ground mechanics and spatial Nyquist sampling, never by radio transmission capability.**

---

## 2. Derivation of the Nyquist Spatial Sampling Limit

Subsidence trough formation follows Knothe's Gaussian error integral. The lateral reach of the surface deformation bowl is characterized by the **Radius of Influence ($r$)**:

$$r = \frac{H}{\tan(\beta)}$$

Where:
* $H$ = Depth of the coal seam below surface ($150\text{m}$ to $400\text{m}$).
* $\beta$ = Major angle of draw of the overlying strata (typically $60^\circ$ to $65^\circ \implies \tan\beta \approx 1.7\text{ to }2.1$).

For an overburden depth of $H = 150\text{m}$ and $\tan\beta = 2.0$:
$$r = \frac{150}{2.0} = \mathbf{75.0\text{ meters}}$$

```
+-----------------------------------------------------------------------------------+
|                        KNOTHE SUBSIDENCE TROUGH PROFILE                           |
|                                                                                   |
|  Surface Level ──────────────────────────┐             ┌─────────────────────────  |
|                                          \             /                           |
|                                           \   Trough  /                            |
|                                            \  Bottom /                             |
|                                             \_______/                              |
|  <── Inflection Zone ──>                                                           |
|       (Peak Strain)                                                                |
|  |←────────────────────────── r = 75 m ──────────────────────────→|                |
+-----------------------------------------------------------------------------------+
```

### The Spatial Nyquist Limit
To mathematically reconstruct the continuous surface curvature ($\partial^2 S / \partial x^2$) and capture the peak tensile strain inflection point without spatial aliasing, the spatial sampling grid ($\Delta$) must resolve the Gaussian bell-curve curvature:

$$\Delta \le \frac{r}{2.86}$$

Substituting $r = 75\text{ m}$:
$$\Delta \le \frac{75.0}{2.86} \approx \mathbf{26.2\text{ meters}} \implies \text{Adopted Grid: } \mathbf{15\text{ to }25\text{ meters}}$$

---

## 3. The Sampling Error Knee Curve (Deep Seams)

For deeper extraction panels, such as the Adriyala Longwall Project ($H = 375\text{m}$, $r = 187.5\text{m}$), spatial error analysis across synthetic and empirical datasets yields the following interpolation error knee:

| Grid Spacing ($\Delta$) | Maximum Spatial Interpolation Error | Relative Hardware Cost | Engineering Assessment |
| :--- | :--- | :--- | :--- |
| **100 meters** | $142\text{ mm}$ (Severe aliasing) | $1.0\times$ (Low) | Unacceptable; misses localized fissures |
| **60 meters** | $58\text{ mm}$ | $1.6\times$ | Marginal; poor inflection capture |
| **40 meters** | **$21\text{ mm}$ (Error Knee)** | **$2.4\times$** | **Optimal Engineering Knee** |
| **20 meters** | $18\text{ mm}$ | $4.8\times$ | Diminishing returns (doubles cost for 3mm gain) |

For deep seams ($H > 300\text{m}$), **$40\text{m}$ spacing** represents the optimal balance of spatial reconstruction accuracy ($21\text{ mm}$ error) and hardware cost, fitting precisely within the 37-scout baseline. For shallow panels ($H \le 150\text{m}$), the spacing contracts to **$15\text{–}25\text{m}$**.

---

## 4. What Sparse Grids Miss: Localized 2-Meter Shear Fissures

Conventional commercial deployments space loggers at $100\text{m}$ to $300\text{m}$ intervals due to equipment cost. 
* In stratified sedimentary overburden, sudden shearing occurs along joint planes and pre-existing faults.
* A $2\text{m}$ localized fissure opening up between two sparse stations placed $200\text{m}$ apart generates minimal tilt at the distant sensors.
* By enforcing physics-driven $\Delta \le 25\text{m}$ (or $40\text{m}$ for deep seams), AEGIS ensures that at least two Scout nodes straddle any emerging fracture zone, detecting anomalous differential extension immediately.
