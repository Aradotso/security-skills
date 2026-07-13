---
name: macos-security-privacy-hardening
description: Comprehensive security and privacy hardening guide for macOS systems with Apple silicon
triggers:
  - "How do I secure my Mac?"
  - "Configure macOS security settings"
  - "Harden macOS privacy"
  - "Set up FileVault and firewall on Mac"
  - "Improve macOS security posture"
  - "Follow macOS security best practices"
  - "Lock down macOS system"
  - "Configure macOS for maximum privacy"
---

# macOS Security and Privacy Hardening

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This guide provides comprehensive security and privacy hardening techniques for Apple silicon Mac computers running currently supported versions of macOS. It covers enterprise-standard security configurations suitable for both power users and organizations.

## Overview

The macOS Security and Privacy Guide is a community-maintained collection of:

- **Threat modeling frameworks** for identifying and mitigating risks
- **System hardening configurations** for securing macOS at multiple layers
- **Privacy protection techniques** to minimize data leakage
- **Security tools and utilities** for monitoring and protection
- **Best practices** for secure Mac administration

**Important**: This guide targets Apple silicon Macs. Intel-based Macs have hardware-level vulnerabilities (like checkm8) that cannot be patched.

## Accessing the Guide

The guide is available at: https://drduh.github.io/macOS-Security-and-Privacy-Guide/

Or clone the repository:

```bash
git clone https://github.com/drduh/macOS-Security-and-Privacy-Guide.git
cd macOS-Security-and-Privacy-Guide
```

## Core Security Concepts

### Threat Modeling

Before applying any configurations, create a threat model:

1. **Identify Assets**: What needs protection (passwords, files, communications)
2. **Identify Adversaries**: Who might attack (roommate, thief, corporation, nation-state)
3. **Identify Capabilities**: What attackers can do (physical access, network sniffing, malware)
4. **Identify Mitigations**: How to counter each threat

Example threat model table:

```
Adversary       | Motivation        | Capabilities              | Mitigation
----------------|-------------------|---------------------------|---------------------------
Roommate        | Curiosity         | Physical proximity        | Use biometrics, lock screen
Thief           | Financial gain    | Device theft              | FileVault, Find My, strong password
Criminal        | Financial         | Malware, phishing         | Sandbox, updates, 2FA
Corporation     | Data collection   | Telemetry, tracking       | Block connections, reset IDs
Nation State    | Surveillance      | Advanced exploitation     | E2EE, secure boot, monitoring
```

## Essential Security Commands

### System Updates

```bash
# Check for updates
softwareupdate --list

# Install all available updates
sudo softwareupdate --install --all

# Enable automatic updates
sudo softwareupdate --schedule on

# Download and install automatically
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticCheckEnabled -bool true
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticDownload -bool true
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticallyInstallMacOSUpdates -bool true
```

### FileVault (Full Disk Encryption)

```bash
# Check FileVault status
sudo fdesetup status

# Enable FileVault (requires restart)
sudo fdesetup enable

# List FileVault-enabled users
sudo fdesetup list

# Add user to FileVault
sudo fdesetup add -usertoadd username
```

### Firewall Configuration

```bash
# Enable application firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable stealth mode (ignore ICMP ping)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Enable logging
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Block all incoming connections
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setblockall on

# Prevent built-in software from being auto-allowed
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsigned off

# Check firewall status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Packet Filtering (PF)

Create `/etc/pf.conf` for advanced firewall rules:

```bash
# Basic PF configuration
# Block all incoming by default, allow outgoing
block in all
pass out all keep state

# Allow loopback
pass on lo0 all

# Allow established connections
pass in proto tcp from any to any flags S/SA keep state
pass in proto udp from any to any keep state

# Allow specific services (SSH example)
# pass in proto tcp from any to any port 22 keep state
```

Enable PF:

```bash
# Load PF rules
sudo pfctl -f /etc/pf.conf

# Enable PF
sudo pfctl -e

# Check PF status
sudo pfctl -s info

# View active rules
sudo pfctl -s rules

