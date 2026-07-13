---
name: macos-security-privacy-hardening
description: Comprehensive security and privacy hardening guide for macOS systems with Apple silicon
triggers:
  - how do I secure my macOS system
  - implement macOS security hardening
  - configure macOS privacy settings
  - setup FileVault and firewall on Mac
  - harden macOS against threats
  - improve macOS security and privacy
  - setup secure macOS workstation
  - configure macOS security best practices
---

# macOS Security and Privacy Hardening

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This guide provides enterprise-standard security and privacy hardening techniques for macOS systems, particularly Apple silicon Macs running currently supported macOS versions. It covers threat modeling, system configuration, encryption, firewalls, DNS security, browser hardening, and operational security practices.

## Overview

The macOS Security and Privacy Guide is a comprehensive collection of security hardening techniques that covers:

- Threat modeling and risk assessment
- Hardware security considerations
- System installation and initial configuration
- FileVault disk encryption
- Firewall configuration (application, kernel, and third-party)
- DNS security with DNSCrypt and encrypted DNS
- Browser hardening (Firefox, Chrome, Safari)
- Certificate authority management
- VPN and Tor configuration
- PGP/GPG encryption
- System monitoring and auditing
- Physical security measures

**Key Security Principle**: Apple silicon Macs are strongly recommended over Intel Macs due to hardware-level security vulnerabilities in Intel CPUs that cannot be patched.

## Installation and Setup

### Initial System Setup

1. **Install Latest macOS**:
   ```bash
   # Check for system updates
   softwareupdate --list
   
   # Install all available updates
   sudo softwareupdate --install --all
   
   # Enable automatic updates
   sudo softwareupdate --schedule on
   ```

2. **Enable FileVault Disk Encryption**:
   ```bash
   # Enable FileVault (requires restart)
   sudo fdesetup enable
   
   # Check FileVault status
   sudo fdesetup status
   
   # List FileVault enabled users
   sudo fdesetup list
   ```

3. **Configure Firmware Password** (Apple silicon):
   ```bash
   # Set firmware password in Recovery Mode
   # Restart and hold Command+R, then in Terminal:
   firmwarepasswd -setpasswd
   
   # Verify firmware password is set
   firmwarepasswd -check
   ```

## Core Security Configurations

### Firewall Setup

#### Enable Application Layer Firewall

```bash
# Enable firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable logging
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Enable stealth mode (don't respond to ping)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Prevent built-in software from being auto-allowed
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsigned off
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsignedapp off

# Restart firewall
sudo pkill -HUP socketfilterfw
```

#### Kernel-Level Packet Filtering (pf)

Create `/etc/pf.conf`:

```bash
# Default deny policy
set block-policy drop
set fingerprints "/etc/pf.os"
set ruleset-optimization basic
set skip on lo0

# Scrub incoming packets
scrub in all no-df

# Block all by default
block in log all
block out log all

# Allow outgoing connections
pass out quick proto {tcp, udp, icmp} from any to any keep state

# Allow specific incoming services (example: SSH)
# pass in quick proto tcp from any to any port 22 keep state
```

Enable pf firewall:

```bash
# Test configuration
sudo pfctl -nf /etc/pf.conf

# Enable pf
sudo pfctl -e

# Load rules
sudo pfctl -f /etc/pf.conf

# View current rules
sudo pfctl -sr

# View statistics
sudo pfctl -si
```

### Admin and User Accounts

```bash
# Create standard user account (not admin)
sudo dscl . -create /Users/username
sudo dscl . -create /Users/username UserShell /bin/bash
sudo dscl . -create /Users/username RealName "User Name"
sudo dscl . -create /Users/username UniqueID 501
sudo dscl . -create /Users/username PrimaryGroupID 20
sudo dscl . -create /Users/username NFSHomeDirectory /Users/username
sudo dscl . -passwd /Users/username

# Create home directory
sudo createhomedir -c -u username

# Disable guest account
sudo defaults write /Library/Preferences/com.apple.loginwindow GuestEnabled -bool false

# Show full name in login window instead of username
sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWFULLNAME -bool true

# Disable automatic login
sudo defaults delete /Library/Preferences/com.apple.loginwindow autoLoginUser
```

