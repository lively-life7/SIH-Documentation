# Regulatory Compliance & Statutory Alignment

**Module 09 — Deployment & Impact**  
**Cross-References:** [`national-alignment.md`](national-alignment.md) · [Module 03 Spectrum Compliance](../03-mesh-networking/spectrum-compliance.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md)

---

### Question: How does AEGIS satisfy the statutory mandates of the Directorate General of Mines Safety (DGMS), the Mines Act, 1952, and Indian telecommunication laws?

**Answer:** AEGIS is architected to achieve complete legal and technical compliance across Indian mining safety and wireless communications statutes:

| Regulatory Body / Act | Specific Regulation / Standard | Statutory Mandate | AEGIS Architectural Compliance |
| :--- | :--- | :--- | :--- |
| **DGMS** | **Coal Mines Regulations (CMR) 2017, Reg. 104** | Precautions against danger of inundation from surface water bodies and overlying aquifers. | High-density ($15\text{–}25\text{ m}$) sensor grid detects tensile strain inflection ($\varepsilon \ge 1500\ \mu\varepsilon$) over confining shale aquitards, alerting engineers before fractures breach water bodies. |
| **DGMS** | **Coal Mines Regulations (CMR) 2017, Reg. 106** | Ground movement surveillance and bench stability across active mechanized extraction faces. | Automated continuous micro-strain, tilt ($2\ \mu\text{rad}$ resolution), and baseline extensometer logging along active face advance profiles. |
| **DGMS** | **Coal Mines Regulations (CMR) 2017, Reg. 111** | Mandatory formulation and execution of a Strata Control and Monitoring Plan (SCAMP). | Delivers continuous, tamper-proof digital telemetry records, completely replacing error-prone manual paper survey books. |
| **DGMS** | **Coal Mines Regulations (CMR) 2017, Reg. 112** | Continuous surveillance over extracted goaves, depillaring districts, and subsidence basins. | Multi-hop wireless LoRa mesh maintains 60-second telemetry streaming across active, extracted, and abandoned goaf boundaries. |
| **DGMS** | **DGMS Circular No. 7 of 1997 (Tech)** | Maximum permissible Peak Particle Velocity (PPV) thresholds and mandatory blasting ledgers. | High-speed 3-axis accelerometer on Anchor Relays captures blast shockwaves, cross-referencing against the statutory shift blasting log (`events.csv`). |
| **WPC / Min. of Comm.** | **Gazette Notification GSR 564(E)** | Delicensed 865–867 MHz band; maximum carrier bandwidth $\le 200\text{ kHz}$; conducted power $\le 1\text{ W}$. | Operates strictly at **125.0 kHz carrier bandwidth** (Test T18) and $30.0\text{ dBm}$ (1.0 W conducted power), fully complying with national spectrum law. |
| **Ministry of Labour** | **The Mines Act, 1952 (Section 22)** | Statutory power of inspectors to halt extraction in conditions of imminent danger to human life. | Deterministic C8 Safety Engine trips a 125 dB physical evacuation siren in $< 1.4\text{ seconds}$ upon verified multi-node spatial quorum breach. |
| **Bureau of Indian Stds.** | **IS 14881:2001** | Standard guidelines for design and installation of ground monitoring instrumentation. | IP67 water/dust ingress protection, high-tensile steel rebar foundation anchoring, and mechanical temperature-compensated invar extensometry. |

---

### Question: How does AEGIS implement the vibration criteria of DGMS Circular No. 7 of 1997, and how does it differentiate production blasting from catastrophic collapse?

**Answer:** Under DGMS Circular No. 7 of 1997, maximum allowable Peak Particle Velocity (PPV) for surface structures overlying coal workings is strictly regulated by dominant frequency:
* **Domestic Mud / Unreinforced Structures:** $\text{PPV} < 5.0\text{ mm/s}$ (for dominant frequency $< 8.0\text{ Hz}$); $\text{PPV} < 10.0\text{ mm/s}$ ($8.0\text{ to }25.0\text{ Hz}$).
* **Industrial Substations / Reinforced Concrete:** $\text{PPV} < 15.0\text{ mm/s}$ ($< 25.0\text{ Hz}$).

To avoid catastrophic false evacuation alarms during scheduled blasting:
1. **Automated Blast Shift Ingestion (Test T39):**
   The colliery blasting schedule is ingested into `data/events.csv`. When high-energy transient vibrations occur within a scheduled blast window, the C8 Alarm Engine correlates the vibration signature with the blast manifest, applying a statutory veto (`vetoed_by = BLAST`). The siren remains silent while the blast PPV compliance record is logged to the statutory ledger.
2. **Unlogged Explosive & Dynamic Shockwave Tripping (Test T40):**
   If an unscheduled high-energy shockwave ($\text{PPV} > 15.0\text{ mm/s}$) is detected outside an authorized blasting window, the system trips a **CLASS-A / UNLOGGED_BLAST** alarm within 1.0 second, alerting colliery safety managers to unmonitored explosive use or sudden roof weighting.

---

### Question: How does AEGIS ensure that its digital records are legally defensible and tamper-proof during a statutory DGMS court of inquiry?

**Answer:** In the event of a catastrophic ground failure or fatal roof fall, mining companies frequently face DGMS inquiries where physical paper logs are challenged for post-incident alteration. AEGIS enforces a 3-layer cryptographic and procedural chain of custody:

1. **Immutable Wire-Level Ingestion Log:**
   Every LoRa frame captured by the Master Gateway is written immediately to an append-only transaction log containing the raw 23-byte hexadecimal payload, gateway microsecond arrival timestamp, and physical RF link metrics (RSSI, SNR).
2. **Non-Destructive In-Memory Transformation:**
   Raw telemetry rows are stored permanently in `nodes.csv`. All C7 calibration routines (thermal walk compensation, common-mode rejection subtraction) are computed dynamically in memory. The original raw physical observations are never overwritten, truncated, or zero-filled (Test T28).
3. **Cryptographic SHA-256 Provenance Ledger (Test T37):**
   Configuration manifests (`config/nodes.json`), Python analytical scripts, and historical 36-hour Parquet archive blocks are hashed using SHA-256. Continuous integration checks confirm that historical data files match their recorded cryptographic checksums. Any attempt by mine personnel to retroactively alter past displacement readings breaks the cryptographic hash chain, rendering the tampering immediately visible to DGMS statutory auditors.
