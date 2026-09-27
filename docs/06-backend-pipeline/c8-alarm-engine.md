# C8 Safety Alarm Engine & Byzantine Quorum Gating

**Module 06 — Backend Pipeline**  
**Cross-References:** [`c7-corrector.md`](c7-corrector.md) · [Module 05 Safety Boundary](../05-ai-ml-pipeline/safety-boundary.md) · [Module 07 Alert System](../07-digital-twin-and-ui/alert-system.md)

---

### Question: Why is the C8 Safety Alarm Engine granted exclusive, deterministic authority over mine evacuation sirens, and why are AI/ML models strictly barred from this trip path?
**Answer:** The C8 Safety Engine is the sole subsystem in the AEGIS architecture authorized to trigger mine-wide emergency evacuation sirens, automated supervisor phone call trees, and SCADA emergency lockouts. It operates under 100% deterministic, closed-form rule sets with zero dependency on artificial neural networks, deep learning heuristics, or uncalibrated autoencoders.

Under Directorate General of Mines Safety (DGMS) statutory frameworks and Indian Coal Mines Regulations (CMR 2017), life-critical safety systems must be fail-safe, mathematically bounded, and 100% auditable in post-incident legal inquiries. Machine learning models—including Physics-Informed Neural Networks (PINNs)—can suffer out-of-distribution hallucinations, vanishing gradients, or unpredictable inferences when encountering rare physical fracture dynamics. In AEGIS, machine learning (Module 05 / C9) is placed behind a strict architectural firewall and utilized solely for advisory continuous 3D digital twin rendering. Every life-critical evacuation trip originates strictly within C8 through transparent physical thresholds, spatial Byzantine consensus, and statutory blast register cross-referencing.

```
[Cleaned Calibrated State from C7 Corrector]
                 │
                 ▼  Stage 1: Multi-Tier Threshold Evaluation
[Check Strain ε > 1500 µε, Tilt > 10 mm/m, Crack Wire Breached]
                 │
                 ▼  Stage 2: DGMS Circular 7/1997 Blast Veto
[Cross-reference events.csv: Is vibration within ±5s of blast?]
                 │      ├── Yes ──> [VETO ALARM: Tagged BLAST_EXCLUDED]
                 └── No ┘
                 ▼  Stage 3: 5-Station Byzantine Quorum Gate
[Do ≥ 3 spatial neighbors confirm ≥ 3σ anomaly in Knothe radius?]
                 │      ├── No  ──> [SUPPRESS: Flagged SINGLE_NODE_ANOMALY]
                 └── Yes ┘
                 ▼  Stage 4: F9 Temporal Staleness Verification
[Is telemetry fresher than 180 seconds?]
                 │      ├── No  ──> [HOLD: Mark System STALE, emit no QUIET]
                 └── Yes ┘
                 ▼
    ★ EVACUATION DISPATCH TRIPPED ★
    - Physical Siren Relay Activated (<1.4s total latency)
    - Automated SMS Dispatched to Shift Supervisors
    - Control Room Emergency SCADA Lockout Latch
```

---

### Question: What multi-tier safety alert thresholds are enforced under DGMS Coal Mines Regulations (CMR 2017) and British Coal Board standards?
**Answer:** C8 evaluates calibrated physical states against four graded operational safety tiers derived from statutory subsidence engineering guidelines:

| Alert Tier | Visual Level | Physical Threshold Condition | Action Dispatched |
| :--- | :--- | :--- | :--- |
| **Normal (Quiet)** | Green | $ε ≤ 500\ µε$, Tilt $≤ 3\text{ mm/m}$, Crack = `00` | Routine telemetry logging; continuous monitoring |
| **Tier 1: Advisory** | Blue | $500\ µε < ε ≤ 1000\ µε$ OR Tilt $> 3\text{ mm/m}$ | Visual dashboard advisory alert; maintenance notification |
| **Tier 2: Warning** | Yellow | $1000\ µε < ε ≤ 1500\ µε$ OR Tilt $> 6\text{ mm/m}$ | Automated SMS alert to Geotechnical Officer; 15-min ack window |
| **Tier 3: Critical** | Orange | $ε > 1500\ µε$ OR Acceleration $> 3×$ baseline | Automated shift supervisor call tree; 3-min escalation countdown |
| **Tier 4: Emergency** | **Red** | **Quorum confirmed $ε > 1500\ µε$ OR Crack Wire = `11`** | **Full-Mine Evacuation Siren (<1.4s); SCADA Emergency Lockout** |

---

### Question: How does the 5-Station Byzantine Quorum Gate prevent false mine evacuations caused by localized sensor damage (Tests T42 & T43)?
**Answer:** In active open-cast and longwall coal mines, surface sensor stations operate in harsh industrial environments. Stations can be physically damaged by loose rockfall, trampled by wild cattle, or suffer internal ADC pin short-circuits. If a single compromised node suddenly registers a violent strain spike of $10,000\ \mu\varepsilon$, an unsophisticated threshold engine would trigger an immediate mine-wide panic and unnecessary multi-crore production shutdown.

