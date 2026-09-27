# The Global Verification Test Register (T1 – T46)

**Module 08 — Verification**  
**Cross-References:** [`build-order.md`](build-order.md) · [`field-validation-plan.md`](field-validation-plan.md) · [Module 00 System Boundaries](../00-executive-gateway/system-architecture.md)

---

## 1. Single Global Test Numbering Authority

To eliminate conflicting test definitions across subsystems, this register serves as the **sole authority for test numbering across the AEGIS platform**. No subsystem may invent or re-use a test identifier.

---

## 2. Complete Catalogue of Automated Tests

### Group A: Physics & Numerical Integrity (Tests T1 – T10)

| Test | Owner | Asserts / Acceptance Criteria | Impact If Test Fails |
| :--- | :--- | :--- | :--- |
| **T1** | Physics | Volume conservation: $\iint S_{\text{final}} \approx a \cdot m \cdot A$ within $1.0\%$ (ratio 1.0000) | Mass leakage in subsidence integration |
| **T2** | Physics | Asymptotic decay: $S_{\text{final}} \to 0$ beyond influence radius $r$ (Anchors $< 1\text{ mm}$) | Distorts baseline elevation datum |
| **T3** | Physics | Analytic derivatives match central finite differences to $< 1 \times 10^{-7}$ | Tilt and strain equations mathematically invalid |
| **T4** | Physics | Peak magnitudes match closed form ($\varepsilon_{\text{max}} = 12,649\ \mu\varepsilon$, $T_{\text{max}} = 25,978\ \mu\text{rad}$) to $\pm 2\%$ | Wrong radius $r$ or Awershin ratio $B_{\text{horiz}}$ |
| **T5** | Physics | Spatio-temporal separability: $S(t) / S(t') = \eta(t) / \eta(t')$ everywhere on grid | Time decay curve violates Knothe mechanics |
| **T6** | Physics | Corruption chain executes strictly in documented order (Order-hash assertion) | Thermal and sag errors cross-contaminate |
| **T7** | C7/Clean | **Undoing sag then thermal leaves residual within white-noise band. IF RED, STOP.** | **C7 cleaning pipeline cannot recover ground truth** |
| **T8** | Ingest | `import truth` from anywhere under `backend/` raises `ModuleNotFoundError` | Circular ground-truth leakage into production |
| **T9** | Physics | Analytic ground truth reproduces check points to $< 1 \times 10^{-6}$ | Numeric drift in simulation generator |
| **T10**| Physics | Simulated crack trip occurs at $t_{\text{crack}} = 8.93\text{ days} \pm 1$ sample | Advance early warning prediction invalid |

### Group B: Mesh Wireless & RF Protocol (Tests T11 – T26)

| Test | Owner | Asserts / Acceptance Criteria | Impact If Test Fails |
| :--- | :--- | :--- | :--- |
| **T11**| Mesh | `relay_kill` event loses **zero rows**; packets recover via backup parent slots | Structural relay destruction causes data loss |
| **T12**| Mesh | Measured worst-case node duty cycle $< 1.0\%$ across continuous 40-day run | Violates radio duty cycle conventions |
| **T13**| Mesh | Unexpected node reboot resetting sequence counter produces no upsert collision | Telemetry database crashes or overwrites rows |
| **T14**| Ingest | Every `nodes.csv` row's `node_id` exists in `nodes.json` configuration manifest | Unregistered orphan telemetry in backend |
| **T15**| Physics | Both Bedrock Anchors report $|S| < 1\text{ mm}$ across all time slices | Infiltrating subsidence into reference anchor |
| **T16**| Mesh | Every `(node, parent)` link distance is within derived reliable RF propagation range | Radio packets drop due to excessive hop distance |
| **T17**| Mesh | No two nodes share a primary TDMA slot or a backup emergency slot | Co-channel packet collision storms |
| **T18**| Mesh | **Carrier bandwidth $\le 200\text{ kHz}$ on all presets (125 kHz). Statutory, non-negotiable.** | **Statutory violation of Indian Gazette GSR 564(E)** |
| **T19**| Mesh | Gateway measured duty cycle $< 1.0\%$ (Verified with Bitmap ACKs) | Gateway saturates channel and violates norms |
| **T20**| Twin | Largest Empty Circle ($2 \cdot R_{\text{LEC}} \le d_{\text{committed}} = 358\text{m}$) published in UI header | False claims of blind-spot-free coverage |
| **T21**| Mesh | Every primary and backup parent hop has $\ge 10\text{ dB}$ margin at 90th percentile shadowing | High packet loss during rain or coal dust storms |
| **T22**| C8/Alarm| `relay_kill` produces diagnostic **F4**, NEVER false ground-collapse alarm F3 | False full-mine evacuation from dead battery |
| **T23**| Ingest | `epoch` is unsigned 32-bit (`uint32`); 100-day continuous run produces no wraparound | Clock wraparound after 45 days |
| **T24**| Mesh | 3-hour gateway beacon outage: static-slot fallback produces zero slot collisions | Network collapses into ALOHA chaos during outage |
| **T25**| Mesh | 20 simultaneous emergency node trips produce $\le 3$ originated RF packets | Emergency alarm broadcast storm |
| **T26**| Mesh | Measured Relay RX duty cycle $\le 4\times$ Leaf RX duty cycle | Anchor relay battery exhausts prematurely |

