---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for CAN bus, automotive protocols, and ECU vulnerability assessment
triggers:
  - test vehicle network security
  - scan CAN bus for vulnerabilities
  - analyze automotive ECU communication
  - perform vehicle penetration testing
  - test car network protocols
  - assess automotive cybersecurity
  - fuzzing vehicle CAN messages
  - vehicle network intrusion detection
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for automotive cybersecurity professionals to assess vulnerabilities in vehicle networks, particularly CAN (Controller Area Network) bus systems and ECU (Electronic Control Unit) communications. The framework provides tools for message injection, fuzzing, monitoring, and analyzing automotive protocols.

**Key capabilities:**
- CAN bus message capture and analysis
- ECU vulnerability scanning
- Protocol fuzzing for automotive networks
- Network traffic replay and injection
- Real-time monitoring and intrusion detection
- UDS (Unified Diagnostic Services) testing

## Installation

### Requirements

- Python 3.7+
- CAN interface hardware (USB-to-CAN adapter, OBD-II dongle, or virtual CAN)
- Linux kernel with SocketCAN support (recommended)
- Root/administrator privileges for hardware access

### Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip install -r requirements.txt

# Install system dependencies (Linux)
sudo apt-get install can-utils

# Set up virtual CAN interface for testing (optional)
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

```bash
# Configure physical CAN interface (example with slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0

# Verify interface
ip link show can0
candump can0
```

## Core Components

### 1. CAN Message Capture

Capture and log CAN bus messages for analysis:

```python
import can
import time

def capture_can_messages(interface='vcan0', duration=10):
    """Capture CAN messages from specified interface"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    start_time = time.time()
    print(f"[*] Capturing CAN messages on {interface} for {duration} seconds...")
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append({
                'timestamp': msg.timestamp,
                'arbitration_id': hex(msg.arbitration_id),
                'data': msg.data.hex(),
                'dlc': msg.dlc
            })
            print(f"ID: {hex(msg.arbitration_id)} Data: {msg.data.hex()}")
    
    bus.shutdown()
    return messages

# Usage
captured = capture_can_messages('vcan0', duration=30)
print(f"[+] Captured {len(captured)} messages")
```

### 2. Message Injection

Send crafted CAN messages to test ECU responses:

```python
import can

def inject_can_message(interface='vcan0', arb_id=0x123, data=[0x01, 0x02, 0x03]):
    """Inject a CAN message onto the bus"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"[+] Sent message - ID: {hex(arb_id)}, Data: {bytes(data).hex()}")
        return True
    except can.CanError as e:
        print(f"[-] Failed to send message: {e}")
        return False
    finally:
        bus.shutdown()

# Usage
inject_can_message('vcan0', arb_id=0x7DF, data=[0x02, 0x01, 0x00])
```

### 3. CAN Fuzzing

Fuzz CAN messages to discover vulnerabilities:

```python
import can
import random
import time

def fuzz_can_messages(interface='vcan0', arb_id_range=(0x000, 0x7FF), 
                     num_messages=100, delay=0.01):
    """Fuzz CAN bus with random messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Starting CAN fuzzing on {interface}")
    print(f"[*] ID range: {hex(arb_id_range[0])} - {hex(arb_id_range[1])}")
    
    for i in range(num_messages):
        arb_id = random.randint(arb_id_range[0], arb_id_range[1])
        data_length = random.randint(1, 8)
        data = [random.randint(0, 255) for _ in range(data_length)]
        
        msg = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[{i+1}/{num_messages}] ID: {hex(arb_id)}, Data: {bytes(data).hex()}")
            time.sleep(delay)
        except can.CanError as e:
            print(f"[-] Error sending message: {e}")
    
    bus.shutdown()
    print("[+] Fuzzing completed")

# Usage - fuzz specific ID range
fuzz_can_messages('vcan0', arb_id_range=(0x100, 0x200), num_messages=50)
```

### 4. UDS Diagnostic Testing

Test UDS (Unified Diagnostic Services) protocol:

```python
import can
import time

class UDSClient:
    """Simple UDS client for diagnostic testing"""
    
    def __init__(self, interface='vcan0', request_id=0x7DF, response_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.request_id = request_id
        self.response_id = response_id
    
    def send_request(self, service, data=[]):
        """Send UDS request"""
        payload = [len(data) + 1, service] + data
        msg = can.Message(
            arbitration_id=self.request_id,
            data=payload,
            is_extended_id=False
        )
        self.bus.send(msg)
        print(f"[>] UDS Request: Service {hex(service)}, Data: {bytes(payload).hex()}")
    
    def read_response(self, timeout=2.0):
        """Read UDS response"""
        start_time = time.time()
        while time.time() - start_time < timeout:
            msg = self.bus.recv(timeout=0.5)
            if msg and msg.arbitration_id == self.response_id:
                print(f"[<] UDS Response: {msg.data.hex()}")
                return msg.data
        print("[-] No response received")
        return None
    
    def read_dtc(self):
        """Read Diagnostic Trouble Codes (Service 0x19)"""
        print("[*] Reading DTCs...")
        self.send_request(0x19, [0x02, 0xFF])
        return self.read_response()
    
    def read_vin(self):
        """Read Vehicle Identification Number (Service 0x22)"""
        print("[*] Reading VIN...")
        self.send_request(0x22, [0xF1, 0x90])
        return self.read_response()
    
    def close(self):
        self.bus.shutdown()

# Usage
uds = UDSClient('vcan0')
uds.read_vin()
uds.read_dtc()
uds.close()
```

### 5. Traffic Analysis

Analyze captured CAN traffic for patterns:

```python
from collections import Counter
import json

def analyze_can_traffic(messages):
    """Analyze CAN message patterns"""
    arb_ids = [msg['arbitration_id'] for msg in messages]
    id_counter = Counter(arb_ids)
    
    print("[*] CAN Traffic Analysis")
    print(f"Total messages: {len(messages)}")
    print(f"Unique IDs: {len(id_counter)}")
    print("\nTop 10 most frequent IDs:")
    
    for arb_id, count in id_counter.most_common(10):
        percentage = (count / len(messages)) * 100
        print(f"  {arb_id}: {count} messages ({percentage:.2f}%)")
    
    # Identify periodic messages
    print("\n[*] Detecting periodic messages...")
    periodic = detect_periodic_messages(messages)
    for arb_id, period in periodic.items():
        print(f"  {arb_id}: ~{period:.2f}ms period")
    
    return {
        'total_messages': len(messages),
        'unique_ids': len(id_counter),
        'id_frequency': dict(id_counter),
        'periodic': periodic
    }

def detect_periodic_messages(messages, threshold=0.9):
    """Detect messages sent at regular intervals"""
    from collections import defaultdict
    
    timestamps_by_id = defaultdict(list)
    for msg in messages:
        timestamps_by_id[msg['arbitration_id']].append(msg['timestamp'])
    
    periodic = {}
    for arb_id, timestamps in timestamps_by_id.items():
        if len(timestamps) < 5:
            continue
        
        intervals = [timestamps[i+1] - timestamps[i] 
                    for i in range(len(timestamps)-1)]
        avg_interval = sum(intervals) / len(intervals)
        variance = sum((x - avg_interval)**2 for x in intervals) / len(intervals)
        
        if variance < (avg_interval * 0.1):  # Low variance = periodic
            periodic[arb_id] = avg_interval * 1000  # Convert to ms
    
    return periodic

# Usage
analysis = analyze_can_traffic(captured)
```

## Configuration

### Environment Variables

```bash
# CAN interface configuration
export CAN_INTERFACE=can0
export CAN_BITRATE=500000

# UDS configuration
export UDS_REQUEST_ID=0x7DF
export UDS_RESPONSE_ID=0x7E8

# Logging
export LOG_LEVEL=INFO
export LOG_FILE=/var/log/s800/can_traffic.log
```

### Configuration File Example