## DNS Security

### Install and Configure DNSCrypt

```bash
# Install via Homebrew
brew install dnscrypt-proxy

# Edit configuration
vim /usr/local/etc/dnscrypt-proxy.toml
```

DNSCrypt configuration (`/usr/local/etc/dnscrypt-proxy.toml`):

```toml
listen_addresses = ['127.0.0.1:53', '[::1]:53']
max_clients = 250

server_names = ['cloudflare', 'cloudflare-ipv6']

[query_log]
  file = '/var/log/dnscrypt-proxy/query.log'

[nx_log]
  file = '/var/log/dnscrypt-proxy/nx.log'

[sources]
  [sources.'public-resolvers']
    urls = ['https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v3/public-resolvers.md']
    cache_file = 'public-resolvers.md'
    minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
    refresh_delay = 72
    prefix = ''
```

Start DNSCrypt:

```bash
# Create log directory
sudo mkdir -p /var/log/dnscrypt-proxy

# Start service
sudo brew services start dnscrypt-proxy

# Configure system DNS to use localhost
networksetup -setdnsservers Wi-Fi 127.0.0.1
networksetup -setdnsservers Ethernet 127.0.0.1

# Test DNS resolution
dig @127.0.0.1 example.com
```

### Hosts File Configuration

```bash
# Edit hosts file
sudo vim /etc/hosts

# Add entries to block tracking domains
# Example entries:
# 0.0.0.0 ads.example.com
# 0.0.0.0 tracker.example.com

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

## System Services Hardening

### Disable Unnecessary Services

```bash
# Disable Bonjour multicast advertising
sudo defaults write /Library/Preferences/com.apple.mDNSResponder.plist NoMulticastAdvertisements -bool true

# Disable Captive Portal
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.captive.control.plist Active -bool false

# Disable Spotlight indexing
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist 2>/dev/null

# Disable Handoff
defaults write ~/Library/Preferences/ByHost/com.apple.coreservices.useractivityd.plist ActivityAdvertisingAllowed -bool no
defaults write ~/Library/Preferences/ByHost/com.apple.coreservices.useractivityd.plist ActivityReceivingAllowed -bool no

# Disable remote Apple events
sudo systemsetup -setremoteappleevents off

# Disable remote login (SSH) if not needed
sudo systemsetup -setremotelogin off

# Disable wake on network access
sudo systemsetup -setwakeonnetworkaccess off

# Disable Bluetooth if not needed
sudo defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 0
```

## Privacy Configuration

### Disable Telemetry and Tracking

```bash
# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false
launchctl disable user/com.apple.assistantd

# Disable Siri suggestions
defaults write com.apple.lookup.shared LookupSuggestionsDisabled -bool true

# Disable Spotlight Suggestions
defaults write com.apple.spotlight orderedItems -array \
  '{"enabled" = 0;"name" = "APPLICATIONS";}' \
  '{"enabled" = 0;"name" = "MENU_SPOTLIGHT_SUGGESTIONS";}'

# Disable diagnostic data
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.SubmitDiagInfo.plist

# Disable personalized ads
defaults write com.apple.AdLib allowApplePersonalizedAdvertising -bool false

# Clear advertising identifier
defaults write com.apple.AdLib allowIdentifierForAdvertising -bool false
```

### Location Services

```bash
# Disable location services entirely (if not needed)
sudo launchctl unload /System/Library/LaunchDaemons/com.apple.locationd.plist

