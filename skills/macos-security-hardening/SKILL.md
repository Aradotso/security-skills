---
name: macos-security-hardening
description: Comprehensive guide for securing and hardening macOS systems with privacy-focused configurations, firewall rules, and security best practices
triggers:
  - how do I secure my macOS system
  - improve privacy on Mac
  - harden macOS security settings
  - configure macOS firewall and encryption
  - setup secure macOS environment
  - macOS security best practices
  - lock down macOS for privacy
  - enterprise security for Mac
---

# macOS Security Hardening

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive guidance for securing macOS systems based on the community-maintained drduh/macOS-Security-and-Privacy-Guide. It covers system hardening, encryption, firewall configuration, DNS security, and privacy enhancements for both Apple silicon and Intel Macs.

## Overview

The macOS Security and Privacy Guide is a collection of battle-tested techniques for:

- **Full disk encryption** with FileVault
- **Firewall configuration** (application-level and packet filtering)
- **DNS security** with DNSCrypt and encrypted DNS profiles
- **Privacy hardening** by disabling telemetry and tracking
- **Secure browsing** configurations for Firefox, Chrome, and Safari
- **System monitoring** with OpenBSM and DTrace
- **Physical security** measures and admin account separation

**Important**: Apple silicon Macs are strongly recommended over Intel Macs due to hardware-level security vulnerabilities in Intel CPUs that cannot be patched.

## Threat Modeling Framework

Before applying any security measures, create a threat model:

### Asset Identification

```bash
# List your critical assets:
# - Passwords and credentials
# - Financial data
# - Private communications
# - Work documents
# - Personal photos/videos
```

### Adversary Analysis Template

| Adversary | Motivation | Capabilities | Mitigation |
|-----------|-----------|--------------|------------|
| Roommate | Curiosity | Physical access, screen viewing | Use FileVault, auto-lock screen, privacy filters |
| Thief | Financial gain | Device theft, shoulder surfing | Find My Mac, strong passwords, remote wipe |
| Criminal | Data theft | Malware, phishing, exploits | Gatekeeper, firewall, updated software |
| Corporation | Data collection | Telemetry, tracking | Block telemetry, DNS filtering, VPN |
| Nation State | Surveillance | Advanced exploits, traffic analysis | Full encryption, Tor, air-gapped backups |

## System Updates

Keep macOS and all software current:

```bash
# Check for system updates
softwareupdate --list

# Install all available updates
sudo softwareupdate --install --all

# Enable automatic updates
sudo softwareupdate --schedule on

# Install security updates only
sudo softwareupdate --install --recommended
```

## FileVault Full Disk Encryption

FileVault encrypts your entire disk using XTS-AES-128 with a 256-bit key.

```bash
# Check FileVault status
sudo fdesetup status

# Enable FileVault (GUI method recommended for recovery key)
sudo fdesetup enable

# List FileVault enabled users
sudo fdesetup list

# Add user to FileVault
sudo fdesetup add -usertoadd username

# Change FileVault password
sudo fdesetup changerecovery -personal
```

**Important**: Store your recovery key in a secure location separate from your Mac (e.g., password manager, safe).

### Hibernation Mode for Enhanced Security

```bash
# Show current hibernation mode
pmset -g | grep hibernatemode

# Set hibernation mode 25 (secure - clears memory keys)
sudo pmset -a hibernatemode 25
sudo pmset -a destroyfvkeyonstandby 1
sudo pmset -a standby 0
sudo pmset -a autopoweroff 0

# Require password immediately after sleep
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0
```

## Admin and User Account Separation

Create separate admin and standard user accounts for daily use:

```bash
# Create standard user account (via System Preferences recommended)
# Or via command line:
sudo dscl . -create /Users/standarduser
sudo dscl . -create /Users/standarduser UserShell /bin/bash
sudo dscl . -create /Users/standarduser RealName "Standard User"
sudo dscl . -create /Users/standarduser UniqueID 503
sudo dscl . -create /Users/standarduser PrimaryGroupID 20
sudo dscl . -create /Users/standarduser NFSHomeDirectory /Users/standarduser
sudo dscl . -passwd /Users/standarduser

# Verify user is not admin
dsmemberutil checkmembership -U standarduser -G admin

# Disable root account
sudo dsenableroot -d
```

## Firmware Security

```bash
# Check firmware password status
sudo firmwarepasswd -check

# Set firmware password (prevents booting from external media)
sudo firmwarepasswd -setpasswd

# Verify secure boot status (Apple silicon)
csrutil status

# Check system integrity protection
csrutil status

# View security mode (Apple silicon)
# In Recovery Mode:
csrutil authenticated-root status
```

## Firewall Configuration

### Application Layer Firewall

