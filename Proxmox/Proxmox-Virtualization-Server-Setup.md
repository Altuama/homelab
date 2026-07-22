## Project Overview

This project began with a spare SFF PC and the requirement to host multiple virtualized environments as flexibly and cost-effectively as possible.

## Step 0 – Cleaning and Inspection

The OptiPlex 7040 was disassembled, inspected, and cleaned to obtain a clear understanding of the available hardware. This information assists during configuration and maintenance.

- 32 GB DDR4
- Intel i7-6700
- 1 TB SATA SSD
- One Ethernet NIC

## Step 1 – Proxmox Installation

A USB flash drive was prepared with the Proxmox VE 9.4 installer, inserted, and used to boot the system. The graphical installation was performed, and the following custom settings were applied:

- Hostname: `pve.lab`
- IP address: `10.0.0.113`
- Default gateway: `10.0.0.1`

All other installation options were left at their defaults.

## Step 2 – Physical Placement and Verification

The server was restarted and placed in its permanent location: the unfinished basement area, which is used for storage and houses the router.

### Connectivity Verification

```bash
ping 10.0.0.1
```

The Proxmox web interface was accessed at `http://10.0.0.113:8006` and a successful login was confirmed.

## Step 3 – System Preparation

### Desired Configuration

- Additional NICs for three networks (Blue, Purple, Red)
- Upload and store the following ISO files:
    - Windows Server 2019
    - Windows 10
    - Ubuntu Server
    - Ubuntu Desktop
    - TrueNAS Scale
- Windows Server: enable RDP
- Linux Server: enable SSH
- Windows Client: enable RDP
- Linux Client: enable SSH

### Virtual Machine Deployment

Four virtual machines were downloaded and installed via the Proxmox web interface:
- Windows Server
- Linux Server (Ubruno)
- Windows Client
- Linux Client

No major issues were encountered during deployment.

## Access Information

| Service        | Address / Method         |
| -------------- | ------------------------ |
| Proxmox Web UI | `http://10.0.0.113:8006` |
| Windows Server | RDP `10.0.0.143:3389`    |
| Linux Server   | SSH `wayne@10.0.0.187`   |
| Windows Client | RDP `10.0.0.23:3389`     |
| Linux Client   | SSH `wayne@10.0.0.121`   |

## Tailscale Configuration

Tailscale was installed on the Linux server running within the Proxmox host. IP forwarding was not initially enabled, which prevented subnet advertisement. The following commands were executed to enable IP forwarding:

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```
Tailscale hostname: `zapus-kingsnake.ts.net`
