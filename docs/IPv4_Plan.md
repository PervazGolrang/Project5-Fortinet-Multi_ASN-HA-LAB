# IP Address Plan

This is a complete addressing scheme that use structured RFC 1918 and documentation prefixes for clarity, /32 for loopbacks, /31 for P2P links where supported, and larger subnets used for branches and DMZ.

---

## Global Address Blocks

| Block        | Range           | Purpose                                |
|--------------|-----------------|----------------------------------------|
| Loopbacks    | 10.255.0.0/24   | Router-IDs and BGP peering             |
| PE-CE Links  | 192.168.0.0/24  | Service provider customer-facing links |
| PE Backbone  | 172.16.0.0/30   | PE1 ↔ PE2 via WAN-EM                   |
| Branch A LAN | 10.10.0.0/16    | Oslo internal networks                 |
| Branch B LAN | 10.20.0.0/16    | Bergen internal networks               |
| Hub LAN      | 10.100.0.0/16   | Central site and DMZ                   |
| VPN Tunnels  | 172.31.0.0/24   | IPsec overlay addressing               |
| **Home Network** | **192.168.40.0/24** | **Management (my home router)**            |

---

## Loopback Addresses

| Device   | Loopback0     | Router-ID  | ASN   |
|----------|---------------|------------|-------|
| PE1      | 10.255.0.1/32 | 10.255.0.1 | 2000  |
| PE2      | 10.255.0.2/32 | 10.255.0.2 | 2000  |
| FGT-A    | 10.255.1.1/32 | 10.255.1.1 | 65001 |
| R1-A     | 10.255.1.2/32 | 10.255.1.2 | 65001 |
| FGT-HUB1 | 10.255.3.1/32 | 10.255.3.1 | 3481  |
| FGT-HUB2 | 10.255.3.2/32 | 10.255.3.2 | 3481  |
| R2-H     | 10.255.3.3/32 | 10.255.3.3 | 3481  |
| FGT-B    | 10.255.2.1/32 | 10.255.2.1 | 65002 |
| R3-B     | 10.255.2.2/32 | 10.255.2.2 | 65002 |

---

## Point-to-Point Links

### Service Provider Backbone

| Link         | Interface A | Interface B | IP A       | IP B       | Subnet        |
|--------------|-------------|-------------|------------|------------|---------------|
| PE1 ↔ WAN-EM | PE1 eth0    | WAN-EM e0   | 172.16.0.1 | 172.16.0.2 | 172.16.0.0/30 |
| PE2 ↔ WAN-EM | PE2 eth0    | WAN-EM e1   | 172.16.0.5 | 172.16.0.6 | 172.16.0.4/30 |

### PE-CE Links (BGP Peering)

| Link           | Interface A | Interface B    | IP A         | IP B         | Subnet          |
|----------------|-------------|----------------|--------------|--------------|-----------------|
| PE1 ↔ FGT-A    | PE1 eth1    | FGT-A port1    | 192.168.0.1  | 192.168.0.2  | 192.168.0.0/30  |
| PE2 ↔ FGT-A    | PE1 eth1    | FGT-A port2    | 192.168.0.5  | 192.168.0.6  | 192.168.0.4/30  |
| PE1 ↔ FGT-HUB1 | PE1 eth2    | FGT-HUB1 port1 | 192.168.0.9  | 192.168.0.10 | 192.168.0.8/30  |
| PE2 ↔ FGT-HUB1 | PE2 eth2    | FGT-HUB1 port2 | 192.168.0.13 | 192.168.0.14 | 192.168.0.12/30 |
| PE1 ↔ FGT-HUB2 | PE1 eth3    | FGT-HUB2 port1 | 192.168.0.17 | 192.168.0.18 | 192.168.0.16/30 |
| PE2 ↔ FGT-HUB2 | PE2 eth3    | FGT-HUB2 port2 | 192.168.0.21 | 192.168.0.22 | 192.168.0.20/30 |
| PE1 ↔ FGT-B    | PE1 eth4    | FGT-B port1    | 192.168.0.25 | 192.168.0.26 | 192.168.0.24/30 |
| PE2 ↔ FGT-B    | PE2 eth4    | FGT-B port2    | 192.168.0.29 | 192.168.0.30 | 192.168.0.28/30 |

### Internal Links (OSPF)

