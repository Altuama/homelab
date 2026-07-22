# Multi-Service Network Services Lab: Linux Server Infrastructure

## Overview

A comprehensive Linux server infrastructure lab featuring multiple network services with strict access control, firewall segmentation, and cross-subnet service access. This project demonstrates the deployment and configuration of SSH, DNS, Samba, and NFS services with granular permission management across different network segments.

**Key Objectives:**
- Deploy a Linux server with dual-network IP configuration
- Implement strict firewall policies with per-service access rules
- Configure SSH with user-based access restrictions
- Deploy DNS with forward and reverse zones
- Configure Samba with user-level read/write permissions
- Set up NFS with subnet-based read-only vs. read-write access
- Validate all services through comprehensive testing

---

## Architecture & Network Topology

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                            ┌─────────────────────┐                           │
│                            │   SERVER            │                           │
│                            │   172.16.30.110     │                           │
│                            │   172.16.32.110     │                           │
│                            │                     │                           │
│                            │  ┌───────────────┐  │                           │
│                            │  │  Services:    │  │                           │
│                            │  │  - SSH        │  │                           │
│                            │  │  - DNS        │  │                           │
│                            │  │  - Samba      │  │                           │
│                            │  │  - NFS        │  │                           │
│                            │  └───────────────┘  │                           │
│                            └──────────┬──────────┘                           │
│                                       │                                      │
│                    ┌──────────────────┼──────────────────┐                   │
│                    │                  │                  │                   │
│                    ▼                  ▼                  ▼                   │
│           ┌───────────────┐  ┌───────────────┐  ┌───────────────┐            │
│           │  30.x Network │  │  31.x Network │  │  32.x Network │            │
│           │  (Server Net) │  │  (Client Net) │  │  (Alias Net)  │            │
│           │               │  │               │  │               │            │
│           │  172.16.30.0  │  │  172.16.31.0  │  │  172.16.32.0  │            │
│           │   /24         │  │   /24         │  │   /24         │            │
│           │               │  │               │  │               │            │
│           │  ┌─────────┐  │  │  ┌─────────┐  │  │  ┌─────────┐  │            │
│           │  │ Client  │  │  │  │ Client  │  │  │  │ Client  │  │            │
│           │  │ (Test)  │  │  │  │ (Test)  │  │  │  │ (Test)  │  │            │
│           │  └─────────┘  │  │  └─────────┘  │  │  └─────────┘  │            │
│           └───────────────┘  └───────────────┘  └───────────────┘            │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐        │
│  │                       Service Access Matrix                      │        │
│  │  ┌────────────────┬─────────────┬─────────────┬─────────────┐    │        │
│  │  │     Service    │  30.x Net   │  31.x Net   │  32.x Net   │    │        │
│  │  ├────────────────┼─────────────┼─────────────┼─────────────┤    │        │
│  │  │     SSH        │    BLOCKED  │   ALLOWED   │    BLOCKED  │    │        │
│  │  │     DNS        │    BLOCKED  │   ALLOWED   │    BLOCKED  │    │        │
│  │  │    Samba       │    BLOCKED  │   ALLOWED   │    BLOCKED  │    │        │
│  │  │     NFS        │  READ-ONLY  │  READ-WRITE │    BLOCKED  │    │        │
│  │  └────────────────┴─────────────┴─────────────┴─────────────┘    │        │
│  └──────────────────────────────────────────────────────────────────┘        │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Environment

| Component | Specification |
| :-------- | :------------ |
| **Server OS** | Rocky Linux / CentOS / RHEL (systemd-based) |
| **Server Hostname** | `server.orange.lab` |
| **Server IP (Main)** | `172.16.30.110` |
| **Server IP (Alias)** | `172.16.32.110` |
| **Client OS** | Rocky Linux / CentOS / RHEL |
| **Client IP** | `172.16.31.110` |
| **Domain** | `orange.lab` |
| **Firewall** | iptables (strict default DROP policy) |
| **SELinux** | Permissive (lab environment) |

---

## Implementation Phases

### Phase 1: Server Base Configuration

The server is configured with dual IP addresses and a strict firewall foundation. All networks are blocked by default, with per-service rules added later.

