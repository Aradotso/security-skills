---
name: spoof-suite-security-hardware
description: Hardware-level spoofing toolkit for device identity obfuscation, network masking, and fingerprint randomization across multiple layers
triggers:
  - how do I spoof hardware identifiers with Hardware-HardSp00f
  - configure device fingerprint randomization for privacy
  - set up multi-layer network identity spoofing
  - generate synthetic hardware profiles with HardSp00f
  - mask MAC address and device signatures
  - create adaptive entropy injection for hardware spoofing
  - implement ARP DNS and SNI spoofing strategies
  - troubleshoot hardware identity multiplexing issues
---

# Hardware-HardSp00f Security Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

Hardware-HardSp00f is a comprehensive hardware-level spoofing toolkit that obfuscates device identities across multiple layers (ARP, DNS, SNI, browser headers, device fingerprints). It operates at the kernel-to-application boundary to create synthetic hardware personas that pass validation checks while isolating your true hardware identity.

**Key capabilities:**
- Hardware identity multiplexing (CPU, memory, storage, network)
- Multi-layer spoofing (ARP, DNS, SNI, packet manipulation)
- Browser fingerprint randomization (canvas, WebGL, user-agent)
- Adaptive entropy injection from hardware thermal noise
- Session-bound cryptographic identity isolation

## Installation

### Download and Setup

```bash
# Download from official repository
git clone https://github.com/Shantanu-U69/Spoof-Suite-Security.git
cd Spoof-Suite-Security

# For Linux (requires root/sudo)
sudo ./install.sh

# Verify installation
hardsp00fd --version
```

### System Requirements

- **Windows**: Windows 10/11 (2026 builds), Admin privileges
- **Linux**: Kernel 5.15+, root access for kernel module
- **macOS**: SIP must be disabled (limited support)

### Dependencies

```bash
# Linux
sudo apt-get install build-essential linux-headers-$(uname -r)
sudo apt-get install libpcap-dev libnl-3-dev

# Arch-based
sudo pacman -S base-devel linux-headers libpcap libnl

# Load kernel module (Linux)
sudo modprobe hardsp00f
```

## Core Commands

### Basic Daemon Control

```bash
# Start daemon with default profile
hardsp00fd --start

# Start with custom profile
hardsp00fd --profile /path/to/profile.yaml --start

# Stop daemon
hardsp00fd --stop

# Check status
hardsp00fd --status

# Restart with verbose logging
hardsp00fd --restart --verbose 3
```

### Profile Management

```bash
# List available profiles
hardsp00fd --list-profiles

# Generate new random profile
hardsp00fd --generate-profile --output my_profile.yaml

# Validate profile
hardsp00fd --validate-profile stealth_session.yaml

# Apply profile without daemon mode
hardsp00fd --profile my_profile.yaml --mode oneshot
```

### Interface-Specific Operations

```bash
# Spoof specific interface
hardsp00fd --interface eth0 --randomize-mac

# List all interfaces
hardsp00fd --list-interfaces

# Target multiple interfaces
hardsp00fd --target "eth0,wlan0" --profile my_profile.yaml

# Preserve loopback
hardsp00fd --target all_network_interfaces --preserve-loopback
```

## Configuration

### Profile Structure

Create a YAML profile at `~/.config/hardsp00f/profiles/custom.yaml`:

```yaml
profile:
  name: "custom_stealth_2026"
  persona:
    architecture: "random_walk_plus_mode"
    entropy_source: "hardware_thermal_noise"
  
  device_fingerprint:
    cpu:
      vendor: ["GenuineIntel", "AuthenticAMD"]
      cores: [4, 6, 8, 12]
      hyperthreading: false
      cache_kb: [8192, 12288, 16384]
    
    memory:
      type: ["DDR4", "DDR5"]
      total_gb: [16, 32, 64]
      speed_mhz: [3200, 3600, 4800]
    
    storage:
      model: ["Samsung 990 Pro", "WD Black SN850X", "Crucial P5 Plus"]
      interface: "NVMe"
      size_gb: [512, 1024, 2048]
    
    network:
      mac_vendor: ["Intel Corporation", "Realtek Semiconductor", "Broadcom"]
      interface_type: "Gigabit Ethernet"
      link_speed: [1000, 2500, 10000]
  
  spoofing_strategy:
    arp:
      enabled: true
      interval_ms: 30000
      jitter_percent: 15
      replay_protection: true
    
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
      cipher_suites:
        - "ECDHE-RSA-AES128-GCM-SHA256"
        - "ECDHE-RSA-AES256-GCM-SHA384"
    
    browser_header:
      enabled: true
      user_agent_generator: "bayesian_decision_forest"
      canvas_noise_level: 0.02
      webgl_renderer: "ANGLE (Intel, Intel(R) UHD Graphics 630 Direct3D11 vs_5_0 ps_5_0)"
      accept_language: ["en-US,en;q=0.9", "en-GB,en;q=0.9"]
    
    packet:
      ttl_variation: [64, 128, 255]
      window_size_randomize: true
      fragment_strategy: "adaptive"
  
  persistence:
    mode: "until_reboot"
    restore_on_failure: true
    log_rotation_mb: 100
  
  logging:
    level: "info"
    destination: "/var/log/hardsp00f/custom_session.log"
    format: "structured_json"
```

