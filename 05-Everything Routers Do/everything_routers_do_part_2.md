# Everything Routers Do (Part 2): How Routers Forward Packets

* **Course:** Practical Networking — *Networking Fundamentals*
* **Lesson:** Lesson 5 (Part 2) — Video #10: *Everything Routers do - Part 2 - How Routers forward Packets*
* **Core Focus:** Router ARP Tables, Routing Table + ARP Table packet forwarding walkthrough (Request & Response), Multi-Router Scaling, Layer 2 vs. Layer 3 Header Behavior, and What the Internet Actually Is

---

## 1. Routers Also Have ARP Tables

Just like Hosts (PCs and Servers), **Routers maintain an ARP Table** (Address Resolution Protocol Table).

* **What an ARP Table Is:** A mapping of **Layer 3 addresses (IPv4 Addresses)** to **Layer 2 addresses (MAC Addresses)**.
* **Universal Rule:** **Everything with an IP address has an ARP Table** — including PCs, servers, phones, and every router interface configured with an IP.
* **Initial State & Population:**
  * **ARP Tables start out empty** when devices boot up.
  * They are **populated dynamically as needed** when network traffic requires sending a packet to a specific next-hop or local host IP address whose MAC address is not yet known.

### Why Does a Router Need an ARP Table?
* A router's **Routing Table** tells it **which network** to reach and **which exit interface / next-hop IP** to send the packet toward (Layer 3 decision).
* However, to actually transmit bits onto a wire (Layer 1), the router must wrap the Layer 3 IP packet inside a **Layer 2 Ethernet frame** with a **Destination MAC address**.
* The router uses its **ARP Table** (and ARP Broadcasts if the entry is missing) to resolve that next-hop IP or final destination host IP into a Layer 2 MAC address.

---

## 2. Network Topology: Using the Routing Table & ARP Table

Below is the multi-router topology from the lesson connecting **Network A** (`10.1.1.0/24`), a transit link **Network B** (`10.2.2.0/24`), and **Network C** (`10.3.3.0/24`).

```text
  [ Network A: 10.1.1.0/24 ]          [ Network B: 10.2.2.0/24 ]          [ Network C: 10.3.3.0/24 ]

+------------+                     +--------------+                     +--------------+                     +------------+
|   Host A   |                     |      R1      |                     |      R2      |                     |   Host C   |
|            |                     |              |                     |              |                     |            |
| IP: .1.10  |-------------------->| .1.1   .2.1  |-------------------->| .2.2   .3.1  |-------------------->| IP: .3.30  |
| MAC: AAAA  |        (eth0)       | R1e0   R1e1  |       (eth1)        | R2e0   R2e1  |       (eth0)        | MAC: CCCC  |
| GW: 10.1.1.1                     +--------------+                     +--------------+                     | GW: 10.3.3.1
+------------+                                                                                               +------------+
```

### Device Addressing Reference Table

| Device | Interface / Role | IP Address | MAC Address | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Host A** | Network A Host | `10.1.1.10` | `AAAA` | `10.1.1.1` (R1) |
| **Router 1 (R1)** | Left Interface (`eth0` — Network A) | `10.1.1.1` | `R1e0` | N/A |
| **Router 1 (R1)** | Right Interface (`eth1` — Network B) | `10.2.2.1` | `R1e1` | N/A |
| **Router 2 (R2)** | Left Interface (`eth0` — Network B) | `10.2.2.2` | `R2e0` | N/A |
| **Router 2 (R2)** | Right Interface (`eth1` — Network C) | `10.3.3.1` | `R2e1` | N/A |
| **Host C** | Network C Host | `10.3.3.30` | `CCCC` | `10.3.3.1` (R2) |

### Routing Tables on R1 and R2

**R1 Routing Table:**

