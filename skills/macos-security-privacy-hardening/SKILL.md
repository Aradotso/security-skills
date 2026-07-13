---
name: macos-security-privacy-hardening
description: Community guide to securing and improving privacy on macOS with practical configurations and commands
triggers:
  - how do I secure my macOS system
  - harden macOS security and privacy
  - configure macOS firewall and encryption
  - set up FileVault and security settings on Mac
  - improve macOS privacy settings
  - configure DNS encryption on macOS
  - secure my MacBook configuration
  - macOS security best practices
---

# macOS Security and Privacy Hardening

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This guide provides comprehensive techniques for improving security and privacy on Apple silicon Mac computers running currently supported versions of macOS. It covers threat modeling, system hardening, encryption, network security, and privacy configurations.

## What This Guide Covers

The macOS Security and Privacy Guide is a collection of:
- Threat modeling frameworks for personal security
- System-level security configurations
- Disk encryption and firmware security
- Firewall and network privacy settings
- DNS encryption and privacy
- Browser hardening techniques
- Application security and sandboxing
- System monitoring and auditing
- Physical security measures

**Important**: This guide targets Apple silicon Macs. Intel-based Macs have hardware-level vulnerabilities that Apple cannot patch.

## Threat Modeling Framework

Before implementing security measures, create a threat model:

### Asset Identification

List your assets by sensitivity:
- **Public**: Non-sensitive information
- **Sensitive**: Personal data, passwords, browsing history
- **Secret**: Financial data, private communications, encryption keys

### Adversary Analysis Table

```markdown
| Adversary | Motivation | Capabilities | Mitigation |
|-----------|-----------|--------------|------------|
| Roommate | Curiosity | Physical access, shoulder surfing | Use biometrics, privacy screen, keep locked |
| Thief | Financial gain | Device theft, shoulder surfing | Full disk encryption, Find My, biometrics |
| Criminal | Financial | Social engineering, malware, password reuse | Sandboxing, updates, unique passwords |
| Corporation | User data | Telemetry, behavioral tracking | Block connections, reset identifiers |
| Nation State | Surveillance | Network monitoring, advanced cryptanalysis | E2EE, strong passwords, secure element hardware |
```

## System Installation and Initial Setup

### Clean Installation

```bash
# Check macOS version
sw_vers

# Download latest macOS installer (from Recovery Mode: Cmd+R at boot)
# Or create bootable installer
sudo /Applications/Install\ macOS\ Sonoma.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume

# Check for system updates
softwareupdate --list

# Install all available updates
sudo softwareupdate --install --all

# Enable automatic updates
sudo softwareupdate --schedule on
```

### Admin and User Account Separation

Create separate admin and standard user accounts:

```bash
# Create standard user account (do this from admin account)
sudo dscl . -create /Users/standarduser
sudo dscl . -create /Users/standarduser UserShell /bin/bash
sudo dscl . -create /Users/standarduser RealName "Standard User"
sudo dscl . -create /Users/standarduser UniqueID 503
sudo dscl . -create /Users/standarduser PrimaryGroupID 20
sudo dscl . -create /Users/standarduser NFSHomeDirectory /Users/standarduser
sudo dscl . -passwd /Users/standarduser

# Create home directory
sudo createhomedir -c -u standarduser

# Verify account is not admin
dscl . -read /Groups/admin GroupMembership
```

**Best Practice**: Use the standard account for daily work. Only use admin account for system changes.

## Firmware Security

### Set Firmware Password (Intel Macs)

For Apple silicon Macs, firmware security is automatic. For Intel Macs:

```bash
# Check if firmware password is set
sudo firmwarepasswd -check

# Set firmware password (Intel Macs only)
sudo firmwarepasswd -setpasswd

# Verify it's enabled
sudo firmwarepasswd -verify
```

### Check Security Settings

```bash
# Check Secure Boot status (Apple silicon)
nvram 94b73556-2197-4702-82a8-3e1337dafbfb:AppleSecureBootPolicy

# Check System Integrity Protection
csrutil status

# Check Gatekeeper status
spctl --status
```

