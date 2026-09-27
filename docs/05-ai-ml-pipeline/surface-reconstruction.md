# 3D Continuous Surface Reconstruction & Spatial Validation

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [Module 03 Grid Spacing](../03-mesh-networking/grid-spacing-rationale.md) · [Module 07 3D Visualization](../07-digital-twin-and-ui/3d-visualization.md)

---

## 1. Sparse-to-Dense Continuum Reconstruction

Field monitoring networks are inherently discrete: sensors exist only at specific physical coordinates $(x_i, y_i)$. However, structural engineers and safety planners require a continuous 3D digital twin of the terrain to evaluate risks to infrastructure spanning continuous linear corridors (e.g., railway lines, pipelines, and drainage channels).

```
[Discrete Field Telemetry: 37 Nodes]
               │
               ▼  Evaluated through C9 PINN Inference Graph
[Continuous Coordinate Evaluation: N(x, y, t)]
               │
               ▼  Interpolated across 64 × 64 Regular Terrain Matrix
[High-Resolution 3D Surface Mesh & Deformation Heatmap]
               │
               ▼  Streamed via WebSocket / JSON to 3D HMI
[CesiumJS / Three.js SCADA Mission Control Digital Twin]
```

### The Output Grid
* **Resolution:** $64 \times 64$ regular grid spanning $x \in [25\text{m}, 775\text{m}]$ and $y \in [25\text{m}, 375\text{m}]$ ($4,096$ surface evaluation nodes).
* **Metrics Emitted Per Grid Vertex:** Vertical displacement $\hat{S}$, slope gradient magnitude $|\nabla \hat{S}|$, and principal horizontal tensile strain $\hat{\varepsilon}_{\text{max}}$.

---

## 2. Leave-One-Out (LOO) Residual Cross-Validation

Evaluating AI model accuracy in a live mining environment is challenging because real-world "answer keys" do not exist between sensor stations.

AEGIS implements **Leave-One-Out (LOO) Cross-Validation**:
1. During validation passes, the system systematically masks node $k$ from the training set.
2. The PINN reconstructs the surface using only the remaining $N-1$ sensor stations.
3. The model's predicted displacement $\hat{S}(x_k, y_k)$ is evaluated against the held-out sensor's physical measurement $S_k^{\text{obs}}$.
4. **LOO Residual Metric:**
   $$\text{Residual}_{\text{LOO}} = \sqrt{\frac{1}{N} \sum_{k=1}^N \left| \hat{S}_{-k}(x_k, y_k) - S_k^{\text{obs}} \right|^2}$$
5. If $\text{Residual}_{\text{LOO}}$ exceeds $25\text{ mm}$, the operator dashboard displays an uncertainty flag, indicating that local strata behavior is deviating from theoretical models.

---

## 3. The Largest Empty Circle (LEC) Metric & $d_{\text{committed}}$ (Test T20)

A common marketing claim in mine monitoring is "100% blind-spot-free coverage." In physical reality, any discrete sensor array has blind spots between nodes.

AEGIS addresses this through the **Largest Empty Circle (LEC)** metric:
* **Definition:** The largest circle that can be inscribed between sensor locations within the active subsidence basin without containing a single node.
* **Detection Limit ($d_{\text{committed}} = 358\text{ m}$, Test T20):**
  The diameter of the largest empty circle across the baseline layout is calculated geometrically as $358\text{ meters}$.
* **Statutory Transparency:**
  Per Gate G15 and Test `T20`, the value of $d_{\text{committed}}$ is prominently displayed in the SCADA dashboard header:
  > *"System Spatial Resolution: Features larger than $358\text{m}$ are 100% guaranteed to be detected. Localized fissures smaller than $358\text{m}$ occurring entirely within empty grid pockets carry statistical probability bounds."*

Publishing the mathematical blind spot demonstrates engineering honesty before regulatory safety bodies and DGMS inspectors.
