# Everything Routers Do — Networking Fundamentals Study Notes

> **Topic:** Router Core Functions, Network Nodes, RFCs, Routing Tables, Route Population Methods (Directly Connected, Static, Dynamic), and Dynamic Routing Protocols  
> **Format:** Structured Study Guide & Topology Reference

---

## 1. Core Fundamentals: What Routers Do

From your handwritten notes:
* **Network Connectivity & Addressing:** Routers are connected to multiple networks (subnets), and each router interface has its own **IP Address** (Layer 3) and **MAC Address** (Layer 2).
* **Packet Forwarding:** Unlike regular hosts (PCs/servers) which drop packets not addressed to them, **routers forward packets that are not destined to themselves** toward their destination network.
* **Unknown Destinations:** When a router receives a packet with an **unknown destination IP** (i.e., no matching route exists in its Routing Table, and no Default Route `0.0.0.0/0` is configured), **the packet is dropped** (and typically an ICMP *Destination Unreachable* message is sent back to the source).

---

## 2. Network Nodes: Definition, Types & Key Concepts

### What is a Node?
In networking, a **Node** is **any active device or redistribution point connected to a network** that has a physical or virtual network interface and can **create, receive, process, store, or forward data** across a communication channel.

> **Graph View (Nodes vs. Links):**
> A network topology is like a map or graph:
> - **Nodes (Vertices):** The devices themselves (PCs, servers, switches, routers, firewalls).
> - **Links (Edges):** The physical or wireless connections between nodes (Ethernet cables, fiber-optic links, Wi-Fi, WAN circuits).

### Main Types of Network Nodes

| Node Category | Subtypes / Examples | Layer | Primary Role |
| :--- | :--- | :--- | :--- |
| **End Nodes (Hosts / Endpoints / End Systems)** | PCs, laptops, servers, smartphones, printers, IP cameras, IoT devices | Layers 3–7 | Originate or consume network traffic. They **drop** packets not addressed to them (unless acting as a router/gateway). |
| **Intermediary (Transit) Nodes** | **Repeaters / Hubs** *(L1)*, **Switches / Bridges / Wireless APs** *(L2)*, **Routers / L3 Switches** *(L3)*, **Firewalls / Load Balancers** *(L3–L7)* | Layers 1–7 | Forward, switch, route, filter, or regenerate data **between** end nodes so traffic can cross LANs and the Internet. |

*(Note: Passive components like patch cables or wall jacks are **media/links**, not nodes, because they do not address, process, or actively regenerate traffic. Similarly, in OSI/IS-IS terminology, end nodes are called **End Systems (ES)** and routers are called **Intermediate Systems (IS)**.)*

### Core Concepts Related to Nodes in a Network

1. **Node Addressing & Identity:**
   - **MAC Address (Layer 2 Identity):** Uniquely identifies a node's Network Interface Card (NIC) on its **local network segment (LAN)**.
   - **IP Address (Layer 3 Identity):** Logically identifies a node across **multiple networks**. A router node has *multiple* interfaces, each with its own IP and MAC address.
   - **Hostname / DNS Name:** Human-readable label mapped to a node's IP address (e.g., `router1.corp.local`).
2. **Source, Transit, and Destination Nodes:**
   - **Source Node:** The originating host that creates the packet and puts its own IP in `Src IP`.
   - **Transit Nodes (Intermediate Nodes):** Switches and routers in the middle that pass the frame/packet along.
   - **Destination Node:** The final target host listed in `Dst IP`.
3. **Adjacent Nodes (Neighbors) & Next-Hop Node:**
   - **Neighbor / Adjacent Node:** Another node sharing a **direct link or local subnet** (e.g., two routers on `10.0.12.0/30` forming an OSPF/EIGRP neighbor adjacency, or a host and its default gateway).
   - **Next-Hop Node:** The immediate next **Layer 3 node (router)** to which a packet is forwarded on its way toward a remote destination network.
4. **Hops, Hop Count & TTL:**
   - **Hop:** The traversal of a packet from one **Layer 3 node (router)** to the next (Layer 2 switches are generally transparent to Layer 3 hop counts).
   - **Hop Count:** Total number of router nodes a packet passes through (used directly as the metric in RIP, max `15`).
   - **TTL (Time To Live):** A counter in the IP header decremented by `1` at every router node; if `TTL` hits `0`, the node drops the packet to stop infinite routing loops.