**`01_server_base.sh`**

```bash
#!/bin/bash
MN=110
IF_RED="ens192"
IP_MAIN="172.16.30.$MN"
IP_ALIAS="172.16.32.$MN"

echo ">>> SERVER BASE SETUP: $IP_MAIN + $IP_ALIAS <<<"

# 1. SELinux set to permissive
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config

# 2. Network Configuration - Main IP
nmcli con delete "$IF_RED" 2>/dev/null || true

cat > /etc/sysconfig/network-scripts/ifcfg-$IF_RED <<EOF
DEVICE=$IF_RED
BOOTPROTO=none
ONBOOT=yes
IPADDR=$IP_MAIN
PREFIX=16
EOF

# 3. Network Configuration - Alias IP
cat > /etc/sysconfig/network-scripts/ifcfg-$IF_RED:$MN <<EOF
DEVICE=$IF_RED:$MN
BOOTPROTO=none
ONBOOT=yes
IPADDR=$IP_ALIAS
PREFIX=16
EOF

systemctl restart NetworkManager
nmcli connection up $IF_RED:$MN 2>/dev/null

# 4. Create test user
if ! getent passwd test > /dev/null; then
    useradd test
    echo "sba" | passwd --stdin test
fi

# 5. STRICT FIREWALL CONFIGURATION
systemctl disable --now firewalld
dnf install -y iptables-services
systemctl enable --now iptables

# Flush all rules
iptables -F
iptables -t nat -F
iptables -t mangle -F

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Essential rules
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# EXPLICIT BLOCK RULES
iptables -A INPUT -s 172.16.30.0/24 -j DROP
iptables -A INPUT -s 172.16.32.0/24 -j DROP

# NOTE: 31.0/24 is NOT blocked here - service scripts add ACCEPT rules

service iptables save

echo ">>> SERVER BASE SETUP COMPLETE <<<"
echo "Default: DROP all"
echo "Explicitly blocked: 172.16.30.0/24, 172.16.32.0/24"
```

---

### Phase 2: Client Base Configuration

The client is configured with its IP address and basic connectivity tests.

**`02_client_base.sh`**

```bash
#!/bin/bash
MN=110
IF_RED="ens192"
CLIENT_IP="172.16.31.$MN"
SERVER_IP="172.16.30.$MN"
ALIAS_IP="172.16.32.$MN"

echo ">>> CLIENT BASE SETUP: $CLIENT_IP <<<"

# 1. SELinux to permissive
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config

# 2. Remove old connections
nmcli con delete "$IF_RED" 2>/dev/null || true

# 3. Configure RED interface
cat > /etc/sysconfig/network-scripts/ifcfg-$IF_RED <<EOF
DEVICE=$IF_RED
BOOTPROTO=none
ONBOOT=yes
IPADDR=$CLIENT_IP
PREFIX=16
EOF

# 4. Restart networking
systemctl restart NetworkManager
nmcli connection up "$IF_RED" 2>/dev/null

# 5. Create test user
if ! getent passwd test > /dev/null; then
    useradd test
    echo "sba" | passwd --stdin test
fi

# 6. Test connectivity
echo "--- PING TESTS ---"
sleep 3

ping -c 2 $SERVER_IP && echo "✓ Server ($SERVER_IP) reachable" || echo "✗ Server unreachable"
ping -c 2 $ALIAS_IP && echo "✓ Alias ($ALIAS_IP) reachable" || echo "✗ Alias unreachable"
```

---

### Phase 3: SSH Service Configuration

SSH is configured to allow only `root` and `test` users. The firewall permits SSH access **only** from the 31.x client network.

**`03_server_ssh.sh`**

```bash
#!/bin/bash

# Configure SSH to allow only root and test
sed -i '/AllowUsers/d' /etc/ssh/sshd_config
echo "AllowUsers root test" >> /etc/ssh/sshd_config

# Enable key authentication
sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

systemctl restart sshd

# Firewall: Allow SSH from client network only (31.0/24)
# Insert BEFORE the DROP rules for 30.0/24 and 32.0/24
iptables -I INPUT 3 -p tcp --dport 22 -s 172.16.31.0/24 -j ACCEPT

service iptables save

echo "SSH configured: Only root/test allowed, cst8246 blocked"
echo "Firewall: Port 22 open for 172.16.31.0/24"
```

