---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive CAN bus and vehicle network vulnerabilities
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - perform automotive security testing
  - scan vehicle network for exploits
  - test car network protocols
  - automotive penetration testing framework
  - vehicle ECU security assessment
  - CAN bus fuzzing and testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks, focusing on CAN (Controller Area Network) bus analysis, ECU (Electronic Control Unit) security assessment, and vulnerability discovery in automotive systems. The framework provides tools for protocol analysis, fuzzing, message injection, and security testing of in-vehicle networks.

## Installation

### Prerequisites

- Python 3.7+
- SocketCAN kernel modules (Linux)
- CAN interface hardware (USB-CAN adapter, OBD-II dongle, etc.)

### Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Install SocketCAN utilities (Linux)
sudo apt-get install can-utils

# Set up virtual CAN interface for testing
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

```bash
# Configure physical CAN interface (example with slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0

# Set CAN bitrate (500 kbit/s is common)
sudo ip link set can0 type can bitrate 500000
```

## Core Components

### 1. CAN Bus Sniffer

Monitor and capture CAN bus traffic:

```python
import can

# Initialize CAN bus connection
bus = can.interface.Bus(channel='can0', bustype='socketcan')

# Capture CAN messages
def sniff_can_traffic(duration=10):
    """Capture CAN messages for specified duration"""
    messages = []
    
    for msg in bus:
        timestamp = msg.timestamp
        arbitration_id = hex(msg.arbitration_id)
        data = msg.data.hex()
        
        print(f"[{timestamp}] ID: {arbitration_id} Data: {data}")
        messages.append(msg)
        
        if len(messages) >= duration * 100:  # Approximate duration
            break
    
    return messages

# Save captured traffic
captured = sniff_can_traffic(duration=30)
```

### 2. CAN Message Injection

Send crafted CAN messages to the bus:

```python
import can

def send_can_message(arbitration_id, data, channel='can0'):
    """Send a CAN message to the bus"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    # Create CAN message
    message = can.Message(
        arbitration_id=arbitration_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(message)
        print(f"Sent: ID={hex(arbitration_id)} Data={data.hex()}")
        return True
    except can.CanError as e:
        print(f"Error sending message: {e}")
        return False
    finally:
        bus.shutdown()

# Example: Send diagnostic message
send_can_message(0x7DF, bytes([0x02, 0x01, 0x0D, 0x00, 0x00, 0x00, 0x00, 0x00]))
```

### 3. CAN Bus Fuzzing

Automated fuzzing for vulnerability discovery:

```python
import can
import random
import time

def fuzz_can_bus(target_id_range, duration=60, channel='can0'):
    """Fuzz CAN bus with random messages"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    start_time = time.time()
    fuzz_count = 0
    
    while time.time() - start_time < duration:
        # Random arbitration ID within range
        arb_id = random.randint(target_id_range[0], target_id_range[1])
        
        # Random data payload (8 bytes max for standard CAN)
        data_length = random.randint(1, 8)
        data = bytes([random.randint(0, 255) for _ in range(data_length)])
        
        message = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(message)
            fuzz_count += 1
            time.sleep(0.01)  # Rate limiting
        except can.CanError:
            pass
    
    bus.shutdown()
    print(f"Fuzzing complete. Sent {fuzz_count} messages.")
    return fuzz_count

# Fuzz specific ID range
fuzz_can_bus(target_id_range=(0x100, 0x200), duration=120)
```

### 4. Protocol Analysis

Analyze CAN message patterns and frequencies:

```python
from collections import defaultdict
import can
import time

def analyze_can_traffic(duration=30, channel='can0'):
    """Analyze CAN traffic patterns"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    message_stats = defaultdict(lambda: {'count': 0, 'data_samples': []})
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            arb_id = msg.arbitration_id
            message_stats[arb_id]['count'] += 1
            if len(message_stats[arb_id]['data_samples']) < 10:
                message_stats[arb_id]['data_samples'].append(msg.data.hex())
    
    bus.shutdown()
    
    # Generate report
    print("\n=== CAN Traffic Analysis ===")
    for arb_id, stats in sorted(message_stats.items()):
        print(f"\nID: {hex(arb_id)}")
        print(f"  Count: {stats['count']}")
        print(f"  Frequency: {stats['count']/duration:.2f} msgs/sec")
        print(f"  Sample data: {stats['data_samples'][:3]}")
    
    return message_stats

# Run analysis
stats = analyze_can_traffic(duration=60)
```

### 5. UDS Diagnostic Scanner

Universal Diagnostic Services (UDS) protocol testing:

```python
import can
import time

class UDSScanner:
    def __init__(self, channel='can0'):
        self.bus = can.interface.Bus(channel=channel, bustype='socketcan')
        self.diagnostic_id = 0x7DF  # Broadcast diagnostic request
        self.response_ids = range(0x7E8, 0x7F0)  # ECU response range
    
    def send_uds_request(self, service_id, data_params=[]):
        """Send UDS diagnostic request"""
        data = [len(data_params) + 1, service_id] + data_params
        data.extend([0x00] * (8 - len(data)))  # Pad to 8 bytes
        
        message = can.Message(
            arbitration_id=self.diagnostic_id,
            data=bytes(data),
            is_extended_id=False
        )
        
        self.bus.send(message)
    
    def read_response(self, timeout=1.0):
        """Read UDS response"""
        responses = []
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            msg = self.bus.recv(timeout=0.1)
            if msg and msg.arbitration_id in self.response_ids:
                responses.append(msg)
        
        return responses
    
    def scan_ecus(self):
        """Scan for responding ECUs"""
        print("Scanning for ECUs...")
        active_ecus = []
        
        # Send diagnostic session control
        self.send_uds_request(0x10, [0x01])
        responses = self.read_response(timeout=2.0)
        
        for resp in responses:
            ecu_id = resp.arbitration_id
            active_ecus.append(ecu_id)
            print(f"Found ECU: {hex(ecu_id)} - Data: {resp.data.hex()}")
        
        return active_ecus
    
    def read_dtc(self):
        """Read Diagnostic Trouble Codes"""
        self.send_uds_request(0x19, [0x02, 0x0F])  # Read DTC by status mask
        return self.read_response(timeout=2.0)
    
    def close(self):
        self.bus.shutdown()

# Usage
scanner = UDSScanner(channel='can0')
ecus = scanner.scan_ecus()
dtcs = scanner.read_dtc()
scanner.close()
```

## Configuration

### Config File Structure

Create `config.json` for test parameters:

```json
{
  "interface": {
    "channel": "can0",
    "bustype": "socketcan",
    "bitrate": 500000
  },
  "security_tests": {
    "fuzz_duration": 120,
    "target_id_ranges": [
      [256, 512],
      [1536, 2048]
    ],
    "rate_limit_ms": 10
  },
  "logging": {
    "enabled": true,
    "output_dir": "./logs",
    "capture_all": false
  },
  "uds": {
    "diagnostic_id": "0x7DF",
    "response_range": ["0x7E8", "0x7EF"],
    "timeout_ms": 1000
  }
}
```

### Load Configuration

```python
import json

def load_config(config_path='config.json'):
    """Load testing configuration"""
    with open(config_path, 'r') as f:
        config = json.load(f)
    return config

config = load_config()
channel = config['interface']['channel']
bitrate = config['interface']['bitrate']
```

## Common Testing Patterns

### 1. Replay Attack Testing

```python
import can
import time

def capture_and_replay(capture_duration=30, replay_count=5, channel='can0'):
    """Capture traffic and replay messages"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    # Capture phase
    print(f"Capturing for {capture_duration} seconds...")
    captured_messages = []
    start = time.time()
    
    while time.time() - start < capture_duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            captured_messages.append(msg)
    
    print(f"Captured {len(captured_messages)} messages")
    
    # Replay phase
    print(f"Replaying {replay_count} times...")
    for i in range(replay_count):
        for msg in captured_messages:
            replay_msg = can.Message(
                arbitration_id=msg.arbitration_id,
                data=msg.data,
                is_extended_id=msg.is_extended_id
            )
            bus.send(replay_msg)
            time.sleep(0.001)
        print(f"Replay {i+1} complete")
    
    bus.shutdown()

# Execute replay attack test
capture_and_replay(capture_duration=30, replay_count=3)
```

### 2. Message Frequency Analysis

```python
def detect_periodic_messages(duration=60, channel='can0'):
    """Detect periodic CAN messages"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    message_timestamps = defaultdict(list)
    start = time.time()
    
    while time.time() - start < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            message_timestamps[msg.arbitration_id].append(msg.timestamp)
    
    bus.shutdown()
    
    # Calculate periods
    periodic_messages = {}
    for arb_id, timestamps in message_timestamps.items():
        if len(timestamps) > 10:
            intervals = [timestamps[i+1] - timestamps[i] 
                        for i in range(len(timestamps)-1)]
            avg_period = sum(intervals) / len(intervals)
            periodic_messages[arb_id] = avg_period
            print(f"ID {hex(arb_id)}: ~{avg_period*1000:.2f}ms period")
    
    return periodic_messages

# Detect periodic messages
periodic = detect_periodic_messages(duration=30)
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Reload SocketCAN modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Check dmesg for errors
dmesg | grep can
```

### Permission Denied

```bash
# Add user to dialout group for serial access
sudo usermod -a -G dialout $USER

# Set CAN interface permissions
sudo chmod 666 /dev/ttyUSB0
```

### No Messages Received

```python
# Verify bus configuration
import can

bus = can.interface.Bus(channel='can0', bustype='socketcan')
print(f"Bus state: {bus.state}")

# Check bitrate match
# Ensure your bitrate matches the vehicle network (common: 125k, 250k, 500k)
```

### Environment Variables

```bash
# Set default CAN interface
export CAN_INTERFACE=can0
export CAN_BITRATE=500000
export S800_LOG_LEVEL=DEBUG
export S800_OUTPUT_DIR=/var/log/s800
```

## Safety Warnings

⚠️ **CRITICAL**: This framework is for authorized security testing only. Unauthorized vehicle network manipulation can:
- Compromise vehicle safety systems
- Cause physical damage or injury
- Violate laws and regulations

Always:
- Obtain proper authorization before testing
- Test on isolated/bench systems when possible
- Never test on public roads or operational vehicles
- Follow responsible disclosure practices
