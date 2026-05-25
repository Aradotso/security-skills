---
name: s800-vehicle-network-security-testing
description: Test and analyze vehicle network security, focusing on CAN bus and automotive protocol vulnerabilities
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - scan automotive protocols
  - use S800 vehicle testing framework
  - check car network security
  - test vehicle ECU communication
  - analyze automotive network traffic
  - perform vehicle penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800-Vehicle-Network-Security-Testing-Framework is a security testing tool designed for automotive network analysis and vulnerability assessment. It focuses on CAN (Controller Area Network) bus security testing, protocol analysis, and ECU (Electronic Control Unit) communication testing for vehicle security research and penetration testing.

**Note:** This is a test framework. Use only on authorized systems and test environments.

## Installation

### Prerequisites

```bash
# Python 3.7+ required
python3 --version

# Install system dependencies (Linux)
sudo apt-get update
sudo apt-get install -y python3-pip can-utils

# For hardware CAN interface support
sudo apt-get install -y python3-can
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Or install manually if requirements.txt is unavailable
pip3 install python-can cantools pyyaml colorama
```

### Hardware Setup

For physical CAN bus testing, you'll need:
- CAN interface adapter (e.g., USB-CAN, PCAN-USB)
- SocketCAN support on Linux

```bash
# Set up SocketCAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ifconfig can0
```

## Core Features

### 1. CAN Bus Sniffing

Monitor CAN bus traffic and capture messages:

```python
#can_sniffer.py
import can
import sys

def sniff_can_bus(interface='can0', duration=10):
    """Capture CAN bus messages"""
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        print(f"[*] Sniffing on {interface} for {duration} seconds...")
        
        messages = []
        for msg in bus:
            print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
            messages.append(msg)
            
            if len(messages) >= duration * 10:  # Approximate
                break
                
        return messages
    except Exception as e:
        print(f"[-] Error: {e}")
        return []

# Usage
if __name__ == "__main__":
    sniff_can_bus(interface='can0', duration=30)
```

### 2. Message Injection

Send crafted CAN messages for testing:

```python
# can_injector.py
import can
import time

def send_can_message(interface='can0', arb_id=0x123, data=[0x00, 0x01, 0x02]):
    """Inject CAN message onto the bus"""
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        
        message = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        bus.send(message)
        print(f"[+] Sent: ID=0x{arb_id:03X} Data={bytes(data).hex()}")
        
    except Exception as e:
        print(f"[-] Error sending message: {e}")

def fuzzing_attack(interface='can0', target_id=0x100, iterations=100):
    """Fuzz a specific CAN ID with random data"""
    import random
    
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    for i in range(iterations):
        data = [random.randint(0, 255) for _ in range(8)]
        msg = can.Message(arbitration_id=target_id, data=data)
        bus.send(msg)
        
        if i % 10 == 0:
            print(f"[*] Fuzzing progress: {i}/{iterations}")
        time.sleep(0.01)
    
    print("[+] Fuzzing complete")

# Usage
send_can_message(interface='can0', arb_id=0x7E0, data=[0x02, 0x01, 0x00])
```

### 3. UDS Protocol Testing

Test Unified Diagnostic Services (UDS) functionality:

```python
# uds_scanner.py
import can
import time

class UDSScanner:
    """Scan for UDS-capable ECUs"""
    
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.timeout = 0.5
        
    def send_diagnostic_session(self, ecu_id):
        """Start diagnostic session (0x10)"""
        # Standard UDS request: 0x10 0x03 (Extended Diagnostic Session)
        msg = can.Message(
            arbitration_id=ecu_id,
            data=[0x02, 0x10, 0x03, 0x00, 0x00, 0x00, 0x00, 0x00],
            is_extended_id=False
        )
        self.bus.send(msg)
        
    def scan_ecu_range(self, start=0x7E0, end=0x7E8):
        """Scan for responsive ECUs in range"""
        active_ecus = []
        
        for ecu_id in range(start, end + 1):
            self.send_diagnostic_session(ecu_id)
            time.sleep(0.1)
            
            # Check for response
            response = self.bus.recv(timeout=self.timeout)
            if response and response.arbitration_id == (ecu_id + 0x08):
                print(f"[+] Active ECU found: 0x{ecu_id:03X}")
                active_ecus.append(ecu_id)
            else:
                print(f"[-] No response from: 0x{ecu_id:03X}")
                
        return active_ecus
    
    def read_dtc(self, ecu_id):
        """Read Diagnostic Trouble Codes (Service 0x19)"""
        msg = can.Message(
            arbitration_id=ecu_id,
            data=[0x02, 0x19, 0x02, 0xFF, 0x00, 0x00, 0x00, 0x00]
        )
        self.bus.send(msg)
        
        response = self.bus.recv(timeout=1.0)
        if response:
            print(f"[+] DTC Response: {response.data.hex()}")
            return response.data
        return None

# Usage
scanner = UDSScanner(interface='can0')
active_ecus = scanner.scan_ecu_range(0x7E0, 0x7E7)
```

### 4. Replay Attack

Record and replay CAN traffic:

```python
# replay_attack.py
import can
import time
import pickle

def record_traffic(interface='can0', duration=60, output_file='capture.pkl'):
    """Record CAN traffic to file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    print(f"[*] Recording for {duration} seconds...")
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            timestamp = time.time()
            messages.append((timestamp, msg))
            print(f"Captured: 0x{msg.arbitration_id:03X}")
    
    with open(output_file, 'wb') as f:
        pickle.dump(messages, f)
    
    print(f"[+] Saved {len(messages)} messages to {output_file}")
    return messages

def replay_traffic(interface='can0', input_file='capture.pkl', speed=1.0):
    """Replay recorded CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    with open(input_file, 'rb') as f:
        messages = pickle.load(f)
    
    print(f"[*] Replaying {len(messages)} messages at {speed}x speed...")
    
    base_time = messages[0][0] if messages else 0
    start_time = time.time()
    
    for timestamp, msg in messages:
        # Calculate delay
        original_delay = timestamp - base_time
        target_time = start_time + (original_delay / speed)
        
        # Wait until target time
        while time.time() < target_time:
            time.sleep(0.001)
        
        bus.send(msg)
        print(f"Replayed: 0x{msg.arbitration_id:03X}")
    
    print("[+] Replay complete")

# Usage
# record_traffic(duration=30, output_file='door_unlock.pkl')
# replay_traffic(input_file='door_unlock.pkl', speed=1.0)
```

## Configuration

### Virtual CAN Setup (for testing without hardware)

```bash
# Create virtual CAN interface
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Test virtual interface
candump vcan0 &
cansend vcan0 123#DEADBEEF
```

### Configuration File

Create `config.yaml` for framework settings:

```yaml
# config.yaml
can_interface:
  default: can0
  bitrate: 500000
  virtual: vcan0

scan_ranges:
  diagnostic: [0x7E0, 0x7E7]
  extended: [0x18DA00F1, 0x18DAFFFF]

logging:
  level: INFO
  output_dir: ./logs
  format: "%(asctime)s - %(levelname)s - %(message)s"

fuzzing:
  iterations: 1000
  delay_ms: 10
  randomize_data: true

security:
  safe_mode: true
  whitelist_ids: [0x100, 0x200, 0x300]
```

Load configuration in Python:

```python
import yaml

def load_config(config_file='config.yaml'):
    """Load framework configuration"""
    with open(config_file, 'r') as f:
        config = yaml.safe_load(f)
    return config

config = load_config()
interface = config['can_interface']['default']
scan_range = config['scan_ranges']['diagnostic']
```

## Common Testing Patterns

### Full ECU Discovery and Analysis