```bash
# Enable application firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable logging
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Enable stealth mode (don't respond to pings)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Block all incoming connections
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setblockall on

# Allow signed applications
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsigned on

# Check status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Packet Filtering with pf

Create `/etc/pf.conf`:

```bash
# /etc/pf.conf - Packet Filter Configuration

# Interfaces
wifi = "en0"
lan = "en1"

# Block all incoming by default
set block-policy drop
set skip on lo0

# Scrub packets
scrub in all no-df

# Default deny
block log all

# Allow outgoing connections
pass out quick on $wifi inet keep state
pass out quick on $lan inet keep state

# Allow essential services
pass in quick on $wifi proto icmp icmp-type { echoreq, unreach }
pass in quick on $wifi proto tcp from any to any port 22 keep state  # SSH (if needed)

# Allow established connections
pass in quick on $wifi proto { tcp, udp } from any to any keep state
```

Enable and load pf:

```bash
# Test configuration
sudo pfctl -nf /etc/pf.conf

# Enable and load firewall
sudo pfctl -ef /etc/pf.conf

# Check status
sudo pfctl -s all

# View statistics
sudo pfctl -s info

# Reload rules
sudo pfctl -f /etc/pf.conf

# Disable
sudo pfctl -d
```

## DNS Security

### Encrypted DNS Profiles

Install DNS over HTTPS/TLS profile:

```bash
# Download Cloudflare DNS profile
curl -o ~/Downloads/Cloudflare-1.1.1.1.mobileconfig \
  https://1.1.1.1/Cloudflare-1.1.1.1.mobileconfig

# Or Quad9
curl -o ~/Downloads/Quad9-DoH.mobileconfig \
  https://www.quad9.net/support/set-up-guides/macos/Quad9-DoH.mobileconfig

# Install via System Preferences > Profiles
open ~/Downloads/Cloudflare-1.1.1.1.mobileconfig
```

### Hosts File Blocking

```bash
# Backup original hosts file
sudo cp /etc/hosts /etc/hosts.backup

# Download and install hosts file (blocks ads/tracking)
curl -o /tmp/hosts https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
sudo cp /tmp/hosts /etc/hosts

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

### DNSCrypt Proxy

```bash
# Install via Homebrew
brew install dnscrypt-proxy

# Configure /usr/local/etc/dnscrypt-proxy.toml
cat > /usr/local/etc/dnscrypt-proxy.toml << 'EOF'
server_names = ['cloudflare', 'quad9-dnscrypt-ip4-nofilter-pri']
listen_addresses = ['127.0.0.1:53']
max_clients = 250
ipv4_servers = true
ipv6_servers = false
dnscrypt_servers = true
doh_servers = true
require_dnssec = true
require_nolog = true
require_nofilter = false
force_tcp = false
timeout = 2500
keepalive = 30
log_level = 2
use_syslog = true
cache = true
cache_size = 512
cache_min_ttl = 600
cache_max_ttl = 86400
cache_neg_ttl = 60
EOF

# Start service
sudo brew services start dnscrypt-proxy

# Configure macOS to use local DNS
networksetup -setdnsservers Wi-Fi 127.0.0.1
networksetup -setdnsservers Ethernet 127.0.0.1

# Verify
scutil --dns | grep "nameserver"
```

## Privacy Hardening

### Disable Telemetry and Tracking

```bash
# Disable Spotlight Suggestions
defaults write com.apple.safari UniversalSearchEnabled -bool false
defaults write com.apple.safari SuppressSearchSuggestions -bool true

# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false
launchctl disable "user/$UID/com.apple.assistantd"

# Disable personalized ads
defaults write com.apple.AdLib allowApplePersonalizedAdvertising -bool false

# Disable crash reporting
defaults write com.apple.CrashReporter DialogType none

# Disable diagnostic data
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.SubmitDiagInfo.plist

# Disable Handoff
defaults write com.apple.coreservices.useractivityd ActivityAdvertisingAllowed -bool false
defaults write com.apple.coreservices.useractivityd ActivityReceivingAllowed -bool false

# Disable location services for system services
sudo defaults write /var/db/locationd/Library/Preferences/ByHost/com.apple.locationd \
  LocationServicesEnabled -bool false

# Safari privacy settings
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true
defaults write com.apple.Safari WebKitStorageBlockingPolicy -int 1
defaults write com.apple.Safari BlockStoragePolicy -int 2
```

### Disable Unnecessary Services

```bash
# Disable guest account
sudo dscl . -delete /Users/Guest
sudo defaults write /Library/Preferences/com.apple.AppleFileServer guestAccess -bool false
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.smb.server AllowGuestAccess -bool false

# Disable remote Apple Events
sudo systemsetup -setremoteappleevents off

# Disable Internet Sharing
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.nat NAT -dict Enabled -int 0

# Disable screen sharing
sudo launchctl disable system/com.apple.screensharing

# Disable printer sharing
cupsctl --no-share-printers

# Disable Bluetooth (if not needed)
sudo defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 0
```

