---
name: macos-security-privacy-hardening
description: Secure and harden macOS systems following enterprise-standard security practices and privacy guidelines
triggers:
  - how do I secure my macOS system
  - harden macOS security settings
  - configure macOS privacy and security
  - setup FileVault and firmware password
  - configure macOS firewall and DNS encryption
  - secure macOS installation and setup
  - macOS security best practices
  - improve macOS privacy settings
---

# macOS Security and Privacy Hardening

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive guidance for securing and hardening macOS systems based on the drduh/macOS-Security-and-Privacy-Guide. It covers security configurations, privacy settings, encryption, firewalls, and monitoring for Apple silicon Macs running currently supported macOS versions.

## Overview

The macOS Security and Privacy Guide provides enterprise-standard security practices for:

- **System hardening**: Firmware passwords, FileVault encryption, secure boot
- **Privacy protection**: Disabling telemetry, configuring DNS encryption, certificate management
- **Network security**: Firewalls, VPN configuration, DNS filtering
- **Access control**: User account separation, authentication policies
- **Monitoring**: System auditing, network monitoring, execution tracking

**Important**: This guide targets Apple silicon Macs. Intel Macs have unpatched hardware vulnerabilities and are not recommended.

## Threat Modeling

Before applying security measures, create a threat model:

### Identify Assets

List what you're protecting:
- Devices (phone, laptop)
- Data (passwords, browsing history, documents)
- Accounts (email, banking, social media)

### Identify Adversaries

Define who you're defending against:
- **Casual attacker**: Roommate, opportunistic thief
- **Criminal**: Malware distribution, financial fraud
- **Corporation**: Data collection, behavioral tracking
- **Nation state/APT**: Targeted surveillance, advanced persistent threats

### Example Threat Model Table

```markdown
| Adversary    | Motivation        | Capabilities                | Mitigation                           |
|--------------|-------------------|----------------------------|--------------------------------------|
| Roommate     | Privacy invasion  | Physical access to device  | Use biometrics, screen lock          |
| Thief        | Financial gain    | Steal unlocked device      | Find My, device encryption           |
| Criminal     | Financial         | Malware, social engineering| Sandboxing, automatic updates        |
| Corporation  | Data marketing    | Telemetry collection       | Block connections, disable telemetry |
| Nation State | Surveillance      | Network monitoring         | E2EE, strong passwords, hardware keys|
```

## System Installation

### Secure Installation Process

1. **Download macOS**: Use the latest supported version for your Mac
   ```bash
   # Check current macOS version
   sw_vers
   
   # Check for updates
   softwareupdate --list
   
   # Install all updates
   sudo softwareupdate --install --all
   ```

2. **Create bootable installer** (if doing clean install):
   ```bash
   # Download macOS installer from App Store first
   sudo /Applications/Install\ macOS\ Sonoma.app/Contents/Resources/createinstallmedia \
     --volume /Volumes/MyVolume
   ```

3. **System Activation**: Apple silicon Macs require activation with Apple servers during installation for theft prevention

### Initial Setup

Skip Apple Account creation during setup if not needed. You can install system updates without an Apple Account:

```bash
# Set automatic updates via command line
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticCheckEnabled -bool true
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticDownload -bool true
sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticallyInstallMacOSUpdates -bool true
```

## Admin and User Accounts

### Principle of Least Privilege

Separate admin and standard user accounts:

```bash
# Create admin account (during initial setup)
# Then create standard user account:

# List users
dscl . list /Users | grep -v '^_'

# Create new standard user (replace USERNAME)
sudo dscl . -create /Users/USERNAME
sudo dscl . -create /Users/USERNAME UserShell /bin/zsh
sudo dscl . -create /Users/USERNAME RealName "User Name"
sudo dscl . -create /Users/USERNAME UniqueID 501
sudo dscl . -create /Users/USERNAME PrimaryGroupID 20
sudo dscl . -create /Users/USERNAME NFSHomeDirectory /Users/USERNAME
sudo dscl . -passwd /Users/USERNAME
sudo dscl . -append /Groups/com.apple.access_ssh GroupMembership USERNAME

# Create home directory
sudo createhomedir -c -u USERNAME

# Verify user is not admin
dscl . -read /Groups/admin GroupMembership
```

