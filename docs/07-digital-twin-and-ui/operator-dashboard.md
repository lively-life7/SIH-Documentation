# Operator Mission Control Dashboard & Telemetry Inspector

**Module 07 — Digital Twin & UI**  
**Cross-References:** [`3d-visualization.md`](3d-visualization.md) · [`alert-system.md`](alert-system.md) · [Module 06 Data Architecture](../06-backend-pipeline/data-architecture.md)

---

### Question: What is the architectural layout and functional segmentation of the AEGIS Operator Mission Control interface?
**Answer:** The AEGIS Operator Mission Control interface is engineered as a synchronized multi-pane supervisory control and data acquisition (SCADA) console designed for 24/7 continuous operation in mine control rooms. It partitions mission-critical information into dedicated visual regions to maximize situational awareness without overwhelming the human operator:

```
+-----------------------------------------------------------------------------------+
|  AEGIS SCADA v2.0 | PANEL: ADR-LW-04 | GATEWAY: ONLINE | d_committed: 358m | MODE: LIVE |
+------------------------------------+----------------------------------------------+
| [3D TERRAIN TWIN]                  | [LIVE TELEMETRY INSPECTOR]                   |
| - 3D Three.js / CesiumJS viewport  | Node ID: 14 (Tier 1B Strain Station)         |
| - Extraction face tracking: 4m/day | Battery: 3.32V [||||||||  ] 84% Solar: ON    |
| - Exclusion rings: 1.2R / 1.5R     | Tilt X: -12.4 mrad | Tilt Y: +4.2 mrad      |
| - Active crack alert indicators    | Strain: 1,240 µε  | Ext: +12.4 mm            |
|                                    | Vibration RMS: 0.12 mm/s | f_dom: 18 Hz (TRUCK)|
|                                    | RSSI: -84 dBm | SNR: +7.2 dB | Hops: 2      |
+------------------------------------+----------------------------------------------+
| [TIMELINE SCRUBBER & HISTORICAL REPLAY]                                           |
| [|<] [<<] [PLAY] [>>] [>|]  Day: 24.5 / 40.0  Time: 2026-09-27 14:30:00 UTC       |
| ──────────────────────────────●───────────────────────────────────────────────   |
+-----------------------------------------------------------------------------------+
```

1. **Top Status Banner:** Displays global mine telemetry: active extraction panel ID, edge gateway link heartbeat, committed advance distance ($d_{\text{committed}}$), active siren relay status, and operating mode (`LIVE` vs `HISTORICAL_REPLAY`).
2. **Left Viewport (3D Terrain Twin):** High-framerate WebGL viewport showing continuous subsidence topography, infrastructure vectors, advancing extraction face, and dynamic exclusion perimeters ($1.2r$ and $1.5r$).
3. **Right Inspector Pane (Live Telemetry):** Deep diagnostic inspector that renders electrical, RF, and mechanical sensor streams for any selected station.
4. **Bottom Dock (Timeline Scrubber):** Interactive historical scrubber allowing scrubbing through multi-month operational archives.

---

### Question: What real-time diagnostic parameters and spectral signatures can be inspected for an individual sensor node?
**Answer:** Clicking any station marker in the 3D twin opens the Live Telemetry Inspector, displaying four synchronized diagnostic telemetry groups:

1. **Mechanical & Geotechnical Gauges:**
   * **Biaxial Tilt:** Radial compass dials displaying inclination in the $X$ and $Y$ planes ($\text{mrad}$ or $\text{mm/m}$).
   * **Tensile Strain:** Dynamic gauge showing microstrain ($\mu\varepsilon$) relative to statutory CMR 2017 thresholds ($500$, $1000$, and $1500\ \mu\varepsilon$).
   * **Borehole Extensometer:** Linear rod displacement ($\text{mm}$) showing strata layer separation.
