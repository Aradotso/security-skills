---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for CAN bus and automotive protocol analysis
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive penetration testing
  - use S800 framework
  - scan vehicle network vulnerabilities
  - test automotive protocols
  - fuzzing CAN messages
  - vehicle ECU testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and security analysis of automotive networks, particularly CAN (Controller Area Network) bus systems. It provides tools for capturing, analyzing, fuzzing, and replaying CAN messages to identify vulnerabilities in vehicle Electronic Control Units (ECUs).

**Note**: This is a test/development framework. Use only on authorized test vehicles or lab environments. Never test on production vehicles without proper authorization.

## Installation

### Prerequisites

- Python 3.7+
- SocketCAN interface (Linux)
- CAN hardware adapter (e.g., PCAN, Kvaser, CANtact)

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Or install common CAN libraries manually
pip install python-can cantools bitstring
```

### Hardware Setup

```bash
# Load SocketCAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Setup virtual CAN for testing (no hardware)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Setup physical CAN interface (example with slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0
```

## Core Components

### CAN Message Capture

```python
import can

# Initialize CAN bus connection
bus = can.interface.Bus(channel='can0', bustype='socketcan')

# Capture CAN messages
def capture_traffic(duration=10):
    """Capture CAN traffic for analysis"""
    messages = []
    
    try:
        for msg in bus:
            messages.append({
                'timestamp': msg.timestamp,
                'arbitration_id': hex(msg.arbitration_id),
                'data': msg.data.hex(),
                'dlc': msg.dlc
            })
            
            if len(messages) >= duration * 100:  # Approximate messages
                break
                
    except KeyboardInterrupt:
        pass
    
    return messages

# Save captured traffic
import json

traffic = capture_traffic(duration=30)
with open('can_capture.json', 'w') as f:
    json.dump(traffic, f, indent=2)
```

### Message Analysis

```python
from collections import Counter

def analyze_can_ids(messages):
    """Analyze CAN ID frequency and patterns"""
    ids = [msg['arbitration_id'] for msg in messages]
    id_counts = Counter(ids)
    
    print("Most common CAN IDs:")
    for can_id, count in id_counts.most_common(10):
        print(f"  {can_id}: {count} messages")
    
    return id_counts

def find_periodic_messages(messages, threshold=0.95):
    """Identify periodically transmitted messages"""
    from collections import defaultdict
    
    timestamps = defaultdict(list)
    for msg in messages:
        timestamps[msg['arbitration_id']].append(msg['timestamp'])
    
    periodic = {}
    for can_id, times in timestamps.items():
        if len(times) < 3:
            continue
            
        intervals = [times[i+1] - times[i] for i in range(len(times)-1)]
        avg_interval = sum(intervals) / len(intervals)
        std_dev = (sum((x - avg_interval)**2 for x in intervals) / len(intervals))**0.5
        
        if std_dev / avg_interval < (1 - threshold):
            periodic[can_id] = {
                'interval': avg_interval,
                'count': len(times)
            }
    
    return periodic
```

### CAN Fuzzing

```python
import random
import time

