---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks including CAN bus, LIN, FlexRay and other vehicle communication protocols
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive penetration testing
  - scan vehicle network vulnerabilities
  - test car communication protocols
  - vehicle security assessment framework
  - automotive network fuzzing
  - vehicle ECU testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks. It provides tools for testing, analyzing, and securing vehicle communication protocols including CAN (Controller Area Network), LIN (Local Interconnect Network), FlexRay, and other automotive bus systems. The framework enables security researchers and automotive engineers to perform penetration testing, vulnerability scanning, protocol fuzzing, and traffic analysis on vehicle networks.

**Note**: This is a testing framework. Only use on authorized systems and vehicles. Unauthorized vehicle network testing may be illegal and dangerous.

## Installation

### Prerequisites

```bash
# Install required system dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y python3 python3-pip can-utils git

# Install SocketCAN kernel modules (if not already loaded)
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Or install manually if requirements.txt is minimal
pip3 install python-can cantools scapy
```

### Virtual CAN Setup (for testing without hardware)

```bash
# Create virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Verify interface
ifconfig vcan0
```

## Core Components

### 1. CAN Bus Testing

#### Sniffing CAN Traffic

```python
import can
import time

# Initialize CAN interface
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

def sniff_can_traffic(duration=10):
    """Capture CAN frames for specified duration"""
    print(f"Sniffing CAN traffic for {duration} seconds...")
    start_time = time.time()
    frames = []
    
    while time.time() - start_time < duration:
        message = bus.recv(timeout=1.0)
        if message:
            frames.append({
                'timestamp': message.timestamp,
                'arbitration_id': hex(message.arbitration_id),
                'data': message.data.hex(),
                'dlc': message.dlc
            })
            print(f"ID: {hex(message.arbitration_id)} Data: {message.data.hex()}")
    
    return frames

# Capture traffic
captured_frames = sniff_can_traffic(duration=30)
```

#### Sending CAN Messages

```python
import can

bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

def send_can_frame(arbitration_id, data):
    """Send a CAN frame"""
    message = can.Message(
        arbitration_id=arbitration_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(message)
        print(f"Sent: ID={hex(arbitration_id)}, Data={data.hex()}")
        return True
    except can.CanError as e:
        print(f"Error sending message: {e}")
        return False

# Example: Send engine RPM data (example arbitration ID)
send_can_frame(0x201, bytes([0x10, 0x20, 0x30, 0x40, 0x50, 0x60, 0x70, 0x80]))
```

#### CAN Fuzzing

```python
import can
import random
import time

bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

def fuzz_can_ids(start_id=0x000, end_id=0x7FF, delay=0.01):
    """Fuzz CAN arbitration IDs with random data"""
    print(f"Fuzzing CAN IDs from {hex(start_id)} to {hex(end_id)}")
    
    for arb_id in range(start_id, end_id + 1):
        # Generate random 8-byte payload
        data = bytes([random.randint(0, 255) for _ in range(8)])
        
        message = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(message)
            print(f"Fuzzed ID: {hex(arb_id)}")
        except can.CanError as e:
            print(f"Error at ID {hex(arb_id)}: {e}")
        
        time.sleep(delay)

# Fuzz a specific range
fuzz_can_ids(start_id=0x200, end_id=0x2FF, delay=0.005)
```

### 2. Protocol Analysis

#### DBC File Parsing

```python
import cantools

def parse_dbc_file(dbc_path):
    """Parse DBC database file for CAN protocol definitions"""
    db = cantools.database.load_file(dbc_path)
    
    print(f"Database: {db.version}")
    print(f"Messages: {len(db.messages)}")
    
    for message in db.messages:
        print(f"\nMessage: {message.name}")
        print(f"  ID: {hex(message.frame_id)}")
        print(f"  Length: {message.length} bytes")
        print(f"  Signals:")
        for signal in message.signals:
            print(f"    - {signal.name}: {signal.unit}")
    
    return db

# Load and parse DBC file
# db = parse_dbc_file('vehicle_protocol.dbc')
```

#### Decode CAN Messages

```python
import can
import cantools

def decode_can_message(db, message):
    """Decode CAN message using DBC database"""
    try:
        decoded = db.decode_message(message.arbitration_id, message.data)
        print(f"Decoded message {hex(message.arbitration_id)}:")
        for signal_name, value in decoded.items():
            print(f"  {signal_name}: {value}")
        return decoded
    except KeyError:
        print(f"Unknown message ID: {hex(message.arbitration_id)}")
        return None

# Example usage
# db = cantools.database.load_file('vehicle.dbc')
# bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
# 
# while True:
#     message = bus.recv()
#     if message:
#         decode_can_message(db, message)
```

### 3. Replay Attacks

