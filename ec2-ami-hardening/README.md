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
<img width="1710" height="1112" alt="Screenshot 2026-03-28 at 2 25 14 PM" src="https://github.com/user-attachments/assets/71d11c6d-24e2-469e-b548-359a45237538" />

- `sshd-hardening.png` — SSH config verified
<img width="1710" height="1112" alt="Screenshot 2026-03-28 at 2 24 07 PM" src="https://github.com/user-attachments/assets/f2be6927-80e6-4d8e-b6ab-e9cf4f1289ea" />

- `ami-available.png` — AMI status in AWS console
<img width="1710" height="1112" alt="Screenshot 2026-03-27 at 4 33 32 PM" src="https://github.com/user-attachments/assets/140ba8aa-9f1c-4e82-9036-c0cf638df016" />

- `new-instance-verified.png` — redeployed instance passing all checks
<img width="1710" height="1112" alt="Screenshot 2026-03-27 at 9 24 23 PM" src="https://github.com/user-attachments/assets/b5098931-e121-479c-b477-e4940a9f8d5a" />
