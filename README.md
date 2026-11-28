# CyberSecurity-Networking-Basics

Foundational networking study notes and Cisco Packet Tracer labs as a first step toward cybersecurity.

## Purpose

Strong cybersecurity skills start with understanding how data actually moves across a network. This repository documents core networking concepts, CLI verification, and hands-on Cisco Packet Tracer topologies at a foundational level.

## Topics Covered

- **Networking Fundamentals**: Hosts, clients, servers, IPv4 addressing (32-bit / 4 octets, binary-to-decimal conversion), networks, switching, routing, the OSI Model (MAC vs. IP, ARP, ports), subnetting, and DNS.
- **Cisco Packet Tracer Labs**: Topologies, addressing, basic connectivity verification, and multi-subnet routing.

## Repository Contents

- [`Packet Traveling.md`](./Packet%20Traveling.md) — Root-level master summary & fast-revision guide (*How Data Moves Through the Internet*) tying together MAC, ARP, and Routing tables, DNS + HTTP web browsing flow, and quick checks across all six topic folders.
- [`two-subnets-router-lab.pkt`](./two-subnets-router-lab.pkt) — Combined Cisco Packet Tracer lab applying host addressing, local switching, and cross-network routing across two subnets.
- [`01-How the Data Flows/`](./01-How%20the%20Data%20Flows/) — Explains how data flows from a PC across the local network until it reaches the router:
  - [`00-network_fundamentals_study_notes_part1.md`](./01-How%20the%20Data%20Flows/00-network_fundamentals_study_notes_part1.md) — Study notes Part 1 (Hosts, Clients/Servers, IP Addresses, Networks).
  - [`00-network_devices_and_routing.md`](./01-How%20the%20Data%20Flows/00-network_devices_and_routing.md) — Network devices, switching, gateways, and routing.
- [`02-OSI Model/`](./02-OSI%20Model/) — Covers the OSI Model (Layers 1–7), MAC vs. IP addressing, ARP, ports, and encapsulation/decapsulation:
  - [`00-osi_model_and_addressing_part1.md`](./02-OSI%20Model/00-osi_model_and_addressing_part1.md) — OSI Model & Network Addressing Part 1 (Layers 1–3, MAC vs. IP, ARP).
  - [`01-osi_model_part2.md`](./02-OSI%20Model/01-osi_model_part2.md) — OSI Model Part 2 (Layers 4–7, TCP/UDP ports, stream isolation, encapsulation/decapsulation).
  - [`the forgotten layers of osi model.md`](./02-OSI%20Model/the%20forgotten%20layers%20of%20osi%20model.md) — Layers 5–7 (Session cookies, Presentation bit encoding/ASCII, Application commands), HTTP vs. FTP, and abstraction layers.
- [`03-Everything Hosts Do/`](./03-Everything%20Hosts%20Do/) — Covers direct and router-connected host communication, subnetting math, ARP resolution, DNS, and L2 hop-by-hop vs. L3 end-to-end delivery:
  - [`direct_host_connections_in_details.md`](./03-Everything%20Hosts%20Do/direct_host_connections_in_details.md) — Direct host connections, subnetting, ARP, and DNS.
  - [`hosts_connected_through_a_router.md`](./03-Everything%20Hosts%20Do/hosts_connected_through_a_router.md) — Hosts connected through single/multiple routers and L2 vs. L3 packet flow.
- [`04-How Switches Work/`](./04-How%20Switches%20Work/) — Covers Layer 2 switching, MAC address tables, learning/forwarding/flooding, unicast vs. broadcast frames, VLANs, and access vs. trunk ports:
  - [`switches_part_1.md`](./04-How%20Switches%20Work/switches_part_1.md) — Core switch concepts, MAC address table, and the three switch actions.
  - [`switches_part_2.md`](./04-How%20Switches%20Work/switches_part_2.md) — Unicast vs. broadcast, VLANs, multi-switch operation, and 802.1Q trunking.
