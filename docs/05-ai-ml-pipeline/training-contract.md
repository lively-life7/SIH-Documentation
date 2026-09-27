# The PINN Training Contract & Identifiability Constraints

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [`loss-formulation.md`](loss-formulation.md) · [Module 08 Test T36](../08-verification/test-register.md)

---

## 1. The 24-Hour Training Window (Test T36)

A critical architectural flaw in early prototypes was training the surface reconstruction model on a single instantaneous telemetry snapshot ($t = t_{\text{current}}$).

### The Mathematical Identifiability Breakdown
Under Knothe's model:
$$S(x, y, t) = a \cdot m_{\text{seam}} \cdot f(x, y) \cdot (1 - e^{-c \cdot t})$$
* At any fixed instant in time $t_0$, the product $a \cdot (1 - e^{-c \cdot t_0})$ evaluates to a single scalar constant $K$.
* An optimization algorithm observing only one time slice can fit $K$ perfectly using an infinite number of conflicting $(a, c)$ parameter pairs (e.g., high subsidence factor $a$ with slow decay $c$, or low $a$ with fast $c$).
* The time decay constant $c$ is **mathematically unidentifiable from a single time snapshot**.

### The Contractual Fix: 48 Half-Hour Slices (Test T36)
AEGIS enforces a strict **24-Hour Rolling Training Window**:
* Telemetry is sampled across 48 discrete 30-minute time slices ($t \in [t - 24\text{h}, \, t]$).
* Because the time dimension spans 24 hours of active strata movement, the derivative $\partial S / \partial t$ can be resolved numerically, breaking the parameter degeneracy and making $\hat{c}$ uniquely identifiable (verified by test `T36`).

---

## 2. Parameter Partitioning Matrix

The training contract strictly specifies which parameters the neural network is permitted to optimize and which parameters remain frozen:

| Parameter | Type | Value / Bounds | Handling |
| :--- | :--- | :--- | :--- |
| **Seam Depth ($H$)** | Ground Truth | Measured borehole depth (e.g., $150.0\text{m}$) | **FROZEN** (Constant) |
| **Draw Angle ($\tan\beta$)** | Stratigraphy | Calibrated from core samples ($2.0$) | **FROZEN** (Constant) |
| **Influence Radius ($r$)** | Kinematic | Calculated analytically ($H / \tan\beta = 75.0\text{m}$) | **FROZEN** (Constant) |
| **Horizontal Ratio ($B_{\text{horiz}}$)** | Mechanics | Awershin ratio ($0.32 \cdot r = 24.0\text{m}$) | **FROZEN** (Constant) |
| **Subsidence Factor ($\hat{a}$)** | Geological | Bound: $[0.40, \, 0.90]$ | **LEARNED by PINN** |
| **Time Decay ($\hat{c}$)** | Rheological | Bound: $[0.005, \, 0.050]\text{ day}^{-1}$ | **LEARNED by PINN** |

---

## 3. Retraining Cadence & Execution Budget

* **Retraining Interval:** Executed asynchronously every **6 hours** or immediately following a confirmed $20\text{ meter}$ advance of the longwall face.
* **Compute Footprint:** Training across 48 time slices takes $\sim 45\text{ seconds}$ on a modest edge server GPU (NVIDIA Jetson Orin or Intel Core i7 host CPU), consuming negligible background resources.
* **Convergence Threshold (Test T33):**
  At Day 40 of mining advance, the training contract asserts:
  * Combined profile $\hat{a} \cdot \hat{\eta}(t)$ within $\pm 10\%$ of empirical ground truth.
  * Subsidence factor $\hat{a}$ within $\pm 15\%$ of core survey baseline.
  * Time decay constant $\hat{c}$ within $\pm 25\%$ of long-term settlement curves.
