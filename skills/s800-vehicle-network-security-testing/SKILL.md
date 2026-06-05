---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, sniffing, and attack simulation capabilities
triggers:
  - test vehicle network security
  - fuzz CAN bus messages
  - analyze automotive network traffic
  - simulate vehicle network attacks
  - sniff CAN or LIN bus
  - perform vehicle penetration testing
  - test ECU security
  - automotive security assessment
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

S800 is a comprehensive security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for fuzzing, traffic sniffing, message injection, replay attacks, and security assessment of Electronic Control Units (ECUs).

## Installation

### Prerequisites

```bash
# Install SocketCAN utilities (Linux)
sudo apt-get install can-utils

# Install Python dependencies
pip install python-can cantools
```

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Setup CAN interface (example for virtual CAN)
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For physical testing:
- CAN adapter (e.g., CANable, PCAN-USB, Kvaser)
- OBD-II connector for vehicle access
- Proper isolation and safety equipment

```bash
# Setup physical CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Components

### 1. CAN Bus Sniffer

Monitor and capture CAN traffic:

```python
import can
import time

def sniff_can_traffic(interface='vcan0', duration=10):
    """Capture CAN bus traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Sniffing on {interface} for {duration} seconds...")
    start_time = time.time()
    messages = []
    
    try:
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                messages.append(msg)
                print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    except KeyboardInterrupt:
        print("\n[*] Sniffing stopped")
    finally:
        bus.shutdown()
    
    return messages

# Usage
captured_msgs = sniff_can_traffic('can0', duration=30)
```

### 2. Message Injection

Send crafted CAN messages:

```python
import can

def inject_can_message(interface, can_id, data):
    """Inject a CAN message onto the bus"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Create message
    msg = can.Message(
        arbitration_id=can_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"[+] Sent CAN ID 0x{can_id:03X}: {data.hex()}")
    except can.CanError as e:
        print(f"[-] Failed to send message: {e}")
    finally:
        bus.shutdown()

# Example: Unlock doors (theoretical)
inject_can_message('can0', 0x3B3, bytes([0x01, 0x00, 0x00, 0x00]))
```

### 3. Fuzzing Module

Automated fuzzing for vulnerability discovery:

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        
    def fuzz_random(self, can_id, iterations=1000, delay=0.01):
        """Fuzz a specific CAN ID with random data"""
        print(f"[*] Fuzzing CAN ID 0x{can_id:03X} - {iterations} iterations")
        
        for i in range(iterations):
            # Generate random data (8 bytes for standard CAN)
            data = bytes([random.randint(0, 255) for _ in range(8)])
            
            msg = can.Message(
                arbitration_id=can_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                if i % 100 == 0:
                    print(f"[*] Progress: {i}/{iterations}")
                time.sleep(delay)
            except can.CanError as e:
                print(f"[-] Error at iteration {i}: {e}")
                
    def fuzz_bit_flip(self, can_id, base_data, iterations=256):
        """Fuzz by flipping individual bits"""
        print(f"[*] Bit-flip fuzzing CAN ID 0x{can_id:03X}")
        
        for byte_idx in range(len(base_data)):
            for bit_idx in range(8):
                data = bytearray(base_data)
                data[byte_idx] ^= (1 << bit_idx)  # Flip bit
                
                msg = can.Message(
                    arbitration_id=can_id,
                    data=bytes(data),
                    is_extended_id=False
                )
                
                self.bus.send(msg)
                time.sleep(0.01)
                
    def close(self):
        self.bus.shutdown()

# Usage
fuzzer = CANFuzzer('can0')
fuzzer.fuzz_random(0x123, iterations=500)
fuzzer.fuzz_bit_flip(0x456, bytes([0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07]))
fuzzer.close()
```

### 4. Replay Attack

Capture and replay CAN messages:

```python
import can
import time
import pickle

class ReplayAttack:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.captured_messages = []
        
    def capture(self, duration=10):
        """Capture messages for replay"""
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        print(f"[*] Capturing for {duration} seconds...")
        
        start_time = time.time()
        timestamps = []
        
        try:
            while time.time() - start_time < duration:
                msg = bus.recv(timeout=1.0)
                if msg:
                    relative_time = time.time() - start_time
                    self.captured_messages.append((relative_time, msg))
                    timestamps.append(relative_time)
        finally:
            bus.shutdown()
            
        print(f"[+] Captured {len(self.captured_messages)} messages")
        
    def replay(self, speed_multiplier=1.0):
        """Replay captured messages"""
        if not self.captured_messages:
            print("[-] No messages to replay")
            return
            
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        print(f"[*] Replaying {len(self.captured_messages)} messages (speed: {speed_multiplier}x)")
        
        start_time = time.time()
        for timestamp, msg in self.captured_messages:
            # Calculate when to send this message
            target_time = start_time + (timestamp / speed_multiplier)
            
            # Wait until target time
            while time.time() < target_time:
                time.sleep(0.001)
                
            bus.send(msg)
            print(f"[+] Replayed 0x{msg.arbitration_id:03X}: {msg.data.hex()}")
            
        bus.shutdown()
        
    def save(self, filename):
        """Save captured messages to file"""
        with open(filename, 'wb') as f:
            pickle.dump(self.captured_messages, f)
        print(f"[+] Saved to {filename}")
        
    def load(self, filename):
        """Load captured messages from file"""
        with open(filename, 'rb') as f:
            self.captured_messages = pickle.load(f)
        print(f"[+] Loaded {len(self.captured_messages)} messages")

# Usage
replay = ReplayAttack('can0')
replay.capture(duration=20)
replay.save('captured_traffic.pkl')
replay.replay(speed_multiplier=2.0)  # Replay at 2x speed
```

