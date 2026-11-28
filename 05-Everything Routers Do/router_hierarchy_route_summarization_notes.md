# Router Hierarchy & Route Summarization

- **Source:** Handwritten study page (`d03e9acd-82cc-4f15-a65d-9d00fbb02bea.jpg`) & Networking Fundamentals concepts

---

## 1. Why Routers Typically Connect in a Hierarchy

Routers generally connect in a structured **hierarchical topology** (such as Core $\rightarrow$ Distribution/Regional $\rightarrow$ Edge/Access routers) rather than a flat or completely ad-hoc mesh of every router connecting to every other router.

### Top-Level Benefits Listed in Notes
1. **Easier to scale** and **more consistent connectivity**
2. **Hierarchy allows for Route Summarization**

---

## 2. Explaining "Easier to Scale" & "More Consistent Connectivity" Using Topology

Consider two topologies connecting multiple regional/departmental networks:

### Poorly Scaled / Non-Hierarchical Topology (Mesh-like Growth)
```text
        [R1 - Site A] ---- [R2 - Site B]
          |      \          /      |
          |       \        /       |
          |        [R3 - Site C]   |
          |            |           |
        [R4 - Site D] -+---- [R5 - Site E]
```
- Adding a new site (`R6`) requires figuring out links and peering relationships with many existing routers, increasing configuration complexity and error risk as the network grows.
- Inconsistent link paths and ad-hoc connections make connectivity behavior harder to predict and maintain.

### Hierarchical Topology (Core / Regional / Edge)
```text
                  [ Core Router (R1) ]
                   /        |         \
                  /         |          \
       [Region A (R2)] [Region B (R3)] [Region C (R4)]
         /        \          |            /        \
    [LAN A1]   [LAN A2]   [LAN B1]    [LAN C1]   [LAN C2]
```

#### How This Makes It **Easier to Scale**
- When a new branch or subnet is added (for example, `LAN C3` under `Region C`), you attach it cleanly to the appropriate tier (`R4`) without redesigning the entire network or wiring every router to every other router.
- Each layer has a clear role: **Edge/Regional routers** connect local subnets upward, and **Core/Upper routers** move traffic between major regions efficiently.

#### How This Provides **More Consistent Connectivity**
- Traffic follows predictable, structured paths through well-defined aggregation points instead of bouncing across arbitrary links.
- Redundancy and failover can be designed systematically at tier boundaries (for example, dual links from a regional router to core routers), keeping reachability stable and uniform across sites.

---

## 3. Hierarchy Allows for Route Summarization

### What Is Route Summarization?
**Route summarization** (also called *route aggregation*) is combining multiple contiguous/related specific network prefixes into a **single broader summary prefix** advertised to other parts of the network.

Because a **hierarchical topology** groups contiguous IP blocks under specific branches/regions, routers higher in the hierarchy can advertise one summary route instead of many individual routes.

### Topology Example for Route Summarization
Suppose **Region A (`R2`)** owns four subnets inside `10.1.0.0` with mask `255.255.255.0` ($/24$):
- `10.1.0.0/24`
- `10.1.1.0/24`
- `10.1.2.0/24`
- `10.1.3.0/24`

All four sit under `R2`, which connects to `R1`:
```text
       [ R1 (Upstream / Core) ]
                 |
                 |  Advertises ONLY summary: 10.1.0.0/22
                 |
       [ R2 (Region A Router) ]
       /   |        |       \
  .0/24  .1/24    .2/24    .3/24
```

### Routing Table Comparison

#### Routing Table on `R2` (Specific Subnets Known Locally)
| Destination Prefix | Next-Hop / Exit Interface | Notes |
| :--- | :--- | :--- |
| `10.1.0.0/24` | `Gi0/0` (LAN A1) | Specific subnet under `R2` |
| `10.1.1.0/24` | `Gi0/1` (LAN A2) | Specific subnet under `R2` |
| `10.1.2.0/24` | `Gi0/2` (LAN A3) | Specific subnet under `R2` |
| `10.1.3.0/24` | `Gi0/3` (LAN A4) | Specific subnet under `R2` |

