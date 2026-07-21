# Technical Implementation Notes

[Back to the project overview](../README.md)

This document contains the addressing, configuration and validation detail behind the recruiter-focused project overview.

## IP Addressing

### VLAN Networks

| VLAN | Department | Subnet | HSRP Gateway | Core SW1 | Core SW2 |
| --- | --- | --- | --- | --- | --- |
| 5 | Administration | `10.11.5.0/24` | `10.11.5.1` | `10.11.5.2` | `10.11.5.3` |
| 10 | Doctors | `10.11.10.0/24` | `10.11.10.1` | `10.11.10.2` | `10.11.10.3` |
| 15 | Nurses | `10.11.15.0/24` | `10.11.15.1` | `10.11.15.2` | `10.11.15.3` |
| 20 | Medical Devices | `10.11.20.0/24` | `10.11.20.1` | `10.11.20.2` | `10.11.20.3` |
| 25 | Pharmacy | `10.11.25.0/24` | `10.11.25.1` | `10.11.25.2` | `10.11.25.3` |
| 30 | Servers | `10.11.30.0/24` | `10.11.30.1` | `10.11.30.2` | `10.11.30.3` |

### Internal Transit Networks

| Link | Subnet | Device 1 | Device 2 |
| --- | --- | --- | --- |
| Core SW1 ↔ FW1 inside | `10.11.254.0/30` | Core SW1: `10.11.254.1` | FW1: `10.11.254.2` |
| FW1 outside ↔ EDGE-R1 | `10.11.254.4/30` | FW1: `10.11.254.5` | EDGE-R1: `10.11.254.6` |
| Core SW2 ↔ FW2 inside | `10.11.254.8/30` | Core SW2: `10.11.254.9` | FW2: `10.11.254.10` |
| FW2 outside ↔ EDGE-R2 | `10.11.254.12/30` | FW2: `10.11.254.13` | EDGE-R2: `10.11.254.14` |

### WAN and Public Networks

| Network | Endpoint 1 | Endpoint 2 |
| --- | --- | --- |
| `203.0.113.0/30` | ISP-R1: `203.0.113.1` | EDGE-R1: `203.0.113.2` |
| `203.0.113.4/30` | ISP-R2: `203.0.113.5` | EDGE-R2: `203.0.113.6` |
| `198.51.100.0/24` | ISP-R1: `198.51.100.1`; ISP-R2: `198.51.100.2` | Public server: `198.51.100.10` |

The public server currently uses `198.51.100.1` as its only default gateway.

## Redundancy

### LACP EtherChannel

Core interfaces `Fa0/8` and `Fa0/9` are bundled into `Port-Channel 1`, providing 200 Mbps of aggregate capacity, load balancing and link redundancy.

```text
1      Po1(SU)           LACP   Fa0/8(P) Fa0/9(P)
```

### HSRP Gateway Redundancy

- Core SW1 normally operates as Active with priority `110`.
- Core SW2 normally operates as Standby with priority `100`.
- Preemption is configured, and clients use the `.1` virtual gateway in each VLAN.
- When Core SW1 was made unavailable, Core SW2 became Active for VLANs 5, 10, 15, 20, 25 and 30.
- Clients did not require a gateway change during failover.

![HSRP verification](../diagrams/hsrp-all-vlans-verification.png)

Internal HSRP failover succeeded. End-to-end public-server connectivity did not fully survive the test because the server has no redundant default gateway.

### Dual Perimeter Paths

```text
Core SW1 → FW1 → EDGE-R1 → ISP-R1
Core SW2 → FW2 → EDGE-R2 → ISP-R2
```

Both paths are addressed. Full second-path routing and public-side failover validation remain in progress.

## Routing

### Static, Default and Return Routing

Every Layer 3 device requires a route toward the destination and a return route toward the source.

```text
Hospital PC
→ HSRP virtual gateway
→ multilayer core switch
→ ASA firewall
→ edge router
→ ISP router
→ public web server
```

- Core switches use default routes toward their local firewalls.
- Firewalls use default routes toward their edge routers.
- Edge routers use default routes toward their ISP routers.
- Firewalls, edge routers and ISP routers use routes toward `10.11.0.0/16`.
- More-specific routes take priority over default routes.

