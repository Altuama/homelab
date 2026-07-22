# Multi‑Domain Windows Infrastructure: Active Directory, DNS & Exchange Server 2019

## Overview

Two independent Windows Server 2022 domain controllers, each running its own Active Directory forest, DNS namespace, and Exchange Server 2019 organization, configured to resolve each other's namespaces and exchange mail across domains.

This models a scenario like two organisations needing mail interoperability **without** merging directories or establishing a trust relationship – a common requirement during post‑acquisition integration or inter‑company collaboration.

**Key Objectives:**
- Deploy two independent AD forests with separate DNS zones.
- Install and configure Exchange Server 2019 in each forest.
- Establish cross‑domain name resolution using conditional forwarders.
- Enable mail flow between the two organisations.
- Validate via OWA, raw SMTP, and an IMAP/SMTP client (Thunderbird).

---

## Environment

| | **Server A** | **Server B** |
| :--- | :--- | :--- |
| **Role** | Domain Controller / DNS / Exchange Mailbox | Domain Controller / DNS / Exchange Mailbox |
| **OS** | Windows Server 2022 Datacenter (Desktop Experience) | Windows Server 2022 Datacenter (Desktop Experience) |
| **Resources** | 8GB RAM, 2 vCPU, 60GB disk | 8GB RAM, 2 vCPU, 60GB disk |
| **Domain** | `dm89043.cst8342.com` | `dm89044.cst8342.com` |
| **Hostname** | `SERVER89043` | `SERVER89044` |
| **Red NIC IP** | `172.16.89.43` | `172.16.89.44` |
| **Blue NIC IP** | `172.16.89.143` | `172.16.89.144` |
| **Exchange Org Name** | `Proj89043altu0014 Organization` | `Proj89044altu0014 Organization` |

> **Note:** Both forests are deliberately unrelated—no trust, no shared schema. Inter‑domain mail flow relies entirely on DNS conditional forwarders and Exchange configuration.

---

## Skills Demonstrated

| Category | Skills |
| :------- | :----- |
| **Server Deployment** | Windows Server 2022 VM provisioning; NIC renaming; static IP configuration |
| **Active Directory** | New forest promotion; OU design; user creation and attribute editing (`mail`); account lifecycle (enable/disable) |
| **DNS** | Forward/reverse lookup zones; A/MX records; conditional forwarders; external forwarders |
| **Exchange 2019** | Prerequisite installation; schema/AD preparation; mailbox database creation; mailbox enabling; email address policies |
| **Mail Services** | POP3/IMAP4 service activation; OWA access; client configuration (Thunderbird) |
| **Testing & Validation** | `nslookup`, raw SMTP (`telnet`), OWA, IMAP/SMTP client tests |

---

## Implementation Phases

### Phase 1: Base Server Deployment

**Create two VMs** (Hyper‑V, VMware, or Proxmox) with the specifications in the Environment table.

**Install Windows Server 2022 Datacenter (Desktop Experience)** from ISO. Set local Administrator password (e.g., `Passw0rd!123` – **change in production**).

**Network Configuration:**

- Rename network adapters using PowerShell (on both servers):
  ```powershell
  Rename-NetAdapter -Name "Ethernet" -NewName "Red"
  Rename-NetAdapter -Name "Ethernet 2" -NewName "Blue"
  ```

- Set static IPs:
  - **SERVER89043:** Red `172.16.89.43/16` (DNS self), Blue `172.16.89.143/16` (no gateway)
  - **SERVER89044:** Red `172.16.89.44/16` (DNS self), Blue `172.16.89.144/16` (no gateway)

- Change computer names to `SERVER89043` and `SERVER89044` respectively, **do not join a domain** – reboot after change.

---

### Phase 2: Active Directory & DNS Role Installation

On **both** servers:

1. Open Server Manager → Add Roles and Features.
2. Select **Active Directory Domain Services** and **DNS Server** (include management tools).
3. After installation, click **"Promote this server to domain controller"**.

**Promote each as a new forest:**
- **SERVER89043:**
  - Root domain: `dm89043.cst8342.com`
  - DSRM password: `Passw0rd!123`
  - NetBIOS name: `DM89043`
- **SERVER89044:**
  - Root domain: `dm89044.cst8342.com`
  - DSRM password: `Passw0rd!123`
  - NetBIOS name: `DM89044`

Both servers will reboot after promotion.

---

### Phase 3: Prerequisites for Exchange 2019

