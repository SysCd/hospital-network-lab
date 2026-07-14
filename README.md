# Hospital Network Lab

> **Status: Work in Progress**

A Cisco Packet Tracer hospital network demonstrating VLAN segmentation, Layer 3 switching, subnetting, IP addressing, redundancy and enterprise network design.

## Implemented Upgrades

| Upgrade                        | Status                | Purpose                                                       |
| ------------------------------ | --------------------- | ------------------------------------------------------------- |
| 🔗 **EtherChannel using LACP** | ✅ Implemented        | Bundles two core links for increased bandwidth and redundancy |
| 🛡️ **HSRP Gateway Redundancy** | ✅ VLAN 5 implemented | Provides automatic default-gateway failover                   |

## Network Overview

| VLAN | Department      | Subnet        | Gateway    |
| ---- | --------------- | ------------- | ---------- |
| 5    | Administration  | 10.11.5.0/24  | 10.11.5.1  |
| 10   | Doctors         | 10.11.10.0/24 | 10.11.10.1 |
| 15   | Nurses          | 10.11.15.0/24 | 10.11.15.1 |
| 20   | Medical Devices | 10.11.20.0/24 | 10.11.20.1 |
| 25   | Pharmacy        | 10.11.25.0/24 | 10.11.25.1 |
| 30   | Servers         | 10.11.30.0/24 | 10.11.30.1 |

## Architecture

- Dual Cisco multilayer core switches
- Six departmental VLANs
- Dedicated access switches
- Inter-VLAN routing
- EtherChannel core redundancy
- HSRP gateway redundancy
- Hospital edge router
- ISP router and simulated public network
- Internal server VLAN
- External public web server

## Logical Topology

![Hospital Network Logical Topology](diagrams/Hospital%20Network%20Logical%20Topology%20v1.3.png)

## VLAN and IP Addressing Plan

![Hospital Network VLAN and IP Plan](diagrams/hospital-network-vlan-ip-plan.png)

## Network Upgrades

### 🔗 EtherChannel using LACP

Two FastEthernet links between `HOSP-CORE-SW1` and `HOSP-CORE-SW2` are bundled into `Port-Channel 1`.

- Interfaces: `Fa0/8` and `Fa0/9`
- Protocol: LACP
- Aggregate capacity: 200 Mbps
- Provides link redundancy and load balancing

#### Verification

```text
1      Po1(SU)           LACP   Fa0/8(P) Fa0/9(P)
```

- `S` — Layer 2 EtherChannel
- `U` — Port-Channel is operational
- `P` — Interface is successfully bundled

### 🛡️ HSRP Gateway Redundancy

HSRP provides a shared virtual default gateway across both multilayer core switches.

#### VLAN 5 Addressing

| Role                 | Address     |
| -------------------- | ----------- |
| HSRP virtual gateway | `10.11.5.1` |
| Core SW1 SVI         | `10.11.5.2` |
| Core SW2 SVI         | `10.11.5.3` |

- Core SW1 normally operates as Active
- Core SW2 normally operates as Standby
- Automatic failover occurs if the Active gateway becomes unavailable
- PCs continue using `10.11.5.1` as their default gateway
- The same `.1`, `.2`, `.3` addressing pattern is planned for the remaining VLANs

## Topology Progress

- `v1.0` — Initial VLAN topology
- `v1.2` — Dual core switches and EtherChannel
- `v1.3` — HSRP gateway redundancy

## Repository Contents

```text
hospital-network-lab/
├── diagrams/
│   ├── Hospital Network Logical Topology v1.0.png
│   ├── Hospital Network Logical Topology v1.2.png
│   ├── Hospital Network Logical Topology v1.3.png
│   └── hospital-network-vlan-ip-plan.png
├── hospital.pkt
├── .gitignore
└── README.md
```
