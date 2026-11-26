# Everything Hosts Do — Direct Connections, Subnetting & ARP

- **Source:** Practical Networking — *Networking Fundamentals* Playlist (Video 5: Everything Hosts Do)
- **Topic:** Direct Host-to-Host Communication (No Switch or Router), Subnetting, ARP Resolution, and DNS

---

## Overview: Types of Host Connections

There are **2 types of connections** between hosts:
1. **Direct Connection:** Host A and Host B are connected directly to each other (without any switch or router in between).
2. **Indirect Connection:** Hosts communicate across switches and/or routers (covered in subsequent lessons).

### Requirements for Directly Connected Hosts (Host A & Host B)
When **Host A** and **Host B** are connected directly to each other:
- Both hosts have a **NIC** (Network Interface Card) and a **MAC Address** (Layer 2 physical address).
- Both hosts have an **IP Address** and a **Subnet Mask** (Layer 3 logical address).
- **Subnet Mask:** Identifies the size of the IP network (which portion of the IP address represents the **Network** and which portion represents the **Host**).

---

## Section 1: Subnetting — What It Is, Network Impact & Easy Math

### 1. What Is Subnetting?
**Subnetting** is the process of taking one large IP network and dividing it into smaller, logical sub-networks (**Subnets**). 
- Instead of placing every device into one massive network pool, you borrow bits from the **Host portion** of an IP address and assign them to the **Network portion** using the **Subnet Mask**.

### 2. What Does Subnetting Do to the Network?
- **Determines Local vs. Remote Traffic:** A host uses its IP address and Subnet Mask to decide if a destination IP is on the **same local network** (can talk directly via MAC/ARP) or on a **foreign/remote network** (must send traffic to a router/gateway).
- **Reduces Broadcast Traffic:** Limits Layer 2 broadcasts (such as ARP requests) to smaller groups of hosts, improving overall network performance.
- **Improves Organization & Security:** Separates departments, servers, and user devices into isolated network boundaries so access controls can be enforced.

### 3. How to Do Subnetting Mathematically (The Easy Way)

Every IPv4 address and Subnet Mask is **32 bits** long, split into **4 Octets** (8 bits each).

#### Step 1: Understand the Subnet Mask
In binary, a Subnet Mask is a continuous string of `1`s followed by `0`s:
- `1` = **Network bit** (identifies the network)
- `0` = **Host bit** (identifies individual hosts inside that network)

| CIDR Notation | Decimal Subnet Mask | Binary Representation | Network Bits | Host Bits |
| :--- | :--- | :--- | :---: | :---: |
| `/24` | `255.255.255.0` | `11111111.11111111.11111111.00000000` | 24 | 8 |
| `/25` | `255.255.255.128` | `11111111.11111111.11111111.10000000` | 25 | 7 |
| `/26` | `255.255.255.192` | `11111111.11111111.11111111.11000000` | 26 | 6 |
| `/27` | `255.255.255.224` | `11111111.11111111.11111111.11100000` | 27 | 5 |
| `/30` | `255.255.255.252` | `11111111.11111111.11111111.11111100` | 30 | 2 |

#### Step 2: The 3 Golden Formulas
Let:
- $s$ = number of **borrowed bits** (extra `1`s added to the original mask)
- $h$ = number of **remaining host bits** (`0`s left in the mask, where $h = 32 - \text{CIDR}$)

1. **Number of Subnets Created:**
   $$\text{Subnets} = 2^s$$

2. **Total IP Addresses per Subnet (Block Size):**
   $$\text{Block Size} = 2^h \quad (\text{or } 256 - \text{Interesting Octet})$$

3. **Usable Host IPs per Subnet:**
   $$\text{Usable Hosts} = 2^h - 2$$
   *(We subtract $2$ because the **first IP** is the **Network ID** and the **last IP** is the **Broadcast Address**—neither can be assigned to a host.)*

