---
name: server-security-init-skill
description: Agent skill for safely initializing and hardening fresh Ubuntu/Debian SSH servers with staged security improvements
triggers:
  - "help me secure a new Ubuntu server"
  - "initialize security on my fresh Debian VPS"
  - "harden SSH access on my new server"
  - "set up firewall and fail2ban on Ubuntu"
  - "secure a new Linux server with SSH keys"
  - "bootstrap security for a fresh cloud instance"
  - "configure UFW and disable root login"
  - "initialize server security best practices"
---

# Server Security Init Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill guides you through safely initializing and hardening fresh Ubuntu/Debian SSH servers. It implements a staged bootstrap flow: selecting or generating local SSH keys, requiring public-key login before remote changes, creating a non-root sudo user, changing the SSH port, disabling root/password login, configuring UFW firewall, setting up fail2ban, and verifying no lockout conditions exist.

## What This Skill Does

The server-security-init skill provides guidance for:

- **Public-key SSH bootstrap** without exposing root passwords in commands, logs, or config files
- **Staged SSH hardening** with verified non-root sudo login before disabling old access paths
- **UFW firewall setup** with default-deny inbound policy and explicit port allowances
- **Fail2ban jail configuration** for SSH brute-force protection with management IP exceptions
- **Debian 12/systemd socket checks** to prevent ssh.socket from silently keeping SSH on the old port
- **Server-side verification** using `ss -ltnp`, `sshd -T`, `systemctl`, and `ufw status`
- **Recovery checks** for fail2ban bans, firewall mistakes, and SSH listener mismatches

## Installation

This skill should be installed by copying the `server-security-init/` directory to your AI agent's user-level skills directory.

**For AI agents:**
- Install only the `server-security-init/` directory, NOT the repository root
- Do not execute server initialization during skill installation
- After installation, inform the user if their agent needs to restart or reload skills

**For humans using npx:**
```bash
npx skills add https://github.com/DeerYang/server-security-init-skill/tree/main/server-security-init -g
```

Target specific agents:
```bash
npx skills add https://github.com/DeerYang/server-security-init-skill/tree/main/server-security-init -g -a codex
```

## Staged Security Initialization Flow

### Stage 1: Pre-Flight Checks and Bootstrap

Before making any changes, gather and verify:

```python
# Example bootstrap facts to collect
bootstrap_info = {
    "server_ip": "203.0.113.42",
    "current_user": "root",
    "ssh_port": 22,
    "management_ips": ["198.51.100.5"],  # Your current IP
    "local_ssh_key": "~/.ssh/id_ed25519",
    "target_user": "admin",
    "target_ssh_port": 2222
}
```

**Verify existing SSH key or generate:**
```bash
# Check for existing key
ls -la ~/.ssh/id_ed25519.pub

# Generate if needed (DO NOT automate this without user confirmation)
ssh-keygen -t ed25519 -C "server-bootstrap-$(date +%Y%m%d)" -f ~/.ssh/id_ed25519
```

**Test password-based root SSH access (initial):**
```bash
ssh -p 22 root@203.0.113.42 'echo "SSH accessible"'
```

### Stage 2: Install Public Key

Copy your public key to the server:

```bash
# Using ssh-copy-id (prompts for password)
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 22 root@203.0.113.42

# Or manually
ssh -p 22 root@203.0.113.42 'mkdir -p ~/.ssh && chmod 700 ~/.ssh'
cat ~/.ssh/id_ed25519.pub | ssh -p 22 root@203.0.113.42 'cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys'
```

**Verify public-key login works:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 root@203.0.113.42 'echo "Public key works"'
```

**CRITICAL:** Do not proceed until public-key authentication is verified.

### Stage 3: Create Non-Root Sudo User

```bash
# Create user and grant sudo
ssh -i ~/.ssh/id_ed25519 -p 22 root@203.0.113.42 << 'EOF'
adduser --disabled-password --gecos "Admin User" admin
usermod -aG sudo admin
echo "admin ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/admin
chmod 440 /etc/sudoers.d/admin
EOF
```

**Copy SSH key to new user:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 root@203.0.113.42 << 'EOF'
mkdir -p /home/admin/.ssh
cp /root/.ssh/authorized_keys /home/admin/.ssh/
chown -R admin:admin /home/admin/.ssh
chmod 700 /home/admin/.ssh
chmod 600 /home/admin/.ssh/authorized_keys
EOF
```