## FileVault Full Disk Encryption

### Enable FileVault

```bash
# Enable FileVault (requires admin password)
sudo fdesetup enable

# Check FileVault status
sudo fdesetup status

# List FileVault-enabled users
sudo fdesetup list

# Add user to FileVault
sudo fdesetup add -usertoadd username
```

### Recovery Key Management

```bash
# Get recovery key (do this immediately after enabling FileVault)
sudo fdesetup changerecovery -personal

# Verify encryption progress
diskutil apfs list

# Check encryption status
diskutil cs list  # For older Macs with CoreStorage
fdesetup status   # For APFS volumes
```

**Important**: Store recovery key offline in a secure location (e.g., safe, password manager).

## Firewall Configuration

### Application Layer Firewall

```bash
# Enable built-in firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable logging
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Enable stealth mode (don't respond to ICMP ping)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Block all incoming connections
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setblockall on

# Allow specific application
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /Applications/Firefox.app

# Remove application from firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --remove /Applications/Firefox.app

# Check firewall status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Packet Filter (PF) Firewall

Create a PF configuration file:

```bash
# Create PF rules file
sudo nano /etc/pf.conf
```

Example `/etc/pf.conf`:

```pf
# Interfaces
ext_if = "en0"
int_if = "lo0"

# Default policy: block all
set block-policy drop
set skip on lo0

# Scrub incoming packets
scrub in all

# Block all by default
block all

# Allow outgoing traffic
pass out quick on $ext_if proto { tcp, udp, icmp } keep state

# Allow incoming SSH (if needed)
# pass in quick on $ext_if proto tcp to port 22 keep state

# Allow incoming connections for specific apps
# pass in quick on $ext_if proto tcp to port 443 keep state

# Block spoofed addresses
antispoof quick for $ext_if
```

Enable and manage PF:

```bash
# Test PF configuration
sudo pfctl -nf /etc/pf.conf

# Enable PF
sudo pfctl -e

# Load rules
sudo pfctl -f /etc/pf.conf

# Check PF status
sudo pfctl -s info

# View rules
sudo pfctl -s rules

# Disable PF
sudo pfctl -d

# View blocked packets
sudo pfctl -s states
```

## DNS Privacy and Encryption

### DNS over HTTPS (DoH) with Configuration Profile

Create a DNS profile for encrypted DNS:

```xml
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
                <string>https://dns.quad9.net/dns-query</string>
            </dict>
            <key>PayloadDisplayName</key>
            <string>Quad9 DNS over HTTPS</string>
            <key>PayloadIdentifier</key>
            <string>com.apple.dnsSettings.managed</string>
            <key>PayloadType</key>
            <string>com.apple.dnsSettings.managed</string>
            <key>PayloadUUID</key>
            <string>YOUR-UUID-HERE</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
        </dict>
    </array>
    <key>PayloadDisplayName</key>
    <string>Encrypted DNS</string>
    <key>PayloadIdentifier</key>
    <string>com.example.dns</string>
    <key>PayloadType</key>
    <string>Configuration</string>
    <key>PayloadUUID</key>
    <string>YOUR-UUID-HERE</string>
    <key>PayloadVersion</key>
    <integer>1</integer>
</dict>
</plist>
```

Install the profile:

```bash
# Save as encrypted-dns.mobileconfig
# Double-click to install or use:
sudo profiles install -path=/path/to/encrypted-dns.mobileconfig

# List installed profiles
sudo profiles list

# Remove profile
sudo profiles remove -identifier com.example.dns
```

### DNSCrypt-Proxy

Install and configure DNSCrypt:

```bash
# Install via Homebrew
brew install dnscrypt-proxy

# Configure DNSCrypt
sudo nano /usr/local/etc/dnscrypt-proxy.toml
```

Example `dnscrypt-proxy.toml`:

```toml
listen_addresses = ['127.0.0.1:5355']
max_clients = 250
ipv4_servers = true
ipv6_servers = false
dnscrypt_servers = true
doh_servers = true