```python
import can
import time
import pickle

class CANReplay:
    def __init__(self, channel='vcan0'):
        self.bus = can.interface.Bus(channel=channel, bustype='socketcan')
        self.recorded_frames = []
    
    def record(self, duration=60, output_file='capture.pkl'):
        """Record CAN traffic"""
        print(f"Recording for {duration} seconds...")
        start_time = time.time()
        
        while time.time() - start_time < duration:
            message = self.bus.recv(timeout=1.0)
            if message:
                self.recorded_frames.append({
                    'timestamp': message.timestamp - start_time,
                    'arbitration_id': message.arbitration_id,
                    'data': list(message.data),
                    'is_extended_id': message.is_extended_id
                })
        
        with open(output_file, 'wb') as f:
            pickle.dump(self.recorded_frames, f)
        
        print(f"Recorded {len(self.recorded_frames)} frames to {output_file}")
    
    def replay(self, input_file='capture.pkl', speed=1.0):
        """Replay recorded CAN traffic"""
        with open(input_file, 'rb') as f:
            frames = pickle.load(f)
        
        print(f"Replaying {len(frames)} frames at {speed}x speed...")
        start_time = time.time()
        
        for frame in frames:
            # Wait for appropriate timestamp
            target_time = frame['timestamp'] / speed
            while time.time() - start_time < target_time:
                time.sleep(0.001)
            
            message = can.Message(
                arbitration_id=frame['arbitration_id'],
                data=bytes(frame['data']),
                is_extended_id=frame['is_extended_id']
            )
            
            self.bus.send(message)
        
        print("Replay complete")

# Usage
# replay = CANReplay(channel='vcan0')
# replay.record(duration=30, output_file='unlock_sequence.pkl')
# replay.replay(input_file='unlock_sequence.pkl', speed=1.0)
```

### 4. Vulnerability Scanning

```python
import can
import time

class VehicleScanner:
    def __init__(self, channel='vcan0'):
        self.bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    def scan_active_ids(self, timeout=5):
        """Identify active CAN IDs on the bus"""
        print(f"Scanning for active IDs (timeout: {timeout}s)...")
        active_ids = set()
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            message = self.bus.recv(timeout=0.1)
            if message:
                active_ids.add(message.arbitration_id)
        
        print(f"Found {len(active_ids)} active IDs:")
        for arb_id in sorted(active_ids):
            print(f"  {hex(arb_id)}")
        
        return list(active_ids)
    
    def test_diagnostic_services(self):
        """Test for UDS (Unified Diagnostic Services) endpoints"""
        diagnostic_ids = [0x7DF, 0x7E0, 0x7E8]  # Common UDS IDs
        
        for diag_id in diagnostic_ids:
            # Send diagnostic session control request
            request = can.Message(
                arbitration_id=diag_id,
                data=bytes([0x02, 0x10, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00]),
                is_extended_id=False
            )
            
            self.bus.send(request)
            print(f"Sent diagnostic request to {hex(diag_id)}")
            
            # Wait for response
            time.sleep(0.1)
            response = self.bus.recv(timeout=1.0)
            if response:
                print(f"  Response: {response.data.hex()}")

# Usage
# scanner = VehicleScanner(channel='vcan0')
# active_ids = scanner.scan_active_ids(timeout=10)
# scanner.test_diagnostic_services()
```

## Configuration

### Interface Configuration

```python
# config.py
CAN_INTERFACE = {
    'channel': 'vcan0',  # or 'can0' for physical interface
    'bustype': 'socketcan',
    'bitrate': 500000  # 500 kbit/s (common for CAN)
}

# For hardware interfaces
HARDWARE_CONFIG = {
    'interface': 'socketcan',
    'channel': 'can0',
    'bitrate': 500000,
    'receive_own_messages': False
}

# Logging configuration
LOGGING_CONFIG = {
    'log_file': 'can_capture.log',
    'format': 'csv',  # or 'json', 'pickle'
    'timestamp': True
}
```

### Environment Variables

```python
import os

# Use environment variables for sensitive configuration
CAN_CHANNEL = os.getenv('CAN_CHANNEL', 'vcan0')
LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
OUTPUT_DIR = os.getenv('OUTPUT_DIR', './output')
```

## Common Testing Patterns

### Pattern 1: Traffic Analysis Workflow

```python
import can
from collections import defaultdict
import time

def analyze_traffic_patterns(channel='vcan0', duration=60):
    """Analyze CAN traffic patterns and frequencies"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    id_stats = defaultdict(lambda: {'count': 0, 'data_samples': []})
    start_time = time.time()
    
    print(f"Analyzing traffic for {duration} seconds...")
    
    while time.time() - start_time < duration:
        message = bus.recv(timeout=1.0)
        if message:
            arb_id = message.arbitration_id
            id_stats[arb_id]['count'] += 1
            id_stats[arb_id]['data_samples'].append(message.data.hex())
    
    print("\nTraffic Analysis Results:")
    print("-" * 60)
    
    for arb_id in sorted(id_stats.keys()):
        stats = id_stats[arb_id]
        frequency = stats['count'] / duration
        unique_data = len(set(stats['data_samples']))
        
        print(f"ID: {hex(arb_id)}")
        print(f"  Count: {stats['count']}")
        print(f"  Frequency: {frequency:.2f} Hz")
        print(f"  Unique patterns: {unique_data}")
        print()
    
    return id_stats

# Run analysis
# stats = analyze_traffic_patterns(duration=30)
```