**Verify new user sudo access:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 'sudo echo "Sudo works"'
```

**CRITICAL:** Do not proceed until the new user can SSH in and sudo without password.

### Stage 4: Configure Firewall (UFW)

```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 << 'EOF'
sudo apt-get update
sudo apt-get install -y ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow new SSH port BEFORE enabling firewall
sudo ufw allow 2222/tcp comment 'SSH'

# Allow other services as needed
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'

# Enable firewall (answer yes)
sudo ufw --force enable

# Verify
sudo ufw status numbered
EOF
```

**Verify firewall rules:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 'sudo ufw status verbose'
```

Expected output:
```
Status: active
Logging: on (low)

To                         Action      From
--                         ------      ----
2222/tcp                   ALLOW IN    Anywhere                   # SSH
80/tcp                     ALLOW IN    Anywhere                   # HTTP
443/tcp                    ALLOW IN    Anywhere                   # HTTPS
```

### Stage 5: Change SSH Port

**Edit sshd_config:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 << 'EOF'
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
sudo sed -i 's/^#Port 22$/Port 2222/' /etc/ssh/sshd_config
sudo sed -i 's/^Port 22$/Port 2222/' /etc/ssh/sshd_config
EOF
```

**Check for ssh.socket (Debian 12+):**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 << 'EOF'
if systemctl is-enabled ssh.socket 2>/dev/null | grep -q enabled; then
    echo "WARNING: ssh.socket is enabled and may keep SSH on port 22"
    echo "Disabling ssh.socket and using ssh.service directly..."
    sudo systemctl disable --now ssh.socket
    sudo systemctl enable ssh.service
fi
EOF
```

**Restart SSH service:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 'sudo systemctl restart ssh'
```

**Verify new port is listening:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 22 admin@203.0.113.42 'sudo ss -ltnp | grep :2222'
```

Expected output:
```
LISTEN 0      128          0.0.0.0:2222       0.0.0.0:*    users:(("sshd",pid=1234,fd=3))
```

**Test new port:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 admin@203.0.113.42 'echo "New port works"'
```

**CRITICAL:** Keep the old SSH session open until new port is verified.

### Stage 6: Harden SSH Configuration

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 admin@203.0.113.42 << 'EOF'
sudo sed -i 's/^#PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#PasswordAuthentication .*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/^PasswordAuthentication .*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/^#PubkeyAuthentication .*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/^#ChallengeResponseAuthentication .*/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/^ChallengeResponseAuthentication .*/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config

# Test configuration
sudo sshd -t

# Restart SSH
sudo systemctl restart ssh
EOF
```

**Verify effective configuration:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 admin@203.0.113.42 'sudo sshd -T | grep -E "^(permitrootlogin|passwordauthentication|pubkeyauthentication|port)"'
```

Expected output:
```
port 2222
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
```

### Stage 7: Configure Fail2ban

```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 admin@203.0.113.42 << 'EOF'
sudo apt-get install -y fail2ban

# Create local jail configuration
sudo tee /etc/fail2ban/jail.local > /dev/null <<'CONFIG'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
ignoreip = 127.0.0.1/8 ::1 198.51.100.5