# Disable PF
sudo pfctl -d
```

### User Account Management

```bash
# Create non-admin user account
sudo dscl . -create /Users/username
sudo dscl . -create /Users/username UserShell /bin/zsh
sudo dscl . -create /Users/username RealName "Full Name"
sudo dscl . -create /Users/username UniqueID 1001
sudo dscl . -create /Users/username PrimaryGroupID 20
sudo dscl . -create /Users/username NFSHomeDirectory /Users/username
sudo dscl . -passwd /Users/username

# Create home directory
sudo createhomedir -c -u username

# Add user to admin group (if needed)
sudo dscl . -append /Groups/admin GroupMembership username

# List all users
dscl . list /Users

# Check if user is admin
dscl . -read /Groups/admin GroupMembership
```

## Privacy Configuration

### Disable Telemetry and Analytics

```bash
# Disable diagnostic and usage data
sudo defaults write /Library/Application\ Support/CrashReporter/DiagnosticMessagesHistory.plist AutoSubmit -bool false
sudo defaults write /Library/Application\ Support/CrashReporter/DiagnosticMessagesHistory.plist ThirdPartyDataSubmit -bool false

# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false
defaults write com.apple.Siri StatusMenuVisible -bool false

# Disable personalized ads
defaults write com.apple.AdLib allowApplePersonalizedAdvertising -bool false

# Disable Safari suggestions
defaults write com.apple.Safari UniversalSearchEnabled -bool false
defaults write com.apple.Safari SuppressSearchSuggestions -bool true

# Disable Spotlight suggestions
defaults write com.apple.lookup.shared LookupSuggestionsDisabled -bool true
```

### Location Services

```bash
# Disable location services entirely
sudo defaults write /var/db/locationd/Library/Preferences/ByHost/com.apple.locationd LocationServicesEnabled -int 0

# Check location services status
defaults read /var/db/locationd/Library/Preferences/ByHost/com.apple.locationd
```

### DNS Configuration

Install and configure DNSCrypt for encrypted DNS:

```bash
# Install via Homebrew
brew install dnscrypt-proxy

# Configure DNSCrypt
sudo cp /usr/local/etc/dnscrypt-proxy.toml.example /usr/local/etc/dnscrypt-proxy.toml

# Edit configuration
sudo nano /usr/local/etc/dnscrypt-proxy.toml
```

Example DNSCrypt configuration:

```toml
server_names = ['cloudflare', 'cloudflare-ipv6']
listen_addresses = ['127.0.0.1:53', '[::1]:53']
max_clients = 250
ipv4_servers = true
ipv6_servers = true
dnscrypt_servers = true
doh_servers = true
require_dnssec = true
require_nolog = true
require_nofilter = true
force_tcp = false
timeout = 5000
keepalive = 30
cert_refresh_delay = 240
```

Start DNSCrypt:

```bash
# Start service
sudo brew services start dnscrypt-proxy

# Configure system to use DNSCrypt
sudo networksetup -setdnsservers Wi-Fi 127.0.0.1
sudo networksetup -setdnsservers Ethernet 127.0.0.1

# Verify DNS settings
scutil --dns
```

### Hosts File Blocking

Add tracking/malware domains to `/etc/hosts`:

```bash
# Backup existing hosts file
sudo cp /etc/hosts /etc/hosts.backup

# Download and merge hosts file (example using StevenBlack's list)
curl https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts | sudo tee -a /etc/hosts

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

## Security Monitoring

### OpenBSM Audit

```bash
# Enable auditing
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.auditd.plist

# Check audit status
sudo audit -s

# View audit logs
sudo praudit -xn /var/audit/*

# Configure audit policy
sudo nano /etc/security/audit_control
```

Example `audit_control` configuration:

```
dir:/var/audit
flags:lo,aa,ad,fd,fm,-all
minfree:5
naflags:lo,aa,ad,fd,fm,-all
policy:cnt,argv
filesz:10M
expire-after:90d
```

### System Monitoring Scripts

Monitor process execution:

```bash
#!/bin/bash
# monitor_processes.sh - Log new process execution

while true; do
    ps axo lstart,uid,pid,ppid,comm | tail -n +2 | \
    awk '{$1=$2=$3=$4=""; print substr($0,5)}' | \
    while read line; do
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $line" >> /var/log/process_monitor.log
    done
    sleep 1
done
```

Monitor network connections:

```bash
#!/bin/bash
# monitor_network.sh - Log network connections

lsof -i -n -P | grep ESTABLISHED | \
awk '{print $1, $2, $3, $8, $9}' | \
while read line; do
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $line" >> /var/log/network_monitor.log
done
```

### DTrace Monitoring

Monitor file system access:

```bash
# Monitor file opens
sudo dtrace -n 'syscall::open*:entry { printf("%s %s", execname, copyinstr(arg0)); }'

# Monitor network connections
sudo dtrace -n 'syscall::connect:entry { printf("%s", execname); }'

# Monitor process execution
sudo dtrace -n 'proc:::exec-success { trace(curpsinfo->pr_psargs); }'
```

## Browser Hardening

### Firefox Configuration

Create `user.js` in Firefox profile directory (`~/Library/Application Support/Firefox/Profiles/*.default-release/`):

```javascript
// Disable telemetry
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("toolkit.telemetry.archive.enabled", false);

// Enable tracking protection
user_pref("privacy.trackingprotection.enabled", true);
user_pref("privacy.trackingprotection.socialtracking.enabled", true);

// Disable WebRTC (prevents IP leak)
user_pref("media.peerconnection.enabled", false);

// Enable HTTPS-Only Mode
user_pref("dom.security.https_only_mode", true);
user_pref("dom.security.https_only_mode_ever_enabled", true);

// Disable password manager
user_pref("signon.rememberSignons", false);

// Disable form autofill
user_pref("browser.formfill.enable", false);

// Enhanced fingerprinting protection
user_pref("privacy.resistFingerprinting", true);

// Disable geolocation
user_pref("geo.enabled", false);

// Clear on shutdown
user_pref("privacy.sanitize.sanitizeOnShutdown", true);
user_pref("privacy.clearOnShutdown.cache", true);
user_pref("privacy.clearOnShutdown.cookies", true);
user_pref("privacy.clearOnShutdown.history", true);
```

### Safari Configuration

```bash
# Disable auto-fill
defaults write com.apple.Safari AutoFillPasswords -bool false
defaults write com.apple.Safari AutoFillCreditCardData -bool false

# Enable "Do Not Track"
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true

# Disable preloading top hit
defaults write com.apple.Safari PreloadTopHit -bool false

# Block pop-ups
defaults write com.apple.Safari WebKitJavaScriptCanOpenWindowsAutomatically -bool false
defaults write com.apple.Safari com.apple.Safari.ContentPageGroupIdentifier.WebKit2JavaScriptCanOpenWindowsAutomatically -bool false

# Enable fraud warnings
defaults write com.apple.Safari WarnAboutFraudulentWebsites -bool true

# Disable plugins
defaults write com.apple.Safari WebKitPluginsEnabled -bool false

# Show full URL
defaults write com.apple.Safari ShowFullURLInSmartSearchField -bool true

# Clear history items older than 1 day
defaults write com.apple.Safari HistoryAgeInDaysLimit -int 1
```

## SSH Hardening

Create/edit `~/.ssh/config`:

```ssh-config
# Default settings for all hosts
Host *
    # Use strong key exchange algorithms
    KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
    
    # Use strong ciphers
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
    
    # Use strong MACs
    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
    
    # Disable password authentication
    PasswordAuthentication no
    ChallengeResponseAuthentication no
    
    # Enable compression
    Compression yes
    
    # Connection settings
    ServerAliveInterval 60
    ServerAliveCountMax 3
    
    # Use SSH keys only
    PubkeyAuthentication yes
    
    # Hash known hosts for privacy
    HashKnownHosts yes
```

Edit SSH daemon config `/etc/ssh/sshd_config`:

```bash
# Disable root login
PermitRootLogin no

# Key-based authentication only
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Limit authentication attempts
MaxAuthTries 3

# Disconnect after failed attempts
MaxSessions 2

# Use protocol 2 only
Protocol 2

# Strong ciphers only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# Disable X11 forwarding
X11Forwarding no

# Log verbosely
LogLevel VERBOSE
```

Generate strong SSH keys:

```bash
# Ed25519 key (recommended)
ssh-keygen -t ed25519 -a 100 -C "user@host"

# RSA key (if Ed25519 not supported)
ssh-keygen -t rsa -b 4096 -o -a 100 -C "user@host"

# Set proper permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys
```

## Encryption and Key Management

### GPG Configuration

```bash
# Install GPG
brew install gnupg

# Generate key
gpg --full-generate-key
# Choose: (1) RSA and RSA, 4096 bits, does not expire

# List keys
gpg --list-keys
gpg --list-secret-keys

# Export public key
gpg --armor --export user@example.com > public_key.asc

# Encrypt file
gpg --encrypt --recipient user@example.com file.txt

# Decrypt file
gpg --decrypt file.txt.gpg > file.txt

# Sign file
gpg --sign file.txt

# Verify signature
gpg --verify file.txt.gpg
```

GPG configuration (`~/.gnupg/gpg.conf`):

```
# Strong preferences
personal-cipher-preferences AES256 AES192 AES
personal-digest-preferences SHA512 SHA384 SHA256
personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed
default-preference-list SHA512 SHA384 SHA256 AES256 AES192 AES ZLIB BZIP2 ZIP Uncompressed

# Use strong hash
cert-digest-algo SHA512
s2k-digest-algo SHA512
s2k-cipher-algo AES256

# Long key IDs
keyid-format 0xlong
with-fingerprint

# No version in output
no-emit-version
no-comments

# Use agent
use-agent
```

### Password Management

```bash
# Generate strong password
openssl rand -base64 32

# Generate diceware passphrase
# Install diceware: brew install diceware
diceware --num 6 --wordlist en_eff

# Store in Keychain
security add-generic-password -a "$USER" -s "service_name" -w "password"

# Retrieve from Keychain
security find-generic-password -a "$USER" -s "service_name" -w

# Delete from Keychain
security delete-generic-password -a "$USER" -s "service_name"
```

## Wi-Fi Security

```bash
# Disable Wi-Fi if not needed
networksetup -setairportpower en0 off

# Forget network
sudo /System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -z

# Disable auto-join for network
networksetup -setnetworkserviceenabled "Wi-Fi" off

# Use randomized MAC (requires manual setting in System Preferences)
# Check current MAC
ifconfig en0 | grep ether

# Set static MAC temporarily
sudo ifconfig en0 ether aa:bb:cc:dd:ee:ff

# Scan networks
/System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -s
```

## System Integrity and Hardening

### System Integrity Protection (SIP)

```bash
# Check SIP status
csrutil status

# Disable SIP (requires Recovery Mode)
# Boot to Recovery (Command+R), then:
# csrutil disable

# Enable SIP (recommended)
# Boot to Recovery (Command+R), then:
# csrutil enable
```

### Gatekeeper

```bash
# Check Gatekeeper status
spctl --status

# Enable Gatekeeper
sudo spctl --master-enable

# Check app signature
spctl -a -v /Applications/AppName.app

# Allow unsigned app (not recommended)
sudo spctl --add /Applications/AppName.app
```

### Disable Services

```bash
# Disable AirDrop
defaults write com.apple.NetworkBrowser DisableAirDrop -bool YES

# Disable Bonjour
sudo defaults write /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist ProgramArguments -array-add "-NoMulticastAdvertisements"

# Disable Captive Portal
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.captive.control Active -bool false

# Disable IR receiver
sudo defaults write /Library/Preferences/com.apple.driver.AppleIRController DeviceEnabled -int 0

# Disable Bluetooth
sudo defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 0
```

## Backup and Recovery

### Time Machine