### Pattern 2: Man-in-the-Middle Setup

```python
import can
import threading

class CANBridge:
    """Bridge between two CAN interfaces for MITM attacks"""
    
    def __init__(self, if1='can0', if2='can1'):
        self.bus1 = can.interface.Bus(channel=if1, bustype='socketcan')
        self.bus2 = can.interface.Bus(channel=if2, bustype='socketcan')
        self.running = False
        self.message_filter = None
    
    def set_filter(self, filter_func):
        """Set a filter function to modify/drop messages"""
        self.message_filter = filter_func
    
    def forward_messages(self, source_bus, dest_bus, name):
        """Forward messages from source to destination"""
        while self.running:
            message = source_bus.recv(timeout=0.1)
            if message:
                # Apply filter if set
                if self.message_filter:
                    message = self.message_filter(message)
                
                if message:  # Forward if not filtered out
                    dest_bus.send(message)
                    print(f"{name}: Forwarded {hex(message.arbitration_id)}")
    
    def start(self):
        """Start bridging"""
        self.running = True
        
        t1 = threading.Thread(target=self.forward_messages, 
                             args=(self.bus1, self.bus2, "IF1->IF2"))
        t2 = threading.Thread(target=self.forward_messages,
                             args=(self.bus2, self.bus1, "IF2->IF1"))
        
        t1.start()
        t2.start()
        
        print("MITM bridge started")
    
    def stop(self):
        """Stop bridging"""
        self.running = False
        print("MITM bridge stopped")

# Example filter function
def rpm_limiter_filter(message):
    """Limit engine RPM by modifying CAN messages"""
    if message.arbitration_id == 0x201:  # Example RPM message
        data = bytearray(message.data)
        # Modify RPM bytes (implementation depends on protocol)
        data[0] = min(data[0], 0x50)  # Limit value
        message.data = bytes(data)
    return message

# Usage
# bridge = CANBridge(if1='can0', if2='can1')
# bridge.set_filter(rpm_limiter_filter)
# bridge.start()
```

## Troubleshooting

### Interface Issues

```bash
# Check if CAN interface exists
ip link show can0

# Bring interface up
sudo ip link set can0 up type can bitrate 500000

# Check interface status
ip -details link show can0

# Monitor errors
candump can0,0:0,#FFFFFFFF  # Show all frames including errors
```

### Permission Issues

```bash
# Add user to dialout group for serial device access
sudo usermod -a -G dialout $USER

# Set permissions for CAN interface
sudo ip link set can0 up type can bitrate 500000
sudo chmod 666 /dev/can0
```

### Python Debugging

```python
import can
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

# Test interface connection
try:
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    print("Connected successfully")
    
    # Send test message
    test_msg = can.Message(arbitration_id=0x123, data=[0x01, 0x02, 0x03])
    bus.send(test_msg)
    print("Test message sent")
    
except can.CanError as e:
    print(f"CAN Error: {e}")
except Exception as e:
    print(f"Error: {e}")
```

### Hardware Interface Issues

```bash
# Check kernel modules
lsmod | grep can

# Load required modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe slcan  # For serial CAN adapters

# Test with candump
candump -L can0  # Log frames with timestamps

# Test sending
cansend can0 123#DEADBEEF
```

## Safety and Legal Considerations

**CRITICAL WARNINGS:**

- Only test on vehicles you own or have explicit authorization to test
- Never test on public roads or production vehicles
- Vehicle network manipulation can cause safety hazards
- Some jurisdictions have laws against unauthorized vehicle network access
- Always use appropriate safety measures and emergency shutoff procedures
- Test in controlled environments with proper safety equipment

```python
# Example safety wrapper
class SafetyWrapper:
    def __init__(self, bus):
        self.bus = bus
        self.emergency_stop = False
    
    def send_with_safety(self, message, critical_ids=None):
        """Send message with safety checks"""
        if self.emergency_stop:
            print("EMERGENCY STOP ACTIVE - Message blocked")
            return False
        
        # Block critical system IDs by default
        if critical_ids and message.arbitration_id in critical_ids:
            print(f"BLOCKED: Critical ID {hex(message.arbitration_id)}")
            return False
        
        return self.bus.send(message)
```

## Resources

- CAN Bus specification: ISO 11898
- UDS protocol: ISO 14229
- J1939 (Heavy vehicle): SAE J1939
- python-can documentation: https://python-can.readthedocs.io/
