# Homelab and Projects

Hi, my name is Wayne Altuama. I'm a recent Algonquin College graduate and a Computer Systems and Networking Technician, and this repo documents the home lab Im building to practice the skills ive developed while garnering new skills.
# Homelab Projects

| Project                                                               | Stack                                            |
| --------------------------------------------------------------------- | ------------------------------------------------ |
| [Proxmox Virtualization Server Setup](Proxmox-Cluster.md)             | Proxmox VE                                       |
| VirtualBox to Proxmox Migration                                       | Proxmox, VirtualBox, VBoxManage                  |
| [Self-Hosted Obsidian Vault Sync](Self-Hosted-Obsidian-Vault-Sync.md) | CouchDB, Self-hosted LiveSync, Tailscale, Docker |
| [2-Node Proxmox Cluster](Proxmox-Cluster.md)                          | Proxmox VE                                       |
### The Lab

| Host                            | Role                       | Specs                                               |
| ------------------------------- | -------------------------- | --------------------------------------------------- |
| `pve.lab` (Dell OptiPlex 7040)  | Primary Proxmox hypervisor | Intel i7-6700, 32GB DDR4, 1TB SATA SSD              |
| `pve2.lab` (Dell OptiPlex 7020) | Secondary Proxmox node     | Intel i7 (DDR3 gen), 32GB DDR3, 512GB SSD, GTX 1650 |
| External SSD                    | Backup target              | 512GB                                               |

_All hosts sit behind Tailscale, no ports forwarded to the internet._

# Algonquin College Projects

| Project                                                                                 | Stack                                                                                        |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| [Advanced Enterprise LAN & Routing](./Algonquin/Advanced-Enterprise-LAN-and-Routing.md) | Cisco IOS, Packet Tracer, IPv4, OSPFv2, BGPv4, RSTP, LACP                                    |
| [Enterprise LAN & Routing](./Algonquin/Enterprise-LAN-and-Routing.md)                   | Cisco IOS, IPv4, IPv6, OSPFv2, Routing, SSH, TFTP                                            |
| [Windows Enterprise Administration](./Algonquin/Windows-Enterprise-Administration.md)   | Windows Server 2022, Exchange Server 2019, Active Directory, DNS, OWA, Thunderbird, VMware   |
| [Linux Network Services](./Algonquin/Linux-Network-Services.md)                         | RHEL, Bash, BIND DNS, NFS, Samba, SSH, iptables, VMware                                      |
| Malware Traffic Analysis - Pushdo Trojan                                                | Security Onion, Sguil, Kibana, Wireshark, NetworkMiner, VirusTotal, MITRE ATT&CK, VirtualBox |
| Enterprise MSP Project                                                                  | Proxmox, Windows Server, Ubuntu, VPN, Veeam, LDAP, Active Directory, TrueNAS                 |
| Cloud Helpdesk Ticketing System                                                         | Spiceworks, Cloud Hosted, Email Authentication, Client Portal                                |

## Skills Demonstrated

**Virtualization & Infrastructure**

- Proxmox VE deployment and administration (2-node cluster with HA)  
- Virtual machine lifecycle management (creation, migration, backup)
- VirtualBox to Proxmox migration using `VBoxManage` and Proxmox import tools
- Docker containerization (CouchDB deployment via Docker Compose)
- Resource allocation and optimization (CPU, RAM, storage)
- Hardware selection and capacity planning

**Networking & Routing**

- Tailscale VPN configuration (subnet routing, MagicDNS, HTTPS certificates)
- Cisco IOS routing and switching (OSPFv2 multi-area, BGPv4 peering)
- IPv4/IPv6 dual-stack implementation
- Routing protocol tuning (reference bandwidth, cost manipulation, timers)
- Route summarization and redistribution
- Redundancy implementation (floating static routes, dynamic failover)
- Enterprise switching (RSTP, VLANs, VTPv2, LACP EtherChannel)
- Trunk links and VLAN segmentation

**Systems Administration - Windows**

- Windows Server 2022 deployment and configuration (physical/virtual)
- Active Directory Domain Services (multi-forest design, OUs, users, groups)
- DNS Server configuration (forward/reverse zones, conditional forwarders, MX records)    
- Exchange Server 2019 installation and administration
- Mailbox management (databases, enabling users, email address policies)
- Cross-domain mail flow configuration
- IMAP4/POP3 service configuration for third-party clients
- OWA and client-based mail testing (Thunderbird)

**Systems Administration - Linux**

- RHEL-based Linux server administration
- BIND DNS server deployment (forward/reverse zones, recursion control)
- NFS server configuration (network-based access control, RO vs. RW exports)
- Samba file server deployment (user-level permissions, read-only vs. read-write)
- SSH server hardening (user restrictions, key-based authentication)
- iptables firewall management (default DROP policies, per-service access rules)
- Bash scripting for automation and deployment
- Package management and service administration (systemd)
- SELinux configuration (permissive/enforcing modes)

**Security Operations & Threat Analysis**

- Security Onion deployment and configuration
- Alert triage and event correlation using Sguil
- Log visualization and filtering with Kibana
- Packet capture analysis using Wireshark
- Host identification and OS fingerprinting with NetworkMiner
- File extraction from HTTP traffic
- Cryptographic hash generation (SHA1, SHA256, SHA512)
- Threat intelligence verification using VirusTotal
- MITRE ATT&CK framework mapping (tactics and techniques)
- Indicator of Compromise (IOC) identification and documentation

**Network Security**

- Strict firewall policy implementation (default DROP, per-service allow)
- Network segmentation and access control
- SSH encryption and authentication hardening
- Port security and access port configuration

**Tools & Methodologies**

- Cisco Packet Tracer for network simulation
- VMware/VirtualBox for virtualization
- Incident investigation methodology
- Threat intelligence gathering
- Technical documentation and report writing
- Command-line proficiency (Linux, Windows PowerShell, Cisco IOS)

## Contact

- LinkedIn: [https://www.linkedin.com/in/hussein-altuama-7b6203335/](https://www.linkedin.com/in/hussein-altuama-7b6203335/)
- Resume: available on request
- Email: waynealtuama@gmail.com
