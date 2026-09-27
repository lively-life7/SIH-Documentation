# Regulatory Compliance & Statutory Alignment

**Module 09 — Deployment & Impact**  
**Cross-References:** [`national-alignment.md`](national-alignment.md) · [Module 03 Spectrum Compliance](../03-mesh-networking/spectrum-compliance.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md)

---

## 1. Compliance Matrix: Indian Mining & Telecommunications Statutes

The AEGIS monitoring platform is architected to satisfy all applicable Indian statutory regulations governing mine safety, explosives vibration control, and wireless radio emissions:

| Regulatory Body / Act | Specific Statute / Regulation | Statutory Requirement | AEGIS Architectural Compliance |
| :--- | :--- | :--- | :--- |
| **Directorate General of Mines Safety (DGMS)** | **Coal Mines Regulations (CMR) 2017, Reg. 111** | Mandatory Strata Control and Monitoring Plan (SCAMP) | Provides continuous digital surface displacement and strain logging. |
| **DGMS** | **Coal Mines Regulations (CMR) 2017, Reg. 112** | Continuous surveillance over extracted goaves and aquifers | High-density 15–25m grid detects tensile fissuring over aquifers. |
| **DGMS** | **DGMS Circular No. 7 of 1997** | Ground vibration monitoring and mandatory blast record keeping | Automated ingestion of `events.csv` blast log; PPV threshold checks. |
| **WPC / Min. of Comm.**| **Gazette Notification GSR 564(E)** | RF emission limits in 865–867 MHz band; bandwidth $\le 200\text{ kHz}$ | Operates at 125 kHz BW; transmit power $\le 30\text{ dBm}$ (Test T18). |
| **Ministry of Labour** | **The Mines Act, 1952 (Sec. 22)** | Power to prohibit extraction in dangerous conditions | Deterministic C8 engine trips automated siren (<1.4s) on critical breach. |
| **Indian Standard** | **IS 14881:2001** | Guidelines for design and construction of ground monitoring systems | Stainless steel anchoring pegs, IP67 dust/water protection. |

---

## 2. DGMS Circular 7 of 1997: Blast Vibration Verification

Under DGMS Circular 7 of 1997, maximum allowable Peak Particle Velocity (PPV) for industrial structures overlying mining workings is strictly bounded:
* Overlying mud/brick residential structures: $\text{PPV} < 5.0\text{ mm/s}$ (dominant frequency $< 8\text{ Hz}$) to $10.0\text{ mm/s}$ ($8\text{ to }25\text{ Hz}$).
* Industrial substations / concrete buildings: $\text{PPV} < 15.0\text{ mm/s}$.

AEGIS co-locates 3-axis high-speed accelerometer bursts on Anchor stations to log peak PPV across every blast shift, writing certified compliance records directly to the database.

---

## 3. Tamper-Proof Digital Audit Trails

In conventional mining operations, ground survey ledgers are recorded in physical paper books vulnerable to post-incident alteration or loss.

AEGIS enforces complete digital accountability:
1. **Immutable Ingestion Log:** Every received packet is logged with its raw 23-byte hex payload, gateway reception timestamp, and RF signal characteristics.
2. **Deterministic Processing:** Raw data is stored untouched in `nodes.csv`. Cleaning transformations occur dynamically in memory, ensuring that the original physical observation is never destroyed.
3. **Cryptographic Validation:** System manifests (`nodes.json`), generator source scripts, and historical Parquet blocks are cryptographically hashed using SHA-256. Any manual alteration of sensor records breaks the hash verification chain in CI.