On **each** server, **in this order**:

1. **Install .NET Framework 4.8** (download from Microsoft).
2. **Install Visual C++ Redistributable for Visual Studio 2012**.
3. Run the following PowerShell command (as Administrator) to install required Windows features:
   ```powershell
   Install-WindowsFeature NET-Framework-45-Features, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, Web-Mgmt-Console, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Lgcy-Mgmt-Console, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation, Server-Media-Foundation
   ```
4. **Restart** the server.
5. Mount the Exchange 2019 ISO and from the `Setup` directory, run:
   - Schema preparation:
     ```cmd
     .\Setup.exe /PrepareSchema /IAcceptExchangeServerLicenseTerms
     ```
   - AD preparation (use the organisation name from the Environment table):
     ```cmd
     .\Setup.exe /PrepareAD /OrganizationName:"Proj89043altu0014 Organization" /IAcceptExchangeServerLicenseTerms
     ```
     *(adjust organisation name for SERVER89044)*

---

### Phase 4: Exchange Server 2019 Installation

On **each** server, from the same ISO directory:

```cmd
.\Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /Mode:Install /Role:Mailbox /OrganizationName:"Proj89043altu0014 Organization"
```

The installation takes 30–60 minutes and will reboot automatically.

**Verify** by browsing to:
- `https://SERVER89043.dm89043.cst8342.com/ecp`
- `https://SERVER89044.dm89044.cst8342.com/ecp`

---

### Phase 5: DNS Configuration

On **both** servers:

#### Clean Up Unnecessary Zones
- In DNS Manager, delete all forward lookup zones except your own domain (e.g., `dm89043.cst8342.com`). Keep `TrustAnchors` if present.

#### Create A and MX Records
For each domain:

| Record Type | Name | Value |
| :---------- | :--- | :---- |
| **A** | `mail` | Server's Red IP |
| **A** | `autodiscover` | Server's Red IP |
| **MX** | (domain root) | `mail.<domain>` with priority 10 |

Example for SERVER89043:
- A: `mail` → `172.16.89.43`
- A: `autodiscover` → `172.16.89.43`
- MX: `mail.dm89043.cst8342.com` (priority 10)

Repeat on SERVER89044 with its own IP and domain.

#### Create Reverse Lookup Zone
- New Zone → Primary → IPv4 Reverse Lookup → Network ID: `172.16` → Finish.

#### Set External Forwarders
- Server Properties → Forwarders → Add `8.8.8.8`.

#### Create Conditional Forwarders
- On SERVER89043: forward `dm89044.cst8342.com` to `172.16.89.44`
- On SERVER89044: forward `dm89043.cst8342.com` to `172.16.89.43`
- Store in Active Directory.

---

### Phase 6: Active Directory Users & OUs

#### On SERVER89043 (`dm89043.cst8342.com`)
Create OUs:
- `Projaltu0014`
- `Contaltu0014`

Create users:

| Full Name | Username | Container/OU | `mail` attribute | Status |
| :-------- | :------- | :----------- | :--------------- | :----- |
| Doug Dacey | `daceyd` | Users (default) | `Doug.Dacey@dm89043.cst8342.com` | Enabled |
| Jack Doealtu0014 | `Doealtu0014` | `Projaltu0014` | `Doealtu0014@dm89043.cst8342.com` | Enabled |
| Sue Lialtu0014 | `Lialtu0014` | `Contaltu0014` | `Lialtu0014@dm89043.cst8342.com` | **Disabled** (account locked) |

#### On SERVER89044 (`dm89044.cst8342.com`)
Create OU:
- `Usersaltu0014`

Create users:

| Full Name | Username | Container/OU | `mail` attribute | Status |
| :-------- | :------- | :----------- | :--------------- | :----- |
| Doug Dacey | `daceyd` | Users (default) | `Doug.Dacey@dm89044.cst8342.com` | Enabled |
| Joe Arcatlu0014 | `Arcatlu0014` | `Usersaltu0014` | `Arcatlu0014@dm89044.cst8342.com` | Enabled |

> **Note:** For each user, set the `mail` attribute via **Attribute Editor** immediately after creation.

---

### Phase 7: Exchange Configuration

#### Create Dedicated Mailbox Database (Doug Dacey)
- Exchange Admin Center → Servers → Databases → New database: name `Dacey`, assign to the local server.

#### Enable Mailboxes
- For each user, select the user in Recipients → Mailboxes, then click **Enable** in the details pane.
- Assign Doug Dacey to the `Dacey` database; others use the default database.

