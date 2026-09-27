# Pipeline Flow: End-to-End Execution & Data Contracts

**Module 06 — Backend Pipeline**  
**Cross-References:** [`data-architecture.md`](data-architecture.md) · [`c7-corrector.md`](c7-corrector.md) · [`c8-alarm-engine.md`](c8-alarm-engine.md)

---

## 1. End-to-End Pipeline Execution Flow

The AEGIS backend coordinates incoming wireless telemetry, calibration transformations, deterministic hazard detection, and 3D terrain reconstruction across a deterministic execution graph:

```mermaid
flowchart TD
    subgraph INGEST_LANE["1. Wire Ingestion Lane"]
        WIRE["23-Byte Binary Stream<br/>(90.4 ms LoRa Airtime @ SF7)"] --> UNPACK["unpack_wire_frame()<br/>Validates CRC-16 & Node ID"]
        UNPACK --> CSV[("nodes.csv<br/>36h Rolling Store (~8 MB)")]
    end

    subgraph CORRECTION_LANE["2. C7 Calibration Pipeline"]
        CSV --> C7["c7_corrector.py<br/>8-Step Sequential Transformation"]
        JSON["nodes.json<br/>Geometry & Calibration Manifest"] --> C7
        C7 --> CALIB[("calibrated_state<br/>Dict: {node_id, epoch, SI_units, σ, valid_mask}")]
    end

    subgraph SAFETY_LANE["3. C8 Deterministic Safety Engine (EXCLUSIVE ALARMS)"]
        CALIB --> C8["c8_alarm_engine.py<br/>Knothe least-squares fit & 5-node Byzantine Quorum"]
        EVENTS["events.csv<br/>DGMS blast & lightning register"] --> C8
        C8 --> SIREN["Physical Evacuation Siren<br/>(<1.4s hardware relay contact)"]
        C8 --> SMS["Automated Geotech SMS Tree"]
        C8 --> ALARM_EVT["alarm_events JSON Stream"]
    end

    subgraph TWIN_LANE["4. C9 PINN Digital Twin (PURELY ADVISORY)"]
        CALIB --> C9["c9_pinn_model.py<br/>Sparse-to-Dense Mesh Interpolator"]
        C9 --> MESH[("64 × 64 Surface Mesh<br/>JSON / WebSocket Stream")]
        C9 -.->|"HARD ARCHITECTURAL FIREWALL: ZERO ACCESS"| C8
    end

    subgraph HMI_LANE["5. Mission Control SCADA"]
        ALARM_EVT --> DASH["3D Operator Dashboard<br/>React + Three.js / Electron Desktop"]
        MESH --> DASH
    end
```

---

## 2. Stage-by-Stage Latency & Data Contract Ledger

| Stage | Input Data Structure | Output Data Structure | Typical Latency | Primary Failure Guard |
| :--- | :--- | :--- | :--- | :--- |
| **Ingestion** | 23-byte binary LoRa frame | Row in `nodes.csv` (120 bytes) | $< 15\text{ ms}$ | CRC-16 error rejection; de-dup ring |
| **C7 Cleaning**| Raw `nodes.csv` + `nodes.json` | Calibrated state dictionary (SI units) | $< 35\text{ ms}$ | Sag undo prior to thermal subtraction |
| **C8 Quorum** | Calibrated state + `events.csv` | Alarm classification (Quiet/Warn/Crit) | $< 10\text{ ms}$ | 5-node Byzantine quorum + DGMS blast veto |
| **Siren Action**| Dry-contact relay command | 125 dB physical audio siren | $< 250\text{ ms}$ | Local hardware latch; independent of internet |
| **C9 Twin** | Calibrated state (48 time slices) | $64 \times 64$ elevation & strain mesh | $\sim 45\text{ s}$ (Async)| PINN decoupled from real-time alarm loop |

### Inviolable Firewall Check:
The line from C9 to C8 is non-existent. At no point in the code can C9 outputs, predictions, or tensor weights be imported or read by C8 routines, ensuring complete DGMS regulatory auditability.