| Destination Network | Next-Hop / Action | Exit Interface |
| :--- | :--- | :--- |
| `10.1.1.0/24` | Directly Connected | `eth0` (`10.1.1.1`) |
| `10.2.2.0/24` | Directly Connected | `eth1` (`10.2.2.1`) |
| `10.3.3.0/24` | Next-Hop: `10.2.2.2` (R2) | `eth1` (`10.2.2.1`) |

**R2 Routing Table:**

| Destination Network | Next-Hop / Action | Exit Interface |
| :--- | :--- | :--- |
| `10.2.2.0/24` | Directly Connected | `eth0` (`10.2.2.2`) |
| `10.3.3.0/24` | Directly Connected | `eth1` (`10.3.3.1`) |
| `10.1.1.0/24` | Next-Hop: `10.2.2.1` (R1) | `eth0` (`10.2.2.2`) |

> **Initial State:** All ARP Tables on **Host A**, **R1**, **R2**, and **Host C** start **completely empty**.

---

## 3. Every Single Step to Request Data (Host A $\rightarrow$ Host C)

**Goal:** Host A (`10.1.1.10`) wants to send data to Host C (`10.3.3.30`).

### Phase 1: Host A Prepares and Sends the Packet to R1
1. **Create Layer 3 Packet:** Host A builds an IP packet with **Source IP = `10.1.1.10`** and **Destination IP = `10.3.3.30`**.
2. **Local vs. Foreign Check:** Host A compares its own network (`10.1.1.0/24`) with the Destination IP (`10.3.3.30`). They are on **different networks**, so Host A knows it must send the packet to its **Default Gateway (`10.1.1.1`)**.
3. **Host A ARP Table Lookup:** Host A checks its ARP table for `10.1.1.1`. It is **empty**.
4. **ARP Request & Reply (Network A):**
   * Host A broadcasts an **ARP Request** on Network A: *"Who has IP `10.1.1.1`? Tell `10.1.1.10` (`AAAA`)."*
   * **R1** receives the ARP Request, learns Host A's mapping (`10.1.1.10` $\rightarrow$ `AAAA`) in R1's ARP table, and sends a unicast **ARP Reply** back to `AAAA`: *"I am `10.1.1.1`, and my MAC is `R1e0`."*
   * Host A records `10.1.1.1` $\rightarrow$ `R1e0` in its ARP table.
5. **Encapsulate & Transmit (Hop 1):** Host A wraps the L3 packet inside a Layer 2 Ethernet frame and transmits it on the wire:
   * **L2 Header:** `Src MAC: AAAA` | `Dst MAC: R1e0`
   * **L3 Header:** `Src IP: 10.1.1.10` | `Dst IP: 10.3.3.30`

### Phase 2: Router 1 (R1) Receives, Looks Up, and Forwards to R2
6. **Receive & Strip Layer 2:** R1 receives the frame on `eth0`, sees `Dst MAC: R1e0` matches its interface, and **strips off (de-encapsulates) the old Layer 2 header**.
7. **Inspect Layer 3 & Routing Table Lookup:** R1 inspects `Dst IP: 10.3.3.30` and searches its Routing Table:
   * Match found: `10.3.3.0/24` $\rightarrow$ **Next-Hop IP = `10.2.2.2`** via **Exit Interface = `eth1`**.
8. **R1 ARP Table Lookup:** R1 checks its ARP table for the next-hop IP `10.2.2.2`. It is **not found**.
9. **ARP Request & Reply (Network B):**
   * R1 broadcasts an **ARP Request** out `eth1` onto Network B: *"Who has IP `10.2.2.2`? Tell `10.2.2.1` (`R1e1`)."*
   * **R2** receives the ARP Request, adds R1's mapping (`10.2.2.1` $\rightarrow$ `R1e1`) to R2's ARP table, and sends a unicast **ARP Reply** back to `R1e1`: *"I am `10.2.2.2`, and my MAC is `R2e0`."*
   * R1 records `10.2.2.2` $\rightarrow$ `R2e0` in its ARP table.