```python
# config.py
import os

class S800Config:
    # CAN Interface
    CAN_INTERFACE = os.getenv('CAN_INTERFACE', 'vcan0')
    CAN_BITRATE = int(os.getenv('CAN_BITRATE', 500000))
    
    # UDS Settings
    UDS_REQUEST_ID = int(os.getenv('UDS_REQUEST_ID', '0x7DF'), 16)
    UDS_RESPONSE_ID = int(os.getenv('UDS_RESPONSE_ID', '0x7E8'), 16)
    
    # Fuzzing
    FUZZ_DELAY = float(os.getenv('FUZZ_DELAY', 0.01))
    FUZZ_ID_MIN = int(os.getenv('FUZZ_ID_MIN', '0x000'), 16)
    FUZZ_ID_MAX = int(os.getenv('FUZZ_ID_MAX', '0x7FF'), 16)
    
    # Logging
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    LOG_FILE = os.getenv('LOG_FILE', 'can_traffic.log')
```

## Common Testing Patterns

### Pattern 1: Baseline Capture

Establish normal traffic baseline:

```python
import can
import json
from datetime import datetime

def capture_baseline(interface='vcan0', duration=300):
    """Capture baseline traffic profile"""
    print(f"[*] Capturing baseline for {duration}s...")
    messages = capture_can_messages(interface, duration)
    
    baseline = {
        'timestamp': datetime.now().isoformat(),
        'duration': duration,
        'messages': messages,
        'analysis': analyze_can_traffic(messages)
    }
    
    filename = f"baseline_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    with open(filename, 'w') as f:
        json.dump(baseline, f, indent=2)
    
    print(f"[+] Baseline saved to {filename}")
    return baseline
```

### Pattern 2: Replay Attack Testing

```python
def replay_attack(messages, interface='vcan0', speed_multiplier=1.0):
    """Replay captured messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Replaying {len(messages)} messages at {speed_multiplier}x speed")
    
    for i, msg_data in enumerate(messages):
        if i > 0:
            delay = (messages[i]['timestamp'] - messages[i-1]['timestamp']) / speed_multiplier
            time.sleep(delay)
        
        msg = can.Message(
            arbitration_id=int(msg_data['arbitration_id'], 16),
            data=bytes.fromhex(msg_data['data']),
            is_extended_id=False
        )
        
        bus.send(msg)
        print(f"[{i+1}/{len(messages)}] Replayed: {msg_data['arbitration_id']}")
    
    bus.shutdown()
    print("[+] Replay completed")
```

### Pattern 3: Anomaly Detection

```python
def detect_anomalies(baseline, live_messages):
    """Compare live traffic against baseline"""
    baseline_ids = set(baseline['analysis']['id_frequency'].keys())
    live_ids = set(msg['arbitration_id'] for msg in live_messages)
    
    new_ids = live_ids - baseline_ids
    missing_ids = baseline_ids - live_ids
    
    print("[*] Anomaly Detection Results:")
    if new_ids:
        print(f"[!] New IDs detected: {', '.join(new_ids)}")
    if missing_ids:
        print(f"[!] Missing IDs: {', '.join(missing_ids)}")
    
    return {
        'new_ids': list(new_ids),
        'missing_ids': list(missing_ids)
    }
```

## Troubleshooting

### Interface Not Found

```bash
# Check available interfaces
ip link show

# Ensure SocketCAN module loaded
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Set up virtual CAN for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Permission Denied

```bash
# Add user to dialout group (for USB devices)
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 your_script.py
```

### No Messages Received

```python
# Verify bus is active
import can

bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
print(f"Bus state: {bus.state}")
print(f"Channel info: {bus.channel_info}")
bus.shutdown()
```

### Hardware Compatibility

```bash
# Test CAN interface with candump
candump can0

# Send test message with cansend
cansend can0 123#DEADBEEF

# Check interface statistics
ip -details -statistics link show can0
```

## Safety Warnings

- **Never test on production vehicles without authorization**
- **Always use isolated test environments**
- **Vehicle networks control safety-critical systems**
- **Unauthorized access may be illegal**
- **Fuzzing can cause ECU malfunctions or damage**
- **Always have kill switches and recovery procedures ready**
