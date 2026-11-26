# Networking Fundamentals — Hosts Connected Through a Router

- **Source Video:** *Everything Hosts do to speak on the Internet - Part 2 — Networking Fundamentals - Lesson 3* (Video #6 in the *Practical Networking* Playlist)
- **Previous Context Check:** In Part 1 (Lesson 3a — *Hosts Connected Directly*), **Subnetting / Subnet Masks** were introduced conceptually as the rule a host uses to compare its own IP address against a destination IP address to answer one critical question: *"Is this target IP on my local network, or on a foreign network?"* (Full mathematical subnetting is covered later in Module 2 of the course).

---

## 1. Core Rule: Local vs. Foreign Networks

Whenever a host wants to send data, it uses its **IP Address** and **Subnet Mask** to determine where the destination lives:

1. **Same Network (Local):**
   - The host uses **ARP** to find the **destination host's MAC address** directly.
   - Layer 2 Destination MAC = **Target Host's MAC**.
2. **Different Network (Foreign):**
   - The host **cannot** reach the destination directly at Layer 2 because broadcasts (ARP) do not cross routers.
   - Instead, the host sends the packet to its **Default Gateway (Router)**.
   - Layer 2 Destination MAC = **Default Gateway's (Router's) MAC**.
   - Layer 3 Destination IP = **Final Destination Host's IP** (never the router's IP!).

> **Golden Rule of Packet Traveling:**
> - **Layer 3 (IP Header)** gets the packet **End-to-End** (from original sender to final receiver).
> - **Layer 2 (MAC Header)** gets the packet **Hop-by-Hop** (from one device interface to the very next device interface).

---

## 2. Case 1: Two Hosts Connected Through a Single Router

### Network Topology & Addressing

```text
+-------------------+                 +-----------------------+                 +-------------------+
|      Host A       |                 |       Router R1       |                 |      Host C       |
|-------------------|                 |-----------------------|                 |-------------------|
| IP:  11.11.11.77  |                 | Left NIC (Gateway A): |                 | IP:  33.33.33.99  |
| MAC: aaaa         |                 |   IP:  11.11.11.1     |                 | MAC: cccc         |
| GW:  11.11.11.1   |                 |   MAC: 1111           |                 | GW:  33.33.33.1   |
+---------+---------+                 |                       |                 +---------+---------+
          |                           | Right NIC (Gateway C):|                           |
          |    Network 11.11.11.x     |   IP:  33.33.33.1     |    Network 33.33.33.x     |
          +---------------------------+   MAC: 3333           +---------------------------+
                                      +-----------------------+
```

*(Note: MAC addresses are shortened to 4 hex digits like `aaaa` for readability, just as in the video).*

---

### Step-by-Step: Sending Data from Host A to Host C (Forward Path)

#### Step 1: Host A Determines the Network & Builds Layer 3
- Host A wants to send data to **Host C (`33.33.33.99`)** (learned via DNS, user input, etc.).
- Host A compares `33.33.33.99` with its own IP (`11.11.11.77`) using its Subnet Mask and realizes Host C is on a **foreign network**.
- Host A creates the **Layer 3 (IP) Header**:
  - **Source IP:** `11.11.11.77` (Host A)
  - **Destination IP:** `33.33.33.99` (Host C)

#### Step 2: Host A Resolves the Default Gateway's MAC via ARP
- Because Host C is on a foreign network, Host A must deliver the packet to its **Default Gateway (`11.11.11.1`)**.
- Host A checks its **ARP Cache** for `11.11.11.1`. Assume the cache is empty:
  1. **ARP Request (Broadcast):** Host A broadcasts to `ffff` asking: *"Who has IP `11.11.11.1`? Tell `11.11.11.77` (`aaaa`)."*
  2. Router R1 receives it, records Host A (`11.11.11.77 -> aaaa`) in its own ARP cache, and replies.
  3. **ARP Reply (Unicast):** Router R1 sends a unicast reply to `aaaa`: *"I am `11.11.11.1`, and my MAC is `1111`."*
  4. Host A saves `11.11.11.1 -> 1111` in its ARP Cache.

#### Step 3: Host A Builds Layer 2 and Sends the Frame to R1 (Hop 1)
- Host A wraps the Layer 3 packet inside a **Layer 2 (Ethernet) Header** and puts it on the wire:

```text
[ L2: Src MAC: aaaa | Dst MAC: 1111 ] [ L3: Src IP: 11.11.11.77 | Dst IP: 33.33.33.99 ] [ DATA ]
```

#### Step 4: Router R1 Receives, Inspects, and Routes the Packet
1. **L2 Strip:** Router R1 sees Destination MAC `1111` matches its left interface, accepts the frame, and **strips off the old L2 header**.
2. **L3 Lookup:** R1 inspects the Destination IP (`33.33.33.99`) and checks its **Routing Table**.
   - R1 sees that `33.33.33.x` is **Directly Connected** to its right interface (`33.33.33.1`).
