# Step 03 - FortiGate Interfaces and VDOMs

This step configures all the FortiGate interfaces for WAN/LAN connectivity, and creates VDOMs at the hub-FWs for seperation of security zones.

---

## Branch Sites (FGT-A, FGT-B)

Branch firewalls will remain iun a single root VDOM for simplicity.

### FGT-A (Branch A - Oslo)
```bash
config system interface
    edit Loopback0
        set type loopback
        set ip 10.255.1.1 255.255.255.255
        set allowaccess ping
end
!
config system interface
    edit port1
        set vdom "root"
        set mode static
        set ip 192.168.0.2 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit port2
        set ip 192.168.0.6 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
end
```

### FGT-B (Branch B - Bergen)
```bash
config system interface
    edit Loopback0
        set type loopback
        set ip 10.255.2.1 255.255.255.255
        set allowaccess ping
end
!
config system interface
    edit port1
        set vdom "root"
        set mode static
        set ip 192.168.0.26 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit port2
        set ip 192.168.0.30 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
end
```

---

## Hub Site (FGT-HUB1, FGT-HUB2)

Hub firewalls will use VDOMs to seperate the WAN and LAN security zones. When VDOMs are created, it will default to the name "root". After enabling the VDOM mode, it will require a restart.

### FGT-HUB1 - Enable VDOMs and Create
```bash
config system global
    set vdom-mode multi-vdom
end
```

```bash
config vdom
    edit LAN
end
!
config vdom
    edit root
        set short-name "WAN"
end
```

### FGT-HUB1 - Configure WAN VDOM Interfaces

Ener the WAN VDOM context before continyuing with the rest.

```bash
# Enter WAN VDOM context
config vdom
    edit WAN
```

Simple configuration steps afterwards:

```bash
config system interface
    edit Loopback0
        set vdom "WAN"
        set type loopback
        set ip 10.255.3.1 255.255.255.255
        set allowaccess ping
end
!
config system interface
    edit "port1"
        set vdom "WAN"
        set mode static
        set ip 192.168.0.10 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit "port2"
        set vdom "WAN"
        set ip 192.168.0.14 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
end
```

### FGT-HUB1 - Creating and assigning Inter-VDOM Links

The VDOM link will be created from global context, as this would create two interfaces `vdom-link0` and `vdom-link1`. Then assignation of `vdom-link0` will be assigned to **WAN VDOM**, and `vdom-link1` to **LAN VDOM**.

```bash
config system vdom-link
    edit vdom-link
end
!
config vdom
    edit WAN
!
config system interface
    edit vdom-link0
        set vdom "WAN"
        set ip 10.100.255.1 255.255.255.252
        set allowaccess ping
        set description "LAN VDOM"
end
!
config vdom
    edit LAN
!
config system interface
    edit vdom-link1
        set vdom "LAN"
        set ip 10.100.255.2 255.255.255.252
        set allowaccess ping
        set description "WAN VDOM"
end
```

### FGT-HUB1 - Configure LAN VDOM Interfaces

```bash
config vdom
    edit LAN
!
config system interface
    edit port3
        set vdom "LAN"
    next
end
```

---

### FGT-HUB2 - Enable VDOMs and Create

```bash
config system global
    set vdom-mode multi-vdom
end
!
config vdom
    edit LAN
end
!
config vdom
    edit root
        set short-name "WAN"
end
```

### FGT-HUB2 - Configure WAN VDOM Interfaces
```bash
config vdom
    edit WAN
!
config system interface
    edit Loopback0
        set vdom "WAN"
        set type loopback
        set ip 10.255.3.2 255.255.255.255
        set allowaccess ping
end
!
config system interface
    edit port1
        set vdom "WAN"
        set mode static
        set ip 192.168.0.18 255.255.255.252
        set allowaccess ping
        set description "WAN1 to PE1"
    next
    edit port2
        set vdom "WAN"
        set ip 192.168.0.22 255.255.255.252
        set allowaccess ping
        set description "WAN2 to PE2"
end
```

### FGT-HUB2 - Create Inter-VDOM Link
```bash
config system vdom-link
    edit "vdom-link"
end
```

### FGT-HUB2 - Assign Inter-VDOM Link Interfaces
```bash
config vdom
    edit WAN
!
config system interface
    edit vdom-link0
        set vdom "WAN"
        set ip 10.100.255.1 255.255.255.252
        set allowaccess ping
        set description "LAN VDOM"
end
!
config vdom
    edit "LAN"
!
config system interface
    edit vdom-link1
        set vdom "LAN"
        set ip 10.100.255.2 255.255.255.252
        set allowaccess ping
        set description "WAN VDOM"
end
```

### FGT-HUB2 - Configure LAN VDOM Interfaces

```bash
config vdom
    edit LAN
end
!
config system interface
    edit port3
        set vdom "LAN"
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

* bilde

**Ping PE routers:**
```bash
execute ping 192.168.0.1
execute ping 192.168.0.5
```

* bilde