10. **Re-encapsulate & Transmit (Hop 2):** R1 builds a **brand-new Layer 2 header** around the original Layer 3 packet and sends it out `eth1`:
    * **L2 Header:** `Src MAC: R1e1` | `Dst MAC: R2e0` *(Updated for Hop 2)*
    * **L3 Header:** `Src IP: 10.1.1.10` | `Dst IP: 10.3.3.30` *(Unchanged)*

### Phase 3: Router 2 (R2) Receives, Looks Up, and Delivers to Host C
11. **Receive & Strip Layer 2:** R2 receives the frame on `eth0`, verifies `Dst MAC: R2e0` matches its interface, and **strips off the Layer 2 header**.
12. **Inspect Layer 3 & Routing Table Lookup:** R2 inspects `Dst IP: 10.3.3.30` and searches its Routing Table:
    * Match found: `10.3.3.0/24` $\rightarrow$ **Directly Connected** via **Exit Interface = `eth1`**.
    * Because the destination network is directly connected, R2 knows the packet is going directly to the final destination IP (`10.3.3.30`) rather than another router.
13. **R2 ARP Table Lookup:** R2 checks its ARP table for `10.3.3.30`. It is **not found**.
14. **ARP Request & Reply (Network C):**
    * R2 broadcasts an **ARP Request** out `eth1` onto Network C: *"Who has IP `10.3.3.30`? Tell `10.3.3.1` (`R2e1`)."*
    * **Host C** receives the ARP Request, adds R2's mapping (`10.3.3.1` $\rightarrow$ `R2e1`) to Host C's ARP table, and sends a unicast **ARP Reply** back to `R2e1`: *"I am `10.3.3.30`, and my MAC is `CCCC`."*
    * R2 records `10.3.3.30` $\rightarrow$ `CCCC` in its ARP table.
15. **Re-encapsulate & Transmit (Hop 3):** R2 builds a **new Layer 2 header** around the original Layer 3 packet and sends it out `eth1`:
    * **L2 Header:** `Src MAC: R2e1` | `Dst MAC: CCCC` *(Updated for Hop 3)*
    * **L3 Header:** `Src IP: 10.1.1.10` | `Dst IP: 10.3.3.30` *(Unchanged)*
16. **Final Delivery at Host C:** Host C receives the frame, verifies `Dst MAC: CCCC` matches its NIC, strips L2, verifies `Dst IP: 10.3.3.30` matches its IP, and hands the data up to Layer 4+ for processing.

---

## 4. Every Single Step to Respond with Data (Host C $\rightarrow$ Host A)

**Goal:** Host C (`10.3.3.30`) sends the response packet back to Host A (`10.1.1.10`). Notice how **ARP tables are now already populated** from the request journey!

### Phase 1: Host C Prepares and Sends the Response to R2
1. **Create Layer 3 Response Packet:** Host C builds an IP packet with **Source IP = `10.3.3.30`** and **Destination IP = `10.1.1.10`**.
2. **Local vs. Foreign Check:** Host C sees `10.1.1.10` is on a foreign network (`10.1.1.0/24` $\neq$ `10.3.3.0/24`), so it targets its **Default Gateway (`10.3.3.1`)**.
3. **Host C ARP Table Lookup (Cache Hit!):** Host C checks its ARP table for `10.3.3.1` and **immediately finds `R2e1`** (learned during Step 14 of the request). **No ARP broadcast is needed.**
4. **Encapsulate & Transmit (Hop 1):** Host C sends the frame to R2:
   * **L2 Header:** `Src MAC: CCCC` | `Dst MAC: R2e1`
   * **L3 Header:** `Src IP: 10.3.3.30` | `Dst IP: 10.1.1.10`