server_names = ['cloudflare', 'quad9-dnscrypt-ip4-nofilter-pri']

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
```

Start DNSCrypt:

```bash
# Create log directory
sudo mkdir -p /var/log/dnscrypt-proxy

# Start service
sudo brew services start dnscrypt-proxy

# Set system DNS to use DNSCrypt
networksetup -setdnsservers Wi-Fi 127.0.0.1

# Verify DNS
scutil --dns

# Test DNS resolution
dig @127.0.0.1 -p 5355 example.com
```

### Hosts File Blocking

```bash
# Backup original hosts file
sudo cp /etc/hosts /etc/hosts.backup

# Download and install hosts file (blocks ads/trackers)
curl -o /tmp/hosts https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
sudo cp /tmp/hosts /etc/hosts

# Or manually edit
sudo nano /etc/hosts

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

Example custom hosts entries:

```text
# Block specific domains
0.0.0.0 ads.example.com
0.0.0.0 tracker.example.com
0.0.0.0 facebook.com
0.0.0.0 www.facebook.com
```

## Privacy Settings

### Disable Telemetry and Analytics

```bash
# Disable diagnostic data
sudo defaults write /Library/Application\ Support/CrashReporter/DiagnosticMessagesHistory.plist AutoSubmit -bool false
sudo defaults write /Library/Application\ Support/CrashReporter/DiagnosticMessagesHistory.plist ThirdPartyDataSubmit -bool false

# Disable Siri analytics
defaults write com.apple.assistant.support 'Siri Data Sharing Opt-In Status' -int 2

# Disable personalized ads
defaults write com.apple.AdLib allowApplePersonalizedAdvertising -bool false

# Disable handoff
defaults write ~/Library/Preferences/ByHost/com.apple.coreservices.useractivityd ActivityAdvertisingAllowed -bool no
defaults write ~/Library/Preferences/ByHost/com.apple.coreservices.useractivityd ActivityReceivingAllowed -bool no

# Disable Spotlight Suggestions
defaults write com.apple.safari UniversalSearchEnabled -bool false
defaults write com.apple.Safari SuppressSearchSuggestions -bool true
```

### Location Services

```bash
# Disable location services (requires reboot)
sudo defaults write /var/db/locationd/Library/Preferences/ByHost/com.apple.locationd LocationServicesEnabled -int 0

# Check location services status
defaults read /var/db/locationd/Library/Preferences/ByHost/com.apple.locationd

# Clear location cache
sudo rm -rf /var/db/locationd/*
```

### Bluetooth Privacy

```bash
# Disable Bluetooth
sudo defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 0

# Re-enable when needed
sudo defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 1
sudo killall -HUP bluetoothd
```

### Camera and Microphone

```bash
# Check which apps have camera access
sudo sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db "SELECT client FROM access WHERE service='kTCCServiceCamera'"

# Check which apps have microphone access
sudo sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db "SELECT client FROM access WHERE service='kTCCServiceMicrophone'"

# Reset all privacy permissions (requires reboot)
sudo tccutil reset All
```

## Browser Hardening

### Firefox Configuration

Create `user.js` in Firefox profile directory (`~/Library/Application Support/Firefox/Profiles/xxxxxxxx.default-release/`):

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
user_pref("datareporting.policy.dataSubmissionEnabled", false);

// DNS over HTTPS
user_pref("network.trr.mode", 2);
user_pref("network.trr.uri", "https://dns.quad9.net/dns-query");

// HTTPS-Only Mode
user_pref("dom.security.https_only_mode", true);

// Disable WebGL
user_pref("webgl.disabled", true);

// Disable location services
user_pref("geo.enabled", false);

// Disable WebRTC (can leak IP)
user_pref("media.peerconnection.enabled", false);
```

### Safari Hardening

```bash
# Enable developer menu
defaults write com.apple.Safari IncludeDevelopMenu -bool true

