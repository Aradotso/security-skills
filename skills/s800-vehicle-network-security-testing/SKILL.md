---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, packet injection, and vulnerability assessment capabilities
triggers:
  - test vehicle network security
  - scan automotive CAN bus vulnerabilities
  - fuzz vehicle network protocols
  - inject CAN bus packets
  - analyze vehicle network traffic
  - assess automotive cybersecurity
  - test ECU security vulnerabilities
  - perform vehicle penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. The framework provides tools for security assessment, vulnerability discovery, packet injection, fuzzing, and traffic analysis on vehicle electronic control units (ECUs).

**Key capabilities:**
- CAN/LIN/FlexRay bus sniffing and monitoring
- Protocol fuzzing and mutation testing
- Packet injection and replay attacks
- ECU enumeration and fingerprinting
- Diagnostic service exploitation (UDS, KWP2000)
- Traffic analysis and anomaly detection
- Security vulnerability assessment

## Installation

### Prerequisites

```bash
# Install SocketCAN tools (Linux)
sudo apt-get update
sudo apt-get install can-utils

# Install Python dependencies
pip install python-can cantools pyserial

# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework
```

### Hardware Requirements

- CAN interface adapter (e.g., CANable, PCAN-USB, Kvaser)
- OBD-II connector or direct ECU access
- Supported interfaces: SocketCAN, PCAN, Kvaser, Vector

### Setup CAN Interface (Linux)

```bash
# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Configure CAN interface (example for slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ifconfig can0 up

# Set bitrate (500kbps standard for most vehicles)
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up
```

## Core Components

### 1. CAN Bus Sniffer

Monitor and capture CAN traffic:

```python
import can

# Initialize CAN interface
bus = can.interface.Bus(channel='can0', bustype='socketcan')

# Sniff CAN messages
def sniff_can_traffic(duration=60):
    """Capture CAN messages for specified duration"""
    messages = []
    
    try:
        while len(messages) < duration * 100:  # Approximate message rate
            msg = bus.recv(timeout=1.0)
            if msg:
                messages.append({
                    'timestamp': msg.timestamp,
                    'arbitration_id': hex(msg.arbitration_id),
                    'data': msg.data.hex(),
                    'dlc': msg.dlc
                })
                print(f"ID: {hex(msg.arbitration_id)} Data: {msg.data.hex()}")
    except KeyboardInterrupt:
        print("Sniffing stopped")
    
    return messages

# Start sniffing
traffic = sniff_can_traffic(duration=30)
```

### 2. Packet Injection

Send crafted CAN messages:

```python
import can
import time

bus = can.interface.Bus(channel='can0', bustype='socketcan')

def send_can_message(arb_id, data):
    """Send a single CAN message"""
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    bus.send(msg)
    print(f"Sent: ID={hex(arb_id)} Data={data.hex()}")

def replay_attack(arb_id, data, count=10, interval=0.1):
    """Replay a captured message multiple times"""
    for i in range(count):
        send_can_message(arb_id, data)
        time.sleep(interval)

# Example: Inject diagnostic message
send_can_message(0x7DF, bytes([0x02, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]))

# Replay attack example
replay_attack(0x123, bytes([0xDE, 0xAD, 0xBE, 0xEF, 0x00, 0x00, 0x00, 0x00]))
```

### 3. CAN Fuzzer

Fuzz CAN messages to discover vulnerabilities:

```python
import can
import random
import time

bus = can.interface.Bus(channel='can0', bustype='socketcan')

def fuzz_can_id_range(start_id, end_id, iterations=1000):
    """Fuzz CAN IDs within a range with random data"""
    for _ in range(iterations):
        arb_id = random.randint(start_id, end_id)
        data_length = random.randint(0, 8)
        data = bytes([random.randint(0, 255) for _ in range(data_length)])
        
        msg = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"Fuzzed: ID={hex(arb_id)} DLC={data_length} Data={data.hex()}")
            time.sleep(0.01)  # Rate limiting
        except Exception as e:
            print(f"Error sending: {e}")

def smart_fuzz_message(base_id, base_data, mutations=100):
    """Mutate a known-good message"""
    for _ in range(mutations):
        mutated_data = bytearray(base_data)
        
        # Random mutation strategies
        strategy = random.choice(['bit_flip', 'byte_replace', 'boundary'])
        
        if strategy == 'bit_flip':
            byte_idx = random.randint(0, len(mutated_data) - 1)
            bit_idx = random.randint(0, 7)
            mutated_data[byte_idx] ^= (1 << bit_idx)
        
        elif strategy == 'byte_replace':
            byte_idx = random.randint(0, len(mutated_data) - 1)
            mutated_data[byte_idx] = random.randint(0, 255)
        
        elif strategy == 'boundary':
            byte_idx = random.randint(0, len(mutated_data) - 1)
            mutated_data[byte_idx] = random.choice([0x00, 0xFF, 0x7F, 0x80])
        
        send_can_message(base_id, bytes(mutated_data))
        time.sleep(0.05)

# Execute fuzzing
fuzz_can_id_range(0x100, 0x7FF, iterations=500)
```