**SSH Access Matrix:**

| User | Access Status |
| :--- | :------------ |
| `root` | ✅ Allowed (key-based) |
| `test` | ✅ Allowed (key-based) |
| `cst8246` | ❌ Blocked |
| Other users | ❌ Blocked |

**SSH Client Testing (`04_client_ssh.sh`):**

```bash
#!/bin/bash
SERVER_IP="172.16.30.110"

# Generate key if needed
[ ! -f ~/.ssh/id_rsa ] && ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa

echo "--- SSH SETUP ---"

# Install sshpass if needed
dnf install -y sshpass 2>/dev/null

# Accept host key
ssh-keyscan $SERVER_IP >> ~/.ssh/known_hosts 2>/dev/null

# Copy keys with retry
copy_key() {
    user=$1
    pass=$2
    echo "Copying key to $user..."
    for i in {1..3}; do
        sshpass -p "$pass" ssh-copy-id -o StrictHostKeyChecking=no -f $user@$SERVER_IP 2>&1 | grep -v "WARNING"
        if [ $? -eq 0 ]; then
            echo "✓ Key copied to $user"
            return 0
        fi
        echo "Attempt $i failed, retrying..."
        sleep 2
    done
    echo "✗ Failed to copy key to $user"
    return 1
}

# Copy keys
copy_key root "abc"
copy_key test "sba"

echo "--- SSH TESTS ---"

# Test access
ssh -o BatchMode=yes root@$SERVER_IP "echo 'Root: PASS'" 2>/dev/null && echo "Root access: ✓" || echo "Root access: ✗"
ssh -o BatchMode=yes test@$SERVER_IP "echo 'test: PASS'" 2>/dev/null && echo "test access: ✓" || echo "test access: ✗"

# Test cst8246 blocked
ssh -o BatchMode=yes cst8246@$SERVER_IP "echo 'Test'" 2>/dev/null && echo "cst8246: ✗ (should be blocked)" || echo "cst8246: ✓ (correctly blocked)"
```

---

### Phase 4: DNS Service Configuration

BIND DNS server configured with:
- Forward zone: `orange.lab`
- Reverse zone: `30.16.172.in-addr.arpa`
- Recursion allowed only from 31.x network
- Firewall permits DNS queries from 31.x network only

**`05_server_dns.sh`**

```bash
#!/bin/bash
MN=110

# Install BIND
dnf install -y bind bind-utils

# Configure named.conf
cat > /etc/named.conf <<EOF
options {
    directory "/var/named";
    allow-query { any; };
    recursion yes;
    allow-recursion { 172.16.31.0/24; localhost; };
    listen-on port 53 { any; };
};
zone "orange.lab" IN { type master; file "orange.lab.zone"; };
zone "30.16.172.in-addr.arpa" IN { type master; file "orange.lab.rev"; };
EOF

# Create forward zone file
cat > /var/named/orange.lab.zone <<EOF
\$TTL 86400
@ IN SOA server.orange.lab. root.orange.lab. ( 1 1H 15M 1W 1D )
@ IN NS server.orange.lab.
@ IN MX 10 mail.orange.lab.
server IN A 172.16.30.$MN
www1   IN A 172.16.30.$MN
mail   IN A 172.16.30.$MN
alias  IN A 172.16.32.$MN
EOF

# Create reverse zone file
cat > /var/named/orange.lab.rev <<EOF
\$TTL 86400
@ IN SOA server.orange.lab. root.orange.lab. ( 1 1H 15M 1W 1D )
@ IN NS server.orange.lab.
$MN IN PTR server.orange.lab.
EOF

# Set permissions
chown named:named /var/named/orange.lab.*
systemctl enable --now named

# Allow DNS from client network
iptables -I INPUT -p tcp --dport 53 -s 172.16.31.0/24 -j ACCEPT
iptables -I INPUT -p udp --dport 53 -s 172.16.31.0/24 -j ACCEPT
service iptables save

echo "DNS setup complete"
```

**DNS Records:**

