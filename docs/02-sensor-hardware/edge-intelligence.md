# Edge Intelligence & Vibration Discrimination

**Module 02 — Sensor Hardware**  
**Cross-References:** [`sensor-suite.md`](sensor-suite.md) · [Module 03 Wire Format](../03-mesh-networking/wire-format.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md)

---

## 1. The Bandwidth Bottleneck: Edge Feature Extraction

A major challenge in geotechnical monitoring is vibration analysis. Micro-fractures within the overlying strata emit transient acoustic and seismic stress waves prior to macro-scale shear rupture.

However, continuous streaming of raw high-frequency waveforms over a low-power wireless mesh network is physically impossible:
* A 3-axis accelerometer sampled at a modest $400\text{ Hz}$ generates:
  $$\text{Data Rate} = 400\text{ samples/s} \times 3\text{ axes} \times 2\text{ bytes/sample} = 2,400\text{ Bytes/second}$$
* Over a 60-second window, that represents $144\text{ KB}$ per node.
* In contrast, LoRa at SF7 / 125 kHz provides an effective channel throughput of only $\sim 5.4\text{ kbps}$, legally bounded by a $1\%$ duty-cycle ceiling ($\sim 6.8\text{ Bytes/second}$ average throughput per node).

Attempting to stream raw waveform data would collapse the RF channel within seconds.

AEGIS solves this via **On-Node Edge Spectral Classification**:
1. During each 60-second superframe, the ESP32 samples a $640\text{ ms}$ high-speed vibration burst (256 points @ $400\text{ Hz}$).
2. The onboard floating-point unit runs a 256-point real Fast Fourier Transform (FFT) in $1.5\text{ ms}$.
3. The spectrum is collapsed into three high-value scalar features:
   * `vib_rms_x100` (uint16): Root-Mean-Square vibration intensity ($0.01\text{ mm/s}$ resolution).
   * `vib_peak_x100` (uint16): Peak Particle Velocity ($0.01\text{ mm/s}$ resolution).
   * `vib_fdom_hz` (uint8): Dominant energy frequency ($1\text{ Hz}$ bins, $0\text{ to }200\text{ Hz}$).
4. This compresses $144,000\text{ bytes}$ of raw data into **5 bytes** on the wire—a compression ratio of **$28,800:1$** while preserving the diagnostic frequency fingerprint.

---

## 2. 4-Band Spectral Discrimination

Different industrial and geological activities produce distinct vibration spectra. The dominant frequency (`vib_fdom_hz`) allows the backend engine to classify the ground disturbance:

```
+-----------------------------------------------------------------------------------+
|                        4-BAND VIBRATION DISCRIMINATION                            |
|                                                                                   |
|  [ 8 – 20 Hz ]      [ 40 – 80 Hz ]      [ 50 Hz ±0.5 ]      [ 100 – 250 Hz ]      |
|  Dump Trucks &      Production Blasts   Armored Conveyors   Microseismic Rock     |
|  Haul Traffic       (DGMS Verified)     Harmonic Notch      Tensile Fractures     |
|  --> VETO           --> VETO            --> NOTCH           --> ESCALATE          |
+-----------------------------------------------------------------------------------+
```

1. **Band 1: Surface Haul Traffic ($8\text{ to }20\text{ Hz}$):**
   Heavy dumper trucks and excavators moving over access roads produce low-frequency surface Rayleigh waves between 8 and 20 Hz. When detected, the engine marks the event as vehicular interference and vetoes false subsidence alarms.
2. **Band 2: Armored Face Conveyors ($50.0\text{ Hz} \pm 0.5\text{ Hz}$):**
   Heavy underground longwall shearers and armored face conveyors driven by 50 Hz mains induction motors radiate a strong, continuous 50 Hz mechanical harmonic. An edge notch filter rejects this stationary machine noise.
3. **Band 3: Production Blasting ($40\text{ to }80\text{ Hz}$):**
   Deep borehole explosive detonations produce transient shock waves with peak particle velocity (PPV) exceeding $15\text{ mm/s}$ in the 40–80 Hz range. The engine cross-references this frequency with the digital DGMS blast register to veto false evacuation alarms.
4. **Band 4: Sub-Surface Rock Fracture ($100\text{ to }250\text{ Hz}$):**
   The brittle shear and tensile rupture of overlying sandstone beds emits high-frequency microseismic acoustic bursts between 100 Hz and 250 Hz. Energy in this band represents genuine rock mass delamination, triggering immediate priority escalation.

---

## 3. Preservation of Safety Authority

While feature extraction occurs at the edge, **the node has zero authority to trip an evacuation alarm**. The node firmware only packages the extracted spectral parameters (`rms`, `peak`, `fdom`) into the telemetry packet. All threshold comparisons, quorum agreements, and alarm triggers occur inside the centralized C8 safety engine.
