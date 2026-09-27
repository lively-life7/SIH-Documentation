# 3D SCADA Digital Twin & Terrain Visualization

**Module 07 — Digital Twin & UI**  
**Cross-References:** [`operator-dashboard.md`](operator-dashboard.md) · [Module 05 Surface Reconstruction](../05-ai-ml-pipeline/surface-reconstruction.md)

---

## 1. 3D Digital Twin Engine (Three.js & CesiumJS)

The AEGIS Mission Control interface renders an interactive 3D digital twin of the mine surface using **React 19, Three.js (`@react-three/fiber`), and CesiumJS geospatial tiles**. 

The digital twin transforms numerical sensor streams into an intuitive, real-time spatial environment that mine managers, geologists, and civil authorities can inspect from any angle.

```
+-----------------------------------------------------------------------------------+
|                     3D MISSION CONTROL TERRAIN STACK                              |
|                                                                                   |
|  [ Layer 4: Infrastructure Vector Overlays ] (Railways, Pipelines, Power Pylons)  |
|  [ Layer 3: Dynamic Exclusion Rings ]        (1.2R Advisory & 1.5R Danger Zones)  |
|  [ Layer 2: PINN Continuous Deformation ]    (Color-coded Heatmap: mm/day)        |
|  [ Layer 1: Base Topographic Elevation ]     (30m CartoDEM / Satellite Imagery)   |
+-----------------------------------------------------------------------------------+
```

---

## 2. Dynamic Safety Exclusion Zones ($1.2R$ and $1.5R$)

Under mining subsidence mechanics, surface danger does not stop at the edge of the extracted panel. Displacements radiate outward by the angle of draw, defined by the Radius of Influence ($r = H / \tan\beta$).

The digital twin dynamically computes and projects two concentric exclusion perimeters centered over the advancing extraction face:

1. **Outer Advisory Perimeter ($1.2 \cdot r$):**
   * *Threshold Condition:* Anticipated ground tilt $> 2\text{ mm/m}$ or horizontal strain $> 200\ \mu\varepsilon$.
   * *Protocol:* Alert posted to infrastructure maintenance teams (Indian Railways, highway engineers). Speed restrictions enforced on rail links.
2. **Inner Danger Exclusion Zone ($1.5 \cdot r$):**
   * *Threshold Condition:* Active ground curvature exceeding tensile cracking limits ($\varepsilon > 1000\ \mu\varepsilon$).
   * *Protocol:* Total surface access restriction. Unmanned monitoring only. Power utilities alerted for overhead transmission cable tensioning.

---

## 3. High-Frequency Real-Time Shaders

To maintain smooth 60 FPS performance on control room workstations without GPU throttling:
* The $64 \times 64$ elevation deformation grid is passed directly to the GPU as a float displacement texture map.
* A custom WebGL vertex shader perturbs the base topographic terrain mesh in real time.
* Color fragment shaders visualize strain concentration using an intuitive color ramp:
  * **Blue / Green:** Elastic micro-strain ($< 500\ \mu\varepsilon$, structurally stable).
  * **Yellow / Orange:** Plastic deformation zone ($500\text{ to }1500\ \mu\varepsilon$, advisory monitoring).
  * **Crimson / Violet:** Critical tensile rupture zone ($> 1500\ \mu\varepsilon$, active cracking).