3. **ARP for Host C:** R1 checks its ARP cache for `33.33.33.99`. If unknown:
   - R1 sends an **ARP Request** out its right interface: *"Who has `33.33.33.99`? Tell `33.33.33.1` (`3333`)."*
   - Host C replies with an **ARP Reply**: *"I am `33.33.33.99`, my MAC is `cccc`."*
   - R1 saves `33.33.33.99 -> cccc` in its ARP cache (and Host C saves `33.33.33.1 -> 3333`).

#### Step 5: Router R1 Rewrites Layer 2 and Delivers to Host C (Hop 2)
- R1 keeps the **L3 header completely untouched**, generates a **brand-new L2 header**, and transmits the frame to Host C:

```text
[ L2: Src MAC: 3333 | Dst MAC: cccc ] [ L3: Src IP: 11.11.11.77 | Dst IP: 33.33.33.99 ] [ DATA ]
```

- **Host C** sees Destination MAC `cccc`, strips L2, sees Destination IP `33.33.33.99` matches its own IP, strips L3, and passes the **DATA** up to the application!

---

### Step-by-Step: Host C Replying to Host A (Return Path)

When Host C sends a response back to Host A, the exact same logic runs in reverse:

1. **Host C** creates L3: `Src IP: 33.33.33.99` | `Dst IP: 11.11.11.77`.
2. Host C sees `11.11.11.77` is on a foreign network, so it targets its Default Gateway (`33.33.33.1`, MAC `3333` — already cached from the earlier ARP!).
3. **Hop 1 (Host C $\rightarrow$ R1):**
   ```text
   [ L2: Src MAC: cccc | Dst MAC: 3333 ] [ L3: Src IP: 33.33.33.99 | Dst IP: 11.11.11.77 ] [ REPLY ]
   ```
4. **Router R1** strips L2, checks its Routing Table for `11.11.11.77`, finds `11.11.11.x` on its left interface, looks up `11.11.11.77` (`aaaa`) in its ARP cache, and builds a new L2 header.
5. **Hop 2 (R1 $\rightarrow$ Host A):**
   ```text
   [ L2: Src MAC: 1111 | Dst MAC: aaaa ] [ L3: Src IP: 33.33.33.99 | Dst IP: 11.11.11.77 ] [ REPLY ]
   ```
6. **Host A** receives the frame, verifies MAC `aaaa` and IP `11.11.11.77`, and processes the reply.

---

## 3. Case 2: Two Hosts Connected Through Multiple Routers (R1 & R2)

What if Host A (`11.11.11.77`) wants to talk to **Host D (`44.44.44.88`)**, which sits behind **two routers** (`R1` and `R2`)?

### Multi-Router Topology

```text
+----------+        +---------------+        +---------------+        +----------+
|  Host A  |        |   Router R1   |        |   Router R2   |        |  Host D  |
|----------|        |---------------|        |---------------|        |----------|
|11.11.11.77        |L: 11.11.11.1  |        |L: 22.22.22.2  |        |44.44.44.88
|MAC: aaaa |        |   MAC: 1111   |        |   MAC: 2222   |        |MAC: dddd |
|GW: .11.1 |        |R: 22.22.22.1  |        |R: 44.44.44.1  |        |GW: .44.1 |
+----+-----+        |   MAC: 2211   |        |   MAC: 4411   |        +----+-----+
     |              +-------+-------+        +-------+-------+             |
     |  Net 11.11.11.x      |    Net 22.22.22.x      |   Net 44.44.44.x    |
     +----------------------+------------------------+---------------------+
```

### Complete Hop-by-Hop Flow (Host A $\rightarrow$ R1 $\rightarrow$ R2 $\rightarrow$ Host D)

Notice that **Host A does NOT know or care** how many routers exist between itself and Host D. Host A simply sends the packet to its Default Gateway (`R1`).

```text
HOP 1: Host A ---> Router R1
L2 Header: [ Src MAC: aaaa | Dst MAC: 1111 ]  <-- MACs of Host A & R1 Left NIC
L3 Header: [ Src IP: 11.11.11.77 | Dst IP: 44.44.44.88 ]  <-- End-to-End IPs
   │
   ▼  (R1 strips L2, checks Routing Table for 44.44.44.88 -> Next-Hop: 22.22.22.2, ARPs for 22.22.22.2 -> 2222)
HOP 2: Router R1 ---> Router R2
L2 Header: [ Src MAC: 2211 | Dst MAC: 2222 ]  <-- MACs of R1 Right NIC & R2 Left NIC
L3 Header: [ Src IP: 11.11.11.77 | Dst IP: 44.44.44.88 ]  <-- UNCHANGED!
   │
   ▼  (R2 strips L2, checks Routing Table for 44.44.44.88 -> Directly Connected, ARPs for 44.44.44.88 -> dddd)
HOP 3: Router R2 ---> Host D
L2 Header: [ Src MAC: 4411 | Dst MAC: dddd ]  <-- MACs of R2 Right NIC & Host D
L3 Header: [ Src IP: 11.11.11.77 | Dst IP: 44.44.44.88 ]  <-- UNCHANGED!
```

