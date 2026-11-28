# Packet Traveling — How Data Moves Through the Internet

- **Course:** Practical Networking — *Networking Fundamentals* (Module 1 Summary — Lesson 7 / Video #13)
- **Video:** [How Data moves through the Internet - Networking Fundamentals](https://www.youtube.com/watch?v=YJGGYKAV4pA&list=PLIFyRwBY_4bRLmKfP1KnZA6rZbRHtxmXi&index=13) (`26:46`)
- **Role in This Repository:** Master summary & fast-revision guide tying together every topic covered in folders `01` through `06` and the hands-on lab [`two-subnets-router-lab.pkt`](./two-subnets-router-lab.pkt).

> **Quick Jump to Topic Folders:**
> [`01-How the Data Flows`](./01-How%20the%20Data%20Flows/) | [`02-OSI Model`](./02-OSI%20Model/) | [`03-Everything Hosts Do`](./03-Everything%20Hosts%20Do/) | [`04-How Switches Work`](./04-How%20Switches%20Work/) | [`05-Everything Routers Do`](./05-Everything%20Routers%20Do/) | [`06-Network Protocols`](./06-Network%20Protocols/)

---

## 1. Overview: Tying Everything Together (`00:00–01:03`)

This summary lesson demonstrates how data actually moves from one host to another across switches, routers, and the Internet—bringing together every piece from Module 1:

1. **OSI Model (Layers 1–7):** Layer 2 handles **hop-to-hop** delivery on a local link; Layer 3 handles **end-to-end** delivery across networks.
2. **Hosts (Same Network & Foreign Networks):** How hosts decide whether a destination is local or must go through a Default Gateway.
3. **Switches (Layer 2):** How switches learn MAC addresses and forward or flood frames within a local network.
4. **Routers (Layer 3):** How routers inspect Routing Tables, swap Layer 2 headers, resolve next hops with ARP, and hand packets off across the Internet.
5. **Network Protocols:** How **ARP**, **DNS**, and **HTTP** work together to load a website when you type a URL into a browser.

> **Universal Rule:** Hosts, switches, and routers all follow simple, consistent packet-processing rules. If you understand these rules on a small topology, the exact same logic scales to topologies with hundreds of devices.

---

## 2. The Three Core Tables of Networking (`01:03–02:55`)

Understanding how packets travel through any network comes down to **three tables**, each mapping one piece of information to another:

| Table | Used By | What It Maps | How It Is Populated |
| :--- | :--- | :--- | :--- |
| **MAC Address Table** | **Switches** | **Switch Port** ↔ **MAC Address** | **Dynamically** as frames arrive on switch ports (learns/refreshes from **Source MAC**; forwards/floods using **Destination MAC**) |
| **ARP Table** *(ARP Cache)* | **Hosts & Routers** | **IP Address** ↔ **MAC Address** | **Dynamically** via **ARP Request** (broadcast `ffff`) and **ARP Reply** (unicast); starts empty |
| **Routing Table** | **Hosts & Routers** | **IP Network** → **Interface** (`D.C.`) or **Next-Hop IP** | **Populated ahead of time** (Directly Connected, Static, or Dynamic routes); unmatched destinations are **dropped** |

### Key Device Notes
- **Switches when passing traffic through:** Because traffic is moving *through* the switch—not *to* or *from* it—the switch's own MAC address and IP address are irrelevant. Only the switch's **port numbers** (here **Port 4** and **Port 5**) matter. The switch only holds a **MAC Address Table**.
- **Hosts behave like small routers at Layer 3:** Both **Hosts** and **Routers** maintain an **ARP Table** and a **Routing Table** (hosts typically use a default route `0.0.0.0/0` pointing to their Default Gateway).
- **Empty Routing Tables Drop Traffic:** Both the **MAC Address Table** and **ARP Tables** populate dynamically *while traffic flows*. However, **Routing Tables must already be populated before sending traffic**—if a router receives a packet whose Destination IP matches no route, the packet is dropped.

---

## 3. Topology & Pre-Populated Routing Tables (`01:44–04:51`)

The illustration uses **3 Networks**, **3 Hosts** (`Host A`, `Host B`, `Host C`), **1 Switch** on Network 1, and **3 Routers** (`R1`, `R2`, `R3`) connected across the Internet:

```text
   [ Network 1: 11.8.8.0/24 ]                              [ Network 2: 22.7.7.0/24 ]
                                                        +--- [ R2 ] -------------- [ Host B ]
  [ Host A ] ---- (Port 4) [ Switch ] (Port 5) ---- [ R1 ] -- (Internet)    eee2 .1          b2b2 .22
  a1a1  .11                                   eee1  .1   |
                                                        +--- [ R3 ] -------------- [ Host C ]
                                                           eee3 .1          c3c3 .33
                                                           [ Network 3: 33.6.6.0/24 ]
```

> **Simplified MAC Notation:** Only the first 4 hexadecimal digits of each NIC's MAC address (`a1a1`, `b2b2`, `c3c3`, `eee1`, `eee2`, `eee3`) are shown to keep the walkthrough easy to follow.

### Device Addressing Reference Table

| Network | Device | IP Address | MAC Address | Switch Port | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `11.8.8.0/24` | **Host A** | `11.8.8.11` | `a1a1` | Port `4` | `11.8.8.1` (`R1`) |
| `11.8.8.0/24` | **Router 1 (`R1`)** | `11.8.8.1` | `eee1` | Port `5` | *N/A* |
| `22.7.7.0/24` | **Router 2 (`R2`)** | `22.7.7.1` | `eee2` | *(omitted)* | *N/A* |
| `22.7.7.0/24` | **Host B** | `22.7.7.22` | `b2b2` | *(omitted)* | `22.7.7.1` (`R2`) |
| `33.6.6.0/24` | **Router 3 (`R3`)** | `33.6.6.1` | `eee3` | *(omitted)* | *N/A* |
| `33.6.6.0/24` | **Host C** | `33.6.6.33` | `c3c3` | *(omitted)* | `33.6.6.1` (`R3`) |

> **Why Networks 2 & 3 Show Direct Links:** In real networks, **Host B** and **Host C** would normally plug into one or more local switches connected to **R2** and **R3**. The diagram omits those extra switches because Switch operation is already shown on Network 1 (and covered in [`04-How Switches Work/`](./04-How%20Switches%20Work/)).

### Initial State of Tables Before Any Traffic Moves (`02:55–04:51`)
- **MAC Address Table (Switch):** **Empty**
- **ARP Tables (Host A, R1, R2, Host B, R3, Host C):** All **Empty**
- **Routing Tables (Pre-Populated):**
  - **Host A:** `0.0.0.0/0` → `11.8.8.1` *(Default Route: matches all IPv4 addresses, points to R1 for any foreign network)*
  - **Host B:** `0.0.0.0/0` → `22.7.7.1`
  - **Host C:** `0.0.0.0/0` → `33.6.6.1`
  - **R1:** `11.8.8.0/24` → `D.C.` (Directly Connected), `0.0.0.0/0` → `Internet`
  - **R2:** `22.7.7.0/24` → `D.C.` (Directly Connected), `0.0.0.0/0` → `Internet`
  - **R3:** `33.6.6.0/24` → `D.C.` (Directly Connected), `0.0.0.0/0` → `Internet`

> **Abstraction Note:** Normally each router's default route (`0.0.0.0/0`) points to the specific IP address of the next upstream ISP/core router. To keep focus on packet flow, all intermediate router hops and WAN technologies inside the cloud are simplified as `Internet`.

---

## 4. Packet #1: Host A → Host B (`11.8.8.11` → `22.7.7.22`) (`04:51–14:23`)

Inside the **Data Payload** are just **1s and 0s**—arbitrary computer data. The network does not know or care what the payload does; its job is solely to deliver it.

### Step-by-Step Request Journey
1. **Build Layer 3 Header (Host A):** Host A attaches a Layer 3 IP header for **end-to-end delivery**:
   - `Src IP: 11.8.8.11` | `Dst IP: 22.7.7.22`
2. **Local vs. Foreign Check (Host A):** Host A compares `22.7.7.22` with its own IP and subnet mask (`11.8.8.0/24`) and realizes Host B is on a **foreign network**. Host A checks its routing table (`0.0.0.0/0`) and selects its Default Gateway: **`11.8.8.1`** (`R1`).
3. **ARP Check & Hold (Host A):** Host A's ARP table is **empty**, so it does not know `11.8.8.1`'s MAC address and cannot build the Layer 2 header yet. Host A holds the data packet and initiates **ARP**.
4. **Broadcast ARP Request (Host A):** Host A sends an ARP Request frame (`Src MAC: a1a1`, `Dst MAC: ffff` — all F's broadcast MAC) asking who has `11.8.8.1`. *Important:* The ARP Request payload also advertises Host A's own `11.8.8.11 → a1a1` mapping.
5. **Switch Learns & Floods (`Port 4`):**
   - Switch receives the frame on **Port 4** with `Src MAC: a1a1` → **learns** `Port 4 ↔ a1a1` in its MAC Address Table.
   - Switch sees `Dst MAC: ffff` (broadcast) → performs **Flooding** out all other ports (if other hosts were attached, they would receive copies too). Traffic goes out **Port 5** toward **R1**.
6. **R1 Learns Sender & Sends ARP Reply:**
   - **R1** inspects the ARP Request and **learns** Host A's mapping: `11.8.8.11 → a1a1` in R1's ARP table.
   - **R1** sends a **unicast ARP Reply** (`Src MAC: eee1`, `Dst MAC: a1a1`) directly back toward Host A.
7. **Switch Learns & Forwards (`Port 5` → `Port 4`):**
   - Switch receives R1's frame on **Port 5** with `Src MAC: eee1` → **learns** `Port 5 ↔ eee1`.
   - Switch looks up `Dst MAC: a1a1`, finds **Port 4**, and **forwards** the frame out Port 4 to Host A.
8. **Host A Resolves & Encapsulates Data Packet:**
   - **Host A** records `11.8.8.1 → eee1` in its ARP table.
   - Host A now wraps the L3 packet in a Layer 2 Ethernet header for **hop-to-hop delivery**:
     - **L2 Header:** `Src MAC: a1a1` | `Dst MAC: eee1`
     - **L3 Header:** `Src IP: 11.8.8.11` | `Dst IP: 22.7.7.22`
9. **Switch Refreshes & Forwards (`Port 4` → `Port 5`):**
   - Switch receives the data frame on **Port 4**, sees `Port 4 ↔ a1a1` already exists, and **refreshes** that entry's timer.
   - Looks up `Dst MAC: eee1` → **forwards** out **Port 5** to **R1**.
10. **R1 Strips L2 & Routes Toward Internet:**
    - **R1** **strips the Layer 2 header** (hop 1 is complete; that L2 header is no longer needed).
    - Checks Routing Table for unchanged `Dst IP: 22.7.7.22` → matches `0.0.0.0/0 → Internet`.
    - Passed from router to router across WAN technologies through the Internet until it reaches **R2**.
11. **R2 Matches `D.C.` Route & Holds for ARP:**
    - **R2** checks Routing Table for `22.7.7.22` → matches `22.7.7.0/24 → D.C.` (final hop!).
    - R2's ARP table is **empty**, so R2 holds the packet and broadcasts an **ARP Request** (`Src MAC: eee2`, `Dst MAC: ffff`) onto `22.7.7.0/24`.
12. **Host B Learns R2 & Replies:**
    - **Host B** receives R2's ARP Request and **learns** `22.7.7.1 → eee2` in Host B's ARP table.
    - **Host B** sends a **unicast ARP Reply** (`Src MAC: b2b2`, `Dst MAC: eee2`) back to R2.
13. **R2 Learns Host B, Adds New L2 & Delivers:**
    - **R2** records `22.7.7.22 → b2b2` in its ARP table.
    - **R2** builds a **new Layer 2 header** (`Src MAC: eee2`, `Dst MAC: b2b2`) around the original Layer 3 packet and transmits it.
14. **Final Receipt (Host B):** **Host B** **strips L2**, **strips L3**, and passes the **1s and 0s payload** up for processing.

---

## 5. Response #1: Host B → Host A (`22.7.7.22` → `11.8.8.11`) (`14:23–17:28`)

> **Why This Return Trip Is Much Faster:** Every table along this path (**Host B ARP**, **R1 ARP**, and **Switch MAC Table**) is already populated from the request! **Zero ARP broadcasts occur.**

1. **Build Response (Host B):** Creates response payload (1s and 0s), attaches L3 header (`Src IP: 22.7.7.22`, `Dst IP: 11.8.8.11`). Sees foreign network → targets Default Gateway `22.7.7.1` (`R2`).
2. **ARP Hit & Transmit (Host B):** Finds `22.7.7.1 → eee2` in ARP table → attaches L2 header (`Src MAC: b2b2`, `Dst MAC: eee2`) → sends to **R2**.
3. **R2 Forwards to Internet:** **R2** strips L2, looks up `11.8.8.11` → matches `0.0.0.0/0 → Internet` → travels across routers to **R1**.
4. **R1 Matches `D.C.` & ARP Hit:** **R1** looks up `11.8.8.11` → matches `11.8.8.0/24 → D.C.` → checks ARP table and already knows `11.8.8.11 → a1a1` → attaches new L2 header (`Src MAC: eee1`, `Dst MAC: a1a1`) → transmits into Switch **Port 5**.
5. **Switch Refreshes & Forwards (`Port 5` → `Port 4`):** Switch sees `Src MAC: eee1` on Port 5 (already known → **refreshes** entry), sees `Dst MAC: a1a1` → **forwards** out **Port 4** to **Host A**.
6. **Delivery (Host A):** **Host A** strips L2 (NIC-to-NIC delivery complete), strips L3 (end-to-end delivery complete), and processes the response payload.

---

## 6. Packet #2: Host A → Host C (`11.8.8.11` → `33.6.6.33`) (`17:28–21:34`)

If this starts to feel repetitive—that is a great sign! Every device obeys the exact same deterministic rules.

1. **Build & Transmit (Host A):** Attaches L3 (`Src IP: 11.8.8.11`, `Dst IP: 33.6.6.33`). Foreign network → targets Default Gateway `11.8.8.1` (`R1`). Host A already has `11.8.8.1 → eee1` cached (**no ARP needed!**) → attaches L2 (`Src MAC: a1a1`, `Dst MAC: eee1`) → transmits to Switch **Port 4**.
2. **Switch Refreshes & Forwards:** Sees `a1a1` on Port 4 (**refreshes** entry), forwards `eee1` out **Port 5** to **R1**.
3. **R1 Routes Across Internet:** **R1** strips L2, matches `33.6.6.33` to `0.0.0.0/0 → Internet`, forwards across Internet routers to **R3**.
4. **R3 Matches `D.C.` & Triggers ARP (`R3` ARP Table Is Empty):**
   - **R3** matches `33.6.6.33` to `33.6.6.0/24 → D.C.`
   - R3's ARP table is empty → holds packet and broadcasts **ARP Request** (`Src MAC: eee3`, `Dst MAC: ffff`).
   - **Host C** receives ARP Request, **learns** `33.6.6.1 → eee3` in Host C's ARP table, and sends **unicast ARP Reply** (`Src MAC: c3c3`, `Dst MAC: eee3`).
   - **R3** **learns** `33.6.6.33 → c3c3` in R3's ARP table.
5. **R3 Encapsulates & Delivers to Host C:** R3 wraps the untouched L3 packet inside a new L2 header (`Src MAC: eee3`, `Dst MAC: c3c3`) and transmits to **Host C**. **Host C** strips L2, strips L3, and processes the data.

---

## 7. Response #2: Host C → Host A (`33.6.6.33` → `11.8.8.11`) (`21:34–23:59`)

Again, **zero ARP needed on either end**—all entries are cached!

1. **Host C:** Attaches L3 (`Src IP: 33.6.6.33`, `Dst IP: 11.8.8.11`), finds gateway `33.6.6.1 → eee3` in ARP table, attaches L2 (`Src MAC: c3c3`, `Dst MAC: eee3`), sends to **R3**.
2. **R3:** Strips L2, matches `11.8.8.11` to `0.0.0.0/0 → Internet`, forwards across Internet to **R1**.
3. **R1:** Matches `11.8.8.0/24 → D.C.`, finds `11.8.8.11 → a1a1` in ARP table, attaches new L2 (`Src MAC: eee1`, `Dst MAC: a1a1`), sends to Switch **Port 5**.
4. **Switch:** Refreshes `Port 5 ↔ eee1`, looks up `a1a1`, forwards out **Port 4** to **Host A**.
5. **Host A:** Strips L2, strips L3, processes response from Host C — **Mission Successful!**

---

## 8. Final Populated Tables & Header Behavior Summary

### Final State of All Device Tables After Both Exchanges

| Device | Table Type | Final Entries |
| :--- | :--- | :--- |
| **Switch** | MAC Address Table | `Port 4 ↔ a1a1` \| `Port 5 ↔ eee1` |
| **Host A** | ARP Table / Routing Table | ARP: `11.8.8.1 → eee1` \| Routing: `0.0.0.0/0 → 11.8.8.1` |
| **R1** | ARP Table / Routing Table | ARP: `11.8.8.11 → a1a1` \| Routing: `11.8.8.0/24 → D.C.`, `0.0.0.0/0 → Internet` |
| **R2** | ARP Table / Routing Table | ARP: `22.7.7.22 → b2b2` \| Routing: `22.7.7.0/24 → D.C.`, `0.0.0.0/0 → Internet` |
| **Host B** | ARP Table / Routing Table | ARP: `22.7.7.1 → eee2` \| Routing: `0.0.0.0/0 → 22.7.7.1` |
| **R3** | ARP Table / Routing Table | ARP: `33.6.6.33 → c3c3` \| Routing: `33.6.6.0/24 → D.C.`, `0.0.0.0/0 → Internet` |
| **Host C** | ARP Table / Routing Table | ARP: `33.6.6.1 → eee3` \| Routing: `0.0.0.0/0 → 33.6.6.1` |

### Header Changes Across Hops (L2 Changes, L3 Stays Intact)

| Exchange | Hop | L2 Header (`Src MAC → Dst MAC`) | L3 Header (`Src IP → Dst IP`) |
| :--- | :--- | :--- | :--- |
| **Packet #1** (`A → B`) | Host A → Switch → R1 | `a1a1 → eee1` *(New L2)* | `11.8.8.11 → 22.7.7.22` |
| **Packet #1** (`A → B`) | R2 → Host B | `eee2 → b2b2` *(Rewritten L2)* | `11.8.8.11 → 22.7.7.22` *(Unchanged)* |
| **Response #1** (`B → A`) | Host B → R2 | `b2b2 → eee2` *(New L2)* | `22.7.7.22 → 11.8.8.11` |
| **Response #1** (`B → A`) | R1 → Switch → Host A | `eee1 → a1a1` *(Rewritten L2)* | `22.7.7.22 → 11.8.8.11` *(Unchanged)* |
| **Packet #2** (`A → C`) | Host A → Switch → R1 | `a1a1 → eee1` *(New L2)* | `11.8.8.11 → 33.6.6.33` |
| **Packet #2** (`A → C`) | R3 → Host C | `eee3 → c3c3` *(Rewritten L2)* | `11.8.8.11 → 33.6.6.33` *(Unchanged)* |
| **Response #2** (`C → A`) | Host C → R3 | `c3c3 → eee3` *(New L2)* | `33.6.6.33 → 11.8.8.11` |
| **Response #2** (`C → A`) | R1 → Switch → Host A | `eee1 → a1a1` *(Rewritten L2)* | `33.6.6.33 → 11.8.8.11` *(Unchanged)* |

---

## 9. Classic Interview Question: *"What Happens When You Type `www.google.com` Into a Web Browser?"* (`23:59–26:46`)

Everything in the simulation above was designed to answer this classic networking and security interview question. Browsing to a website happens in **two main phases**:

1. **Phase 1 — DNS Lookup (Resolving Name → IP):**
   - Think of **Packet #1 & Response #1 (`Host A ↔ Host B`)** as a **DNS Query** sent to a **DNS Server (`Host B`)**.
   - The DNS Server replies with the IP address of the website you want to visit (`33.6.6.33`).
2. **Phase 2 — HTTP/HTTPS Request (Fetching the Webpage):**
   - Think of **Packet #2 & Response #2 (`Host A ↔ Host C`)** as an **HTTP/HTTPS Request** sent to the **Web Server (`Host C`)** at `33.6.6.33`.
   - The Web Server returns the webpage response back across the routers and switch to your browser (**Host A**).

### Structured 6-Step Interview Answer Checklist
1. **Host Config (DHCP):** Host already obtained its 4 required parameters (**IP Address**, **Subnet Mask**, **Default Gateway**, **DNS Server IP**) via DHCP (`DORA`).
2. **DNS Resolution:** Browser needs the web server's IP → builds a DNS request to the DNS Server (`22.7.7.22`).
3. **Local vs. Foreign + Gateway ARP:** Host sees `22.7.7.22` is on a foreign network → targets Default Gateway (`11.8.8.1`) → uses ARP broadcast/unicast reply if the gateway MAC is not yet cached.
4. **Local Switching + Multi-Router Internet Hops:** Switch learns Source MAC and forwards toward R1 → R1 strips L2, matches routing table (`0.0.0.0/0`), rewrites L2 for next hop across Internet routers → R2 resolves target MAC via ARP if needed and delivers to DNS Server → DNS Server returns web server IP (`33.6.6.33`).
5. **HTTP/HTTPS Request to Web Server:** Host builds packet to `33.6.6.33` (reusing cached Default Gateway MAC `eee1`—no ARP needed) → traverses Switch, R1, Internet, R3 → R3 resolves Host C MAC if needed and delivers to Web Server.
6. **Response & Troubleshooting Mindset:** Web server sends webpage back cleanly over populated tables. *Why this matters in real life:* Whenever someone cannot reach email or a website, tracing these exact steps tells you where the failure is—host config, DNS, ARP, switch port, default gateway/routing, or the destination server.

---

## 10. 60-Second Fast-Revision Cheat Sheet (All Folders Index)

| Folder / Lab | Core Takeaways for Quick Revision |
| :--- | :--- |
| [`01-How the Data Flows`](./01-How%20the%20Data%20Flows/) | Hosts act as clients/servers with 32-bit IPv4 addresses; Repeaters/Hubs regenerate signals (shared medium), Bridges/Switches forward locally, Routers connect networks & act as gateways. |
| [`02-OSI Model`](./02-OSI%20Model/) | Layers 1–7 (`Physical → Application`): **L2 MAC** gives hop-to-hop NIC delivery; **L3 IP** gives end-to-end delivery; **L4 TCP/UDP Ports** separate application streams via the 4-tuple; encapsulation wraps down and strips up. |
| [`03-Everything Hosts Do`](./03-Everything%20Hosts%20Do/) | Hosts use IP + Subnet Mask to check *same network vs. foreign network*, ARP to learn local/gateway MACs, DNS to resolve names, and send off-subnet traffic to the Default Gateway. |
| [`04-How Switches Work`](./04-How%20Switches%20Work/) | Switches build a **MAC Address Table** (`Port ↔ MAC`) and perform 3 actions: **Learn** (Source MAC), **Forward** (known unicast Dst MAC), **Flood** (unknown unicast or broadcast `FF:FF:FF:FF:FF:FF`). Segments traffic with **VLANs** & **802.1Q Trunks**. |
| [`05-Everything Routers Do`](./05-Everything%20Routers%20Do/) | Routers connect **nodes/networks**, use **Routing Tables** (`C`/`L`, Static `S`, Dynamic `RIP/OSPF/EIGRP/IS-IS/BGP`), **Longest Prefix Match**, **Route Summarization**, and **ARP Tables** to rewrite L2 per hop while keeping L3 intact. |
| [`06-Network Protocols`](./06-Network%20Protocols/) | Standardized rules (**RFCs**): **ARP** (`RFC 826`), **FTP** (`20/21`), **SMTP** (`25/587/465`), **HTTP** (`80`), **SSL/TLS & HTTPS** (`443`), **DNS** (`53`), **DHCP** (`67/68` DORA assigns IP, Mask, Gateway, DNS). |
| [`two-subnets-router-lab.pkt`](./two-subnets-router-lab.pkt) | Packet Tracer hands-on lab: 2 PCs (`192.168.1.10`, `.11`) on Switch 1 + 1 Server (`192.168.2.10`) on independent Switch 2, connected via Router gateways (`192.168.1.1` / `192.168.2.1`) and verified with `ping`. |
