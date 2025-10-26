# ASN Plan

This document containts the ASN allocation and BGP peering relationships for the FortiGate hub-spoke topology.

---

## ASN Assignments

| ASN   | Organization        | Devices                  | Role                      |
|-------|---------------------|--------------------------|---------------------------|
| 2000  | Nordic Telecom (SP) | PE1, PE2                 | Service provider backbone |
| 65001 | Oslo Branch         | FGT-A, R1-A              | Branch office A           |
| 3481  | Central Hub         | FGT-HUB1, FGT-HUB2, R2-H | Security hub with HA      |
| 65002 | Bergen Branch       | FGT-B, R3-B              | Branch office B           |

---

## BGP Peering

### eBGP Sessions (PE-CE)

| Local Router | Local AS | Remote Router | Remote AS | Neighbor IP  |
|--------------|----------|---------------|-----------|--------------|
| PE1          | 2000     | FGT-A         | 65001     | 192.168.0.2  |
| PE2          | 2000     | FGT-A         | 65001     | 192.168.0.6  |
| PE1          | 2000     | FGT-HUB1      | 3481      | 192.168.0.10 |
| PE2          | 2000     | FGT-HUB1      | 3481      | 192.168.0.14 |
| PE1          | 2000     | FGT-HUB2      | 3481      | 192.168.0.18 |
| PE2          | 2000     | FGT-HUB2      | 3481      | 192.168.0.22 |
| PE1          | 2000     | FGT-B         | 65002     | 192.168.0.26 |
| PE2          | 2000     | FGT-B         | 65002     | 192.168.0.30 |

### iBGP Session (PE-PE)

| Local Router | Remote Router | Neighbor IP | Source IP  |
|--------------|---------------|-------------|------------|
| PE           | PE2           | 10.255.0.2  | 10.255.0.1 |

---

## Route Advertisement

### Branches/Hub to PE Routers

**Advertised:**
- Local site aggregate (10.10.0.0/16, 10.20.0.0/16, 10.100.0.0/16)

**Not Advertised:**
- Default route
- Routes learned from PE routers

### PE Routers to Customers

**Advertised:**
- Default route (0.0.0.0/0)

---

## OSPF Areas

| Site     | OSPF Area | Devices                  |
|----------|-----------|--------------------------|
| Branch A | Area 1    | FGT-A, R1-A              |
| Hub      | Area 0    | FGT-HUB1, FGT-HUB2, R2-H |
| Branch B | Area 2    | FGT-B, R3-B              |

---

## Redistribution

**At FortiGates:**
- BGP → OSPF: Default route (0.0.0.0/0) with tag 100
- OSPF → BGP: Local aggregate (/16 summary), deny routes with tag 100