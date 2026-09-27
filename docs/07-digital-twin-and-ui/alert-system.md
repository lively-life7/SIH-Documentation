# Automated Alert Dispatch & Siren Activation System

**Module 07 — Digital Twin & UI**  
**Cross-References:** [`operator-dashboard.md`](operator-dashboard.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md)

---

## 1. Sub-1.4 Second Emergency Siren Activation

When ground failure occurs, every second of evacuation delay increases fatality risk. AEGIS enforces a hard latency ceiling of **$< 1.4\text{ seconds}$** from physical sensor trip to acoustic siren emission.

```
[Physical Ground Rupture / Crack Breached]
               │
               ▼  1. Hardware Edge Detection (< 5 ms)
[Microcontroller GPIO Interrupt / ADC Threshold Trip]
               │
               ▼  2. Emergency Subslot Uplink (69.9 ms Airtime)
[LoRa P3 Alert Packet Transmitted on SF7 / 125 kHz]
               │
               ▼  3. Gateway Radio Capture (< 20 ms)
[Master Gateway SX1302 Concentrator Decodes CRC-16]
               │
               ▼  4. Edge Gateway Firmware Validation (< 50 ms)
[Local C8 Quorum & DGMS Blast Veto Evaluated on Edge Hub]
               │
               ▼  5. Direct Hardware Relay Contact Closure (< 200 ms)
[Solid-State Relay Latches 24V DC Circuit on 10m Mast]
               │
               ▼  6. Motor Run-Up to Full Decibels (< 1,000 ms)
[125 dB Omnidirectional Mine Evacuation Siren Sounds]
```

$$\text{Total Measured Critical Path Latency} = 5 + 69.9 + 20 + 50 + 200 + 1000 = \mathbf{1,344.9\text{ ms}} < \mathbf{1.4\text{ seconds}}$$

### Local Siren Independence from Cloud
A non-negotiable safety invariant is that **the physical evacuation siren does NOT depend on cellular internet connectivity or cloud servers**:
* The Master Gateway runs a local instance of the deterministic C8 validation logic.
* If a valid emergency trip packet arrives from the field mesh, the gateway's onboard solid-state relay triggers the physical siren directly, even if the 4G/NB-IoT cellular link is completely severed.

---

## 2. Multi-Channel Alert Dispatch Matrix

In parallel with acoustic field sirens, AEGIS dispatches redundant digital notifications across multiple communication channels:

| Alert Tier | Visual Level | Dispatched Channels | Recipient Group | Acknowledgment Window |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1: Advisory** | Blue | SCADA Banner, Audit Log | Shift Geotechnical Trainee | Logged automatically |
| **Tier 2: Warning** | Yellow | Dashboard Flash, SMS Broadcast | Shift Mining Overman | 15 Minutes |
| **Tier 3: Critical** | Orange | SMS, Automated Phone Call, Strobe | Geotechnical In-Charge & Manager | 3 Minutes |
| **Tier 4: Emergency**| **Red** | **125 dB Siren, Automated Voice Call, SMS** | **Entire Mine Shift Crew & GM** | **IMMEDIATE EVACUATION** |

---

## 3. Automated Escalation & Operator Acknowledgment Protocol

To prevent alerts from languishing on an unattended control room screen during shift changes:
1. When a **Tier 3 (Critical)** alert trips, an automated 3-minute countdown begins in the SCADA dashboard.
2. The control room operator must enter their unique DGMS digital PIN to acknowledge the warning.
3. **Automated Escalation Rule:**
   * If the alert is **unacknowledged after 3 minutes**, the backend automatically triggers voice phone calls to the Colliery Agent and Safety Officer.
   * If the alert remains **unacknowledged after 5 minutes**, the system autonomously escalates the event to **Tier 4 Emergency**, latching the physical evacuation siren.
