---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for CAN bus, automotive protocols, and vulnerability assessment
triggers:
  - test vehicle network security
  - scan CAN bus vulnerabilities
  - perform automotive security testing
  - analyze vehicle network traffic
  - use S800 testing framework
  - test automotive protocols
  - assess vehicle cybersecurity
  - fuzz CAN bus messages
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and vulnerability assessment of automotive networks, particularly CAN (Controller Area Network) bus systems. It provides tools for traffic analysis, fuzzing, simulation, and exploitation of vehicle communication protocols.

**Note**: This framework is intended for authorized security testing, research, and educational purposes only. Always obtain proper authorization before testing vehicle systems.

## Installation

### Prerequisites

```bash
# Install required system dependencies (Linux)
sudo apt-get update
sudo apt-get install -y can-utils python3 python3-pip git

# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Create virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Or install common dependencies manually
pip3 install python-can cantools scapy pyserial
```

### Hardware Setup (Optional)

For physical CAN bus testing, connect a CAN adapter:

```bash
# Configure physical CAN interface (example for SocketCAN)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ip -details link show can0
```

## Core Features

### 1. CAN Bus Sniffing

Capture and analyze CAN bus traffic:

```python
import can
import time

def sniff_can_traffic(interface='vcan0', duration=10):
    """Sniff CAN bus traffic for analysis"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Sniffing on {interface} for {duration} seconds...")
    messages = []
    
    start_time = time.time()
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append(msg)
            print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    
    bus.shutdown()
    return messages

# Usage
captured_msgs = sniff_can_traffic(interface='can0', duration=30)
```

### 2. CAN Message Injection

Send crafted CAN messages for testing:

```python
import can

def send_can_message(interface, arb_id, data):
    """Send a single CAN message"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"[+] Sent: ID=0x{arb_id:03X} Data={data.hex()}")
    except can.CanError as e:
        print(f"[-] Error sending message: {e}")
    finally:
        bus.shutdown()

# Example: Send door unlock command (example ID/data)
send_can_message('can0', 0x123, bytes([0x01, 0x02, 0x03, 0x04]))
```

### 3. CAN Bus Fuzzing

Automated fuzzing for vulnerability discovery:

```python
import can
import random
import time

def fuzz_can_bus(interface, target_ids=None, duration=60):
    """Fuzz CAN bus with random messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    if target_ids is None:
        # Common automotive CAN IDs range
        target_ids = range(0x100, 0x7FF)
    
    print(f"[*] Starting CAN fuzzing for {duration} seconds...")
    start_time = time.time()
    fuzz_count = 0
    
    try:
        while time.time() - start_time < duration:
            arb_id = random.choice(target_ids)
            data_length = random.randint(1, 8)
            data = bytes([random.randint(0, 255) for _ in range(data_length)])
            
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            bus.send(msg)
            fuzz_count += 1
            time.sleep(0.01)  # Adjust for desired rate
            
    except KeyboardInterrupt:
        print("\n[!] Fuzzing interrupted by user")
    finally:
        bus.shutdown()
        print(f"[*] Sent {fuzz_count} fuzz messages")

# Fuzz specific IDs
fuzz_can_bus('can0', target_ids=[0x123, 0x456, 0x789], duration=30)
```

### 4. CAN Traffic Replay

Replay captured traffic for testing:

```python
import can
import time

def replay_can_traffic(interface, log_file, speed_multiplier=1.0):
    """Replay CAN messages from a log file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Replaying traffic from {log_file}...")
    
    try:
        # Read log file (ASC or other format)
        with open(log_file, 'r') as f:
            messages = []
            prev_timestamp = None
            
            for line in f:
                # Parse message (example for simple format)
                # Format: timestamp,id,data
                parts = line.strip().split(',')
                if len(parts) >= 3:
                    timestamp = float(parts[0])
                    arb_id = int(parts[1], 16)
                    data = bytes.fromhex(parts[2])
                    
                    if prev_timestamp:
                        delay = (timestamp - prev_timestamp) / speed_multiplier
                        time.sleep(max(0, delay))
                    
                    msg = can.Message(arbitration_id=arb_id, data=data)
                    bus.send(msg)
                    print(f"[+] Replayed: ID=0x{arb_id:03X}")
                    
                    prev_timestamp = timestamp
                    
    except Exception as e:
        print(f"[-] Replay error: {e}")
    finally:
        bus.shutdown()

# Replay at 2x speed
replay_can_traffic('can0', 'captured_traffic.log', speed_multiplier=2.0)
```

### 5. UDS (Unified Diagnostic Services) Testing

Test diagnostic protocols:

```python
import can
import time

class UDSScanner:
    def __init__(self, interface, request_id=0x7DF, response_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.request_id = request_id
        self.response_id = response_id
    
    def send_uds_request(self, service, data=None):
        """Send UDS request and wait for response"""
        payload = [service]
        if data:
            payload.extend(data)
        
        msg = can.Message(
            arbitration_id=self.request_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
        
        # Wait for response
        timeout = time.time() + 2.0
        while time.time() < timeout:
            response = self.bus.recv(timeout=0.5)
            if response and response.arbitration_id == self.response_id:
                return response.data
        
        return None
    
    def scan_uds_services(self):
        """Scan for supported UDS services"""
        print("[*] Scanning UDS services...")
        supported = []
        
        # Common UDS service IDs
        services = {
            0x10: "DiagnosticSessionControl",
            0x11: "ECUReset",
            0x22: "ReadDataByIdentifier",
            0x27: "SecurityAccess",
            0x2E: "WriteDataByIdentifier",
            0x31: "RoutineControl",
            0x3E: "TesterPresent"
        }
        
        for sid, name in services.items():
            response = self.send_uds_request(sid)
            if response and response[0] == sid + 0x40:
                print(f"[+] Service 0x{sid:02X} ({name}) - Supported")
                supported.append(sid)
            elif response and response[0] == 0x7F:
                print(f"[-] Service 0x{sid:02X} ({name}) - Rejected (NRC: 0x{response[2]:02X})")
        
        return supported
    
    def close(self):
        self.bus.shutdown()

# Usage
scanner = UDSScanner('can0')
supported_services = scanner.scan_uds_services()
scanner.close()
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Set CAN bitrate
export S800_CAN_BITRATE=500000

# Enable verbose logging
export S800_DEBUG=1

# Set log output directory
export S800_LOG_DIR=/var/log/s800
```

### Configuration File

Create `s800_config.json`:

```json
{
  "interface": {
    "type": "socketcan",
    "channel": "can0",
    "bitrate": 500000
  },
  "logging": {
    "enabled": true,
    "directory": "./logs",
    "format": "asc"
  },
  "fuzzing": {
    "rate": 100,
    "target_ids": [256, 512, 768],
    "data_length_range": [1, 8]
  },
  "uds": {
    "request_id": "0x7DF",
    "response_id": "0x7E8",
    "timeout": 2.0
  }
}
```

## Common Testing Patterns

### Pattern 1: Baseline Traffic Analysis

```python
import can
from collections import defaultdict

def analyze_baseline_traffic(interface, duration=60):
    """Establish baseline of normal CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    id_stats = defaultdict(lambda: {'count': 0, 'data_patterns': set()})
    
    print(f"[*] Collecting baseline for {duration} seconds...")
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            id_stats[msg.arbitration_id]['count'] += 1
            id_stats[msg.arbitration_id]['data_patterns'].add(msg.data.hex())
    
    bus.shutdown()
    
    # Print analysis
    print("\n[*] Baseline Analysis:")
    for arb_id, stats in sorted(id_stats.items()):
        print(f"ID 0x{arb_id:03X}: {stats['count']} messages, "
              f"{len(stats['data_patterns'])} unique patterns")
    
    return id_stats

baseline = analyze_baseline_traffic('can0', duration=120)
```

### Pattern 2: Differential Testing

```python
def differential_can_test(interface, action_description):
    """Capture traffic before and after an action"""
    print(f"[*] Capturing BEFORE {action_description}...")
    before = sniff_can_traffic(interface, duration=10)
    
    input(f"[!] Perform action: {action_description} - Press Enter when done...")
    
    print(f"[*] Capturing AFTER {action_description}...")
    after = sniff_can_traffic(interface, duration=10)
    
    # Find differences
    before_ids = {msg.arbitration_id: msg.data for msg in before}
    after_ids = {msg.arbitration_id: msg.data for msg in after}
    
    print("\n[*] Differences detected:")
    for arb_id in set(list(before_ids.keys()) + list(after_ids.keys())):
        if arb_id not in before_ids:
            print(f"[+] NEW ID: 0x{arb_id:03X}")
        elif arb_id not in after_ids:
            print(f"[-] REMOVED ID: 0x{arb_id:03X}")
        elif before_ids[arb_id] != after_ids[arb_id]:
            print(f"[~] CHANGED ID: 0x{arb_id:03X}")
            print(f"    Before: {before_ids[arb_id].hex()}")
            print(f"    After:  {after_ids[arb_id].hex()}")

# Test unlocking door
differential_can_test('can0', "Unlock driver door with key fob")
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Ensure CAN modules are loaded
lsmod | grep can

# Reload modules
sudo modprobe -r vcan
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to dialout/can group
sudo usermod -aG dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 s800_test.py
```

### No Messages Received

```python
# Verify interface is up and configured
import can

try:
    bus = can.interface.Bus(channel='can0', bustype='socketcan')
    print("[+] Interface connected successfully")
except Exception as e:
    print(f"[-] Error: {e}")
```

### Bitrate Mismatch

```bash
# Reconfigure bitrate (common rates: 125000, 250000, 500000, 1000000)
sudo ip link set can0 down
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up
```

## Safety and Legal Considerations

- **Never test on vehicles in motion or critical systems without proper safety measures**
- **Always obtain written authorization before testing**
- **Use isolated test benches when possible**
- **Keep emergency shutdown procedures ready**
- **Document all testing activities**
- **Comply with local laws and regulations regarding vehicle security research**