#### Step 3: Quick Worked Example
Suppose you have the network `192.168.1.0/24` and want to split it using `/26` (`255.255.255.192`):
- **Borrowed bits ($s$):** $26 - 24 = 2 \rightarrow 2^2 = \mathbf{4\text{ Subnets}}$
- **Host bits left ($h$):** $32 - 26 = 6 \rightarrow 2^6 = \mathbf{64\text{ Total IPs per subnet}}$ (Block Size = $256 - 192 = 64$)
- **Usable Hosts per subnet:** $64 - 2 = \mathbf{62\text{ Usable Hosts}}$

| Subnet # | Network ID (First IP) | Usable Host Range | Broadcast IP (Last IP) |
| :--- | :--- | :--- | :--- |
| **Subnet 1** | `192.168.1.0` | `192.168.1.1` – `192.168.1.62` | `192.168.1.63` |
| **Subnet 2** | `192.168.1.64` | `192.168.1.65` – `192.168.1.126` | `192.168.1.127` |
| **Subnet 3** | `192.168.1.128` | `192.168.1.129` – `192.168.1.190` | `192.168.1.191` |
| **Subnet 4** | `192.168.1.192` | `192.168.1.193` – `192.168.1.254` | `192.168.1.255` |

---

## Section 2: Direct Host-to-Host Communication Process (Step-by-Step + Diagram)

### Scenario Setup
- **Host A:** IP `10.1.1.10` | Mask `255.255.255.0` (`/24`) | MAC `AA:AA:AA:AA:AA:AA`
- **Host B:** IP `10.1.1.20` | Mask `255.255.255.0` (`/24`) | MAC `BB:BB:BB:BB:BB:BB`
- **Connection:** Direct cable between Host A's NIC and Host B's NIC (no switch, no router).

### How Does Host A Know Host B's IP Address Already?
Before sending data, Host A must already know the destination IP (`10.1.1.20`). The common ways Host A obtains it are:
1. **Manual / Direct Input:** A user or administrator directly types the IP address into an application or command (e.g., `ping 10.1.1.20`, `ssh user@10.1.1.20`, or `http://10.1.1.20`).
2. **DNS (Domain Name System):** The user types a human-readable hostname (e.g., `hostb.local` or `example.com`), and Host A queries a **DNS Server** (or local `hosts` file) to translate that domain name into an IP address.
3. **Configuration Files / Hardcoded Endpoints:** Applications or services read the destination IP from config files, environment variables, or service discovery records.

### End-to-End Interaction Diagram (ARP + Data Transfer)

```text
+-----------------------------------+                       +-----------------------------------+
|              HOST A               |                       |              HOST B               |
|  IP:   10.1.1.10 /24              |                       |  IP:   10.1.1.20 /24              |
|  MAC:  AA:AA:AA:AA:AA:AA          |                       |  MAC:  BB:BB:BB:BB:BB:BB          |
+-----------------------------------+                       +-----------------------------------+
                 |                                                            |
  [1] Knows Dest IP (10.1.1.20)                                               |
      Checks Subnet Mask (/24):                                               |
      10.1.1.x == Same Network!                                               |
                 |                                                            |
  [2] Checks ARP Cache for 10.1.1.20                                          |
      --> MISS (MAC unknown)                                                  |
                 |                                                            |
  [3] ARP REQUEST (Broadcast)                                                 |
      Src MAC: AA:..:AA | Dst MAC: FF:FF:FF:FF:FF:FF                          |
      "Who has 10.1.1.20? Tell 10.1.1.10"                                     |
                 | ---------------------------------------------------------> |
                 |                                              [4] Receives Broadcast
                 |                                                  Updates own ARP Cache:
                 |                                                  (10.1.1.10 -> AA:..:AA)
                 |                                                            |
                 |                                              [5] ARP REPLY (Unicast)
                 |                                                  Src MAC: BB:..:BB
                 |                                                  Dst MAC: AA:..:AA
                 |                                                  "10.1.1.20 is at BB:..:BB"
                 | <--------------------------------------------------------- |
  [6] Updates ARP Cache:                                                      |
      (10.1.1.20 -> BB:..:BB)                                                 |
                 |                                                            |
  [7] DATA PACKET / FRAME (e.g., ICMP Ping / HTTP)                            |
      L3: Src IP 10.1.1.10  -> Dst IP 10.1.1.20                               |
      L2: Src MAC AA:..:AA  -> Dst MAC BB:..:BB                               |
                 | ---------------------------------------------------------> |
                 |                                              [8] Processes Data &
                 |                                                  Sends Reply Back
                 | <--------------------------------------------------------- |
```

