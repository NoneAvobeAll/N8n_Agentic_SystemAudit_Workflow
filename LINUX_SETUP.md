# Linux Server Setup Guide

Detailed guide for configuring Linux servers for audit with the N8n Multi-Server Audit Workflow.

## Prerequisites

- Linux server (Ubuntu, Debian, CentOS, RHEL, etc.)
- SSH server installed and running
- SSH access (password or key-based)
- Sudo access (for some commands)

## Quick Setup (3 minutes)

Run these commands on your Linux server:

```bash
# 1. Ensure SSH is running
sudo systemctl status ssh

# 2. Install required tools (if not present)
# Ubuntu/Debian:
sudo apt-get update
sudo apt-get install -y sysstat procps

# CentOS/RHEL:
sudo yum install -y sysstat procps-ng

# 3. Create dedicated audit user (optional but recommended)
sudo adduser n8n_audit
sudo usermod -aG sudo n8n_audit

# 4. Configure passwordless sudo for read-only commands (optional)
sudo visudo
# Add this line:
# n8n_audit ALL=(ALL) NOPASSWD: /usr/bin/free, /usr/bin/df, /usr/bin/uptime, /usr/bin/journalctl, /usr/bin/systemctl is-active, /usr/bin/iostat

# 5. Test SSH from N8n host
ssh n8n_audit@YOUR_LINUX_SERVER_IP
```

## Detailed Configuration

### 1. SSH Server Setup

#### Install and Enable SSH

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

**CentOS/RHEL:**
```bash
sudo yum install -y openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo systemctl status sshd
```

#### Configure SSH

Edit `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

**Recommended Settings:**
```ssh
# Disable root login (security best practice)
PermitRootLogin no

# Allow only specific users (optional)
AllowUsers n8n_audit your_username

# Disable password authentication if using keys (recommended)
PasswordAuthentication yes  # Keep yes for initial setup, change to no after setting up keys

# Enable key-based authentication
PubkeyAuthentication yes

# Set maximum authentication attempts
MaxAuthTries 3
```

Restart SSH:
```bash
sudo systemctl restart ssh
```

### 2. Install Required System Tools

The workflow needs these tools for monitoring:

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y \
    sysstat \
    procps \
    coreutils \
    util-linux \
    grep \
    sed \
    awk \
    curl

# Optional but recommended for additional metrics
sudo apt-get install -y \
    htop \
    iotop \
    net-tools
```

**CentOS/RHEL:**
```bash
sudo yum update -y
sudo yum install -y \
    sysstat \
    procps-ng \
    coreutils \
    util-linux \
    grep \
    sed \
    awk \
    curl

# Optional but recommended
sudo yum install -y \
    htop \
    iotop \
    net-tools
```

**Arch Linux:**
```bash
sudo pacman -Syu
sudo pacman -S --needed \
    sysstat \
    procps-ng \
    coreutils \
    util-linux \
    grep \
    sed \
    gawk \
    curl
```

### 3. Configure System Monitoring Tools

#### Enable System Statistics (sysstat)

```bash
# Enable sysstat service
sudo systemctl enable sysstat
sudo systemctl start sysstat

# Configure collection interval
sudo nano /etc/sysstat/sysstat

# Add or modify this line (collect every 10 minutes)
# ACTIVATE="yes"
# COLLECT_TIME=10
```

#### Configure Journal Persistence

```bash
# Create config directory
sudo mkdir -p /etc/systemd/journald.conf.d

# Configure persistent logs
sudo bash -c 'cat > /etc/systemd/journald.conf.d/99-audit.conf <<EOF
[Journal]
Storage=persistent
SystemMaxUse=1G
MaxRetentionSec=30day
EOF'

# Restart journal
sudo systemctl restart systemd-journald
```

### 4. Create Dedicated Audit User

**Best Practice:** Create a dedicated user for N8n audits.

```bash
# Create user
sudo adduser n8n_audit

# Add to sudo group (if sudo commands needed)
sudo usermod -aG sudo n8n_audit

# Add to adm group (to read logs)
sudo usermod -aG adm n8n_audit

# Add to systemd-journal group (to read journal logs)
sudo usermod -aG systemd-journal n8n_audit  # Debian/Ubuntu
sudo usermod -aG journal n8n_audit          # CentOS/RHEL
```

### 5. Configure Sudo for Read-Only Commands

**Option A: Passwordless Sudo (Recommended)**

