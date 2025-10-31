# Step 03 - FortiGate Interfaces and VDOMs

This step configures all FortiGate interfaces for WAN/LAN connectivity. All FortiGates use single root VDOM.

---

## Branch Sites (FGT-A, FGT-B)

Delete the `fortilink` IP as its a FortiSwitch function, and it's in the way for FGT-A Loopback0.

### FGT-A (Branch A - Oslo)
```bash
config system interface
    edit "fortilink"
        unset ip
end
!
config system interface
    edit "Loopback0"
        set vdom "root"
        set type loopback
        set ip 10.255.1.1 255.255.255.255
        set allowaccess ping
    next
    edit "port1"
        set vdom "root"
        set mode static
        set ip 192.168.0.2 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit "port2"
        set vdom "root"
        set ip 192.168.0.6 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
    !
end
```

### FGT-B (Branch B - Bergen)
```bash
config system interface
    edit "fortilink"
        unset ip
end
!
config system interface
    edit "Loopback0"
        set vdom "root"
        set type loopback
        set ip 10.255.2.1 255.255.255.255
        set allowaccess ping
    next
    edit "port1"
        set vdom "root"
        set mode static
        set ip 192.168.0.26 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit "port2"
        set vdom "root"
        set ip 192.168.0.30 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
    !
end
```

---

## Hub Site (FGT-HUB1, FGT-HUB2)

### FGT-HUB1
```bash
config system interface
    edit "Loopback0"
        set vdom "root"
        set type loopback
        set ip 10.255.3.1 255.255.255.255
        set allowaccess ping
    next
    edit "port1"
        set vdom "root"
        set mode static
        set ip 192.168.0.10 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit "port2"
        set vdom "root"
        set ip 192.168.0.14 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
    !
end
```

### FGT-HUB2

```bash
config system interface
    edit "Loopback0"
        set vdom "root"
        set type loopback
        set ip 10.255.3.2 255.255.255.255
        set allowaccess ping
    next
    edit "port1"
        set vdom "root"
        set mode static
        set ip 192.168.0.18 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit "port2"
        set vdom "root"
        set ip 192.168.0.22 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
    !
end
```

---

## Verification

### Branch Firewalls

**From FGT-A:**
```bash
get system interface physical port1
get system interface physical port2
get system interface physical port3
```

[Step03 - Get interface information on Ports 1-3](/images/step03_get_int_physic_1-3.png)

### Hub Firewalls

**Ping R2-H:**
```bash
execute ping 10.100.254.2
```

[Step03 - Ping from FGT-HUB1 to R2-H](/images/step03_ping_r2-h_from_hub1.png)