## Browser Hardening

### Firefox Configuration

Create `user.js` in Firefox profile directory (`~/Library/Application Support/Firefox/Profiles/*.default-release/`):

```javascript
// Privacy settings
user_pref("privacy.trackingprotection.enabled", true);
user_pref("privacy.trackingprotection.socialtracking.enabled", true);
user_pref("privacy.donottrackheader.enabled", true);
user_pref("privacy.resistFingerprinting", true);
user_pref("privacy.firstparty.isolate", true);

// Disable telemetry
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("datareporting.healthreport.uploadEnabled", false);

// DNS over HTTPS
user_pref("network.trr.mode", 2);
user_pref("network.trr.uri", "https://cloudflare-dns.com/dns-query");

// Security
user_pref("security.ssl.require_safe_negotiation", true);
user_pref("security.tls.version.min", 3);
user_pref("security.cert_pinning.enforcement_level", 2);

// WebRTC
user_pref("media.peerconnection.enabled", false);

// Auto-updates
user_pref("app.update.auto", true);
```

### Safari Hardening

```bash
# Enable Develop menu
defaults write com.apple.Safari IncludeDevelopMenu -bool true

# Warn about fraudulent websites
defaults write com.apple.Safari WarnAboutFraudulentWebsites -bool true

# Block pop-ups
defaults write com.apple.Safari WebKitJavaScriptCanOpenWindowsAutomatically -bool false

# Update extensions automatically
defaults write com.apple.Safari InstallExtensionUpdatesAutomatically -bool true

# Show full URL
defaults write com.apple.Safari ShowFullURLInSmartSearchField -bool true

# Disable autofill
defaults write com.apple.Safari AutoFillPasswords -bool false
defaults write com.apple.Safari AutoFillCreditCardData -bool false

# Enable "Do Not Track"
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true
```

## System Monitoring

### OpenBSM Audit

```bash
# Enable audit system
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.auditd.plist

# Configure audit policy
sudo vi /etc/security/audit_control

# Monitor authentication attempts
sudo praudit -l /var/audit/* | grep "authentication"

# Monitor file access
sudo praudit -l /var/audit/* | grep "file"

# Real-time monitoring
sudo tail -f /var/audit/current | praudit -l
```

### Network Monitoring

```bash
# Monitor active connections
netstat -an | grep ESTABLISHED

# Monitor listening ports
sudo lsof -iTCP -sTCP:LISTEN -n -P

# Monitor network traffic
sudo tcpdump -i en0 -n

# Monitor DNS queries
sudo tcpdump -i en0 port 53

# Use nettop for real-time monitoring
nettop -m tcp

# Little Snitch alternative - manual monitoring
sudo fs_usage -w -f network | grep -v "mdnsresponder"
```

### Process Monitoring

```bash
# Monitor new process execution
sudo fs_usage -w -f exec

# Monitor file system changes
sudo fs_usage -w -f filesys

# DTrace scripts - monitor exec
sudo dtrace -n 'proc:::exec-success { printf("%s %s\n", execname, curpsinfo->pr_psargs); }'

# Monitor network connections
sudo dtrace -n 'syscall::connect:entry { printf("%s[%d] connecting\n", execname, pid); }'
```

## SSH Hardening

Edit `/etc/ssh/sshd_config`:

```bash
# Strong SSH configuration
Protocol 2
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes
UsePAM yes
X11Forwarding no
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
MaxSessions 2
AllowUsers yourusername

# Ciphers and algorithms
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
HostKeyAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256
```

Generate strong SSH keys:

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -a 100 -C "your_email@example.com"

# Or RSA 4096-bit
ssh-keygen -t rsa -b 4096 -o -a 100 -C "your_email@example.com"

# Set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
```

## Metadata Removal

```bash
# Remove metadata from images
exiftool -all= image.jpg

# Or use ImageOptim
brew install --cask imageoptim
open -a ImageOptim image.jpg

# Remove extended attributes
xattr -cr /path/to/file

# Securely delete files (on APFS, standard rm is usually sufficient)
rm -P sensitive_file.txt

# For more thorough deletion
srm -vz sensitive_file.txt  # (requires installation)
```

## Password Management

```bash
# Generate strong passwords
openssl rand -base64 32

# Or use diceware method
brew install diceware
diceware -n 6

# macOS Keychain management
security find-generic-password -ga "account_name"
security add-generic-password -a "account_name" -s "service_name" -w

# List keychain items
security dump-keychain -d login.keychain-db

