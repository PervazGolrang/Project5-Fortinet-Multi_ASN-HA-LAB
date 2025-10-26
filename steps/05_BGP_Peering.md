# Step 05 - BGP Peering

This step configures route advertisement, distribution, and the eBGP sessions between the FortiGates and PE routers.

---

## PE Routers - Add Customer BGP Sessions

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
write
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
write
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
end
write
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
write
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
write
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
write
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
write
```

### Redistribution (OSPF to BGP)
```bash
config router bgp
    config redistribute ospf
        set status enable
        set route-map "OSPF-TO-BGP"
    end
end
!
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
            !
        end
    !
end
write
```

---

## Hub - FGT-HUB1

BGP runs in the **WAN VDOM**, OSPF runs in the **LAN VDOM**. Redistribution would occur between the two VDOMs via the inter-VDOM link and the static routes.

### BGP Configuration (WAN VDOM)
```bash
config vdom
edit WAN
!
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
        !
    end
end
```

### Static Route in WAN VDOM (for BGP advertisement)
```bash
config vdom
edit WAN
!
config router static
    edit 2
        set dst 10.100.0.0 255.255.0.0
        set device "vdom-link0"
        set gateway 10.100.255.2
        set comment "Hub aggregate via LAN VDOM"
    !
end
write
```

### Redistribute Default Route to LAN VDOM

Creation of a static default route (in **LAN VDOM**) is pointed to the **WAM VDOM:**

```bash
config vdom
edit LAN
!
config router static
    edit 2
        set dst 0.0.0.0 0.0.0.0
        set device "vdom-link1"
        set gateway 10.100.255.1
        set comment "Default route via WAN VDOM"
    !
end
write
```

### Redistribute Default into OSPF (LAN VDOM)
```bash
config vdom
edit LAN
!
config router ospf
    set default-information-originate enable
    set default-metric 100
end
write
```

---

## Hub - FGT-HUB2

### BGP Configuration (WAN VDOM)
```bash
config vdom
edit WAN
!
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
        !
    end
end
write
```

### Static Routes and OSPF Default Origination

```bash
config vdom
edit LAN
!
config router static
    edit 1
        set dst 0.0.0.0 0.0.0.0
        set device "vdom-link1"
        set gateway 10.100.255.1
        set comment "Default route via WAN VDOM"
    !
end
!
config router ospf
    set default-information-originate enable
    set default-metric 100
end
write
```

---

## Verification

**From PE1, check routes received from FGT-A:**
```bash
show bgp neighbors 192.168.0.2 routes
```

* bilde

### Check Redistribution

**From FGT-A, check OSPF routes:**
```bash
get router info routing-table ospf
```

* bilde