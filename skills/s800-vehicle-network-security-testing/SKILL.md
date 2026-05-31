---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive CAN bus and vehicle network protocols
triggers:
  - test vehicle CAN bus security
  - scan automotive network vulnerabilities
  - perform vehicle network penetration testing
  - analyze CAN bus traffic
  - fuzz automotive protocols
  - test car network security
  - run S800 vehicle security tests
  - automotive security assessment
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800-Vehicle-Network-Security-Testing-Framework is a specialized security testing tool for automotive networks, focusing on Controller Area Network (CAN bus) and other in-vehicle communication protocols. It provides capabilities for vulnerability scanning, traffic analysis, fuzzing, and penetration testing of vehicle network systems.

## Installation

### Prerequisites

- Python 3.7+
- CAN interface hardware (e.g., CANtact, PCAN-USB, SocketCAN)
- Linux recommended (for SocketCAN support)

### Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Or install common dependencies manually
pip install python-can cantools scapy pyserial
```

### Hardware Configuration

For SocketCAN (Linux):
```bash
# Setup virtual CAN interface for testing
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# For physical CAN interfaces (example with can0)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Functionality

### CAN Bus Sniffing

Monitor and capture CAN bus traffic:

```python
import can
from datetime import datetime

# Initialize CAN bus interface
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

# Sniff CAN messages
def sniff_can_traffic(duration=10):
    """Capture CAN traffic for analysis"""
    messages = []
    start_time = datetime.now()
    
    print(f"[*] Sniffing CAN traffic on {bus.channel_info}")
    
    while (datetime.now() - start_time).seconds < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append(msg)
            print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    
    return messages

# Usage
captured = sniff_can_traffic(duration=30)
print(f"[+] Captured {len(captured)} CAN messages")
```

### CAN Message Injection

Send crafted CAN messages for testing:

```python
import can
import time

def send_can_message(bus, arb_id, data):
    """Send a CAN message"""
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"[+] Sent: ID=0x{arb_id:03X} Data={data.hex()}")
        return True
    except can.CanError as e:
        print(f"[-] Error sending message: {e}")
        return False

# Initialize bus
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

# Send test messages
send_can_message(bus, 0x123, bytes([0x01, 0x02, 0x03, 0x04]))
send_can_message(bus, 0x456, bytes([0xFF, 0xFF, 0xFF, 0xFF]))
```

### CAN Fuzzing

Fuzz CAN IDs and payloads to discover vulnerabilities:

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, channel='vcan0', bustype='socketcan'):
        self.bus = can.interface.Bus(channel=channel, bustype=bustype)
    
    def fuzz_random(self, num_messages=100, delay=0.01):
        """Send random CAN messages"""
        print(f"[*] Starting random fuzzing ({num_messages} messages)")
        
        for i in range(num_messages):
            arb_id = random.randint(0x000, 0x7FF)
            data_length = random.randint(0, 8)
            data = bytes([random.randint(0, 255) for _ in range(data_length)])
            
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            self.bus.send(msg)
            time.sleep(delay)
            
            if (i + 1) % 10 == 0:
                print(f"[*] Sent {i + 1}/{num_messages} fuzzed messages")
    
    def fuzz_specific_id(self, arb_id, num_messages=50):
        """Fuzz payloads for a specific CAN ID"""
        print(f"[*] Fuzzing ID 0x{arb_id:03X}")
        
        for i in range(num_messages):
            data_length = random.randint(1, 8)
            data = bytes([random.randint(0, 255) for _ in range(data_length)])
            
            msg = can.Message(arbitration_id=arb_id, data=data)
            self.bus.send(msg)
            time.sleep(0.05)

# Usage
fuzzer = CANFuzzer(channel='vcan0')
fuzzer.fuzz_random(num_messages=200)
fuzzer.fuzz_specific_id(arb_id=0x7DF, num_messages=100)
```

### Replay Attack

Capture and replay CAN traffic:

```python
import can
import time