# Disable AutoFill
defaults write com.apple.Safari AutoFillFromAddressBook -bool false
defaults write com.apple.Safari AutoFillPasswords -bool false
defaults write com.apple.Safari AutoFillCreditCardData -bool false
defaults write com.apple.Safari AutoFillMiscellaneousForms -bool false

# Block cookies from third parties
defaults write com.apple.Safari BlockStoragePolicy -int 2

# Enable "Do Not Track"
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true

# Disable preloading top hit
defaults write com.apple.Safari PreloadTopHit -bool false

# Show full URL
defaults write com.apple.Safari ShowFullURLInSmartSearchField -bool true

# Disable opening "safe" files
defaults write com.apple.Safari AutoOpenSafeDownloads -bool false

# Clear history items
defaults write com.apple.Safari HistoryAgeInDaysLimit -int 1
```

## System Services Management

### Disable Unnecessary Services

```bash
# List all services
sudo launchctl list

# Disable services (examples - adjust based on needs)
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist

# Disable Bonjour
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist

# Disable AirDrop
sudo launchctl unload -w /System/Library/LaunchAgents/com.apple.sharingd.plist

# Re-enable if needed
sudo launchctl load -w /path/to/service.plist
```

### Login Items

```bash
# List login items
osascript -e 'tell application "System Events" to get the name of every login item'

# Remove login item
osascript -e 'tell application "System Events" to delete login item "ItemName"'
```

## System Integrity Protection (SIP)

```bash
# Check SIP status
csrutil status

# Disable SIP (only from Recovery Mode - not recommended)
# Boot into Recovery (Cmd+R), then:
csrutil disable

# Re-enable SIP (from Recovery Mode)
csrutil enable

# Clear SIP-protected files (requires SIP disabled)
sudo csrutil clear
```

**Warning**: Disabling SIP significantly reduces system security. Only disable temporarily if absolutely necessary.

## Password Management

### System Password Policy

```bash
# Require password immediately after sleep or screen saver begins
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

# Set screen saver timeout (seconds)
defaults -currentHost write com.apple.screensaver idleTime -int 300

# Require admin password for system-wide preferences
sudo security authorizationdb read system.preferences > /tmp/system.preferences.plist
sudo security authorizationdb write system.preferences < /tmp/system.preferences.plist
```

### Password Generation

```bash
# Generate strong password (32 chars)
LC_ALL=C tr -dc 'A-Za-z0-9!@#$%^&*' < /dev/urandom | head -c 32; echo

# Generate diceware passphrase
brew install diceware
diceware -n 6

# Generate password with OpenSSL
openssl rand -base64 32
```

## System Monitoring

### OpenBSM Auditing

```bash
# Check audit status
sudo audit -s

# View audit configuration
sudo cat /etc/security/audit_control

# Configure audit flags
sudo nano /etc/security/audit_control
```

Example `audit_control`:

```text
dir:/var/audit
flags:lo,aa,ad,fd,fm,-all
minfree:5
naflags:lo,aa
policy:cnt,argv
filesz:1M
expire-after:90d
```

Enable and manage auditing:

```bash
# Enable auditing
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.auditd.plist

# View audit logs
sudo praudit -xn /var/audit/current