5. **Multi-Homed Nodes:**
   - A node equipped with **two or more active network connections/interfaces** (for example, every router connecting multiple subnets, a server connected to two switches, or an enterprise connected to two different ISPs).
6. **Redundancy & Single Point of Failure (SPOF):**
   - If all traffic depends on a single transit node (e.g., one router or core switch) and that node goes down, connectivity fails (**Single Point of Failure**). Adding backup nodes and links lets dynamic routing protocols (OSPF, EIGRP, BGP) route around a failed node automatically.
7. **Autonomous Systems (AS) & Edge vs. Core Nodes:**
   - **Edge / Gateway Nodes:** Nodes sitting at the boundary between a local network and an external network/ISP (e.g., Default Gateway or BGP edge router).
   - **Core / Backbone Nodes:** High-speed transit routers inside a network or ISP that forward traffic rapidly between edge nodes.
   - **Leaf vs. Spine Nodes:** Common modern data-center topology terms where **Leaf nodes** connect end hosts/servers and **Spine nodes** interconnect the leaves.

```text
  [End Node: Host A] ----(Link)---- [Transit Node: SW1] ----(Link)---- [Transit Node: R1] ----(Link)---- [Transit Node: R2] ----(Link)---- [End Node: Host B]
     (Source Node)                    (L2 Switch Node)                  (Default Gateway /                   (Next-Hop Node /                  (Destination Node)
                                                                         Multi-Homed L3 Node)                 Adjacent Neighbor to R1)
```

---

## 3. RFC (Request for Comments)

### What is an RFC?
An **RFC (Request for Comments)** is a formal, peer-reviewed publication from the **IETF (Internet Engineering Task Force)** and the **Internet Society (ISOC)** that defines the technical standards, protocols, architectures, procedures, and best practices that make the Internet work.

* **Origin:** Started in 1969 by Steve Crocker during the ARPANET project as informal memos asking fellow engineers for feedback ("Request for Comments").
* **Immutable Archive:** Once an RFC is published and assigned a number (e.g., **RFC 791** for IPv4, **RFC 826** for ARP, **RFC 2328** for OSPFv2), it is **never edited or overwritten**. If a standard evolves, a *new* RFC is published that updates or obsoletes the old one.

### Why are RFCs Important? (The Use & Importance of RFCs)
1. **Global Interoperability:** Ensures that hardware and software from completely different vendors (Cisco, Juniper, Linux, Windows, Apple) can communicate seamlessly because they follow the exact same rules.
2. **Standardization of Protocols:** Defines exact packet header structures, bit layouts, port numbers, state machines, and error-handling behaviors.
3. **Open & Transparent Engineering:** Anyone can read RFCs for free, implement protocols without proprietary lock-in, and contribute to future Internet standards.
4. **Security & Auditing Reference:** For security researchers and pentesters, RFCs are the ultimate source of truth—understanding how a protocol *must* behave makes it easier to spot non-compliant implementations, edge-case bugs, and protocol-level vulnerabilities.

### Components of an RFC Document
Every standard RFC follows a strict, recognizable structure:

| Component | Description |
| :--- | :--- |
| **1. Header Block** | Appears at the top-left and top-right of page 1. Lists the **RFC Number**, **Category/Status** (Standards Track, Informational, Experimental, Best Current Practice), **Author(s)**, **Affiliation**, **Date**, and references (`Updates:` or `Obsoletes:`). |
| **2. Title & Abstract** | A concise title followed by a short summary explaining what the document specifies or addresses. |
| **3. Status of This Memo & Copyright** | Boilerplate text clarifying whether the document is an official Internet Standard or an informational memo, along with IETF Trust copyright terms. |
| **4. Table of Contents** | Numbered outline of all sections and subsections for quick navigation. |
| **5. Introduction & Terminology** | Explains the problem statement, background context, and mandatory requirement keywords defined by **RFC 2119** (`MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, `MAY`). |
| **6. Protocol Specification & Packet Diagrams** | The technical core: ASCII bit-level header diagrams (32-bit rows), state diagrams, algorithms, timers, and message formats. |
| **7. Security Considerations** | A mandatory section analyzing potential threats, attack vectors, spoofing risks, and recommended mitigations for the protocol. |
| **8. IANA Considerations** | Specifies any numbers, protocol IDs, port numbers, or flags that the **Internet Assigned Numbers Authority (IANA)** must register or manage. |
| **9. References** | Divided into **Normative References** (essential to implement the standard) and **Informative References** (helpful background reading). |

---

## 4. Routing & The Routing Table

### What is Routing?
**Routing** is the Layer 3 (Network Layer) process of selecting the optimal path across one or more interconnected networks to move IP packets from a source host to a destination host.

When a packet arrives at a router:
1. The router strips off the incoming Layer 2 frame header/trailer.
2. It inspects the **Destination IP Address** inside the Layer 3 IP header.
3. It searches its **Routing Table** using **Longest Prefix Match** (choosing the route with the most specific subnet mask, e.g., `/26` beats `/24`, which beats `/0`).
4. If a match is found, it determines the **Outgoing Interface** and **Next-Hop IP**, encapsulates the packet in a new Layer 2 frame (using ARP if needed to resolve the next-hop MAC address), and forwards it.
5. If **no match is found**, the router **drops the packet**.

### What is a Routing Table?
A **Routing Table** is a structured data table stored in a router's RAM that acts as a **map of all the networks the router knows how to reach**, along with the exit interfaces and next-hop gateways required to get there.

### Components of a Routing Table
Every entry (route) in a Routing Table typically contains the following fields:

| Component | Code / Symbol | Explanation |
| :--- | :--- | :--- |
| **Route Source (Protocol Code)** | `C`, `L`, `S`, `O`, `R`, `B`, `D` | Tells *how* the router learned the route:<br>• `C` = **Connected** (directly attached network)<br>• `L` = **Local** (`/32` IP of the router's own interface)<br>• `S` = **Static** (manually configured by an admin)<br>• `O` = **OSPF** / `R` = **RIP** / `D` = **EIGRP** / `B` = **BGP** (Dynamic) |
| **Destination Network & Prefix** | e.g., `192.168.20.0/24` | The target network ID and subnet mask (CIDR prefix) that this route leads to. |
| **Administrative Distance (AD)** | e.g., `[1/0]` or `[110/20]` *(first number)* | A trustworthiness score (`0` to `255`) used when the router learns the *same* network from *two different sources*. **Lower AD = More Trusted** (`Connected = 0`, `Static = 1`, `EIGRP = 90`, `OSPF = 110`, `RIP = 120`). |
| **Metric** | e.g., `[110/20]` *(second number)* | The "cost" of the path within a single routing protocol (e.g., hop count in RIP, bandwidth-based cost in OSPF). **Lower Metric = Better Path**. |
| **Next-Hop IP Address** | e.g., `via 10.0.12.2` | The IP address of the next router along the path toward the destination network. (For directly connected networks, there is no next hop). |
| **Outgoing (Exit) Interface** | e.g., `GigabitEthernet0/1` | The physical or virtual port on *this* router out of which the packet must be transmitted. |
| **Route Age / Timer** | e.g., `00:14:32` | How long the route has been active in the table (primarily used by dynamic routing protocols to age out dead links). |

---

## 5. How the Routing Table & MAC/ARP Table Are Populated (3 Methods)

> **Important Layer 3 vs. Layer 2 Distinction:**
> * **IP Routes (Layer 3)** are populated into the **Routing Table** via the **3 methods** below (Directly Connected, Static, Dynamic).
> * **MAC Addresses (Layer 2)** are populated into the router's **ARP Cache / ARP Table** dynamically via **ARP (Address Resolution Protocol)** whenever the router needs to encapsulate a packet onto an Ethernet link to reach a local host or a Next-Hop router.

---

### Method 1: Directly Connected Routes (`C` and `L`)

#### Detailed Explanation
* **How it works:** Whenever a network administrator assigns an **IP Address** and **Subnet Mask** to a router's interface and enables it (`no shutdown` / Link Up), the router automatically calculates the network address (by performing a bitwise `AND` between the IP and Subnet Mask) and injects two entries directly into its Routing Table:
  1. **`C` (Connected Route):** The entire subnet attached to that interface (e.g., `192.168.10.0/24`).
  2. **`L` (Local Route):** The exact `/32` host IP address assigned to the router's own interface (e.g., `192.168.10.1/32`).
* **Administrative Distance:** `0` (most trusted source possible).
* **How IP & MAC are populated during traffic flow:**
  * **IP Routing Table:** Populated automatically the moment interfaces come up.
  * **MAC (ARP Table):** When Host A (`192.168.10.10`) sends a packet to Host B (`192.168.20.10`), R1 looks up `192.168.20.0/24` in its Routing Table, sees it is **Directly Connected** out `Gi0/1`, broadcasts an **ARP Request** on `Gi0/1` asking *"Who has 192.168.20.10?"*, and records Host B's MAC address in R1's ARP table upon receiving the **ARP Reply**.

#### Topology & Process Diagram: Directly Connected

```text
   Subnet A: 192.168.10.0/24                        Subnet B: 192.168.20.0/24
  +-------------------------+                      +-------------------------+
  | Host A                  |                      | Host B                  |
  | IP:  192.168.10.10/24   |                      | IP:  192.168.20.10/24   |
  | MAC: AA:AA:AA:AA:AA:AA  |                      | MAC: BB:BB:BB:BB:BB:BB  |
  | GW:  192.168.10.1       |                      | GW:  192.168.20.1       |
  +------------+------------+                      +------------+------------+
               |                                                |
               |   +----------------------------------------+   |
               +-->| Gi0/0            [R1]            Gi0/1 |<--+
                   | IP:  192.168.10.1   IP:  192.168.20.1  |
                   | MAC: 11:11:11:11    MAC: 22:22:22:22   |
                   +----------------------------------------+
```

**Resulting R1 Routing Table (Populated Automatically):**
```text
Code | Network            | Next-Hop          | Exit Interface
-----+--------------------+-------------------+---------------
C    | 192.168.10.0/24    | Directly Connected| Gi0/0
L    | 192.168.10.1/32    | Directly Connected| Gi0/0
C    | 192.168.20.0/24    | Directly Connected| Gi0/1
L    | 192.168.20.1/32    | Directly Connected| Gi0/1
```

**Resulting R1 ARP Table (Populated via ARP on Connected Links):**
```text
IP Address      | MAC Address       | Interface
----------------+-------------------+----------
192.168.10.10   | AA:AA:AA:AA:AA:AA | Gi0/0
192.168.20.10   | BB:BB:BB:BB:BB:BB | Gi0/1
```

---

### Method 2: Static Routes (`S`)

#### Detailed Explanation
* **How it works:** A router only knows its *directly connected* networks out of the box. For remote networks separated by one or more other routers, a **Network Administrator manually types a static route command** telling the router: *"To reach Destination Network X, send the packet to Next-Hop Router IP Y (or out Exit Interface Z)."*
* **Administrative Distance:** `1` (second most trusted, right after Directly Connected).
* **Pros & Cons:**
  * *Pros:* Zero CPU/bandwidth overhead (no routing protocol packets sent), predictable paths, very secure (no protocol spoofing), ideal for small networks or **Default Routes (`0.0.0.0/0`)** pointing to an ISP.
  * *Cons:* Does **not scale** well in large networks; if a link fails, the administrator must manually reconfigure routes (unless Floating Static Routes are used).
* **How IP & MAC are populated during traffic flow:**
  * **IP Routing Table:** R1 learns `192.168.20.0/24 via 10.0.12.2` from the admin's manual configuration.
  * **MAC (ARP Table):** When R1 needs to forward a packet to Next-Hop `10.0.12.2`, R1 sends an **ARP Request** out `Gi0/1` for `10.0.12.2` and caches **R2's MAC address** (`33:33:33:33`). R1 never needs to know Host B's MAC address—only the next hop's MAC!

#### Topology & Process Diagram: Static Routes

```text
  Subnet A: 192.168.10.0/24        Transit Link: 10.0.12.0/30        Subnet B: 192.168.20.0/24
  +--------------------+          +--------------------------+          +--------------------+
  | Host A             |          |           [R1]           |          | Host B             |
  | 192.168.10.10/24   |--------->| Gi0/0              Gi0/1 |          | 192.168.20.10/24   |
  | MAC: AA:AA...      |          | 192.168.10.1   10.0.12.1 |          | MAC: BB:BB...      |
  +--------------------+          |                MAC:22:22 |          +--------------------+
                                  +-------------+------------+                     ^
                                                |                                  |
                                    Admin configures Static Route                  |
                                    ip route 192.168.20.0/24 -> 10.0.12.2          |
                                                |                                  |
                                                v                                  |
                                  +-------------+------------+                     |
                                  | Gi0/0              Gi0/1 |---------------------+
                                  | 10.0.12.2   192.168.20.1 |
                                  | MAC:33:33                |
                                  |           [R2]           |
                                  +--------------------------+
                                    Admin configures Return Route
                                    ip route 192.168.10.0/24 -> 10.0.12.1
```

**Resulting R1 Routing Table:**
```text
Code | Network            | AD/Metric | Next-Hop           | Exit Interface
-----+--------------------+-----------+--------------------+---------------
C    | 192.168.10.0/24    | [0/0]     | Directly Connected | Gi0/0
C    | 10.0.12.0/30       | [0/0]     | Directly Connected | Gi0/1
S    | 192.168.20.0/24    | [1/0]     | via 10.0.12.2      | Gi0/1
```

**Resulting R1 ARP Table (Resolves Next-Hop MAC):**
```text
IP Address      | MAC Address       | Interface
----------------+-------------------+----------
192.168.10.10   | AA:AA:AA:AA:AA:AA | Gi0/0
10.0.12.2       | 33:33:33:33:33:33 | Gi0/1   <-- R2's Gi0/0 MAC learned via ARP
```

---

### Method 3: Dynamic Routes (`O`, `R`, `D`, `B`)

#### Detailed Explanation
* **How it works:** Routers run **Dynamic Routing Protocols** (such as OSPF, EIGRP, RIP, or BGP) to automatically discover neighboring routers, advertise their directly connected networks to each other, calculate the best paths using mathematical algorithms, and dynamically update their Routing Tables when links go up or down.
* **Step-by-Step Dynamic Population Process:**
  1. **Neighbor Discovery:** Routers send periodic **Hello** (or update) packets out enabled interfaces to discover adjacent routers running the same protocol.
  2. **Route / Topology Exchange:** Neighbors exchange routing updates or Link-State Advertisements (LSAs) describing the networks they know about and their link costs.
  3. **Best-Path Calculation:** Each router runs its protocol's algorithm (e.g., Dijkstra's SPF for OSPF, Bellman-Ford for RIP, DUAL for EIGRP) and installs the winning routes into its **Routing Table**.
  4. **ARP Resolution for Forwarding:** Once the dynamic route (`via Next-Hop IP`) is in the Routing Table, the router uses **ARP** on the exit interface to resolve the Next-Hop router's MAC address before forwarding packets.
  5. **Automatic Convergence:** If a link breaks, routers automatically detect the missed Hellos, recompute backup paths, and update their Routing Tables without human intervention.

#### Topology & Process Diagram: Dynamic Routes

```text
                        [R2] (Transit Router)
                       /    \
         10.0.12.0/30 /      \ 10.0.23.0/30
                     /        \
   +----------------+          +----------------+
   |      [R1]      |          |      [R3]      |
   | Gi0/0    Gi0/1 |          | Gi0/0    Gi0/1 |
   +---+--------+---+          +---+--------+---+
       |        |                  |        |
       |        +------------------+        |
       |            10.0.13.0/30            |
       v          (Backup Link)             v
  192.168.10.0/24                      192.168.30.0/24
    (Host A)                             (Host C)

  Dynamic Process:
  1. R1, R2, R3 exchange "Hello" packets & form Neighbor Adjacencies.
  2. R3 advertises "192.168.30.0/24" to R2 and R1.
  3. R1 calculates best path (e.g., via R1->R3 direct link, cost 10 vs via R2, cost 20).
  4. R1 installs route: O 192.168.30.0/24 [110/11] via 10.0.13.2, Gi0/1.
  5. If link R1-R3 fails, R1 automatically reconverges to use R1 -> R2 -> R3!
```

**Resulting R1 Routing Table (Populated Dynamically via OSPF):**
```text
Code | Network            | AD/Metric | Next-Hop           | Exit Interface
-----+--------------------+-----------+--------------------+---------------
C    | 192.168.10.0/24    | [0/0]     | Directly Connected | Gi0/0
C    | 10.0.12.0/30       | [0/0]     | Directly Connected | Gi0/2
C    | 10.0.13.0/30       | [0/0]     | Directly Connected | Gi0/1
O    | 192.168.30.0/24    | [110/11]  | via 10.0.13.2      | Gi0/1
O    | 10.0.23.0/30       | [110/20]  | via 10.0.12.2      | Gi0/2
```

---

## 6. Dynamic Routing Protocols Explained in Detail

Dynamic routing protocols are classified along two main dimensions:
1. **Scope (IGP vs. EGP):**
   * **IGP (Interior Gateway Protocol):** Routes traffic *within* a single organization or Autonomous System (AS) — e.g., **RIP, OSPF, EIGRP, IS-IS**.
   * **EGP (Exterior Gateway Protocol):** Routes traffic *between* different Autonomous Systems across the global Internet — e.g., **BGP**.
2. **Routing Logic (Distance-Vector vs. Link-State vs. Path-Vector):**
   * **Distance-Vector ("Routing by Rumor"):** Routers only know the direction (exit interface/next-hop) and distance (metric) advertised by their immediate neighbors; they do not see the full network map.
   * **Link-State ("Full Topology Map"):** Every router builds an identical, complete map of the entire network topology and independently calculates the shortest path to every destination.
   * **Path-Vector:** Tracks the full path of Autonomous Systems (`AS_PATH`) to prevent loops across the Internet and enforce routing policies.

---

### Protocol 1: RIP (Routing Information Protocol)
* **Type:** Distance-Vector IGP
* **Administrative Distance (AD):** `120`
* **Metric:** **Hop Count** (number of routers a packet must pass through).
  * Maximum valid hop count = **15 hops**.
  * **16 hops = Unreachable (Infinity)**.
* **How it Works:**
  * Every **30 seconds**, RIP routers broadcast/multicast their *entire* routing table to directly connected neighbors (RIPv1 uses Broadcast `255.255.255.255`; RIPv2 uses Multicast `224.0.0.9` and supports CIDR/VLSM).
  * Neighbors increment the hop count by `+1` and keep the route with the lowest hop count.
* **Loop Prevention Mechanisms:**
  * **Split Horizon:** Never advertise a route back out the same interface from which it was learned.
  * **Route Poisoning & Poison Reverse:** Advertise a failed route with a metric of `16` (infinite) so neighbors immediately purge it.
  * **Hold-Down Timers:** Ignore updates about a downed route for a set period to let the network stabilize.
* **Limitations:** Slow convergence, high bandwidth overhead (sending full tables every 30s), 15-hop limit restricts it to very small networks, and ignores link speed (a slow 10 Mbps link with 1 hop beats a fast 10 Gbps path with 2 hops).

---

### Protocol 2: OSPF (Open Shortest Path First)
* **Type:** Link-State IGP (Open Standard — RFC 2328 for OSPFv2 / IPv4, RFC 5340 for OSPFv3 / IPv6)
* **Administrative Distance (AD):** `110`
* **Metric:** **Cost** (calculated based on interface bandwidth: $\text{Cost} = \frac{\text{Reference Bandwidth}}{\text{Interface Bandwidth}}$, where default Reference Bandwidth is $100\text{ Mbps}$). Lower cost = faster, preferred path.
* **How it Works (3-Table Architecture & Dijkstra's SPF):**
  1. **Neighbor Discovery (Hello Protocol):** Routers send `Hello` packets via Multicast (`224.0.0.5`) every 10 seconds to form neighbor adjacencies (verifying Area ID, Subnet Mask, Hello/Dead timers, and Authentication) and store them in the **Neighbor Table**.
  2. **LSA Flooding:** Routers generate **Link-State Advertisements (LSAs)** describing their local interfaces and connected neighbors, flooding them across the area so every router builds an identical **Link-State Database (LSDB)** (Topology Table).
  3. **SPF Calculation:** Each router independently runs **Dijkstra's Shortest Path First (SPF)** algorithm on its LSDB to compute the loop-free shortest path tree to every network and installs the best routes into the **Routing Table**.
* **Key Features:**
  * **Hierarchical Areas:** Uses **Area 0 (Backbone Area)** at the center, with other areas connecting to Area 0 to isolate topology changes and reduce CPU/memory load.
  * **Triggered Updates:** Sends updates immediately when a topology change occurs (fast convergence) rather than periodically flooding full tables.

---

### Protocol 3: EIGRP (Enhanced Interior Gateway Routing Protocol)
* **Type:** Advanced Distance-Vector (often called "Hybrid") IGP (Originally Cisco proprietary, later published as open informational RFC 7868)
* **Administrative Distance (AD):** `90` (Internal routes) / `170` (External redistributed routes)
* **Metric:** **Composite Metric** calculated using **Bandwidth** (slowest link along the path) and **Delay** (cumulative delay across all links along the path), with optional Reliability and Load factors.
* **How it Works (DUAL Algorithm):**
  1. **Neighbor Table:** Discovers neighbors using `Hello` packets sent to Multicast `224.0.0.10` using **RTP (Reliable Transport Protocol)**.
  2. **Topology Table:** Stores all routes learned from neighbors, identifying:
     * **Successor:** The primary best loop-free route (installed in the Routing Table).
     * **Feasible Successor:** A pre-calculated, guaranteed loop-free **backup route** kept ready in the Topology Table.
  3. **DUAL (Diffusing Update Algorithm):** If the primary route (Successor) fails, EIGRP **immediately switches to the Feasible Successor in milliseconds** without needing to recompute or query the whole network.
* **Key Features:** Extremely fast convergence, supports unequal-cost load balancing (`variance`), and sends partial/bounded updates only to affected routers.

---

### Protocol 4: IS-IS (Intermediate System to Intermediate System)
* **Type:** Link-State IGP (RFC 1142 / ISO 10589)
* **Administrative Distance (AD):** `115`
* **Metric:** **Default Metric / Cost** (typically `10` per link by default, adjustable for wide metrics).
* **How it Works:**
  * Very similar in operation to OSPF: exchanges `Hello` PDUs, floods **Link-State PDUs (LSPs)** to build an identical **LSDB**, and runs **Dijkstra's SPF algorithm**.
  * Uses a two-level hierarchy (**Level 1** intra-area routing and **Level 2** inter-area backbone routing).
* **Key Difference from OSPF:** IS-IS operates directly over **Layer 2 (Data Link Layer)** rather than IP, meaning it naturally carries both IPv4 and IPv6 routing information in a single protocol instance without needing separate versions. Widely used in large ISP and Telecom core networks.

---

### Protocol 5: BGP (Border Gateway Protocol — BGP-4)
* **Type:** Path-Vector EGP (RFC 4271) — **The Routing Protocol of the Internet**
* **Administrative Distance (AD):** `20` for **eBGP** (External BGP between different Autonomous Systems) / `200` for **iBGP** (Internal BGP within the same AS).
* **Metric:** **Path Attributes** (policy-driven selection rather than a simple bandwidth/hop metric), evaluated in strict order: `Weight` $\rightarrow$ `Local Preference` $\rightarrow$ `Locally Originated` $\rightarrow$ `AS_PATH Length` $\rightarrow$ `Origin` $\rightarrow$ `MED` $\rightarrow$ `eBGP over iBGP`.
* **How it Works:**
  * Unlike IGPs that auto-discover neighbors via multicast, BGP peers are **manually configured** and establish a reliable **TCP session on Port 179**.
  * Exchanges Network Layer Reachability Information (NLRI / IP prefixes) tagged with **Path Attributes**—most notably **`AS_PATH`**, which lists every Autonomous System Number (ASN) the advertisement has traversed.
  * **Loop Prevention:** If a BGP router sees its **own ASN** inside an incoming `AS_PATH`, it immediately rejects the route to prevent inter-domain routing loops.
* **Key Features:** Built to handle massive scale (over 950,000+ IPv4 routes in the global Internet routing table) and gives ISPs and enterprises granular business/security policy control over how traffic enters and leaves their network.

---

## 7. Quick Comparison Summary of Dynamic Routing Protocols

| Protocol | Category | Algorithm | Default AD | Metric | Transport / Multicast | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **RIPv2** | Distance-Vector (IGP) | Bellman-Ford | `120` | Hop Count (Max 15) | UDP Port 520 (`224.0.0.9`) | Legacy / tiny lab networks |
| **OSPFv2** | Link-State (IGP) | Dijkstra's SPF | `110` | Cost (Bandwidth-based) | IP Protocol 89 (`224.0.0.5` / `.6`) | Enterprise & multi-vendor networks |
| **EIGRP** | Advanced Distance-Vector (IGP) | DUAL | `90` (Int) / `170` (Ext) | Composite (Bandwidth + Delay) | IP Protocol 88 (`224.0.0.10`) | Enterprise networks (Cisco-centric) |
| **IS-IS** | Link-State (IGP) | Dijkstra's SPF | `115` | Cost (Configurable) | Direct Layer 2 (CLNS) | Large Service Providers / ISPs |
| **BGP-4** | Path-Vector (EGP) | Best-Path Selection | `20` (eBGP) / `200` (iBGP) | Path Attributes (`AS_PATH`, Local Pref) | **TCP Port 179** (Unicast) | The Internet / Multi-homed Enterprises |