class CANReplay:
    def __init__(self, channel='vcan0', bustype='socketcan'):
        self.bus = can.interface.Bus(channel=channel, bustype=bustype)
        self.captured_messages = []
    
    def capture(self, duration=10):
        """Capture CAN messages"""
        print(f"[*] Capturing for {duration} seconds...")
        start_time = time.time()
        
        while (time.time() - start_time) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.captured_messages.append({
                    'timestamp': time.time() - start_time,
                    'msg': msg
                })
        
        print(f"[+] Captured {len(self.captured_messages)} messages")
    
    def replay(self, speed_multiplier=1.0):
        """Replay captured messages"""
        if not self.captured_messages:
            print("[-] No messages to replay")
            return
        
        print(f"[*] Replaying {len(self.captured_messages)} messages")
        start_time = time.time()
        
        for item in self.captured_messages:
            # Wait for appropriate time
            target_time = item['timestamp'] / speed_multiplier
            while (time.time() - start_time) < target_time:
                time.sleep(0.001)
            
            # Send message
            self.bus.send(item['msg'])
        
        print("[+] Replay complete")

# Usage
replayer = CANReplay(channel='vcan0')
replayer.capture(duration=15)
replayer.replay(speed_multiplier=2.0)  # Replay 2x faster
```

### UDS (Unified Diagnostic Services) Scanner

Scan for diagnostic services:

```python
import can
import time

