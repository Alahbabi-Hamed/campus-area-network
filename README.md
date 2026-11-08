# AI-Assisted Campus Area Network Design

## Project Overview

This repository contains the full design, configuration, and testing documentation for a Campus Area Network (CAN) built in Cisco Packet Tracer. The network serves three two-story academic buildings and an outdoor common area for a college campus. GitHub Copilot was used as an AI design assistant throughout the planning and configuration process, and every recommendation it provided was validated before implementation.

The finished network supports wired and wireless users across multiple academic departments, centralized servers, a DMZ for public-facing services, inter-VLAN routing, DHCP, OSPF, NAT/PAT, and full Internet connectivity through a simulated ISP.

## Campus Topology

The campus consists of three buildings arranged around a shared outdoor student common area:

- **Building A** (Central Administration and Technology): Houses Computer Science, Business, Administration, IT Sandbox, the campus server room, and the DMZ.
- **Building B** (Science and Engineering): Houses Engineering and Mathematics/Science departments.
- **Building C** (Humanities and Communication): Houses Humanities and Arts/Communication departments.

Each building has two floors with a dedicated switch closet on each floor. All buildings connect through a central core switch (CORE-SW1) located in Building A.

## Network Architecture

The design follows a three-tier hierarchical model:

| Layer | Devices | Role |
|-------|---------|------|
| Core | CORE-SW1 (3560-24PS) | Central L3 switch handling inter-VLAN routing via SVIs, DHCP services, and OSPF |
| Distribution | BLDGB-SW1, BLDGC-SW1 (3560-24PS) | Aggregate traffic from access switches in Buildings B and C via trunk links |
| Access | 7 x 2960-24TT switches | Connect end devices on each floor, one per floor per building plus a server switch |

Building A access switches connect directly to CORE-SW1 because the core switch is physically in that building. Buildings B and C each have a distribution-layer 3560 switch that aggregates their floor switches before connecting to the core.

## Security and Internet Edge

| Device | Role |
|--------|------|
| CAMPUS-FW1 (2911 Router) | Campus firewall. Separates internal traffic from the Internet edge using NAT/PAT. Runs OSPF with CORE-SW1. |
| EDGE-RTR1 (2911 Router) | Campus edge router connecting to the ISP via a public /30 link (203.0.113.0/30). |
| ISP-RTR1 (2911 Router) | Simulated ISP router providing the upstream Internet connection. |

## Addressing Summary

The entire campus operates within the 10.50.0.0/16 private address block, subnetted using VLSM to match actual host requirements per department. 17 VLANs were created covering 8 academic/administrative departments, 3 wireless zones, 3 management zones, a server VLAN, and a DMZ VLAN.

Full addressing details are in [ip-addressing.md](ip-addressing.md).

## Wireless Design

A single SSID ("CampusNet") covers all three buildings and the outdoor area. Each building has access points on both floors, and one outdoor AP serves the common area. Wireless security uses WPA2-PSK with a shared passphrase. Each building gets its own wireless VLAN (111, 112, 113) to keep broadcast domains separated while still presenting one unified network to users.

## Key Protocols and Services

| Protocol/Service | Implementation |
|------------------|----------------|
| Inter-VLAN Routing | Layer 3 SVIs on CORE-SW1 |
| Dynamic Routing | OSPF Area 0 between CORE-SW1 and CAMPUS-FW1 |
| DHCP | 11 pools on CORE-SW1 serving all departmental and wireless VLANs |
| NAT/PAT | Overload NAT on CAMPUS-FW1 translating 10.50.0.0/16 to the outside interface |
| DNS | DNS-SRV1 at 10.50.90.9, referenced by all DHCP pools |
| Trunking | 802.1Q trunks on all inter-switch links with native VLAN 1 |
| Static Routing | Default routes on CORE-SW1, CAMPUS-FW1, EDGE-RTR1; static return route on ISP-RTR1 |

## Repository Contents

| File | Description |
|------|-------------|
| README.md | This file. Project overview and architecture summary. |
| ip-addressing.md | Complete VLSM addressing plan with subnet calculations, VLAN table, server assignments, DHCP pools. |
| copilot-design-log.md | Log of 6 AI-assisted design interactions covering topology, subnetting, VLAN strategy, routing, security, and troubleshooting. |
| verification.md | Testing results including ping tests, DHCP bindings, OSPF adjacency, NAT translations, and traceroute output. |
| campus-network.pkt | Cisco Packet Tracer project file with the completed and tested network. |

## How to Open

1. Install Cisco Packet Tracer 8.2.2 or later.
2. Open `campus-network.pkt`.
3. All device configurations are saved to startup-config.
4. PCs and laptops should retain their DHCP leases. If not, toggle the IP configuration from Static to DHCP on any endpoint to re-acquire an address.