class CANFuzzer:
    def __init__(self, bus, target_id=None):
        self.bus = bus
        self.target_id = target_id
    
    def fuzz_random_data(self, can_id, duration=10):
        """Send random data to a specific CAN ID"""
        start_time = time.time()
        count = 0
        
        while time.time() - start_time < duration:
            # Generate random 8-byte payload
            data = bytes([random.randint(0, 255) for _ in range(8)])
            
            msg = can.Message(
                arbitration_id=can_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                count += 1
                time.sleep(0.01)  # 10ms delay
            except can.CanError as e:
                print(f"Error sending: {e}")
        
        print(f"Sent {count} fuzzed messages to {hex(can_id)}")
    
    def fuzz_bit_flip(self, original_msg, iterations=100):
        """Flip individual bits in a known message"""
        for i in range(iterations):
            data = bytearray(original_msg.data)
            
            # Flip random bit
            byte_idx = random.randint(0, len(data)-1)
            bit_idx = random.randint(0, 7)
            data[byte_idx] ^= (1 << bit_idx)
            
            msg = can.Message(
                arbitration_id=original_msg.arbitration_id,
                data=bytes(data),
                is_extended_id=original_msg.is_extended_id
            )
            
            self.bus.send(msg)
            time.sleep(0.05)
    
    def fuzz_increment(self, can_id, start_value=0, count=256):
        """Increment data values systematically"""
        for i in range(count):
            value = (start_value + i) % 256
            data = bytes([value] * 8)
            
            msg = can.Message(
                arbitration_id=can_id,
                data=data,
                is_extended_id=False
            )
            
            self.bus.send(msg)
            time.sleep(0.1)
```

### Message Replay

```python
class CANReplay:
    def __init__(self, bus):
        self.bus = bus
    
    def replay_from_file(self, filename, speed=1.0):
        """Replay captured CAN traffic from file"""
        with open(filename, 'r') as f:
            messages = json.load(f)
        
        if not messages:
            print("No messages to replay")
            return
        
        start_time = messages[0]['timestamp']
        replay_start = time.time()
        
        for msg_data in messages:
            # Calculate when to send this message
            original_offset = msg_data['timestamp'] - start_time
            target_time = replay_start + (original_offset / speed)
            
            # Wait until the right time
            wait_time = target_time - time.time()
            if wait_time > 0:
                time.sleep(wait_time)
            
            # Send message
            msg = can.Message(
                arbitration_id=int(msg_data['arbitration_id'], 16),
                data=bytes.fromhex(msg_data['data']),
                is_extended_id=False
            )
            
            self.bus.send(msg)
        
        print(f"Replayed {len(messages)} messages")
    
    def replay_single_message(self, can_id, data, count=10, interval=0.1):
        """Repeatedly send a specific message"""
        msg = can.Message(
            arbitration_id=can_id,
            data=bytes.fromhex(data) if isinstance(data, str) else data,
            is_extended_id=False
        )
        
        for i in range(count):
            self.bus.send(msg)
            print(f"Sent message {i+1}/{count}: {hex(can_id)} - {msg.data.hex()}")
            time.sleep(interval)
```

## Common Testing Patterns

### Baseline Traffic Capture

```python
def establish_baseline(bus, duration=60):
    """Capture baseline traffic for comparison"""
    print(f"Capturing baseline for {duration} seconds...")
    
    baseline = []
    start = time.time()
    
    while time.time() - start < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            baseline.append({
                'id': msg.arbitration_id,
                'data': msg.data.hex(),
                'timestamp': msg.timestamp
            })
    
    # Analyze baseline
    unique_ids = set(m['id'] for m in baseline)
    print(f"Captured {len(baseline)} messages with {len(unique_ids)} unique IDs")
    
    return baseline
```

### Differential Analysis

```python
def compare_traffic(baseline, test_traffic):
    """Compare two traffic captures to find differences"""
    baseline_ids = set(m['arbitration_id'] for m in baseline)
    test_ids = set(m['arbitration_id'] for m in test_traffic)
    
    new_ids = test_ids - baseline_ids
    missing_ids = baseline_ids - test_ids
    
    print("New CAN IDs in test:", [hex(x) for x in new_ids])
    print("Missing CAN IDs:", [hex(x) for x in missing_ids])
    
    return {
        'new': new_ids,
        'missing': missing_ids
    }
```

### UDS Diagnostics Testing

```python
def send_uds_request(bus, ecu_id, service, data=b''):
    """Send UDS (Unified Diagnostic Services) request"""
    # UDS request format: Service ID + Data
    payload = bytes([service]) + data
    
    msg = can.Message(
        arbitration_id=ecu_id,
        data=payload,
        is_extended_id=False
    )
    
    bus.send(msg)
    
    # Wait for response (ECU_ID + 0x08 typically)
    response = bus.recv(timeout=1.0)
    return response

# Example: Read DTC (Diagnostic Trouble Codes)
response = send_uds_request(bus, 0x7E0, 0x19, b'\x02\xFF\xFF')
```

## Configuration

### Environment Variables

```bash
# Set CAN interface
export CAN_INTERFACE=can0

# Set log level
export S800_LOG_LEVEL=DEBUG

# Set capture directory
export CAPTURE_DIR=/var/log/s800/captures
```

### Configuration File Example

```python
# config.py
CONFIG = {
    'interface': os.getenv('CAN_INTERFACE', 'vcan0'),
    'bitrate': 500000,
    'capture_dir': os.getenv('CAPTURE_DIR', './captures'),
    'fuzzing': {
        'delay_ms': 10,
        'max_iterations': 1000
    },
    'filters': {
        'whitelist': [0x100, 0x200, 0x300],  # Only these IDs
        'blacklist': [0x7FF]  # Exclude these IDs
    }
}
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify SocketCAN module loaded
lsmod | grep can

# Check permissions
sudo chmod 666 /dev/ttyUSB0
```

### No Messages Received

```python
# Test with candump
candump can0

# Check bus status
ip -details link show can0

# Verify bitrate matches vehicle (common: 500kbps, 250kbps)
sudo ip link set can0 type can bitrate 500000
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python your_script.py
```

## Safety Warnings

- **Never test on vehicles in motion**
- **Always use isolated test environments when possible**
- **Keep emergency stop procedures ready**
- **Document all testing activities**
- **Obtain proper authorization before testing**
