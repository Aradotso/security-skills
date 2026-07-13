---
name: macos-security-and-privacy-guide
description: Comprehensive guide and commands for securing macOS systems, covering encryption, firewalls, DNS, privacy settings, and security hardening
triggers:
  - how do I secure my Mac
  - configure macOS security settings
  - harden macOS privacy
  - set up FileVault and firewall on Mac
  - macOS security best practices
  - disable telemetry on macOS
  - configure DNS encryption on Mac
  - lock down macOS system
---

# macOS Security and Privacy Guide

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive guidance for securing and hardening macOS systems based on the community-maintained drduh/macOS-Security-and-Privacy-Guide. It covers threat modeling, encryption, firewalls, DNS configuration, privacy settings, and security monitoring.

## Overview

The macOS Security and Privacy Guide is a collection of techniques for improving security and privacy on Apple silicon Macs. It targets power users adopting enterprise-standard security and covers:

- Threat modeling and asset protection
- Full disk encryption with FileVault
- Firewall configuration (application and kernel-level)
- DNS encryption (DNSCrypt, DoH, DoT)
- Browser hardening
- System monitoring and audit
- Privacy settings and telemetry disabling
- Physical security measures

**Requirements:**
- Apple silicon Mac (M1 or newer recommended)
- Currently supported macOS version
- Administrative access

## Installation and Setup

This is a guide repository, not installable software. Access it at:
- Online: https://drduh.github.io/macOS-Security-and-Privacy-Guide/
- Repository: https://github.com/drduh/macOS-Security-and-Privacy-Guide

## Core Security Commands

### System Updates

```bash
# Check for updates
softwareupdate --list

# Install all available updates
sudo softwareupdate --install --all

# Enable automatic updates
sudo softwareupdate --schedule on

# Install security updates only
sudo softwareupdate --install --recommended
```

### FileVault Full Disk Encryption

```bash
# Enable FileVault (GUI method preferred for recovery key)
sudo fdesetup enable

# Check FileVault status
sudo fdesetup status

# List FileVault-enabled users
sudo fdesetup list

# Add user to FileVault
sudo fdesetup add -usertoadd username
```

### Firmware Password (Legacy Intel Macs)

```bash
# Set firmware password (reboot to recovery mode first)
# On Apple silicon, use Startup Security Utility instead

# Check if firmware password is set (Intel)
sudo firmwarepasswd -check
```

### Firewall Configuration

```bash
# Enable application firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable stealth mode (don't respond to pings)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Enable logging
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Block all incoming connections
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setblockall on

# Allow signed applications
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setallowsigned on

# Check firewall status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Packet Filter (pf) Configuration

```bash
# Create pf rules file
sudo nano /etc/pf.conf

# Example pf.conf rules:
# Block all incoming traffic except SSH and web
# scrub-anchor "com.apple/*"
# nat-anchor "com.apple/*"
# rdr-anchor "com.apple/*"
# dummynet-anchor "com.apple/*"
# anchor "com.apple/*"
# load anchor "com.apple" from "/etc/pf.anchors/com.apple"
# 
# block all
# pass out proto {tcp, udp, icmp} keep state
# pass in proto tcp to any port {22, 80, 443} keep state

# Enable pf
sudo pfctl -e

# Load rules
sudo pfctl -f /etc/pf.conf

# Check pf status
sudo pfctl -s info

# View active rules
sudo pfctl -s rules

# Disable pf
sudo pfctl -d
```

## Privacy and Telemetry

### Disable Spotlight Suggestions

```bash
# Disable Spotlight Suggestions
defaults write com.apple.Safari UniversalSearchEnabled -bool false
defaults write com.apple.Safari SuppressSearchSuggestions -bool true

