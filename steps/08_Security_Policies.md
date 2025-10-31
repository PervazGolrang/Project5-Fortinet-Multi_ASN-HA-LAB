# Step 08 - Security Policies

This step configures firewall policies, NAT, and UTM profiles using the **FortiGate GUI**. Policies control the traffic flow, and enforce security.

---

## Address Objects

Address objects simplify the policy creation by grouping the subnets.

### FGT-A

FGT-A: `http://192.168.40.221`
1. **Policy & Objects -> Addresses**
2. **Create New -> Address**

| Name         | Type   | Subnet/Range   |
|--------------|--------|----------------|
| BRANCH-A-LAN | Subnet | 10.10.0.0/16   |
| HUB-DMZ      | Subnet | 10.100.10.0/24 |
| HUB-ALL      | Subnet | 10.100.0.0/16  |
| BRANCH-B-LAN | Subnet | 10.20.0.0/16   |

### FGT-B

FGT-B: `http://192.168.40.222`

| Name         | Subnet         |
|--------------|----------------|
| BRANCH-B-LAN | 10.20.0.0/16   |
| HUB-DMZ      | 10.100.10.0/24 |
| HUB-ALL      | 10.100.0.0/16  |
| BRANCH-A-LAN | 10.10.0.0/16   |

### FGT-HUB

FGT-HUB: `http://192.168.40.224`

| Name         | Subnet         |
|--------------|----------------|
| BRANCH-A-LAN | 10.10.0.0/16   |
| BRANCH-B-LAN | 10.20.0.0/16   |
| DMZ-SERVICES | 10.100.10.0/24 |
| ALL-BRANCHES | 10.0.0.0/8     |

---

## IPS Profiles

### FGT-A

The IPS setup profile is used to **block** severe thears that Fortinet has classified with the severity **Critical** and **High**.

**Security Profiles -> Intrusion Prevention**

- Name: `IPS-Internet`
- Severity: **Critical**, **High**
- Action: **Block**

[Step08 - IPS configuration](/images/step08_FGT-A_IPS.png)

### FGT-A - Antivirus Profile

The AntiVirus profile is set for port 80 and 20/21, to protect against malware from file transfer and non-TLS web. 

**Security Profiles -> AntiVirus**

- Name: `AV-Internet`
- Scanning: **HTTP**, **FTP**
- Action: **Block**

[Step08 - AntiVirus configuration](/images/step08_FGT-A_AV.png)

### FGT-A - Web Filter Profile

The WebFilter profile will manage the internet content access, I chose the two fields below to remove degenerate content while allowing a different degenerate content.

**Security Profiles -> Web Filter**

- Name: `WebFilter-Internet`
- FortiGuard Categories: Enable
- Category **Gambling** -> Set action: **Block**
- Category **Pornography** -> Set action: **Allow**

[Step08 - WebFilter configuration](/images/step08_FGT-A_WF.png)


### FGT-A - Create Application Control Profile

The Application Control Profile will block peer-to-peer (P2P) application traffic to prevent unauthorized file-sharing.

**Security Profiles -> Application Control**

- Name: `AppControl-Internet`
- Click **Create New** under Application Overrides
- Application: **P2P**
- Action: **Block**

[Step08 - Application Control configuration](/images/step08_Application_control.png)

**I'm not copy-pasting it, but repeat the same UTM profiles on FGT-B and FGT-HUB.**

---










## Firewall Policies

### FGT-A Policies

**Policy & Objects -> Firewall Policy**

---

#### Policy 1: LAN to Internet (Primary WAN)

- **Name:** `LAN-Internet-Primary`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `port1`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `all`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **ON**

**Security Profiles:**
- **IPS:** `IPS-Internet`
- **AntiVirus:** `AV-Internet`
- **Web Filter:** `WebFilter-Internet`
- **Application Control:** `AppControl-Internet`

[Step08 - Policy 1: LAN to Internet](/images/step08_policy1.png)

---

#### Policy 2: LAN to Internet (Backup WAN)

- **Name:** `LAN-Internet-Backup`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `port2`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `all`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **ON**

**Security Profiles:**
- **IPS:** `IPS-Internet`
- **AntiVirus:** `AV-Internet`
- **Web Filter:** `WebFilter-Internet`
- **Application Control:** `AppControl-Internet`

[Step08 - Policy 2: LAN to Internet](/images/step08_policy2.png)

---

#### Policy 3: LAN to Hub (Primary Tunnel)

- **Name:** `LAN-to-HUB-Primary`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `To HUB`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `HUB-ALL`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **None** (trusted by the VPN traffic)

[Step08 - Policy 3: LAN to Hub](/images/step08_policy3.png)

---

#### Policy 4: LAN to Hub (Backup Tunnel)

- **Name:** `LAN-to-HUB-Backup`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `To Hub Backup`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `HUB-ALL`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **None**