| Link            | Interface A    | Interface B | IP A         | IP B         | Subnet          | OSPF Area |
|-----------------|----------------|-------------|--------------|--------------|-----------------|-----------|
| FGT-A ↔ R1-A    | FGT-A port3    | R1-A Gi0/0  | 10.10.254.1  | 10.10.254.2  | 10.10.254.0/30  | Area 1    |
| FGT-HUB1 ↔ R2-H | FGT-HUB1 port3 | R2-H eth1   | 10.100.254.1 | 10.100.254.2 | 10.100.254.0/30 | Area 0    |
| FGT-HUB2 ↔ R2-H | FGT-HUB2 port3 | R2-H eth2   | 10.100.254.5 | 10.100.254.6 | 10.100.254.4/30 | Area 0    |
| FGT-B ↔ R3-B    | FGT-B port3    | R3-B Gi0/0  | 10.20.254.1  | 10.20.254.2  | 10.20.254.0/30  | Area 2    |

### Management Connections (Home Network - 192.168.40.0/24)

| Device   | Management Port | Management IP    | Gateway      |
|----------|----------------|-------------------|--------------|
| FGT-A    | port4          | 192.168.40.221/24 | 192.168.40.1 |
| FGT-B    | port4          | 192.168.40.222/24 | 192.168.40.1 |
| FGT-HUB1 | port3          | 192.168.40.223/24 | 192.168.40.1 |
| FGT-HUB2 | port3          | 192.168.40.224/24 | 192.168.40.1 |

**Connection:** 
All interfaces connect to the "External Connection" node that bridges to my home network. FGT-HUB1 and FGT-HUB2 use port3 to connect to R2-H, which is using a static route.

### HA Links (Direct Cables)

| Link                | Interface A | Interface B | Purpose              |
|---------------------|-------------|-------------|----------------------|
| FGT-HUB1 ↔ FGT-HUB2 | port4       | port4       | Heartbeat (no IP)    |
| FGT-HUB1 ↔ FGT-HUB2 | port5       | port5       | Session sync (no IP) |

---

## LAN Segments

### Branch A - Oslo

| Network       | Gateway           | DHCP Range       | Static IPs             | Purpose           |
|---------------|-------------------|------------------|------------------------|-------------------|
| 10.10.10.0/24 | 10.10.10.1 (R1-A) | 10.10.10.100-199 | Desktop1: 10.10.10.10  | User workstations |
| 10.10.99.0/24 | 10.10.99.1 (R1-A) | None             | Oslo_mgmt: 10.10.99.10 | Local management  |

### Hub - Central

| Network        | Gateway              | DHCP | Static IPs                         | Purpose              |
|----------------|----------------------|------|------------------------------------|----------------------|
| 10.100.10.0/24 | 10.100.10.254 (R2-H) | None | FAZ: .10, RADIUS: .30, Syslog: .40 | DMZ services         |
| 10.100.99.0/24 | 10.100.99.1 (R2-H)   | None | HUB_mgmt: 10.100.99.10             | Local hub management |

**DMZ Services (not configured on this lab - only visual):**
- FortiAnalyzer: 10.100.10.10
- FreeRADIUS: 10.100.10.30
- Syslog-NG: 10.100.10.40

### Branch B - Bergen

| Network       | Gateway           | DHCP Range       | Static IPs             | Purpose           |
|---------------|-------------------|------------------|------------------------|-------------------|
| 10.20.20.0/24 | 10.20.20.1 (R3-B) | 10.20.20.100-199 | Desktop2: 10.20.20.10  | User workstations |
| 10.20.99.0/24 | 10.20.99.1 (R3-B) | None             | Berg_mgmt: 10.20.99.10 | Local management  |

---

## IPsec VPN Tunnel Overlay

### Branch A to Hub

| Tunnel Name      | Local Underlay      | Remote Underlay          | Local Tunnel IP | Remote Tunnel IP | Subnet        |
|------------------|---------------------|--------------------------|-----------------|------------------|---------------|
| BRANCH-A-HUB-PRI | 192.168.0.2 (port1) | 192.168.0.10 (HUB port1) | 172.31.1.1      | 172.31.1.2       | 172.31.1.0/30 |
| BRANCH-A-HUB-BAK | 192.168.0.6 (port2) | 192.168.0.14 (HUB port2) | 172.31.1.5      | 172.31.1.6       | 172.31.1.4/30 |

### Branch B to Hub

| Tunnel Name      | Local Underlay       | Remote Underlay          | Local Tunnel IP | Remote Tunnel IP | Subnet        |
|------------------|----------------------|--------------------------|-----------------|------------------|---------------|
| BRANCH-B-HUB-PRI | 192.168.0.26 (port1) | 192.168.0.10 (HUB port1) | 172.31.2.1      | 172.31.2.2       | 172.31.2.0/30 |
| BRANCH-B-HUB-BAK | 192.168.0.30 (port2) | 192.168.0.14 (HUB port2) | 172.31.2.5      | 172.31.2.6       | 172.31.2.4/30 |