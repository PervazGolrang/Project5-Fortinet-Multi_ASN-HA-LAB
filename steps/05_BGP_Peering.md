# Step 05 - BGP Peering

This step configures route advertisement, distribution, and the eBGP sessions between the FortiGates and PE routers.

---

## PE Routers

### PE1
```bash
router bgp 2000
 !
 neighbor 192.168.0.2 remote-as 65001
 neighbor 192.168.0.2 description "FGT-A"
 !
 neighbor 192.168.0.10 remote-as 3481
 neighbor 192.168.0.10 description "FGT-HUB1"
 !
 neighbor 192.168.0.18 remote-as 3481
 neighbor 192.168.0.18 description "FGT-HUB2"
 !
 neighbor 192.168.0.26 remote-as 65002
 neighbor 192.168.0.26 description "FGT-B"
 !
 address-family ipv4 unicast
  neighbor 192.168.0.2 activate
  neighbor 192.168.0.2 default-originate
  neighbor 192.168.0.10 activate
  neighbor 192.168.0.10 default-originate
  neighbor 192.168.0.18 activate
  neighbor 192.168.0.18 default-originate
  neighbor 192.168.0.26 activate
  neighbor 192.168.0.26 default-originate
 exit-address-family
end
```

Adding OSPF here compared to [Step04](/steps/04_OSPF_Configuration.md), as it would now have both OSPF and BGP.

```bash
router ospf
 ospf router-id 10.255.0.1
!
interface lo
 ip ospf area 0
!
interface eth0
 ip ospf area 0
!
interface eth1
 ip ospf passive
!
interface eth2
 ip ospf passive
!
interface eth3
 ip ospf passive
!
interface eth4
 ip ospf passive
exit
```

### PE2
```bash
router bgp 2000
 !
 neighbor 192.168.0.6 remote-as 65001
 neighbor 192.168.0.6 description "FGT-A"
 !
 neighbor 192.168.0.14 remote-as 3481
 neighbor 192.168.0.14 description "FGT-HUB1"
 !
 neighbor 192.168.0.22 remote-as 3481
 neighbor 192.168.0.22 description "FGT-HUB2"
 !
 neighbor 192.168.0.30 remote-as 65002
 neighbor 192.168.0.30 description "FGT-B"
 !
 address-family ipv4 unicast
  neighbor 192.168.0.6 activate
  neighbor 192.168.0.6 default-originate
  neighbor 192.168.0.14 activate
  neighbor 192.168.0.14 default-originate
  neighbor 192.168.0.22 activate
  neighbor 192.168.0.22 default-originate
  neighbor 192.168.0.30 activate
  neighbor 192.168.0.30 default-originate
 exit-address-family
end
```

```bash
router ospf
 ospf router-id 10.255.0.2
!
interface lo
 ip ospf area 0
!
interface eth0
 ip ospf area 0
!
interface eth1
 ip ospf passive
!
interface eth2
 ip ospf passive
!
interface eth3
 ip ospf passive
!
interface eth4
 ip ospf passive
exit
```

---

## Branch A - FGT-A

### BGP Configuration
```bash
config router bgp
    set as 65001
    set router-id 10.255.1.1
    config neighbor
        edit "192.168.0.1"
            set remote-as 2000
            set description "PE1"
        next
        edit "192.168.0.5"
            set remote-as 2000
            set description "PE2"
    end
    config network
        edit 1
            set prefix 10.10.0.0 255.255.0.0
        end
    !
end
```

### Redistribution (BGP to OSPF)

The `set tag 100` tags the packets, used for loop-prevention.

```bash
config router ospf
    config redistribute bgp
        set status enable
        set metric 100
        set tag 100
    end
end
```

Since there's a BGP to OSPF redistribution, I added loop-prevention, which requires the creation of two rules for the route-map, a short explination:
- Rule 1: Denies all routes with tag 100 (BGP tagged with 100 above, `set tag 100`)
- Rule 2: Permits all other OSPF routes, as it will `deny` as default if not permitted.

Works similarly to Cisco route-maps.

```bash
config router route-map
    edit "OSPF-TO-BGP"
        config rule
            edit 1
                set match-tag 100
                set set-route-tag 0
                set action deny
            next
            edit 2
                set action permit
        end
    !
end
!
config router bgp
    config redistribute ospf
        set status enable
        set route-map "OSPF-TO-BGP"
    end
end
```

---

## Branch B - FGT-B

### BGP Configuration
```bash
config router bgp
    set as 65002
    set router-id 10.255.2.1
    config neighbor
        edit "192.168.0.25"
            set remote-as 2000
            set description "PE1"
        next
        edit "192.168.0.29"
            set remote-as 2000
            set description "PE2"
        !
    end
    config network
        edit 1
            set prefix 10.20.0.0 255.255.0.0
    end
end
```

### Redistribution (BGP to OSPF)
```bash
config router ospf
    config redistribute bgp
        set status enable
        set metric 100
        set tag 100
    end
end
```

### Redistribution (OSPF to BGP)
```bash
config router route-map
    edit "OSPF-TO-BGP"
        config rule
            edit 1
                set match-tag 100
                set set-route-tag 0
                set action deny
            next
            edit 2
                set action permit
        end
    !
end
!
config router bgp
    config redistribute ospf
        set status enable
        set route-map "OSPF-TO-BGP"
    end
end
```

---

## Hub - FGT-HUB1

### BGP Configuration
```bash
config router bgp
    set as 3481
    set router-id 10.255.3.1
    config neighbor
        edit "192.168.0.9"
            set remote-as 2000
            set description "PE1"
        next
        edit "192.168.0.13"
            set remote-as 2000
            set description "PE2"
        !
    end
    config network
        edit 1
            set prefix 10.100.0.0 255.255.0.0
        end
    !
end
```

### Redistribute OSPF to BGP
```bash
config router bgp
    config redistribute ospf
        set status enable
    end
end
```

### Redistributing the default route to OSP
```bash
config router ospf
    set default-information-originate enable
    set default-metric 100
end
```

---

## Hub - FGT-HUB2

### BGP Configuration
```bash
config router bgp
    set as 3481
    set router-id 10.255.3.2
    config neighbor
        edit "192.168.0.17"
            set remote-as 2000
            set description "PE1"
        next
        edit "192.168.0.21"
            set remote-as 2000
            set description "PE2"
        !
    end
    config network
        edit 1
            set prefix 10.100.0.0 255.255.0.0
        end
    !
end
```

### Redistribute OSPF to BGP
```bash
config router bgp
    config redistribute ospf
        set status enable
    end
end
```

### Redistributing the default route to OSPF
```bash
config router ospf
    set default-information-originate enable
    set default-metric 100
end
```

---

## Verification

**PE1 BGP Table**
```bash
show bgp neighbors 192.168.0.2 routes
```

[Ste05 - BGP Summary of PE1](/images/step05_pe1_bgp_sum.png)