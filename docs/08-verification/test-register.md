# The Global Verification Test Register (T1 – T46)

**Module 08 — Verification**  
**Cross-References:** [`build-order.md`](build-order.md) · [`field-validation-plan.md`](field-validation-plan.md) · [Module 00 System Boundaries](../00-executive-gateway/system-architecture.md)

---

### Question: Why does AEGIS enforce a single, immutable Global Test Register (T1–T46), and how does it prevent cross-subsystem verification drift?

**Answer:** In complex cyber-physical deployments combining geotechnical mechanics, wireless radio firmware, distributed ingestion pipelines, and safety-critical alarm engines, teams frequently create competing or conflicting unit tests. A test named "test_strain_threshold" in firmware might test raw ADC integer ticks, while a test with the same name in the backend tests calibrated micro-strains, creating catastrophic false assurances of safety.

To eliminate ambiguity, this document serves as the **sole, immutable authority for verification across the AEGIS project**. Every automated test is assigned a globally unique identifier (T1 through T46). Subsystem repositories are strictly barred from renumbering, overriding, or creating informal off-register tests. If any single test in this register fails in CI, the entire build is rejected and deployment halts.

---

### Question: How are the forward physics and discrete numerical models verified in Group A (Tests T1 – T10)?

**Answer:** Tests T1 through T10 establish the absolute mathematical validity of the ground mechanics engine, ensuring that synthetic ground truth accurately reflects empirical Knothe subsidence physics before any sensor degradation is applied:

| Test ID | Subsystem Owner | Exact Acceptance Criteria & Assertions | Catastrophic Risk Averted If Test Fails |
| :--- | :--- | :--- | :--- |
| **T1** | Physics Engine | **Volume Conservation:** Surface integral of final subsidence equals total extracted volume within $1.0\%$: $\iint S_{\text{final}}(x,y)\,dx\,dy = a \cdot m \cdot A$ (Ratio: $1.0000 ± 0.01$). | Unphysical mass creation or mass leakage in subsidence integration. |
| **T2** | Physics Engine | **Asymptotic Decay:** Subsidence asymptotically approaches zero outside influence radius $r$: $S(x,y) < 1.0\text{ mm}$ at Bedrock Anchor locations ($d ≥ 1.5r$). | Artificial displacement leaks into undisturbed bedrock reference anchors. |
| **T3** | Physics Engine | **Analytic Derivative Equivalence:** Closed-form analytic spatial derivatives ($\partial S/\partial x$, $\partial^2 S/\partial x^2$) match central finite differences to within absolute tolerance $< 1 × 10^{-7}$. | Tilt and horizontal strain calculations mathematically corrupted. |
| **T4** | Physics Engine | **Peak Magnitude Verification:** Closed-form peak values ($ε_{\text{max}} = 12,649\ µε$, $T_{\text{max}} = 25,978\ µrad$) match numerical maximums to within $± 2.0\%$. | Erroneous influence radius $r$ or Awershin horizontal displacement ratio $B_{\text{horiz}}$. |
| **T5** | Physics Engine | **Spatio-Temporal Separability:** Profile shape preserves time separability: $S(x,y,t) / S(x,y,t') \equiv \eta(t) / \eta(t')$ identically across all spatial points. | Dynamic time-relaxation curve violates fundamental Knothe mechanics. |
| **T6** | Physics Engine | **Corruption Order Invariance:** The 6-stage degradation pipeline executes in exact, cryptographically hashed sequence: thermal $\to$ bias walk $\to$ ADC quantization $\to$ sag $\to$ dropouts $\to$ swelling. | Cross-contamination of irreversible physical degradation mechanisms. |
| **T7** | C7 Pipeline | **CRITICAL STOP-THE-LINE GATE:** Inverting mechanical sag followed by thermal compensation leaves a residual error bounded within the sensor white-noise envelope ($≤ 1.2\ µε$, $≤ 8.0\ µrad$). | **C7 calibration cannot invert degradation; true ground motion is permanently lost.** |
| **T8** | Ingestion Engine | **Zero Ground-Truth Leakage:** Executing `import truth` or accessing simulation state from any module within `backend/` raises `ModuleNotFoundError`. | Machine learning or alarm engines artificially "cheating" by inspecting synthetic truth. |
| **T9** | Physics Engine | **Deterministic Re-computability:** Re-running analytical forward models with identical seed and manifest matches baseline checkpoints to $< 1 × 10^{-6}$. | Numerical drift or non-deterministic floating-point accumulation across platforms. |
| **T10** | Physics Engine | **Critical Inflection Prediction:** Tensile strain exceeds critical crack threshold ($ε ≥ 1500\ µε$) at exactly $t = 8.93\text{ days} ± 1$ time-step. | Predictive early warning model invalid; cannot guarantee 7-day advance notice. |

