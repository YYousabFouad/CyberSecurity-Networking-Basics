# Packet Traveling

How data travels from one host to another across switches, routers, and the Internet—bringing together the core concepts from all study folders in this repository for fast revision.

---

## 1. Core Rules of Packet Traveling

- **Layer 3 (IP Addresses) — End-to-End Delivery:**
  - `Source IP` and `Destination IP` identify the original sender and final receiver.
  - IP addresses stay unchanged across the entire journey from sender to receiver.
- **Layer 2 (MAC Addresses) — Hop-to-Hop Delivery:**
  - `Source MAC` and `Destination MAC` move a frame across one local link at a time.
  - Layer 2 headers are stripped off and recreated at every router along the path.
- **Data Payload:**
  - Simply `1`s and `0`s—the network does not know or care what the application data does; its only job is delivering it.

---

## 2. The Three Core Tables

Understanding how packets flow through a network comes down to three tables:

### MAC Address Table (Switches)
- Maps **Switch Port ↔ MAC Address**.
- Populated **dynamically** from the **Source MAC** of incoming frames (refreshed if already present).
- Checked against the **Destination MAC** to **Forward** (known unicast) or **Flood** (broadcast `ffff` or unknown unicast).
- *Pass-through note:* Because traffic moves *through* a switch (not *to* or *from* it), the switch's own MAC and IP addresses do not matter—only its ports do.

### ARP Table / ARP Cache (Hosts & Routers)
- Maps **IP Address ↔ MAC Address** on a local network.
- Starts **empty** and fills **dynamically** as needed via **ARP**:
  - **ARP Request (Broadcast `ffff`):** Asks everyone on the local network for the MAC owning a target IP, and also carries the sender's own `IP → MAC` mapping.
  - **ARP Reply (Unicast):** The owner of the target IP replies directly with its MAC address.

### Routing Table (Hosts & Routers)
- Maps **IP Network → Directly Connected (`D.C.`)** or **Next-Hop IP / Internet**.
- **Must be populated ahead of time**—if a router receives a packet whose Destination IP matches no route, the packet is **dropped**.
- **Hosts also have a Routing Table:** Usually a **Default Route (`0.0.0.0/0`)** covering all IPv4 addresses and pointing to the host's **Default Gateway**.

---

## 3. What Each Device Does

### Host
1. Creates the Layer 3 packet (`Source IP`, `Destination IP`).
2. Compares `Destination IP` with its own IP and subnet mask to decide if the destination is **local** or on a **foreign network**.
3. Targets the destination host directly (if local) or its **Default Gateway (`0.0.0.0/0`)** (if foreign).
4. Checks its **ARP Table** for that next target's MAC address—if missing, holds the packet and runs ARP first.
5. Wraps the packet in a Layer 2 header (`Source MAC = host`, `Destination MAC = local host or gateway`) and sends it onto the wire.
6. *When receiving:* Strips Layer 2 → strips Layer 3 → passes the payload up for processing.

### Switch
1. Looks at the incoming port and **Source MAC**, then learns or refreshes `Port ↔ Source MAC` in its **MAC Address Table**.
2. Looks up the **Destination MAC**:
   - **Known unicast:** **Forwards** the frame out only the single matching port.
   - **Broadcast (`ffff`) or unknown unicast:** **Floods** the frame out all other active ports.

### Router
1. Receives the frame and **strips off the incoming Layer 2 header** (hop complete).
2. Inspects the unchanged **Destination IP** and searches its **Routing Table**:
   - **Directly Connected (`D.C.`):** Target is on a local network attached to the router.
   - **Next-Hop / Default Route (`0.0.0.0/0`):** Forward toward another router.
   - **No match:** **Drops** the packet.
3. Checks its **ARP Table** for the next-hop router's MAC (or final host's MAC if `D.C.`)—if missing, holds the packet and runs ARP.
4. Attaches a **brand-new Layer 2 header** (`Source MAC = outgoing interface`, `Destination MAC = next-hop or host MAC`) around the untouched Layer 3 packet and transmits it.

---

## 4. End-to-End Example Across Networks

Below is a topology connecting **3 Networks**, **3 Hosts** (`Host A`, `Host B`, `Host C`), **1 Switch** (`Port 4` and `Port 5`), and **3 Routers** (`R1`, `R2`, `R3`) across the Internet:

```text
Network 1 (11.8.8.0/24)
  Host A (.11 | a1a1) --- [P4 Switch P5] --- R1 (.1 | eee1)
                                                  |
                                            ( Internet )
                                              |      |
   Network 2 (22.7.7.0/24)                    |      |
     R2 (.1 | eee2) --- Host B (.22 | b2b2) <-+      |
                                                     |
   Network 3 (33.6.6.0/24)                           |
     R3 (.1 | eee3) --- Host C (.33 | c3c3) <--------+
```

> *Notes on the diagram:* MAC addresses are shortened to their first 4 hex digits (`a1a1`, `b2b2`, `c3c3`, `eee1`, `eee2`, `eee3`). In real networks, switches would also connect `Host B` to `R2` and `Host C` to `R3`. `( Internet )` abstracts the intermediate ISP routers and WAN links between `R1`, `R2`, and `R3`.

