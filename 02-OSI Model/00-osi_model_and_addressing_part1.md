# OSI Model & Network Addressing

## Overview of the OSI Model

* **Purpose of Networking:** Allow 2 hosts to share data with one another.

* **The OSI Model (*Open Systems Interconnection*):** The rules for networking are divided into **7 layers**, known as the OSI Model.

  * Each layer serves a specific function.

  * If all layers are functioning, hosts can share data.

## Layer 1 – Physical Layer $\rightarrow$ Transporting Bits

* **Core Concept:**

  * Computer data exists in the form of **bits** (`0`s and `1`s).

  * Something has to physically transport those bits between hosts.

* **Layer 1 Technologies:**

  * Cables, Wi-Fi, Repeaters, Hubs.

## Layer 2 – Data Link Layer $\rightarrow$ Hop-to-Hop Delivery

* **Core Concept:**

  * Interacts directly with the wire (Physical Layer) via a **NIC** (*Network Interface Card* / *Wi-Fi Access Card*).

  * Often, communication between hosts requires **multiple hops** across intermediate devices.

* **Layer 2 Addressing Scheme: MAC Addresses**

  * To accomplish hop-to-hop delivery, Layer 2 uses **MAC addresses**.

  * **Format:** $48\text{ bits}$, represented as $12$ hexadecimal digits.

  * Every NIC has a unique MAC address.

  * **Different Ways to Represent a MAC Address:**

    * **Windows:** `94-65-9C-3B-8A-E5`

    * **Linux:** `94:65:9C:3B:8A:E5`

    * **Cisco Switches:** `9465.9C3B.8AE5`

* **Layer 2 Technologies:**

  * NICs, Switches.

## Layer 3 – Network Layer $\rightarrow$ End-to-End Delivery

* **Core Concept:**

  * Uses its own addressing scheme—the **IP Address**—to handle delivery from the original source to the final destination across different networks.

* **Layer 3 Technologies:**

  * Routers, Hosts (anything with an IP address).

## Why Are There Both IP Addresses and MAC Addresses?

Each one serves a different purpose:

* **MAC Address** and **IP Address** are independent functions.

### Detailed Explanation: Why Two Types of Addresses Are Needed

1. **Local vs. Global Scope (Hop-to-Hop vs. End-to-End):**

   * **MAC Addresses (Layer 2)** operate strictly within a **local network segment** (hop-to-hop). A switch uses MAC addresses to deliver frames from one device to the very next physical interface on the same local network. However, MAC addresses are flat—burned into the hardware NIC—and contain no information about *where* in the world or on which network a device is located.

   * **IP Addresses (Layer 3)** are **hierarchical and logical** (end-to-end). Just like a postal address (Country $\rightarrow$ City $\rightarrow$ Street), an IP address identifies both the *network* a device belongs to and the *host* within that network, allowing routers to forward packets across the entire internet from the original sender to the final receiver.

2. **Portability and Network Changes:**

   * If you take your laptop from home to a coffee shop, your **MAC address** stays the same because it is tied to your physical network card, while your **IP address** changes to match the coffee shop's network so routers know where to send your traffic.

3. **How They Work Together (Analogy):**

   * Imagine sending a letter from an office in Cairo to an office in Alexandria using courier vans:

     * **IP Address (Source & Destination):** Written on the envelope as *"From: Office A, Cairo* $\rightarrow$ *To: Office B, Alexandria."* This never changes during the journey; it tells the system the ultimate start and end points (**End-to-End**).

     * **MAC Address (Source & Destination):** Represents the license plates of the courier vans handing off the package at each checkpoint along the highway (**Hop-to-Hop**). At every router/hop, the Layer 2 MAC addresses change to reflect the current sender and the next immediate router, while the Layer 3 IP addresses remain fixed on the original source and final destination.

## ARP: Address Resolution Protocol

* **Definition:** Links a **Layer 3 address** (IP Address) to a **Layer 2 address** (MAC Address) so devices on a local network can resolve the physical hardware address corresponding to a target IP address.

## How a Packet Travels: Client PC $\rightarrow$ Routers $\rightarrow$ Destination Server/PC

Consider a communication path across multiple networks:

$$\text{Client PC} \longrightarrow \text{Router 1} \longrightarrow \text{Router 2} \longrightarrow \text{Router 3} \longrightarrow \text{Destination Server / PC}$$

Throughout this entire journey, two golden rules apply:

* **Layer 3 IP Addresses (Source & Destination):** **Stay the SAME** from start to finish (End-to-End).
* **Layer 2 MAC Addresses (Source & Destination):** **Change at EVERY single hop** (Hop-to-Hop).

### Step-by-Step Packet Journey

1. **Step 1: Client PC Prepares the Packet**
   * The **Client PC** wants to send data to the **Destination Server/PC** (which is on a different network).
   * **Layer 3 Header:** `Src IP: Client PC` | `Dst IP: Destination Server/PC`
   * Because the destination is outside its local subnet, the Client PC knows it must send the packet to its **Default Gateway (Router 1)**.
   * It uses **ARP** (if not already cached) to find **Router 1's MAC address**.
   * **Layer 2 Header:** `Src MAC: Client PC` | `Dst MAC: Router 1 (incoming interface)`
   * The frame is converted into bits (Layer 1) and sent across the wire/Wi-Fi (through any local switch) to **Router 1**.

2. **Step 2: Hop 1 — Router 1 $\rightarrow$ Router 2**
   * **Router 1** receives the bits, strips off the Layer 2 header (seeing its own MAC address), and inspects the **Layer 3 Destination IP**.
   * Router 1 checks its **Routing Table** and sees that to reach the Destination IP, the next hop is **Router 2**.
   * Router 1 builds a **brand-new Layer 2 header** for the next link (using ARP to get Router 2's MAC address):
     * **Layer 3 Header (Unchanged):** `Src IP: Client PC` | `Dst IP: Destination Server/PC`
     * **Layer 2 Header (Updated):** `Src MAC: Router 1 (outgoing interface)` | `Dst MAC: Router 2 (incoming interface)`
   * Router 1 transmits the bits to **Router 2**.

3. **Step 3: Hop 2 — Router 2 $\rightarrow$ Router 3**
   * **Router 2** receives the frame, strips off the old Layer 2 header, and reads the **Layer 3 Destination IP**.
   * Its routing table points to **Router 3** as the next hop.
   * Router 2 creates a **new Layer 2 header** for the link to Router 3:
     * **Layer 3 Header (Unchanged):** `Src IP: Client PC` | `Dst IP: Destination Server/PC`
     * **Layer 2 Header (Updated):** `Src MAC: Router 2 (outgoing interface)` | `Dst MAC: Router 3 (incoming interface)`
   * Router 2 forwards the bits to **Router 3**.

4. **Step 4: Final Hop — Router 3 $\rightarrow$ Destination Server / PC**
   * **Router 3** strips off the Layer 2 header and inspects the **Layer 3 Destination IP**.
   * Router 3 sees that the Destination IP belongs to a **directly connected local network**.
   * Router 3 uses **ARP** to find the **Destination Server/PC's MAC address** and creates the final Layer 2 header:
     * **Layer 3 Header (Unchanged):** `Src IP: Client PC` | `Dst IP: Destination Server/PC`
     * **Layer 2 Header (Updated):** `Src MAC: Router 3 (outgoing interface)` | `Dst MAC: Destination Server/PC`
   * The frame travels through any local switch on the destination network and arrives at the **Destination Server/PC**, which accepts the frame (matching its MAC), verifies the packet (matching its IP), and processes the data!

### Quick Summary Table Across Hops

| Hop / Link | Source IP (L3) | Destination IP (L3) | Source MAC (L2) | Destination MAC (L2) |
| :--- | :--- | :--- | :--- | :--- |
| **Client PC $\rightarrow$ Router 1** | Client IP | Server/PC IP | Client MAC | Router 1 MAC |
| **Router 1 $\rightarrow$ Router 2** | Client IP | Server/PC IP | Router 1 MAC | Router 2 MAC |
| **Router 2 $\rightarrow$ Router 3** | Client IP | Server/PC IP | Router 2 MAC | Router 3 MAC |
| **Router 3 $\rightarrow$ Server/PC** | Client IP | Server/PC IP | Router 3 MAC | Server/PC MAC |