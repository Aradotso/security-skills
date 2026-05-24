---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for automotive CAN/LIN bus penetration testing and vulnerability assessment
triggers:
  - test vehicle network security
  - scan automotive CAN bus
  - perform vehicle penetration testing
  - analyze car network vulnerabilities
  - test automotive ECU security
  - scan vehicle communication protocols
  - conduct CAN bus fuzzing
  - assess vehicle network threats
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for automotive penetration testing, focusing on CAN (Controller Area Network), LIN (Local Interconnect Network), and other automotive communication protocols. It provides tools for scanning, analyzing, fuzzing, and exploiting vulnerabilities in vehicle network systems and Electronic Control Units (ECUs).

**Note**: This project appears to be in early development or test phase. Use with caution and only on authorized test vehicles.

## Installation

### Prerequisites

- Python 3.7+
- SocketCAN support (Linux) or compatible CAN interface
- Hardware: CAN adapter (e.g., CANtact, PCAN-USB, Kvaser)

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Or install common automotive security libraries
pip install python-can cantools scapy
```

### Hardware Setup

```bash
# Linux: Enable SocketCAN interface
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Create virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Physical CAN interface (adjust bitrate as needed)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Concepts

### CAN Bus Basics

- **CAN ID**: Identifier for CAN messages (11-bit standard or 29-bit extended)
- **DLC**: Data Length Code (0-8 bytes)
- **Data**: Payload of the CAN frame
- **Arbitration**: Lower CAN ID has higher priority

### Common Attack Vectors

1. **Sniffing**: Passive monitoring of CAN traffic
2. **Fuzzing**: Sending randomized data to discover vulnerabilities
3. **Replay**: Capturing and replaying CAN messages
4. **DoS**: Flooding the bus with high-priority messages
5. **ECU Impersonation**: Sending crafted messages to control vehicle functions

## Usage Examples

### 1. CAN Bus Scanning and Sniffing

```python
import can
import time

# Initialize CAN interface
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

def scan_can_traffic(duration=10):
    """Scan and log CAN traffic for specified duration"""
    print(f"Scanning CAN traffic for {duration} seconds...")
    start_time = time.time()
    can_ids = set()
    
    while time.time() - start_time < duration:
        message = bus.recv(timeout=1.0)
        if message:
            can_ids.add(message.arbitration_id)
            print(f"ID: 0x{message.arbitration_id:03X} Data: {message.data.hex()}")
    
    print(f"\nUnique CAN IDs discovered: {len(can_ids)}")
    return sorted(can_ids)

# Run scan
discovered_ids = scan_can_traffic(duration=30)
```

### 2. CAN Message Replay Attack

```python
import can
import time

class CANReplayer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.recorded_messages = []
    
    def record(self, duration=10):
        """Record CAN messages"""
        print(f"Recording for {duration} seconds...")
        start_time = time.time()
        
        while time.time() - start_time < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.recorded_messages.append({
                    'id': msg.arbitration_id,
                    'data': msg.data,
                    'timestamp': msg.timestamp
                })
        
        print(f"Recorded {len(self.recorded_messages)} messages")
    
    def replay(self, delay=0):
        """Replay recorded messages"""
        print(f"Replaying {len(self.recorded_messages)} messages...")
        
        for msg_data in self.recorded_messages:
            msg = can.Message(
                arbitration_id=msg_data['id'],
                data=msg_data['data'],
                is_extended_id=False
            )
            self.bus.send(msg)
            time.sleep(delay)
        
        print("Replay complete")

# Usage
replayer = CANReplayer(interface='can0')
replayer.record(duration=10)
time.sleep(2)
replayer.replay(delay=0.01)
```

### 3. CAN Fuzzing Framework

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def fuzz_single_id(self, can_id, iterations=100):
        """Fuzz a specific CAN ID with random data"""
        print(f"Fuzzing CAN ID 0x{can_id:03X} for {iterations} iterations...")
        
        for i in range(iterations):
            # Generate random data length and payload
            data_length = random.randint(0, 8)
            data = bytes([random.randint(0, 255) for _ in range(data_length)])
            
            msg = can.Message(
                arbitration_id=can_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"Sent: ID=0x{can_id:03X} Data={data.hex()}")
                time.sleep(0.01)  # Throttle to avoid bus saturation
            except can.CanError as e:
                print(f"Error sending message: {e}")
    
    def fuzz_range(self, start_id=0x000, end_id=0x7FF, messages_per_id=10):
        """Fuzz a range of CAN IDs"""
        print(f"Fuzzing CAN ID range 0x{start_id:03X} to 0x{end_id:03X}")
        
        for can_id in range(start_id, end_id + 1):
            self.fuzz_single_id(can_id, iterations=messages_per_id)
            time.sleep(0.1)

# Usage
fuzzer = CANFuzzer(interface='can0')
fuzzer.fuzz_single_id(0x123, iterations=50)
```

### 4. ECU Diagnostic Services Scanner

```python
import can
import time