### Starting State
- **Switch MAC Table:** Empty.
- **All ARP Tables:** Empty.
- **Pre-Populated Routing Tables:**
  - `Host A`: `0.0.0.0/0 → 11.8.8.1`
  - `Host B`: `0.0.0.0/0 → 22.7.7.1`
  - `Host C`: `0.0.0.0/0 → 33.6.6.1`
  - `R1`: `11.8.8.0/24 → D.C.` and `0.0.0.0/0 → Internet`
  - `R2`: `22.7.7.0/24 → D.C.` and `0.0.0.0/0 → Internet`
  - `R3`: `33.6.6.0/24 → D.C.` and `0.0.0.0/0 → Internet`

### A. First Request: Host A → Host B (`11.8.8.11 → 22.7.7.22`) — Cold Tables
- **Host A resolves Default Gateway (`R1`) via ARP:**
  - Host A adds Layer 3 (`11.8.8.11 → 22.7.7.22`), sees `22.7.7.22` is on a foreign network, and targets its gateway `11.8.8.1`.
  - Because Host A's ARP table is empty, it holds the packet and broadcasts an **ARP Request** (`Src MAC: a1a1`, `Dst MAC: ffff`) containing `11.8.8.11 → a1a1`.
  - **Switch** learns `Port 4 ↔ a1a1` and **floods** the broadcast out **Port 5** toward `R1` (any other attached hosts would receive a copy too).
  - **R1** learns `11.8.8.11 → a1a1` from the request and sends a **unicast ARP Reply** (`Src MAC: eee1`, `Dst MAC: a1a1`).
  - **Switch** learns `Port 5 ↔ eee1` and **forwards** out **Port 4** to Host A; Host A records `11.8.8.1 → eee1`.
- **Packet leaves Network 1 (`Hop 1`):**
  - Host A encapsulates the data and sends it onto the wire:

```text
[ L2: a1a1 -> eee1 ] [ L3: 11.8.8.11 -> 22.7.7.22 ] [ Data ]
```

  - **Switch** sees `a1a1` on Port 4 (**refreshes** `Port 4 ↔ a1a1`) and **forwards** out **Port 5** to `R1`.
- **R1 routes across the Internet:**
  - **R1** strips the old Layer 2 header (`a1a1 → eee1`), matches `22.7.7.22` to `0.0.0.0/0 → Internet`, and forwards it router-to-router across the Internet to `R2`.
- **R2 resolves Host B via ARP and delivers (`Final Hop`):**
  - **R2** sees `22.7.7.22` matches `22.7.7.0/24 → D.C.` (Directly Connected).
  - R2's ARP table is empty, so it broadcasts an **ARP Request** (`Src MAC: eee2`, `Dst MAC: ffff`); **Host B** learns `22.7.7.1 → eee2` and sends a **unicast ARP Reply** (`b2b2 → eee2`); **R2** learns `22.7.7.22 → b2b2`.
  - **R2** builds a new Layer 2 header around the unchanged Layer 3 packet and delivers it to Host B:

```text
[ L2: eee2 -> b2b2 ] [ L3: 11.8.8.11 -> 22.7.7.22 ] [ Data ]
```

  - **Host B** strips Layer 2, strips Layer 3, and processes the data payload.

### B. Return Response: Host B → Host A (`22.7.7.22 → 11.8.8.11`) — Cached Tables (No ARP)
Because **Host B**, **R1**, and the **Switch** already populated their tables during the request, **no ARP broadcasts are needed**:

- **Host B:** Sees `11.8.8.11` is foreign → already knows gateway `22.7.7.1 → eee2` → sends `[ L2: b2b2 -> eee2 ] [ L3: 22.7.7.22 -> 11.8.8.11 ] [ Data ]` to `R2`.
- **R2:** Strips Layer 2 → matches `0.0.0.0/0 → Internet` → forwards across the Internet to `R1`.
- **R1:** Matches `11.8.8.0/24 → D.C.` → already knows `11.8.8.11 → a1a1` → attaches new Layer 2 (`eee1 → a1a1`) and sends to Switch **Port 5**.
- **Switch:** Refreshes `Port 5 ↔ eee1` and **forwards** out **Port 4** to Host A.
- **Host A:** Strips Layer 2, strips Layer 3, and processes Host B's response.

### C. Talking to a New Destination: Host A ↔ Host C (`11.8.8.11 ↔ 33.6.6.33`)
The exact same device rules repeat—only unvisited links need ARP:

- **Packet #2 (`Host A → Host C`):**
  - Network 1 is already cached (**0 ARP**): Host A sends directly to `R1` (`eee1`), Switch refreshes `Port 4` and forwards out `Port 5`, and `R1` strips Layer 2 and routes across the Internet to `R3`.
  - `R3` matches `33.6.6.0/24 → D.C.` and has an empty ARP table, so it runs **one ARP exchange** with `Host C` (`Host C` learns `33.6.6.1 → eee3`, `R3` learns `33.6.6.33 → c3c3`).
  - `R3` delivers `[ L2: eee3 -> c3c3 ] [ L3: 11.8.8.11 -> 33.6.6.33 ] [ Data ]` to `Host C`, which strips Layer 2 and Layer 3.
