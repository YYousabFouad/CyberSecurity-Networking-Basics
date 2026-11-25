# Network Fundamentals

## Hosts, Clients, and Servers
- **Host:** Any device that sends or receives traffic.
  - **Examples:** Computers, laptops, phones, printers, servers, cloud servers, and IoT devices.
- Hosts can act as a **client** or **server** (relative to a specific communication).
  - **Client:** Initiates (sends) a request.
  - **Server:** Responds with data.
- **Communication Examples:**
  - Computer (Client) $\xrightarrow{\text{send}}$ Server $\xrightarrow{\text{respond}}$ Computer (Client)
  - Client $\xrightarrow{\text{send}}$ File Server $\xrightarrow{\text{respond}}$ Server
- **Server Definition:** Computers with software installed that respond to specific requests.

## IP Addresses
- **IP Address:** The identity of each host.
- Every traffic flow has **2 IP addresses**:
  1. **Source IP Address** (client)
  2. **Destination IP Address** (server)
- IP addresses are **32 bits** long.
  - **Bit:** $1$ or $0$.
  - Represented as **4 octets** of values from $0$ to $255$.
- **Binary-to-Decimal Conversion Example:**
  - Binary stream: `10001000 00010110 00010001 01100010`
  - Dotted octets: `10001000` . `00010110` . `00010001` . `01100010`
  - Decimal equivalent: `136` . `22` . `17` . `98`
- IP addresses are **hierarchically assigned**.

## Networks and the Internet
- **Network:** What transports traffic between hosts.
  - Anytime 2 hosts are connected, you have a network.
  - A logical grouping of hosts that require similar connectivity.
  - Networks can contain other networks, sometimes called **sub-networks (subnets)**.
- **Internet:** A group of interconnected networks.

---

## Practice & Assessment Log
- **Quiz:** Network & Subnets Fundamentals Check — **10/10**
- **Verified Concepts:** Client/Server roles, IPv4 32-bit/Octet structure, Hierarchical IP allocation, Network vs. Subnet segmentation, and Router role.
