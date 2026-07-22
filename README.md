# homelab
### Self-hosted infrastructure, security, and automation projects

Hi, my name is Wayne Altuama. I'm a recent Algonquin College graduate and a Computer Systems and Networking Technician, and this repo documents the home lab Im building to practice the skills ive developed while garnering new skills.

## The Lab

| Host                           | Role                       | Specs                                               |
| ------------------------------ | -------------------------- | --------------------------------------------------- |
| `pve.lab` (Dell OptiPlex 7040) | Primary Proxmox hypervisor | Intel i7-6700, 32GB DDR4, 1TB SATA SSD              |
| `pve2.lab` (Dell OptiPlex 7020)  | Secondary Proxmox node     | Intel i7 (DDR3 gen), 32GB DDR3, 512GB SSD, GTX 1650 |
| External SSD                   | Backup target              | 512GB                                               |

All hosts sit behind Tailscale, no ports forwarded to the internet.

## Projects

| Project                             | Status   | Stack                                            |
| ----------------------------------- | -------- | ------------------------------------------------ |
| [Proxmox Virtualization Server Setup](./Proxmox/Proxmox-Virtualization-Server-Setup.md) | Complete | Proxmox VE                                       |
| VirtualBox to Proxmox Migration     | Complete | Proxmox, VirtualBox                              |
| Self-Hosted Obsidian Vault Sync     | Complete | CouchDB, Self-hosted LiveSync, Tailscale, Docker |
| VirtualBox to Proxmox VM Migration  | Complete | VirtualBox, VBoxManage, Proxmox                  |
| 2-Node Proxmox Cluster              | Complete | Proxmox VE                                       |

## Roadmap

Projects planned and in design:

- [ ] Security Monitoring & SIEM (Wazuh)
- [ ] Infrastructure Automation (Ansible)
- [ ] Container Orchestration (K3s)
- [ ] Enterprise Networking Environment (Packet Tracer) 
- [ ] File Replication & Backups (Proxmox Backup Server)

## Skills Demonstrated

**Virtualization & Infrastructure:** Proxmox VE, VirtualBox migration, Docker,
**Networking:** Tailscale (subnet routing, MagicDNS, HTTPS certs)
**Systems Administration:** Windows Server, Linux server administration

## Contact

- LinkedIn: https://www.linkedin.com/in/hussein-altuama-7b6203335/
- Resume: available on request
- Email: waynealtuama@gmail.com
