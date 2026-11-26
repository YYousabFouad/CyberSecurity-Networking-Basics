# OSI Model: Layers 4, 5, 6, and 7 Study Notes

---

## Layer 4 — Transport — Service-to-Service Delivery

- **Core Function:** Distinguish data streams *(make sure that the right program receives the right data)*.
- **Addressing Scheme — Ports:**
  - **TCP (`0–65535`)** & **UDP (`0–65535`)** are 2 different strategies to distinguish data streams.
  - **TCP:** Favors **reliability**.
  - **UDP:** Favors **efficiency**.
- **Server Behavior:** Servers listen for requests on **pre-defined ports** (e.g., HTTP on port `80`, HTTPS on port `443`, IRC on `6667`).
- **Client Behavior (Example: IRC — Internet Relay Chat):**
  - Clients select a **random (ephemeral) port** for each connection.
  - **Layer 3 (Network):** Source and Destination **IP addresses**.
  - **Layer 4 (Transport):** Source and Destination **ports**.

### How Data Streams Are Isolated (Different Programs/Tabs vs. Same Program/Tabs)

When a single computer runs multiple applications—or multiple browser tabs connected to the same web server—the operating system isolates each data stream using a **4-Tuple (Socket Pair)**:

$$\text{Connection Identifier} = (\text{Source IP},\ \text{Source Port},\ \text{Destination IP},\ \text{Destination Port})$$

1. **Different Programs (e.g., Chrome Browser vs. IRC Client):**
   - Each application opens its own socket and is assigned a **unique random Source Port** by the OS (e.g., Chrome gets `51400`, IRC gets `51401`).
   - Even if both talk to external servers, incoming packets arrive with different **Destination Ports** (`51400` vs. `51401`), so the OS hands each stream strictly to the matching program.

2. **Same Program, Different Tabs (e.g., Tab 1 and Tab 2 both open to `example.com:443`):**
   - Both tabs share the same **Source IP**, **Destination IP**, and **Destination Port (`443`)**.
   - To keep Tab 1's data isolated from Tab 2's data, the browser/OS opens **two separate connections** and assigns a **different random Source Port** to each tab (e.g., Tab 1 uses `52110`, Tab 2 uses `52111`).
   - When the server replies, traffic sent back to `52110` goes exclusively to Tab 1, and traffic sent back to `52111` goes exclusively to Tab 2.

#### Stream Isolation Diagram

```text
+-----------------------------------------------------------------------+
|                       CLIENT HOST (192.168.1.10)                      |
|                                                                       |
|  +-----------------------+  +-----------------------+  +-----------+  |
|  |   Browser — Tab 1     |  |   Browser — Tab 2     |  | IRC App   |  |
|  |   (example.com)       |  |   (example.com)       |  | (Chat)    |  |
|  +-----------+-----------+  +-----------+-----------+  +-----+-----+  |
|              |                          |                    |        |
|        Src Port: 52110            Src Port: 52111      Src Port: 51401|
|              |                          |                    |        |
|  +-----------v--------------------------v--------------------v-----+  |
|  |               OS Transport Layer (TCP/UDP Demultiplexing)       |  |
|  |  • Stream A: 192.168.1.10:52110 <---> 93.184.216.34:443         |  |
|  |  • Stream B: 192.168.1.10:52111 <---> 93.184.216.34:443         |  |
|  |  • Stream C: 192.168.1.10:51401 <---> 198.51.100.25:6667        |  |
|  +----------------------------------+------------------------------+  |
+-------------------------------------|---------------------------------+
                                      |
                        Single Physical Network Link
                                      |
                    +-----------------+-----------------+
                    |                                   |
                    v                                   v
      +---------------------------+       +---------------------------+
      |   Web Server (Example)    |       |        IRC Server         |
      |   IP: 93.184.216.34       |       |   IP: 198.51.100.25       |
      |   Listening Port: 443     |       |   Listening Port: 6667    |
      +---------------------------+       +---------------------------+
```

---

## Layers 5, 6, 7 — Session, Presentation, Application

- **Distinction Between Layers 5, 6, and 7:** The distinction between these layers is **somewhat vague**.
- **Model Consolidation:** Other networking models (such as the **TCP/IP Model**) combine these three layers into a **single Application layer**.

### Diagram: OSI Model vs. TCP/IP Model

```text
+-----------------------------------+-----------------------------------+
|             OSI MODEL             |           TCP/IP MODEL            |
|            (7 Layers)             |      (4 / 5 Layer Equivalent)     |
+-----------------------------------+-----------------------------------+
|  7. Application                   |                                   |
|  6. Presentation                  |  Application                      |
|  5. Session                       |  (Combines L5, L6, L7)            |
+-----------------------------------+-----------------------------------+
|  4. Transport                     |  Transport                        |
+-----------------------------------+-----------------------------------+
|  3. Network                       |  Internet                         |
+-----------------------------------+-----------------------------------+
|  2. Data Link                     |  Network Access / Link            |
|  1. Physical                      |  (Combines L1 & L2)               |
+-----------------------------------+-----------------------------------+
```

