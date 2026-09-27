# Edge Intelligence & Vibration Discrimination

**Module 02 — Sensor Hardware**  
**Cross-References:** [`sensor-suite.md`](sensor-suite.md) · [`wire-format.md`](../03-mesh-networking/wire-format.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md) · [Module 00 Key Metrics Summary](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Edge Feature Extraction and Bandwidth Optimization

### Question: Why is continuous streaming of raw high-frequency geotechnical vibration waveforms infeasible over low-power wireless mesh networks in mining environments?

**Answer:** In underground coal mining panels, brittle micro-fractures in the overlying strata generate high-frequency acoustic and seismic stress waves prior to macro-scale rock failure. Accurately capturing these dynamics requires sampling at $\ge 400\text{ Hz}$. However, streaming raw time-series data across a low-power wireless mesh is physically and legally impossible:

1. **Raw Ingestion Bandwidth:**
   A 3-axis accelerometer sampled at $400\text{ Hz}$ with $16\text{-bit}$ ($2\text{ bytes}$) resolution produces:
   $$\text{Data Rate} = 400\text{ samples/sec} \times 3\text{ axes} \times 2\text{ bytes/sample} = 2,400\text{ Bytes/second}$$
   Over a standard 60-second telemetry superframe, a single sensor generates $144\text{ KB}$ of raw uncompressed waveform data.
2. **Channel Capacity Bottleneck:**
   Long-range LoRa modulation operating at Spreading Factor 7 (SF7) with a $125\text{ kHz}$ bandwidth yields an instantaneous physical data rate of only $5.47\text{ kbps}$ ($\approx 684\text{ Bytes/second}$).
3. **Regulatory Ceiling:**
   Under Indian WPC (Wireless Planning & Coordination) regulations, unlicensed sub-GHz transmissions are restricted to an operational duty cycle of $\le 1\%$, limiting a node's legal transmission throughput to:
   $$\text{Maximum Legal Throughput} \le 684\text{ Bytes/second} \times 0.01 = \mathbf{6.84\text{ Bytes/second}}$$
   Streaming raw time-series data would violate spectrum regulations by a factor of $350\times$, saturate the RF channel within seconds, and rapidly deplete node batteries.

### Question: How does AEGIS extract critical seismic and geotechnical features on resource-constrained embedded hardware without losing structural failure signatures?

**Answer:** AEGIS resolves the bandwidth bottleneck by executing on-node digital signal processing (DSP) and spectral feature extraction directly on the ESP32 microcontroller before radio transmission:

```
+-----------------------------------------------------------------------------------+
|                        ON-NODE EDGE SPECTRAL COMPRESSION                          |
|                                                                                   |
|  [ 256-Point Burst @ 400 Hz ]                                                     |
|  (640 ms time window, 1,536 raw bytes)                                            |
|                │                                                                  |
|                ▼                                                                  |
|  [ ESP32 Hardware FPU 256-Point Real FFT ] ──── Execution Time: 1.5 ms           |
|                │                                                                  |
|                ▼                                                                  |
|  [ Feature Parameter Extraction ]                                                 |
|  ├─ vib_rms_x100   (uint16, 2 bytes) : Root-Mean-Square intensity                  |
|  ├─ vib_peak_x100  (uint16, 2 bytes) : Peak Particle Velocity (PPV)               |
|  └─ vib_fdom_hz    (uint8,  1 byte ) : Dominant Spectral Frequency Peak          |
|                │                                                                  |
|                ▼                                                                  |
|  [ Compressed Wire Telemetry Payload ] ──────── 5 Bytes Total Payload             |
|  Compression Ratio: 28,800:1                                                      |
+-----------------------------------------------------------------------------------+
```

1. **Synchronous Burst Acquisition:**
   Every 60-second superframe, the ESP32 samples a $640\text{ ms}$ burst of 256 vibration points at $400\text{ Hz}$ across the primary vertical axis.
2. **On-Chip Fast Fourier Transform:**
   The ESP32 dual-core Xtensa LX6 hardware Floating-Point Unit (FPU) executes an optimized 256-point real-valued Fast Fourier Transform (FFT) in $1.5\text{ milliseconds}$, transforming the time-domain signal into 128 discrete frequency bins from $0\text{ Hz}$ to $200\text{ Hz}$ with a frequency resolution of:
   $$\Delta f = \frac{f_s}{N} = \frac{400\text{ Hz}}{256} = 1.5625\text{ Hz/bin}$$
3. **Scalar Parametric Compression:**
   The continuous spectrum is condensed into three compact scalar attributes:
   * **`vib_rms_x100` (uint16, 2 bytes):** Root-Mean-Square vibration velocity ($0.01\text{ mm/s}$ resolution), quantifying continuous mechanical disturbance energy.
   * **`vib_peak_x100` (uint16, 2 bytes):** Peak Particle Velocity (PPV in $0.01\text{ mm/s}$ resolution), capturing shock impulses against statutory DGMS blasting limits.
   * **`vib_fdom_hz` (uint8, 1 byte):** The dominant spectral frequency bin ($1\text{ Hz}$ resolution, $0\text{ to }200\text{ Hz}$) containing peak spectral power density.
4. **Data Compression Ratio:**
   This on-node compression transforms $144,000\text{ bytes}$ of raw multi-axis samples per minute into just **5 bytes** on the wire—achieving an exact data reduction ratio of **28,800:1** while fully retaining the physical failure fingerprint.

---

## 2. 4-Band Spectral Discrimination

### Question: Mining environments are subject to severe surface and subsurface acoustic noise (heavy dumpers, conveyor belts, blasting). How does the system discriminate between non-hazardous industrial vibrations and genuine subsurface rock mass failure?

**Answer:** Industrial operations and geological dynamics exhibit distinct spectral signatures. By evaluating the dominant frequency parameter (`vib_fdom_hz`) alongside peak particle velocity, the system categorizes ground excitations into four operational bands:

```
+-----------------------------------------------------------------------------------+
|                        4-BAND VIBRATION DISCRIMINATION                            |
|                                                                                   |
|  [ 8 – 20 Hz ]      [ 40 – 80 Hz ]      [ 50 Hz ±0.5 ]      [ 100 – 250 Hz ]      |
|  Dump Trucks &      Production Blasts   Armored Conveyors   Microseismic Rock     |
|  Haul Traffic       (DGMS Verified)     Harmonic Notch      Tensile Fractures     |
|  ──> VETO           ──> VETO            ──> NOTCH           ──> ESCALATE          |
+-----------------------------------------------------------------------------------+
```

| Frequency Band | Physical Source | Typical Amplitude (PPV) | Operational Classification & System Action |
| :--- | :--- | :--- | :--- |
| **Band 1: 8 – 20 Hz** | Surface Haulage (40-tonne dumpers, excavators) | $0.5\text{ to }3.0\text{ mm/s}$ | **Vehicular Surface Noise (Veto):** Heavy vehicles generate low-frequency surface Rayleigh waves. When detected, the alarm engine suppresses subsidence warnings to prevent false alarms. |
| **Band 2: 50.0 ± 0.5 Hz** | Armored Face Conveyors (AFC) & Shearers | $0.8\text{ to }4.0\text{ mm/s}$ | **Stationary Machine Harmonic (Notch Rejection):** Heavy longwall induction motors emit a steady 50 Hz rotational vibration. An on-node digital notch filter isolates and rejects this harmonic. |
| **Band 3: 40 – 80 Hz** | Deep Production Blasting Detonations | $10.0\text{ to }40.0\text{ mm/s}$ | **Controlled Explosive Detonation (Veto via Blast Log):** Characterized by high PPV shockwaves. The alarm engine cross-references timestamps with the DGMS digital blast schedule to veto evacuation trips. |
| **Band 4: 100 – 250 Hz** | Subsurface Strata Delamination & Micro-Fracturing | $0.2\text{ to }5.0\text{ mm/s}$ | **Rock Mass Tensile/Shear Failure (Immediate Escalation):** High-frequency brittle tensile fracturing of sandstone overlying beds. Directly escalates to Level-2 Geotechnical Warning. |

### Question: How are low-frequency vehicle disturbances and blast events prevented from causing false alarms?

**Answer:** Discrimination uses both spectral characteristics and temporal/spatial cross-validation:
1. **Haul Road Proximity Masking:**
   Nodes located adjacent to primary haulage corridors identify low dominant frequencies ($8\text{ to }20\text{ Hz}$) that correlate with vehicle transit speeds. The backend assigns a spatial weighting factor, down-weighting low-frequency impulses that lack corresponding tilt or tensile strain deformation.
2. **Blast Log Temporal Cross-Referencing:**
   Controlled blasting generates explosive shockwaves with significant energy in the $40\text{ to }80\text{ Hz}$ band. When a node transmits an elevated PPV in this band, the central alarm engine checks the statutory mine blasting ledger. If a scheduled blast occurred within $\pm 30\text{ seconds}$ of the vibration burst, the alarm engine tags the event as operational blasting and suppresses emergency sirens while recording the dynamic response.

---

## 3. Computational Overhead and Safety Authority Architecture

### Question: What is the computational, memory, and energy penalty of executing 256-point FFT feature extraction on the ESP32 microcontroller?

**Answer:** Edge processing is optimized to operate well within the ESP32’s physical processing margins:
* **Execution Latency:** The ESP32’s 32-bit dual-core Xtensa LX6 processor operating at $240\text{ MHz}$ executes a 256-point real radix-2 Cooley-Tukey FFT in **$1.5\text{ milliseconds}$** utilizing the integrated hardware FPU.
* **RAM Footprint:** A 256-point floating-point buffer requires $256 \times 4\text{ bytes} = 1,024\text{ bytes}$ of RAM. Even with input and twiddle-factor scratchpads, total buffer utilization remains under $4\text{ KB}$—less than $1.3\%$ of the available $320\text{ KB}$ internal SRAM.
* **Energy Consumption per Burst:**
  $$\text{Energy}_{\text{FFT}} = V_{cc} \times I_{\text{active}} \times t_{\text{compute}} = 3.3\text{ V} \times 0.050\text{ A} \times 0.0015\text{ s} \approx 0.00025\text{ Joules} = \mathbf{0.25\text{ mJ}}$$
  Relative to the node's total 60-second superframe energy consumption of $\approx 2.4\text{ Joules}$, the edge FFT processing consumes less than **$0.015\%$** of the active energy budget, ensuring battery life is unaffected.

### Question: Does an individual edge sensor node possess autonomous authority to trip mine evacuation sirens or issue emergency alerts?

**Answer:** **No. Edge nodes have zero autonomous safety trip authority.** In high-consequence mining safety systems, allowing individual sensor nodes to trigger evacuation alarms introduces unacceptable risks of false trips due to localized mechanical impacts (e.g., animal disturbance, falling rocks, or direct vehicular strikes).

* **Edge Responsibility:** Nodes act exclusively as high-integrity data collectors. They filter sensor noise, calculate spectral parameters (`rms`, `peak`, `fdom`), package the telemetry, and transmit it upstream over the LoRa mesh.
* **Centralized Safety Decision Engine:** All safety threshold evaluations, multi-node spatial quorum checks, and emergency alarm determinations are strictly centralized within the **C8 Alarm Engine** running on the Gateway Hub and Central Server. To declare an active subsidence emergency, the C8 engine requires:
  1. A spatial quorum of at least three geographically adjacent sensor nodes simultaneously reporting coherent tilt and strain anomalies.
  2. Temporal persistence over multiple consecutive superframe intervals ($>120\text{ seconds}$) to eliminate transient mechanical spikes.
  3. Negative correlation with the statutory mine blasting register.
