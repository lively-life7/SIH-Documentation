# Technical Glossary: Plain Engineering Definitions

**Appendices**  
**Cross-References:** [`acronyms.md`](acronyms.md) · [`constants-reference.md`](constants-reference.md) · [Module 00 Index](../00-executive-gateway/system-architecture.md)

---

| Term | Plain Technical Meaning |
| :--- | :--- |
| **Subsidence** | Vertical sinking, depression, and lateral displacement of the ground surface caused by subterranean coal extraction. |
| **Panel** | A defined rectangular subterranean block of coal bounded by development headings, extracted via longwall or depillaring methods. |
| **Subsidence Trough** | The continuous dish-shaped depression that forms on the terrain surface above and around an extracted coal panel. |
| **Knothe Model** | The classical mathematical influence formulation predicting the spatial shape and temporal rate of surface subsidence troughs. |
| **Influence Radius ($r$)** | The horizontal distance past the edge of the extraction panel to which ground movement extends ($r = H / \tan\beta$). |
| **Angle of Draw ($\beta$)** | The angle measured from the vertical defining the outer boundary of ground movement above an extraction face. |
| **Surface Tilt ($T$)** | The slope angle or gradient of the ground surface ($\partial S / \partial x$), representing the first spatial derivative of displacement. |
| **Horizontal Strain ($\varepsilon$)** | The relative stretching or compression of the ground surface ($\partial U_x / \partial x$), representing the second derivative of displacement. **The primary early warning precursor.** |
| **Wire Extensometer** | A tensioned invar wire or rod stretched between two ground pegs to measure differential 3D baseline displacement directly. |
| **Scout Node** | A self-contained, solar-powered field station (Tier 1) deployed across the panel carrying sensors, microcontroller, and LoRa radio. |
| **Anchor Relay** | A field station (Tier 2) acting as a cluster head and mesh relay, forwarding aggregated child packets to the gateway over a backbone trunk. |
| **Bedrock Anchor** | A fixed reference station installed outside the influence basin ($> 1.5r$) into undisturbed bedrock, providing the zero-displacement baseline. |
| **Master Gateway** | The central communications hub (Tier 3) equipped with a 10m mast, SX1302 concentrator, cellular modem, and hardware siren relay. |
| **LoRa (Long Range)** | A proprietary chirp spread spectrum (CSS) radio modulation technique optimized for low power and long-distance telemetry. |
| **Spreading Factor (SF)**| The number of chirps used per data symbol in LoRa modulation. Lower SF (SF7) is fast with short airtime; higher SF (SF8) provides longer range. |
| **Bandwidth (BW)** | The frequency width of the RF carrier signal. Fixed at **125 kHz** to comply with Indian statutory ceilings ($\le 200\text{ kHz}$). |
| **Airtime** | The physical on-air transmission duration of a wireless packet, representing the primary energy and channel capacity constraint. |
| **Duty Cycle** | The percentage of time a radio transmitter occupies the shared RF spectrum. Self-imposed at $\le 1.0\%$ across all AEGIS nodes. |
| **TDMA** | Time-Division Multiple Access: A collision-free scheduling protocol where each node transmits only in its pre-assigned micro-time slot. |
| **Epoch** | A 60-second synchronized time counter since scenario startup, serving as the master identity key for all telemetry rows. |
| **Superframe** | The repeating 60-second TDMA scheduling cycle comprising beacon sync, cluster uplinks, backbone trunks, and quiet periods. |
| **Bitmap ACK** | A single broadcast packet acknowledging multiple child transmissions simultaneously using a bitmask, cutting downlink airtime by 87%. |
| **PDR** | Packet Delivery Ratio: The percentage of successfully received packets over a transmission channel. |
| **PINN** | Physics-Informed Neural Network: A neural network constrained by differential physical laws that reconstructs continuous 3D terrain without alarm authority. |
| **LOO Residual** | Leave-One-Out cross-validation residual: An accuracy metric computed by masking one node and evaluating prediction error. |
| **$d_{\text{committed}}$** | The diameter of the Largest Empty Circle ($358\text{m}$) between sensor stations, representing the published spatial detection limit. |
| **DGMS** | Directorate General of Mines Safety: The statutory regulatory body governing mine occupational safety in India. |
| **C7 Corrector** | The backend 8-step calibration pipeline that converts raw degraded bitstreams into clean physical engineering units. |
| **C8 Safety Detector** | The deterministic, 100% mathematically auditable alarm engine holding exclusive authority to trip evacuation sirens. |
| **C9 Digital Twin** | The purely advisory AI pipeline that interpolates sparse field points into a dense 64×64 surface mesh for 3D SCADA visualization. |
| **Rolling Store** | The 36-hour bounded telemetry buffer (`nodes.csv`, ~8 MB) maintained on disk to guarantee fixed storage consumption. |
