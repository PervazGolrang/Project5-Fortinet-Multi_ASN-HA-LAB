# Project 3 - Fortinet Multi-ASN Network with BGP, OSPF, and HA

This repository contains a multi-ASN topology built using CML 2.9.0, which focuses on Fortinet Firewalls as edge security, HA-clustering, and IGP/EGP routing using BGP and OSPF. The project integrates FRR for provider edge, Cisco IOSv for branch-routing, and DMZ services (FAZ, DNSMASQ, RADUS, SYSLOG NG, NTP Server). The lab is supposed to demonstrate a semi-realistic real-world ISP-branch scenarios with dual-homing, failover, and security policies, using a mix of both Fortinet GUI and CLI for configs.

---

## What This Lab Does
This lab simulates a central backhone ASN (3481), to provide transit to two branch ASNs (65001, 65002) via a provider ASN (2000):
- **Fortinet FGCP HA**: Active-passive clustering on HUB-firewalls for redundancy.
- **BGP Peering**: Multi-homed eBGP between branches and PEs; iBGP in backbone.
- **OSPF Areas**: Multi-area setup with Area 0 (backbone), Area 1 (left branch), Area 2 (right branch).
- **Security & Services**: Firewall policies, VPN (IPSec), DMZ hosting analytics/logging/auth-services.
- **Endpoint Testing**: Browser containers (Firefox/Chrome) and desktops for reachability and validation.
- **WAN Emulation**: Simulated delays/jitter via WAN-EM.

---

## Network Topology
![Network Topology](topology/project3_multi_asn_topology.png)

### Drawio Topology
[project3_topology.drawio](topology/project3_topology.drawio)

---

## File Structure

├── configs/                    # Device‑specific startup‑configs (.yaml)
├── docs/                       # Network design documentation
│   ├── ASN_Plan.md  
│   ├── Design.md
│   ├── IP_Plan.md              
│   ├── Security_Policies.md  
│   └── Services_Config.md      
├── steps/                      # Step-by-step implementation guides
│   ├── 01_Base_Interfaces.md
│   ├── 02_OSPF_Config.md
│   ├── 03_BGP_Config.md
│   ├── 04_FGCP_HA.md
│   ├── 05_Firewall_Policies.md
│   ├── 06_DMZ_Services.md
│   ├── 07_WAN_Emulation.md
│   └── 08_Verification.md
├── topology/                   # Diagrams (.png, .drawio)
├── wireshark/                  # Packet captures for validation
└── notes.md                    # Lab journal, troubleshooting    

---

## Lab Requirements

| Component   | Requirement                       | Notes                                                                 |
| ----------- | --------------------------------- | --------------------------------------------------------------------- |
| RAM         | 32GB (w/ KSM)                     | FortiGates (4-8GB each), FRR/IOSv (1GB each), services (2GB total)    |
| vCPU        | 21 vCPUs                          | 4 vCPU per FortiGate HA pair, 2 per FRR/PE, 1 per others              |
| Platform    | CML 2.9.0                         | FortiGate 7.4.x, FRR 8.x, IOSv 15.9, Linux containers (DNSMASQ, etc.) |

Refer to [notes.md](/notes.md) to tune and enable KSM on CML. Do note it would take up to 15 minutes to fully complete the memory de-duplication.

---

## Implementation Steps

**Step 1:** Interface/IP setup and loopbacks  
**Step 2:** OSPF foundation across areas  
**Step 3:** BGP sessions (eBGP multi-homing, iBGP in backbone)  
**Step 4:** FGCP HA clustering on HUB FortiGates  
**Step 5:** Firewall policies, NAT, and security profiles  
**Step 6:** DMZ services config (FAZ, DNS, RADIUS, SYSLOG, NTP)  
**Step 7:** WAN-EM setup for latency/jitter simulation  
**Step 8:** Verification, failover testing, and packet captures

Each step builds upon the previous step - using GUI for the Fortinet Policies, and the CLI for routing/debugging.

---

## Notes

This lab was built to bridge my CCNP ENT, Cisco SCOR, and Fortinet FCP knowledge. CML is limited, e.g. no real WAN variance, so the mean focus is on configurations over performance testing. Troubleshooting and lab-changes is mentioned on [notes.md](/notes.md), which also covers common HA sync issues and BGP convergence.

The goal of this lab is to expend my education from theoretical Fortinet and security knowledge, to practical lab-knowledge.