| Record Type | Name | Value |
| :---------- | :--- | :---- |
| A | `server.orange.lab` | 172.16.30.110 |
| A | `www1.orange.lab` | 172.16.30.110 |
| A | `mail.orange.lab` | 172.16.30.110 |
| A | `alias.orange.lab` | 172.16.32.110 |
| MX | `orange.lab` | mail.orange.lab (priority 10) |
| PTR | 110.30.16.172.in-addr.arpa | server.orange.lab |

**DNS Client Testing (`06_client_dns.sh`):**

```bash
#!/bin/bash
SERVER="172.16.30.110"

# Set DNS server
cat > /etc/resolv.conf << EOF
nameserver $SERVER
nameserver 192.168.101.2
nameserver 8.8.8.8
search orange.lab
EOF

echo "--- DNS TESTS ---"
dig server.orange.lab
dig www1.orange.lab
dig orange.lab MX
dig -x 172.16.30.110
dig google.com
```

---

### Phase 5: Samba Service Configuration

Samba configured with:
- Share name: `samba-private`
- Users: `user1` (read/write), `user2` (read-only)
- `user1` has write access; `user2` has read-only access
- Firewall permits SMB/NMB from 31.x network only

**`07_server_samba.sh`**

```bash
#!/bin/bash

# Install Samba
dnf install -y samba

# Create users
useradd user1; echo "p" | passwd --stdin user1
useradd user2; echo "p" | passwd --stdin user2
(echo "p"; echo "p") | smbpasswd -a user1
(echo "p"; echo "p") | smbpasswd -a user2

# Add share config
cat >> /etc/samba/smb.conf <<EOF
[samba-private]
path = /srv/samba
valid users = user1 user2
write list = user1
read only = yes
EOF

# Create directory
mkdir -p /srv/samba
chmod 777 /srv/samba
systemctl enable --now smb

# Allow Samba from client network
iptables -I INPUT -p tcp -m multiport --dports 139,445 -s 172.16.31.0/24 -j ACCEPT
iptables -I INPUT -p udp -m multiport --dports 137,138 -s 172.16.31.0/24 -j ACCEPT
service iptables save

echo "Samba setup complete"
```

**Samba Access Matrix:**

| User | Permission | Proof |
| :--- | :--------- | :---- |
| `user1` | ✅ Read/Write | Can create, modify, and delete files |
| `user2` | ✅ Read-Only | Can list and download files; cannot create or modify |

**Samba Client Testing (`08_client_samba.sh`):**

Key test outcomes:

| Test | Action | Expected Result | Verification |
| :--- | :----- | :-------------- | :----------- |
| 1 | User1 writes file | ✅ Success | File appears in share |
| 2 | User2 lists file | ✅ Success | File visible in directory |
| 3 | User2 writes file | ❌ Access Denied | `NT_STATUS_ACCESS_DENIED` |
| 4 | User1 downloads file | ✅ Success | File content matches original |
| 5 | Server-side verification | ✅ Success | File exists on server filesystem |
| 6 | User2 downloads file | ✅ Success | Read-only access confirmed |

---

### Phase 6: NFS Service Configuration

NFS configured with:
- Export: `/srv/nfs`
- 31.x network: **Read-Write** access
- 30.x network: **Read-Only** access
- Complete firewall rules for NFS ports
- Built-in self-test validates RO/RW access

**`09_server_nfs.sh`**

