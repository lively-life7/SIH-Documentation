# Composite Loss Formulation & Boundary Constraints

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [`training-contract.md`](training-contract.md) · [`safety-boundary.md`](safety-boundary.md) · [Module 08 Test Register](../08-verification/test-register.md)

---

## 1. Multi-Objective Optimization Formulation

### Question: What is the formal mathematical formulation of the composite loss function governing the C9 PINN, and how does it prevent overfitting to sparse sensor points?

**Answer:** Standard neural networks minimize empirical risk (mean squared error on observed sensor points), which leads to unphysical spatial interpolation artifacts and non-physical deformation between sparse sensor locations.

The C9 Physics-Informed Neural Network (PINN) resolves this ill-posed inverse problem by minimizing a **multi-objective loss function** that couples observational data misfit, governing differential strata mechanics, and rigid boundary anchor locks:

$$\mathcal{L}_{\text{total}} = w_{\text{data}} \mathcal{L}_{\text{data}} + w_{\text{PDE}} \mathcal{L}_{\text{PDE}} + w_{\text{anchor}} \mathcal{L}_{\text{anchor}} + w_{\text{bound}} \mathcal{L}_{\text{bound}}$$

```
                                  COMPOSITE PINN LOSS
                 ┌─────────────────────────┼─────────────────────────┐
                 ▼                         ▼                         ▼
         [ DATA-FIT LOSS ]         [ PDE RESIDUAL LOSS ]     [ ANCHOR LOCK LOSS ]
         - Provenance weighted     - Knothe Differential Eq  - Zero displacement lock
         - Real: 3.0×, Sim: 0.1×   - Penalizes unphysical S  - Prevents DC grid drift
```

Where the loss weights balance the relative gradient magnitudes:
* $w_{\text{data}} = 1.0$ (Primary observational alignment)
* $w_{\text{PDE}} = 0.5$ (Differential strata physics regularizer)
* $w_{\text{anchor}} = 2.0$ (Non-negotiable absolute elevation datum lock)
* $w_{\text{bound}} = 0.2$ (Far-field asymptotic suppression)

---

## 2. Loss Term Mathematical Derivations

### Question: How is the observational data loss ($\mathcal{L}_{\text{data}}$) formulated with provenance weighting, and why is uniform Mean Squared Error (MSE) inadequate for mining telemetry?

**Answer:** Mining field datasets combine heterogeneous observation channels: high-precision optical leveling surveys, real-time wireless mesh sensor telemetry, and synthetic noise-injected edge cases. A naive uniform MSE treats all observations as equally authoritative, causing the network to learn synthetic noise artifacts or transient packet loss dropouts as true geological deformation.

AEGIS implements a **Provenance-Weighted Data Loss**:

$$\mathcal{L}_{\text{data}} = \frac{1}{N} \sum_{i=1}^N \omega_{\text{prov}}(i) \cdot \left| \hat{S}(x_i, y_i, t_i) - S_i^{\text{obs}} \right|^2$$

Where the scalar weight $\omega_{\text{prov}}(i)$ is assigned dynamically per Gate G04:
* $\omega_{\text{prov}} = 3.0$ for `real` field observations (historical leveling surveys, calibrated in-situ sensors, Sentinel-1 InSAR benchmarks).
* $\omega_{\text{prov}} = 1.0$ for `pinned` empirical calibration points (baseline elevation grids fitted to verified core draw angles).
* $\omega_{\text{prov}} = 0.1$ for `synthetic` simulation points (stress-test edge cases, battery sag injections, and simulated packet drops).

Penalizing synthetic telemetry by $10\times$ ensures that the network fits physical field data with highest fidelity while using synthetic points strictly for structural regularization.

### Question: What is the governing Knothe Partial Differential Equation (PDE), and how is the differential residual loss ($\mathcal{L}_{\text{PDE}}$) computed via automatic differentiation?

**Answer:** Strata relaxation above an extracted seam is governed by Knothe's time-rate differential equation, which states that vertical subsidence velocity is proportional to the remaining distance to asymptotic final subsidence:

$$\frac{\partial S(x, y, t)}{\partial t} = c \cdot \left[ S_{\text{final}}(x, y) - S(x, y, t) \right]$$

Rearranging this into a differential residual operator:
$$\mathcal{R}_{\text{Knothe}}[S] \equiv \frac{\partial S}{\partial t} + c \cdot S - c \cdot S_{\text{final}}(x, y) = 0$$

During PINN training, the network output $\hat{S}(x, y, t)$ and dynamic learned parameter $\hat{c}$ are evaluated across $M$ spatio-temporal collocation points distributed throughout the domain:

$$\mathcal{L}_{\text{PDE}} = \frac{1}{M} \sum_{j=1}^M \left| \left.\frac{\partial \hat{S}}{\partial t}\right|_{(x_j, y_j, t_j)} + \hat{c} \cdot \hat{S}(x_j, y_j, t_j) - \hat{c} \cdot S_{\text{final}}(x_j, y_j; \, \hat{a}) \right|^2$$

The temporal derivative $\partial \hat{S} / \partial t$ is computed analytically via reverse-mode **automatic differentiation (autodiff)** through the PyTorch computational graph. Collocation points require no physical sensors; they sample arbitrary coordinates across the extraction basin, forcing the neural network to satisfy the physics of continuum strata relaxation everywhere.