```bash
# Edit sudoers file
sudo visudo

# Add this line at the end
n8n_audit ALL=(ALL) NOPASSWD: /usr/bin/free, /usr/bin/df, /usr/bin/uptime, /usr/bin/journalctl, /usr/bin/systemctl, /usr/bin/iostat, /usr/bin/netstat, /usr/bin/ss, /usr/bin/who
```

**Option B: Minimal Sudo (Most Secure)**

```bash
# Edit sudoers file
sudo visudo

# Add specific commands only
n8n_audit ALL=(ALL) NOPASSWD: /usr/bin/free, /usr/bin/df, /usr/bin/uptime
n8n_audit ALL=(ALL) NOPASSWD: /usr/bin/journalctl *-p err*
n8n_audit ALL=(ALL) NOPASSWD: /usr/bin/systemctl is-active
```

**Option C: No Sudo Required (Least Privilege)**

Most audit commands don't require sudo. Test without sudo first:

```bash
# As the n8n_audit user, test commands
free -m
df -h /
uptime
journalctl -p err -n 10 --no-pager
systemctl is-active ssh
```

If all commands work, no sudo configuration needed!

### 6. Setup SSH Key Authentication (Recommended)

**On N8n Host:**
```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -f ~/.ssh/n8n_audit -N ""

# Copy public key to Linux server
ssh-copy-id -i ~/.ssh/n8n_audit.pub n8n_audit@YOUR_LINUX_SERVER_IP

# Test connection
ssh -i ~/.ssh/n8n_audit n8n_audit@YOUR_LINUX_SERVER_IP
```

**Alternative: Manual Copy**
```bash
# Copy public key manually
cat ~/.ssh/n8n_audit.pub | ssh n8n_audit@YOUR_LINUX_SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Set correct permissions
ssh n8n_audit@YOUR_LINUX_SERVER_IP "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### 7. Configure Firewall

Allow SSH traffic:

**Ubuntu/Debian (UFW):**
```bash
# Allow SSH from specific IP
sudo ufw allow from N8N_SERVER_IP to any port 22

# Or allow from anywhere (less secure)
sudo ufw allow 22/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

**CentOS/RHEL (firewalld):**
```bash
# Allow SSH from specific IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="N8N_SERVER_IP/32" service name="ssh" accept'

# Or allow SSH from anywhere
sudo firewall-cmd --permanent --add-service=ssh

# Reload firewall
sudo firewall-cmd --reload

# Check status
sudo firewall-cmd --list-all
```

### 8. Test Required Commands

Test all commands that the workflow uses:

```bash
# CPU Load
uptime

# Memory
free -m

# Disk Usage
df -h /

# Disk I/O (may require sysstat)
iostat -x 1 2

# Journal Errors
journalctl -p err -n 10 --no-pager

# Service Status
systemctl is-active ssh
systemctl is-active nginx

# Process Count
ps aux | wc -l

# Network Connections
netstat -an | grep ESTABLISHED | wc -l
# Or
ss -an | grep ESTAB | wc -l

# Logged Users
who

# Uptime
uptime -p
```

## Distribution-Specific Notes

### Ubuntu/Debian

```bash
# Additional tools
sudo apt-get install -y \
    apt-utils \
    apt-show-versions \
    needrestart

# Check package updates
apt list --upgradable

# View distribution info
cat /etc/os-release
lsb_release -a
```

### CentOS/RHEL

```bash
# Additional tools
sudo yum install -y \
    yum-utils \
    dnf-plugins-core

# Check package updates
sudo yum check-update

# View distribution info
cat /etc/os-release
cat /etc/redhat-release
```

### Arch Linux

```bash
# Additional tools
sudo pacman -S --needed \
    pacutils \
    pacman-contrib

# Check package updates
pacman -Qu

# View distribution info
cat /etc/os-release
cat /etc/arch-release
```

### Alpine Linux

```bash
# Install basic tools
apk add --no-cache \
    sysstat \
    procps \
    coreutils \
    util-linux \
    grep \
    sed \
    awk \
    curl \
    openssh-server

# OpenRC services
rc-update add sshd default
rc-update add sysstat default
rc-service sshd start
rc-service sysstat start
```

## Security Hardening

### 1. Disable SSH Root Login

```bash
sudo nano /etc/ssh/sshd_config

# Set this
PermitRootLogin no

# Restart SSH
sudo systemctl restart ssh
```

### 2. Limit SSH Users

