# Secure EC2 Hardening + AMI Creation

## Overview
Hardened an Ubuntu EC2 instance using Bash automation, then baked it into a
reusable Amazon Machine Image (AMI) for standardized, repeatable secure deployments.
Demonstrates the full lifecycle: provision → harden → image → redeploy.

## Architecture
- Ubuntu 22.04 EC2 instance (t2.micro)
- SSH restricted to specific IP — no password auth, no root login
- UFW firewall — deny all inbound except port 22
- fail2ban — auto-bans IPs after 5 failed SSH attempts
- Custom AMI created from hardened instance
- New EC2 launched from AMI and verified identical configuration

## Technologies
- AWS EC2, AMI, EBS
- Ubuntu 22.04 LTS
- UFW, OpenSSH, fail2ban
- Bash scripting

## Hardening Applied

| Control | Implementation |
|---|---|
| Root login | Disabled (`PermitRootLogin no`) |
| Password auth | Disabled (`PasswordAuthentication no`) |
| Max auth tries | Set to 3 |
| Firewall | UFW active, deny all inbound except SSH |
| Brute force protection | fail2ban — 5 attempts = 10 min ban |
| Non-root user | `clouduser` with sudo privileges |
| SSH access | Key-based only, restricted to my IP |

## Setup Script
```bash
#!/bin/bash
# harden.sh — EC2 baseline hardening for Ubuntu 22.04

# System update
sudo apt update -y
sudo apt upgrade -y

# Install tools
sudo apt install -y curl wget unzip net-tools htop ufw fail2ban

# SSH hardening
sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?MaxAuthTries.*/MaxAuthTries 3/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw --force enable

# fail2ban SSH jail
sudo tee /etc/fail2ban/jail.local <<'EOF'
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 5
bantime = 600
findtime = 600
EOF
sudo systemctl restart fail2ban

echo "Hardening complete."
```

## AMI Creation

1. Run `harden.sh` on a fresh Ubuntu 22.04 EC2 instance
2. Verify hardening with the commands below
3. Clean instance: `sudo rm -rf /tmp/* && history -c`
4. AWS Console → EC2 → Actions → Image and templates → Create image
5. Name: `hardened-ubuntu-v1`
6. Wait for status: Available (~2–5 minutes)
7. Launch new instance from AMI and re-verify

## Verification Commands
```bash
# Confirm SSH hardening
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|maxauthtries"

# Confirm firewall
sudo ufw status verbose

# Confirm fail2ban
sudo fail2ban-client status sshd

# Confirm non-root user
groups clouduser
```

Expected output:
```
permitrootlogin no
passwordauthentication no
maxauthtries 3

Status: active
Default: deny (incoming), allow (outgoing)
22/tcp  ALLOW IN  Anywhere

Currently banned: 0

clouduser : clouduser sudo
```

## Screenshots
- `ufw-status.png` — firewall active and configured
- `sshd-hardening.png` — SSH config verified
- `ami-available.png` — AMI status in AWS console
- `new-instance-verified.png` — redeployed instance passing all checks