# Or disable for specific apps via System Preferences > Security & Privacy > Privacy > Location Services
```

## Browser Hardening

### Firefox Configuration

Create/edit `user.js` in Firefox profile directory (`~/Library/Application Support/Firefox/Profiles/*.default-release/`):

```javascript
// Privacy settings
user_pref("privacy.trackingprotection.enabled", true);
user_pref("privacy.trackingprotection.socialtracking.enabled", true);
user_pref("privacy.trackingprotection.fingerprinting.enabled", true);
user_pref("privacy.trackingprotection.cryptomining.enabled", true);

// Disable telemetry
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("toolkit.telemetry.archive.enabled", false);
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);

// DNS over HTTPS
user_pref("network.trr.mode", 2);
user_pref("network.trr.uri", "https://cloudflare-dns.com/dns-query");

// Disable WebRTC IP leak
user_pref("media.peerconnection.enabled", false);

// HTTPS-only mode
user_pref("dom.security.https_only_mode", true);
user_pref("dom.security.https_only_mode_ever_enabled", true);

// Disable geolocation
user_pref("geo.enabled", false);

// Disable password manager
user_pref("signon.rememberSignons", false);

// Enhanced cookie protection
user_pref("privacy.firstparty.isolate", true);
```

### Safari Hardening

```bash
# Prevent cross-site tracking
defaults write com.apple.Safari WebKitPreferences.storageBlockingPolicy -int 1

# Disable autofill
defaults write com.apple.Safari AutoFillPasswords -bool false
defaults write com.apple.Safari AutoFillCreditCardData -bool false

# Block all cookies (strict mode)
defaults write com.apple.Safari BlockStoragePolicy -int 2

# Enable fraudulent website warnings
defaults write com.apple.Safari WarnAboutFraudulentWebsites -bool true

# Disable preloading top hit
defaults write com.apple.Safari PreloadTopHit -bool false

# Disable Safari suggestions
defaults write com.apple.Safari UniversalSearchEnabled -bool false
defaults write com.apple.Safari SuppressSearchSuggestions -bool true

# Show full URL
defaults write com.apple.Safari ShowFullURLInSmartSearchField -bool true
```

## Certificate Authority Management

```bash
# List all certificates
security find-certificate -a /System/Library/Keychains/SystemRootCertificates.keychain
security find-certificate -a /Library/Keychains/System.keychain

# List trusted root CAs
security dump-trust-settings

# Distrust a specific certificate (by hash)
sudo security add-trusted-cert -d -r deny -k /Library/Keychains/System.keychain /path/to/cert.pem

# Remove untrusted certificate authorities
# This should be done through Keychain Access app:
# Open Keychain Access > System Roots > Right-click cert > Get Info > Trust > Never Trust
```

## System Monitoring

### OpenBSM Audit

```bash
# Enable audit
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.auditd.plist

# Configure audit flags in /etc/security/audit_control
# Example: audit login, logout, authentication events
sudo vim /etc/security/audit_control

# Add to flags: lo,aa,ad,fd,fm,-all

# Restart auditd
sudo audit -s

# View audit logs
sudo praudit -xn /var/audit/*

# Query specific events (failed authentication)
sudo praudit /var/audit/* | grep "authentication failure"
```

### Network Monitoring

```bash
# Monitor network connections
lsof -i -P -n | grep LISTEN

# Monitor DNS queries (with dnscrypt-proxy)
tail -f /var/log/dnscrypt-proxy/query.log

# Monitor outgoing connections
sudo tcpdump -i en0 -n

# Network statistics
netstat -an | grep ESTABLISHED

# Use Little Snitch or LuLu for application-level firewall
# LuLu (open source):
# Download from https://objective-see.com/products/lulu.html
```

### Execution Monitoring

```bash
# Monitor process execution with DTrace
sudo dtrace -n 'proc:::exec-success { printf("%s launched by %s\n", execname, curpsinfo->pr_fname); }'

# Monitor file system events
sudo fs_usage -w | grep -v "0.000"

# Watch for new listening ports
sudo lsof -i -P | grep LISTEN

# Use KnockKnock to scan for persistent software
# Download from https://objective-see.com/products/knockknock.html
```

## Backup and Recovery

### Time Machine with Encryption

```bash
# Enable Time Machine encryption
sudo tmutil setdestination -a /Volumes/BackupDrive
sudo diskutil apfs enableFileVault /Volumes/BackupDrive

# Manual backup
tmutil startbackup

# List backups
tmutil listbackups

# Compare backups
tmutil compare

# Verify backup integrity
tmutil verifychecksums /path/to/backup
```

### Secure Backup Script

```bash
#!/bin/bash
# secure-backup.sh - Encrypted backup script

BACKUP_DIR="/Volumes/ExternalDrive/Backups"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_NAME="backup_${DATE}.tar.gz.gpg"

# Create encrypted backup
tar czf - ~/Documents ~/Projects | \
  gpg --symmetric --cipher-algo AES256 --compress-algo none \
  --output "${BACKUP_DIR}/${BACKUP_NAME}"

# Verify backup was created
if [ -f "${BACKUP_DIR}/${BACKUP_NAME}" ]; then
  echo "Backup created: ${BACKUP_NAME}"
  shasum -a 256 "${BACKUP_DIR}/${BACKUP_NAME}" > "${BACKUP_DIR}/${BACKUP_NAME}.sha256"
else
  echo "Backup failed!"
  exit 1
fi

# Remove backups older than 30 days
find "${BACKUP_DIR}" -name "backup_*.tar.gz.gpg" -mtime +30 -delete
```

## SSH Hardening

### Configure Secure SSH

Edit `/etc/ssh/sshd_config` (if SSH is enabled):

```bash
# Disable root login
PermitRootLogin no

# Use public key authentication only
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding
X11Forwarding no

# Use strong ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org

# Limit authentication attempts
MaxAuthTries 3
MaxSessions 2

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2
```

Generate secure SSH keys:

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519

# Or RSA 4096-bit
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# Set proper permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

## Password Management

### Use macOS Keychain from Command Line

```bash
# Add password to keychain
security add-generic-password -a "$USER" -s "ServiceName" -w "password"

# Retrieve password
security find-generic-password -a "$USER" -s "ServiceName" -w

# Generate strong password
openssl rand -base64 32

# Generate diceware passphrase (requires word list)
shuf -n 6 /usr/share/dict/words | tr '\n' ' ' && echo
```

## Physical Security

```bash
# Require password immediately after sleep/screensaver
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

# Enable screen saver with timeout
defaults -currentHost write com.apple.screensaver idleTime -int 300

# Show message on lock screen
sudo defaults write /Library/Preferences/com.apple.loginwindow LoginwindowText "If found, please contact: email@example.com"

# Disable automatic login
sudo defaults delete /Library/Preferences/com.apple.loginwindow autoLoginUser

# Log out after inactivity (3600 seconds = 1 hour)
sudo defaults write /Library/Preferences/.GlobalPreferences com.apple.autologout.AutoLogOutDelay -int 3600
```

## Metadata Removal

```bash
# Remove metadata from files
exiftool -all= file.jpg

# Or use mdls/xattr
xattr -l file.pdf
xattr -c file.pdf  # Clear all extended attributes

# Remove location data from photos
exiftool -gps:all= photo.jpg

# Secure file deletion (on APFS, overwriting is not effective)
# Instead, use encrypted volumes and delete normally
rm -P file.txt  # Note: -P is deprecated on APFS
```

## Troubleshooting

### FileVault Issues

```bash
# Check FileVault status
sudo fdesetup status

# Validate FileVault recovery key
sudo fdesetup validaterecovery

# Fix FileVault if it won't enable
sudo fdesetup disable
sudo diskutil apfs list
sudo fdesetup enable
```

### Firewall Debugging

```bash
# Check pf status
sudo pfctl -si

# View pf logs
sudo log show --predicate 'process == "kernel" AND eventMessage CONTAINS "pf"' --last 1h

# Test firewall rules
sudo pfctl -vvn -f /etc/pf.conf

# Clear pf statistics
sudo pfctl -F all

# Application firewall logs
log show --predicate 'process == "socketfilterfw"' --last 1h
```

### DNS Resolution Issues

```bash
# Test DNS resolution
dig example.com @127.0.0.1

# Check dnscrypt-proxy status
sudo brew services list | grep dnscrypt-proxy

# View dnscrypt-proxy logs
tail -f /usr/local/var/log/dnscrypt-proxy.log

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Reset DNS settings
networksetup -setdnsservers Wi-Fi "Empty"
networksetup -setdnsservers Wi-Fi 127.0.0.1
```

### Permission Issues

```bash
# Reset permissions on home directory
sudo chown -R $USER:staff ~
chmod 755 ~

# Fix SIP-protected files (requires disabling SIP in Recovery Mode)
# Only do this if absolutely necessary
csrutil status
# Reboot to Recovery Mode, then:
# csrutil disable
# Reboot, make changes, then re-enable
# csrutil enable

# Reset keychain permissions
security unlock-keychain ~/Library/Keychains/login.keychain-db
```

## Common Patterns

### Security Audit Script

```bash
#!/bin/bash
# security-audit.sh - Basic macOS security audit

echo "=== macOS Security Audit ==="
echo ""

echo "FileVault Status:"
sudo fdesetup status
echo ""

echo "Firmware Password:"
sudo firmwarepasswd -check
echo ""

echo "Firewall Status:"
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
echo ""

echo "System Integrity Protection:"
csrutil status
echo ""

echo "Gatekeeper Status:"
spctl --status
echo ""

echo "Automatic Updates:"
sudo softwareupdate --schedule
echo ""

echo "Listening Ports:"
lsof -i -P -n | grep LISTEN
echo ""

echo "Remote Login (SSH):"
sudo systemsetup -getremotelogin
echo ""

echo "Screen Saver Password:"
defaults read com.apple.screensaver askForPassword
echo ""

echo "Installed Applications (not from App Store):"
system_profiler SPApplicationsDataType | grep -B 3 "Obtained from: Identified Developer"
```

### Security Hardening Script

```bash
#!/bin/bash
# harden-macos.sh - Automated security hardening

set -e

echo "Applying macOS security hardening..."

# Enable FileVault
if ! sudo fdesetup status | grep -q "On"; then
  echo "Enabling FileVault..."
  sudo fdesetup enable -user "$USER"
fi

# Enable Firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Disable Bonjour
sudo defaults write /Library/Preferences/com.apple.mDNSResponder.plist NoMulticastAdvertisements -bool true

# Require password immediately
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

# Disable guest account
sudo defaults write /Library/Preferences/com.apple.loginwindow GuestEnabled -bool false

# Show full path in Finder
defaults write com.apple.finder _FXShowPosixPathInTitle -bool true

# Enable software updates
sudo softwareupdate --schedule on

# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false

# Disable remote services
sudo systemsetup -setremoteappleevents off
sudo systemsetup -setremotelogin off

echo "Security hardening complete. Restart required for some changes."
```

## Best Practices

1. **Regular Updates**: Keep macOS and all software updated
2. **Principle of Least Privilege**: Use standard user account for daily work, admin only when needed
3. **Encryption Everywhere**: Enable FileVault, encrypt backups, use HTTPS/TLS
4. **Network Security**: Use VPN on untrusted networks, configure firewall properly
5. **Strong Authentication**: Use long passphrases, enable 2FA where possible
6. **Backup Strategy**: Regular encrypted backups to offline storage
7. **Privacy First**: Disable telemetry, use privacy-focused alternatives
8. **Monitor System**: Regularly check logs and running processes
9. **Physical Security**: Never leave machine unlocked, use firmware password
10. **Threat Modeling**: Regularly reassess your threat model and adjust defenses

## Additional Resources

- Official NIST macOS Security Guide: https://github.com/usnistgov/macos_security
- Objective-See Security Tools: https://objective-see.com/products.html
- Apple Platform Security Guide: https://support.apple.com/guide/security/welcome/web
- macOS Security Updates: https://support.apple.com/HT201222