### Question: What is the Bedrock Anchor Boundary Lock ($\mathcal{L}_{\text{anchor}}$), and why does disabling it cause a catastrophic $> 10\text{ cm}$ elevation datum drift (Test T34)?

**Answer:** Reference bedrock anchor nodes are physically positioned outside the extraction influence basin ($x > x_2 + 2r$), anchored deep into stable geological formations where physical subsidence must remain identically zero across all epochs.

The Bedrock Anchor Loss enforces this physical boundary condition:

$$\mathcal{L}_{\text{anchor}} = \frac{1}{K} \sum_{k=1}^K \left| \hat{S}(x_{\text{anchor}, k}, y_{\text{anchor}, k}, t) \right|^2$$

**The Catastrophic Failure Mode of Disabling Anchor Loss (Test T34):**  
In an unconstrained neural network, the output layer bias parameter $b_{\text{out}}$ can shift freely without violating the PDE residual loss (since $\partial(S + \text{const}) / \partial t = \partial S / \partial t$). Automated CI Test `T34` demonstrates that when $\mathcal{L}_{\text{anchor}}$ is omitted, the network experiences **systematic DC datum drift exceeding $10\text{ cm}$ ($> 100\text{ mm}$)** across the entire terrain grid.

Because civil infrastructure decisions depend on absolute elevation benchmarks, and spatial tilt derivatives become distorted near the boundary, locking the bedrock anchors to $|\hat{S}| < 1.0\text{ mm}$ with a high loss penalty ($w_{\text{anchor}} = 2.0$) establishes an unyielding absolute coordinate datum.

### Question: How is the Asymptotic Boundary Decay Loss ($\mathcal{L}_{\text{bound}}$) formulated to prevent unphysical far-field deformation?

**Answer:** Mining subsidence basins do not propagate infinitely. Physical deformation decays to negligible levels beyond the radius of influence $r$. Outside the far-field boundary domain $\Omega_{\text{far}} = \{ (x, y) \mid \text{dist}((x, y), \text{Panel}) > 2r \}$, vertical settlement must vanish.

The far-field loss integrates the squared predicted settlement over the exterior domain:

$$\mathcal{L}_{\text{bound}} = \frac{1}{|\Omega_{\text{far}}|} \iint_{\Omega_{\text{far}}} \left| \hat{S}(x, y, t) \right|^2 \, dx\, dy$$

In discrete computation, this integral is approximated over $P$ far-field boundary anchor points:
$$\mathcal{L}_{\text{bound}} = \frac{1}{P} \sum_{p=1}^P \left| \hat{S}(x_p, y_p, t) \right|^2$$

This prevents the neural network from hallucinating non-physical "flapping" or spontaneous ground upheavals in unmonitored periphery zones.

---

## 3. Two-Stage Optimization: Adam to L-BFGS Transition

### Question: What optimization strategy is used to train the composite PINN loss, and why is standard stochastic gradient descent (Adam) followed by L-BFGS quasi-Newton optimization?

**Answer:** Training physics-informed neural networks with stiff multi-objective loss landscapes is notoriously susceptible to local minima, gradient pathologies, and slow asymptotic convergence when using first-order optimizers alone.

AEGIS implements a rigorous **Two-Stage Hybrid Optimization Pipeline**:

```
[ Stage 1: Adam Global Search ] ────────► [ Stage 2: L-BFGS Fine-Tuning ] ────────► [ Convergence ]
  - 1,000 to 2,000 iterations               - Second-order Quasi-Newton (Hessian)     - Residual LOO < 25 mm
  - Learning rate η = 1e-3                  - Line search (Armijo-Goldstein)          - â within ±15%
  - Escapes rugged local minima             - Rapid quadratic convergence near basin  - ĉ within ±25%
```

1. **Stage 1 — Adam First-Order Warm-Up (Global Exploration):**  
   The network is trained for $1,000$ to $2,000$ iterations using the Adam optimizer with an initial learning rate $\gamma = 1 \times 10^{-3}$ and cosine annealing decay. Adam's momentum terms ($\beta_1 = 0.9, \beta_2 = 0.999$) allow the optimizer to traverse the rugged, non-convex multi-objective landscape, establishing an approximate physical basin without getting trapped in early high-frequency local minima.

2. **Stage 2 — L-BFGS Second-Order Convergence (Local Exploitation):**  
   Once the total loss plateau is reached ($\Delta \mathcal{L} / \mathcal{L} < 10^{-4}$), optimization switches to the **Limited-memory Broyden-Fletcher-Goldfarb-Shanno (L-BFGS)** algorithm with strong Wolfe line search. L-BFGS approximates the inverse Hessian matrix ($\mathbf{H}^{-1}$) using curvature information from recent gradient history:
   $$\mathbf{x}_{k+1} = \mathbf{x}_k - \alpha_k \mathbf{H}_k^{-1} \nabla \mathcal{L}_{\text{total}}(\mathbf{x}_k)$$
   This second-order transition yields rapid super-linear (quadratic) convergence, driving the PDE physics residual $\mathcal{L}_{\text{PDE}}$ and anchor datum errors to near-machine precision ($\mathcal{L}_{\text{anchor}} < 10^{-6}$), which first-order methods cannot achieve within reasonable time bounds.