### Environment Variables

```bash
# Set default profile directory
export HARDSP00F_PROFILE_DIR="$HOME/.config/hardsp00f/profiles"

# Set log directory
export HARDSP00F_LOG_DIR="/var/log/hardsp00f"

# Enable debug mode
export HARDSP00F_DEBUG=1

# Disable telemetry (if AI integration enabled)
export HARDSP00F_NO_TELEMETRY=1

# AI API keys (for profile generation)
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
```

## Code Examples

### Python API Usage

```python
#!/usr/bin/env python3
from hardsp00f import HardwareSpoofer, ProfileGenerator

# Initialize spoofer
spoofer = HardwareSpoofer()

# Load profile
profile = spoofer.load_profile("stealth_session_2026")

# Apply spoofing
spoofer.apply_profile(profile, interfaces=["eth0", "wlan0"])

# Check current status
status = spoofer.get_status()
print(f"Active interfaces: {status['interfaces']}")
print(f"MAC addresses: {status['mac_addresses']}")

# Generate random profile
generator = ProfileGenerator()
new_profile = generator.generate(
    cpu_vendors=["GenuineIntel"],
    memory_gb=[16, 32],
    entropy_source="thermal"
)

# Save profile
new_profile.save("/tmp/generated_profile.yaml")

# Stop spoofing
spoofer.stop()
```

### Programmatic MAC Randomization

```python
from hardsp00f.network import NetworkSpoofer

# Initialize network spoofer
net_spoofer = NetworkSpoofer()

# Randomize MAC for specific interface
net_spoofer.randomize_mac("eth0", vendor="Intel Corporation")

# Get current MAC
current_mac = net_spoofer.get_mac("eth0")
print(f"Current MAC: {current_mac}")

# Restore original MAC
net_spoofer.restore_mac("eth0")
```

### Browser Fingerprint Spoofing

```python
from hardsp00f.browser import BrowserSpoofer

# Initialize browser spoofer
browser_spoofer = BrowserSpoofer()

# Generate synthetic fingerprint
fingerprint = browser_spoofer.generate_fingerprint(
    user_agent_family="Chrome",
    canvas_noise=0.02,
    webgl_vendor="ANGLE",
    screen_resolution=(1920, 1080)
)

# Apply to current session
browser_spoofer.apply_fingerprint(fingerprint)

# Get current fingerprint hash
current_hash = browser_spoofer.get_fingerprint_hash()
print(f"Fingerprint hash: {current_hash}")
```

### AI-Assisted Profile Generation

```python
from hardsp00f.ai import AIProfileGenerator
import os

# Initialize with API keys from environment
ai_gen = AIProfileGenerator(
    openai_key=os.getenv("OPENAI_API_KEY"),
    anthropic_key=os.getenv("ANTHROPIC_API_KEY"),
    privacy_mode=True
)

# Generate contextual profile
profile = ai_gen.generate_for_context(
    target_environment="corporate_network",
    threat_model="medium",
    operating_system="windows_11"
)

# Optimize existing profile
optimized = ai_gen.optimize_profile(
    existing_profile="stealth_session.yaml",
    detection_risks=["arp_fingerprinting", "dns_correlation"]
)

# Save optimized profile
optimized.save("optimized_stealth.yaml")
```

## Common Patterns

### Complete Session Workflow

```bash
#!/bin/bash

# 1. Generate custom profile
hardsp00fd --generate-profile \
  --cpu-vendor Intel \
  --memory-gb 32 \
  --output /tmp/session_profile.yaml

# 2. Validate profile
hardsp00fd --validate-profile /tmp/session_profile.yaml

# 3. Start daemon with logging
hardsp00fd --profile /tmp/session_profile.yaml \
  --target all_network_interfaces \
  --mode adaptive_multiplex \
  --persistence until_reboot \
  --verbose 2 \
  --log /var/log/hardsp00f/session_$(date +%Y%m%d).log \
  --daemonize

# 4. Monitor status
watch -n 5 'hardsp00fd --status'

# 5. On completion, stop and cleanup
hardsp00fd --stop
hardsp00fd --cleanup-logs --older-than 7d
```

