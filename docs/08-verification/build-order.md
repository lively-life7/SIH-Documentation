# The 6-Day Staged Build Order & Freezing Protocol

**Module 08 — Verification**  
**Cross-References:** [`test-register.md`](test-register.md) · [`field-validation-plan.md`](field-validation-plan.md) · [Module 00 Project Charter](../00-executive-gateway/project-charter.md)

---

### Question: Why does AEGIS enforce an immutable 6-day phased build order rather than standard continuous integration or monolithic development?

**Answer:** Monolithic development in cyber-physical geotechnical systems creates catastrophic circular debugging loops. If analytical ground truth generators, physical corruption chains, mesh protocols, and machine learning models are developed concurrently, a failure in surface reconstruction cannot be isolated—engineers cannot determine whether the fault originated in a numerical discretization error in Knothe physics, an uncompensated thermal drift in the C7 calibration layer, a dropped packet in the TDMA superframe, or an architectural bug in the neural network.

To guarantee mathematical determinism, AEGIS enforces a strict sequential 6-day build order governed by formal automated gates (G01–G10). A downstream subsystem cannot be compiled, simulated, or trained until all preceding foundational layers have achieved 100% test passage in automated continuous integration:

```
[Day 1: Ground Constants & Manifest] ────> Gates G01, G02 (V1–V12, E1–E8 Green)
               │
               ▼
[Day 2: Physics Layers 0–2]          ────> Gates G03, G04 (T1–T5, T9, T10 Green)
               │
               ▼
[Day 3: Physical Corruption Layer]   ────> Gates G05, G06 (T6, T7 Green — CRITICAL STOP GATE)
               │
               ▼
[Day 4: Mesh Protocol & TDMA Layer]  ────> Gates G07, G08 (T11, T12, T16–T26 Green)
               │
               ▼
[Day 5: Persistence & Ingestion]     ────> Gates G09, G10 (T8, T13, T27–T31 Green)
               │
               ▼
[Day 6: ARCHITECTURAL FREEZE]        ────> FULL SUITE T1–T46 GREEN (Production Sealed)
```

---

### Question: How are the daily milestones, deliverables, and automated gates (G01–G10) structured across the build cycle?

**Answer:** Each milestone delivers isolated, cryptographically verifiable artifacts tested against uncompromising stop-the-line criteria:

| Milestone Day | Deliverables & Code Modules | Automated Gates & Test Suite | Strict Stop-the-Line Criteria |
| :--- | :--- | :--- | :--- |
| **Day 1: Static Contracts** | `sim/constants.py`, `config/nodes.json` (layout manifest), `data/events.csv` (blast ledger) | **G01 (V1–V12), G02 (E1–E8)** | Any spatial coordinate outside panel boundary fails build; any missing required parameter aborts compilation immediately. |
| **Day 2: Forward Physics** | Analytical $S(x,y,t)$, tilt ($T$), curvature ($K$), strain ($\varepsilon$), and USBM vibration decay | **G03 (T1, T2, T5), G04 (T3, T4, T9, T10)** | Volume conservation ratio $\iint S / (a \cdot m \cdot A) \neq 1.0000 \pm 0.01$; analytic derivatives diverge from finite differences by $> 1 \times 10^{-7}$. |
| **Day 3: Physical Corruption** | 6-stage degradation pipeline: thermal walk, quantization, mechanical sag, dropouts | **G05 (T6), G06 (T7)** | **CRITICAL GATE (T7):** If post-calibration residual exceeds hardware white-noise floor, **STOP THE LINE**. Downstream models will not be built. |
| **Day 4: Network Simulation** | TDMA superframe scheduler, Spreading Factor links, bitmap ACKs, relay failover | **G07 (T17, T18, T19), G08 (T11, T12, T16, T21, T24–T26)** | Worst-case node duty cycle $\ge 1.0\%$; RF carrier bandwidth $> 200\text{ kHz}$ (statutory breach); unacknowledged packet loss during relay failover $> 0\%$. |
| **Day 5: Persistence & Pipes** | Raw ingestion worker, 36h rolling `nodes.csv`, Parquet archival engine | **G09 (T27–T31), G10 (T8, T13, T14)** | Telemetry conservation violated ($\sum \text{Produced} \neq \text{Persisted} + \text{Lost}$); `backend/` attempts to import truth variables (`ModuleNotFoundError`). |
| **Day 6: Final Freeze** | End-to-end integration, scenario tuning, PINN convergence, alarm verification | **FULL SUITE T1–T46** | Any test warning, NaN loss, non-zero exit code, or unhandled exception halts system deployment. |

---

### Question: Why is Day 3 Gate G06 (Test T7) designated as an absolute "Stop-the-Line" checkpoint?

**Answer:** Gate G06 evaluates Test T7: the mathematical invertibility of the physical sensor degradation pipeline. In Day 3, synthetic ground truth undergoes a 6-stage degradation chain:
1. Soil-structure thermal coupling ($k_T \cdot \Delta T$).
2. Gauss-Markov random bias walk ($\sigma_b = 3.0\ \mu\text{rad}$, $\tau = 6.0\text{ h}$).
3. ADC quantization rounding ($2\ \mu\text{rad}$, $1\ \mu\varepsilon$, $10\ \mu\text{m}$ LSB).
4. Mechanical anchoring sag and cable relaxation.
5. High-shadow fading attenuation and packet dropouts.
6. Common-mode topsoil swelling.

If Test T7 fails, it proves that the C7 Calibration Pipeline cannot mathematically recover ground truth within the sensor's baseline Gaussian white-noise envelope ($\sigma_w = 8.0\ \mu\text{rad}$, $1.2\ \mu\varepsilon$). Permitting development to proceed past Day 3 when T7 is red would inject permanent systematic bias into the C8 Byzantine Quorum detector (causing false evacuation alarms) and corrupt the loss gradients of the C9 Physics-Informed Neural Network (preventing PDE convergence). Development halts until either the physical sensor model is re-calibrated or the C7 inversion math is corrected.

---

### Question: What constitutes an "Architectural Freeze" upon passing Day 6, and what operational constraints does it impose?

**Answer:** Once Day 6 achieves green status across all 46 verification tests, the codebase enters an immutable **Architectural Freeze**. This locks four core operational interfaces:

1. **Immutable 23-Byte Binary Wire Format:**
   No bitfields, encodings, or byte offsets in the over-the-air LoRa frame may be altered. The exact packing layout (including the lower 8 bits of the epoch counter, 16-bit signed integer transducer channels, and 8-bit diagnostic bitmasks) is frozen across embedded firmware and gateway parsers.
2. **Immutable System Schemas:**
   The JSON schema for `config/nodes.json` (defining station coordinates, roles, TDMA slots, and primary/backup parents) and the tabular schema for `nodes.csv` (storing 36 hours of raw time-series) are sealed. No columns may be added, renamed, or reordered.
3. **Immutable C7 $\to$ C8/C9 Analytical Contract:**
   The calibrated state dictionary passed across the C7 cleaning boundary to the C8 Safety Detector and C9 Digital Twin is frozen. It guarantees that downstream engines receive fully corrected physical values alongside an explicit dynamic uncertainty $\sigma_{\text{final}}$ and quality flags.
4. **Post-Freeze Codebase Discipline:**
   Under SIH 2026 judging protocols, any theoretical optimizations, cosmetic refactorings, or architectural expansions conceived after Day 6 are formally classified as *v2.1 Roadmaps*. They are barred from being committed as live code modifications during judging defense or active field trials to protect statutory system integrity.
