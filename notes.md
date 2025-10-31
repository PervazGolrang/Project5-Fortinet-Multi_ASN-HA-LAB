# Lab Journal and Troubleshooting

This project was designed to showcase my CCNP Enterprise (ENCOR + ENARSI) and Fortinet FCP (FCSS Enterprise Firewall 7.4, FortiGate Admin 7.4) knowledge in a realistic topology. Built on CML 2.9.0, it integrates Fortinet FGCP HA, BGP/OSPF routing, IPsec tunnels, and security policies, blending GUI and CLI configurations.

---

## Lab Limitations and Issues

### FortiGate Evaluation License Restrictions

**VDOM Not Supported:**
VDOMs are disabled on evaluation license, even if not mentioned on the webui, or most documentation. The lab is adjusted to use a single root VDOM only.

**Firewall Policy Limit:**
At Policy 3 in Step08, the FGT-HUB hit a policy limit. The evaluation license restricts a total policy count, this led to only Policy 1 and Policy 2 to be applied. The remaning policies are documented, but not implemented.

**Interface Limit:**
FortiOS 7.6.4 restricts evaluation licenses to 3 interfaces, which is unusably low for this lab. This led to me downgrading to FortiOS 7.2.0 for a 10-interface support, as it does not have an as-strict SmartLicense limiter.

### FortiAnalyzer Removed

**Original Plan:**
The original plan was to have Step 09 for FortiAnalyzer, for centralized logging, and Step 10 for DMZ services. This was later removed. The FortiAnalyzer image is corrupted on multiple download attempts from Fortinet's own website. Tried 7.4.2, 7.2.0 - all failed. Decided to remove steps 09-10 entirely. DMZ subnets (10.100.10.0/24) remains in design but unused.

The primary goal, of Steps 01-08 are complete and functional; the primary tunnels are UP andworking, and inter-branch communication is functional. The lab demonstrates core skills without logging/DMZ services.

---

## Performance and Infrastructure

### CML Memory Optimization

CML doesn't include KSM (Kernel Same-page Merging) by default, as compared to EVE-NG (comes with optimized UKSM). Installing and configuring UKSM was far too complex, so I installed KSM, which worked similarly, albeit less efficient, however, it was still a significantly improvement for memory deduplication. Manually installing and configuring KSM was simple:

Install the KSM Tuned daemon:
```bash
sudo apt update
sudo apt install ksmtuned
```

Edit the config:
```bash
sudo nano /etc/ksmtuned.conf
```
Change these settings for more aggressive deduplication settings. This change reduces memory deduplication completion from 50 minutes to 20 minutes:
```bash
KSM_MONITOR_INTERVAL=10         # How often in seconds KSM runs a check, default is 60s (too long)
KSM_SLEEP_MSEC=20               # Sleep between scan, default is 10ms (a bit conservative)
KSM_NPAGES_MAX=10000            # Maximum pages to scan, default is 1250 (very low)
KSM_THRES_COEF=70               # Activates KSM at 30% RAM usage, default is 20 (far too late)
```

Save with **CTRL+X**, then restart the daemon and enable on boot:
```bash
sudo systemctl restart ksmtuned # Restarts
sudo systemctl enable ksmtuned  # Enables on boot
```

Memory deduplication completion time reduced from 50 minutes to 20 minutes.

---

## FortiGate Configuration

### Version & Licensing

**FortiOS Version:** 7.2.0 (downgraded from 7.6.4) for FortiGate

**Reason for the downgrade:**
- FortiOS 7.6.4 uses a type of SmartLicense with extreme restrictions:
  - Maximum 3 interfaces (lab requires 6+ per firewall)
  - 1 license per FortiCare account (impossible for multi-firewall labs)
  - 1 vCPU and 2GB RAM limit
- FortiOS 7.2.0 evaluation license allows:
  - Up to 10 interfaces
  - Better resource limits
  - Multiple VMs per account

 FortiOS 7.2.0 is quite a bit older and lacks some security features, but it was necessary for the lab completion.

### Network Driver (NIC)

Issue here is that, during the project, FortiGate does not support `vmxnet3` driver in CML. I changed this to `Virtio` in the node settings. If it didn't work, I woul of tried `E1000` and accept the reduced performance. If all three failed, then I would of assumed he image was corrupted.

---

## FRR (Free Range Routing) Configuration

### Daemon Enablement

In CML, FRR has most of its routing protocols disabled by default, this includes BGP, PBR, and BFD. Enabled it in the node settings.

**Default state (bgpd disabled):**
```bash
# bgpd
ospfd
# ospf6d
# ripd
# ripngd
# isisd
# etc.
```

**Modified for this and future labs:**
```bash
bgpd          # BGP
ospfd         # OSPF
ospf6d        # OSPFv3
pimd          # PIM IPv4
pim6d         # PIM IPv6
ldpd          # LDP (MPLS label distribution)
pbrd          # Policy-based routing
bfdd          # BFD
vrrpd         # VRRP
pathd         # Segment Routing / Traffic Engineering
```

---

## References

- FortiOS 7.0 Documentation: https://docs.fortinet.com/product/fortigate/7.2
- FortiOS 7.6 Documentation: https://docs.fortinet.com/product/fortigate/7.6
- FRR Documentation: https://docs.frrouting.org/
- CML Documentation: https://developer.cisco.com/docs/modeling-labs/