### Network Interface Rotation

```python
from hardsp00f.network import NetworkSpoofer
import time

spoofer = NetworkSpoofer()
interfaces = ["eth0", "wlan0"]

# Rotate MAC addresses every 5 minutes
for i in range(12):  # 1 hour total
    for iface in interfaces:
        spoofer.randomize_mac(iface)
        print(f"[{time.strftime('%H:%M:%S')}] Rotated {iface}: {spoofer.get_mac(iface)}")
    
    time.sleep(300)  # 5 minutes

# Restore original MACs
for iface in interfaces:
    spoofer.restore_mac(iface)
```

### Multi-Layer Spoofing Orchestration

```python
from hardsp00f import HardwareSpoofer, NetworkSpoofer, BrowserSpoofer

# Initialize all spoofers
hw_spoofer = HardwareSpoofer()
net_spoofer = NetworkSpoofer()
browser_spoofer = BrowserSpoofer()

# Load profile
profile = hw_spoofer.load_profile("stealth_session_2026")

# Apply hardware spoofing
hw_spoofer.apply_profile(profile)

# Apply network spoofing
net_spoofer.apply_arp_spoofing(interval=30000, jitter=15)
net_spoofer.apply_dns_spoofing(resolvers=["8.8.8.8", "1.1.1.1"])

# Apply browser spoofing
browser_spoofer.apply_fingerprint(
    browser_spoofer.generate_fingerprint(canvas_noise=0.02)
)

# Verify all layers active
status = {
    "hardware": hw_spoofer.is_active(),
    "network": net_spoofer.is_active(),
    "browser": browser_spoofer.is_active()
}
print(f"Spoofing layers: {status}")
```

## Troubleshooting

### Kernel Module Not Loading (Linux)

```bash
# Check if module exists
ls /lib/modules/$(uname -r)/extra/hardsp00f.ko

# Check dmesg for errors
dmesg | grep hardsp00f

# Rebuild module
cd /usr/src/hardsp00f
make clean && make
sudo make install
sudo modprobe hardsp00f

# Verify loaded
lsmod | grep hardsp00f
```

### MAC Randomization Not Persisting

```bash
# Check NetworkManager interference
sudo systemctl stop NetworkManager

# Apply MAC manually
sudo hardsp00fd --interface eth0 --randomize-mac --force

# Make persistent across reboots (systemd)
sudo systemctl enable hardsp00fd
```

### Permission Denied Errors

```bash
# Ensure running as root/admin
sudo hardsp00fd --start

# Check capabilities (Linux)
sudo setcap cap_net_admin,cap_net_raw+eip /usr/bin/hardsp00fd

# Check SELinux/AppArmor
sudo setenforce 0  # Temporarily disable SELinux
sudo aa-complain /usr/bin/hardsp00fd  # AppArmor
```

### Profile Validation Failures

```bash
# Check YAML syntax
yamllint ~/.config/hardsp00f/profiles/custom.yaml

# Validate with verbose output
hardsp00fd --validate-profile custom.yaml --verbose 3

# Test profile in dry-run mode
hardsp00fd --profile custom.yaml --dry-run
```

### AI Integration Issues

```python
# Test API connectivity
from hardsp00f.ai import AIProfileGenerator

ai_gen = AIProfileGenerator(
    openai_key=os.getenv("OPENAI_API_KEY"),
    debug=True
)

# Verify key validity
if ai_gen.test_connection():
    print("API connected successfully")
else:
    print("API connection failed - check key and network")
```

### High Performance Impact

```bash
# Reduce entropy generation frequency
hardsp00fd --profile custom.yaml --entropy-interval 60000  # 60 seconds

# Disable heavy features
hardsp00fd --profile custom.yaml \
  --disable browser_header \
  --disable unicode_spoofing

# Check resource usage
hardsp00fd --status --show-resources
```

### Log Analysis

```bash
# Parse structured JSON logs
cat /var/log/hardsp00f/session.log | jq '.[] | select(.level=="error")'

# Monitor real-time
tail -f /var/log/hardsp00f/session.log | jq .

# Export session summary
hardsp00fd --export-session-stats --output /tmp/stats.json
```

## Security Considerations

**Legal Compliance**: Only use on networks and systems you own or have explicit authorization to test. Unauthorized spoofing may violate laws.

**Ethical Usage**: Designed for security research, penetration testing, and privacy research only.

**Session Isolation**: Always terminate sessions properly to avoid identity leaks:

```bash
# Proper shutdown
hardsp00fd --stop --cleanup

# Emergency restore
hardsp00fd --emergency-restore
```

**Log Management**: Regularly clean logs containing spoofed identities:

```bash
hardsp00fd --cleanup-logs --secure-delete
```
