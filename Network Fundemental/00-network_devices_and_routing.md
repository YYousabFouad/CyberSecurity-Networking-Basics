# Network Devices, Routing, and Switching

## Signal Decay, Repeaters, Hubs, and Bridges
- **Signal Decay:** Data crossing a wire decays as it travels.
- **Repeaters:** Regenerate signals.
- **Hub:** Multi-port repeaters.
  - Facilitates scaling communication between additional hosts.
  - Everyone receives everyone else's data.
- **Bridge:** Sits between hub-connected hosts.
  - Bridges only have 2 ports.
  - Bridges learn which hosts are on each side.
  - Contain packets only to their relative networks.

## Switches and Routers
- **Switches:** Combination of Hubs and Bridges.
  - Multiple ports.
  - Learn which hosts are on each port.
  - Facilitate communication within a network.
- **Router:** Facilitates communication between networks.
  - **Network:** Grouping of hosts that require similar connectivity.
  - Provides a traffic control point.
  - Routers learn which networks they are attached to.
  - **Route:** The knowledge of each of these different networks.
  - Routes are stored in a **Routing Table**.
  - **Routing Table:** All networks a router knows about.
  - Routers have IP addresses in the networks they are attached to.

## Gateways, Routing, and Switching
- **Gateway:** Each host's way out of their local network.
  - Routers create the hierarchy in networks and the entire internet.
  - **Routing:** The process of moving data between networks.
  - **Switching:** The process of moving data within networks.