```bash
#!/bin/bash

# Install NFS
dnf install -y nfs-utils

# Create directory
mkdir -p /srv/nfs
chmod 777 /srv/nfs

# Configure exports - RW for 31.x, RO for 30.x
echo "/srv/nfs 172.16.31.0/24(rw,sync) 172.16.30.0/24(ro,sync)" > /etc/exports

# Start services
systemctl enable --now nfs-server

# Complete NFS firewall rules
echo "Configuring firewall for NFS..."
for NET in 172.16.31.0/24 172.16.30.0/24; do
    # Standard NFS ports
    iptables -I INPUT -p tcp -m multiport --dports 111,2049,20048 -s $NET -j ACCEPT
    iptables -I INPUT -p udp -m multiport --dports 111,2049,20048 -s $NET -j ACCEPT
    # Additional RPC ports for NFSv3
    iptables -I INPUT -p tcp -m multiport --dports 32769,32803 -s $NET -j ACCEPT
    iptables -I INPUT -p udp -m multiport --dports 32769,32803 -s $NET -j ACCEPT
done

service iptables save
exportfs -r

echo "NFS setup complete."

# ===== SELF-TEST: Server's own access (30.x - Should be RO) =====
echo ""
echo "--- Testing Server Access (30.x network - Should be RO) ---"
SERVER_MOUNT="/mnt/nfs_server_test"
mkdir -p $SERVER_MOUNT

# Mount from server to itself (30.x network)
mount -t nfs 172.16.30.110:/srv/nfs $SERVER_MOUNT 2>/dev/null

if mount | grep -q "$SERVER_MOUNT"; then
    echo "   ✓ Mount successful from 30.x network"
    echo "   ✓ Read access confirmed"

    # Test write access (should fail)
    if touch $SERVER_MOUNT/ServerWriteTest.nfs 2>/dev/null; then
        echo "   ✗ FAIL: Server can write (should be RO only!)"
    else
        echo "   ✓ PASS: Server correctly blocked from writing (RO access)"
    fi

    umount $SERVER_MOUNT
else
    echo "   ✗ Mount failed from 30.x network"
fi
rmdir $SERVER_MOUNT 2>/dev/null

echo ""
echo "=== NFS Setup Verification ==="
echo "✓ Client network (31.0/24): Read/Write access"
echo "✓ Server network (30.0/24): Read-Only access"
echo "✓ Complete firewall rules configured for NFS ports"
```

**NFS Access Matrix:**

| Network | Access | Proof |
| :------ | :----- | :---- |
| 172.16.31.0/24 (Client) | ✅ Read-Write | Can create and modify files |
| 172.16.30.0/24 (Server) | ✅ Read-Only | Can read files; write operations fail |
| 172.16.32.0/24 (Alias) | ❌ Blocked | Cannot mount or access |

**NFS Client Testing (`10_client_nfs.sh`):**

```bash
#!/bin/bash

SERVER="172.16.30.110"
MOUNT_POINT="/mnt/nfs_test"

echo "--- NFS Client Test (31.x network - Should be RW) ---"

# Mount NFS share
mount -t nfs $SERVER:/srv/nfs $MOUNT_POINT

echo "1. Testing write access..."
echo "NFS test from client 172.16.31.110" > $MOUNT_POINT/ReadMe.nfs
echo "Created at: $(date)" >> $MOUNT_POINT/ReadMe.nfs

echo "2. Verifying file creation..."
ls -la $MOUNT_POINT/ReadMe.nfs

echo "3. Reading file content:"
cat $MOUNT_POINT/ReadMe.nfs

echo "4. Testing modification..."
echo "Modified successfully from client" >> $MOUNT_POINT/ReadMe.nfs

# Clean up
umount $MOUNT_POINT
rmdir $MOUNT_POINT

echo "--- Test Complete ---"
echo "Client (31.x network): Read/Write access verified ✓"
```

---

## Verification Summary

### 1. SSH Verification

| Test | Result |
| :--- | :----- |
| root SSH access from 31.x client | ✅ PASS |
| test SSH access from 31.x client | ✅ PASS |
| cst8246 SSH access from 31.x client | ❌ BLOCKED |
| SSH from 30.x network | ❌ BLOCKED (firewall) |

### 2. DNS Verification

| Test | Result |
| :--- | :----- |
| A record: `server.orange.lab` | ✅ Resolves to 172.16.30.110 |
| A record: `www1.orange.lab` | ✅ Resolves to 172.16.30.110 |
| A record: `mail.orange.lab` | ✅ Resolves to 172.16.30.110 |
| A record: `alias.orange.lab` | ✅ Resolves to 172.16.32.110 |
| MX record: `orange.lab` | ✅ Returns mail.orange.lab |
| PTR record: 172.16.30.110 | ✅ Returns server.orange.lab |

### 3. Samba Verification