### Require Administrator Password

```bash
# Require password for system preferences
sudo security authorizationdb write system.preferences authenticate-admin

# Set password requirements
sudo pwpolicy -setglobalpolicy "minChars=12 requiresAlpha=1 requiresNumeric=1"
```

## Firmware Password

Set a firmware password to prevent booting from external media:

```bash
# Check if firmware password is set
sudo firmwarepasswd -check

# Set firmware password (Apple silicon)
# Must be done in Recovery Mode:
# 1. Restart and hold power button until "Loading startup options" appears
# 2. Click Options, then Utilities > Startup Security Utility
# 3. Turn on firmware password
```

## FileVault Encryption

Enable full-disk encryption:

```bash
# Check FileVault status
sudo fdesetup status

# Enable FileVault (creates recovery key - SAVE THIS SECURELY)
sudo fdesetup enable

# List FileVault users
sudo fdesetup list

# Add user to FileVault
sudo fdesetup add -usertoadd USERNAME

# Change FileVault password
sudo fdesetup changepassword -user USERNAME
```

**Important**: Save the recovery key in a secure location. Without it, data is unrecoverable if you forget your password.

## Lockdown Mode

For high-threat models, enable Lockdown Mode:

```bash
# Check Lockdown Mode status (no direct command, use UI or defaults)
defaults read /Library/Preferences/com.apple.security LockdownModeEnabled

# Enable via: System Settings > Privacy & Security > Lockdown Mode
```

Lockdown Mode restrictions:
- Most message attachments blocked
- Web technologies restricted (JIT, fonts)
- Wired connections blocked when locked
- Configuration profiles blocked

## Firewall Configuration

### Application Layer Firewall

```bash
# Enable built-in firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# Enable logging
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setloggingmode on

# Enable stealth mode (don't respond to probes)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Check status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Packet Filter (PF)

Create advanced firewall rules with PF:

```bash
# Create PF configuration
sudo nano /etc/pf.conf
```

Example `/etc/pf.conf`:
```pf
# Interfaces
ext_if = "en0"
lo_if = "lo0"

# Default deny
set block-policy drop
set skip on lo

# Scrub incoming packets
scrub in all

# Block all by default
block log all

# Allow established connections
pass in quick proto tcp from any to any flags S/SA keep state
pass out quick keep state

# Allow DNS
pass out quick proto {tcp udp} to any port 53

# Allow HTTPS
pass out quick proto tcp to any port 443

# Allow NTP
pass out quick proto udp to any port 123

# Block Facebook, Google, etc. (example)
table <blocklist> persist file "/etc/pf.blocklist"
block drop quick from any to <blocklist>
```

Create blocklist:
```bash
# Create blocklist file
sudo nano /etc/pf.blocklist
```

Example `/etc/pf.blocklist`:
```
# Facebook
31.13.64.0/18
66.220.144.0/20
69.63.176.0/20

# Google
216.58.192.0/19
172.217.0.0/16
```

Enable PF:
```bash
# Check syntax
sudo pfctl -vnf /etc/pf.conf

# Enable PF
sudo pfctl -ef /etc/pf.conf

# View rules
sudo pfctl -sr

# View blocked packets
sudo pfctl -si

# Flush rules
sudo pfctl -F all
```

## Disable Services

Minimize attack surface by disabling unnecessary services:

```bash
# Disable Spotlight suggestions
defaults write com.apple.safari UniversalSearchEnabled -bool false
defaults write com.apple.safari SuppressSearchSuggestions -bool true

# Disable Siri
defaults write com.apple.assistant.support "Assistant Enabled" -bool false
launchctl disable "user/$UID/com.apple.assistantd"
launchctl disable "gui/$UID/com.apple.assistantd"
sudo launchctl disable 'system/com.apple.assistantd'

# Disable Handoff
defaults write ~/Library/Preferences/ByHost/com.apple.coreservices.useractivityd ActivityAdvertisingAllowed -bool false
defaults write ~/Library/Preferences/ByHost/com.apple.coreservices.useractivityd ActivityReceivingAllowed -bool false