class UDSScanner:
    def __init__(self, channel='vcan0', bustype='socketcan'):
        self.bus = can.interface.Bus(channel=channel, bustype=bustype)
        self.uds_request_id = 0x7DF  # OBD-II functional address
        self.uds_response_ids = range(0x7E8, 0x7F0)  # Response range
    
    def send_uds_request(self, service_id, data=None):
        """Send UDS diagnostic request"""
        payload = [service_id]
        if data:
            payload.extend(data)
        
        msg = can.Message(
            arbitration_id=self.uds_request_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
    
    def scan_services(self, services=None):
        """Scan for supported UDS services"""
        if services is None:
            # Common UDS service IDs
            services = [
                0x10,  # DiagnosticSessionControl
                0x11,  # ECUReset
                0x22,  # ReadDataByIdentifier
                0x27,  # SecurityAccess
                0x2E,  # WriteDataByIdentifier
                0x3E,  # TesterPresent
                0x85,  # ControlDTCSetting
            ]
        
        print("[*] Scanning for UDS services...")
        found_services = []
        
        for service in services:
            self.send_uds_request(service, data=[0x00])
            time.sleep(0.1)
            
            # Check for response
            msg = self.bus.recv(timeout=0.5)
            if msg and msg.arbitration_id in self.uds_response_ids:
                if len(msg.data) > 0 and msg.data[0] != 0x7F:
                    found_services.append(service)
                    print(f"[+] Service 0x{service:02X} supported")
        
        return found_services

# Usage
scanner = UDSScanner(channel='vcan0')
supported = scanner.scan_services()
print(f"[+] Found {len(supported)} supported services")
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=vcan0
export S800_CAN_BUSTYPE=socketcan
export S800_CAN_BITRATE=500000

# Logging configuration
export S800_LOG_LEVEL=INFO
export S800_LOG_FILE=/var/log/s800/security_test.log
```

### Configuration File Example

```python
# config.py
import os

class S800Config:
    # CAN Interface Settings
    CAN_INTERFACE = os.getenv('S800_CAN_INTERFACE', 'vcan0')
    CAN_BUSTYPE = os.getenv('S800_CAN_BUSTYPE', 'socketcan')
    CAN_BITRATE = int(os.getenv('S800_CAN_BITRATE', '500000'))
    
    # Testing Parameters
    FUZZ_DELAY = 0.01  # seconds between fuzz messages
    CAPTURE_TIMEOUT = 1.0  # seconds
    
    # Security Testing
    UDS_TIMEOUT = 0.5
    MAX_RETRIES = 3
    
    # Logging
    LOG_LEVEL = os.getenv('S800_LOG_LEVEL', 'INFO')
    LOG_FILE = os.getenv('S800_LOG_FILE', 'security_test.log')
```

## Common Testing Patterns

### Comprehensive Vehicle Network Assessment

```python
import can
import time
from collections import defaultdict

class VehicleSecurityAssessment:
    def __init__(self, channel='vcan0'):
        self.bus = can.interface.Bus(channel=channel, bustype='socketcan')
        self.traffic_stats = defaultdict(int)
    
    def baseline_traffic_analysis(self, duration=60):
        """Establish baseline traffic patterns"""
        print(f"[*] Analyzing baseline traffic for {duration}s")
        start_time = time.time()
        
        while (time.time() - start_time) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.traffic_stats[msg.arbitration_id] += 1
        
        print("[+] Baseline analysis complete")
        print(f"[+] Identified {len(self.traffic_stats)} unique CAN IDs")
        
        # Show most active IDs
        sorted_ids = sorted(self.traffic_stats.items(), 
                          key=lambda x: x[1], reverse=True)
        print("\n[*] Top 10 most active CAN IDs:")
        for arb_id, count in sorted_ids[:10]:
            print(f"    0x{arb_id:03X}: {count} messages")
        
        return self.traffic_stats
    
    def detect_anomalies(self, monitoring_duration=30):
        """Monitor for unusual CAN traffic"""
        print(f"[*] Monitoring for anomalies ({monitoring_duration}s)")
        anomalies = []
        start_time = time.time()
        
        while (time.time() - start_time) < monitoring_duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                # Check for unknown IDs
                if msg.arbitration_id not in self.traffic_stats:
                    anomalies.append({
                        'type': 'unknown_id',
                        'id': msg.arbitration_id,
                        'data': msg.data.hex()
                    })
                    print(f"[!] Unknown ID: 0x{msg.arbitration_id:03X}")
        
        return anomalies

# Usage
assessment = VehicleSecurityAssessment(channel='vcan0')
baseline = assessment.baseline_traffic_analysis(duration=30)
anomalies = assessment.detect_anomalies(monitoring_duration=60)
print(f"[+] Detected {len(anomalies)} anomalies")
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify CAN kernel modules
lsmod | grep can

# Load required modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to dialout group (for USB CAN adapters)
sudo usermod -a -G dialout $USER

# Or run with sudo
sudo python3 your_script.py
```

### No Messages Received

```python
# Verify bus is up and bitrate matches
import can

try:
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    print(f"[+] Connected to {bus.channel_info}")
    
    # Test with timeout
    msg = bus.recv(timeout=5.0)
    if msg is None:
        print("[-] No messages received - check if traffic exists")
    else:
        print(f"[+] Received: {msg}")
except Exception as e:
    print(f"[-] Error: {e}")
```

### Buffer Overrun

```python
# Clear receive buffer periodically
def clear_buffer(bus, timeout=0.1):
    """Drain receive buffer"""
    while bus.recv(timeout=timeout) is not None:
        pass

# Use in long-running captures
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
clear_buffer(bus)
```

## Safety and Legal Considerations

**WARNING**: Vehicle network testing can affect safety-critical systems.

- Always test on isolated test benches or simulation environments first
- Never test on vehicles in operation or on public roads without authorization
- Obtain proper authorization before testing any production vehicle
- Follow responsible disclosure practices for discovered vulnerabilities
- Comply with local laws and regulations regarding vehicle modification

```python
# Example: Add safety checks
import os

def safety_check():
    """Verify test environment is safe"""
    if os.getenv('S800_PRODUCTION_MODE') == 'true':
        raise RuntimeError(
            "Production mode detected! "
            "Set S800_PRODUCTION_MODE=false for testing"
        )
    
    confirm = input("Confirm testing on isolated system (yes/no): ")
    if confirm.lower() != 'yes':
        raise RuntimeError("Safety check failed - user cancelled")

# Run before any testing
safety_check()
```