```python
# full_scan.py
import can
import time

class VehicleSecurityScanner:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.discovered_ecus = {}
        
    def passive_discovery(self, duration=30):
        """Passively discover ECUs by monitoring traffic"""
        print(f"[*] Passive discovery for {duration} seconds...")
        start = time.time()
        seen_ids = set()
        
        while time.time() - start < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg and msg.arbitration_id not in seen_ids:
                seen_ids.add(msg.arbitration_id)
                print(f"[+] Found ID: 0x{msg.arbitration_id:03X}")
                
        self.discovered_ecus = {id: {} for id in seen_ids}
        return seen_ids
    
    def active_probe(self, target_ids):
        """Actively probe ECUs with UDS requests"""
        for ecu_id in target_ids:
            print(f"[*] Probing 0x{ecu_id:03X}...")
            
            # Try reading VIN (Service 0x22, DID 0xF190)
            msg = can.Message(
                arbitration_id=ecu_id,
                data=[0x03, 0x22, 0xF1, 0x90, 0x00, 0x00, 0x00, 0x00]
            )
            self.bus.send(msg)
            
            response = self.bus.recv(timeout=0.5)
            if response:
                self.discovered_ecus[ecu_id]['vin_capable'] = True
                print(f"  [+] Responds to VIN request")
    
    def generate_report(self):
        """Generate security assessment report"""
        print("\n" + "="*50)
        print("VEHICLE SECURITY ASSESSMENT REPORT")
        print("="*50)
        print(f"Total ECUs discovered: {len(self.discovered_ecus)}")
        
        for ecu_id, info in self.discovered_ecus.items():
            print(f"\nECU 0x{ecu_id:03X}:")
            for key, value in info.items():
                print(f"  {key}: {value}")

# Usage
scanner = VehicleSecurityScanner(interface='can0')
ids = scanner.passive_discovery(duration=30)
scanner.active_probe(ids)
scanner.generate_report()
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check if interface exists
ip link show can0

# Verify SocketCAN module
lsmod | grep can

# Load modules if missing
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to netdev group
sudo usermod -a -G netdev $USER

# Or run with sudo (not recommended for production)
sudo python3 scanner.py
```

### No Messages Received

```python
# Verify bus activity with candump
# Terminal 1: Listen
candump can0

# Terminal 2: Send test message
cansend can0 123#DEADBEEF

# Check bitrate matches your vehicle (common: 250k, 500k)
sudo ip link set can0 type can bitrate 250000
sudo ip link set up can0
```

### Timeout Issues

```python
# Increase timeout for slower ECUs
bus = can.interface.Bus(channel='can0', bustype='socketcan')
msg = bus.recv(timeout=5.0)  # 5 second timeout

# Or use non-blocking mode
msg = bus.recv(timeout=0)  # Returns None immediately if no message
```

## Safety and Legal Considerations

**WARNING:** Only test on authorized vehicles or test benches. Unauthorized vehicle network testing may be illegal and dangerous.

```python
# Implement safety checks
SAFE_MODE = True  # Set via environment variable
SAFE_MODE = os.getenv('S800_SAFE_MODE', 'true').lower() == 'true'

if SAFE_MODE and arb_id in CRITICAL_IDS:
    print("[!] SAFE MODE: Refusing to send to critical ID")
    sys.exit(1)
```

## Environment Variables

```bash
# Set interface
export S800_CAN_INTERFACE=can0

# Enable safe mode
export S800_SAFE_MODE=true

# Set log level
export S800_LOG_LEVEL=DEBUG

# Configure output directory
export S800_OUTPUT_DIR=/var/log/s800
```

Use in code:

```python
import os

interface = os.getenv('S800_CAN_INTERFACE', 'can0')
safe_mode = os.getenv('S800_SAFE_MODE', 'true') == 'true'
log_level = os.getenv('S800_LOG_LEVEL', 'INFO')
```