- **Response #2 (`Host C → Host A`):**
  - **Zero ARP anywhere:** `Host C` sends to `R3` (`c3c3 → eee3`), `R3` routes across the Internet to `R1`, `R1` reuses `11.8.8.11 → a1a1` (`eee1 → a1a1`), Switch refreshes `Port 5` and forwards out `Port 4`, and `Host A` receives the response.

### D. Final Populated Tables

| Device | MAC / ARP Table | Routing Table |
| :--- | :--- | :--- |
| **Switch** | `Port 4 ↔ a1a1`, `Port 5 ↔ eee1` | *(Layer 2 only — no Routing Table)* |
| **Host A** | `11.8.8.1 ↔ eee1` | `0.0.0.0/0 → 11.8.8.1` |
| **R1** | `11.8.8.11 ↔ a1a1` | `11.8.8.0/24 → D.C.`, `0.0.0.0/0 → Internet` |
| **R2** | `22.7.7.22 ↔ b2b2` | `22.7.7.0/24 → D.C.`, `0.0.0.0/0 → Internet` |
| **Host B** | `22.7.7.1 ↔ eee2` | `0.0.0.0/0 → 22.7.7.1` |
| **R3** | `33.6.6.33 ↔ c3c3` | `33.6.6.0/24 → D.C.`, `0.0.0.0/0 → Internet` |
| **Host C** | `33.6.6.1 ↔ eee3` | `0.0.0.0/0 → 33.6.6.1` |

---

## 5. What Happens When You Type `www.google.com` Into a Browser?

Everything in the example above directly explains how visiting a website works in **two phases**:

- **Phase 1 — DNS Lookup (`Host A ↔ Host B` as DNS Server):**
  - The browser needs the IP address for `www.google.com`, so `Host A` sends a **DNS Request** to the DNS Server (`Host B`) and receives the web server's IP (`33.6.6.33`).
- **Phase 2 — HTTP/HTTPS Request (`Host A ↔ Host C` as Web Server):**
  - `Host A` sends an **HTTP/HTTPS Request** to the Web Server (`Host C` at `33.6.6.33`) and receives the web page back.

### Full Sequence at a Glance
1. **Host Configuration:** The host already has its **IP Address**, **Subnet Mask**, **Default Gateway**, and **DNS Server IP** (typically assigned via DHCP).
2. **DNS Query:** Host sends a DNS query toward the DNS Server (`22.7.7.22`) through its Default Gateway (`R1`), using ARP if the gateway's MAC is not yet cached.
3. **DNS Response:** Switch forwards through `R1`, routers swap Layer 2 across the Internet to `R2`, `R2` resolves the DNS Server via ARP if needed, and the DNS reply returns `33.6.6.33`.
4. **Web Request:** Host builds an HTTP/HTTPS packet to `33.6.6.33`, reuses `R1`'s cached MAC, crosses the Internet to `R3`, and reaches the Web Server (`Host C`).
5. **Web Response:** The Web Server sends the website data back over the already-populated ARP and Switch MAC tables.

### Troubleshooting Takeaway
Knowing each expected step lets you isolate where a broken website or email connection fails: **host IP/mask/gateway/DNS settings**, **DNS resolution**, **ARP**, a **switch port**, a **missing route**, or the **destination server** itself.

---

## 6. Quick Summary Across All Folders

- [`01-How the Data Flows`](./01-How%20the%20Data%20Flows/) — Hosts (clients/servers), 32-bit IPv4 addresses, repeaters/hubs vs. bridges/switches, routers, and gateways.
- [`02-OSI Model`](./02-OSI%20Model/) — Layers 1–7, L2 MAC (hop-to-hop) vs. L3 IP (end-to-end), ARP, TCP/UDP ports (4-tuple), and encapsulation/decapsulation.
- [`03-Everything Hosts Do`](./03-Everything%20Hosts%20Do/) — Same-network vs. foreign-network checks, subnetting/CIDR, ARP resolution, DNS, and forwarding via Default Gateways.
- [`04-How Switches Work`](./04-How%20Switches%20Work/) — MAC address tables, Learn / Forward / Flood, unicast vs. broadcast frames, VLANs, and 802.1Q trunking.
- [`05-Everything Routers Do`](./05-Everything%20Routers%20Do/) — Network nodes, RFCs, routing and ARP tables, `C`/`S`/dynamic routes, hierarchy, route summarization, Longest Prefix Match, and `0.0.0.0/0`.
- [`06-Network Protocols`](./06-Network%20Protocols/) — ARP (`RFC 826`), FTP, SMTP, HTTP, SSL/TLS, HTTPS, DNS, DHCP (`DORA`), and the 4 host connectivity settings.
- [`two-subnets-router-lab.pkt`](./two-subnets-router-lab.pkt) — Packet Tracer lab connecting two PCs (`192.168.1.10`, `.11`) on Switch 1 and one Server (`192.168.2.10`) on independent Switch 2 through a Router (`192.168.1.1` / `192.168.2.1`), verified with `ping`.
