---
name: s800-vehicle-network-security-testing
description: Test and analyze vehicle network security using the S800 framework for CAN, LIN, and automotive protocol testing
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - use S800 framework for automotive testing
  - scan vehicle network protocols
  - perform automotive penetration testing
  - test car network security with S800
  - analyze automotive bus communication
  - vehicle ECU security testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and security analysis of automotive networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and other automotive protocols. The framework provides tools for monitoring, fuzzing, and exploiting vulnerabilities in vehicle electronic control units (ECUs) and network communications.

## Installation

### Prerequisites

```bash
# Install required dependencies
sudo apt-get update
sudo apt-get install -y python3 python3-pip can-utils
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies (if requirements.txt exists)
pip3 install -r requirements.txt

# Set up CAN interface (socketcan)
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Create virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For real vehicle testing, you'll need:
- CAN adapter (e.g., CANable, PCAN-USB, Kvaser)
- OBD-II connector or direct ECU access

```bash
# Configure real CAN interface (example with slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0

# Or with SocketCAN native interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Features

### 1. CAN Bus Monitoring

Monitor and capture CAN traffic:

```python
import can
import os

def monitor_can_bus(interface='vcan0', duration=10):
    """Monitor CAN bus traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Monitoring {interface} for {duration} seconds...")
    
    for msg in bus:
        print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
        
    bus.shutdown()

# Usage
monitor_can_bus('can0', duration=30)
```

### 2. CAN Frame Injection

Send custom CAN frames for testing:

```python
import can
import time

def inject_can_frame(interface, can_id, data):
    """Inject a CAN frame onto the bus"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Create CAN message
    msg = can.Message(
        arbitration_id=can_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"Sent: ID=0x{can_id:03X} Data={data.hex()}")
    except can.CanError as e:
        print(f"Failed to send: {e}")
    finally:
        bus.shutdown()

# Example: Send unlock command (hypothetical)
inject_can_frame('can0', 0x123, bytes([0x01, 0x02, 0x03, 0x04]))
```

### 3. CAN Fuzzing

Fuzz CAN IDs and payloads to discover vulnerabilities:

```python
import can
import random
import time

def fuzz_can_bus(interface, target_id=None, iterations=1000):
    """Fuzz CAN bus with random or targeted frames"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    for i in range(iterations):
        # Random or targeted CAN ID
        can_id = target_id if target_id else random.randint(0x000, 0x7FF)
        
        # Random data payload (0-8 bytes)
        data_len = random.randint(0, 8)
        data = bytes([random.randint(0, 255) for _ in range(data_len)])
        
        msg = can.Message(
            arbitration_id=can_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            if i % 100 == 0:
                print(f"Sent {i} frames...")
        except can.CanError as e:
            print(f"Error at iteration {i}: {e}")
            
        time.sleep(0.01)  # Small delay to avoid flooding
    
    bus.shutdown()
    print(f"Fuzzing complete: {iterations} frames sent")

# Fuzz specific ECU
fuzz_can_bus('can0', target_id=0x7DF, iterations=500)
```

### 4. UDS (Unified Diagnostic Services) Testing

Test diagnostic services:

