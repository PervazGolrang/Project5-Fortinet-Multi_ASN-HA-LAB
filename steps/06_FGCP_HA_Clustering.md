# Step 06 - FGCP High Availability Clustering

This step configures the FGCP active-passive Ha clustering for the hub firewalls (FGT-HUB1 and FGT-HUB2). After completing both firewalls, they will operate as a single logical firewall with automatic failover. **FGT-HUB1** will act as the primary, and **FGT-HUB2** as the secondary.

---

HA seems quite simple, but in Fortinet it's tricky GUI-wise compared to other vendors. It would practically require to fill in 10+ fields anually with a mix of click-type-click-type. It was much faster, less error-prone, and less boring to use the cli.

The main reason why I kept seperate links is, due to, even though Heartbeat traffic is small, it's frequent (millisecnds), so session sync can be quite large (up in the thousands) - seperating them prevens session sync delays. This is quite a small deployment, but **It's still best practice** in my opinion. There is nothing lost, other than one port, and reduces future changes if it came to it.

## FGT-HUB1 (Primary)

**Short glossary:**
- `session-sync-dev "port5"`: Session synchronization interface (failover and load-balancing)
- `hbdev "port4" 50`: Heartbeat interface with priority 50
- `session-pickup enable`: Syncs TCP/UDP sessions between units
- `ha-mgmt-status enable`: Allows individual management access
- `override enable`: Enables preemption (only FGT-HUB1)
- `monitor`: Triggers a failover if link goes down

After applying `set mode a-p`, FGT-HUB1 will wait for a **secondary** (FGT-HUB2) to join the cluster.

### HA Configuration

```bash
config system ha
    set group-name "HUB-CLUSTER"
    set mode a-p
    set password root123!
    set hbdev "port4" 50
    set session-sync-dev "port5"
    set session-pickup enable
    set session-pickup-connectionless enable
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
    set session-sync-dev "port5"
    set session-pickup enable
    set session-pickup-connectionless enable
    set override disable
    set priority 100
    set monitor "port1" "port2" "port3" "port6"
end
```

**FGT-HUB1** will detect **FGT-HUB2** through FGCP on the heartbeat at port4. A priority selection will occur, where **FGT-HUB1** becomes the `active` unit due to the higher `priority 200`, and **FGT-HUB2** becomes `standby`. The active unit (**HUB1**) pushes the configuration to the standby, which takes about 60 seconds to complete. After that, session sync starts on port5. Management will be interrupted, and full cluster formation takes up to 5 minutes, around 2 minutes on faster systems.

---

## Verification

### HA Status

**FGT-HUB1:**
```bash
get system ha status
```

[Step06 - System HA Status](/images/step06_system_ha_status.png)