# Step 04 - OSPF Configuration

This step configures OSPF as internal routing within each AS. Branch sites will use a single-area OSPF, while the hub is the backbone (Area 0).

**Simple view:**
- Branch A: OSPF Area 1
- Branch B: OSPF Area 2
- Hub: OSPF Area 0

---

## Branch A - Oslo (OSPF Area 1)

### R1-A (Cisco IOSv)
```bash
interface Loopback0
 ip address 10.255.1.2 255.255.255.255
 ip ospf 1 area 1
exit
!
interface GigabitEthernet0/0
 description "FGT-A port3"
 ip address 10.10.254.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 1
 no shutdown
exit
!
router ospf 1
 router-id 10.255.1.2
 passive-interface default
 no passive-interface GigabitEthernet0/0
end 
write
```

### FGT-A

Annoyingly FortiOS 7.6.3 uses dotted-decimal notation for OSPF areas (0.0.0.1 = Area 1).

```bash
config router ospf
    set router-id 10.255.1.1
    config area
        edit 0.0.0.1
    end
    config network
        edit 1
            set prefix 10.10.254.0 255.255.255.252
            set area 0.0.0.1
        next
        edit 2
            set prefix 10.255.1.1 255.255.255.255
            set area 0.0.0.1
    end
end
write
```

---

## Branch B - Bergen (OSPF Area 2)

### R3-B (Cisco IOSv)
```bash
interface Loopback0
 ip address 10.255.2.2 255.255.255.255
 ip ospf 1 area 2
exit
!
interface GigabitEthernet0/0
 description "FGT-B port3"
 ip address 10.20.254.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 2
 no shutdown
exit
!
router ospf 1
 router-id 10.255.2.2
 passive-interface default
 no passive-interface GigabitEthernet0/0
end
write
```

### FGT-B
```bash
config router ospf
    set router-id 10.255.2.1
    config area
        edit 0.0.0.2
    end
    config network
        edit 1
            set prefix 10.20.254.0 255.255.255.252
            set area 0.0.0.2
        next
        edit 2
            set prefix 10.255.2.1 255.255.255.255
            set area 0.0.0.2
    end
end
write
```

---

## Hub - Central (OSPF Area 0)

### R2-H (FRR)
```bash
router ospf
 
 ospf router-id 10.255.3.3
exit
!
interface eth1
 ip ospf area 0
!
interface eth2
 ip ospf area 0
!
interface eth3
 ip ospf area 0
 ip ospf interface passive
!
end
write
```

### FGT-HUB1 (LAN VDOM)

OSPF will run in the LAN VDOM since it peers with R2-H over port3.

```bash
config vdom
edit LAN
!
config router ospf
    set router-id 10.255.3.1
    config area
        edit 0.0.0.0
    end
    config network
        edit 1
            set prefix 10.100.254.0 255.255.255.252
            set area 0.0.0.0
    end
end
write
```

### FGT-HUB2 (LAN VDOM)
```bash
config vdom
 edit LAN
!
config router ospf
    set router-id 10.255.3.2
    config area
        edit 0.0.0.0
        next
    end
    config network
        edit 1
            set prefix 10.100.254.4 255.255.255.252
            set area 0.0.0.0
    end
end
write
```

---

## Verification

**From R1-A:**
```bash
show ip ospf neighbor
```

* bilde

**From FGT-A and FGT-B:**
```bash
get router info ospf neighbor
```

* bilde

**Check OSPF routes:**
```bash
get router info routing-table ospf
```

* bilde