### 4. UDS (Unified Diagnostic Services) Scanner

Scan for diagnostic services:

```python
import can
import time

bus = can.interface.Bus(channel='can0', bustype='socketcan')

UDS_SERVICES = {
    0x10: "Diagnostic Session Control",
    0x11: "ECU Reset",
    0x14: "Clear Diagnostic Information",
    0x19: "Read DTC Information",
    0x22: "Read Data By Identifier",
    0x23: "Read Memory By Address",
    0x27: "Security Access",
    0x28: "Communication Control",
    0x2E: "Write Data By Identifier",
    0x31: "Routine Control",
    0x34: "Request Download",
    0x35: "Request Upload",
    0x36: "Transfer Data",
    0x37: "Request Transfer Exit",
    0x3D: "Write Memory By Address",
    0x3E: "Tester Present"
}

def send_uds_request(ecu_id, service_id, data=None):
    """Send UDS request and wait for response"""
    request_data = [service_id]
    if data:
        request_data.extend(data)
    
    # Pad to 8 bytes
    while len(request_data) < 8:
        request_data.append(0x00)
    
    msg = can.Message(
        arbitration_id=ecu_id,
        data=bytes(request_data),
        is_extended_id=False
    )
    
    bus.send(msg)
    
    # Wait for response (ECU typically responds on ID + 0x08)
    response_id = ecu_id + 0x08
    timeout = time.time() + 2.0
    
    while time.time() < timeout:
        response = bus.recv(timeout=0.5)
        if response and response.arbitration_id == response_id:
            return response.data
    
    return None

def scan_uds_services(ecu_id=0x7DF):
    """Scan for supported UDS services"""
    print(f"Scanning ECU ID: {hex(ecu_id)}")
    supported_services = []
    
    for service_id, service_name in UDS_SERVICES.items():
        response = send_uds_request(ecu_id, service_id)
        
        if response:
            # Check for positive response (service_id + 0x40)
            if response[0] == service_id + 0x40:
                print(f"✓ {hex(service_id)}: {service_name}")
                supported_services.append((service_id, service_name))
            # Check for negative response
            elif response[0] == 0x7F:
                nrc = response[2]
                print(f"✗ {hex(service_id)}: {service_name} (NRC: {hex(nrc)})")
        
        time.sleep(0.1)
    
    return supported_services

# Scan for services
scan_uds_services(ecu_id=0x7E0)
```

### 5. ECU Enumeration

Discover active ECUs on the bus:

```python
import can
import time
from collections import defaultdict

bus = can.interface.Bus(channel='can0', bustype='socketcan')

def enumerate_ecus(duration=30):
    """Enumerate active ECUs by monitoring traffic"""
    ecu_map = defaultdict(lambda: {'count': 0, 'data_samples': []})
    
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            arb_id = msg.arbitration_id
            ecu_map[arb_id]['count'] += 1
            
            if len(ecu_map[arb_id]['data_samples']) < 5:
                ecu_map[arb_id]['data_samples'].append(msg.data.hex())
    
    print("\n=== ECU Enumeration Results ===")
    for arb_id in sorted(ecu_map.keys()):
        info = ecu_map[arb_id]
        print(f"\nID: {hex(arb_id)}")
        print(f"  Messages: {info['count']}")
        print(f"  Samples: {info['data_samples'][:3]}")
    
    return ecu_map

def probe_ecu_diagnostic(start_id=0x7E0, end_id=0x7E7):
    """Probe potential ECU diagnostic IDs"""
    active_ecus = []
    
    for ecu_id in range(start_id, end_id + 1):
        # Send tester present
        response = send_uds_request(ecu_id, 0x3E, [0x00])
        
        if response:
            print(f"Active ECU found: {hex(ecu_id)}")
            active_ecus.append(ecu_id)
        
        time.sleep(0.1)
    
    return active_ecus

# Enumerate ECUs
enumerate_ecus(duration=20)
```

## Configuration

### Environment Variables

```bash
# CAN interface configuration
export CAN_INTERFACE="can0"
export CAN_BITRATE="500000"

# Logging
export S800_LOG_LEVEL="INFO"
export S800_LOG_FILE="/var/log/s800/security.log"

# Target configuration
export TARGET_ECU_ID="0x7E0"
```

### Configuration File Example

```json
{
  "interface": {
    "type": "socketcan",
    "channel": "can0",
    "bitrate": 500000
  },
  "fuzzing": {
    "id_range": [256, 2047],
    "iterations": 10000,
    "rate_limit_ms": 10
  },
  "uds": {
    "timeout_ms": 2000,
    "retry_count": 3
  },
  "logging": {
    "level": "DEBUG",
    "output": "security_test.log"
  }
}
```

## Common Testing Patterns