2. **Vibration Power Spectral Density (PSD):**
   * Real-time FFT spectrum displaying dominant frequency bins ($f_{\text{dom}}$) and RMS vibration amplitude.
   * **Automated Source Tagging:** Telemetry classifies vibration sources into operational categories:
     * $10\text{--}25\text{ Hz}$: Heavy haul truck traffic / dumpers
     * $50\text{--}60\text{ Hz}$: Longwall shearer motors / armored face conveyors
     * $> 100\text{ Hz}$ impulse: Explosive rock blasting
     * $500\text{--}2000\text{ Hz}$ high-frequency transient: Micro-seismic rock fracturing / bridge snapping
3. **RF Wireless Mesh Health:**
   * Received Signal Strength Indicator (RSSI, $\text{dBm}$) and Signal-to-Noise Ratio (SNR, $\text{dB}$).
   * Active TDMA slot index, hop count to gateway, and primary/secondary parent node routing IDs.
4. **Power & Thermal Subsystem:**
   * Live battery voltage ($V_{\text{bat}}$) curve with low-voltage cutoff threshold line ($3.2\text{ V}$).
   * Photovoltaic solar harvesting status (`ACTIVE` / `INACTIVE`) and silicon die temperature ($T_{\text{die}}$).

---

### Question: How does the 690-Day Deterministic Historical Replay feature work, and why is bit-identical reconstruction critical for DGMS inquiries?
**Answer:** Following any surface cracking incident or rockfall event, statutory inspectors from the Directorate General of Mines Safety (DGMS) require an auditable, step-by-step reconstruction of ground deformation leading up to the failure.

The AEGIS Timeline Scrubber delivers **Bit-Identical Historical Replay**:
1. **Integer State Persistence:** Cumulative ground displacements and sensor states are persisted in integer coordinates (`int32` millimeters and $\mu\varepsilon$) in columnar Parquet files, eliminating floating-point rounding errors across different platforms.
2. **Deterministic Time-Travel Query:** Moving the timeline slider to any historical timestamp $T$ issues an asynchronous partition slice request:
   ```sql
   SELECT timestamp_utc, node_id, strain_ue, tilt_x_mrad, tilt_y_mrad, alert_level
   FROM "archive/readings.parquet"
   WHERE timestamp_utc BETWEEN $T - 300s AND $T
   ```
3. **Identical Visual State:** The 3D digital twin re-populates the WebGL displacement textures and sensor dials to match the exact physical state presented to control room operators at that historical second.
4. **Regulatory Defense:** Mine management can demonstrate conclusively to DGMS inspectors whether precursors (accelerating tilt rates, microseismic chatter, or crack wire alerts) were detected, whether alarms escalated properly, and whether operator acknowledgment protocols were adhered to under CMR 2017 regulations.

---

### Question: What role-based operating modes are provided, and how do they mitigate operator alarm fatigue?
**Answer:** Alarm fatigue is a documented safety hazard in industrial mining control rooms; excessive trivial alerts cause operators to mute or ignore life-critical alarms. AEGIS mitigates this by providing three tailored Role-Based Access Control (RBAC) operational profiles:

1. **Shift Control Room Operator Mode:**
   * **Focus:** Immediate life safety, active siren trip states, gateway heartbeats, and field evacuation triggers.
   * **UI Ergonomics:** High-contrast dark theme with large, glanceable alert cards. Sub-critical engineering telemetry is tucked away, eliminating clutter. Only actionable Tier 2, 3, and 4 alerts demand attention.
2. **Mine Planning & Geotechnical Engineer Mode:**
   * **Focus:** Deep structural analysis and predictive planning.
   * **Tooling:** Unlocks Knothe empirical curve overlays, strain tensor derivative vector fields, continuous subsidence volume calculations, and extraction face advance rate correlations.
3. **DGMS Statutory Inspector Mode:**
   * **Focus:** Compliance verification and forensic auditing.
   * **Auditing:** Read-only access to tamper-proof audit trails, cryptographic alert acknowledgment signatures, raw uncalibrated ADC telemetry records, and DGMS Circular 7/1997 blast cross-reference logs.