# Disable AirDrop
defaults write com.apple.NetworkBrowser DisableAirDrop -bool true

# Disable Bonjour multicast advertisements
sudo defaults write /Library/Preferences/com.apple.mDNSResponder.plist NoMulticastAdvertisements -bool true

# Disable infrared receiver
sudo defaults write /Library/Preferences/com.apple.driver.AppleIRController DeviceEnabled -bool false

# Disable Bluetooth if not needed
sudo defaults write /Library/Preferences/com.apple.Bluetooth ControllerPowerState -int 0
sudo killall -HUP bluetoothd
```

## DNS Configuration

### DNS Encryption with DNSCrypt

Install and configure DNSCrypt-proxy:

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dnscrypt-proxy
brew install dnscrypt-proxy

# Configure DNSCrypt
nano $(brew --prefix)/etc/dnscrypt-proxy.toml
```

Example DNSCrypt configuration:
```toml
server_names = ['cloudflare', 'cloudflare-ipv6']
listen_addresses = ['127.0.0.1:53']
max_clients = 250
ipv4_servers = true
ipv6_servers = true
dnscrypt_servers = true
doh_servers = true
require_dnssec = true
require_nolog = true
require_nofilter = false
force_tcp = false

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

# Configure system to use DNSCrypt
networksetup -setdnsservers Wi-Fi 127.0.0.1
networksetup -setdnsservers Ethernet 127.0.0.1

# Verify DNS
scutil --dns
```

### DNS Configuration Profiles

Create a configuration profile for encrypted DNS:

```bash
# Create DNS profile XML
cat > ~/cloudflare-dns.mobileconfig << 'EOF'
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
                <key>ServerAddresses</key>
                <array>
                    <string>1.1.1.1</string>
                    <string>1.0.0.1</string>
                </array>
                <key>ServerURL</key>
                <string>https://cloudflare-dns.com/dns-query</string>
            </dict>
            <key>PayloadType</key>
            <string>com.apple.dnsSettings.managed</string>
            <key>PayloadIdentifier</key>
            <string>com.cloudflare.1dot1dot1dot1</string>
            <key>PayloadUUID</key>
            <string>A1E3F4E3-5B4A-4F1E-8E3D-123456789ABC</string>
            <key>PayloadDisplayName</key>
            <string>Cloudflare DNS</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
        </dict>
    </array>
    <key>PayloadType</key>
    <string>Configuration</string>
    <key>PayloadIdentifier</key>
    <string>com.cloudflare.1dot1dot1dot1</string>
    <key>PayloadUUID</key>
    <string>B2F4G5F4-6C5B-5G2F-9F4E-234567890BCD</string>
    <key>PayloadDisplayName</key>
    <string>Cloudflare DNS</string>
    <key>PayloadVersion</key>
    <integer>1</integer>
</dict>
</plist>
EOF

# Install profile (will prompt for password)
sudo profiles install -path ~/cloudflare-dns.mobileconfig

# Verify
sudo profiles list
```

### Hosts File Blocking

Block tracking domains via hosts file:

```bash
# Backup current hosts file
sudo cp /etc/hosts /etc/hosts.backup

# Download blocklist (using StevenBlack's hosts file)
curl https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts | \
  sudo tee -a /etc/hosts

# Or create custom entries
sudo nano /etc/hosts
```

Example custom hosts entries:
```
# Block Facebook
0.0.0.0 facebook.com
0.0.0.0 www.facebook.com
0.0.0.0 m.facebook.com

# Block Google Analytics
0.0.0.0 google-analytics.com
0.0.0.0 www.google-analytics.com
0.0.0.0 ssl.google-analytics.com

# Block ads
0.0.0.0 ads.example.com
0.0.0.0 tracking.example.com
```

Flush DNS cache:
```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

## Certificate Management

Manage trusted root certificates:

```bash
# List certificates
security dump-keychain -d /System/Library/Keychains/SystemRootCertificates.keychain

# Export certificates
security export -k /System/Library/Keychains/SystemRootCertificates.keychain \
  -t certs -o ~/root-certs.pem

# Disable certificate (example - Chinese CA)
sudo security delete-certificate -c "CNNIC ROOT" \
  /System/Library/Keychains/SystemRootCertificates.keychain