# Search for specific events
sudo praudit /var/audit/* | grep "login"

# Clear audit logs
sudo audit -n
```

### DTrace Monitoring

Monitor system calls:

```bash
# Monitor file opens by process
sudo dtrace -n 'syscall::open*:entry { printf("%s %s", execname, copyinstr(arg0)); }'

# Monitor network connections
sudo dtrace -n 'syscall::connect:entry { printf("%s", execname); }'

# Track process execution
sudo dtrace -n 'proc:::exec-success { printf("%s", execname); }'

# Monitor file writes
sudo dtrace -n 'syscall::write:entry /arg0 == 1/ { printf("%s", execname); }'
```

### Network Monitoring

```bash
# Show active network connections
sudo lsof -i -P -n

# Monitor specific port
sudo lsof -i :443

# Show listening ports
sudo lsof -i -P | grep LISTEN

# Network statistics
netstat -an | grep ESTABLISHED

# Packet capture
sudo tcpdump -i en0 -n

# Monitor DNS queries
sudo tcpdump -i en0 port 53

# Monitor specific host
sudo tcpdump -i en0 host 192.168.1.1
```

### Little Snitch Alternative Monitoring

```bash
# Monitor outgoing connections
sudo lsof -i | grep -E 'TCP|UDP'

# Create connection log script
cat > ~/monitor-connections.sh << 'EOF'
#!/bin/bash
while true; do
    date >> ~/connection-log.txt
    lsof -i -P -n >> ~/connection-log.txt
    sleep 60
done
EOF

chmod +x ~/monitor-connections.sh
```

## SSH Hardening

### SSH Configuration

Edit `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended settings:

```text
# Disable password authentication (use keys only)
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# Disable root login
PermitRootLogin no

# Use strong ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org

# Disable X11 forwarding
X11Forwarding no

# Disable TCP forwarding (if not needed)
AllowTcpForwarding no

# Limit users
AllowUsers yourusername

# Log verbosely
LogLevel VERBOSE
```

Restart SSH:

```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

### SSH Key Generation

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -a 100 -C "your_email@example.com"

# Or RSA 4096-bit
ssh-keygen -t rsa -b 4096 -a 100 -C "your_email@example.com"

# Set proper permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# Add key to agent
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# Configure SSH to always use keychain
cat >> ~/.ssh/config << 'EOF'
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF
```

## Metadata Removal

### Remove File Metadata

```bash
# View metadata
mdls filename.jpg

# Remove all extended attributes
xattr -cr filename.jpg

# Remove EXIF data from images
brew install exiftool
exiftool -all= image.jpg

# Batch remove metadata
exiftool -all= -r /path/to/directory/

# Remove metadata from PDFs
brew install qpdf
qpdf --linearize input.pdf output.pdf
```

### Clear System Caches

```bash
# Clear DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Clear Font cache
sudo atsutil databases -remove

# Clear application cache
rm -rf ~/Library/Caches/*

# Clear system log
sudo rm -rf /private/var/log/asl/*.asl

# Clear quarantine history
sqlite3 ~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2 'delete from LSQuarantineEvent'

# Clear recent items
defaults delete com.apple.recentitems
defaults delete com.apple.finder FXRecentFolders
```

## Physical Security

### Automatic Logout

```bash
# Set auto-logout timeout (seconds)
sudo defaults write /Library/Preferences/.GlobalPreferences com.apple.autologout.AutoLogOutDelay -int 3600

# Require password after screensaver
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0
```

### Disable Guest Account

```bash
# Disable guest account
sudo defaults write /Library/Preferences/com.apple.loginwindow GuestEnabled -bool false

# Disable automatic login
sudo defaults delete /Library/Preferences/com.apple.loginwindow autoLoginUser
```

### Secure Boot Messaging

```bash
# Set login window text
sudo defaults write /Library/Preferences/com.apple.loginwindow LoginwindowText "Property of [Your Name]. If found, contact [email/phone]"

# Show hint for forgotten passwords
sudo defaults write /Library/Preferences/com.apple.loginwindow RetriesUntilHint -int 3
```

## Backup Strategy

### Time Machine Encryption

```bash
# Enable Time Machine encryption
tmutil setdestination -a -p /Volumes/BackupDrive

# Check encryption status
tmutil destinationinfo

# Start backup
tmutil startbackup

# List backups
tmutil listbackups

# Delete old backups
tmutil delete /Volumes/BackupDrive/Backups.backupdb/MachineName/YYYY-MM-DD-HHMMSS
```

### Encrypted Archives

```bash
# Create encrypted archive with strong password
tar czf - /path/to/directory | openssl enc -aes-256-cbc -pbkdf2 -out backup.tar.gz.enc

# Extract encrypted archive
openssl enc -d -aes-256-cbc -pbkdf2 -in backup.tar.gz.enc | tar xz

# Create encrypted disk image
hdiutil create -encryption AES-256 -size 10g -volname "Backup" -fs APFS backup.dmg

# Mount encrypted image
hdiutil attach backup.dmg
```

## Wi-Fi Security

### Wi-Fi Hardening

```bash
# Disable auto-join for networks
networksetup -setairportnetwork en0 ""

# List known networks
networksetup -listpreferredwirelessnetworks en0

# Remove network
sudo networksetup -removepreferredwirelessnetwork en0 "NetworkName"

# Disconnect from Wi-Fi
networksetup -setairportpower en0 off

# Connect to specific network
networksetup -setairportnetwork en0 "NetworkName" "password"

# Randomize MAC address (temporary)
sudo ifconfig en0 ether $(openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/.$//')

# Revert to hardware MAC
sudo ifconfig en0 ether $(networksetup -getmacaddress en0 | awk '{print $3}')
```

## Application Security

### Gatekeeper Management

```bash
# Check Gatekeeper status
spctl --status

# Enable Gatekeeper
sudo spctl --master-enable

# Verify app signature
spctl --assess --verbose /Applications/Firefox.app

# Allow unsigned app (not recommended)
xattr -rd com.apple.quarantine /Applications/UnsignedApp.app

# Check app notarization
spctl -a -vv -t install /Applications/App.app
```

### App Sandbox Verification

```bash
# Check if app is sandboxed
codesign -dvvv --entitlements - /Applications/App.app 2>&1 | grep -A 5 sandbox

# List app containers (sandboxed apps)
ls ~/Library/Containers/

# View app entitlements
codesign -d --entitlements - /Applications/App.app
```

## Lockdown Mode

Enable maximum security for high-risk users:

```bash
# Check Lockdown Mode status
defaults read com.apple.security.lockdownmode

# Note: Lockdown Mode must be enabled through System Settings
# System Settings > Privacy & Security > Lockdown Mode

# Verify Lockdown Mode features
defaults read /Library/Preferences/com.apple.security.lockdownmode
```

Lockdown Mode restricts:
- Message attachments and link previews
- Web browsing features (JIT compilation, fonts)
- Incoming FaceTime calls from unknown contacts
- Shared albums in Photos
- Wired connections to devices

## Security Checklist

### Daily/Weekly Tasks

```bash
#!/bin/bash
# security-check.sh - Run regular security checks

echo "=== Security Check ==="

# Check for system updates
echo "Checking for updates..."
softwareupdate -l

# Check firewall status
echo "Firewall status:"
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# Check FileVault status
echo "FileVault status:"
fdesetup status

# Check for suspicious processes
echo "Checking for high network usage:"
lsof -i -P -n | head -20

# Check failed login attempts
echo "Failed login attempts:"
sudo log show --predicate 'process == "sudo" OR process == "login"' --last 24h | grep -i "fail"

# Check listening services
echo "Listening services:"
sudo lsof -i -P | grep LISTEN

# Check for unsigned applications
echo "Recently modified applications:"
find /Applications -type d -name "*.app" -mtime -7

echo "=== Check Complete ==="
```

Make executable and run:

```bash
chmod +x security-check.sh
./security-check.sh
```

## Troubleshooting

### Reset Privacy Permissions

```bash
# Reset all TCC permissions
sudo tccutil reset All

# Reset specific service
sudo tccutil reset Camera
sudo tccutil reset Microphone
```

### DNS Issues

```bash
# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Test DNS resolution
scutil --dns
dig example.com

# Check DNS configuration
networksetup -getdnsservers Wi-Fi
```

### Firewall Connectivity Issues

```bash
# Temporarily disable firewall for testing
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off

# Check blocked connections
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --listapps

# Re-enable firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
```

### SSH Connection Problems

```bash
# Check SSH service status
sudo launchctl list | grep ssh

# Test SSH configuration
sudo sshd -t

# View SSH logs
sudo log show --predicate 'process == "sshd"' --last 1h

# Restart SSH service
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

## Additional Resources

- [NIST macOS