class UDSScanner:
    """Unified Diagnostic Services (UDS) scanner"""
    
    # Common UDS services
    UDS_SERVICES = {
        0x10: "Diagnostic Session Control",
        0x11: "ECU Reset",
        0x27: "Security Access",
        0x28: "Communication Control",
        0x3E: "Tester Present",
        0x22: "Read Data By Identifier",
        0x2E: "Write Data By Identifier",
        0x31: "Routine Control",
    }
    
    def __init__(self, interface='vcan0', request_id=0x7DF, response_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.request_id = request_id
        self.response_id = response_id
    
    def send_uds_request(self, service_id, data=None):
        """Send UDS request and wait for response"""
        payload = [service_id]
        if data:
            payload.extend(data)
        
        msg = can.Message(
            arbitration_id=self.request_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
        print(f"Sent UDS request: Service=0x{service_id:02X} Data={bytes(payload).hex()}")
        
        # Wait for response
        response = self.bus.recv(timeout=2.0)
        if response and response.arbitration_id == self.response_id:
            print(f"Response: {response.data.hex()}")
            return response.data
        
        return None
    
    def scan_services(self):
        """Scan for supported UDS services"""
        print("Scanning for supported UDS services...")
        supported = []
        
        for service_id, service_name in self.UDS_SERVICES.items():
            response = self.send_uds_request(service_id)
            
            if response and len(response) > 0:
                if response[0] == (service_id + 0x40):  # Positive response
                    supported.append((service_id, service_name))
                    print(f"✓ Supported: {service_name}")
            
            time.sleep(0.1)
        
        return supported

# Usage
scanner = UDSScanner(interface='can0', request_id=0x7DF, response_id=0x7E8)
supported_services = scanner.scan_services()
```

### 5. CAN Bus DoS Attack Detection

```python
import can
import time
from collections import defaultdict

class CANMonitor:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.message_counts = defaultdict(int)
        self.timestamps = defaultdict(list)
    
    def monitor_for_dos(self, duration=60, threshold=1000):
        """Monitor CAN bus for potential DoS attacks"""
        print(f"Monitoring for {duration} seconds (threshold: {threshold} msg/sec)...")
        start_time = time.time()
        
        while time.time() - start_time < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                can_id = msg.arbitration_id
                self.message_counts[can_id] += 1
                self.timestamps[can_id].append(msg.timestamp)
        
        # Analyze for suspicious activity
        print("\n--- Analysis Results ---")
        for can_id, count in sorted(self.message_counts.items(), 
                                     key=lambda x: x[1], reverse=True):
            rate = count / duration
            if rate > threshold:
                print(f"⚠ ALERT: CAN ID 0x{can_id:03X} - {count} messages ({rate:.1f} msg/sec)")
            else:
                print(f"CAN ID 0x{can_id:03X} - {count} messages ({rate:.1f} msg/sec)")

# Usage
monitor = CANMonitor(interface='can0')
monitor.monitor_for_dos(duration=30, threshold=100)
```

## Configuration

### Environment Variables

```bash
# CAN interface configuration
export CAN_INTERFACE="can0"
export CAN_BITRATE="500000"
export VCAN_INTERFACE="vcan0"

# Logging
export LOG_LEVEL="INFO"
export LOG_FILE="/var/log/s800/vehicle_security.log"

# Security settings
export SAFE_MODE="true"  # Prevents destructive commands
```

### Configuration File Example

```python
# config.py

# CAN Bus Configuration
CAN_CONFIG = {
    'interface': 'can0',
    'bustype': 'socketcan',
    'bitrate': 500000,
    'timeout': 1.0
}

# Target ECU addresses
ECU_ADDRESSES = {
    'ENGINE': 0x7E0,
    'TRANSMISSION': 0x7E1,
    'ABS': 0x7E2,
    'AIRBAG': 0x7E3,
    'GATEWAY': 0x7DF,
}

# Fuzzing parameters
FUZZING_CONFIG = {
    'iterations': 1000,
    'delay': 0.01,  # seconds between messages
    'safe_ids': [0x000, 0x001],  # IDs to never fuzz
}

# Security thresholds
SECURITY_THRESHOLDS = {
    'max_message_rate': 1000,  # messages per second
    'dos_detection_window': 10,  # seconds
}
```

## Common Patterns

### Safe Testing Wrapper

```python
import can
import os

class SafeCANTester:
    """Wrapper that enforces safety checks"""
    
    def __init__(self, interface='vcan0', safe_mode=True):
        self.safe_mode = safe_mode or os.getenv('SAFE_MODE', 'true').lower() == 'true'
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        
        # Critical CAN IDs that should never be modified
        self.protected_ids = {0x000, 0x001, 0x7FF}
    
    def send_message(self, can_id, data):
        """Send message with safety checks"""
        if self.safe_mode and can_id in self.protected_ids:
            print(f"⚠ BLOCKED: CAN ID 0x{can_id:03X} is protected")
            return False
        
        msg = can.Message(arbitration_id=can_id, data=data, is_extended_id=False)
        
        try:
            self.bus.send(msg)
            return True
        except can.CanError as e:
            print(f"Error: {e}")
            return False
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify SocketCAN modules
lsmod | grep can

# Reload modules if needed
sudo modprobe -r vcan
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python your_script.py
```

### Bus Off State

```python
# Reset CAN interface if it enters bus-off state
import subprocess

def reset_can_interface(interface='can0'):
    subprocess.run(['sudo', 'ip', 'link', 'set', 'down', interface])
    subprocess.run(['sudo', 'ip', 'link', 'set', 'up', interface])
```

## Safety Guidelines

1. **Always test on isolated networks or virtual CAN interfaces first**
2. **Never test on a vehicle in motion or on public roads**
3. **Use safe mode by default to prevent accidental damage**
4. **Keep detailed logs of all testing activities**
5. **Obtain proper authorization before testing any vehicle**
6. **Understand the legal implications of automotive security testing**

## Legal and Ethical Considerations

Vehicle network security testing must only be performed:
- On vehicles you own or have explicit written authorization to test
- In controlled environments (not on public roads)
- In compliance with local laws and regulations
- With proper safety measures in place

Unauthorized access to vehicle systems may violate laws including the Computer Fraud and Abuse Act (CFAA) and similar legislation in other jurisdictions.