---

## How Two Hosts Communicate Using the 7 Layers of the OSI Model

When a **Client Host** sends data to a **Server Host**, the data moves **top-to-bottom** on the Client (**Encapsulation**), travels across the physical medium, and moves **bottom-to-top** on the Server (**Decapsulation**).

### Protocol Data Units (PDUs) & Encapsulation Formulas

- **Segment** $= \text{Data} + \text{TCP/UDP Header (L4)}$
- **Packet** $= \text{Segment} + \text{IP Header (L3)}$
- **Frame** $= \text{Packet} + \text{MAC Addresses (L2)}$

### End-to-End Encapsulation & Decapsulation Diagram

```text
      CLIENT HOST (Sender)                             SERVER HOST (Receiver)
   [Direction: Top ---> Down]                        [Direction: Bottom ---> Up]
         (Encapsulation)                                   (Decapsulation)

+-------------------------------+                 +-------------------------------+
| L7: Application               |                 | L7: Application               |
| Generates user data (HTTP/IRC)|                 | Delivers data to server app   |
+---------------+---------------+                 +---------------^---------------+
                |                                                 |
+---------------v---------------+                 +---------------+---------------+
| L6: Presentation              |                 | L6: Presentation              |
| Formats, encodes, encrypts    |                 | Decrypts, decodes, formats    |
+---------------+---------------+                 +---------------^---------------+
                |                                                 |
+---------------v---------------+                 +---------------+---------------+
| L5: Session                   |                 | L5: Session                   |
| Manages dialog / session ID   |                 | Validates / manages session   |
+---------------+---------------+                 +---------------^---------------+
                | [ Data ]                                        | [ Data ]
+---------------v---------------+                 +---------------+---------------+
| L4: Transport                 |                 | L4: Transport                 |
| Adds TCP/UDP Ports            |                 | Reads Ports, strips L4 header |
| PDU: [ L4 | Data ] = Segment  |                 | Extracts Data from Segment    |
+---------------+---------------+                 +---------------^---------------+
                | [ Segment ]                                     | [ Segment ]
+---------------v---------------+                 +---------------+---------------+
| L3: Network                   |                 | L3: Network                   |
| Adds Src & Dst IP Addresses   |                 | Reads IPs, strips L3 header   |
| PDU: [ L3 | Segment ] = Packet|                 | Extracts Segment from Packet  |
+---------------+---------------+                 +---------------^---------------+
                | [ Packet ]                                      | [ Packet ]
+---------------v---------------+                 +---------------+---------------+
| L2: Data Link                 |                 | L2: Data Link                 |
| Adds Src & Dst MAC + Trailer  |                 | Reads MACs, strips L2 header  |
| PDU: [ L2 | Packet | FCS ]    |                 | Extracts Packet from Frame    |
|      = Frame                  |                 |                               |
+---------------+---------------+                 +---------------^---------------+
                | [ Frame ]                                       | [ Frame ]
+---------------v---------------+                 +---------------+---------------+
| L1: Physical                  |                 | L1: Physical                  |
| Converts Frame to bits/signals|                 | Receives bits, rebuilds Frame |
+---------------+---------------+                 +---------------^---------------+
                |                                                 |
                +=========> Physical Medium (Cables/Wi-Fi) =======+
                             (10110010101110010101...)
```

### Step-by-Step Walkthrough

1. **Client (Top $\rightarrow$ Down / Encapsulation):**
   - **L7–L5:** The client program creates the message, formats/encrypts it, and establishes a session.
   - **L4 (Transport):** Wraps the data with Source & Destination Ports $\rightarrow$ creates a **Segment**.
   - **L3 (Network):** Wraps the Segment with Source & Destination IP addresses $\rightarrow$ creates a **Packet**.
   - **L2 (Data Link):** Wraps the Packet with Source & Destination MAC addresses $\rightarrow$ creates a **Frame**.
   - **L1 (Physical):** Converts the Frame into raw electrical, optical, or radio bits (`0`s and `1`s) onto the wire/air.
2. **Server (Bottom $\rightarrow$ Up / Decapsulation):**
   - **L1 (Physical):** Captures the raw bits and reconstructs the Frame.
   - **L2 (Data Link):** Verifies the destination MAC address matches its NIC, strips L2 headers, and passes up the **Packet**.
   - **L3 (Network):** Verifies the destination IP address matches, strips L3 headers, and passes up the **Segment**.
   - **L4 (Transport):** Checks the destination Port to identify the right listening service, strips L4 headers, and passes up the **Data**.
   - **L5–L7:** Manages the session, decrypts/decodes the payload, and delivers the final request to the server application.

---

## Important Conceptual Takeaways

- **Neither of these rules are strict** *(exceptions exist in real-world protocols)*.
- **The OSI Model is simply a model** — not rigid rules that everything strictly adheres to, but a **conceptualization of what is required for data to flow** across a network.