[Step08 - Policy 4: LAN to Hub](/images/step08_policy4.png)

---

#### Policy 5: LAN to Branch B (Primary)

- **Name:** `LAN-to-BRANCH-B-Primary`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `To HUB`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `BRANCH-B-LAN`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **OFF**

[Step08 - Policy 5: LAN to Branch B | Primary](/images/step08_policy5.png)

---

#### Policy 6: LAN to Branch B (Backup)

- **Name:** `LAN-to-BRANCH-B-Backup`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `To Hub Backup`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `BRANCH-B-LAN`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **OFF**

[Step08 - Policy 6: LAN to Branch B | Backup](/images/step08_policy6.png)

---

### FGT-B Policies

I'm not copy-pasting this. Repeat the same six (6) policies on FGT-B: `http://192.168.40.222`

**Substitutions:**
- Replace `BRANCH-A-LAN` with `BRANCH-B-LAN`
- Replace `BRANCH-B-LAN` with `BRANCH-A-LAN`
- Keep all the other settings identical

---

### FGT-B Policies

Repeat the EXACT same 6 policies on FGT-B (`http://192.168.40.222`):

**Substitutions:**
- Replace `BRANCH-A-LAN` with `BRANCH-B-LAN`
- Replace `BRANCH-B-LAN` with `BRANCH-A-LAN`
- Keep all other settings identical

---

The Firwall policies follow similarly to the images from FGT-A, with differences in name, interface, source, and destination. Images for FGT-HUB will not be provided.

### FGT-HUB Policies

**Policy & Objects -> Firewall Policy**

#### Policy 1: Branch A to Branch B (Primary)

- **Name:** `BRANCH-A-to-B-Primary`
- **Incoming Interface:** `To BRANCH-A`
- **Outgoing Interface:** `To BRANCH-B`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `BRANCH-B-LAN`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **OFF**

---

#### Policy 2: Branch A to Branch B (Backup)

- **Name:** `BRANCH-A-to-B-Backup`
- **Incoming Interface:** `To BRANCH-A BAK`
- **Outgoing Interface:** `To BRANCH-B BAK`
- **Source:** `BRANCH-A-LAN`
- **Destination:** `BRANCH-B-LAN`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **OFF**

---

#### Policy 3: Branch B to Branch A (Primary)

- **Name:** `BRANCH-B-to-A-Primary`
- **Incoming Interface:** `To BRANCH-B`
- **Outgoing Interface:** `To BRANCH-A`
- **Source:** `BRANCH-B-LAN`
- **Destination:** `BRANCH-A-LAN`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **OFF**

At the creation of Policy 3, I was met with a Firewal Policy limit, as it exceeded the evaluation for FortiOS 7.2.0.

[Step08 - Evaluation mode limitation](/images/Step08_a_painful_existence_for_labbing.png)

The following policies would still follow what would of been, if I could have more policies. 

---

#### Policy 4: Branch B to Branch A (Backup)

- **Name:** `BRANCH-B-to-A-Backup`
- **Incoming Interface:** `To BRANCH-B BAK`
- **Outgoing Interface:** `To BRANCH-A BAK`
- **Source:** `BRANCH-B-LAN`
- **Destination:** `BRANCH-A-LAN`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**
- **Security Profiles:** **OFF**

---

#### Policy 5-8: VPN to DMZ

**Repetivie: four separate policies are created for each tunnel:**

Source, destination and service for Policy 6-8 are identical to Policy 5.

**Policy 5:**
- **Name:** `VPN-to-DMZ-A-Primary`
- **Incoming Interface:** `To BRANCH-A`
- **Outgoing Interface:** `port3`
- **Source:** `ALL-BRANCHES`
- **Destination:** `all`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **OFF**

**Policy 6:**
- **Name:** `VPN-to-DMZ-A-Backup`
- **Incoming Interface:** `To BRANCH-A BAK`
- **Outgoing Interface:** `port3`

**Policy 7:**
- **Name:** `VPN-to-DMZ-B-Primary`
- **Incoming Interface:** `To BRANCH-B`
- **Outgoing Interface:** `port3`

**Policy 8:**
- **Name:** `VPN-to-DMZ-B-Backup`
- **Incoming Interface:** `To BRANCH-B BAK`
- **Outgoing Interface:** `port3`

---

#### Policy 9: DMZ to Internet (Primary)

**Configuration:**
- **Name:** `DMZ-Internet-Primary`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `port1`
- **Source:** `all`
- **Destination:** `all`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **ON**

---

#### Policy 10: DMZ to Internet (Backup)

**Configuration:**
- **Name:** `DMZ-Internet-Backup`
- **Incoming Interface:** `port3`
- **Outgoing Interface:** `port2`
- **Source:** `all`
- **Destination:** `all`
- **Service:** `ALL`
- **Action:** `ACCEPT`
- **NAT:** **ON**