### Phase 2: Router 2 (R2) Forwards the Response to R1
5. **Receive & Strip Layer 2:** R2 accepts the frame on `eth1` (`Dst MAC: R2e1`) and strips off the Layer 2 header.
6. **Routing Table Lookup:** R2 inspects `Dst IP: 10.1.1.10` and checks its Routing Table:
   * Match found: `10.1.1.0/24` $\rightarrow$ **Next-Hop IP = `10.2.2.1`** via **Exit Interface = `eth0`**.
7. **R2 ARP Table Lookup (Cache Hit!):** R2 checks its ARP table for `10.2.2.1` and **immediately finds `R1e1`** (learned during Step 9 of the request). **No ARP broadcast is needed.**
8. **Re-encapsulate & Transmit (Hop 2):** R2 attaches a new Layer 2 header and forwards the frame out `eth0`:
   * **L2 Header:** `Src MAC: R2e0` | `Dst MAC: R1e1`
   * **L3 Header:** `Src IP: 10.3.3.30` | `Dst IP: 10.1.1.10`

### Phase 3: Router 1 (R1) Forwards the Response to Host A
9. **Receive & Strip Layer 2:** R1 accepts the frame on `eth1` (`Dst MAC: R1e1`) and strips off the Layer 2 header.
10. **Routing Table Lookup:** R1 inspects `Dst IP: 10.1.1.10` and checks its Routing Table:
    * Match found: `10.1.1.0/24` $\rightarrow$ **Directly Connected** via **Exit Interface = `eth0`**.
11. **R1 ARP Table Lookup (Cache Hit!):** R1 checks its ARP table for `10.1.1.10` and **immediately finds `AAAA`** (learned during Step 4 of the request). **No ARP broadcast is needed.**
12. **Re-encapsulate & Transmit (Hop 3):** R1 attaches a new Layer 2 header and forwards the frame out `eth0`:
    * **L2 Header:** `Src MAC: R1e0` | `Dst MAC: AAAA`
    * **L3 Header:** `Src IP: 10.3.3.30` | `Dst IP: 10.1.1.10`
13. **Final Delivery at Host A:** Host A receives the frame (`Dst MAC: AAAA`), strips L2, verifies `Dst IP: 10.1.1.10`, and delivers the response data up the OSI stack.

---

## 5. Events Between R2 and R1 (Scaling Across Multiple Routers)

* **Key Principle:** The exact sequence of events that happens between **R1 and R2** (and **R2 and R1**) **repeats identically for any number of routers in the path** — whether there are 2 routers, 5 routers, or 20 routers between two hosts.
* Every single router along the path executes the exact same loop:
  1. Receive frame and strip incoming Layer 2 header.
  2. Look up the Destination IP in its **Routing Table** to find the Next-Hop IP and Exit Interface.
  3. Look up that Next-Hop IP in its **ARP Table** (sending an ARP Request/Reply if not already cached) to get the Next-Hop MAC address.
  4. Apply a new Layer 2 header (`Src MAC = Exit Interface`, `Dst MAC = Next-Hop MAC`) and forward the packet out the exit interface.

---

## 6. Deep Dive: What Each Router Looks Up, What Layers It Adds, and How It Adds Them

Whenever a router forwards a packet from one network to another, it acts as a **Layer 3 boundary** that **destroys and recreates Layer 2** while **preserving Layer 3 end-to-end**.

