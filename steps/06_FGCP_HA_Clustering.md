# Step 06 - FGCP High Availability Clustering

This step configures the FGCP active-passive Ha clustering for the hub firewalls (FGT-HUB1 and FGT-HUB2). After completing both firewalls, they will operate as a single logical firewall with automatic failover. **FGT-HUB1** will act as the primary, and **FGT-HUB2** as the secondary.

---

## FGT-HUB1 (Primary)

**Short glossary:**
- `hbdev "port4" 50`: Heartbeat interface with priority 50
- `session-pickup enable`: Syncs TCP/UDP sessions between units
- `ha-mgmt-status enable`: Allows individual management access
- `override enable`: Enables preemption
- `monitor`: Triggers a failover if link goes down

After applying `set mode a-p`, FGT-HUB1 will wait for a **secondary** (FGT-HUB2) to join the cluster.

### HA Configuration
```bash
config system ha
    set group-name "HUB-CLUSTER"
    set mode a-p
    set password root123!
    set hbdev "port4" 50
    set session-pickup enable
    set session-pickup-connectionless enable
    set session-pickup-nat enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "port3"
            set gateway 10.100.254.2
        !
    end
    set override enable
    set priority 200
    set monitor "port1" "port2" "port3" "port6"
end
```

---

## FGT-HUB2 (Secondary)

### HA Configuration
```bash
config system ha
    set group-name "HUB-CLUSTER"
    set mode a-p
    set password root123!
    set hbdev "port4" 50
    set session-pickup enable
    set session-pickup-connectionless enable
    set session-pickup-nat enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "port3"
            set gateway 10.100.254.6
        next
    end
    set override disable
    set priority 100
    set monitor "port1" "port2" "port3" "port6"
end
```

**FGT-HUB1** will detect **FGT-HUB2** through FGCP on the heartbeat at port4. A priority selection will occur, where **FGT-HUB1** becomes the `active` unit due to the higher `priority 200`, and **FGT-HUB2** becomes `standby`. The active unit (**HUB1**) pushes the configuration to the standby, which takes about 60 seconds to complete. After that, session sync starts on port5. Management will be interrupted, and full cluster formation takes up to 5 minutes, around 2 minutes on faster systems.

---

## Verification

### HA Status

**From FGT-HUB1 or FGT-HUB2:**
```bash
get system ha status
```

* bilde

### Cluster from GUI

**Access cluster via virtual IP:**
Open browser: `ny ip fra pc/ruter`

* bilde

### Management Access

**Access FGT-HUB1 and FGT-HUB2:**
```bash
ssh admin@10.100.10.1
ssh admin@10.100.10.2
```

* bilde
* wireshark?

---

## Failover Testing

### Test #1 - Manual Failover

From FGT-HUB1 (active):
```bash
execute ha failover set 1
```

*Æ wireshark

**Verification:**
```bash
get system ha status
```

* bilde

### Test #2 - Interface Failure

From FGT-HUB1 (active):
```bash
config system interface
    edit "port1"
        set status down
end
```

* bilde
* wireshark?