# Add custom CA (e.g., for corporate proxy)
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain ~/custom-ca.crt

# View certificate details
security find-certificate -c "Certificate Name" -p \
  /System/Library/Keychains/SystemRootCertificates.keychain | \
  openssl x509 -text -noout
```

## Browser Security

### Firefox Hardening

Install Firefox and configure for privacy:

```bash
# Install Firefox
brew install --cask firefox

# Firefox config location
# ~/Library/Application Support/Firefox/Profiles/*.default-release/user.js
```

Create `user.js` for privacy:
```javascript
// Disable telemetry
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);

// Enable tracking protection
user_pref("privacy.trackingprotection.enabled", true);
user_pref("privacy.trackingprotection.socialtracking.enabled", true);
user_pref("privacy.trackingprotection.fingerprinting.enabled", true);
user_pref("privacy.trackingprotection.cryptomining.enabled", true);

// Enable HTTPS-only mode
user_pref("dom.security.https_only_mode", true);
user_pref("dom.security.https_only_mode_ever_enabled", true);

// Disable WebRTC (prevents IP leaks)
user_pref("media.peerconnection.enabled", false);

// Enable DNS over HTTPS
user_pref("network.trr.mode", 2);
user_pref("network.trr.uri", "https://cloudflare-dns.com/dns-query");

// Disable referer
user_pref("network.http.referer.XOriginPolicy", 2);

// Clear data on shutdown
user_pref("privacy.sanitize.sanitizeOnShutdown", true);
user_pref("privacy.clearOnShutdown.cache", true);
user_pref("privacy.clearOnShutdown.cookies", true);
user_pref("privacy.clearOnShutdown.history", true);

// Disable geolocation
user_pref("geo.enabled", false);

// Resist fingerprinting
user_pref("privacy.resistFingerprinting", true);
```

### Safari Hardening

```bash
# Disable pre-loading top hit
defaults write com.apple.Safari PreloadTopHit -bool false

# Disable search suggestions
defaults write com.apple.Safari UniversalSearchEnabled -bool false
defaults write com.apple.Safari SuppressSearchSuggestions -bool true

# Enable tracking prevention
defaults write com.apple.Safari WebKitPreferences.privateClickMeasurementEnabled -bool false
defaults write com.apple.Safari SendDoNotTrackHTTPHeader -bool true

# Disable autofill
defaults write com.apple.Safari AutoFillPasswords -bool false
defaults write com.apple.Safari AutoFillCreditCardData -bool false

# Enable fraudulent site warning
defaults write com.apple.Safari WarnAboutFraudulentWebsites -bool true

# Disable plugins
defaults write com.apple.Safari com.apple.Safari.ContentPageGroupIdentifier.WebKit2JavaEnabled -bool false
defaults write com.apple.Safari com.apple.Safari.ContentPageGroupIdentifier.WebKit2JavaEnabledForLocalFiles -bool false

# Block pop-ups
defaults write com.apple.Safari com.apple.Safari.ContentPageGroupIdentifier.WebKit2JavaScriptCanOpenWindowsAutomatically -bool false
```

## VPN Configuration

### WireGuard Setup

```bash
# Install WireGuard
brew install wireguard-tools

# Create keys
umask 077
wg genkey | tee privatekey | wg pubkey > publickey

# Create configuration
sudo nano /usr/local/etc/wireguard/wg0.conf
```

Example WireGuard configuration:
```ini
[Interface]
PrivateKey = <PRIVATE_KEY_FROM_FILE>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

Start WireGuard:
```bash
# Start VPN
sudo wg-quick up wg0

# Check status
sudo wg show

# Stop VPN
sudo wg-quick down wg0

# Auto-start on boot
sudo ln -sf /usr/local/etc/wireguard/wg0.conf /usr/local/etc/wireguard/wg0.conf
```

## PGP/GPG Configuration

Install and configure GPG:

```bash
# Install GPG
brew install gnupg

# Generate key
gpg --full-generate-key
# Choose: (1) RSA and RSA
# Key size: 4096
# Expiration: 1y (recommended)

# List keys
gpg --list-secret-keys --keyid-format LONG

# Export public key
gpg --armor --export YOUR_EMAIL > publickey.asc

# Export private key (KEEP SECURE)
gpg --armor --export-secret-keys YOUR_EMAIL > privatekey.asc

# Encrypt file
gpg --encrypt --recipient YOUR_EMAIL file.txt

# Decrypt file
gpg --decrypt file.txt.gpg > file.txt

# Sign file
gpg --sign file.txt

# Verify signature
gpg --verify file.txt.gpg
```

Configure GPG agent:
```bash
# Create/edit GPG agent config
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
nano ~/.gnupg/gpg-agent.conf
```

GPG agent configuration:
```
default-cache-ttl 600
max-cache-ttl 7200
enable-ssh-support
pinentry-program /usr/local/bin/pinentry-mac
```

```bash
# Restart GPG agent
gpgconf --kill gpg-agent
gpg-agent --daemon
```

## System Monitoring

### OpenBSM Audit

Enable system auditing:

```bash
# Check audit status
sudo audit -s

# Enable auditing
sudo audit -i

# Configure audit
sudo nano /etc/security/audit_control
```

Example audit configuration:
```
dir:/var/audit
flags:lo,ad,fd,fm,-all
minfree:5
naflags:lo,aa
policy:cnt,argv
filesz:2M
expire-after:10M
```

Start auditing:
```bash
# Start audit daemon
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.auditd.plist

# View audit logs
sudo praudit -xn /var/audit/current
```

### Network Monitoring

Monitor network connections:

```bash
# List open connections
sudo lsof -i

# Monitor specific port
sudo lsof -i :443

# List listening services
sudo lsof -iTCP -sTCP:LISTEN

# netstat alternative
sudo lsof -nP -iTCP -sTCP:LISTEN

# Monitor DNS queries (if using DNSCrypt)
tail -f /var/log/dnscrypt-proxy/query.log

# Network statistics
nettop -m route

# Packet capture
sudo tcpdump -i en0 -n
```

### Process Monitoring

Monitor running processes:

```bash
# List processes with network connections
lsof -i

# Monitor process execution (requires SIP modification - not recommended)
# Use fs_usage instead
sudo fs_usage -w -f filesystem

# Monitor specific process
sudo fs_usage -f pathname $(pgrep ProcessName)

# List launch daemons and agents
launchctl list

# Check for suspicious processes
ps aux | grep -v root

# Monitor file changes
fswatch -0 ~/Documents | xargs -0 -n 1 echo "Changed:"
```

### Little Snitch Alternative (Free)

Use built-in tools for network monitoring:

```bash
# Create logging script
cat > ~/network-monitor.sh << 'EOF'
#!/bin/bash
while true; do
    echo "=== $(date) ==="
    lsof -i -P -n | grep ESTABLISHED
    sleep 5
done
EOF

chmod +x ~/network-monitor.sh

# Run in background
~/network-monitor.sh > ~/network-connections.log 2>&1 &
```

## SSH Hardening

Configure SSH for security:

```bash
# Generate SSH key (Ed25519)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or RSA 4096 (if Ed25519 not supported)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Configure SSH client
nano ~/.ssh/config
```

Example SSH config:
```
# Global defaults
Host *
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3
    
# Specific host
Host myserver
    HostName server.example.com
    User username
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    
# Use ProxyJump for bastion
Host private-server
    HostName 10.0.1.5
    User username
    ProxyJump bastion.example.com
```

Harden SSH daemon (if running SSH server):

```bash
# Edit SSH daemon config
sudo nano /etc/ssh/sshd_config
```

Recommended sshd_config:
```
# Disable root login
PermitRootLogin no

# Disable password authentication
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# Enable public key authentication
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Limit users
AllowUsers your_username

# Change port (optional)
Port 2222

# Protocol
Protocol 2

# Logging
SyslogFacility AUTH
LogLevel INFO

# Disconnect idle sessions
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable X11 forwarding (if not needed)
X11Forwarding no

# Disable TCP forwarding (if not needed)
AllowTcpForwarding no
```

Restart SSH:
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

## Physical Security

### Lock Screen Settings