```bash
# Enable Time Machine
sudo tmutil enable

# Set destination
sudo tmutil setdestination /Volumes/BackupDrive

# Start backup
sudo tmutil startbackup

# Exclude directory
sudo tmutil addexclusion /path/to/exclude

# List exclusions
sudo tmutil isexcluded /path/to/check

# Disable local snapshots
sudo tmutil disablelocal
```

### Encrypted Backups

```bash
# Create encrypted disk image
hdiutil create -size 100g -fs APFS -encryption AES-256 -volname Backup ~/backup.dmg

# Mount encrypted image
hdiutil attach ~/backup.dmg

# Backup with rsync
rsync -avz --delete --exclude-from=exclude.txt ~/ /Volumes/Backup/

# Unmount
hdiutil detach /Volumes/Backup
```

## Metadata Removal

```bash
# Remove extended attributes
xattr -cr /path/to/file

# Remove metadata from image
exiftool -all= image.jpg

# Remove metadata from PDF
exiftool -all:all= document.pdf

# Remove .DS_Store files
find . -name ".DS_Store" -delete

# Disable .DS_Store creation on network volumes
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true

# Disable .DS_Store creation on USB volumes
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true
```

## Common Troubleshooting

### Firewall Issues

If applications can't connect after enabling firewall:

```bash
# Check application firewall status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getappblocked /Applications/App.app

# Allow specific application
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /Applications/App.app
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /Applications/App.app
```

### DNS Resolution Problems

```bash
# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Check DNS servers
scutil --dns

# Test DNS resolution
nslookup example.com
dig example.com

# Reset DNS to DHCP
sudo networksetup -setdnsservers Wi-Fi "Empty"
```

### FileVault Recovery

```bash
# Get recovery key
sudo fdesetup changerecovery -personal

# Disable and re-enable if having issues
sudo fdesetup disable
sudo fdesetup enable
```

### Permission Issues

```bash
# Reset permissions on home directory
sudo chown -R $USER:staff ~/
chmod -R u+rwX,go-rwx ~/

# Reset Keychain permissions
sudo chmod 600 ~/Library/Keychains/*.keychain-db

# Repair disk permissions (pre-El Capitan)
sudo diskutil resetUserPermissions / $(id -u)
```

## Security Checklist

Quick checklist for securing a new Mac:

```bash
#!/bin/bash
# security_setup.sh - Initial Mac security setup

# Update system
sudo softwareupdate --install --all

# Enable FileVault
sudo fdesetup enable

# Enable firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Disable services
defaults write com.apple.NetworkBrowser DisableAirDrop -bool YES
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.captive.control Active -bool false

# Configure Safari
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true
defaults write com.apple.Safari AutoFillPasswords -bool false

# Disable telemetry
sudo defaults write /Library/Application\ Support/CrashReporter/DiagnosticMessagesHistory.plist AutoSubmit -bool false

# Set firmware password (requires restart)
sudo firmwarepasswd -setpasswd

echo "Basic security setup complete. Restart required."
```

## Additional Resources

- Official Guide: https://drduh.github.io/macOS-Security-and-Privacy-Guide/
- Apple Security Guide: https://support.apple.com/guide/security/welcome/web
- NIST macOS Guidelines: https://github.com/usnistgov/macos_security
- CIS Benchmarks: https://www.cisecurity.org/benchmark/apple_os

## Related Tools

Install security utilities via Homebrew:

```bash
# Security scanning
brew install lynis              # Security auditing
brew install rkhunter          # Rootkit detection
brew install chkrootkit        # Rootkit checker

# Network security
brew install nmap              # Network scanning
brew install wireshark         # Packet analysis
brew install mtr               # Network diagnostics

# Encryption
brew install gnupg             # GPG encryption
brew install openssl           # Crypto library
brew install age               # File encryption

# Monitoring
brew install htop              # Process monitoring
brew install iftop             # Network monitoring
brew install osquery           # System query tool
```

This skill provides comprehensive coverage of macOS security hardening based on the drduh guide, with practical commands and configurations ready for immediate use.
