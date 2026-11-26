# Networking Fundamentals — Switches (Part 1): Core Concepts & Three Switch Actions

- **Source Context:** *Everything Switches do - Part 1 — Networking Fundamentals - Lesson 4* (*Practical Networking* Playlist)

---

## 1. What is a Switch?
- A **Switch** is a **Layer 2 (Data Link Layer)** networking device that interconnects multiple hosts/devices within a **Local Area Network (LAN)** using **MAC addresses**.
- Unlike a **Hub** (which blindly repeats/floods every incoming signal out of all ports), a switch makes intelligent forwarding decisions by maintaining a **MAC Address Table** (also called a CAM table).

---

## 2. The Three Core Switch Actions

Every time a switch receives an Ethernet frame on an incoming port, it performs up to three actions using its **MAC Address Table**:

### 1. Learning
- **What it checks:** The frame's **Source MAC Address** and the **ingress (incoming) port** on which the frame arrived.
- **What it does:** Records `(Source MAC Address -> Incoming Port)` in the MAC Address Table (and refreshes its timer if already present).
- **Important Note:** Switches **only learn from Source MAC addresses** — never from Destination MAC addresses.

### 2. Forwarding
- **When it happens:** The frame's **Destination MAC Address** is a specific unicast MAC address that **already exists** in the MAC Address Table.
- **What it does:** Sends the frame **out of only the single specific port** mapped to that Destination MAC address (assuming it is not the same port the frame came in on).

### 3. Flooding
- **When it happens:** Either:
  1. The frame's **Destination MAC Address** is **not yet in** the MAC Address Table (unknown unicast), **OR**
  2. The frame is a **Broadcast** (`ffff.ffff.ffff`) or **Multicast** frame.
- **What it does:** Duplicates the frame and sends it **out of all active ports except the port it arrived on**.

```text
Incoming Frame on Port 1: [ Src MAC: aaaa | Dst MAC: ? or ffff.ffff.ffff ]
        │
        ├──► 1. LEARN:   Record (aaaa -> Port 1) in MAC Address Table
        │
        └──► 2. DECIDE:
                ├── Known Dst MAC (e.g., bbbb -> Port 3) ──► FORWARD out Port 3 only
                └── Unknown Dst MAC or Broadcast          ──► FLOOD out Ports 2, 3, 4 (all except Port 1)
```

---

## 3. Quick Summary of Part 1
- **Switch Layer:** Layer 2 (MAC addresses).
- **Table Used:** MAC Address Table (`MAC Address -> Switch Port`).
- **Three Actions:** **Learn** (Source MAC), **Forward** (known single port), **Flood** (all ports except incoming port).