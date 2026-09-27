# Composite Loss Formulation & Boundary Constraints

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [`training-contract.md`](training-contract.md) · [Module 08 Test T34](../08-verification/test-register.md)

---

## 1. Multi-Objective Objective Function

The C9 PINN optimizer minimizes a multi-objective loss function balancing observational data alignment, differential physics compliance, and reference anchor boundary locks:

$$\mathcal{L}_{\text{total}} = w_{\text{data}} \mathcal{L}_{\text{data}} + w_{\text{PDE}} \mathcal{L}_{\text{PDE}} + w_{\text{anchor}} \mathcal{L}_{\text{anchor}} + w_{\text{bound}} \mathcal{L}_{\text{bound}}$$

```
                                  COMPOSITE PINN LOSS
                 ┌─────────────────────────┼─────────────────────────┐
                 ▼                         ▼                         ▼
         [ DATA-FIT LOSS ]         [ PDE RESIDUAL LOSS ]     [ ANCHOR LOCK LOSS ]
         - Provenance weighted     - Knothe Differential Eq  - Zero displacement lock
         - Real: 3.0×, Sim: 0.1×   - Penalizes unphysical S  - Prevents DC grid drift
```

---

## 2. Loss Term Derivations

### 1. Provenance-Weighted Data Loss ($\mathcal{L}_{\text{data}}$)
Evaluates misfit between predicted subsidence $\hat{S}_i$ and observed field sensor readings $S_i^{\text{obs}}$, weighted by observational provenance:

$$\mathcal{L}_{\text{data}} = \frac{1}{N} \sum_{i=1}^N \omega_{\text{prov}}(i) \cdot \left| \hat{S}(x_i, y_i, t_i) - S_i^{\text{obs}} \right|^2$$

Where provenance weights $\omega_{\text{prov}}$ are assigned per Gate G04:
* $\omega = 3.0$ for `real` field survey and leveling peg observations.
* $\omega = 1.0$ for `pinned` empirical calibration points.
* $\omega = 0.1$ for `synthetic` noise-injected simulation readings.

### 2. Knothe Partial Differential Equation Loss ($\mathcal{L}_{\text{PDE}}$)
Knothe's time-dependent subsidence satisfies the linear first-order differential relaxation equation:

$$\frac{\partial S(x, y, t)}{\partial t} = c \cdot \left[ S_{\text{final}}(x, y) - S(x, y, t) \right]$$

The physics residual $R_{\text{Knothe}}$ is evaluated via automatic differentiation ($\partial \hat{S} / \partial t$) at collocation points across the panel:

$$\mathcal{L}_{\text{PDE}} = \frac{1}{M} \sum_{j=1}^M \left| \frac{\partial \hat{S}_j}{\partial t} + \hat{c} \cdot \hat{S}_j - \hat{c} \cdot S_{\text{final}}(x_j, y_j) \right|^2$$

This term forces the neural network to satisfy the physics of strata relaxation, preventing discontinuous spatial jumps between sparse sensor locations.

### 3. Bedrock Anchor Boundary Lock ($\mathcal{L}_{\text{anchor}}$ — Test T34)
Reference anchors sit outside the extraction influence basin ($x > x_2 + r$), where physical subsidence must remain zero:

$$\mathcal{L}_{\text{anchor}} = \frac{1}{K} \sum_{k=1}^K \left| \hat{S}(x_{\text{anchor}, k}, y_{\text{anchor}, k}, t) \right|^2$$

> [!WARNING]
> **The Critical Role of Anchor Loss (Test T34):**
> Verification tests demonstrate that when $\mathcal{L}_{\text{anchor}}$ is disabled, the neural network's unconstrained bias weights cause the entire global elevation grid to drift upward or downward by over **10 cm** ($> 100\text{ mm}$ systematic offset), rendering tilt derivatives meaningless. Locking the anchors to $|S| < 1\text{ mm}$ establishes a fixed absolute elevation datum.

### 4. Asymptotic Boundary Decay ($\mathcal{L}_{\text{bound}}$)
Penalizes any predicted deformation extending beyond the theoretical influence boundary:
$$\mathcal{L}_{\text{bound}} = \int_{\Omega_{\text{far}}} \left| \hat{S}(x, y, t) \right|^2 \, dx\, dy \quad \text{for } \text{dist}(x, y, \text{panel}) > 2r$$

---

## 3. Loss Balancing & Weight Schedules

To prevent gradient starvation during early training iterations, loss weights are statically scheduled:
* $w_{\text{data}} = 1.0$
* $w_{\text{PDE}} = 0.5$
* $w_{\text{anchor}} = 2.0$ (High priority: datum integrity is non-negotiable)
* $w_{\text{bound}} = 0.2$