# Disable Spotlight internet search
defaults write com.apple.spotlight orderedItems -array \
  '{"enabled" = 1;"name" = "APPLICATIONS";}' \
  '{"enabled" = 1;"name" = "SYSTEM_PREFS";}' \
  '{"enabled" = 1;"name" = "DIRECTORIES";}' \
  '{"enabled" = 1;"name" = "PDF";}' \
  '{"enabled" = 1;"name" = "DOCUMENTS";}'

# Restart Spotlight
killall mds
```

### Disable Unnecessary Services

```bash
# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false
launchctl disable "user/$UID/com.apple.assistantd"

# Disable Handoff
defaults write com.apple.coreservices.useractivityd ActivityAdvertisingAllowed -bool false
defaults write com.apple.coreservices.useractivityd ActivityReceivingAllowed -bool false

# Disable AirDrop
defaults write com.apple.NetworkBrowser DisableAirDrop -bool true

# Disable remote Apple events
sudo systemsetup -setremoteappleevents off

# Disable remote login (SSH if not needed)
sudo systemsetup -setremotelogin off

# Disable Internet Sharing
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.nat NAT -dict Enabled -bool false
```

### Privacy Settings

```bash
# Clear location services cache
sudo rm -rf /var/db/locationd/*

# Disable Captive Portal assistant
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.captive.control Active -bool false

# Disable crash reporter
defaults write com.apple.CrashReporter DialogType none

# Disable Bonjour multicast
sudo defaults write /Library/Preferences/com.apple.mDNSResponder.plist NoMulticastAdvertisements -bool true

# Set hostname (prevents leaking computer name)
sudo scutil --set ComputerName "MacBook"
sudo scutil --set LocalHostName "MacBook"
sudo scutil --set HostName "MacBook"
```

## DNS Configuration

### DNSCrypt Installation (via Homebrew)

```bash
# Install Homebrew first if needed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dnscrypt-proxy
brew install dnscrypt-proxy

# Edit configuration
nano /usr/local/etc/dnscrypt-proxy.toml

# Example configuration:
# server_names = ['cloudflare', 'google']
# listen_addresses = ['127.0.0.1:53']
# require_dnssec = true
# require_nolog = true
# require_nofilter = true

# Start dnscrypt-proxy
sudo brew services start dnscrypt-proxy

# Configure system DNS to use localhost
networksetup -setdnsservers Wi-Fi 127.0.0.1
networksetup -setdnsservers Ethernet 127.0.0.1
```

### DNS over HTTPS/TLS with Configuration Profile

```bash
# Create DNS profile for Cloudflare DoH
# Save as cloudflare-doh.mobileconfig

cat > cloudflare-doh.mobileconfig <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>PayloadContent</key>
    <array>
        <dict>
            <key>DNSSettings</key>
            <dict>
                <key>DNSProtocol</key>
                <string>HTTPS</string>
                <key>ServerURL</key>
                <string>https://cloudflare-dns.com/dns-query</string>
            </dict>
            <key>PayloadType</key>
            <string>com.apple.dnsSettings.managed</string>
            <key>PayloadIdentifier</key>
            <string>com.cloudflare.1dot1dot1dot1.dns</string>
            <key>PayloadUUID</key>
            <string>RANDOM-UUID-HERE</string>
            <key>PayloadDisplayName</key>
            <string>Cloudflare DNS over HTTPS</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
        </dict>
    </array>
    <key>PayloadDisplayName</key>
    <string>Cloudflare DNS</string>
    <key>PayloadIdentifier</key>
    <string>com.cloudflare.dns</string>
    <key>PayloadUUID</key>
    <string>RANDOM-UUID-HERE-2</string>
    <key>PayloadType</key>
    <string>Configuration</string>
    <key>PayloadVersion</key>
    <integer>1</integer>
</dict>
</plist>
EOF

# Install profile (will prompt for admin password)
open cloudflare-doh.mobileconfig
```

### Hosts File Configuration

```bash
# Edit hosts file for ad blocking
sudo nano /etc/hosts

# Add entries like:
# 0.0.0.0 ads.example.com
# 0.0.0.0 tracker.example.com

# Use a maintained blocklist
curl -o /tmp/hosts.txt https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
sudo cp /tmp/hosts.txt /etc/hosts

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

## System Monitoring

### OpenBSM Audit

```bash
# Check audit status
sudo audit -n

# View audit configuration
sudo cat /etc/security/audit_control

# Enable specific audit flags
sudo nano /etc/security/audit_control
# Add flags: lo,aa,ad,fd,fm,-all

# Restart audit system
sudo audit -s

# Review audit logs
sudo praudit -xn /var/audit/*

# Monitor real-time audit events
sudo praudit /dev/auditpipe
```

### DTrace System Monitoring

```bash
# Monitor file opens (requires SIP disabled for dtrace)
sudo dtrace -n 'syscall::open*:entry { printf("%s %s", execname, copyinstr(arg0)); }'

# Monitor network connections
sudo dtrace -n 'syscall::connect:entry { printf("%s", execname); }'

# Monitor process execution
sudo dtrace -n 'proc:::exec-success { trace(curpsinfo->pr_psargs); }'

# Monitor DNS queries
sudo dtrace -n 'syscall::getaddrinfo:entry { printf("%s", copyinstr(arg0)); }'
```

### Network Monitoring

```bash
# List listening ports and processes
sudo lsof -i -P -n | grep LISTEN

# Monitor network connections
nettop -P -L 0

# Use Little Snitch alternative - lulu (free)
brew install --cask lulu

# Monitor DNS queries
sudo tcpdump -i any -n port 53

# Monitor all network traffic
sudo tcpdump -i any -n

# View network statistics
netstat -an

# Check active network connections
lsof -i
```

### Process and Execution Monitoring

```bash
# Monitor running processes
ps aux

# Real-time process monitoring
top -o cpu

# Monitor process launches
sudo opensnoop

# Monitor file system access
sudo fs_usage

# List loaded kernel extensions
kextstat

# Check for suspicious login items
osascript -e 'tell application "System Events" to get the name of every login item'

# List LaunchAgents and LaunchDaemons
ls -la ~/Library/LaunchAgents/
ls -la /Library/LaunchAgents/
sudo ls -la /Library/LaunchDaemons/
```

## User Account Security

### Admin and Standard User Setup

```bash
# Create standard user account
sudo dscl . -create /Users/standarduser
sudo dscl . -create /Users/standarduser UserShell /bin/bash
sudo dscl . -create /Users/standarduser RealName "Standard User"
sudo dscl . -create /Users/standarduser UniqueID 503
sudo dscl . -create /Users/standarduser PrimaryGroupID 20
sudo dscl . -create /Users/standarduser NFSHomeDirectory /Users/standarduser
sudo dscl . -passwd /Users/standarduser

# Create home directory
sudo createhomedir -c -u standarduser

# Remove admin user from admin group (after creating another admin)
sudo dseditgroup -o edit -d username -t user admin

# Require password immediately after sleep/screensaver
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

# Set screen lock timeout (in seconds)
defaults -currentHost write com.apple.screensaver idleTime -int 300
```

### Password Policies

```bash
# Set minimum password length
sudo pwpolicy -n /Local/Default -setglobalpolicy "minChars=14"

# Require mixed case
sudo pwpolicy -n /Local/Default -setglobalpolicy "requiresMixedCase=1"

# Password history
sudo pwpolicy -n /Local/Default -setglobalpolicy "usingHistory=5"

# Maximum failed login attempts
sudo pwpolicy -n /Local/Default -setglobalpolicy "maxFailedLoginAttempts=5"

# Check current password policy
sudo pwpolicy -n /Local/Default -getglobalpolicy
```

## Browser Security

### Safari Hardening

```bash
# Disable auto-fill
defaults write com.apple.Safari AutoFillPasswords -bool false
defaults write com.apple.Safari AutoFillCreditCardData -bool false
defaults write com.apple.Safari AutoFillFromAddressBook -bool false
defaults write com.apple.Safari AutoFillMiscellaneousForms -bool false

# Enable Do Not Track
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true

# Disable preloading top hit
defaults write com.apple.Safari PreloadTopHit -bool false

# Show full URL
defaults write com.apple.Safari ShowFullURLInSmartSearchField -bool true

# Warn about fraudulent websites
defaults write com.apple.Safari WarnAboutFraudulentWebsites -bool true

# Disable plugins
defaults write com.apple.Safari com.apple.Safari.ContentPageGroupIdentifier.WebKit2PluginsEnabled -bool false

# Enable Develop menu
defaults write com.apple.Safari IncludeDevelopMenu -bool true
defaults write com.apple.Safari WebKitDeveloperExtrasEnabledPreferenceKey -bool true

# Clear history and cookies on quit
defaults write com.apple.Safari ClearHistoryOnQuit -bool true
```

### Firefox Hardening

```bash
# Install Firefox
brew install --cask firefox

# Create user.js for privacy settings
mkdir -p ~/Library/Application\ Support/Firefox/Profiles/*.default-release/
cat > ~/Library/Application\ Support/Firefox/Profiles/*.default-release/user.js <<EOF
// Privacy settings
user_pref("privacy.trackingprotection.enabled", true);
user_pref("privacy.trackingprotection.socialtracking.enabled", true);
user_pref("privacy.donottrackheader.enabled", true);
user_pref("privacy.firstparty.isolate", true);
user_pref("privacy.resistFingerprinting", true);

// Disable telemetry
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("toolkit.telemetry.archive.enabled", false);

// Security
user_pref("security.ssl.require_safe_negotiation", true);
user_pref("security.tls.version.min", 3);
user_pref("network.cookie.cookieBehavior", 1);
user_pref("network.cookie.lifetimePolicy", 2);

// Disable WebRTC leak
user_pref("media.peerconnection.enabled", false);
EOF
```

## SSH Configuration

### Secure SSH Setup

```bash
# Generate ED25519 key pair
ssh-keygen -t ed25519 -a 100 -C "user@hostname"

# Or use RSA 4096-bit
ssh-keygen -t rsa -b 4096 -o -a 100 -C "user@hostname"

# Set proper permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# Configure SSH client
cat > ~/.ssh/config <<EOF
Host *
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_ed25519
    Protocol 2
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
    KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
    HostKeyAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256
    ServerAliveInterval 60
    ServerAliveCountMax 3
EOF

chmod 600 ~/.ssh/config

# If enabling SSH server (disabled by default)
sudo systemsetup -setremotelogin on

# Configure sshd
sudo nano /etc/ssh/sshd_config
# Set:
# PermitRootLogin no
# PasswordAuthentication no
# ChallengeResponseAuthentication no
# UsePAM yes
# AllowUsers yourusername

sudo launchctl stop com.openssh.sshd
sudo launchctl start com.openssh.sshd
```

## VPN and Tor

### OpenVPN Configuration

```bash
# Install OpenVPN via Homebrew
brew install openvpn

# Example client configuration
cat > ~/vpn-config.ovpn <<EOF
client
dev tun
proto udp
remote vpn.example.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
verb 3
ca ca.crt
cert client.crt
key client.key
tls-auth ta.key 1
EOF

# Connect to VPN
sudo openvpn --config ~/vpn-config.ovpn
```

### Tor Configuration

```bash
# Install Tor
brew install tor

# Configure torrc
nano /usr/local/etc/tor/torrc
# Add:
# SOCKSPort 9050
# ControlPort 9051
# CookieAuthentication 1

# Start Tor
brew services start tor

# Configure system proxy (manual in Network preferences)
# SOCKS proxy: 127.0.0.1:9050

# Or use with proxychains
brew install proxychains-ng
nano /usr/local/etc/proxychains.conf
# Add: socks5 127.0.0.1 9050

# Use proxychains
proxychains4 curl https://check.torproject.org
```

## Metadata and Privacy

### Remove Metadata from Files

```bash
# Install exiftool
brew install exiftool

# Remove all metadata from image
exiftool -all= image.jpg

# Remove metadata from PDF
exiftool -all= document.pdf

# Check metadata
exiftool image.jpg

# Batch remove metadata
exiftool -all= -r /path/to/directory/

# macOS built-in metadata
mdls file.pdf
xattr file.pdf
xattr -c file.pdf  # Clear extended attributes
```

### Secure File Deletion

```bash
# Securely delete file (write random data)
rm -P sensitive_file.txt

# Use srm (secure remove) - install first
brew install srm
srm -v sensitive_file.txt

# Wipe free space (limited effectiveness on SSD)
diskutil secureErase freespace 0 /Volumes/Macintosh\ HD

# Note: SSDs use wear-leveling, making secure deletion difficult
# FileVault encryption is the best protection for SSDs
```

## Backup Security

### Time Machine Encrypted Backup

```bash
# Format external drive as encrypted
diskutil list
diskutil eraseDisk JHFS+ "Backup" GPT disk2

# Enable FileVault on backup disk via System Preferences
# Or use diskutil:
diskutil apfs encryptVolume /Volumes/Backup -user disk

# Set up Time Machine
sudo tmutil setdestination /Volumes/Backup

# Start backup
sudo tmutil startbackup

# Exclude directories from backup
sudo tmutil addexclusion ~/Downloads
sudo tmutil addexclusion ~/.Trash

# List exclusions
sudo tmutil isexcluded ~/Downloads
```

### Encrypted Archives

```bash
# Create encrypted zip
zip -er archive.zip /path/to/files/

# Create encrypted disk image
hdiutil create -encryption AES-256 -size 1g -fs APFS -volname "Secure" ~/secure.dmg

# Mount encrypted image
hdiutil attach ~/secure.dmg

# Create encrypted tar archive with gpg
tar czf - /path/to/files/ | gpg -c > backup.tar.gz.gpg

# Decrypt and extract
gpg -d backup.tar.gz.gpg | tar xzf -
```

## Lockdown Mode

```bash
# Enable Lockdown Mode via System Settings
# Security & Privacy → Lockdown Mode

# Lockdown Mode disables:
# - Most message attachment types
# - Link previews
# - FaceTime features
# - Web fonts and complex web technologies
# - Shared albums in Photos
# - Device enrollment and configuration profiles

# Check if Lockdown Mode is enabled
defaults read /Library/Preferences/com.apple.security.lockdownmode.plist LockdownModeEnabled
```

## Certificate Authorities

### Managing Trusted CAs

```bash
# List system certificates
security dump-keychain /System/Library/Keychains/SystemRootCertificates.keychain

# Export certificates
security find-certificate -a -p /System/Library/Keychains/SystemRootCertificates.keychain > system-certs.pem

# Trust custom CA
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain custom-ca.crt

# Remove trust from CA
sudo security delete-certificate -c "Certificate Name" /Library/Keychains/System.keychain

# List trusted certificates
security find-certificate -a /Library/Keychains/System.keychain
```

## Common Security Patterns

### Security Audit Script

```bash
#!/bin/bash
# macos-security-audit.sh

echo "=== macOS Security Audit ==="

echo -e "\n[FileVault Status]"
sudo fdesetup status

echo -e "\n[Firewall Status]"
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

echo -e "\n[Gatekeeper Status]"
spctl --status

echo -e "\n[SIP Status]"
csrutil status

echo -e "\n[Firmware Password - Intel Only]"
sudo firmwarepasswd -check 2>/dev/null || echo "Not available on Apple Silicon"

echo -e "\n[Remote Login]"
sudo systemsetup -getremotelogin

echo -e "\n[Remote Management]"
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -status

echo -e "\n[Listening Ports]"
sudo lsof -i -P -n | grep LISTEN

echo -e "\n[Failed Login Attempts]"
grep "Failed" /var/log/system.log 2>/dev/null | tail -n 5 || echo "No recent failed logins"

echo -e "\n[System Updates Available]"
softwareupdate --list 2>&1

echo -e "\n[LaunchDaemons]"
ls -la /Library/LaunchDaemons/ 2>/dev/null | grep -v "com.apple"
```

### Privacy Hardening Script

```bash
#!/bin/bash
# privacy-hardening.sh

echo "Applying privacy settings..."

# Disable analytics
defaults write com.apple.AdLib allowApplePersonalizedAdvertising -bool false
defaults write com.apple.AdLib allowIdentifierForAdvertising -bool false

# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false

# Disable Handoff
defaults write com.apple.coreservices.useractivityd ActivityAdvertisingAllowed -bool false

# Safari privacy
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true
defaults write com.apple.Safari UniversalSearchEnabled -bool false

# Disable crash reporting
defaults write com.apple.CrashReporter DialogType none

# Disable captive portal
sudo defaults write /Library/Preferences/SystemConfiguration/com.apple.captive.control Active -bool false

# Set hostname
sudo scutil --set ComputerName "MacBook"
sudo scutil --set LocalHostName "MacBook"

echo "Privacy settings applied. Restart may be required for some changes."
```

## Troubleshooting

### Firewall Issues

```bash
# Check if firewall is blocking connections
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --listapps

# Allow specific application
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /Applications/MyApp.app

# Remove application from firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --remove /Applications/MyApp.app

# Reset firewall to defaults
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
```

### DNS Not Working

```bash
# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Check DNS servers
scutil --dns

# Reset DNS to automatic (DHCP)
networksetup -setdnsservers Wi-Fi Empty

# Test DNS resolution
dig example.com
nslookup example.com

# Check if dnscrypt-proxy is running
brew services list | grep dnscrypt
```

### VPN Connection Issues

```bash
# Kill stuck VPN processes
sudo killall openvpn

# Check VPN logs
tail -f /var/log/system.log | grep vpn

# Reset network settings
sudo ifconfig en0 down
sudo ifconfig en0 up

# Restart network service
sudo networksetup -setnetworkserviceenabled Wi-Fi off
sudo networksetup -setnetworkserviceenabled Wi-Fi on
```

### SIP and Security Issues

```bash
# Check SIP status
csrutil status

# To disable SIP (not recommended):
# 1. Reboot into Recovery Mode (hold Cmd+R on Intel, hold Power on Apple Silicon)
# 2. Open Terminal from Utilities menu
# 3. Run: csrutil disable
# 4. Reboot

# Re-enable SIP
# csrutil enable

# Check code signing
codesign -v /Applications/App.app
spctl -a -v /Applications/App.app
```

### Audit Logs Not Recording

```bash
# Check audit daemon status
sudo launchctl list | grep audit

# Restart audit daemon
sudo launchctl stop com.apple.auditd
sudo launchctl start com.apple.auditd

# Check audit configuration
sudo cat /etc/security/audit_control

# Verify audit log directory
ls -la /var/audit/
```

## Environment Variables

Reference environment variables for sensitive data:

```bash
# VPN credentials
export VPN_USER="${VPN_USERNAME}"
export VPN_PASS="${VPN_PASSWORD}"

# DNS API keys
export CLOUDFLARE_API_KEY="${CF_API_KEY}"

# SSH key passphrases (use ssh-agent instead)
# Never hardcode passphrases

# GPG key passphrase (use gpg-agent instead)
# Never hardcode passphrases
```

## Additional Resources

- Official Guide: https://drduh.github.io/macOS-Security-and-Privacy-Guide/
- NIST macOS Security: https://github.com/usnistgov/macos_security
- Apple Platform Security: https://support.apple.com/guide/security/
- CIS macOS Benchmarks: https://www.cisecurity.org/benchmark/apple_os
