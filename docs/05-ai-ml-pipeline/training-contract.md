# The PINN Training Contract & Identifiability Constraints

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [`loss-formulation.md`](loss-formulation.md) · [`safety-boundary.md`](safety-boundary.md) · [Module 08 Test Register](../08-verification/test-register.md)

---

## 1. Mathematical Identifiability & The 24-Hour Window

### Question: Why is training a surface reconstruction model on an instantaneous single-epoch telemetry snapshot mathematically flawed?

**Answer:** Training a model on an instantaneous single-time snapshot ($t = t_{\text{current}}$) represents a fatal identifiability failure in inverse problem theory.

Under Knothe's time-dependent formulation:
$$S(x, y, t) = a \cdot m_{\text{seam}} \cdot f_{\text{spatial}}(x, y) \cdot \left( 1 - e^{-c \cdot t} \right)$$

At any fixed single point in time $t_0$, the product of the subsidence coefficient and the time factor collapses into a single scalar value:
$$K(t_0) = a \cdot \left( 1 - e^{-c \cdot t_0} \right)$$

An optimization algorithm observing only one temporal snapshot can match $K(t_0)$ identically using an infinite continuum of conflicting $(a, c)$ parameter pairs:
* An unphysically high subsidence factor ($a = 0.95$) paired with an unphysically slow time decay ($c = 0.005\text{ day}^{-1}$).
* An unphysically low subsidence factor ($a = 0.35$) paired with an unphysically rapid time decay ($c = 0.045\text{ day}^{-1}$).

Because infinitely many curves intersect at a single point, **the time decay parameter $c$ is mathematically unidentifiable from a single temporal slice**. A model trained on a single epoch cannot forecast forward settlement trajectories ($\hat{S}(t + 72\text{h})$).

### Question: How does the 24-Hour Rolling Training Window resolve this parameter degeneracy (Test T36)?

**Answer:** AEGIS enforces a strict **24-Hour Rolling Training Window** containing **48 discrete half-hour time slices**:

$$t \in [t_{\text{current}} - 24\text{ hours}, \, t_{\text{current}}] \quad \text{sampled at } \Delta t = 30\text{ minutes}$$

```
                           24-HOUR ROLLING TRAINING WINDOW
   t - 24h                     t - 12h                     t - 6h                     t (now)
     │                           │                           │                           │
     ▼                           ▼                           ▼                           ▼
[ Slice 1 ] ─── [ Slice 2 ] ─── [ ... ] ─── [ Slice 24 ] ─── [ ... ] ─── [ Slice 47 ] ─── [ Slice 48 ]
  └─────────────────────────────── 48 Temporal Snapshots ──────────────────────────────┘
                                          │
                                          ▼
                Resolves Rate of Change: ∂Ŝ/∂t = c · [S_final - Ŝ]
                Breaks Parameter Degeneracy: Identifies â and ĉ Uniquely!
```

**The Mathematical Resolution:**  
Spanning 24 hours of active strata deformation provides measurable temporal rate-of-change ($\partial S / \partial t$) across all sensor stations. The Knothe differential equation:
$$\frac{\partial S(x, y, t)}{\partial t} = c \cdot S_{\text{final}}(x, y) - c \cdot S(x, y, t)$$
Now possesses independent temporal variations that decouple $a$ (which sets the asymptotic ceiling $S_{\text{final}} = a \cdot m_{\text{seam}}$) from $c$ (which sets the temporal gradient $\partial S / \partial t$).

Automated CI Test `T36` asserts that the training window spans $\ge 12.0\text{ hours}$ (enforcing 24 hours in production), guaranteeing unique mathematical identifiability.

---

## 2. Parameter Partitioning Contract

### Question: Which geotechnical parameters is the neural network permitted to optimize, and which parameters must remain strictly frozen?

**Answer:** To prevent the optimizer from converging to unphysical local minima, the training contract partitions parameters into immutable stratigraphic constants and dynamic learned variables:

