# Step 02 - PE Routers and Service Provider Backbone

This step configures the service provider bakcone (PE1, PE2) and the internal hub router (R2-H). After this step, the underlay network is operational.

---

Some deamons are not enabled by default, I've mentioned that in [notes.md](/notes.md).

## PE1 (Service Provider Edge - 1)
```bash
interface lo
 ip address 10.255.0.1/32
 description "Router-ID"
!
interface eth0
 ip address 172.16.0.1/30
 description "WAN-EM"
 no shutdown
!
interface eth1
 ip address 192.168.0.1/30
 description "FGT-A"
 no shutdown
!
interface eth2
 ip address 192.168.0.9/30
 description "FGT-HUB1"
 no shutdown
!
interface eth3
 ip address 192.168.0.17/30
 description "FGT-HUB2"
 no shutdown
!
interface eth4
 ip address 192.168.0.25/30
 description "FGT-B"
 no shutdown
end
```

### iBGP Configuration
```bash
router bgp 2000
 bgp router-id 10.255.0.1
 no bgp default ipv4-unicast
 neighbor 10.255.0.2 remote-as 2000
 neighbor 10.255.0.2 update-source lo
 !
 address-family ipv4 unicast
  neighbor 10.255.0.2 activate
  neighbor 10.255.0.2 next-hop-self
 exit-address-family
end
```

---

## PE2 (Service Provider Edge - 2)
```bash
interface lo
 ip address 10.255.0.2/32
 description "Router-ID"
!
interface eth0
 ip address 172.16.0.5/30
 description "WAN-EM"
 no shutdown
!
interface eth1
 ip address 192.168.0.5/30
 description "FGT-A"
 no shutdown
!
interface eth2
 ip address 192.168.0.13/30
 description "FGT-HUB1"
 no shutdown
!
interface eth3
 ip address 192.168.0.21/30
 description "FGT-HUB2"
 no shutdown
!
interface eth4
 ip address 192.168.0.29/30
 description "FGT-B"
 no shutdown
end
```

### iBGP Configuration
```bash
router bgp 2000
 bgp router-id 10.255.0.2
 no bgp default ipv4-unicast
 neighbor 10.255.0.1 remote-as 2000
 neighbor 10.255.0.1 update-source lo
 !
 address-family ipv4 unicast
  neighbor 10.255.0.1 activate
  neighbor 10.255.0.1 next-hop-self
 exit-address-family
end
```

---

## R2-H (Hub Internal Router)
```bash
interface lo
 ip address 10.255.3.3/32
 description "Router-ID"
!
interface eth1
 ip address 10.100.254.2/30
 description "FGT-HUB1"
 no shutdown
!
interface eth2
 ip address 10.100.254.6/30
 description "FGT-HUB2"
 no shutdown
!
interface eth3
 ip address 10.100.10.254/24
 description "DMZ-SW"
 no shutdown
end
```