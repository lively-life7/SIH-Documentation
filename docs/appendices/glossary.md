# Technical Glossary: Plain Engineering Definitions

**Appendices**  
**Cross-References:** [`acronyms.md`](acronyms.md) · [`constants-reference.md`](constants-reference.md) · [Module 00 Index](../00-executive-gateway/system-architecture.md)

---

### Question: What are the foundational physical and geological definitions governing ground subsidence and mining-induced strata mechanics?

**Answer:** The following terms define the physical ground deformation phenomena monitored across active mining panels:

| Technical Term | Authoritative Engineering Definition |
| :--- | :--- |
| **Subsidence ($S$)** | The vertical sinking, depression, and three-dimensional displacement of the ground surface caused by subterranean coal seam extraction and subsequent goaf compaction. |
| **Extraction Panel** | A defined rectangular subterranean block of coal bounded by development gate roads, extracted systematically via mechanized longwall shearing or continuous miner depillaring. |
| **Subsidence Trough** | The continuous dish-shaped depression or basin that forms on the terrain surface above and surrounding an extracted coal panel. |
| **Knothe Model** | The classical Gaussian influence function formulation predicting the spatial profile shape and exponential time-dependent relaxation rate ($\eta(t) = 1 - e^{-ct}$) of surface subsidence. |
| **Influence Radius ($r$)** | The horizontal distance extending outward from the extraction boundary to the asymptotic edge of detectable ground movement, defined by $r = H / \tan\beta$. |
| **Angle of Draw ($\beta$)** | The angle measured from the vertical defining the outer boundary of ground movement above an underground extraction face. |
| **Surface Tilt ($T$)** | The slope angle or spatial gradient of the ground surface ($\partial S / \partial x$), representing the first spatial derivative of vertical displacement. |
| **Horizontal Strain ($ε$)** | The relative stretching (tensile, positive) or compression (compressive, negative) of the ground surface ($\partial U_x / \partial x$), representing the spatial derivative of horizontal displacement. **The primary early warning precursor for surface fissuring.** |
| **Wire Extensometer** | A mechanical instrument consisting of a tensioned invar wire or rod coupled to a precision displacement transducer across ground stakes to measure differential baseline displacement directly. |

---

### Question: What are the physical hardware classifications, node roles, and spatial scaling rules across the sensor mesh?

**Answer:** Field hardware is organized into a disciplined three-tier hierarchy scaled algorithmically to panel geometry:

| Hardware Entity | Authoritative Engineering Definition & Structural Role |
| :--- | :--- |
| **Scout Node (Tier 1)** | A self-contained, solar-powered field station deployed across the panel carrying MEMS sensors (tilt, strain, extensometer), an ESP32 microcontroller, and an SX1262 LoRa radio. Unit cost is approximately ₹1,050 to ₹1,400. |
| **Anchor Relay (Tier 2)** | A ruggedized field station acting as a cluster head and backbone mesh relay. Anchors aggregate child Scout packets (nominally 1 Anchor per 5 Scouts) and forward bundled telemetry to the gateway over dedicated trunk slots. |
| **Bedrock Anchor ($A_1, A_2$)** | A specialized Tier 2 reference station installed outside the subsidence basin ($d ≥ 1.5r$) into undisturbed bedrock, providing the absolute elevation datum and common-mode rejection baseline. |
| **Master Gateway (Tier 3)** | The central surface communications and safety hub equipped with a 10m pneumatic mast, SX1302 8-channel concentrator, cellular NB-IoT backhaul, and a deterministic hardware dry-contact relay driving a 125 dB evacuation siren. |
| **Dynamic Array Scaling** | Network size scales dynamically by panel geometry and influence radius $r$ (e.g., a 37-node pilot sub-slice covering $600\text{ m} × 200\text{ m}$ up to a 409-node full-panel array for a $1.8\text{ km}$ longwall) without arbitrary ceilings. |

---

### Question: What are the core radio frequency and medium access control definitions governing telemetry transmission?

**Answer:** Telecommunications parameters are strictly optimized for low power, long range, and statutory radio spectrum compliance:

| Telecommunications Term | Authoritative Engineering Definition |
| :--- | :--- |
| **LoRa (Long Range)** | A proprietary chirp spread spectrum (CSS) sub-GHz radio modulation technique optimized for extreme link budget and low power consumption in industrial environments. |
| **Spreading Factor (SF)** | The number of chirps used per data symbol in LoRa modulation. Lower spreading factors (SF7) yield shorter transmission airtime; higher factors (SF8) provide enhanced link margin for trunk hops. |
| **Carrier Bandwidth (BW)** | The frequency width of the RF transmission channel. Configured strictly at **125.0 kHz** across all presets to satisfy Indian statutory bandwidth ceilings ($≤ 200\text{ kHz}$). |
| **Transmission Airtime** | The physical on-air duration of a radio packet ($90.4\text{ ms}$ for SF7 leaf packets, $406.0\text{ ms}$ for SF8 relay bundles), representing the primary determinant of battery consumption. |
| **Duty Cycle** | The percentage of time a radio transmitter actively occupies the shared RF spectrum. Strictly self-enforced at $< 1.0\%$ across all nodes to prevent channel congestion. |
| **TDMA (Time-Division Multiple Access)** | A deterministic MAC scheduling protocol where each node transmits exclusively in a pre-assigned microsecond time slot, completely eliminating co-channel packet collisions. |
| **Superframe** | The repeating 60-second TDMA scheduling cycle comprising a gateway synchronization beacon, cluster leaf uplink slots, backbone relay trunk slots, and quiet guard intervals. |
| **Bitmap ACK** | A single broadcast packet emitted by an Anchor or Gateway that acknowledges up to 8 child transmissions simultaneously using an 8-bit mask, reducing downlink airtime by 87%. |
| **Packet Delivery Ratio (PDR)** | The percentage of scheduled telemetry frames successfully captured and persisted by the Master Gateway across the operational duration (statutory threshold $≥ 99.5\%$). |

---

### Question: What are the definitions and structural boundaries governing backend data pipelines, digital twins, and safety alarms?

**Answer:** Backend processing enforces strict isolation between deterministic safety-critical alerting and advisory AI surface interpolation:

| Computational Entity | Authoritative Engineering Definition & Safety Boundary |
| :--- | :--- |
| **Epoch** | A synchronized 32-bit integer counter incremented every 60 seconds since scenario initialization, serving as the master temporal key across the platform. |
| **C7 Corrector** | The backend 8-step calibration pipeline that converts raw degraded bitstreams into clean physical engineering units, compensates for thermal drift and mechanical sag, and computes dynamic uncertainty $\sigma_{\text{final}}$. |
| **C8 Safety Detector** | The deterministic, 100% auditable alarm engine holding exclusive statutory authority to trip the 125 dB evacuation siren via multi-node spatial quorum voting within $< 1.4\text{ seconds}$. |
| **C9 Digital Twin** | An advisory scientific machine learning pipeline that interpolates sparse sensor points into a continuous $64 × 64$ surface grid for SCADA visualization. **Holds zero alarm authority.** |
| **PINN (Physics-Informed Neural Network)** | A deep neural network trained using automatic differentiation to minimize both data residuals and the Knothe partial differential equation loss, enforcing physical consistency without truth leakage. |
| **LOO Residual** | Leave-One-Out cross-validation residual: A validation metric computed by systematically withholding one sensor node and calculating the neural network's interpolation error at that coordinate. |
| **$d_{\mathbf{committed}}$ (Largest Empty Circle)** | The diameter of the largest circular area on the surface panel lacking sensor instrumentation ($358\text{ m}$), published directly on the SCADA header to prevent false assumptions of zero-blind-spot coverage. |
| **Rolling Store (`nodes.csv`)** | A circular telemetry buffer maintained on the edge gateway disk strictly bounded to 36 hours of historical readings to prevent storage exhaustion (Test T30). |
| **DGMS** | Directorate General of Mines Safety: The statutory regulatory authority under the Ministry of Labour and Employment governing occupational health and safety across Indian mines. |