The implementation demonstrates directly connected, static, default and return routes; next-hop reachability; summarisation; administrative distance; and forward/return-path troubleshooting.

### Dynamic Routing Development

The target hybrid design uses OSPF internally and eBGP at the ISP boundary. Static routes remain during migration and testing.

| Device or domain | Autonomous system |
| --- | --- |
| Hospital network | `AS 65010` |
| ISP-R1 | `AS 65001` |
| ISP-R2 | `AS 65002` |

#### eBGP Progress

Completed on the first path:

- `EDGE-R1` and `ISP-R1` established a neighbour relationship between `203.0.113.2` and `203.0.113.1`.
- `EDGE-R1` advertises `10.11.0.0/16`.
- `ISP-R1` advertises `198.51.100.0/24`.
- Neighbour state and learned routes were checked with `show ip bgp summary`, `show ip bgp` and `show ip route bgp`.

Remaining work:

- Establish eBGP between `EDGE-R2` and `ISP-R2`.
- Verify the second path, configure path preference and test dual-ISP failover.
- Document the final BGP routing tables.

#### OSPF Progress

OSPF is being studied and prepared; it is not operational in the lab.

Planned scope:

- Advertise hospital VLAN and transit networks.
- Form adjacencies between internal Layer 3 devices.
- Verify neighbours, areas, LSAs and learned routes.
- Replace selected static routes only after successful validation.
- Test failover with the dual-core and dual-perimeter design.

## Firewall and Security Controls

### ASA ICMP Inspection

The following policy is applied on both firewalls:

```cisco
policy-map global_policy
 class inspection_default
  inspect icmp
 exit
exit

service-policy global_policy global
```

This allows each firewall to track ICMP requests and permit matching replies statefully.

### Extended ACLs

The named ACLs are applied inbound on the relevant SVIs. They must exist on both core switches because either switch can become the HSRP Active gateway.

#### Pharmacy VLAN

```cisco
ip access-list extended PHARMACY-IN
 deny ip 10.11.25.0 0.0.0.255 10.11.5.0 0.0.0.255
 permit ip any any
exit

interface vlan 25
 ip access-group PHARMACY-IN in
```

Validation: Pharmacy-to-Administration traffic was denied, Pharmacy-to-public-server traffic was permitted, and ACL counters increased as expected.

#### Server VLAN

```cisco
ip access-list extended SERVERS-IN
 deny ip 10.11.30.0 0.0.0.255 10.11.5.0 0.0.0.255
 permit ip any any
exit

interface vlan 30
 ip access-group SERVERS-IN in
```

This blocks Server VLAN traffic toward Administration while permitting other server traffic.

### Access-Switch Port Security

```cisco
interface fastethernet0/1
 switchport mode access
 switchport access vlan 25
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

The Pharmacy access port allows one sticky MAC address. Restrict mode drops unauthorised frames without shutting down the port.

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Security Violation Count   : 1
```

## Validation

Normal-state testing confirmed:

- Public-server reachability from PCs 1–15
- Inter-VLAN routing and HSRP virtual gateways
- Static, default and return routing
- ASA ICMP inspection
- ACL deny and permit behaviour
- Port-security restrict behaviour
- HSRP failover from Core SW1 to Core SW2
- First-path eBGP neighbour establishment and route exchange

Full public-side and dual-ISP failover remain incomplete.

## Troubleshooting Method

```text
Physical connection
→ interface state
→ IP address and subnet
→ default gateway
→ forward route
→ next-hop reachability
→ return route
→ firewall or ACL policy
→ NAT if required
```

The last successful hop shows where connectivity still works; the first failed hop identifies the next device, link or policy boundary to inspect. Validation used `ping`, `traceroute`, `show ip interface brief`, `show ip route`, `show route`, `show standby brief`, `show access-lists` and `show port-security interface`.

## Planned Enhancements

- Complete second-path eBGP, path preference and dual-ISP failover testing.
- Implement and validate OSPF before replacing selected static routes.
- Add public-side gateway redundancy, IP SLA and object tracking.
- Expand ACL and port-security coverage.
- Add DHCP/DHCP relay, then enable DHCP snooping.
- Add a Voice VLAN, IP phones and QoS.
- Add SSH management, NTP, Syslog, SNMP and realistic NAT.