# Lock keychain
security lock-keychain login.keychain-db
```

## Backup Security

```bash
# Enable Time Machine encryption
tmutil setdestination -a /Volumes/BackupDrive
diskutil apfs enableFileVault /Volumes/BackupDrive -user disk

# Verify backups
tmutil listbackups
tmutil verify

# Create encrypted disk image for backups
hdiutil create -size 50g -encryption AES-256 -type SPARSEBUNDLE \
  -fs "APFS" -volname "SecureBackup" ~/SecureBackup.sparsebundle

# Mount and backup
hdiutil attach ~/SecureBackup.sparsebundle
rsync -avh --delete ~/Documents/ /Volumes/SecureBackup/
hdiutil detach /Volumes/SecureBackup
```

## Lockdown Mode

For high-risk users facing targeted attacks:

```bash
# Check Lockdown Mode status
defaults read com.apple.Security LockdownModeEnabled

# Enable via System Settings > Privacy & Security > Lockdown Mode
# Or programmatically (requires restart):
sudo defaults write /Library/Preferences/com.apple.Security LockdownModeEnabled -bool true
```

## Physical Security

```bash
# Set automatic logout after inactivity (seconds)
defaults write com.apple.screensaver idleTime -int 300

# Show message on lock screen
sudo defaults write /Library/Preferences/com.apple.loginwindow \
  LoginwindowText "If found, please contact: your@email.com"

# Disable automatic login
sudo defaults delete /Library/Preferences/com.apple.loginwindow autoLoginUser

# Enable secure keyboard entry in Terminal
defaults write com.apple.Terminal SecureKeyboardEntry -bool true

# Disable TouchID for sudo (for maximum security)
sudo sed -i.bak 's/^auth.*pam_tid.so/#&/' /etc/pam.d/sudo
```

## Quick Security Audit Script

```bash
#!/bin/bash
# macos-security-audit.sh

echo "=== macOS Security Audit ==="

# FileVault status
echo -n "FileVault: "
fdesetup status

# Firewall status
echo -n "Firewall: "
/usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# Gatekeeper status
echo -n "Gatekeeper: "
spctl --status

# SIP status
echo -n "System Integrity Protection: "
csrutil status

# Firmware password
echo -n "Firmware Password: "
sudo firmwarepasswd -check

# Check for pending updates
echo -n "Pending Updates: "
softwareupdate -l 2>&1 | grep -q "No new software" && echo "None" || echo "Available"

# List sudo users
echo "Admin users:"
dscl . -read /Groups/admin GroupMembership

# Check screensaver settings
echo -n "Screensaver password delay: "
defaults read com.apple.screensaver askForPasswordDelay

# Check SSH status
echo -n "SSH Status: "
sudo systemsetup -getremotelogin

echo "=== Audit Complete ==="
```

## Troubleshooting

### Firewall Issues

```bash
# If connections are blocked unexpectedly
sudo pfctl -d  # Temporarily disable pf
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off

# Check firewall logs
log show --predicate 'process == "socketfilterfw"' --last 1h

# Reset firewall rules
sudo pfctl -F all -f /etc/pf.conf
```

### DNS Problems

```bash
# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Check DNS resolution
scutil --dns
nslookup example.com

# Test DNSCrypt
sudo dnscrypt-proxy -config /usr/local/etc/dnscrypt-proxy.toml -check
```

### FileVault Recovery

```bash
# Boot into Recovery Mode (Intel: Cmd+R, Apple silicon: hold power button)
# Unlock disk:
diskutil apfs unlockVolume disk1s1

# If locked out, use recovery key from safe location
```

### Performance Issues

```bash
# If security features cause slowdowns
# Disable unnecessary features temporarily:
sudo mdutil -i off /  # Disable Spotlight indexing
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist

# Re-enable:
sudo mdutil -i on /
```

## Environment Variables

```bash
# ~/.zshrc or ~/.bash_profile

# GPG configuration
export GPG_TTY=$(tty)

# DNSCrypt proxy
export DNSCRYPT_PROXY_CONFIG="/usr/local/etc/dnscrypt-proxy.toml"

# Homebrew (Apple silicon)
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## Additional Resources

- Official Apple Security Guide: https://support.apple.com/guide/security/
- NIST macOS Guidelines: https://github.com/usnistgov/macos_security
- CIS Benchmarks: https://www.cisecurity.org/benchmark/apple_os
- Objective-See Tools: https://objective-see.com/products.html

## Related Tools

```bash
# Install security tools via Homebrew
brew install gnupg yubikey-personalization pinentry-mac
brew install --cask lulu little-snitch knockknock oversight

# Objective-See security tools
brew install --cask blockblock lulu knockknock ransomwhere oversight
```

This skill provides comprehensive macOS security hardening suitable for both individual users and enterprise deployments. Always test configurations in a safe environment before applying to production systems.