- [`05-Everything Routers Do/`](./05-Everything%20Routers%20Do/) — Covers Layer 3 routing fundamentals, network nodes, RFCs, routing tables, route population methods, dynamic routing protocols, end-to-end packet forwarding, router hierarchy, and route summarization:
  - [`everything_routers_do_part1.md`](./05-Everything%20Routers%20Do/everything_routers_do_part1.md) — Router core functions, network nodes, RFCs, routing table components, directly connected/static/dynamic routes, and RIP/OSPF/EIGRP/IS-IS/BGP.
  - [`everything_routers_do_part_2.md`](./05-Everything%20Routers%20Do/everything_routers_do_part_2.md) — Router ARP tables, request/response forwarding walkthrough across routers, L2 vs. L3 header behavior, and what the Internet is.
  - [`router_hierarchy_route_summarization_notes.md`](./05-Everything%20Routers%20Do/router_hierarchy_route_summarization_notes.md) — Router hierarchy benefits, route summarization, Longest Prefix Match & tie-breakers, and default routes (`0.0.0.0/0`).
- [`06-Network Protocols/`](./06-Network%20Protocols/) — Covers core network and application protocols (`ARP`, `FTP`, `SMTP`, `HTTP`, `SSL/TLS`, `HTTPS`, `DNS`, `DHCP`) and required host configuration items:
  - [`network_protocols_study_notes.md`](./06-Network%20Protocols/network_protocols_study_notes.md) — Protocol definitions, how each works, ports/RFC references, the 4 host connectivity parameters, and quick-reference summary table.
  - [`How to send HTTP request using Telnet manually.md`](./06-Network%20Protocols/How%20to%20send%20HTTP%20request%20using%20Telnet%20manually.md) — HTTP vs. HTML, sending a manual HTTP `GET` request over TCP Port 80 via Telnet, and how browsers automate the process.

## Packet Tracer Lab: `two-subnets-router-lab.pkt`

This hands-on simulation ties together hosts, local switches, and routing across two independent IP networks:

```text
  [ Subnet 1: 192.168.1.0/24 ]                     [ Subnet 2: 192.168.2.0/24 ]

  PC 1 (.1.10) --\                                        /--- Server (.2.10)
                  [ Switch 1 ] --- [ Router ] --- [ Switch 2 ]
  PC 2 (.1.11) --/                (.1.1 | .2.1)
```

### How It Works
- **Switches Within Each Network (Same IP Subnet):** Switches connect hosts that share the same IP network and subnet mask so they can communicate locally using Layer 2 MAC forwarding. **Switch 1** connects the two client PCs (`192.168.1.10` and `192.168.1.11`), while an independent **Switch 2** connects the Server (`192.168.2.10`) on its own network.
- **Router Between the Two Networks (Cross-Network Routing):** Because the PCs (`192.168.1.0/24`) and the Server (`192.168.2.0/24`) belong to two separate IP networks, switches alone cannot bridge them. Both switches connect to separate interfaces on the **Router**, which acts as each subnet's **default gateway** (`192.168.1.1` and `192.168.2.1`) and forwards packets between the two networks.
- **Applied Skills & Verification:** Configured IPv4 addresses, `255.255.255.0` subnet masks, and default gateways on the hosts and router interfaces, then used `ping` to verify end-to-end connectivity between the PCs and the Server.

| Network | Device | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Client LAN** (`192.168.1.0/24`) | Router (PCs Interface) | `192.168.1.1` | `255.255.255.0` | *N/A* |
| **Client LAN** (`192.168.1.0/24`) | PC 1 | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` |
| **Client LAN** (`192.168.1.0/24`) | PC 2 | `192.168.1.11` | `255.255.255.0` | `192.168.1.1` |
| **Server LAN** (`192.168.2.0/24`) | Router (Server Interface) | `192.168.2.1` | `255.255.255.0` | *N/A* |
| **Server LAN** (`192.168.2.0/24`) | Server | `192.168.2.10` | `255.255.255.0` | `192.168.2.1` |

## Tools

- **Cisco Packet Tracer** (`.pkt` / `.pka` lab topologies)
- **Markdown** (`.md` study notes & lab write-ups)
