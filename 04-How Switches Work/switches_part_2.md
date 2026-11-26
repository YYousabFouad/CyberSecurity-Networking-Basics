# Networking Fundamentals — Switches (Part 2): Unicast vs. Broadcast, VLANs & Multiple Switches

- **Source Video:** *Everything Switches do - Part 2 — Networking Fundamentals - Lesson 4* (Video #8 in the *Practical Networking* Playlist)
- **Prerequisite (Part 1):** Familiar with the **MAC Address Table** and the three switch actions (**Learning**, **Flooding**, and **Forwarding**).

---

## 1. Unicast Frames vs. Broadcast Frames

Switches handle frames differently depending on whether the frame is meant for **one specific host** or **every host** on the network:

| Feature | Unicast Frame | Broadcast Frame |
| :--- | :--- | :--- |
| **Definition** | A frame sent from **one host** to **one specific destination host**. | A frame sent from **one host** intended for **every host** on the local network. |
| **Destination MAC** | The specific MAC address of another host (e.g., `bbbb` or `cccc`). | Always the reserved all-ones MAC address: **`ffff.ffff.ffff`** (all `F`s). |
| **Switch Behavior** | **Conditional:**<br>- If the Destination MAC is **unknown** (not in the MAC table yet), the switch **Floods** it.<br>- Once the target replies and the switch **Learns** its MAC, all future frames are **Forwarded** out that single port only. | **Always Flooded:**<br>- A switch **never** learns `ffff.ffff.ffff` as a Source MAC on a single port.<br>- Therefore, a Broadcast frame is **always flooded** out every port in that network/VLAN (except the incoming port). |
| **Common Example** | Normal data transfer (HTTP, SSH, Ping Echo Request/Reply). | **ARP Request** (*"Who has IP `x.x.x.x`? Tell me!"*). |

```text
Unicast (Known MAC)                        Broadcast (Dst MAC: ffff.ffff.ffff)
  Host A ---> [ Switch ] ---> Host B         Host A ---> [ Switch ] ---> Host B
                  (Port 2 only)                              |---------> Host C
                                                             |---------> Host D
                                                     (Flooded to ALL other ports!)
```

---

## 2. VLANs (Virtual Local Area Networks) — Detailed & Simple Explanation

### What is a VLAN?
A **VLAN (Virtual Local Area Network)** is a feature on a switch that **divides a single physical switch into multiple isolated logical groups ("mini-switches")**.

- Without VLANs, every port on a physical switch belongs to **one single broadcast domain** (one network) — meaning a broadcast sent by one computer reaches every single device plugged into that switch.
- With VLANs, you can group specific switch ports together virtually so they behave as if they were plugged into completely separate physical switches.

### Simple Analogy: The "Mini-Switches" Concept
Imagine you have one **24-port physical switch**, and you configure:
- **Ports 1–8** $\rightarrow$ **VLAN 10** (e.g., *HR Department* / Blue Group)
- **Ports 9–16** $\rightarrow$ **VLAN 20** (e.g., *Engineering Department* / Green Group)
- **Ports 17–24** $\rightarrow$ **VLAN 30** (e.g., *Guest Wi-Fi* / Red Group)

Logically, that single physical switch now acts like **three completely separate physical switches** sitting inside one metal box!

```text
                   ONE PHYSICAL 24-PORT SWITCH
+---------------------------------------------------------------+
|  [ VLAN 10: Mini-Switch 1 ]   |   [ VLAN 20: Mini-Switch 2 ]  |
|   Ports 1, 2, 3, 4            |    Ports 5, 6, 7, 8           |
|   Host A <---> Host B         |    Host C <---> Host D        |
+-------------------------------+-------------------------------+
          ^                                     ^
          |------- NO DIRECT L2 TRAFFIC --------|
          (Broadcasts & frames stay strictly inside their own VLAN)
```

### How a Switch Operates Inside Each VLAN
- **Independent MAC Address Tables:** A switch maintains a separate MAC address table (or VLAN-tagged entries) for each VLAN.
- **All 3 Switch Actions Happen Within Each VLAN:**
  1. **Learning:** Happens per VLAN (e.g., learning `aaaa` on Port 1 inside VLAN 10).
  2. **Flooding:** A broadcast (`ffff.ffff.ffff`) or unknown unicast frame arriving on a VLAN 10 port is flooded **ONLY to other ports in VLAN 10**. Ports in VLAN 20 never see it!
  3. **Forwarding:** Unicast frames are forwarded only between ports belonging to the **same VLAN**.
- **How Do Two Different VLANs Communicate?**
  - Because VLANs act like separate networks (usually assigned different IP Subnets, e.g., `10.10.10.x` for VLAN 10 and `10.10.20.x` for VLAN 20), **hosts in different VLANs CANNOT talk directly through the switch at Layer 2**.
  - To communicate between VLAN 10 and VLAN 20, traffic must go up to a **Router** (Layer 3 device) — a process known as **Inter-VLAN Routing**.

### Why Do We Use VLANs? (Real-World & Security Benefits)
1. **Security & Isolation:** Keeps sensitive devices (e.g., Finance servers, Security cameras, Admin PCs) isolated from general staff or guest networks.
2. **Performance (Smaller Broadcast Domains):** Reduces unnecessary broadcast traffic (like ARP) so hosts don't waste CPU processing broadcasts meant for other departments.
3. **Cost & Flexibility:** You don't need to buy separate physical switches for every department or floor — you can logically group users anywhere in a building onto the same VLAN.

---

## 3. How Switches Operate When Using Multiple Switches

In real networks, hosts are rarely connected to just one switch. Usually, **Switch 1** is connected to **Switch 2** via an uplink cable.

### How Multiple Switches Learn & Forward (Single Network / No VLANs)

```text
+--------+         +------------------+                 +------------------+         +--------+
| Host A |---P1----|     Switch 1     |---P3-------P1---|     Switch 2     |---P2----| Host C |
| MAC:aa |         |                  |                 |                  |         | MAC:cc |
+--------+         | MAC Table (SW1): |                 | MAC Table (SW2): |         +--------+
                   |  aa -> Port 1    |                 |  aa -> Port 1    |
                   |  cc -> Port 3    |                 |  cc -> Port 2    |
                   +------------------+                 +------------------+
```

Let's trace what happens when **Host A (`aa`)** sends a frame to **Host C (`cc`)** for the first time:

1. **Host A $\rightarrow$ Switch 1 (Port 1):**
   - **Learn:** Switch 1 sees `Src MAC: aa` arrive on **Port 1**, so it records `aa -> Port 1`.
   - **Flood:** Switch 1 doesn't know where `cc` is yet, so it **floods** the frame out all other ports — including **Port 3** (the link to Switch 2).
2. **Switch 1 $\rightarrow$ Switch 2 (Port 1):**
   - **Learn:** Switch 2 receives the frame on its **Port 1** and sees `Src MAC: aa`. Switch 2 records `aa -> Port 1` (meaning *"to reach `aa`, send it out Port 1 toward Switch 1"*).
   - **Flood:** Switch 2 doesn't know `cc` yet, so it **floods** the frame out its other ports — including **Port 2** to Host C.
3. **Host C Replies (`Src MAC: cc`, `Dst MAC: aa`):**
   - **Switch 2** receives the reply on Port 2, **learns** `cc -> Port 2`, checks its table for `aa`, and **forwards** it directly out **Port 1** to Switch 1.
   - **Switch 1** receives the reply on Port 3, **learns** `cc -> Port 3`, checks its table for `aa`, and **forwards** it directly out **Port 1** to Host A.

> **Key Rule:** A single switch port (like the link between Switch 1 and Switch 2) **can have multiple MAC addresses mapped to it** in a switch's MAC Address Table!

---

### How Multiple Switches Operate WITH VLANs (Access Ports vs. Trunk Ports)

What if **Switch 1** and **Switch 2** both have hosts in **VLAN 10** and **VLAN 20**?

```text
+--------+                                                                   +--------+
| Host A | (VLAN 10)                                               (VLAN 10) | Host C |
+---+----+         +--------------+                 +--------------+         +----+---+
    |              |   Switch 1   |                 |   Switch 2   |              |
    +--[Access 10]-|              |                 |              |-[Access 10]--+
                   |              |=== TRUNK LINK ==|              |
    +--[Access 20]-|              | (Carries V10 &  |              |-[Access 20]--+
    |              |              |  V20 w/ Tags)   |              |              |
+---+----+         +--------------+                 +--------------+         +----+---+
| Host B | (VLAN 20)                                               (VLAN 20) | Host D |
+--------+                                                                   +--------+
```

Instead of running two separate cables between Switch 1 and Switch 2 (one for VLAN 10 and one for VLAN 20), switches use **one single Trunk Link** and two types of ports:

1. **Access Port (Host-Facing Port):**
   - Connects a switch to an end host (PC, printer, server).
   - Belongs to **only ONE VLAN** (e.g., VLAN 10).
   - Sends and receives normal, **untagged** Ethernet frames (the PC has no idea VLANs even exist).
2. **Trunk Port (Switch-to-Switch Port):**
   - Connects **Switch 1 to Switch 2** (or a switch to a router).
   - Carries traffic for **MULTIPLE VLANs** simultaneously over a single physical cable.
   - Uses **VLAN Tagging (802.1Q):** Before Switch 1 sends a frame across the Trunk link, it inserts a small **VLAN Tag** (e.g., `VLAN 10` or `VLAN 20`) into the Layer 2 header.
   - When Switch 2 receives the tagged frame on its Trunk port, it reads the tag (`VLAN 10`), **removes the tag**, and forwards/floods the frame **only** to ports belonging to **VLAN 10** on Switch 2!

---

## 4. Quick Summary Checklist

- **Unicast Frame:** Specific Destination MAC $\rightarrow$ Flooded if unknown, **Forwarded** once learned.
- **Broadcast Frame:** Destination MAC `ffff.ffff.ffff` $\rightarrow$ **Always Flooded** across the local VLAN.
- **VLAN:** Splits one physical switch into multiple isolated **"mini-switches"**; Learning, Flooding, and Forwarding happen independently inside each VLAN.
- **Multiple Switches:**
  - Switch-to-switch ports map **multiple MAC addresses** to the uplink port.
  - **Access Ports** carry **1 VLAN** (untagged) to hosts.
  - **Trunk Ports** carry **multiple VLANs** (tagged with **802.1Q**) between switches.