---

### Question: How does Group B (Tests T11 – T26) ensure wireless mesh reliability, collision avoidance, and spectrum compliance?

**Answer:** Tests T11 through T26 validate the physical LoRa link budget, the collision-free TDMA superframe, and strict compliance with national telecommunication laws:

| Test ID | Subsystem Owner | Exact Acceptance Criteria & Assertions | Catastrophic Risk Averted If Test Fails |
| :--- | :--- | :--- | :--- |
| **T11** | Mesh Protocol | **Relay Failover Losslessness:** Sudden deactivation of an Anchor Relay (`relay_kill`) results in **zero dropped telemetry frames**; child nodes route via backup slots. | Structural collapse destroying a relay severs communication from the critical face. |
| **T12** | Mesh Protocol | **Node Duty Cycle Ceiling:** Measured worst-case node transmission airtime occupies $< 1.0\%$ of spectrum time over a 40-day continuous run. | Violates international low-power radio conventions; accelerates battery depletion. |
| **T13** | Ingestion Engine | **Reboot Sequence Idempotency:** Sudden microcontroller reboot resetting the packet sequence counter to 0 produces zero database upsert collisions or lost rows. | Telemetry database crashes or overwrites historical records following lightning reboots. |
| **T14** | Ingestion Engine | **Manifest Authentication:** Every row written to `nodes.csv` contains a `node_id` that strictly exists within the frozen `config/nodes.json` manifest. | Rogue, unauthenticated, or corrupted RF frames inject ghost nodes into backend. |
| **T15** | Physics Engine | **Bedrock Stability Verification:** Both Bedrock Anchors ($A_1, A_2$) report $|S| < 1.0\text{ mm}$ across all 57,600 epochs of the simulation run. | Ground subsidence infiltrates reference stations, invalidating common-mode rejection. |
| **T16** | Mesh Protocol | **Link Distance Budget:** Every assigned primary and backup parent link distance is within the calculated safe RF propagation range under heavy foliage/dust attenuation. | Nodes assigned to unreachable parents; structural network fragmentation. |
| **T17** | Mesh Protocol | **Deterministic Slot Orthogonality:** No two nodes in the network manifest are assigned overlapping primary TDMA transmission slots or backup emergency slots. | Co-channel packet collisions causing unrecoverable data packet destruction. |
| **T18** | Mesh Protocol | **Indian Spectrum Statute Compliance:** Carrier bandwidth is strictly configured to **125.0 kHz** (well below the legal $≤ 200\text{ kHz}$ statutory ceiling) on all presets. | **Statutory violation of Indian Gazette GSR 564(E); punishable under Wireless Telegraphy Act.** |
| **T19** | Mesh Protocol | **Gateway Duty Cycle Ceiling:** Gateway transmission airtime (including sync beacons and Bitmap ACKs) occupies $< 1.0\%$ of spectrum time. | Master Gateway saturates the ISM channel, blocking emergency uplinks. |
| **T20** | Digital Twin | **Spatial Coverage Transparency:** The Largest Empty Circle diameter ($d_{\text{committed}} ≤ 358\text{ m}$) is computed and continuously displayed on the SCADA header. | Operator misinterprets sparse sensor layouts as blind-spot-free coverage. |
| **T21** | Mesh Protocol | **Shadow Fading Fade Margin:** Every wireless link maintains $≥ 10.0\text{ dB}$ link margin under 90th percentile log-normal shadow fading. | High packet dropouts during severe monsoon rainstorms or coal dust clouds. |
| **T22** | C8 Alarm Engine | **Failover Diagnostic Isolation:** A `relay_kill` event emits a Diagnostic Equipment Fault (**F4**), NEVER a false ground collapse siren (**F3**). | Hardware battery exhaustion triggers a false full-mine emergency evacuation. |
| **T23** | Ingestion Engine | **Clock Counter Overflow Immunity:** Epoch field is formatted as unsigned 32-bit integer (`uint32`); continuous 100-day simulation produces zero wraparound. | System clock wraps around after 45 days, causing time-series chronology reversal. |
| **T24** | Mesh Protocol | **Beacon Outage Fallback:** In the event of a continuous 3-hour gateway beacon outage, nodes transition to drift-tolerant static slots with zero packet collisions. | Gateway GPS lock loss collapses the entire mesh network into ALOHA contention chaos. |
| **T25** | Mesh Protocol | **Broadcast Storm Suppression:** 20 simultaneous critical emergency trip switches produce $≤ 3$ aggregated RF broadcast packets across the network. | Simultaneous sensor trips flood the radio spectrum, jamming emergency siren triggers. |
| **T26** | Mesh Protocol | **Relay Power Proportionality:** Measured Anchor Relay RX power consumption does not exceed $4×$ Leaf Scout power consumption. | Anchor Relay battery exhausts prematurely, severing downstream cluster communications. |

