# Hospital Network Lab

> **Status:** Active development — core switching, routing, redundancy and initial security controls implemented.

A Cisco Packet Tracer hospital network built to demonstrate practical network segmentation, resilient gateway design, perimeter routing, security controls and structured troubleshooting.

[Open the Packet Tracer lab](hospital.pkt) · [Read the technical implementation notes](docs/technical-notes.md) · [View the IP addressing plan](diagrams/hospital-network-ip-addressing-plan-v2.0.png)

## Project at a Glance

| Area | What the lab demonstrates | Status |
| --- | --- | --- |
| Switching | Six VLANs, Layer 3 switching and LACP EtherChannel | ✅ Implemented and tested |
| Resilience | HSRP gateways and dual firewall, edge-router and ISP paths | 🟡 Internal failover validated |
| Routing | Static/default/return routes and first-path eBGP | 🟡 eBGP expansion in progress |
| Security | ASA ICMP inspection, extended ACLs and sticky-MAC port security | ✅ Implemented and tested |
| Operations | Full addressing plan, hop-by-hop troubleshooting and validation | ✅ Documented and tested |

## Logical Topology

![Hospital Network Logical Topology v1.4](diagrams/Hospital%20Network%20Logical%20Topology%20v1.4.png)

The design uses two multilayer core switches, six departmental VLANs, two ASA firewalls, two hospital edge routers, two simulated ISP routers and a public web server.

```text
Hospital VLANs → HSRP core pair → ASA firewalls → edge routers → ISP routers → public server
```

## Key Results

- PCs 1–15 reached the public web server during normal operation.
- HSRP failover from Core SW1 to Core SW2 succeeded across all six VLANs without client gateway changes.
- Two core links operate as one LACP EtherChannel for additional bandwidth and link redundancy.
- Pharmacy and Server VLAN ACLs blocked selected Administration traffic while permitted traffic continued to pass.
- Pharmacy access-port security learned one sticky MAC and recorded a violation from a second device in restrict mode.
- eBGP peering between `EDGE-R1` (`AS 65010`) and `ISP-R1` (`AS 65001`) exchanged the hospital and public routes.

## VLAN Plan

| VLAN | Department | Subnet | HSRP Gateway |
| --- | --- | --- | --- |
| 5 | Administration | `10.11.5.0/24` | `10.11.5.1` |
| 10 | Doctors | `10.11.10.0/24` | `10.11.10.1` |
| 15 | Nurses | `10.11.15.0/24` | `10.11.15.1` |
| 20 | Medical Devices | `10.11.20.0/24` | `10.11.20.1` |
| 25 | Pharmacy | `10.11.25.0/24` | `10.11.25.1` |
| 30 | Servers | `10.11.30.0/24` | `10.11.30.1` |

## Current Development

| Item | Status |
| --- | --- |
| Second eBGP path between `EDGE-R2` and `ISP-R2` | 🟡 In progress |
| BGP path preference and dual-ISP failover testing | 🟡 In progress |
| OSPF internal routing | 🟠 Planned and being studied |
| Public-side gateway redundancy | 🟠 Planned |

> Full public-side failover is not yet complete because the public server uses ISP-R1 as its single default gateway. OSPF is not operational, and the second eBGP path has not yet been validated.

## Repository Guide

| Resource | Description |
| --- | --- |
| [`hospital.pkt`](hospital.pkt) | Cisco Packet Tracer lab file |
| [`docs/technical-notes.md`](docs/technical-notes.md) | Addressing, configurations, routing progress and validation evidence |
| [`Hospital Network Logical Topology v1.4`](diagrams/Hospital%20Network%20Logical%20Topology%20v1.4.png) | Latest logical topology diagram |
| [`hospital-network-ip-addressing-plan-v2.0.png`](diagrams/hospital-network-ip-addressing-plan-v2.0.png) | Transit, WAN and public-network addressing |
| [`hospital-network-vlan-ip-plan.png`](diagrams/hospital-network-vlan-ip-plan.png) | VLAN addressing overview |

## Next Steps

- Complete and validate eBGP on the second ISP path.
- Implement OSPF internally, then replace selected static routes only after verification.
- Add BGP path preference, public-side gateway redundancy and full dual-path failover testing.
- Expand access security, management, monitoring and QoS controls.

## Topology Progress

- `v1.0` — Initial VLAN topology
- `v1.2` — Dual core switches and EtherChannel
- `v1.3` — HSRP gateway redundancy across all VLANs
- `v1.4` — Dual firewalls, edge routers, ISP links and interface addressing