To prevent this, AEGIS implements a **5-Station Byzantine Quorum Gate**:
1. When any station $k$ detects a strain or tilt reading exceeding its local $3\sigma$ baseline threshold, C8 queries the dynamic spatial manifest (`nodes.json`).
2. The engine identifies station $k$'s four nearest physical neighbors located within the Knothe zone of influence ($r = H / \tan\beta$, where $H$ is extraction depth and $\beta$ is the angle of draw).
3. C8 examines the validated readings (filtered by Step 8 validity masks) across all 5 spatial stations.
4. **The Quorum Rule:** At least **3 out of the 5 spatial stations** (a strict majority $\ge 60\%$) must simultaneously confirm an anomalous displacement rate ($\ge 3\sigma$) in the same directional vector.

```
       [Neighbor 1] (Normal)
             \
  [Neighbor 2] --- [Station k: SPIKE 20σ] --- [Neighbor 3] (Normal)
             /
       [Neighbor 4] (Normal)

  Consensus: 1 / 5 Confirms → VETOED (Logged: SINGLE_NODE_ANOMALY)
```

#### Automated Verification Guarantees:
* **Rogue Node Test (Test `T42`):** Continuous integration injects a synthetic $20\sigma$ strain surge into a single node while keeping neighbors nominal. C8 vetoes the siren trip, flags the offending station as `SINGLE_NODE_ANOMALY`, and dispatches an equipment maintenance ticket.
* **Dead Node Exclusion (Test `T43`):** If a sensor fails its internal self-test (`selftest_ok == 0`) or produces frozen identical ADC counts over 10 consecutive epochs, it is immediately stripped of quorum voting rights. Dead nodes cannot cast votes to falsely claim ground stability.

---

### Question: How does the DGMS Circular 7/1997 Blast Veto distinguish regular production blasting from catastrophic slope collapse (Tests T39 & T40)?
**Answer:** Daily production blasting in open-cast benches and longwall development headings detonates heavy explosive charges (40 kg to 200+ kg). These detonations propagate transient high-amplitude seismic shockwaves with Peak Particle Velocity (PPV) exceeding $20\text{ mm/s}$, momentarily flexing surface strain gauges and accelerometers for 1 to 3 seconds before decaying. Without domain intelligence, these shockwaves trigger false evacuation alarms during every scheduled blasting shift.

C8 implements a deterministic blast filtering veto under DGMS Circular 7 of 1997:
1. The engine maintains continuous access to the certified colliery blast register (`data/events.csv`).
2. When a high-amplitude vibration ($> 15\text{ mm/s}$) or transient strain spike is detected, C8 checks `events.csv` for an authorized detonation within a tight **$\pm 5$-second temporal window** and matching mining seam/panel coordinates.
3. If a scheduled detonation matches, C8 suppresses the siren, logs the event as `vetoed_by = BLAST_REGULAR`, and enters a 30-second post-blast stabilization mode (verified by test `T39`).
4. **Unlogged Blast Detection (Test `T40`):** If a violent shockwave ($> 15\text{ mm/s}$) occurs **without** a matching entry in `events.csv`, C8 immediately treats the event as an unlogged blast, seismic burst, or sudden deep strata failure, immediately tripping a **Class-A Seismic Shock Alert** and alerting shift safety controllers.

---

### Question: What is the F9 Temporal Staleness Rule, and why is C8 strictly forbidden from emitting an "All Clear" status during communications loss (Test T46)?
**Answer:** In heavy industrial mining, cellular backhaul links and LoRa mesh gateways can experience temporary outages caused by severe monsoon lightning strikes, auxiliary generator cutovers, or fiber backhaul severed by heavy excavators. 

Under safety-critical systems engineering principles, **the absence of data must never be equated to the absence of danger**:
1. Stale telemetry cannot confirm whether ground strata have remained stable or collapsed during a communications blackout.
2. **Rule F9:** Any station or panel whose telemetry stream is older than **180 seconds** ($3\times$ the nominal reporting period) is automatically classified as `STALE`.
3. **Safety Invariant:** While any station within an active extraction zone is marked `STALE`, C8 is hard-coded to **never emit a `QUIET / ALL_CLEAR` status**.
4. The SCADA dashboard and field mobile interfaces display an assertive high-visibility amber warning banner: **COMMUNICATIONS DEGRADED — SYSTEM CANNOT CONFIRM ALL-CLEAR**.
5. Continuous integration test `T46` asserts this behavior by cutting the synthetic data stream; if C8 outputs a `QUIET` state after 181 seconds of silence, the test suite aborts the build.

---

### Question: What physical actions are dispatched when a Tier 4 Emergency is tripped, and what is the end-to-end siren actuation latency?
**Answer:** When C8 confirms a Tier 4 Emergency (via 5-station Byzantine consensus or physical crack-wire loop severance), it bypasses all software UI queues and immediately actuates local hardware controls:
1. **Physical Siren Relay Actuation:** An onboard solid-state relay on the Master Edge Gateway latches a 24V DC circuit, sounding a 125 dB omnidirectional acoustic mine evacuation siren mounted on the gateway mast. Total latency from sensor trip to acoustic sound emission is guaranteed at **$< 1.4\text{ seconds}$**.
2. **Local Siren Independence:** Because the Master Edge Gateway runs an embedded instance of C8 on bare-metal firmware/Linux, siren actuation occurs locally with zero dependency on internet connectivity, cellular networks, or cloud webhooks.
3. **Automated Notification Tree:** Parallel background workers dispatch priority SMS text alerts and initiate automated text-to-speech phone calls to shift supervisors, colliery managers, and rescue stations.
4. **SCADA Emergency Lockout:** Connected mission control dashboards enter full-screen visual emergency lockout, displaying the evacuation perimeter, escape routes, and affected infrastructure vectors.
