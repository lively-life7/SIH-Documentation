* [Introduction / Overview](README.md)

* **Module 00: Executive Gateway**
  * [Key Metrics & Engineering Scorecard](docs/00-executive-gateway/key-metrics-summary.md)
  * [Project Charter: AEGIS Platform](docs/00-executive-gateway/project-charter.md)
  * [System Architecture & Topology](docs/00-executive-gateway/system-architecture.md)

* **Module 01: Ground Reality & Crisis**
  * [The Crisis Landscape: Mining Ground Hazards & Subsidence](docs/01-ground-reality/crisis-landscape.md)
  * [Evaluation of Existing Monitoring Approaches](docs/01-ground-reality/existing-approaches.md)
  * [The Engineering Opportunity: Real-Time Indigenous Mine Safety](docs/01-ground-reality/opportunity-statement.md)

* **Module 02: Sensor Hardware & Edge**
  * [Bill of Materials (BOM) & Modular Cost Model](docs/02-sensor-hardware/bill-of-materials.md)
  * [Edge Intelligence & Vibration Discrimination](docs/02-sensor-hardware/edge-intelligence.md)
  * [Node Classification & Hardware Hierarchy](docs/02-sensor-hardware/node-classification.md)
  * [Power Architecture & Energy Budget](docs/02-sensor-hardware/power-and-energy.md)
  * [The 7-Sensor Suite & Transducer Specifications](docs/02-sensor-hardware/sensor-suite.md)

* **Module 03: Mesh Networking (TDMA)**
  * [Grid Spacing Rationale: Physics-Driven vs. Radio-Driven](docs/03-mesh-networking/grid-spacing-rationale.md)
  * [Protocol Selection & Wireless Decision Matrix](docs/03-mesh-networking/protocol-selection.md)
  * [Multi-Frequency DAG Routing & Autonomous Failover](docs/03-mesh-networking/routing-and-failover.md)
  * [Radio Spectrum Regulatory Compliance (GSR 564(E))](docs/03-mesh-networking/spectrum-compliance.md)
  * [Store-and-Forward Flash Buffer & Zero Data Loss Guarantee](docs/03-mesh-networking/store-and-forward.md)
  * [TDMA Superframe Scheduling & Collision Avoidance](docs/03-mesh-networking/tdma-scheduling.md)
  * [Binary Wire Format & Packet Structure](docs/03-mesh-networking/wire-format.md)

* **Module 04: Physics Engine (Knothe)**
  * [The 6-Stage Sensor Corruption Chain](docs/04-physics-engine/corruption-chain.md)
  * [Derived Quantities & Kinematic Precursor Detection](docs/04-physics-engine/derived-quantities.md)
  * [Ground Truth Generation & Provenance Quarantine](docs/04-physics-engine/ground-truth-generation.md)
  * [The Knothe Subsidence Model & Mathematical Foundations](docs/04-physics-engine/knothe-model.md)

* **Module 05: AI/ML Pipeline (PINN)**
  * [Composite Loss Formulation & Boundary Constraints](docs/05-ai-ml-pipeline/loss-formulation.md)
  * [Physics-Informed Neural Network (PINN) Architecture](docs/05-ai-ml-pipeline/pinn-architecture.md)
  * [The Safety Boundary: Hard Architectural Firewall Between C8 and C9](docs/05-ai-ml-pipeline/safety-boundary.md)
  * [3D Continuous Surface Reconstruction & Spatial Validation](docs/05-ai-ml-pipeline/surface-reconstruction.md)
  * [The PINN Training Contract & Identifiability Constraints](docs/05-ai-ml-pipeline/training-contract.md)

* **Module 06: Backend & Alarm Engine**
  * [C7 Corrector: The 8-Step Calibration & Cleaning Pipeline](docs/06-backend-pipeline/c7-corrector.md)
  * [C8 Safety Alarm Engine & Byzantine Quorum Gating](docs/06-backend-pipeline/c8-alarm-engine.md)
  * [Backend Data Architecture & Ingestion Schemas](docs/06-backend-pipeline/data-architecture.md)
  * [Pipeline Flow: End-to-End Execution & Data Contracts](docs/06-backend-pipeline/pipeline-flow.md)

* **Module 07: Digital Twin & 3D UI**
  * [3D SCADA Digital Twin & Terrain Visualization](docs/07-digital-twin-and-ui/3d-visualization.md)
  * [Automated Alert Dispatch & Siren Activation System](docs/07-digital-twin-and-ui/alert-system.md)
  * [Operator Mission Control Dashboard & Telemetry Inspector](docs/07-digital-twin-and-ui/operator-dashboard.md)
  * [Web, Desktop & Mobile Platform Architecture](docs/07-digital-twin-and-ui/web-mobile-platform.md)

* **Module 08: Verification & Field Test**
  * [The 6-Day Staged Build Order & Freezing Protocol](docs/08-verification/build-order.md)
  * [Field Validation & Coalfield Pilot Trial Plan](docs/08-verification/field-validation-plan.md)
  * [The Global Verification Test Register (T1 – T46)](docs/08-verification/test-register.md)

* **Module 09: Deployment & Scalability**
  * [Cost-Benefit Analysis & Economic Impact](docs/09-deployment-and-impact/cost-benefit-analysis.md)
  * [Field Installation & Commissioning Procedures](docs/09-deployment-and-impact/installation-procedure.md)
  * [National Alignment: Atmanirbhar Bharat & Smart Mining](docs/09-deployment-and-impact/national-alignment.md)
  * [Regulatory Compliance & Statutory Alignment](docs/09-deployment-and-impact/regulatory-compliance.md)
  * [Horizontal Scalability & Enterprise Architecture](docs/09-deployment-and-impact/scalability.md)

* **Appendices & References**
  * [Table of Acronyms & Abbreviations](docs/appendices/acronyms.md)
  * [Citation Ledger: Authoritative Technical & Regulatory Sources](docs/appendices/citation-ledger.md)
  * [Constants Reference Ledger (`sim/constants.py`)](docs/appendices/constants-reference.md)
  * [Technical Glossary: Plain Engineering Definitions](docs/appendices/glossary.md)
  * [Dynamic Measurement Uncertainty ($\sigma$) Derivations](docs/appendices/sigma-formula.md)
