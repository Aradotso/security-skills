---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, focusing on CAN bus and automotive protocols
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive security testing
  - scan vehicle network vulnerabilities
  - use S800 framework
  - test car network protocols
  - automotive penetration testing
  - vehicle security assessment
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for automotive security researchers and penetration testers. It provides tools for analyzing, testing, and evaluating the security of vehicle networks, particularly CAN (Controller Area Network) bus systems and other automotive communication protocols.

**Note**: This is marked as a test framework by the authors. Use only in controlled environments with proper authorization.

## Installation

### Prerequisites

```bash
# Required dependencies
sudo apt-get update
sudo apt-get install -y python3 python3-pip git
sudo apt-get install -y can-utils socketcan-dev
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies (if requirements.txt exists)
pip3 install -r requirements.txt

# Or install common automotive security libraries
pip3 install python-can cantools scapy
```

### Hardware Setup

For physical vehicle testing, you'll need:
- CAN interface adapter (e.g., CANable, PCAN-USB)
- OBD-II to DB9 cable (if testing via OBD-II port)

```bash
# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Set up virtual CAN for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Set up physical CAN interface (example with can0)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Concepts

### CAN Bus Fundamentals

CAN (Controller Area Network) is a message-based protocol used in vehicles for communication between ECUs (Electronic Control Units).

- **CAN ID**: Identifier for message types (11-bit standard or 29-bit extended)
- **Data Length**: 0-8 bytes per frame
- **Bit Rate**: Typically 125kbps, 250kbps, or 500kbps in automotive

### Security Testing Objectives

1. **Traffic Analysis**: Capture and analyze normal vehicle network traffic
2. **Fuzzing**: Send malformed or unexpected messages to test ECU resilience
3. **Replay Attacks**: Record and replay messages to test security controls
4. **Injection**: Inject crafted messages to trigger specific behaviors
5. **Protocol Analysis**: Reverse engineer proprietary protocols

## Basic Usage Patterns

### CAN Traffic Capture

```python
import can
import time

# Initialize CAN interface
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

def capture_traffic(duration=10):
    """Capture CAN traffic for specified duration"""
    print(f"Capturing traffic for {duration} seconds...")
    messages = []
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append(msg)
            print(f"ID: {hex(msg.arbitration_id)} Data: {msg.data.hex()}")
    
    return messages

# Capture traffic
captured = capture_traffic(duration=30)
print(f"Captured {len(captured)} messages")
```

### CAN Message Injection

```python
import can

def send_can_message(can_id, data):
    """Send a CAN message"""
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    
    # Create message
    msg = can.Message(
        arbitration_id=can_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"Sent: ID={hex(can_id)} Data={data.hex()}")
    except can.CanError:
        print("Message failed to send")
    finally:
        bus.shutdown()

# Example: Send diagnostic request
send_can_message(0x7DF, bytes([0x02, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]))
```

### CAN Fuzzing

```python
import can
import random
import time

def fuzz_can_messages(can_id_range, count=100, delay=0.01):
    """Fuzz CAN messages within specified ID range"""
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    
    for i in range(count):
        # Random CAN ID within range
        can_id = random.randint(can_id_range[0], can_id_range[1])
        
        # Random data length (0-8 bytes)
        data_len = random.randint(0, 8)
        data = bytes([random.randint(0, 255) for _ in range(data_len)])
        
        msg = can.Message(
            arbitration_id=can_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"Fuzz {i+1}: ID={hex(can_id)} Data={data.hex()}")
        except can.CanError as e:
            print(f"Error: {e}")
        
        time.sleep(delay)
    
    bus.shutdown()

# Fuzz diagnostic ID range
fuzz_can_messages((0x7E0, 0x7E8), count=50)
```

### Replay Attack

```python
import can
import time
import pickle

def record_traffic(filename, duration=30):
    """Record CAN traffic to file"""
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    messages = []
    start_time = time.time()
    
    print(f"Recording for {duration} seconds...")
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            timestamp = time.time() - start_time
            messages.append((timestamp, msg))
    
    with open(filename, 'wb') as f:
        pickle.dump(messages, f)
    
    print(f"Recorded {len(messages)} messages to {filename}")
    bus.shutdown()

def replay_traffic(filename, speed=1.0):
    """Replay recorded CAN traffic"""
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    
    with open(filename, 'rb') as f:
        messages = pickle.load(f)
    
    print(f"Replaying {len(messages)} messages at {speed}x speed...")
    start_time = time.time()
    
    for timestamp, msg in messages:
        # Wait for correct timing
        target_time = timestamp / speed
        while time.time() - start_time < target_time:
            time.sleep(0.001)
        
        bus.send(msg)
        print(f"Replayed: ID={hex(msg.arbitration_id)} Data={msg.data.hex()}")
    
    bus.shutdown()

# Record and replay
record_traffic('captured_session.pkl', duration=20)
replay_traffic('captured_session.pkl', speed=1.0)
```

## UDS (Unified Diagnostic Services) Testing

```python
import can
import time