### Key Takeaways for Multiple Routers
- **Every Transit Router (like R1):** Looks up the destination network (`44.44.44.x`) in its Routing Table, finds a **Next-Hop IP** (`22.22.22.2`), ARPs for that next-hop router's MAC (`2222`), and rewrites L2.
- **The Final Router (like R2):** Sees the destination network (`44.44.44.x`) is **Directly Connected**, ARPs for the final host's MAC (`dddd`), and delivers the packet.
- Whether there are **2 routers or 20 routers** across the Internet, this exact process repeats at every single hop!

---

## 4. What Changes in L2 and L3 When You Change the Destination?

Suppose **Host A (`11.11.11.77`)** changes who it is talking to. How do the **Layer 2 (MAC)** and **Layer 3 (IP)** headers behave?

| Scenario | Layer 3 (IP Header) Behavior | Layer 2 (MAC Header) Behavior |
| :--- | :--- | :--- |
| **1. Destination is on the SAME Network**<br>*(e.g., Host A $\rightarrow$ Host B on `11.11.11.x`)* | **Src IP:** `11.11.11.77`<br>**Dst IP:** Host B's IP (`11.11.11.55`)<br>*(Remains constant)* | **Src MAC:** `aaaa` (Host A)<br>**Dst MAC:** `bbbb` (**Host B's MAC directly**)<br>*(No router involved; single hop)* |
| **2. Destination is on a FOREIGN Network**<br>*(e.g., Host A $\rightarrow$ Host C on `33.33.33.x`)* | **Src IP:** `11.11.11.77`<br>**Dst IP:** Host C's IP (`33.33.33.99`)<br>*(Remains constant across ALL routers)* | **Leaving Host A:** `Src: aaaa` \| `Dst: 1111` (**Default Gateway MAC**)<br>**At each Router:** Completely **stripped and rewritten** with the current link's outgoing & incoming NIC MACs. |
| **3. Switching from Host C (`33.33.33.99`) to Host D (`44.44.44.88`)** | **Dst IP changes** to `44.44.44.88` (the new target host's IP). | **Leaving Host A, L2 DOES NOT CHANGE AT ALL!**<br>Because both Host C and Host D are foreign, Host A still sets:<br>`Src MAC: aaaa` \| `Dst MAC: 1111` (Default Gateway R1).<br>*(Only the routers downstream change their L2 headers to follow the new route).* |

### Summary Comparison Diagram: Changing Destination from Host A

```text
Case A: Destination = Local Host (11.11.11.55)
+-------------------------------------------------------------------+
| L2: Src = aaaa | Dst = bbbb (Target Host MAC)                     |
| L3: Src = 11.11.11.77 | Dst = 11.11.11.55 (Target Host IP)        |
+-------------------------------------------------------------------+

Case B: Destination = Foreign Host C (33.33.33.99)
+-------------------------------------------------------------------+
| L2: Src = aaaa | Dst = 1111 (Default Gateway MAC)                 |
| L3: Src = 11.11.11.77 | Dst = 33.33.33.99 (Target Host C IP)      |
+-------------------------------------------------------------------+

Case C: Destination = Foreign Host D (44.44.44.88)
+-------------------------------------------------------------------+
| L2: Src = aaaa | Dst = 1111 (SAME Default Gateway MAC!)           |
| L3: Src = 11.11.11.77 | Dst = 44.44.44.88 (New Target Host D IP)  |
+-------------------------------------------------------------------+
```

---

## 5. Quick Exam & Interview Cheat Sheet

1. **Does a host ever put a router's IP address in the Layer 3 Destination IP field when sending normal traffic?**
   - **No.** The Layer 3 Destination IP is *always* the final destination host's IP. The router's IP is only used to look up the router's MAC address via ARP.
2. **Does a host ever put a foreign host's MAC address in the Layer 2 Destination MAC field?**
   - **No.** A host never knows (or cares about) MAC addresses outside its own local network. For any foreign destination, L2 Destination MAC is always the **Default Gateway's MAC**.
3. **What happens to L2 and L3 at every router hop?**
   - **L2 Header:** Stripped off and rebuilt from scratch at every hop.
   - **L3 Header:** Preserved end-to-end from source host to destination host (except minor fields like TTL/Checksum).