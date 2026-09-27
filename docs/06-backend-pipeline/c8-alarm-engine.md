# C8 Safety Alarm Engine & Byzantine Quorum Gating

**Module 06 — Backend Pipeline**  
**Cross-References:** [`c7-corrector.md`](c7-corrector.md) · [Module 05 Safety Boundary](../05-ai-ml-pipeline/safety-boundary.md) · [Module 07 Alert System](../07-digital-twin-and-ui/alert-system.md)

---

## 1. Exclusive Safety & Alarm Authority

The C8 Safety Engine is the **sole subsystem authorized to trigger emergency evacuation alarms**. It is 100% deterministic, containing zero neural networks, statistical autoencoders, or uncalibrated machine learning models. 

Every trip decision is derived from transparent physical thresholds, spatial consensus quorum checks, and statutory DGMS registers.

```
[Cleaned Calibrated State from C7]
                 │
                 ▼  Stage 1: Multi-Tier Threshold Evaluation
[Check Strain ε > 1500 µε, Tilt > 10 mm/m, Crack Wire Breached]
                 │
                 ▼  Stage 2: DGMS Circular 7/1997 Blast Veto
[Cross-reference events.csv: Is vibration within ±5s of blast?]
                 │      ├── Yes ──> [VETO ALARM: Tagged BLAST_EXCLUDED]
                 └── No ┘
                 ▼  Stage 3: 5-Node Byzantine Quorum Gate
[Do ≥ 3 spatial neighbors confirm ≥ 3σ anomaly?]
                 │      ├── No  ──> [SUPPRESS: Flagged SINGLE_NODE_ANOMALY]
                 └── Yes ┘
                 ▼  Stage 4: F9 Temporal Staleness Verification
[Is data fresher than 180 seconds?]
                 │      ├── No  ──> [HOLD: Mark System STALE, emit no QUIET]
                 └── Yes ┘
                 ▼
    ★ EVACUATION DISPATCH TRIPPED ★
    - Physical Siren Relay Activated (<1.4s)
    - Automated SMS Sent to Shift Supervisors
    - Control Room Emergency SCADA Lockout
```

---

## 2. Multi-Level Safety Alert Thresholds

Thresholds are established in strict accordance with Indian Coal Mines Regulations (CMR 2017) and British Coal Board standard subsidence criteria:

| Alert Tier | Visual Level | Physical Threshold Condition | Action Dispatched |
| :--- | :--- | :--- | :--- |
| **Normal (Quiet)** | Green | $\varepsilon \le 500\ \mu\varepsilon$, Tilt $\le 3\text{ mm/m}$, Crack = 00 | Routine telemetry logging |
| **Tier 1: Advisory** | Blue | $500\ \mu\varepsilon < \varepsilon \le 1000\ \mu\varepsilon$ OR Tilt $> 3\text{ mm/m}$ | Visual dashboard advisory alert |
| **Tier 2: Warning** | Yellow | $1000\ \mu\varepsilon < \varepsilon \le 1500\ \mu\varepsilon$ OR Tilt $> 6\text{ mm/m}$ | SMS alert to Geotechnical Officer |
| **Tier 3: Critical** | Orange | $\varepsilon > 1500\ \mu\varepsilon$ OR Acceleration $> 3\times$ baseline | Automated shift supervisor call tree |
| **Tier 4: Emergency** | **Red** | **Quorum confirmed $\varepsilon > 1500\ \mu\varepsilon$ OR Crack = 11** | **Full-Mine Evacuation Siren (<1.4s)** |

---

## 3. 5-Node Byzantine Quorum Gating (Tests T42 & T43)

In outdoor industrial mining, individual sensor stations can be stepped on by cattle, damaged by rocks, or suffer ADC pin shorts. A single damaged sensor reporting an extreme spike must never trigger an accidental mine-wide evacuation.

AEGIS implements a **5-Node Byzantine Quorum Gate**:
* When node $k$ records an anomaly exceeding $3\sigma$ threshold:
  1. The engine identifies its 4 nearest spatial neighbors located within the Knothe influence radius ($r$).
  2. The engine evaluates the valid calibrated readings from all 5 stations.
  3. **The Quorum Rule:** At least **3 out of the 5 spatial neighbor stations** must simultaneously confirm an anomalous strain rate ($\ge 3\sigma$) in the same directional axis.

### Verification Guarantees in CI:
* **Rogue Node Test (Test T42):** A simulated rogue node is injected with a massive $20\sigma$ strain reading. Because neighboring nodes report normal baseline conditions, the Byzantine quorum vetoes the alert. Zero sirens fire.
* **Dead Node Exclusion (Test T43):** If a sensor fails its internal self-test (`selftest_ok = 0`) or reports a frozen, constant voltage over 10 consecutive cycles, it is marked invalid and stripped of voting rights. Dead sensors cannot falsely report "All Clear".

---

## 4. DGMS Circular 7/1997 Blast Veto (Test T39)

Indian coal mines conduct daily scheduled production blasting. Detonations generate high-amplitude shockwaves (Peak Particle Velocity $> 20\text{ mm/s}$) that momentarily flex surface strain gauges for 1–2 seconds.

* Conventional threshold detectors interpret this blast vibration as an imminent slope collapse, triggering false alarms during every blasting shift.
* **The AEGIS Blast Filter:**
  The C8 engine monitors the digital blast register (`events.csv`). When a vibration surge occurs, the engine checks whether a blast was scheduled within a **±5-second window**.
  * If a scheduled blast matches the timestamp and mining sector, the event is marked `vetoed_by = BLAST_REGULAR`. The siren is suppressed.
  * If a violent shockwave ($> 15\text{ mm/s}$) occurs **without** a corresponding entry in `events.csv`, C8 immediately trips **Class-A Unlogged Blast / Seismic Shock Alert** (verified by test `T40`).

---

## 5. The Staleness Rule: F9 Backhaul Defense (Test T46)

If the gateway loses cellular connectivity or the ingestion buffer stalls:
* Stale historical telemetry cannot reliably confirm whether ground conditions are safe.
* **Rule F9:** Telemetry older than 180 seconds is marked **STALE**.
* While marked STALE, C8 is strictly forbidden from emitting a `QUIET / ALL_CLEAR` status. The dashboard displays a prominent amber banner: **COMMUNICATIONS DEGRADED — SYSTEM CANNOT CONFIRM ALL-CLEAR**.
