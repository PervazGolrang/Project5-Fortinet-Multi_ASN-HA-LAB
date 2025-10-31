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
```

### FGT-A

Annoyingly FortiOS 7.2.0 uses dotted-decimal notation for OSPF areas (0.0.0.1 = Area 1).
Fortinet also has quite the travel only to put a port in P2P.

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
!
config router ospf
    config ospf-interface
        edit port3
            set interface port3
            set ip 10.10.254.1
            set network-type point-to-point
        end
    !
end
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
!
config router ospf
    config ospf-interface
        edit port3
            set interface port3
            set ip 10.20.254.1
            set network-type point-to-point
        end
    !
end
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
 ip ospf passive
end
```

### FGT-HUB1

OSPF will peer with R2-H over port3.

```bash
config router ospf
    set router-id 10.255.3.1
    config area
        edit 0.0.0.0
        next
    end
    config network
        edit 1
            set prefix 10.100.254.0 255.255.255.252
            set area 0.0.0.0
        end
    end
!
config router ospf
    config ospf-interface
        edit port3
            set interface port3
            set ip 10.100.254.1
            set network-type point-to-point
        end
    !
end
```

### FGT-HUB2
```bash
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
    !
end
!
config router ospf
    config ospf-interface
        edit port3
            set interface port3
            set ip 10.100.254.5
            set network-type point-to-point
        end
    !
end
```

---

## Verification

**From R1-A:**
```bash
show ip ospf neighbor
```

[Step04 - R1-A OSPF confirmation](/images/step04_r1_show_ospf_neigh.png)

**From FGT-A and FGT-B:**
```bash
get router info ospf neighbor
```

[Step04 - Branch-FW neighbourship confirmaton](/images/step04_fgt-a-b_ospf_neigh.png)