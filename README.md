# CyberSecurity-Networking-Basics

Foundational networking study notes and Cisco Packet Tracer labs as a first step toward cybersecurity.

## Purpose

Strong cybersecurity skills start with understanding how data actually moves across a network. This repository documents core networking concepts, CLI verification, and hands-on Cisco Packet Tracer topologies at a foundational level.

## Topics Covered

- **Networking Fundamentals**: Hosts, clients, servers, IPv4 addressing (32-bit / 4 octets, binary-to-decimal conversion), networks, switching, routing, the OSI Model (MAC vs. IP, ARP, ports), subnetting, and DNS.
- **Cisco Packet Tracer Labs**: Topologies, addressing, basic connectivity verification, and multi-subnet routing.

## Repository Contents

- [`01-How the Data Flows/`](./01-How%20the%20Data%20Flows/) — Explains how data flows from a PC across the local network until it reaches the router:
  - [`00-network_fundamentals_study_notes_part1.md`](./01-How%20the%20Data%20Flows/00-network_fundamentals_study_notes_part1.md) — Study notes Part 1 (Hosts, Clients/Servers, IP Addresses, Networks).
  - [`00-network_devices_and_routing.md`](./01-How%20the%20Data%20Flows/00-network_devices_and_routing.md) — Network devices, switching, gateways, and routing.
  - [`two-subnets-router-lab.pkt`](./01-How%20the%20Data%20Flows/two-subnets-router-lab.pkt) — Cisco Packet Tracer lab connecting two subnets via a router.
- [`02-OSI Model/`](./02-OSI%20Model/) — Covers the OSI Model (Layers 1–7), MAC vs. IP addressing, ARP, ports, and encapsulation/decapsulation:
  - [`00-osi_model_and_addressing_part1.md`](./02-OSI%20Model/00-osi_model_and_addressing_part1.md) — OSI Model & Network Addressing Part 1 (Layers 1–3, MAC vs. IP, ARP).
  - [`01-osi_model_part2.md`](./02-OSI%20Model/01-osi_model_part2.md) — OSI Model Part 2 (Layers 4–7, TCP/UDP ports, stream isolation, encapsulation/decapsulation).
- [`03-Everything Hosts Do/`](./03-Everything%20Hosts%20Do/) — Covers direct and router-connected host communication, subnetting math, ARP resolution, DNS, and L2 hop-by-hop vs. L3 end-to-end delivery:
  - [`direct_host_connections_in_details.md`](./03-Everything%20Hosts%20Do/direct_host_connections_in_details.md) — Direct host connections, subnetting, ARP, and DNS.
  - [`hosts_connected_through_a_router.md`](./03-Everything%20Hosts%20Do/hosts_connected_through_a_router.md) — Hosts connected through single/multiple routers and L2 vs. L3 packet flow.
- [`04-How Switches Work/`](./04-How%20Switches%20Work/) — Covers Layer 2 switching, MAC address tables, learning/forwarding/flooding, unicast vs. broadcast frames, VLANs, and access vs. trunk ports:
  - [`switches_part_1.md`](./04-How%20Switches%20Work/switches_part_1.md) — Core switch concepts, MAC address table, and the three switch actions.
  - [`switches_part_2.md`](./04-How%20Switches%20Work/switches_part_2.md) — Unicast vs. broadcast, VLANs, multi-switch operation, and 802.1Q trunking.
- [`05-Everything Routers Do/`](./05-Everything%20Routers%20Do/) — Covers Layer 3 routing fundamentals, network nodes, RFCs, routing tables, route population methods, dynamic routing protocols, and end-to-end packet forwarding:
  - [`everything_routers_do_part1.md`](./05-Everything%20Routers%20Do/everything_routers_do_part1.md) — Router core functions, network nodes, RFCs, routing table components, directly connected/static/dynamic routes, and RIP/OSPF/EIGRP/IS-IS/BGP.
  - [`everything_routers_do_part_2.md`](./05-Everything%20Routers%20Do/everything_routers_do_part_2.md) — Router ARP tables, request/response forwarding walkthrough across routers, L2 vs. L3 header behavior, and what the Internet is.

## Tools

- **Cisco Packet Tracer** (`.pkt` / `.pka` lab topologies)
- **Markdown** (`.md` study notes & lab write-ups)