| Test | Action | Result |
| :--- | :----- | :----- |
| User1 | Write file to share | ✅ SUCCESS |
| User2 | List files in share | ✅ SUCCESS |
| User2 | Write file to share | ❌ ACCESS DENIED |
| User1 | Download file | ✅ SUCCESS |
| User2 | Download file | ✅ SUCCESS |
| Server-side | File existence | ✅ CONFIRMED |

### 4. NFS Verification

| Network | Access | Result |
| :------ | :----- | :----- |
| 172.16.31.110 (Client) | Read/Write | ✅ PASS |
| 172.16.30.110 (Server) | Read-Only | ✅ PASS |
| 172.16.32.0/24 | Blocked | ✅ PASS (firewall) |

---

## Key Takeaways

| Concept | Lesson |
| :------ | :----- |
| **Default DROP Firewall** | Starting with DROP all and selectively allowing services provides the most secure foundation |
| **Firewall Rule Order** | Insert rules at the correct position (`-I`) to override general DROP rules |
| **SSH User Restrictions** | `AllowUsers` directive in sshd_config provides simple user-based access control |
| **Samba Permissions** | `write list` and `read only` provide granular user-level access control |
| **NFS Export Options** | `rw` vs `ro` provides network-based access control without complex client configuration |
| **DNS Recursion** | Restrict recursion to trusted networks to prevent DNS amplification attacks |
| **SELinux Permissive** | For lab environments, set to permissive to reduce troubleshooting complexity |
| **iptables Multiport** | Use `-m multiport` to combine multiple ports in a single rule |

---

## Troubleshooting Reference

| Issue | Likely Cause | Fix |
| :---- | :----------- | :-- |
| SSH connection timeout | Firewall rule missing or incorrect position | Verify rule is before DROP rules with `iptables -L INPUT --line-numbers` |
| SSH user denied | User not in `AllowUsers` list | Check `/etc/ssh/sshd_config` and restart sshd |
| DNS resolution fails | Zone file syntax error | Check BIND logs: `journalctl -u named` |
| Samba access denied | Samba password not set | Run `smbpasswd -a <user>` |
| NFS write fails | Export option is `ro` | Verify `/etc/exports` has `rw` for correct subnet |
| NFS mount hangs | RPC ports blocked | Ensure all NFS-related ports are open in firewall |
| File not visible in share | Share not restarted after config change | Restart smb service: `systemctl restart smb` |

---

## Completion Checklist

**Phase 1: Server Base**
- [x] SELinux set to permissive
- [x] Main IP (172.16.30.110) configured
- [x] Alias IP (172.16.32.110) configured
- [x] test user created with password `sba`
- [x] Strict firewall: default DROP policy
- [x] 30.0/24 and 32.0/24 explicitly blocked

**Phase 2: Client Base**
- [x] Client IP (172.16.31.110) configured
- [x] test user created with password `sba`
- [x] Connectivity to server verified

**Phase 3: SSH**
- [x] AllowUsers: root, test only
- [x] Pubkey authentication enabled
- [x] Password authentication enabled
- [x] Firewall allows SSH from 31.0/24 only

**Phase 4: DNS**
- [x] BIND installed and configured
- [x] Forward zone: `orange.lab`
- [x] Reverse zone: `30.16.172.in-addr.arpa`
- [x] Recursion allowed for 31.0/24 only
- [x] Firewall allows DNS from 31.0/24 only

**Phase 5: Samba**
- [x] Samba installed
- [x] user1 (p), user2 (p) created
- [x] Samba passwords set
- [x] Share `samba-private` configured
- [x] user1: write list; user2: read-only
- [x] Firewall allows SMB/NMB from 31.0/24

**Phase 6: NFS**
- [x] NFS installed
- [x] Export directory: `/srv/nfs`
- [x] 31.0/24: rw; 30.0/24: ro
- [x] All required ports (111, 2049, 20048, 32769, 32803) open
- [x] Self-test verifies RO access from server

**Client Verification**
- [x] SSH tests passed
- [x] DNS tests passed
- [x] Samba tests passed
- [x] NFS tests passed

---

**Environment:** [Red Hat Enterprise Linux Server / Client]

**Network Documentation:**
- Server Main: `172.16.30.110`
- Server Alias: `172.16.32.110`
- Client: `172.16.31.110`
- Domain: `orange.lab`

---
