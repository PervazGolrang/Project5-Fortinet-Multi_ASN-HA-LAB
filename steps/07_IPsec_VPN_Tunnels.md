# Step 07 - IPsec VPN Tunnels

This step configures the IPsec VPN tunnels between the two branch sites and the hub, using the **FortiGate GUI VPN Wizard**, and using the CLI for verification. Each branch has two tunnels (primary and backup) for redundancy.

---

## Branch A to Hub - Primary Tunnel

A site-to-site VPN between Branch A and the hub will be made using the wizard. The wizard will go through Phase 1 and Phase 2 with DH 19, SHA256, and AES-GCM. PFS (unique key generation) is enabled for best practices. A firewall policy is created to allow traffic from the branch LAN `10.10.0.0/16` to the hub LAN `10.100.0.0/16`, and a static route is added to send the traffic for the remote network through the VPN tunnel.

### Access FGT-A GUI

WebUI: `http://192.168.40.221`
Login: `admin` / `root`

### VPN Wizard

1. **VPN -> IPsec Wizard**
2. Template: **Site to Site**

---

### Step 1: VPN Setup

**Name:** `To HUB`

---

### Step 2: Authentication

**Remote device:** FortiGate (default)

**Remote IP address:** `192.168.0.10`

**Outgoing interface:** `port1`

**Authentication method:** Pre-shared Key (default)

**Pre-shared key:** `root123!SecureVPN`

---

### Step 3: Policy & Routing

**Local interface:** `port3`

**Local subnet:**
- Type: Subnet
- IP/Netmask: `10.10.0.0/16`

**Remote subnet:**
- Type: Subnet
- IP/Netmask: `10.100.0.0/16`

**Internet Access:** `None` (default)

---

### Adjust Tunnel Interface IP (CLI)

The wizard auto-assigns an IP of 0.0.0.0/0, so the tunnel IP will be adjusted via the CLI.

```bash
config system interface
    edit "To HUB"
        set ip 172.31.1.1 255.255.255.255
        set remote-ip 172.31.1.2 255.255.255.255
    next
end
```

[Step07 - FGT-A VPN Wizard Configuration](/images/step07_fgt-a_vpn_wiz.png)

---

## Branch A to Hub - Backup Tunnel

### VPN Setup
**Name:** `To Hub Backup`

### Authentication
**Remote IP address:** `192.168.0.14`  
**Outgoing interface:** `port2`  
**Pre-shared key:** `root123!SecureVPN`

### Policy & Routing
**Local interface:** `port3`  
**Local subnet:** `10.10.0.0/16`
**Remote subnet:** `10.100.0.0/16`

### Adjust Tunnel IP (CLI)
```bash
config system interface
    edit "To Hub Backup"
        set ip 172.31.1.5 255.255.255.255
        set remote-ip 172.31.1.6 255.255.255.255
    end
end
```

---

## Branch A - Add Route to Branch B

The wizard only creates routes for hub networks. Branch B is added manually.

**Network -> Static Routes**

**Route 1 (via Primary tunnel):**
- Destination: `10.20.0.0/16`
- Interface: `To HUB`
- Administrative Distance: `10`
- Comment: `Branch B via hub primary`

**Route 2 (via Backup tunnel):**
- Destination: `10.20.0.0/16`
- Interface: `To Hub Backup`
- Administrative Distance: `20`
- Comment: `Branch B via hub backup`

---

## Branch B to Hub Tunnels

FGT-B: `http://192.168.40.222`

### Primary Tunnel

**Name:** `To HUB`  
**Remote IP address:** `192.168.0.10`  
**Outgoing interface:** `port1`  
**Pre-shared key:** `root123!SecureVPN`  
**Local interface:** `port3`  
**Local subnet:** `10.20.0.0/16`
**Remote subnet:** `10.100.0.0/16`

**Tunnel IP (CLI):**
```bash
config system interface
    edit "To HUB"
        set ip 172.31.2.1 255.255.255.255
        set remote-ip 172.31.2.2 255.255.255.255
    end
end
```

### Backup Tunnel

**Name:** `To Hub Backup`  
**Remote IP address:** `192.168.0.14`  
**Outgoing interface:** `port2`  
**Pre-shared key:** `root123!SecureVPN`  
**Local interface:** `port3`  
**Local subnet:** `10.20.0.0/16`
**Remote subnet:** `10.100.0.0/16`

**Tunnel IP (CLI):**
```bash
config system interface
    edit "To Hub Backup"
        set ip 172.31.2.5 255.255.255.255
        set remote-ip 172.31.2.6 255.255.255.255
    next
end
```

### Branch B - Add Route to Branch A

**Via GUI or CLI:**
- `10.10.0.0/16` via `To HUB` (distance 10)
- `10.10.0.0/16` via `To Hub Backup` (distance 20)

---

## Hub VPN Configuration

### FGT-HUB Cluster GUI

FGT-HUB1: `http://192.168.40.223`

---

## Branch A Primary Tunnel

1. **VPN → IPsec Wizard → Create New**
2. Template: **Site to Site**

### VPN Setup
**Name:** `To BRANCH-A`

### Authentication
**Remote IP address:** `192.168.0.2`  
**Outgoing interface:** `port1`  
**Pre-shared key:** `root123!SecureVPN`

### Policy & Routing
**Local interface:** `port3`  
**Local subnet:** `10.100.0.0/16`
**Remote subnet:** `10.10.0.0/16`

### Adjust Tunnel IP (CLI)
```bash
config system interface
    edit "To BRANCH-A"
        set ip 172.31.1.2 255.255.255.255
        set remote-ip 172.31.1.1 255.255.255.255
    next
end
```

---

## Branch A Backup Tunnel

**Name:** `To BRANCH-A BAK`  
**Remote IP address:** `192.168.0.6`  
**Outgoing interface:** `port2`  
**Pre-shared key:** `root123!SecureVPN`  
**Local interface:** `port3`  
**Local subnet:** `10.100.0.0/16`
**Remote subnet:** `10.10.0.0/16`

**Tunnel IP (CLI):**
```bash
config system interface
    edit "To BRANCH-A BAK"
        set ip 172.31.1.6 255.255.255.255
        set remote-ip 172.31.1.5 255.255.255.255
    end
end
```

---

## Branch B Primary Tunnel

**Name:** `To BRANCH-B`  
**Remote IP address:** `192.168.0.26`  
**Outgoing interface:** `port1`  
**Pre-shared key:** `root123!SecureVPN`  
**Local interface:** `port3`  
**Local subnet:** `10.100.0.0/16`
**Remote subnet:** `10.20.0.0/16`

**Tunnel IP (CLI):**
```bash
config system interface
    edit "To BRANCH-B"
        set ip 172.31.2.2 255.255.255.255
        set remote-ip 172.31.2.1 255.255.255.255
    end
end
```

---

## Branch B Backup Tunnel

**Name:** `To BRANCH-B BAK`  
**Remote IP address:** `192.168.0.30`  
**Outgoing interface:** `port2`  
**Pre-shared key:** `root123!SecureVPN`  
**Local interface:** `port3`
**Local subnet:** `10.100.0.0/16`
**Remote subnet:** `10.20.0.0/16`

**Tunnel IP (CLI):**
```bash
config system interface
    edit "To BRANCH-B BAK"
        set ip 172.31.2.6 255.255.255.255
        set remote-ip 172.31.2.5 255.255.255.255
    end
end
```

---

## Verification

### Tunnel Status (CLI)
```bash
get vpn ipsec tunnel summary
```

[Step07 - IPsec tunnel summary](/images/step07_ipsec_tunnel_sum.png)