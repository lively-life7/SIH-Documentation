# Radio Spectrum Regulatory Compliance (GSR 564(E))

**Module 03 — Mesh Networking**  
**Cross-References:** [`protocol-selection.md`](protocol-selection.md) · [`tdma-scheduling.md`](tdma-scheduling.md) · [`wire-format.md`](wire-format.md) · [Module 08 Test Register](../08-verification/test-register.md) · [Module 00 Key Metrics Summary](../00-executive-gateway/key-metrics-summary.md)

---

## 1. Statutory Framework and Indian Spectrum Delicensing

### Question: What is the exact statutory framework governing wireless transmissions for mining sensor networks in India, and does AEGIS require an individual operating license from the Wireless Planning and Coordination (WPC) Wing?

**Answer:** Wireless transmissions for the AEGIS system operate strictly within the statutory framework established by the Ministry of Communications & Information Technology (Wireless Planning and Coordination Wing — WPC). 

The regulatory foundation is **Gazette of India Notification G.S.R. 564(E)**, dated 30 July 2008, titled *"Use of Low Power Wireless Equipment in the 865–867 MHz Band for Low Power Wireless Access Systems (Exemption from Licensing Requirement) Rules, 2008"*. Under this statutory order:
* The frequency band **865.000 MHz to 867.000 MHz** is officially delicensed throughout the territory of India for both indoor and outdoor low-power wireless access systems.
* Sensor nodes, backbone relays, and gateway hubs operating within the technical limits of GSR 564(E) **do not require an individual station license, wireless operating license, or spectrum allocation fee**.
* Mining operators can deploy the AEGIS system immediately without bureaucratic delays from telecom regulators or spectrum auction processes.

---

## 2. Parameter Compliance Matrix and Legal Limits

### Question: How does AEGIS ensure absolute mathematical and physical compliance with the statutory emission limits defined under GSR 564(E)?

**Answer:** Every transmission parameter in the AEGIS firmware and RF front-end is locked to maintain strict compliance with GSR 564(E) specifications:

| Technical Parameter | Statutory Ceiling (GSR 564(E)) | AEGIS Operational Specification | Regulatory Compliance Margin | Verification Status |
| :--- | :--- | :--- | :--- | :--- |
| **Frequency Range** | $865.000\text{ to }867.000\text{ MHz}$ | **$865.100\text{ to }866.900\text{ MHz}$** | $100\text{ kHz}$ guard band from band edges | **PASS** (Full containment) |
| **Channel Bandwidth** | **$≤ 200\text{ kHz}$ maximum** | **$125\text{ kHz}$ (BW125)** | $75\text{ kHz}$ buffer below statutory limit | **PASS (Test T18)** |
| **Conducted RF Power** | Maximum $30.0\text{ dBm}$ ($1.0\text{ W}$) | **$+14\text{ dBm}$ (Scout) / $+27\text{ dBm}$ (Relay)** | $3.0\text{ dBm}$ to $16.0\text{ dBm}$ below limit | **PASS** |
| **Effective Radiated Power (ERP)** | Maximum $36.0\text{ dBm}$ ($4.0\text{ W}$) | **$+30.0\text{ dBm}$ ERP (Gateway 8.5 dBi omni)** | $6.0\text{ dB}$ safety margin below statutory ceiling | **PASS** |
| **Statutory Duty Cycle Clause** | **None specified in GSR 564(E)** | **Self-imposed $1.0\%$ ceiling** | Scout: $0.15\%$ max; Anchor: $0.56\%$ max | **PASS (Test T12)** |
| **Operating Licensing** | De-licensed band | Autonomous license-free operation | Zero recurring WPC fees | **PASS** |

```
+-----------------------------------------------------------------------------------+
|                        GSR 564(E) FREQUENCY ALLOCATION MAP                        |
|                                                                                   |
|  865.0 MHz                                                             867.0 MHz  |
|  [Guard] [ Ch 1 ]  [ Ch 2 ]  [ Ch 3 ]  [ Ch 4 ]  [ Ch 5 ]  [ Ch 6 ]   [Guard]     |
|   100kHz  865.1     865.5     865.9     866.1     866.5     866.9      100kHz     |
|  |◄────── 125 kHz ─────►|                                                         |
|  |<────────────────── Statutory Max Bandwidth: 200 kHz ───────────────>|          |
+-----------------------------------------------------------------------------------+
```