#### Routing Table on `R1` (With Route Summarization from `R2`)
Instead of 4 separate `/24` entries for Region A, `R1` can hold a single summarized entry:

| Destination Prefix | Next-Hop | Notes |
| :--- | :--- | :--- |
| `10.1.0.0/22` | `192.168.12.2` (`R2`) | Covers `10.1.0.0` through `10.1.3.255` |

### Why Route Summarization Is Helpful
1. **Smaller Routing Tables:** Routers store fewer entries, saving memory (RAM/TCAM) and keeping route lookups fast.
2. **Reduced Routing Overhead & Stability:** If a link or subnet inside `10.1.2.0/24` flaps or goes down inside Region A, `R2` handles it locally without constantly sending routing updates across the rest of the network—improving stability and convergence.
3. **Better Scalability:** Large hierarchical networks can grow their internal subnets without exploding the routing tables of upstream routers.

---

## 4. How Routers Choose When Multiple Routes Match a Destination IP

A single destination IP address can match **more than one** route entry in a router's routing table (for instance, a specific route and a broader summary route, or the default route).

### Rule 1: Longest Prefix Match (Most Specific Wins)
When multiple entries match a packet's destination IP address, the router chooses the route with the **longest subnet mask / prefix length** (the most specific match, i.e., the largest number of matching leading bits).

#### Example
Suppose a router has these matching entries for destination IP `10.1.2.50`:
1. `0.0.0.0/0` (0 bits) — matches *any* IP
2. `10.0.0.0/8` (8 bits) — matches any `10.x.x.x` IP
3. `10.1.0.0/22` (22 bits) — matches `10.1.0.0`–`10.1.3.255`
4. `10.1.2.0/24` (24 bits) — matches `10.1.2.0`–`10.1.2.255`

All four match `10.1.2.50`, but **`10.1.2.0/24`** has the longest prefix ($/24$ vs. $/22$, $/8$, $/0$), so the router picks **`10.1.2.0/24`**.

### Rule 2: Tie-Breakers If Prefix Length Is Identical
If two routes have the **exact same prefix length** and both match:
- **Administrative Distance (AD):** If learned from different routing sources/protocols, the lower AD is preferred (e.g., Connected < Static < dynamic routing protocols).
- **Metric:** If learned via the same routing protocol with the same prefix length, the route with the lower metric (better path cost) is chosen.
- **Equal-Cost Multi-Path (ECMP):** If prefix length, AD, and metric are tied, the router may balance traffic across those equal paths.

---

## 5. Default Route: The Ultimate Route Summary

### What Is the Default Route?
The **default route**—written as **`0.0.0.0/0`** in IPv4 (and `::/0` in IPv6)—is the **ultimate route summary**. It matches **every possible IP address** because its prefix length is `0` bits (no bits are required to match).

### How It Works in Practice
- Because its prefix length is `/0` (the shortest possible prefix), any more specific route (`/24`, `/16`, `/8`, etc.) will always beat it via **Longest Prefix Match**.
- Therefore, a router only uses the default route as a **last resort** when **no other more specific route** in its routing table matches the destination IP.

### Typical Topology & Use Case
```text
[ LAN Hosts ] ---> [ Edge / Branch Router ] ---> [ ISP / Upstream Router ] ---> [ Internet ]
                    (Has default route:                                         (Has broader/full
                     0.0.0.0/0 -> ISP)                                           Internet routes)
```
- A branch or edge router does not need to store every single route on the entire Internet.
- Instead, it keeps local/internal routes and points **`0.0.0.0/0`** toward its upstream/ISP gateway. Any packet destined for an unknown external IP (like a public website) falls through to the default route and is forwarded toward the Internet.