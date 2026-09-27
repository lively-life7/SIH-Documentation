# Radio Spectrum Regulatory Compliance (GSR 564(E))

**Module 03 — Mesh Networking**  
**Cross-References:** [`protocol-selection.md`](protocol-selection.md) · [`tdma-scheduling.md`](tdma-scheduling.md) · [Module 08 Test T18](../08-verification/test-register.md)

---

## 1. Statutory Framework: Indian Gazette Notification GSR 564(E)

Wireless deployments in Indian mines must comply strictly with the Ministry of Communications & Information Technology (Wireless Planning and Coordination Wing — WPC) statutory standards. 

The regulatory foundation for AEGIS is **Gazette of India Notification GSR 564(E)**, dated 30 July 2008, which delicenses the use of low-power wireless equipment in the **865 – 867 MHz frequency band** for indoor and outdoor operations without an individual operating license.

---

## 2. Regulatory Parameter Compliance Matrix

| Technical Parameter | Statutory Ceiling (GSR 564(E)) | AEGIS Operational Value | Verification Status |
| :--- | :--- | :--- | :--- |
| **Frequency Band** | 865.000 – 867.000 MHz | **865.100 – 866.900 MHz** | **PASS** (Operates entirely within band) |
| **Carrier Bandwidth** | **Maximum 200 kHz** | **125 kHz (BW125)** | **PASS (Test T18)** |
| **Conducted RF Power** | Maximum 30 dBm (1 Watt) | **14 dBm (Scout), 27 dBm (Relay)** | **PASS** (Below 30 dBm ceiling) |
| **Effective Radiated Power (ERP)**| Maximum 36 dBm (4 Watts) | **30 dBm ERP (Gateway 8.5 dBi omni)** | **PASS** (6 dB safety margin) |
| **Statutory Duty Cycle Clause** | **None specified in GSR 564(E)** | **Self-imposed 1.0% limit** | **PASS** (Convention adopted) |
| **Operating Licensing** | License-free (De-licensed band) | Fully license-free | No individual WPC station license required |

---

## 3. The 125 kHz vs. 250 kHz Correction (Test T18)

Early academic prototypes in the IoT space frequently configure Semtech LoRa transceivers at 250 kHz or 500 kHz bandwidth to minimize airtime:
* **The Regulatory Violation:** GSR 564(E) Section 3(b) explicitly establishes a statutory ceiling of **200 kHz channel bandwidth**. Operating LoRa at 250 kHz (`BW250`) on Indian soil is a direct statutory violation subject to confiscation under the Indian Telegraph Act.
* **The Engineering Fix:** AEGIS fixes the carrier bandwidth across all nodes, relays, and gateways at **125 kHz (`BW125`)**, strictly verified in CI by test `T18`.
* **Airtime Recovery:** To recover the airtime lost by dropping from 250 kHz to 125 kHz, relay trunk transmissions were elevated from SF9 to **SF8** and leaf nodes to **SF7**, fully maintaining the 60-second superframe schedule.

---

## 4. The 1.0% Duty Cycle Reality & Stage Defense

A common misstatement among technical presenters is claiming that "Indian radio law mandates a 1% duty cycle." 

> [!NOTE]
> **Authoritative Technical Stance:**
> Gazette Notification GSR 564(E) regulates center frequency, radiated power, and channel bandwidth, but contains **no explicit duty cycle limitation clause**. 
> 
> However, AEGIS enforces a strict **self-imposed 1.0% duty cycle ceiling** across all transmitters:
> 1. The 865–867 MHz band is shared with industrial UHF RFID systems and logistics trackers. Enforcing a 1% ceiling prevents mutual RF blocking.
> 2. Self-imposed transmission caps are the mathematical foundation that allows a TDMA mesh network to scale to dozens of nodes without channel collapse.
> 3. Measured field duty cycles remain far below the convention: **Scout nodes run at $0.10\%$**, and **Anchor relays run at $0.56\%$** (Test `T12`).

Stating this distinction demonstrates genuine mastery of Indian telecommunications law before technical and regulatory judging panels.
