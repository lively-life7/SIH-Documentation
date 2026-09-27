# The Safety Boundary: Hard Architectural Firewall Between C8 and C9

**Module 05 — AI/ML Pipeline**  
**Cross-References:** [`pinn-architecture.md`](pinn-architecture.md) · [Module 00 System Boundaries](../00-executive-gateway/system-architecture.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md)

---

## 1. The Core Architectural Firewall

The most critical architectural principle of the AEGIS platform is the **absolute separation of analytical visualization from life-safety alarm authority**.

```
                           RAW FIELD TELEMETRY
                                    │
               ┌────────────────────┴────────────────────┐
               ▼                                         ▼
   [ C8 DETERMINISTIC DETECTOR ]              [ C9 PINN DIGITAL TWIN ]
   - Classical Knothe curve fit               - Physics-Informed Neural Net
   - 5-Node Byzantine Quorum Gate             - Sparse-to-dense interpolation
   - DGMS blast vibration filter              - 64 × 64 terrain mesh generator
   - 100% Mathematically Auditable            - Probabilistic trend scoring
               │                                         │
               ▼                                         ▼
  ★ EXCLUSIVE ALARM AUTHORITY ★                [ ADVISORY SCADA HMI ]
  - Trips Field Sirens (<1.4s)                 - Color-coded surface mesh
  - Sends Emergency Evacuation SMS             - Forward +72h trend display
  - Relays to Mine Control Room                - ZERO ALARM AUTHORITY
               │                                         │
               └─────────── NO CROSSOVER ────────────────┘
               (Enforced by CI Static Analysis Test T44)
```

---

## 2. Why Neural Networks Are Excluded from Safety Alarms

Placing deep learning models or neural networks in the active trip circuit of an industrial life-safety system introduces unacceptable hazards:

1. **Statutory Certification (DGMS & CMR 2017):**
   Under the Mines Act 1952 and the Coal Mines Regulations (CMR) 2017, safety-critical tripping mechanisms must be deterministic, transparent, and auditable. An inspector cannot certify floating-point matrix weights whose internal decision logic cannot be mathematically traced during an accident inquiry.
2. **Out-of-Distribution Hallucination:**
   Deep neural networks perform unpredictably when exposed to edge-case anomalies (e.g., three sensors struck by lightning, anomalous ground water bursts, or temporary radio packet bursts). A neural network can experience gradient instability and either hallucinate a non-existent disaster or suppress a genuine catastrophic failure signal.
3. **Analogy for Regulators:**
   * **C8 (Classical Detector) = The Police Officer:** Authoritative, transparent, enforces unambiguous statutory laws, and takes immediate physical action.
   * **C9 (PINN Neural Net) = The Meteorologist:** Projects future trends, evaluates atmospheric conditions, and provides advisory forecasts, but holds zero power to declare a mandatory evacuation.

---

## 3. Strict Architectural Isolation (Test T44)

The isolation between C8 and C9 is not an operational guideline; it is enforced in code:
* The C8 alarm engine codebase (`backend/c8/`) contains **zero imports, function calls, or data structures** referencing C9.
* Automated CI test `T44` executes a static analysis check across the codebase:
  ```bash
  grep -rn "C9\|pinn\|S_grid" backend/c8/
  ```
  If any reference to C9 or the PINN surface grid appears within `backend/c8/`, the CI pipeline immediately fails the build.

The evacuation sirens can only be triggered by the deterministic, transparent mathematical algorithms of C8.