```python
import can
import time

class UDSTester:
    """UDS protocol tester"""
    
    UDS_SERVICES = {
        0x10: 'DiagnosticSessionControl',
        0x11: 'ECUReset',
        0x27: 'SecurityAccess',
        0x28: 'CommunicationControl',
        0x3E: 'TesterPresent',
        0x22: 'ReadDataByIdentifier',
        0x2E: 'WriteDataByIdentifier',
        0x31: 'RoutineControl',
        0x34: 'RequestDownload',
        0x36: 'TransferData',
        0x37: 'RequestTransferExit'
    }
    
    def __init__(self, interface, tx_id=0x7DF, rx_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.tx_id = tx_id
        self.rx_id = rx_id
    
    def send_uds_request(self, service, data=None):
        """Send UDS request and wait for response"""
        payload = [service]
        if data:
            payload.extend(data)
        
        msg = can.Message(
            arbitration_id=self.tx_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
        print(f"Sent UDS: {self.UDS_SERVICES.get(service, 'Unknown')} (0x{service:02X})")
        
        # Wait for response
        timeout = time.time() + 2
        while time.time() < timeout:
            response = self.bus.recv(timeout=1)
            if response and response.arbitration_id == self.rx_id:
                print(f"Response: {response.data.hex()}")
                return response.data
        
        print("No response received")
        return None
    
    def start_diagnostic_session(self, session_type=0x03):
        """Start extended diagnostic session"""
        return self.send_uds_request(0x10, [session_type])
    
    def security_access_seed(self, level=0x01):
        """Request seed for security access"""
        return self.send_uds_request(0x27, [level])
    
    def read_did(self, did):
        """Read Data Identifier"""
        did_bytes = [(did >> 8) & 0xFF, did & 0xFF]
        return self.send_uds_request(0x22, did_bytes)
    
    def close(self):
        self.bus.shutdown()

# Usage
tester = UDSTester('can0', tx_id=0x7DF, rx_id=0x7E8)
tester.start_diagnostic_session(0x03)
tester.security_access_seed(0x01)
tester.read_did(0xF190)  # VIN
tester.close()
```

### 5. Replay Attack

Capture and replay CAN traffic:

```python
import can
import time

def capture_can_traffic(interface, duration=10, output_file='capture.log'):
    """Capture CAN traffic to file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    start_time = time.time()
    print(f"Capturing for {duration} seconds...")
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1)
        if msg:
            timestamp = time.time() - start_time
            messages.append((timestamp, msg))
            print(f"[{timestamp:.3f}] ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    
    bus.shutdown()
    
    # Save to file
    with open(output_file, 'w') as f:
        for ts, msg in messages:
            f.write(f"{ts},{msg.arbitration_id:03X},{msg.data.hex()}\n")
    
    print(f"Captured {len(messages)} frames to {output_file}")
    return messages

def replay_can_traffic(interface, input_file='capture.log'):
    """Replay captured CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    with open(input_file, 'r') as f:
        lines = f.readlines()
    
    print(f"Replaying {len(lines)} frames...")
    
    last_ts = 0
    for line in lines:
        ts, can_id, data = line.strip().split(',')
        ts = float(ts)
        can_id = int(can_id, 16)
        data = bytes.fromhex(data)
        
        # Wait for timing
        delay = ts - last_ts
        if delay > 0:
            time.sleep(delay)
        last_ts = ts
        
        msg = can.Message(arbitration_id=can_id, data=data, is_extended_id=False)
        bus.send(msg)
        print(f"Replayed: ID=0x{can_id:03X}")
    
    bus.shutdown()
    print("Replay complete")

# Capture door unlock sequence
capture_can_traffic('can0', duration=5, output_file='unlock.log')

# Replay the sequence later
replay_can_traffic('can0', input_file='unlock.log')
```

### 6. CAN ID Scanner

Discover active CAN IDs:

```python
import can
import time
from collections import defaultdict

def scan_can_ids(interface, duration=30):
    """Scan for active CAN IDs on the bus"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    id_stats = defaultdict(int)
    
    print(f"Scanning for {duration} seconds...")
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1)
        if msg:
            id_stats[msg.arbitration_id] += 1
    
    bus.shutdown()
    
    # Sort by frequency
    sorted_ids = sorted(id_stats.items(), key=lambda x: x[1], reverse=True)
    
    print("\nDiscovered CAN IDs:")
    print("ID      | Count  | Frequency")
    print("-" * 35)
    for can_id, count in sorted_ids:
        freq = count / duration
        print(f"0x{can_id:03X}  | {count:6d} | {freq:.2f} msg/s")
    
    return dict(id_stats)

# Usage
scan_can_ids('can0', duration=60)
```