```bash
# Require password immediately after sleep/screensaver
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

# Set screensaver timeout (seconds)
defaults -currentHost write com.apple.screensaver idleTime -int 300

# Lock screen with hot corner (bottom left)
defaults write com.apple.dock wvous-bl-corner -int 6
defaults write com.apple.dock wvous-bl-modifier -int 0
killall Dock

# Show message on lock screen
sudo defaults write /Library/Preferences/com.apple.loginwindow LoginwindowText \
  "If found, please contact: +1-555-0100"

# Disable automatic login
sudo defaults delete /Library/Preferences/com.apple.loginwindow autoLoginUser

# Disable guest account
sudo defaults write /Library/Preferences/com.apple.loginwindow GuestEnabled -bool false
```

### Hibernate Mode

```bash
# Check current hibernate mode
pmset -g | grep hibernatemode

# Set hibernate mode (3 = copy RAM to disk, power down)
sudo pmset -a hibernatemode 3

# Secure virtual memory (swapfile encryption)
sudo defaults write /Library/Preferences/com.apple.virtualMemory UseEncryptedSwap -bool yes

# Destroy FileVault keys on standby
sudo pmset -a destroyfvkeyonstandby 1

# Enable power nap (optional - may reduce security)
sudo pmset -a powernap 0
```

## Metadata and Artifacts

Remove metadata from files:

```bash
# Remove extended attributes
xattr -cr /path/to/file

# View metadata
mdls /path/to/file

# Clear spotlight metadata
sudo mdutil -E /

# Remove all metadata from file
exiftool -all= file.jpg

# Or use ImageOptim for images
brew install --cask imageoptim
```

Clear system artifacts:

```bash
# Clear QuickLook cache
qlmanage -r cache

# Clear system logs
sudo rm -rf /var/log/*.log
sudo rm -rf ~/Library/Logs/*

# Clear bash history
cat /dev/null > ~/.bash_history && history -c

# Clear zsh history
cat /dev/null > ~/.zsh_history && history -c

# Secure delete (overwrite)
rm -P sensitive-file.txt

# Or use srm (install separately)
brew install srm
srm -v sensitive-file.txt

# Securely erase free space (APFS - runs in background)
diskutil secureErase freespace 0 /Volumes/Macintosh\ HD
```

## Password Management

Use built-in password manager or dedicated solution:

```bash
# Access keychain from command line
security find-generic-password -s "Service Name" -a "Account Name"

# Add password to keychain
security add-generic-password -a "account" -s "service" -w "password"

# Generate random password
openssl rand -base64 32

# Or use pwgen
brew install pwgen
pwgen -sy 32 1

# Create diceware passphrase
brew install diceware
diceware -n 6 --no-caps
```

### KeePassXC Setup

```bash
# Install KeePassXC
brew install --cask keepassxc

# Database location (example)
# ~/Documents/passwords.kdbx

# Enable browser integration in KeePassXC settings
# Install browser extension for Firefox/Safari
```

## Backup Strategy

### Time Machine

```bash
# List Time Machine destinations
tmutil destinationinfo

# Start backup
tmutil startbackup

# Exclude directories from backup
tmutil addexclusion ~/Downloads
tmutil addexclusion ~/Movies

# Verify Time Machine encryption
diskutil info /Volumes/Time\ Machine | grep Encrypted

# Encrypt Time Machine backup (when setting up)
# System Settings > General > Time Machine > Select Disk > Encrypt backups
```

### Manual Encrypted Backup

```bash
# Create encrypted disk image for backup
hdiutil create -size 100g -encryption AES-256 \
  -volname "Backup" -fs APFS ~/backup.dmg

# Mount encrypted image
hdiutil attach ~/backup.dmg

# Backup files (using rsync)
rsync -av --delete ~/Documents/ /Volumes/Backup/Documents/

# Unmount
hdiutil detach /Volumes/Backup
```

## Wi-Fi Security

```bash
# Forget Wi-Fi network
sudo networksetup -removepreferredwirelessnetwork en0 "NetworkName"

# Disable Wi-Fi
networksetup -setairportpower en0 off

# Randomize MAC address (per network, automatic on modern macOS)
# View current MAC
ifconfig en0 | grep ether

# Set MAC address (requires disabling SIP)
# sudo