### Group C: Backend Pipeline, Safety Quorum & Alarms (Tests T27 – T46)

| Test | Owner | Asserts / Acceptance Criteria | Impact If Test Fails |
| :--- | :--- | :--- | :--- |
| **T27**| Ingest | 23-byte pack/unpack round-trips exactly for 10,000 random numerical vectors | Bit truncation or endianness serialization bug |
| **T28**| Ingest | No row exists during a `node_kill` window, and missing cells are **never zero-filled** | Artificial derivative spikes trip false sirens |
| **T29**| Ingest | **$\sum \text{Produced} \equiv \text{Persisted} + \text{Buffered} + \text{Lost} + \text{Dropped}$. Nothing vanishes.** | **Unaccounted silent data loss** |
| **T30**| Ingest | `nodes.csv` row count never exceeds $N \times \text{retention\_h} \times 60$ at any epoch | Unbounded disk growth crashes gateway |
| **T31**| Ingest | Frame replayed 40 hours late via store-and-forward lands on its original epoch | Historical time-series distortion |
| **T32**| C9/PINN| `grep backend/c9/` finds no references to `site.a`, `site.c`, `S_MAX`, or `truth` | PINN cheating by accessing ground truth |
| **T33**| C9/PINN| At Day 40: $\hat{a} \cdot \hat{\eta}(t)$ within 10% of truth; $\hat{a}$ within 15%; $\hat{c}$ within 25% | Surface reconstruction model fails convergence |
| **T34**| C9/PINN| Disabling bedrock anchor loss causes mean grid elevation offset to drift $> 10\text{ cm}$ | Loss of absolute elevation datum |
| **T35**| C9/PINN| Switching neural activation to ReLU forces strain loss constant and unlearnable | Second-derivative vanishing gradient |
| **T36**| C9/PINN| PINN training window spans $\ge 12\text{ hours}$ of distinct time slices | Time decay $\hat{c}$ mathematically unidentifiable |
| **T37**| Ingest | `meta.generator_sha256` matches cryptographic hash of generator source | Simulation runs lose scientific reproducibility |
| **T38**| Physics| The four Knothe separability preconditions are asserted at generation time | Mathematical model applied outside valid domain |
| **T39**| C8/Alarm| Day-3 scheduled blast in `events.csv` $\implies \text{vetoed\_by} = \text{BLAST}$, NO siren | Production blast triggers false mine evacuation |
| **T40**| C8/Alarm| Day-21 unlogged shockwave $\implies \text{CLASS-A / UNLOGGED\_BLAST}$ alarm | Unmonitored explosive event ignored |
| **T41**| C8/Alarm| Day-26 cluster kill event trips `CLASS-A` within one single epoch | Sudden collapse ignored due to buffer lag |
| **T42**| C8/Alarm| **Single Byzantine node with $20\sigma$ strain produces NO alarm (Quorum veto)** | **Rogue broken sensor evacuates mine** |
| **T43**| C8/Alarm| **Node reporting `selftest_ok = 0` or frozen voltage is excluded from quorum** | **Dead sensor falsely reports "All Clear"** |
| **T44**| Safety | **`grep -rn "C9\|pinn\|S_grid" backend/c8/` returns ZERO occurrences** | **AI model breaches hard safety boundary** |
| **T45**| C7/Clean| $f_{\text{gap}}$ uncertainty multiplier is capped at 10.0 | $\sigma$ explodes numerically after 3-day outage |
| **T46**| C8/Alarm| Rule F9: Backhaul loss $> 180\text{s} \implies$ dashboard STALE, C8 emits NO Quiet | Blind system falsely reports "Safe" |

---

## 3. The Four Non-Negotiable Gatekeeper Tests

While all 46 tests must pass before deployment, four tests serve as fundamental architectural gatekeepers:
1. **Test T7 (Calibration Integrity):** Proves that physical calibration algorithms can invert the 6-stage degradation chain.
2. **Test T29 (Conservation of Telemetry):** Proves mathematically that the wireless store-and-forward mesh loses zero telemetry rows.
3. **Test T42 (Byzantine Resilience):** Proves that a physically smashed or electrically shorted sensor cannot trigger an accidental mine evacuation.
4. **Test T43 (Failure-to-Safety):** Proves that a dead sensor cannot report a false "All Clear" signal—the failure mode that could result in worker fatalities.