### Detailed Step-by-Step Breakdown

1. **Same-Network Check (Subnet Calculation):**
   - Host A compares its own IP (`10.1.1.10`) and Subnet Mask (`255.255.255.0`) against Host B's IP (`10.1.1.20`).
   - Both share the network portion `10.1.1.0`, so Host A knows Host B is on the **same local network** and can be reached directly without a router.
2. **ARP Cache Lookup:**
   - To build the Layer 2 Ethernet frame, Host A needs Host B's **MAC address**.
   - Host A checks its local **ARP Cache** (a temporary table in memory mapping IP addresses to MAC addresses). If the entry is missing, ARP begins.
3. **ARP Request (Broadcast):**
   - Host A sends an **ARP Request** frame to the Layer 2 broadcast MAC address (`FF:FF:FF:FF:FF:FF`), asking: *"Who has IP `10.1.1.20`? Tell `10.1.1.10` (`AA:AA:AA:AA:AA:AA`)."*
4. **Host B Learns Host A's MAC & Replies (Unicast):**
   - Host B receives the broadcast, sees its own IP (`10.1.1.20`), and immediately saves Host A's IP-to-MAC mapping in its **own ARP Cache** (so it won't need to ARP for Host A later).
   - Host B sends a direct **Unicast ARP Reply** back to `AA:AA:AA:AA:AA:AA` containing its MAC address (`BB:BB:BB:BB:BB:BB`).
5. **Host A Updates ARP Cache:**
   - Host A receives the reply and stores `10.1.1.20 -> BB:BB:BB:BB:BB:BB` in its **ARP Cache** (typically cached for a few minutes to avoid repeating ARP broadcasts for every packet).
6. **Actual Data Transmission:**
   - Host A now encapsulates the payload (e.g., an ICMP `ping` request) with:
     - **Layer 3 Header:** Source IP `10.1.1.10` | Destination IP `10.1.1.20`
     - **Layer 2 Header:** Source MAC `AA:AA:AA:AA:AA:AA` | Destination MAC `BB:BB:BB:BB:BB:BB`
   - Host B receives the frame, verifies the MAC and IP match, processes the payload, and sends its response directly back using the cached MAC address.

---

## DNS (Domain Name System) — What It Is & What Work It Does

### What Is DNS?
**DNS (Domain Name System)** is the "phonebook of the Internet" — a distributed naming system that translates human-friendly domain names (like `google.com` or `server.local`) into machine-routable **IP addresses** (like `142.250.190.78`).

### What Work Does DNS Do?
1. **Name-to-IP Resolution (Forward Lookup):**
   - Hosts communicate at Layer 3 using numerical **IP addresses**, not text names. When you enter a domain name in a browser or run `ping example.com`, DNS resolves that name into the target's IP address before ARP or packet transmission can begin.
2. **Caching for Speed:**
   - Your OS, browser, and local DNS resolver cache previous DNS lookups (using a **TTL — Time To Live** value) so repeated requests to the same domain do not require querying external DNS servers every time.
3. **Hierarchical Querying:**
   - If the IP is not cached locally, a DNS Resolver queries the DNS hierarchy (**Root Servers $\rightarrow$ TLD Servers like `.com` $\rightarrow$ Authoritative Name Servers**) to retrieve the exact IP record (`A` record for IPv4, `AAAA` record for IPv6).