| Parameter Symbol | Stratigraphic Dimension | Bounds / Nominal | Handling Contract | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **H** | Overburden Seam Depth | 150.0 meters | **FROZEN** | Measured directly from exploratory core boreholes. |
| **\tanβ** | Tangent of Angle of Draw | 2.0 (β ≈ 63.4°) | **FROZEN** | Calibrated from regional stratigraphy. |
| **r** | Knothe Radius of Influence | 75.0 meters (H / \tanβ) | **FROZEN** | Direct geometric derivation from H and \tanβ. |
| **B_horiz** | Awershin Displacement Ratio | 24.0 meters (0.32 · r) | **FROZEN** | Theoretical kinematic coupling ratio. |
| **m_seam** | Extracted Seam Thickness | 3.0 meters | **FROZEN** | Fixed longwall shearer drum cutting height. |
| **â** | Empirical Subsidence Factor | Bound: [0.40, 0.90] | **LEARNED** | Varies dynamically with roof caving and bulking ratio. |
| **ĉ** | Viscoelastic Time Decay | Bound: [0.005, 0.050] day⁻¹ | **LEARNED** | Varies dynamically with longwall face advance rate. |

The learned parameters $\hat{a}$ and $\hat{c}$ are instantiated as bounded PyTorch parameters passed through sigmoid activation bounds:
$$\hat{a} = a_{\text{min}} + (a_{\text{max}} - a_{\text{min}}) \cdot \sigma(w_a)$$
$$\hat{c} = c_{\text{min}} + (c_{\text{max}} - c_{\text{min}}) \cdot \sigma(w_c)$$
This guarantees that even under extreme noise, the parameters cannot diverge into unphysical negative or explosive regimes.

---

## 3. Retraining Cadence & Edge Execution Budget

### Question: What is the retraining cadence of the PINN, and what computational resources are required on edge server hardware?

**Answer:** Continuous edge retraining must respect the compute and thermal envelopes of industrial edge servers deployed at mine substations:

* **Execution Trigger:** Retraining is dispatched asynchronously:
  1. On a fixed **6-hour periodic cadence**; OR
  2. Immediately upon a confirmed **$20.0\text{-meter}$ advance** of the subterranean longwall face (reported by the SCADA shearer PLC).
* **Computational Footprint:**
  * **Dataset Size:** 48 time slices across active sensor stations $\approx 2,000$ to $5,000$ spatio-temporal telemetry points, plus $4,096$ PDE collocation points.
  * **Optimization Steps:** 1,000 Adam iterations followed by 200 L-BFGS iterations.
  * **Execution Runtime:** $\approx 45\text{ seconds}$ on an embedded GPU (NVIDIA Jetson AGX Orin) or $< 90\text{ seconds}$ on an Intel Xeon / Core i7 industrial edge CPU.
* **Non-Blocking Operation:** The retraining job executes in a detached background worker process. The production digital twin continues serving predictions using the current active weights until the new model passes convergence checks, after which a zero-downtime atomic weight swap occurs.

---

## 4. Quantitative Convergence Acceptance Thresholds (Test T33)

### Question: What are the statutory convergence criteria mandated by Test T33 at Day 40 of mining advance?

**Answer:** To ensure that the digital twin provides reliable engineering forecasts, the trained PINN must satisfy three quantitative criteria before its weights are accepted into production:

```
                               TEST T33 CONVERGENCE GATES
   ┌───────────────────────────────────┬───────────────────────────────────┐
   │ Metric Parameter                  │ Statutory Acceptance Threshold     │
   ├───────────────────────────────────┼───────────────────────────────────┤
   │ Combined Profile: â · η̂(t)        │ Within ±10.0% of Ground Truth     │
   │ Subsidence Coefficient: â         │ Within ±15.0% of Field Baseline   │
   │ Time Decay Constant: ĉ            │ Within ±25.0% of Settlement Curve │
   └───────────────────────────────────┴───────────────────────────────────┘
```

**Mathematical Verification at Day 40 ($t = 40\text{ days}$):**
1. **Dynamic Scaling:**
   $$\left| \frac{\hat{a} \cdot \left(1 - e^{-\hat{c} \cdot 40}\right) - a_{\text{true}} \cdot \left(1 - e^{-c_{\text{true}} \cdot 40}\right)}{a_{\text{true}} \cdot \left(1 - e^{-c_{\text{true}} \cdot 40}\right)} \right| \le 0.10$$
2. **Subsidence Factor Recovery:**
   $$\left| \frac{\hat{a} - 0.65}{0.65} \right| \le 0.15 \implies 0.5525 \le \hat{a} \le 0.7475$$
3. **Rheological Decay Recovery:**
   $$\left| \frac{\hat{c} - 0.01414}{0.01414} \right| \le 0.25 \implies 0.0106 \le \hat{c} \le 0.01768\text{ day}^{-1}$$

If an optimization run fails any of these three gates, the updated weights are rejected, the existing validated weights are retained, and a diagnostic notification is dispatched to the geotechnical engineering console.