---

### Question: How does Group C (Tests T27 – T46) guarantee data persistence, machine learning isolation, and fail-safe alarm governance?

**Answer:** Tests T27 through T46 enforce strict data conservation, verify that machine learning models cannot corrupt safety alarms, and assert deterministic emergency siren tripping:

| Test ID | Subsystem Owner | Exact Acceptance Criteria & Assertions | Catastrophic Risk Averted If Test Fails |
| :--- | :--- | :--- | :--- |
| **T27** | Ingestion Engine | **Binary Serialization Round-Trip:** Packing and unpacking the 23-byte wire struct round-trips with zero bit loss across 10,000 random numerical test vectors. | Endianness errors or bitmask truncation silently altering critical sensor values. |
| **T28** | Ingestion Engine | **Missing Data Sanitization:** Missing packets during a sensor blackout are recorded as nulls and are **strictly never zero-filled**. | Artificially injected zeroes create massive artificial derivative spikes ($\partial ε/\partial t$), tripping false sirens. |
| **T29** | Ingestion Engine | **Conservation of Telemetry:** Total frames produced strictly equals the sum of frames persisted, buffered, lost, and dropped: $\sum \text{Produced} \equiv \text{Persisted} + \text{Buffered} + \text{Lost} + \text{Dropped}$. | **Unaccounted, silent data loss; packets vanishing without trace in software queues.** |
| **T30** | Ingestion Engine | **Bounded Rolling Store:** File size of `nodes.csv` never exceeds $N_{\text{nodes}} × 36\text{ h} × 60\text{ samples}$ under continuous execution. | Unbounded telemetry growth consumes edge gateway disk space, crashing the operating system. |
| **T31** | Ingestion Engine | **Out-of-Order Ingestion:** A packet buffered on-node for 40 hours during a network partition correctly lands in its original historical epoch upon reconnection. | Replayed packets overwrite current data, corrupting the real-time safety baseline. |
| **T32** | C9 Digital Twin | **Strict PINN Isolation:** Executing `grep -rn -E "site\.a|site\.c|S_MAX|truth" backend/c9/` returns zero matches. | Neural network cheats during training by reading simulation parameters instead of learning physics. |
| **T33** | C9 Digital Twin | **Parameter Convergence:** At Day 40, estimated subsidence parameters match ground truth within strict bounds: $\hat{a}\cdot\hat{\eta}(t) ≤ 10\%$, $\hat{a} ≤ 15\%$, $\hat{c} ≤ 25\%$. | PINN fails to converge, providing invalid advisory surface interpolation to SCADA. |
| **T34** | C9 Digital Twin | **Datum Pinning:** Disabling the Bedrock Anchor loss term ($\mathcal{L}_{\text{anchor}}$) in the PINN causes the reconstructed mean surface elevation to drift by $> 10\text{ cm}$. | Reconstructed 3D terrain floats freely in space without an absolute geological datum. |
| **T35** | C9 Digital Twin | **Smooth Activation Mandate:** Replacing smooth $\tanh$ or Swish activations with ReLU forces strain loss to zero, terminating second-derivative backpropagation. | Second spatial derivative vanishes ($\partial^2 \text{ReLU} / \partial x^2 \equiv 0$), destroying physical strain training. |
| **T36** | C9 Digital Twin | **Temporal Window Sufficiency:** The PINN training dataset spans $≥ 12.0\text{ hours}$ of distinct time slices. | Time decay coefficient $\hat{c}$ becomes mathematically unidentifiable from static snapshots. |
| **T37** | Ingestion Engine | **Cryptographic Reproducibility:** The `generator_sha256` field in the simulation header strictly matches the SHA-256 hash of the generating source script. | Audit failure; inability to legally prove the provenance and reproducibility of simulation benchmarks. |
| **T38** | Physics Engine | **Precondition Enforcement:** Knothe model preconditions (flat seam dip $< 15^°$, subcritical-to-supercritical width, homogenous cover) asserted at initialization. | Geotechnical model applied outside its valid mathematical and mechanical domain. |
| **T39** | C8 Alarm Engine | **Scheduled Blast Veto:** A Day-3 seismic event matching the DGMS shift blasting schedule in `events.csv` is vetoed (`vetoed_by = BLAST`), emitting NO siren. | Standard production blast triggers an accidental full-mine panic evacuation. |
| **T40** | C8 Alarm Engine | **Unlogged Explosive Alarm:** An unlogged high-energy seismic shockwave trips a **CLASS-A / UNLOGGED_BLAST** alarm within 1.0 second. | Unmonitored explosives breach or illegal mining activity ignored. |
| **T41** | C8 Alarm Engine | **Instantaneous Collapse Detection:** A catastrophic multi-node sudden drop event trips **CLASS-A** within one single epoch ($< 1.4\text{ seconds}$). | Rapid catastrophic roof collapse missed due to moving-average temporal smoothing lags. |
| **T42** | C8 Alarm Engine | **Byzantine Quorum Protection:** A single defective sensor injecting a $20\sigma$ anomalous strain spike produces NO evacuation alarm (Spatial Quorum Veto). | **A smashed sensor or chewed cable evacuates an entire active coalfield.** |
| **T43** | C8 Alarm Engine | **Fail-Safe Health Exclusion:** Any node reporting `selftest_ok = 0` or frozen voltage is instantly quarantined from the safety quorum. | **A dead or failing sensor falsely reports an "All Clear" signal, masking actual collapse.** |
| **T44** | Safety Governance | **Hard Architectural Isolation:** Executing `grep -rn -E "C9|pinn|S_grid" backend/c8/` returns exactly ZERO occurrences. | **Experimental AI code breaches the deterministic safety boundary and compromises the siren.** |
| **T45** | C7 Pipeline | **Uncertainty Growth Ceiling:** The temporal gap multiplier $f_{\text{gap}}$ is strictly capped at **10.0** regardless of offline duration. | Numerical overflow ($NaN$) in covariance matrix calculations following multi-day sensor blackouts. |
| **T46** | C8 Alarm Engine | **Fail-Stale Safety Governance:** If telemetry backhaul is lost for $> 180\text{ seconds}$, dashboard state transitions to **STALE**; C8 emits NO Quiet status. | A blinded, uncommunicative system falsely reassures operators that ground conditions are safe. |

---

### Question: Which four tests represent the non-negotiable architectural gatekeepers of the platform, and why?

**Answer:** While all 46 tests must pass in continuous integration, four specific tests act as the load-bearing pillars of AEGIS:

1. **Test T7 (Calibration Invertibility):** Proves that the physics-based calibration pipeline can remove environmental distortions (temperature swings, mechanical sag, battery drop) and reconstruct pristine physical strain without introducing artificial bias.
2. **Test T29 (Conservation of Telemetry):** Proves mathematically that across the multi-hop wireless mesh, edge ring buffers, and local SQLite rolling stores, not a single bit of telemetry vanishes unaccounted for.
3. **Test T42 (Byzantine Fault Tolerance):** Guarantees through multi-node spatial quorum voting that a single sensor struck by an excavator, struck by rockfall, or shorted by rainwater can never trigger a false mine evacuation.
4. **Test T43 (Failure-to-Safety):** Guarantees that a malfunctioning or dead sensor is immediately stripped of its voting credentials, preventing a dormant or frozen device from reassuring managers that a subsiding panel is safe.