---

## 3. Bandwidth Enforcement: The 125 kHz vs. 250 kHz Correction

### Question: Why is operating LoRa at 250 kHz or 500 kHz bandwidth illegal in India, and how does AEGIS achieve high data throughput while respecting the 200 kHz statutory ceiling?

**Answer:** A frequent, critical engineering failure in imported or amateur IoT implementations is configuring Semtech transceivers to $250\text{ kHz}$ (`BW250`) or $500\text{ kHz}$ (`BW500`) to increase raw data rates:
1. **The Statutory Violation:**
   GSR 564(E) Section 3(b) explicitly establishes that the maximum channel bandwidth of any wireless equipment in the 865–867 MHz band must not exceed **$200\text{ kHz}$**. Operating LoRa at $250\text{ kHz}$ on Indian territory is an illegal transmission subject to criminal seizure and equipment confiscation under the Indian Wireless Telegraphy Act, 1933.
2. **Firmware-Enforced 125 kHz Bandwidth:**
   AEGIS strictly enforces a **$125\text{ kHz}$ carrier bandwidth (`BW125`)** across all Scout nodes, Anchor relays, and Master Gateway concentrators. This compliance is verified automatically in the CI test pipeline by test **`T18`**, which inspects PHY layer register configurations before compile time.
3. **Throughput and Airtime Recovery:**
   To offset the slight reduction in physical data rate when moving from 250 kHz to 125 kHz, AEGIS optimizes Spreading Factors:
   * Scout nodes transmit at **SF7** ($125\text{ kHz}$), keeping the 23-byte leaf packet airtime down to **$90.4\text{ ms}$**.
   * Anchor relays transmit aggregated trunk frames at **SF8** ($125\text{ kHz}$), maintaining transmission durations well within their allocated TDMA sub-windows.
   * This design achieves compliance while preserving the deterministic 60-second superframe schedule.

---

## 4. Duty Cycle Regulatory Reality vs. Self-Imposed Limits

### Question: Does Indian radio regulation mandate a 1% transmission duty cycle in the 865–867 MHz band, and why does AEGIS enforce a 1.0% duty cycle ceiling if it is not legally mandated?

**Answer:** There is widespread misconception in the IoT industry that Indian law mandates a $1\%$ duty cycle in the 865–867 MHz band:

1. **The True Legal Reality:**
   Unlike European ETSI regulations (EN 300 220), which explicitly mandate $1.0\%$ or $0.1\%$ duty cycles for sub-GHz bands, **Gazette Notification GSR 564(E) contains no duty cycle limitation clause**. Legally, an Indian transmitter in this band could broadcast continuously ($100\%$ duty cycle) provided it does not exceed 1 Watt conducted power and $200\text{ kHz}$ bandwidth.
2. **Why AEGIS Enforces a Strict Self-Imposed 1.0% Ceiling:**
   Despite the absence of a statutory limit, AEGIS firmware enforces an uncompromising **$1.0\%$ duty cycle ceiling** across all transmitting radios:
   * **Spectral Coexistence:** The 865–867 MHz band is shared with industrial UHF RFID logistics tags (EPC Gen2) used in Indian logistics and container yards. Restricting transmission time avoids blocking other industrial users.
   * **Collision Prevention in Multi-Node Arrays:** Enforcing strict duty cycle limits is the mathematical foundation of TDMA scheduling. By keeping leaf transmissions under $100\text{ ms}$ per minute, hundreds of nodes can share the spectrum without saturating RF capacity.
   * **Actual Measured Performance:** In active deployment, AEGIS transmissions operate far below the $1.0\%$ ceiling:
     $$\text{Scout Node Duty Cycle} = \frac{90.4\text{ ms}}{60,000\text{ ms}} \times 100\% = \mathbf{0.1507\%} \ll 1.0\%$$
     $$\text{Anchor Relay Duty Cycle (Nominal)} = \frac{336.0\text{ ms}}{60,000\text{ ms}} \times 100\% = \mathbf{0.5600\%} < 1.0\%$$
   These figures (verified by automated test **`T12`**) demonstrate that the AEGIS network consumes only a fraction of its available airtime, guaranteeing scalability and spectral coexistence.