### 5. DBC File Parser

Parse CAN database files for message definitions:

```python
import cantools

def load_dbc_and_decode(dbc_path, can_msg):
    """Load DBC file and decode CAN message"""
    db = cantools.database.load_file(dbc_path)
    
    try:
        # Decode message
        decoded = db.decode_message(can_msg.arbitration_id, can_msg.data)
        print(f"[+] Decoded 0x{can_msg.arbitration_id:03X}:")
        for signal, value in decoded.items():
            print(f"    {signal}: {value}")
        return decoded
    except KeyError:
        print(f"[-] CAN ID 0x{can_msg.arbitration_id:03X} not in DBC")
        return None

# Monitor and decode messages
def monitor_with_dbc(interface, dbc_path):
    """Monitor CAN bus and decode messages using DBC"""
    db = cantools.database.load_file(dbc_path)
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Monitoring {interface} with DBC: {dbc_path}")
    
    try:
        while True:
            msg = bus.recv(timeout=1.0)
            if msg:
                try:
                    decoded = db.decode_message(msg.arbitration_id, msg.data)
                    print(f"\n0x{msg.arbitration_id:03X} - {decoded}")
                except KeyError:
                    print(f"\n0x{msg.arbitration_id:03X} - Unknown: {msg.data.hex()}")
    except KeyboardInterrupt:
        print("\n[*] Stopped")
    finally:
        bus.shutdown()

# Usage
monitor_with_dbc('can0', 'vehicle_database.dbc')
```

## Common Testing Patterns

### Diagnostic Session Testing

```python
def send_uds_diagnostic(interface, ecu_id, service, data=None):
    """Send UDS (Unified Diagnostic Services) command"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Build diagnostic message
    payload = [service]
    if data:
        payload.extend(data)
        
    msg = can.Message(
        arbitration_id=ecu_id,
        data=bytes(payload),
        is_extended_id=False
    )
    
    bus.send(msg)
    print(f"[+] Sent UDS service 0x{service:02X} to ECU 0x{ecu_id:03X}")
    
    # Wait for response
    response = bus.recv(timeout=2.0)
    if response:
        print(f"[+] Response: {response.data.hex()}")
        return response
    else:
        print("[-] No response")
        return None
        
    bus.shutdown()

# Example: Read VIN
send_uds_diagnostic('can0', 0x7E0, 0x22, [0xF1, 0x90])
```

### Traffic Analysis

```python
def analyze_traffic_patterns(messages):
    """Analyze captured CAN traffic for patterns"""
    id_frequency = {}
    id_data_patterns = {}
    
    for msg in messages:
        can_id = msg.arbitration_id
        
        # Count frequency
        id_frequency[can_id] = id_frequency.get(can_id, 0) + 1
        
        # Track unique data patterns
        if can_id not in id_data_patterns:
            id_data_patterns[can_id] = set()
        id_data_patterns[can_id].add(msg.data.hex())
    
    # Report findings
    print("\n[*] Traffic Analysis:")
    print(f"    Unique CAN IDs: {len(id_frequency)}")
    
    print("\n[*] Most Frequent IDs:")
    sorted_ids = sorted(id_frequency.items(), key=lambda x: x[1], reverse=True)
    for can_id, count in sorted_ids[:10]:
        unique_patterns = len(id_data_patterns[can_id])
        print(f"    0x{can_id:03X}: {count} messages, {unique_patterns} unique patterns")

# Usage
messages = sniff_can_traffic('can0', duration=60)
analyze_traffic_patterns(messages)
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Set logging level
export S800_LOG_LEVEL=DEBUG

# Set output directory
export S800_OUTPUT_DIR=/tmp/s800_results
```

### Configuration File

Create `s800_config.json`:

```json
{
  "interface": "can0",
  "bitrate": 500000,
  "logging": {
    "level": "INFO",
    "output": "s800.log"
  },
  "fuzzing": {
    "default_iterations": 1000,
    "delay_ms": 10
  },
  "safety": {
    "enable_kill_switch": true,
    "monitored_ids": ["0x7E0", "0x7E8"]
  }
}
```

## Safety Considerations

**CRITICAL**: Vehicle network testing can be dangerous. Always:

```python
# Implement safety checks
CRITICAL_IDS = [0x2C1, 0x3B3, 0x245]  # Brakes, steering, airbags

def safe_inject(interface, can_id, data):
    """Inject message with safety checks"""
    if can_id in CRITICAL_IDS:
        response = input(f"[!] WARNING: 0x{can_id:03X} is critical. Continue? (yes/no): ")
        if response.lower() != 'yes':
            print("[-] Aborted")
            return False
    
    inject_can_message(interface, can_id, data)
    return True
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify SocketCAN module
lsmod | grep can

# Check permissions
sudo usermod -a -G dialout $USER
```

### Permission Denied

```bash
# Add user to required groups
sudo usermod -a -G can $USER
sudo usermod -a -G dialout $USER

# Reload groups
newgrp can
```

### No Messages Received

```python
# Verify bus activity
import os
os.system('candump can0')

# Check bitrate matches
os.system('ip -details link show can0')
```

### Fuzzing Crashes ECU

Implement monitoring and auto-recovery:

```python
def monitor_ecu_health(interface, ecu_id, timeout=5):
    """Check if ECU responds"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Send ping
    ping = can.Message(arbitration_id=ecu_id, data=[0x3E, 0x00])
    bus.send(ping)
    
    response = bus.recv(timeout=timeout)
    bus.shutdown()
    
    return response is not None
```
