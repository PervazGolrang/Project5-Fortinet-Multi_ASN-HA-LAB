# Step 01 - Initial FortiGate Setup

This step configures console access and management access on all FortiGate firewalls via the console. Primary management if my PC (192.168.40.56) using my home network router as the gateway (192.168.40.1). All the configuration is done via the CLI.

---

## Initial Login

**Default credentials (fresh FortiOS install):**
- Username: `admin`
- Password: *(blank - just press Enter)*

**New initials:**
- Username: `admin`
- Password: `root`

Ignore that the password is simple, this project is for demonstration purposes.

---

## FGT-A (Branch A - Oslo)
```bash
config system global
    set hostname FGT-A
    set timezone Europe/Oslo
end
!
config system interface
    edit port4
        set ip 192.168.40.221 255.255.255.0
        set allowaccess ping https ssh
        set description "Management"
    next
    edit port3
        set ip 10.10.254.1 255.255.255.252
        set allowaccess ping ssh https
        set description "R1-A"
end
!
config router static
    edit 1
        set gateway 192.168.40.1
        set device "port4"
end
!
config system dns
    set primary 192.168.40.1
end
```

---

## FGT-B (Branch B - Bergen)
```bash
config system global
    set hostname FGT-B
    set timezone Europe/Oslo
end
!
config system interface
    edit port4
        set ip 192.168.40.222 255.255.255.0
        set allowaccess ping https ssh
        set description "Management"
    next
    edit port3
        set ip 10.20.254.1 255.255.255.252
        set allowaccess ping https ssh
        set description "R3-B"
end
!
config router static
    edit 1
        set gateway 192.168.40.1
        set device port4
end
!
config system dns
    set primary 192.168.40.1
end
```

---

## FGT-HUB1 (Hub Primary)

A Switch (SW1) is used, as I could not increase the NIC count from the default `1` on the External Connector.

```bash
config system global
    set hostname FGT-HUB1
    set timezone Europe/Oslo
end
!
config system interface
    edit port6
        set ip 192.168.40.223 255.255.255.0
        set allowaccess ping https ssh
        set description "SW1 (Management)"
    next
    edit port3
        set ip 10.100.254.1 255.255.255.252
        set allowaccess ping https ssh
        set description "R2-H (Transit)"
end
!
config router static
    edit 1
        set gateway 192.168.40.1
        set device port6
end
!
config system dns
    set primary 192.168.40.1
end
```

---

## FGT-HUB2 (Hub Secondary)
```bash
config system global
    set hostname FGT-HUB2
    set timezone Europe/Oslo
end
!
config system interface
    edit port6
        set ip 192.168.40.224 255.255.255.0
        set allowaccess ping https ssh
        set description "SW1 (Management)"
    next
    edit port3
        set ip 10.100.254.5 255.255.255.252
        set allowaccess ping https ssh
        set description "R2-H (Transit)"
end
!
config router static
    edit 1
        set gateway 192.168.40.1
        set device port6
    next
end
!
config system dns
    set primary 192.168.40.1
end
```

---

## Verification

Test connectivity:
```bash
ping 192.168.40.221
ping 192.168.40.222
ping 192.168.40.223
ping 192.168.40.224
```

[step01_execute_ping_verf](/images/step01_execute_ping_verf.png)

**GUI Access:**
- FGT-A: `https://192.1admin68.40.221`
- FGT-B: `https://192.168.40.222`
- FGT-HUB1: `https://192.168.40.223`
- FGT-HUB2: `https://192.168.40.224`