### Security Assessment Workflow

```python
import can
import time
import json

class VehicleSecurityTester:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.results = {
            'ecus': [],
            'vulnerabilities': [],
            'traffic_baseline': {}
        }
    
    def full_assessment(self):
        """Complete security assessment"""
        print("Starting security assessment...")
        
        # Step 1: Traffic analysis
        print("\n[1/4] Analyzing baseline traffic...")
        self.analyze_baseline_traffic(duration=30)
        
        # Step 2: ECU enumeration
        print("\n[2/4] Enumerating ECUs...")
        self.enumerate_all_ecus()
        
        # Step 3: Service discovery
        print("\n[3/4] Scanning diagnostic services...")
        self.scan_all_services()
        
        # Step 4: Vulnerability testing
        print("\n[4/4] Testing for vulnerabilities...")
        self.test_vulnerabilities()
        
        return self.results
    
    def analyze_baseline_traffic(self, duration=30):
        """Establish traffic baseline"""
        start = time.time()
        traffic = defaultdict(list)
        
        while time.time() - start < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                traffic[msg.arbitration_id].append({
                    'timestamp': msg.timestamp,
                    'data': msg.data.hex()
                })
        
        self.results['traffic_baseline'] = dict(traffic)
        print(f"Captured {len(traffic)} unique message IDs")
    
    def enumerate_all_ecus(self):
        """Find all active ECUs"""
        # Passive enumeration from baseline
        ecus = list(self.results['traffic_baseline'].keys())
        
        # Active probing
        for ecu_id in range(0x7E0, 0x7E8):
            response = send_uds_request(ecu_id, 0x3E, [0x00])
            if response and ecu_id not in ecus:
                ecus.append(ecu_id)
        
        self.results['ecus'] = ecus
        print(f"Found {len(ecus)} ECUs")
    
    def scan_all_services(self):
        """Scan UDS services on all ECUs"""
        for ecu_id in self.results['ecus']:
            if 0x7E0 <= ecu_id <= 0x7E7:
                services = scan_uds_services(ecu_id)
                self.results[f'ecu_{hex(ecu_id)}_services'] = services
    
    def test_vulnerabilities(self):
        """Test for common vulnerabilities"""
        vulns = []
        
        # Test 1: Unauthenticated access
        for ecu_id in self.results['ecus']:
            if 0x7E0 <= ecu_id <= 0x7E7:
                # Try programming session without auth
                response = send_uds_request(ecu_id, 0x10, [0x02])
                if response and response[0] == 0x50:
                    vulns.append({
                        'ecu': hex(ecu_id),
                        'type': 'Unauthenticated Programming Session',
                        'severity': 'HIGH'
                    })
        
        # Test 2: Memory read without auth
        for ecu_id in self.results['ecus']:
            if 0x7E0 <= ecu_id <= 0x7E7:
                response = send_uds_request(ecu_id, 0x23, 
                                           [0x00, 0x00, 0x10, 0x00, 0x08])
                if response and response[0] == 0x63:
                    vulns.append({
                        'ecu': hex(ecu_id),
                        'type': 'Unauthenticated Memory Read',
                        'severity': 'CRITICAL'
                    })
        
        self.results['vulnerabilities'] = vulns
        print(f"Found {len(vulns)} vulnerabilities")
        
        return vulns
    
    def save_report(self, filename='security_report.json'):
        """Save assessment report"""
        with open(filename, 'w') as f:
            json.dump(self.results, f, indent=2)
        print(f"Report saved to {filename}")

# Run assessment
tester = VehicleSecurityTester(interface='can0')
results = tester.full_assessment()
tester.save_report()
```

## Troubleshooting

### CAN Interface Issues

```bash
# Check if interface is up
ip link show can0

# Verify CAN traffic
candump can0

# Check for errors
ip -details -statistics link show can0

# Reset interface
sudo ip link set can0 down
sudo ip link set can0 up type can bitrate 500000
```

### Permission Errors

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Set udev rules for CAN devices
echo 'KERNEL=="can*", MODE="0666"' | sudo tee /etc/udev/rules.d/99-can.rules
sudo udevadm control --reload-rules
```

### No Response from ECU

- Verify correct bitrate (125k, 250k, 500k, or 1Mbps)
- Check physical connections
- Ensure vehicle is in accessory or ignition-on mode
- Try different diagnostic IDs (0x7DF, 0x7E0-0x7E7)
- Some ECUs require extended diagnostic session first

### Bus Overload

```python
# Implement rate limiting
import time

def rate_limited_send(msg, min_interval=0.01):
    """Send with rate limiting"""
    bus.send(msg)
    time.sleep(min_interval)
```

## Safety Warning

**WARNING**: This framework is for authorized security testing only. Testing on production vehicles can:
- Cause unintended vehicle behavior
- Damage ECUs or components
- Create safety hazards
- Violate laws and regulations

Always test in controlled environments with proper authorization.
