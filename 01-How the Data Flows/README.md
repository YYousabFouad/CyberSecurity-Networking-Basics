# How the Data Flows

This folder explains how data flows from a PC (host/client) across the local network until it reaches the router (default gateway) and moves between different networks.

## What You Will Learn Here

1. **Hosts, Clients, Servers & IPv4 Identity** — How devices are identified with 32-bit IPv4 addresses (source and destination) and grouped into networks and subnets.
2. **Layer 1 & Layer 2 Devices (Repeaters, Hubs, Bridges, Switches)** — How signals travel across the wire and how switches forward traffic within the local network.
3. **Routers & Default Gateways** — How traffic leaves the local subnet through a gateway and gets routed between separate networks using routing tables.

## Contents

- [`00-network_fundamentals_study_notes_part1.md`](./00-network_fundamentals_study_notes_part1.md) — Hosts, clients/servers, IPv4 addressing, networks, and assessment log.
- [`00-network_devices_and_routing.md`](./00-network_devices_and_routing.md) — Signal decay, repeaters, hubs, bridges, switches, routers, gateways, switching vs. routing.

> **Hands-On Lab:** The combined topology lab ([`two-subnets-router-lab.pkt`](../two-subnets-router-lab.pkt)) was moved to the repository root because it brings together host addressing, local switching, and cross-network routing across all sections.