```bash
sudo nano /etc/ssh/sshd_config

# Allow only specific users
AllowUsers n8n_audit

# Or allow groups
AllowGroups n8n_audit_group

# Restart SSH
sudo systemctl restart ssh
```

### 3. Configure SSH Timeouts

```bash
sudo nano /etc/ssh/sshd_config

# Add these lines
ClientAliveInterval 300
ClientAliveCountMax 2

# Restart SSH
sudo systemctl restart ssh
```

### 4. Use Fail2Ban (Optional)

```bash
# Ubuntu/Debian
sudo apt-get install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Create custom jail
sudo bash -c 'cat > /etc/fail2ban/jail.local <<EOF
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF'

sudo systemctl restart fail2ban
```

## Troubleshooting

### Issue: SSH Connection Refused

```bash
# Check SSH service status
sudo systemctl status ssh

# Start SSH if not running
sudo systemctl start ssh

# Check SSH config
sudo sshd -t

# Check firewall
sudo ufw status
sudo iptables -L
```

### Issue: Permission Denied for Commands

```bash
# Check user permissions
id n8n_audit

# Check sudo configuration
sudo -l -U n8n_audit

# Test command manually
sudo -u n8n_audit free -m
sudo -u n8n_audit df -h
```

### Issue: Journalctl Permission Denied

```bash
# Add user to required group
sudo usermod -aG systemd-journal n8n_audit

# Or configure journal permissions
sudo mkdir -p /etc/systemd/journald.conf.d
sudo bash -c 'cat > /etc/systemd/journald.conf.d/99-access.conf <<EOF
[Journal]
Storage=persistent
RuntimeMaxUse=50M
SystemMaxUse=1G
MaxRetentionSec=30day
EOF'

sudo systemctl restart systemd-journald
```

### Issue: iostat Not Found

```bash
# Install sysstat
# Ubuntu/Debian:
sudo apt-get install -y sysstat

# CentOS/RHEL:
sudo yum install -y sysstat

# Enable and start
sudo systemctl enable sysstat
sudo systemctl start sysstat
```

## Testing Checklist

Before adding the server to your N8n workflow:

- [ ] SSH is enabled and running
- [ ] Required tools installed (free, df, uptime, etc.)
- [ ] Audit user created and configured
- [ ] SSH access working from N8n host
- [ ] All audit commands work
- [ ] Firewall allows SSH from N8n host
- [ ] Journal logs accessible
- [ ] Service status checks work
- [ ] Disk I/O stats available (optional)

## Example Configuration for N8n Config Node

```
# Linux server with password authentication
192.168.1.10|n8n_audit|password123|linux|22|ssh

# Linux server with SSH key authentication (use empty password)
192.168.1.11|n8n_audit||linux|22|ssh

# Linux server with custom port
192.168.1.12|n8n_audit|password123|linux|2222|ssh

# Auto-detect (recommended)
192.168.1.13|n8n_audit|password123|auto|auto|auto
```

## Required Commands Summary

| Command | Purpose | Requires Sudo? |
|---------|---------|----------------|
| `uptime` | CPU load | ❌ No |
| `free -m` | Memory usage | ❌ No |
| `df -h` | Disk usage | ❌ No |
| `iostat` | Disk I/O | ❌ No (may need sysstat) |
| `journalctl` | System logs | ⚠️ Sometimes |
| `systemctl is-active` | Service status | ❌ No |
| `ps aux` | Process count | ❌ No |
| `netstat` / `ss` | Network connections | ⚠️ Sometimes |
| `who` | Logged users | ❌ No |

## Advanced: Install sshpass on N8n Host (for password auth)

If using password authentication instead of SSH keys:

**Ubuntu/Debian:**
```bash
sudo apt-get install -y sshpass
```

**CentOS/RHEL:**
```bash
sudo yum install -y sshpass
```

**Arch Linux:**
```bash
sudo pacman -S --needed sshpass
```

**Test:**
```bash
sshpass -p 'password' ssh -o StrictHostKeyChecking=no user@server "echo 'Success'"
```

## Additional Resources

- [SSH Configuration](https://man.openbsd.org/sshd_config)
- [Systemd Journal](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html)
- [Sysstat Documentation](https://github.com/sysstat/sysstat)
- [Linux Performance Monitoring](https://brendangregg.com/linuxperf.html)

## Need Help?

1. Test SSH connection from N8n host manually
2. Verify all audit commands work as the audit user
3. Check N8n execution logs for detailed error messages
4. Review SSH and system logs: `sudo journalctl -u ssh -n 50`