# 3D Continuous Surface Reconstruction & Spatial Validation

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [`loss-formulation.md`](loss-formulation.md) · [Module 03 Grid Spacing](../03-mesh-networking/grid-spacing-rationale.md) · [Module 07 3D Visualization](../07-digital-twin-and-ui/3d-visualization.md) · [Module 08 Test Register](../08-verification/test-register.md)

---

## 1. Sparse-to-Dense Continuum Reconstruction

### Question: How does AEGIS reconstruct a continuous 3D digital twin of surface terrain from a discrete, spatially distributed sensor network?

**Answer:** Physical geotechnical monitoring networks are inherently discrete: instrumentation exists only at specific coordinate locations $(x_i, y_i)$. However, mining structural engineers, railway authorities, and DGMS inspectors require a continuous, gap-free model of terrain deformation to evaluate risks to continuous linear infrastructure (such as conveyor corridors, high-pressure pipelines, haul roads, and railway sidings).

Standard polynomial or spline interpolation fails across large sensor gaps, producing unphysical oscillations (Runge's phenomenon). AEGIS utilizes the trained C9 Physics-Informed Neural Network (PINN) as a **continuous spatio-temporal coordinate operator**:

```
[Discrete Telemetry: Dynamic Spatial Grid Scaled by Knothe Radius r]
                                │
                                ▼  Forward Inference through Trained PINN
       [Continuous Coordinate Evaluation: Ŝ(x, y, t) = N(x, y, t; W, b)]
                                │
                                ▼  Evaluated on 64 × 64 Regular Mesh
       [High-Resolution 3D Surface Mesh & Deformation Vector Field]
                                │
                                ▼  Streamed via Binary WebSocket to WebGL HMI
       [CesiumJS / Three.js SCADA Mission Control Digital Twin]
```

Because the neural network is regularized by Knothe strata mechanics and bedrock anchor locks, querying the network across arbitrary $(x, y)$ coordinates produces smooth, volume-conserving subsidence profiles that naturally bridge unmonitored terrain blind spots.

### Question: What are the spatial dimensions, grid resolution, and kinematic channels generated for the 3D SCADA digital twin?

**Answer:** The continuous coordinate field is evaluated across a uniform high-resolution grid that scales dynamically with the panel geometry:
* **Spatial Domain:** Encompasses the active extraction panel plus a $2.0 \cdot r$ buffer zone on all sides ($x \in [x_{\text{min}}, x_{\text{max}}]$, $y \in [y_{\text{min}}, y_{\text{max}}]$). For a standard $600\text{m} \times 200\text{m}$ extraction panel at $H = 150\text{m}$ ($r = 75\text{m}$), the domain spans $x \in [25\text{m}, 775\text{m}]$ and $y \in [25\text{m}, 375\text{m}]$.
* **Mesh Resolution:** Discretized into a regular $64 \times 64$ evaluation matrix ($4,096$ surface vertices).
* **Emitted Kinematic Attributes per Vertex:**
  1. Vertical Subsidence: $\hat{S}(x, y, t)$ (meters).
  2. Ground Tilt Magnitude: $|\hat{\mathbf{T}}| = \sqrt{(\partial \hat{S}/\partial x)^2 + (\partial \hat{S}/\partial y)^2}$ ($\text{mm/m}$ or $\mu\text{rad}$).
  3. Principal Horizontal Tensile Strain: $\hat{\varepsilon}_{\text{max}}(x, y, t) = B_{\text{horiz}} \cdot \max(\text{eig}(\nabla^2 \hat{S}))$ ($\mu\varepsilon$).
  4. Local Spatial Uncertainty: $\sigma_{\text{spatial}}(x, y)$ derived from distance to nearest physical sensor station.

This matrix is serialized as a compact binary Float32 array and pushed via WebSocket to the frontend 3D WebGL renderer (CesiumJS / Three.js) at sub-second refresh rates.

---

## 2. In-Situ Model Evaluation: Leave-One-Out Cross-Validation

### Question: How can the accuracy of the neural surface reconstruction be objectively validated in an active mine where no continuous "ground-truth answer key" exists?

**Answer:** In active mining deployments, absolute surface truth between sensor stations is unknown. Evaluating an AI model solely on its training loss produces misleadingly optimistic results due to overfitting.

AEGIS implements an automated **Leave-One-Out (LOO) Residual Cross-Validation Scheme**:
1. During scheduled validation cycles, the system systematically holds out sensor station $k$ ($k \in \{1, \dots, N\}$) from the training dataset.
2. The PINN is retrained or fine-tuned using only the remaining $N-1$ sensor stations.
3. The held-out station's physical reading $S_k^{\text{obs}}$ is compared against the model's blind spatial prediction $\hat{S}_{-k}(x_k, y_k, t)$:
   $$\text{Residual}_k = \left| \hat{S}_{-k}(x_k, y_k, t) - S_k^{\text{obs}} \right|$$
4. **The Global LOO Residual Metric:**
   $$\text{Residual}_{\text{LOO}} = \sqrt{\frac{1}{N} \sum_{k=1}^N \left| \hat{S}_{-k}(x_k, y_k, t) - S_k^{\text{obs}} \right|^2}$$

**Operational Acceptance Criteria:**
* If $\text{Residual}_{\text{LOO}} \le 25.0\text{ mm}$, the digital twin is flagged as **VALIDATED** (empirical strata movement matches continuum mechanics).
* If $\text{Residual}_{\text{LOO}} > 25.0\text{ mm}$, the SCADA interface flags an **ANOMALOUS STRATA ALERT**, notifying geotechnical engineers that local geologic discontinuities (e.g., hidden subsurface faults, unmapped palaeochannels, or voids) are causing anomalous deformation that deviates from theoretical continuum physics.

---

## 3. Spatial Resolution & The Largest Empty Circle Metric (Test T20)

### Question: What is the Largest Empty Circle (LEC) metric, and how is the committed spatial resolution (d_committed) derived for a physical sensor deployment (Test T20)?

**Answer:** A common and dangerous claim in commercial IoT monitoring is "100% blind-spot-free coverage." In physical reality, any discrete sensor array has spatial gaps between sensor locations. If a localized sinkhole or crown hole develops entirely within an empty pocket between sensors, it will remain undetected until it expands.

AEGIS resolves this through the **Largest Empty Circle (LEC)** formulation:
1. **Geometric Definition:** Given the planar coordinates of all active sensor stations $\mathbf{P} = \{(x_1, y_1), \dots, (x_N, y_N)\}$, the Largest Empty Circle is the largest circle $C(\mathbf{x}_c, R_{\text{LEC}})$ that can be inscribed inside the active panel boundary such that its interior contains zero sensor nodes:
   $$R_{\text{LEC}} = \max_{\mathbf{x} \in \Omega_{\text{basin}}} \min_{i=1,\dots,N} \|\mathbf{x} - \mathbf{p}_i\|$$
2. **Computational Derivation:** The center of the Largest Empty Circle corresponds to a vertex of the Voronoi diagram (or circumcenter of the Delaunay triangulation) bounded by the mining influence perimeter.
3. **Committed Spatial Resolution ($d_{\text{committed}}$):**
   $$d_{\text{committed}} = 2 \cdot R_{\text{LEC}}$$

For the baseline reference panel deployment geometry, the geometric algorithm computes:
$$d_{\text{committed}} = \mathbf{358.0\text{ meters}}$$

```
+-----------------------------------------------------------------------------------+
|                        LARGEST EMPTY CIRCLE (LEC) GEOMETRY                        |
|                                                                                   |
|           [Node A]                                     [Node B]                   |
|              *                                            *                       |
|               \                                          /                        |
|                \        ┌──────────────────────┐        /                         |
|                 \       │                      │       /                          |
|                  \      │    EMPTY CIRCLE      │      /                           |
|                   \     │                      │     /                            |
|                    \    │  2·R_LEC = 358m      │    /                             |
|                     \   │                      │   /                              |
|                      \  └──────────────────────┘  /                               |
|                       \                          /                                |
|                        *                        *                                 |
|                     [Node C]                 [Node D]                             |
+-----------------------------------------------------------------------------------+
```

### Question: Why does AEGIS mandate publishing d_committed on the SCADA dashboard header, and what is its regulatory significance (Gate G15)?

**Answer:** Regulators (such as DGMS) and mine general managers must know the precise physical limits of automated safety systems. Hiding sensor gaps behind smooth visual interpolations creates false confidence that can lead to fatal accidents.

Under Gate G15 and automated CI Test `T20`, the committed spatial resolution is prominently displayed in the header banner of the AEGIS SCADA dashboard:
> **"System Spatial Resolution: Continuous features larger than $358\text{m}$ are 100% guaranteed to be detected. Localized fissures or sinkholes smaller than $358\text{m}$ occurring entirely within empty grid pockets carry statistical probability bounds."**

Publishing this metric demonstrates uncompromising scientific honesty. It clarifies that while the PINN provides mathematically optimal interpolation, physical sensor density dictates the ultimate spatial resolution of deterministic detection.

---

## 4. Dynamic Spatial Scaling Across Arbitrary Panels

### Question: How does the surface reconstruction pipeline adapt dynamically to different mine layouts rather than relying on a static node count?

**Answer:** The reconstruction engine operates independently of static node counts. When deployed on a new panel:
1. The mesh coordinate boundaries are computed from the panel dimensions:
   $$x \in [x_1 - 2r, \, x_2 + 2r], \quad y \in [y_1 - 2r, \, y_2 + 2r]$$
2. Ingested telemetry is mapped directly from the dynamic configuration manifest (`nodes.json`), supporting arbitrary node counts, variable spacing, and irregular cluster geometries.
3. Delaunay triangulation recomputes $R_{\text{LEC}}$ and $d_{\text{committed}}$ dynamically upon network reconfiguration, updating the SCADA transparency header automatically.
