# Project 5 - Fortinet BGP/OSPF Multi-ASN HA Lab Environment

This repisitory features a multi-ASN topology built in CML 2.9.0, focusing on Fortinet **FortiOS 7.2.0** for edge security, HA clustering, and routing with OSPF and BGP.

---

## What This Lab Does
This lab simulates a central hub ASN (3481) providing transit to two branch ASNs (65001, 65002) via a service provider ASN (2000):

- **Fortinet FGCP HA**: Active-passive clustering on hub firewalls for redundancy
- **BGP Peering**: Multi-homed eBGP between branches and PEs; iBGP in backbone
- **OSPF Areas**: Multi-area setup with Area 0 (backbone), Area 1 (Branch A), Area 2 (Branch B)
- **IPsec VPN**: Site-to-site VPN with primary/backup paths for redundancy
- **Security Policies**: Firewall rules, NAT, UTM profiles (IPS, AV, WebFilter, AppControl)

Syslog, DNSMASQ and RADIUS are here as non-usable services. They are meant to fill up the DMZ, for purpose of visual demonstration.

---

## Network Topology
![Network Topology](topology/project5_multi_asn_topology.png)

---

## File Structure

```
├── configs/                             # Device-specific startup configs
├── docs/                                # Network design documentation
│   ├── IPv4_Plan.md  
│   └── ASN_Plan.md       
├── steps/                               # Step-by-step implementation guides
│   ├── 01_Initial_FortiGate_Setup.md
│   ├── 02_PE_Routers_MPLS.md
│   ├── 03_FortiGate_Interfaces.md
│   ├── 04_OSPF_Configuration.md
│   ├── 05_BGP_Peering.md
│   ├── 06_FGCP_HA_Clustering.md
│   ├── 07_IPsec_VPN_Tunnels.md
│   └── 08_Security_Policies.md
├── topology/                            # Network diagrams
│   ├── project5_multi_asn_topology_unconfigured.png
│   └── project5_multi_asn_topology.png
├── images/                              # Screenshots from implementation
├── README.md                            # This file
└── notes.md                             # Lab journal, troubleshooting   
```

---

## Network Device images

| Device         | Image version       |
| -------------- | ------------------- | 
| FortiGate      | FortiOS v7.2.0      | 
| vIOS           | 15.9(3) M10         | 
| FRR            | 10.2.1              | 
| Desktop        | Alpine Linux 3.21.3 | 

## Lab Requirements

| Component   | Requirement                       | Notes                                                                 |
| ----------- | --------------------------------- | --------------------------------------------------------------------- |
| RAM         | 10GB (w/ KSM)                     | FortiGates (2GB each), FRR/IOSv (1GB each)                            |
| vCPU        | 4 vCPUs                           | 1 per FortiGate, 1 per FRR/PE, shared for others                      |
| Platform    | CML 2.9.0                         | Requires FortiOS 7.2.0, FRR 10.x, IOSv 15.9                           |

Refer to [notes.md](/notes.md) to tune and enable KSM on CML. It would take up to 15 minutes to fully complete the memory de-duplication.

---

## Implementation Steps

**Step 1:** Console access, management IPs, basic connectivity
**Step 2:** BGP backbone (iBGP), OSPF for underlay
**Step 3:** WAN/LAN interfaces, loopbacks
**Step 4:** Multi-area OSPF within each ASN
**Step 5:** eBGP sessions between FortiGates and PE routers
**Step 6:** Active-passive HA on hub firewalls
**Step 7:** Site-to-site VPN with redundant paths
**Step 8:** Firewall rules, NAT, UTM profiles

These steps will also mostly follow the format below in terms of CLI/GUI usage:
- **Steps 1-6:** Primarily CLI (foundation and routing)
- **Steps 7-8:** Primarily GUI (security and VPN)

Full documentation in the `/steps/` directory.

---

## Notes

This lab builds on from experience and knowledge from my previous labs **Project 1** to **Project 4**, where **Project 5** focuses more on Fortinet **security orchestration** rather than on pure routing. This project was built to demonstrate my CCNP ENT, Cisco SCOR, and Fortinet FCP/FCSS Network Security knowledge in a practical, production-like environment. CML-2.9.0 is limited, e.g. no real WAN variance, so the mean focus is on configurations over performance testing. See [`notes.md`](notes.md) for implementation challenges, and troubleshooting.