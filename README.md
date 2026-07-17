# Hospital Network Lab

> **Status: Work in Progress**

A Cisco Packet Tracer hospital network demonstrating VLAN segmentation, Layer 3 switching, subnetting, IP addressing, gateway redundancy, link aggregation and redundant perimeter network design.

## Implemented Upgrades

| Upgrade                             | Status                          | Purpose                                                         |
| ----------------------------------- | ------------------------------- | --------------------------------------------------------------- |
| 🔗 **EtherChannel using LACP**      | ✅ Implemented                  | Bundles two core links for increased bandwidth and redundancy   |
| 🛡️ **HSRP Gateway Redundancy**      | ✅ Implemented across all VLANs | Provides automatic default-gateway failover                     |
| 🌐 **Dual WAN and Perimeter Paths** | 🟡 In progress                  | Introduces dual ISP, edge-router and firewall paths             |
| 📋 **Full IP Addressing Plan**      | ✅ Documented                   | Maps VLAN, transit, WAN and simulated public-network addressing |

## Network Overview

| VLAN | Department      | Subnet          | Virtual Gateway |
| ---- | --------------- | --------------- | --------------- |
| 5    | Administration  | `10.11.5.0/24`  | `10.11.5.1`     |
| 10   | Doctors         | `10.11.10.0/24` | `10.11.10.1`    |
| 15   | Nurses          | `10.11.15.0/24` | `10.11.15.1`    |
| 20   | Medical Devices | `10.11.20.0/24` | `10.11.20.1`    |
| 25   | Pharmacy        | `10.11.25.0/24` | `10.11.25.1`    |
| 30   | Servers         | `10.11.30.0/24` | `10.11.30.1`    |

## Architecture

- Dual Cisco multilayer core switches
- Six departmental VLANs
- Dedicated access switches
- Inter-VLAN routing
- EtherChannel core redundancy
- HSRP gateway redundancy
- Dual Cisco edge routers
- Dual Cisco ASA firewalls
- Dual simulated ISP routers
- Simulated public internet network
- Internal server VLAN
- External public web server
- `/30` point-to-point transit networks
- Static and default routing under development

## Logical Topology

![Hospital Network Logical Topology](diagrams/Hospital%20Network%20Logical%20Topology%20v1.4.png)

## IP Addressing Plan

![Hospital Network IP Addressing Plan](diagrams/hospital-network-ip-addressing-plan-v2.0.png)

## VLAN and IP Addressing Plan

![Hospital Network VLAN and IP Plan](diagrams/hospital-network-vlan-ip-plan.png)

## Internal Transit Networks

| Link                  | Subnet            | Device 1                | Device 2                |
| --------------------- | ----------------- | ----------------------- | ----------------------- |
| Core SW1 ↔ FW1 inside | `10.11.254.0/30`  | Core SW1: `10.11.254.1` | FW1: `10.11.254.2`      |
| FW1 outside ↔ EDGE-R1 | `10.11.254.4/30`  | FW1: `10.11.254.5`      | EDGE-R1: `10.11.254.6`  |
| Core SW2 ↔ FW2 inside | `10.11.254.8/30`  | Core SW2: `10.11.254.9` | FW2: `10.11.254.10`     |
| FW2 outside ↔ EDGE-R2 | `10.11.254.12/30` | FW2: `10.11.254.13`     | EDGE-R2: `10.11.254.14` |

## WAN Addressing

| Link       | Subnet           | ISP Router                  | Hospital Edge Router   |
| ---------- | ---------------- | --------------------------- | ---------------------- |
| WAN Link 1 | `203.0.113.0/30` | ISP Router 1: `203.0.113.1` | EDGE-R1: `203.0.113.2` |
| WAN Link 2 | `203.0.113.4/30` | ISP Router 2: `203.0.113.5` | EDGE-R2: `203.0.113.6` |

## Simulated Public Internet

| Device                        | Address            |
| ----------------------------- | ------------------ |
| ISP Router 1 public interface | `198.51.100.1/24`  |
| ISP Router 2 public interface | `198.51.100.2/24`  |
| Public web server             | `198.51.100.10/24` |
| Public server default gateway | `198.51.100.1`     |

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

HSRP provides shared virtual default gateways across both multilayer core switches.

- Core SW1 normally operates as Active with priority `110`
- Core SW2 normally operates as Standby with priority `100`
- Preemption allows Core SW1 to reclaim the Active role after recovery
- End devices continue using the same `.1` virtual gateway during failover

#### HSRP Addressing

| VLAN | Virtual Gateway | Core SW1 SVI | Core SW2 SVI |
| ---- | --------------- | ------------ | ------------ |
| 5    | `10.11.5.1`     | `10.11.5.2`  | `10.11.5.3`  |
| 10   | `10.11.10.1`    | `10.11.10.2` | `10.11.10.3` |
| 15   | `10.11.15.1`    | `10.11.15.2` | `10.11.15.3` |
| 20   | `10.11.20.1`    | `10.11.20.2` | `10.11.20.3` |
| 25   | `10.11.25.1`    | `10.11.25.2` | `10.11.25.3` |
| 30   | `10.11.30.1`    | `10.11.30.2` | `10.11.30.3` |

#### Verification

![HSRP All VLANs Verification](diagrams/hsrp-all-vlans-verification.png)

Core SW1 is Active across all VLANs, while Core SW2 remains ready to take over automatically if Core SW1 becomes unavailable.

### 🌐 Dual WAN and Perimeter Expansion

The network now includes two independent perimeter paths:

```text
Core SW1 → FW1 → EDGE-R1 → ISP Router 1
Core SW2 → FW2 → EDGE-R2 → ISP Router 2
```

This phase introduces:

- Routed core-to-firewall interfaces
- Separate firewall inside and outside networks
- Dual hospital edge routers
- Separate `/30` WAN links
- Simulated public internet connectivity
- External public web-server testing

Static routes, default routes, firewall policies, NAT and end-to-end failover testing remain in progress.

## Topology Progress

- `v1.0` — Initial VLAN topology
- `v1.2` — Dual core switches and EtherChannel
- `v1.3` — HSRP gateway redundancy across all VLANs
- `v1.4` — Dual firewalls, edge routers, ISP links and full interface addressing

## Repository Contents

```text
hospital-network-lab/
├── diagrams/
│   ├── Hospital Network Logical Topology v1.0.png
│   ├── Hospital Network Logical Topology v1.2.png
│   ├── Hospital Network Logical Topology v1.3.png
│   ├── Hospital Network Logical Topology v1.4.png
│   ├── hospital-network-ip-addressing-plan-v2.0.png
│   ├── hospital-network-vlan-ip-plan.png
│   └── hsrp-all-vlans-verification.png
├── hospital.pkt
├── .gitignore
└── README.md
```