## Configuration

### Environment Variables

```bash
# CAN interface configuration
export CAN_INTERFACE="can0"
export CAN_BITRATE="500000"

# Logging
export S800_LOG_LEVEL="INFO"
export S800_LOG_FILE="/var/log/s800/testing.log"

# Target ECU configuration
export TARGET_TX_ID="0x7DF"
export TARGET_RX_ID="0x7E8"
```

### Configuration File (config.json)

```json
{
  "interface": {
    "type": "socketcan",
    "channel": "can0",
    "bitrate": 500000
  },
  "targets": [
    {
      "name": "Engine ECU",
      "tx_id": "0x7E0",
      "rx_id": "0x7E8"
    },
    {
      "name": "Body Control Module",
      "tx_id": "0x7E1",
      "rx_id": "0x7E9"
    }
  ],
  "fuzzing": {
    "iterations": 1000,
    "delay_ms": 10,
    "id_range": [0, 2047]
  }
}
```

## Common Testing Patterns

### Pattern 1: Baseline Traffic Analysis

```python
def baseline_analysis(interface, duration=60):
    """Establish baseline of normal CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    baseline = {}
    
    print("Establishing baseline...")
    start = time.time()
    
    while time.time() - start < duration:
        msg = bus.recv(timeout=1)
        if msg:
            can_id = msg.arbitration_id
            if can_id not in baseline:
                baseline[can_id] = {
                    'count': 0,
                    'data_patterns': set()
                }
            baseline[can_id]['count'] += 1
            baseline[can_id]['data_patterns'].add(msg.data.hex())
    
    bus.shutdown()
    return baseline
```

### Pattern 2: Anomaly Detection

```python
def detect_anomalies(interface, baseline, duration=30):
    """Detect anomalies compared to baseline"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    anomalies = []
    
    print("Monitoring for anomalies...")
    start = time.time()
    
    while time.time() - start < duration:
        msg = bus.recv(timeout=1)
        if msg:
            can_id = msg.arbitration_id
            
            # Check for unknown IDs
            if can_id not in baseline:
                anomalies.append(f"Unknown ID: 0x{can_id:03X}")
                print(f"⚠ Unknown CAN ID detected: 0x{can_id:03X}")
            
            # Check for unusual data patterns
            elif msg.data.hex() not in baseline[can_id]['data_patterns']:
                anomalies.append(f"Unusual data on 0x{can_id:03X}: {msg.data.hex()}")
                print(f"⚠ Unusual data pattern: 0x{can_id:03X} = {msg.data.hex()}")
    
    bus.shutdown()
    return anomalies
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify CAN modules loaded
lsmod | grep can

# Load modules if needed
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Check interface status
ip -details link show can0
```

### Permission Denied

```bash
# Add user to dialout group for USB adapters
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 script.py
```

### No Traffic Received

```python
# Verify bitrate matches vehicle (common: 125k, 250k, 500k, 1000k)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Test with candump
candump can0

# Check for bus errors
ip -details -statistics link show can0
```

### Buffer Overflow

```python
# Increase receive buffer size
import can

bus = can.interface.Bus(
    channel='can0',
    bustype='socketcan',
    receive_own_messages=False
)

# Or via command line
sudo ip link set can0 txqueuelen 10000
```

## Safety Warnings

⚠️ **IMPORTANT**: 
- Only test on vehicles you own or have explicit permission to test
- Never test on public roads or production vehicles
- Use isolated test benches when possible
- Understand that fuzzing can cause unexpected vehicle behavior
- Always have emergency stop procedures in place
- Comply with local laws and regulations regarding vehicle modification

## Additional Resources

```bash
# Monitor CAN traffic
candump can0

# Generate CAN traffic for testing
cangen can0 -g 10 -I 123 -L 8

# Replay captured traffic
canplayer -I capture.log

# Filter specific IDs
candump can0,123:7FF  # Only show ID 0x123
```