### A. What Each Router Looks Up (2 Tables)
1. **First Lookup — The Routing Table (Layer 3 Decision):**
   * **Input:** The packet's **Destination IP Address** (from the Layer 3 header).
   * **Purpose:** Answers *"Which direction/network does this packet need to go?"*
   * **Output:** Two critical pieces of information:
     * **Exit Interface** (the physical port the packet will leave out of).
     * **Next-Hop IP Address** (the IP of the next router in the chain, or the final host's IP if the destination network is directly connected).
2. **Second Lookup — The ARP Table (Layer 2 Resolution):**
   * **Input:** The **Next-Hop IP Address** returned by the Routing Table (or the final Destination IP if directly connected).
   * **Purpose:** Answers *"What is the physical MAC address of that next device on the local link?"*
   * **Output:** The **Destination MAC Address** needed to build the new Layer 2 frame. *(If the MAC is not in the ARP table yet, the router pauses forwarding, broadcasts an ARP Request out the exit interface, receives the ARP Reply, caches the mapping, and then proceeds.)*

### B. What Layers Each Router Removes & Adds, and How It Adds Them
* **What It Does NOT Touch (Layer 3 — IP Header & Payload):**
  * The router **never changes** the **Source IP** or **Destination IP** (assuming standard routing without NAT). Layer 3 provides **end-to-end logical addressing** from the original sender (`Host A`) to the final receiver (`Host C`).
  * *(Note: Practically, a router also decrements the IP header's TTL field by 1 to prevent infinite routing loops, while keeping Source and Destination IPs intact.)*
* **What It Removes and Adds (Layer 2 — Ethernet Header):**
  * **Step 1 (De-encapsulation):** When a frame arrives on an incoming interface, the router **strips off and discards the incoming Layer 2 header**. That old Layer 2 header only existed to get the packet across the previous hop.
  * **Step 2 (Re-encapsulation — How It Adds the New Layer 2 Layer):** The router constructs a **completely new Layer 2 header** around the existing Layer 3 packet before transmitting it onto the next wire:
    * **New Source MAC Address:** Set to the **MAC address of the router's own Exit Interface** (selected by the Routing Table).
    * **New Destination MAC Address:** Set to the **MAC address of the Next-Hop Router** (or final Host) retrieved from the **ARP Table**.

```text
Incoming Frame on R1 (from Host A):
+------------------------+-----------------------+----------------------------+
| L2: Src=AAAA Dst=R1e0  | L3: Src=.1.10 Dst=.3.30| Data (L4 - L7 Payload)    |
+------------------------+-----------------------+----------------------------+
          |
          v  [ R1 strips old L2, checks Routing Table + ARP Table, adds new L2 ]
          |
Outgoing Frame from R1 (to R2):
+------------------------+-----------------------+----------------------------+
| L2: Src=R1e1 Dst=R2e0  | L3: Src=.1.10 Dst=.3.30| Data (L4 - L7 Payload)    |
+------------------------+-----------------------+----------------------------+
   ^--- Rewritten every hop!   ^--- Stays identical end-to-end!
```

---

## 7. What Is the Internet? (Components of the Internet)

Understanding how **R1 and R2** forward packets reveals what the **Internet** actually is at a foundational networking level:

* **Definition:** The **Internet** is simply **a massive, interconnected web of Networks joined together by Routers** ("a network of networks").
* **How the Components Fit Together:**
  1. **Hosts (Clients & Servers):** End devices (laptops, phones, web servers, cloud instances) that originate requests and serve responses using IP addresses.
  2. **Local Networks & Switches (Layer 2):** Switches connect local hosts together inside individual networks (homes, offices, data centers, ISPs) using MAC addresses.
  3. **Routers (Layer 3):** Sit at the boundaries between different networks. Whether it is your home router, an ISP core router, or a backbone router connecting continents, **every router on the Internet follows the exact same rules** demonstrated by R1 and R2:
     * Receive packet $\rightarrow$ Strip Layer 2 $\rightarrow$ Check Routing Table $\rightarrow$ Check ARP Table $\rightarrow$ Add new Layer 2 $\rightarrow$ Forward to the next hop.
  4. **Physical Links / Media (Layer 1):** Copper cables, fiber-optic undersea cables, and wireless/radio links that carry the electrical, light, or RF signals between switches, routers, and hosts.

> **Key Takeaway:** There is no "magic" inside the Internet cloud. The Internet is just **more networks and more routers** repeating the exact same **Routing Table + ARP Table** forwarding process hop-by-hop until a packet reaches its destination network.