#### Apply Email Address Policy
- Mail flow → Email address policies → edit the default policy to ensure correct SMTP addresses are generated (they should match the `mail` attribute). Apply policy.

#### Start POP3 and IMAP4 Services
- Open Services (`services.msc`).
- Set both `Microsoft Exchange IMAP4` and `Microsoft Exchange POP3` to **Automatic** and **Start** them.

---

### Phase 8: Testing & Validation

#### 1. DNS Resolution (on both servers)
```cmd
nslookup dm89043.cst8342.com
nslookup dm89044.cst8342.com
nslookup mail.dm89043.cst8342.com
nslookup mail.dm89044.cst8342.com
```
All should return correct IPs.

#### 2. Raw SMTP Connectivity
- Install Telnet Client if not present.
- **From SERVER89043:**
  ```cmd
  telnet SERVER89044.dm89044.cst8342.com 25
  ```
- **From SERVER89044:**
  ```cmd
  telnet SERVER89043.dm89043.cst8342.com 25
  ```
A blank screen indicates success.

#### 3. OWA (Webmail)
- Access `https://SERVER89043.dm89043.cst8342.com/owa` and log in as any user from that domain.
- Send a test email to a user in the other domain (e.g., `Joe.Arcatlu0014@dm89044.cst8342.com`).
- Verify delivery in the recipient's mailbox.

#### 4. Thunderbird Client
- Install Thunderbird.
- Configure a user (e.g., Doug Dacey) with:
  - **IMAP server:** `SERVER89043.dm89043.cst8342.com` (port 993, SSL/TLS)
  - **SMTP server:** same host (port 587, STARTTLS)
  - Username: `daceyd`
- Send and receive emails across domains.

#### 5. Disabled Account Verification
- Ensure Sue Lialtu0014 cannot log in or send mail (account disabled).

---

## Key Takeaways

| Concept                    | Lesson                                                                                                          |
| :------------------------- | :-------------------------------------------------------------------------------------------------------------- |
| **Conditional Forwarders** | Essential for cross‑forest name resolution. Without them, mail servers cannot discover each other's MX records. |
| **POP3/IMAP4 Services**    | Disabled by default in Exchange; they must be started manually for non‑OWA clients.                             |
| **Raw SMTP Testing**       | Isolates network/transport issues from client‑configuration problems early.                                     |
| **AD Schema Prep**         | Must be run *before* Exchange installation; organisation name must be consistent.                               |
| **Dual NICs**              | Used here to model separate networks (e.g., management vs. data) – no gateway ensures isolation.                |

---

## Troubleshooting Quick Reference

| Issue                             | Likely Cause                                    | Fix                                                             |
| :-------------------------------- | :---------------------------------------------- | :-------------------------------------------------------------- |
| `nslookup` for other domain fails | Conditional forwarder missing or incorrect IP   | Recreate forwarder with correct target DNS server IP            |
| SMTP telnet times out             | Firewall (Windows or physical) blocking port 25 | Enable inbound rule for SMTP (port 25) in Windows Firewall      |
| Exchange installation fails       | Missing prerequisites or reboot required        | Re‑run prerequisite installation, restart, retry                |
| OWA login fails                   | User not mailbox‑enabled or wrong domain        | Check mailbox status; ensure user logs in with UPN format       |
| IMAP client connection refused    | IMAP service not running or port not open       | Verify service is started; check firewall rule for port 993/143 |

---

## Final Verification Checklist

- [x] Domain names correct (`dm89043.cst8342.com`, `dm89044.cst8342.com`)
- [x] Red NIC IPs: `172.16.89.43` / `172.16.89.44`
- [x] DNS on each NIC points to self
- [x] Conditional forwarders in place for both domains
- [x] A records for `mail` and `autodiscover` present
- [x] MX record points to `mail.<domain>` (priority 10)
- [x] Reverse lookup zone for `172.16.0.0/16`
- [x] All users created with correct `mail` attribute
- [x] Sue Lialtu0014 disabled
- [x] Exchange installed; OWA accessible
- [x] POP3/IMAP4 services started
- [x] `nslookup` resolves all names
- [x] Telnet succeeds on port 25 between servers
- [x] Emails deliver across domains via OWA
- [x] Thunderbird can send/receive over IMAP/SMTP

---

**Environment:** [VMware]  
**College UserID:** `altu0014`

---