[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
backend = systemd
CONFIG

sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
EOF
```

**Verify fail2ban status:**
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 admin@203.0.113.42 'sudo fail2ban-client status sshd'
```

Expected output:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 0
   |- Total banned:     0
   `- Banned IP list:
```

### Stage 8: Update Local SSH Config

Add server entry to local `~/.ssh/config`:

```bash
cat >> ~/.ssh/config << 'EOF'

Host my-server
    HostName 203.0.113.42
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    StrictHostKeyChecking ask
    UserKnownHostsFile ~/.ssh/known_hosts
EOF

chmod 600 ~/.ssh/config
```

**Test simplified connection:**
```bash
ssh my-server 'echo "SSH config works"'
```

## Verification Checklist

After completing all stages, verify the following server-side:

### SSH Service
```bash
ssh my-server << 'EOF'
# Check SSH is listening on correct port only
sudo ss -ltnp | grep sshd

# Check effective sshd configuration
sudo sshd -T | grep -E "^(port|permitrootlogin|passwordauthentication|pubkeyauthentication)"

# Check ssh.socket is not interfering (Debian 12)
systemctl is-enabled ssh.socket
systemctl is-active ssh.socket
EOF
```

### Firewall
```bash
ssh my-server 'sudo ufw status verbose'
```

Verify:
- Status is "active"
- Default incoming is "deny"
- New SSH port is allowed
- No rule allowing port 22

### Fail2ban
```bash
ssh my-server << 'EOF'
sudo systemctl is-active fail2ban
sudo fail2ban-client status sshd
sudo fail2ban-client get sshd ignoreip
EOF
```

Verify:
- fail2ban service is active
- sshd jail is enabled
- Your management IP is in ignoreip

### User and Sudo
```bash
ssh my-server << 'EOF'
whoami
sudo -l
groups
EOF
```

Verify:
- Logged in as non-root user
- User has NOPASSWD sudo
- User is in sudo group

## Common Patterns

### Adding Additional Management IPs

```bash
ssh my-server << 'EOF'
sudo sed -i 's/^ignoreip = .*/& 203.0.113.100/' /etc/fail2ban/jail.local
sudo fail2ban-client reload
sudo fail2ban-client get sshd ignoreip
EOF
```

### Opening Additional Firewall Ports

```bash
ssh my-server << 'EOF'
sudo ufw allow 8080/tcp comment 'Application'
sudo ufw status numbered
EOF
```

### Checking for Banned IPs

```bash
ssh my-server 'sudo fail2ban-client status sshd'
```

### Unbanning an IP

```bash
ssh my-server 'sudo fail2ban-client set sshd unbanip 198.51.100.10'
```

## Troubleshooting

### Locked Out After Port Change

If new port doesn't work and old session is closed:
1. Use provider console/VNC access
2. Check `sudo ss -ltnp | grep sshd` for actual listening port
3. Check `sudo systemctl status ssh.socket` and `ssh.service`
4. Review `/var/log/auth.log` for SSH errors

### UFW Blocked New SSH Port

If you enabled UFW before allowing new SSH port:
1. Use provider console/VNC access
2. `sudo ufw disable`
3. `sudo ufw allow 2222/tcp`
4. `sudo ufw enable`

### Fail2ban Banned Your IP

Check ban status:
```bash
sudo fail2ban-client status sshd
```

Unban yourself:
```bash
sudo fail2ban-client set sshd unbanip YOUR_IP
```

Add to ignoreip:
```bash
sudo nano /etc/fail2ban/jail.local
# Add IP to ignoreip line
sudo fail2ban-client reload
```

### SSH Port Reverts to 22 (Debian 12)

This happens when `ssh.socket` is active:
```bash
sudo systemctl disable --now ssh.socket
sudo systemctl enable --now ssh.service
sudo systemctl restart ssh
sudo ss -ltnp | grep sshd
```

### Root Login Still Works

Check effective configuration:
```bash
sudo sshd -T | grep permitrootlogin
```

If it shows "yes", ensure:
- `/etc/ssh/sshd_config` has `PermitRootLogin no` (uncommented)
- No Match blocks at end of file override this
- Restart: `sudo systemctl restart ssh`

### Password Authentication Still Accepted

```bash
sudo sshd -T | grep passwordauthentication
```

If it shows "yes":
- Edit `/etc/ssh/sshd_config`: `PasswordAuthentication no`
- Also set: `ChallengeResponseAuthentication no`
- Test: `sudo sshd -t`
- Restart: `sudo systemctl restart ssh`

## Safety Notes

- **Always keep an existing SSH session open** when changing SSH or firewall settings
- **Verify each stage** before proceeding to the next
- **Never disable password auth** until public-key login is proven to work
- **Never change SSH port** until firewall allows new port
- **Never disable root login** until non-root sudo user is verified
- **Provider console/VNC access** is the ultimate recovery path
- **Do not paste private keys** into prompts, scripts, or logs
- **Test `sudo sshd -t`** before restarting SSH after config changes

## System Requirements

- Ubuntu 20.04+ or Debian 11+
- OpenSSH server
- systemd init system
- UFW (Uncomplicated Firewall)
- fail2ban package available in apt repositories

This skill provides guidance for AI-assisted server administration. Always review generated commands before execution on production systems.
