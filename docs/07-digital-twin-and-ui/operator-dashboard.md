# Operator Mission Control Dashboard & Telemetry Inspector

**Module 07 — Digital Twin & UI**  
**Cross-References:** [`3d-visualization.md`](3d-visualization.md) · [`alert-system.md`](alert-system.md) · [Module 06 Data Architecture](../06-backend-pipeline/data-architecture.md)

---

## 1. Multi-Pane Mission Control Interface

The operator dashboard provides a synchronized, multi-panel supervisory control interface designed for continuous 24/7 mine control room operations:

```
+-----------------------------------------------------------------------------------+
|  AEGIS SCADA v2.0 | PANEL: ADR-LW-04 | GATEWAY: ONLINE | d_committed: 358m | MODE: LIVE |
+------------------------------------+----------------------------------------------+
| [3D TERRAIN TWIN]                  | [LIVE TELEMETRY INSPECTOR]                   |
| - 3D Three.js / CesiumJS viewport  | Node ID: 14 (Tier 1B Strain)                 |
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

---

## 2. Interactive Node Telemetry Inspector

Operators can click on any physical node peg on the 3D surface to inspect its real-time diagnostic parameters:
* **Sensor Gauges:** Analog-style radial dials displaying current tilt angle, tensile strain, and extensometer extension against configured statutory warning thresholds.
* **Vibration Spectrum:** Real-time power spectral density displaying dominant frequency bins (`f_dom`) with automated source tagging (`TRUCK`, `CONVEYOR`, `BLAST`, `MICROSEISMIC`).
* **RF Link Quality:** Upstream link metrics including RSSI, SNR, current TDMA slot index, and primary/backup parent route status.
* **Power Subsystem:** Live battery voltage curve ($V_{\text{bat}}$), charging current, and temperature-adjusted battery health percentage.

---

## 3. The 690-Day Deterministic Time Scrubber

A critical capability for post-incident accident investigation and statutory DGMS inquiries is **Bit-Identical Historical Replay**:
* Operators can scrub through historical time using the playback slider.
* Because ground displacement accumulation is persisted in integer millimeter coordinates (`int32`), the 3D terrain reconstruction at any historical timestamp $T$ renders identically to how it appeared during live operation.
* Regulators can review the exact ground deformation progression hour-by-hour preceding any surface fissure rupture.

---

## 4. Role-Based Operating Modes

The dashboard provides three distinct interface profiles tailored to user responsibilities:

1. **Shift Control Room Operator Mode:**
   * Focuses on active alarms, siren trip states, radio link heartbeats, and field evacuation triggers.
   * Minimalist visual interface with high-contrast alert banners designed to avoid operator alarm fatigue.
2. **Mine Planning & Geotechnical Engineer Mode:**
   * Unlocks full Knothe curve fitting parameters, strain derivative contour maps, borehole extensometer charts, and extraction face advance rate correlation tools.
3. **DGMS Statutory Inspector Mode:**
   * Provides read-only access to tamper-proof audit ledgers, digital blast register cross-reference logs, alarm trip histories, and raw uncorrected telemetry archives.
