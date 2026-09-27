# The 6-Day Staged Build Order & Freezing Protocol

**Module 08 — Verification**  
**Cross-References:** [`test-register.md`](test-register.md) · [`field-validation-plan.md`](field-validation-plan.md) · [Module 00 Project Charter](../00-executive-gateway/project-charter.md)

---

## 1. The Sequential Build Philosophy

To prevent circular debugging cycles where numerical bugs in simulation generators are misdiagnosed as machine learning training failures, AEGIS enforces a strict **6-Day Phased Build Order**. 

Each stage is protected by an automated gate; developers cannot advance to the next layer until all previous gate tests are green.

```
[Day 1: Ground Constants & Manifest] ────> Gate V1–V12, E1–E8 Green
               │
               ▼
[Day 2: Physics Layers 0–2]          ────> Gate T1–T5, T9, T10 Green
               │
               ▼
[Day 3: Physical Corruption Layer]   ────> Gate T6, T7 (CRITICAL: IF RED, STOP)
               │
               ▼
[Day 4: Mesh Protocol & TDMA Layer]  ────> Gate T11, T12, T16–T26 Green
               │
               ▼
[Day 5: Persistence & Ingestion]     ────> Gate T8, T13, T27–T31 Green
               │
               ▼
[Day 6: ARCHITECTURAL FREEZE]        ────> FULL SUITE T1–T46 GREEN
```

---

## 2. Day-by-Day Milestone Breakdown

| Milestone Day | Deliverables & Code Modules Built | Automated Test Gate | Stop-the-Line Criteria |
| :--- | :--- | :--- | :--- |
| **Day 1: Static Contracts** | `sim/constants.py`, `config/nodes.json` (layout manifest), `data/events.csv` | **V1–V12, E1–E8** | Any coordinate outside panel boundary fails build. |
| **Day 2: Forward Physics** | Analytical $S(x,y,t)$, tilt, curvature, strain, and background vibration | **T1–T5, T9, T10** | Volume conservation must balance to 1.0000; derivatives $< 1\times 10^{-7}$. |
| **Day 3: Physical Corruption** | 6-stage degradation chain: thermal walk, quantization, sag, dropouts | **T6, T7** | **CRITICAL GATE: If T7 fails, STOP.** Calibration cannot recover truth. |
| **Day 4: Network Simulation** | TDMA superframe, Spreading Factor links, bitmap ACKs, relay failover | **T11, T12, T16–T26** | Worst-node duty cycle must stay $< 1.0\%$; carrier bandwidth $\le 200\text{ kHz}$. |
| **Day 5: Persistence & Pipes** | Ingestion engine, 36h `nodes.csv` rolling store, Parquet archive worker | **T8, T13, T27–T31** | `import truth` must fail; zero rows unaccounted for in memory. |
| **Day 6: Final Freeze** | End-to-end integration, scenario tuning, full test suite pass | **FULL SUITE T1–T46** | Every single test must pass without warnings or overrides. |

---

## 3. The Definition of Architectural Freeze

Once Day 6 is achieved and all 46 tests pass in continuous integration:
1. **Immutable Wire Format:** The 23-byte binary struct cannot be altered. No bits or fields may be shifted.
2. **Immutable Schemas:** The JSON configuration schema for `nodes.json` and the CSV column structure for `nodes.csv` are frozen.
3. **Immutable C7 $\to$ C9 Contract:** The calibrated state dictionary structure passed between C7, C8, and C9 is sealed.
4. **Post-Freeze Rule:** Any theoretical improvements or optimizations identified after Day 6 are logged as *v2.1 Roadmaps*; they are never committed as live code modifications during judging or field trial execution.