class UDSClient:
    """Basic UDS diagnostic client"""
    
    def __init__(self, channel='vcan0', tx_id=0x7DF, rx_id=0x7E8):
        self.bus = can.interface.Bus(channel=channel, bustype='socketcan')
        self.tx_id = tx_id
        self.rx_id = rx_id
    
    def send_request(self, service, data=None):
        """Send UDS request"""
        payload = [service]
        if data:
            payload.extend(data)
        
        # Pad to 8 bytes
        payload.extend([0x00] * (8 - len(payload)))
        
        msg = can.Message(
            arbitration_id=self.tx_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
        return self.wait_response()
    
    def wait_response(self, timeout=1.0):
        """Wait for UDS response"""
        start = time.time()
        while time.time() - start < timeout:
            msg = self.bus.recv(timeout=0.1)
            if msg and msg.arbitration_id == self.rx_id:
                return msg.data
        return None
    
    def read_dtc(self):
        """Read Diagnostic Trouble Codes"""
        # Service 0x19 - Read DTC Information
        response = self.send_request(0x19, [0x02, 0x0A])
        if response:
            print(f"DTC Response: {response.hex()}")
            return response
        return None
    
    def read_data_by_id(self, data_id):
        """Read data by identifier"""
        # Service 0x22 - Read Data By Identifier
        response = self.send_request(0x22, [data_id >> 8, data_id & 0xFF])
        if response:
            print(f"Data ID {hex(data_id)}: {response.hex()}")
            return response
        return None
    
    def close(self):
        self.bus.shutdown()

# Example usage
uds = UDSClient(channel='vcan0')
uds.read_dtc()
uds.read_data_by_id(0xF190)  # VIN
uds.close()
```

## Configuration

### Environment Variables

```bash
# Set CAN interface
export CAN_INTERFACE=vcan0

# Set CAN bitrate
export CAN_BITRATE=500000

# Logging level
export S800_LOG_LEVEL=DEBUG

# Output directory for captures
export S800_OUTPUT_DIR=/tmp/s800_captures
```

### Python Configuration

```python
import os

# Configuration class
class S800Config:
    CAN_INTERFACE = os.getenv('CAN_INTERFACE', 'vcan0')
    CAN_BITRATE = int(os.getenv('CAN_BITRATE', '500000'))
    LOG_LEVEL = os.getenv('S800_LOG_LEVEL', 'INFO')
    OUTPUT_DIR = os.getenv('S800_OUTPUT_DIR', './captures')
    
    # Common CAN IDs
    OBD_BROADCAST = 0x7DF
    OBD_RESPONSE_BASE = 0x7E8
    
    # Timeouts
    DEFAULT_TIMEOUT = 1.0
    LONG_TIMEOUT = 5.0

config = S800Config()
```

## Common Testing Scenarios

### Identify Active CAN IDs

```python
import can
from collections import Counter

def identify_active_ids(duration=30):
    """Identify all active CAN IDs on the bus"""
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    id_counter = Counter()
    
    print(f"Scanning for {duration} seconds...")
    start = time.time()
    
    while time.time() - start < duration:
        msg = bus.recv(timeout=0.1)
        if msg:
            id_counter[msg.arbitration_id] += 1
    
    bus.shutdown()
    
    print("\nActive CAN IDs:")
    for can_id, count in id_counter.most_common():
        print(f"ID: {hex(can_id)} - Count: {count}")
    
    return id_counter

# Scan network
active_ids = identify_active_ids(duration=60)
```

### Differential Analysis

```python
def differential_analysis(baseline_file, test_file):
    """Compare two traffic captures to identify changes"""
    with open(baseline_file, 'rb') as f:
        baseline = pickle.load(f)
    
    with open(test_file, 'rb') as f:
        test = pickle.load(f)
    
    baseline_ids = {msg.arbitration_id for _, msg in baseline}
    test_ids = {msg.arbitration_id for _, msg in test}
    
    new_ids = test_ids - baseline_ids
    missing_ids = baseline_ids - test_ids
    
    print("New CAN IDs in test:")
    for can_id in new_ids:
        print(f"  {hex(can_id)}")
    
    print("\nMissing CAN IDs in test:")
    for can_id in missing_ids:
        print(f"  {hex(can_id)}")

# Usage
differential_analysis('baseline.pkl', 'with_action.pkl')
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify CAN module loaded
lsmod | grep can

# Load CAN modules if missing
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to dialout group for USB CAN adapters
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for development)
sudo python3 your_script.py
```

### No Messages Received

```python
# Verify bus is up and configured
import can

try:
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    print("Bus initialized successfully")
    
    # Send test message
    msg = can.Message(arbitration_id=0x123, data=[1,2,3,4,5,6,7,8])
    bus.send(msg)
    
    # Try to receive
    received = bus.recv(timeout=2.0)
    if received:
        print(f"Received: {received}")
    else:
        print("No message received - check bitrate and connections")
except Exception as e:
    print(f"Error: {e}")
```

### Bitrate Mismatch

```bash
# Common automotive bitrates
# Low-speed CAN: 125 kbps
sudo ip link set can0 type can bitrate 125000

# Medium-speed CAN: 250 kbps
sudo ip link set can0 type can bitrate 250000

# High-speed CAN: 500 kbps (most common)
sudo ip link set can0 type can bitrate 500000

# Bring interface up
sudo ip link set up can0
```

## Safety and Legal Warnings

**CRITICAL**: Only use this framework on:
- Vehicle networks you own
- Test benches and simulation environments
- Networks where you have explicit written authorization

Unauthorized testing of vehicle networks may:
- Violate laws (Computer Fraud and Abuse Act, etc.)
- Cause safety issues or vehicle damage
- Void warranties
- Result in legal prosecution

Always test in isolated environments first (virtual CAN, benchtop ECUs) before attempting live vehicle testing.
