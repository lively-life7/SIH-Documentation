# Automated Alert Dispatch & Siren Activation System

**Module 07 — Digital Twin & UI**  
**Cross-References:** [`operator-dashboard.md`](operator-dashboard.md) · [Module 06 C8 Alarm Engine](../06-backend-pipeline/c8-alarm-engine.md)

---

### Question: How does the AEGIS alert system achieve a deterministic sub-1.4 second end-to-end siren activation latency?
**Answer:** In geotechnical slope stability and underground longwall caving, sudden strata collapse can propagate within seconds. AEGIS enforces a strict statutory latency ceiling of **$< 1.4\text{ seconds}$** from physical sensor threshold trip to full acoustic siren emission. 

The deterministic latency breakdown across the six critical path stages is mathematically and experimentally validated:

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

$$\text{Total Critical Path Latency} = 5\text{ ms} + 69.9\text{ ms} + 20\text{ ms} + 50\text{ ms} + 200\text{ ms} + 1,000\text{ ms} = \mathbf{1,344.9\text{ ms}} < \mathbf{1.4\text{ s}}$$

1. **Hardware Edge Detection ($< 5\text{ ms}$):** Transducer threshold breaches or crack-wire severances trigger instantaneous hardware GPIO interrupts on the edge microcontroller.
2. **Emergency Subslot Uplink ($69.9\text{ ms}$):** The station preempts regular TDMA schedules, transmitting a high-priority 14-byte `P3_ALERT` frame on Spreading Factor 7 (SF7) at $125\text{ kHz}$ bandwidth.
3. **Gateway Radio Capture ($< 20\text{ ms}$):** The multi-channel SX1302 LoRa concentrator captures the preamble, demodulates the frame, and verifies the CRC-16 checksum.
4. **Edge Gateway Firmware Validation ($< 50\text{ ms}$):** Embedded C8 logic verifies 5-station spatial Byzantine consensus and confirms absence of scheduled blast vetoes.
5. **Direct Hardware Relay Contact Closure ($< 200\text{ ms}$):** A solid-state optocoupled relay closes a physical 24V DC circuit, energizing the primary siren contactor.
6. **Acoustic Motor Run-Up ($< 1,000\text{ ms}$):** The dual-tone electromechanical siren motor accelerates to full RPM, reaching a statutory $125\text{ dB}$ sound pressure level at $30\text{ meters}$.

---

### Question: Why is local siren activation completely independent of cellular networks and cloud infrastructure?
**Answer:** In mining regions across India (e.g., Godavari Valley, Singrauli, Korba), adverse monsoonal weather, lightning strikes, and remote topography frequently disable public cellular towers and terrestrial fiber connections. 

A life-safety system whose alarm sequence depends on external internet connectivity, third-party cloud brokers, or webhook APIs creates an unacceptable single point of failure:
* The Master Edge Gateway executes an embedded instance of the C8 deterministic safety engine locally on bare-metal firmware or an on-premise industrial single-board computer.
* The 125 dB evacuation siren is hardwired directly to the gateway's onboard solid-state relay on its 10-meter mast.
* When emergency criteria are confirmed, the relay latches autonomously over the local physical circuit. The siren triggers with full sub-1.4 second fidelity even if cellular 4G/NB-IoT backhaul is completely severed.

---

### Question: What is the Multi-Channel Alert Dispatch Matrix across operational tiers under DGMS Coal Mines Regulations?
**Answer:** In parallel with physical acoustic sirens, AEGIS dispatches redundant digital notifications across multiple communication channels tailored to user roles and statutory response procedures:

| Alert Tier | Visual Level | Dispatched Channels | Recipient Group | Statutory Acknowledgment Window |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1: Advisory** | Blue | SCADA Banner, Audit Log | Shift Geotechnical Trainee | Logged automatically; routine review |
| **Tier 2: Warning** | Yellow | Dashboard Flash, SMS Broadcast | Shift Mining Overman | 15 Minutes |
| **Tier 3: Critical** | Orange | SMS, Automated Phone Call, Strobe | Geotechnical In-Charge & Manager | 3 Minutes |
| **Tier 4: Emergency** | **Red** | **125 dB Siren, Automated Voice Call, SMS** | **Entire Mine Shift Crew & GM** | **IMMEDIATE EVACUATION** |

---

### Question: What automated escalation protocol governs unacknowledged alerts to prevent control room operator fatigue or omission?
**Answer:** To eliminate the risk of critical warnings being ignored during shift handovers or operator distraction, AEGIS implements an automated, auditable escalation ladder:

```
[ Tier 3 Critical Alert Tripped ]
               │
               ▼
[ 3-Minute Digital Countdown Initiated on SCADA Dashboard ]
               │
               ├── Operator Acknowledges with DGMS PIN ──> [ Escalation Halted; Action Logged ]
               │
               ▼  If Unacknowledged after 3 Minutes:
[ Automated Outbound Voice Phone Calls Placed to Colliery Agent & Safety Officer ]
               │
               ▼  If Unacknowledged after 5 Minutes:
[ Autonomous Escalation to Tier 4 Emergency: Physical 125 dB Siren Latched ]
```

1. **Digital Countdown:** When a Tier 3 Critical alert trips, the SCADA interface initiates an aggressive audio-visual 3-minute countdown timer.
2. **PIN-Authenticated Acknowledgment:** The control room operator must enter their unique DGMS statutory authorization PIN to acknowledge the condition. Acknowledgment records the operator ID, timestamp, and active sensor states into a cryptographically hashed audit ledger.
3. **Voice Call Escalation (3 Minutes):** If no valid acknowledgment is registered within 180 seconds, the backend telephony engine initiates automated voice phone calls to the Colliery Agent and Safety Officer, reading the sensor coordinates and strain magnitudes via text-to-speech.
4. **Autonomous Emergency Escalation (5 Minutes):** If 300 seconds elapse without authorized acknowledgment, the system autonomously escalates the alert to Tier 4 Emergency, closing the siren relay to evacuate all personnel from the affected sector.
