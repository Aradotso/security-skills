---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for automotive CAN bus, LIN, and FlexRay protocol analysis and vulnerability assessment
triggers:
  - test vehicle network security with S800
  - analyze CAN bus vulnerabilities
  - perform automotive security testing
  - use S800 framework for vehicle networks
  - test car communication protocols
  - scan automotive network for vulnerabilities
  - fuzzing vehicle CAN messages
  - S800 vehicle security assessment
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive vehicle network security testing framework designed for analyzing and assessing vulnerabilities in automotive communication protocols including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay. The framework provides tools for protocol analysis, fuzzing, message injection, and security vulnerability discovery in vehicle networks.

**Note**: This is a test/research framework. Always ensure you have proper authorization before testing any vehicle systems.

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can
pip install cantools
pip install scapy

# For hardware interface support (SocketCAN on Linux)
sudo apt-get install can-utils

# Load kernel modules for CAN
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Clone and Setup

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install framework dependencies
pip install -r requirements.txt

# Setup virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For real vehicle testing, you'll need a CAN interface adapter:

```bash
# Configure physical CAN interface (example with slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0

# Set CAN bitrate (typically 500kbps for automotive)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Components

### 1. CAN Bus Scanner

Scan and monitor CAN bus traffic:

```python
import can
import time

def scan_can_bus(interface='vcan0', duration=10):
    """Scan CAN bus and capture messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    start_time = time.time()
    print(f"Scanning {interface} for {duration} seconds...")
    
    try:
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
    except KeyboardInterrupt:
        print("\nScan interrupted")
    finally:
        bus.shutdown()
    
    return messages

# Usage
captured = scan_can_bus('vcan0', duration=30)
print(f"\nCaptured {len(captured)} messages")
```

### 2. Message Injection

Inject custom CAN messages for testing:

```python
import can

def inject_can_message(interface, arb_id, data):
    """Inject a CAN message onto the bus"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Create message
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"Sent: ID={hex(arb_id)} Data={data.hex()}")
        return True
    except can.CanError as e:
        print(f"Error sending message: {e}")
        return False
    finally:
        bus.shutdown()

# Example: Inject test message
inject_can_message('vcan0', 0x123, bytes([0x11, 0x22, 0x33, 0x44]))

# Inject multiple messages
def inject_sequence(interface, messages):
    """Inject a sequence of CAN messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    for arb_id, data, delay in messages:
        msg = can.Message(arbitration_id=arb_id, data=data)
        bus.send(msg)
        print(f"Sent: {hex(arb_id)}")
        time.sleep(delay)
    
    bus.shutdown()

# Usage
test_sequence = [
    (0x100, bytes([0x01, 0x02, 0x03]), 0.1),
    (0x200, bytes([0x04, 0x05, 0x06]), 0.1),
    (0x300, bytes([0x07, 0x08, 0x09]), 0.1),
]
inject_sequence('vcan0', test_sequence)
```

### 3. CAN Fuzzing

Fuzz CAN bus messages to discover vulnerabilities:

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def fuzz_random(self, duration=60, min_id=0x000, max_id=0x7FF):
        """Random fuzzing of CAN IDs and data"""
        start_time = time.time()
        count = 0
        
        print(f"Starting random fuzzing for {duration} seconds...")
        
        try:
            while time.time() - start_time < duration:
                # Random arbitration ID
                arb_id = random.randint(min_id, max_id)
                
                # Random data length (0-8 bytes)
                dlc = random.randint(0, 8)
                
                # Random data
                data = bytes([random.randint(0, 255) for _ in range(dlc)])
                
                msg = can.Message(
                    arbitration_id=arb_id,
                    data=data,
                    is_extended_id=False
                )
                
                self.bus.send(msg)
                count += 1
                
                # Small delay to avoid overwhelming the bus
                time.sleep(0.01)
        
        except KeyboardInterrupt:
            print("\nFuzzing stopped")
        finally:
            print(f"Sent {count} fuzzed messages")
            self.bus.shutdown()
    
    def fuzz_targeted(self, target_id, num_iterations=1000):
        """Targeted fuzzing of specific CAN ID"""
        print(f"Fuzzing ID {hex(target_id)} with {num_iterations} iterations")
        
        for i in range(num_iterations):
            dlc = random.randint(0, 8)
            data = bytes([random.randint(0, 255) for _ in range(dlc)])
            
            msg = can.Message(arbitration_id=target_id, data=data)
            self.bus.send(msg)
            
            if i % 100 == 0:
                print(f"Progress: {i}/{num_iterations}")
            
            time.sleep(0.005)
        
        self.bus.shutdown()
    
    def fuzz_incremental(self, target_id, base_data):
        """Incremental fuzzing - modify each byte sequentially"""
        print(f"Incremental fuzzing of ID {hex(target_id)}")
        
        for byte_pos in range(len(base_data)):
            for value in range(256):
                data = bytearray(base_data)
                data[byte_pos] = value
                
                msg = can.Message(arbitration_id=target_id, data=bytes(data))
                self.bus.send(msg)
                time.sleep(0.01)
        
        self.bus.shutdown()

# Usage examples
fuzzer = CANFuzzer('vcan0')

# Random fuzzing
# fuzzer.fuzz_random(duration=30)

# Targeted fuzzing
# fuzzer.fuzz_targeted(0x123, num_iterations=500)

# Incremental fuzzing
# fuzzer.fuzz_incremental(0x456, bytes([0x00, 0x00, 0x00, 0x00]))
```

### 4. Protocol Analysis

Analyze and decode CAN messages:

```python
import can
from collections import defaultdict
import statistics

class CANAnalyzer:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.messages = defaultdict(list)
    
    def capture_and_analyze(self, duration=30):
        """Capture traffic and perform statistical analysis"""
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        start_time = time.time()
        
        print(f"Capturing for {duration} seconds...")
        
        try:
            while time.time() - start_time < duration:
                msg = bus.recv(timeout=1.0)
                if msg:
                    self.messages[msg.arbitration_id].append({
                        'timestamp': msg.timestamp,
                        'data': msg.data,
                        'dlc': msg.dlc
                    })
        except KeyboardInterrupt:
            print("\nCapture interrupted")
        finally:
            bus.shutdown()
        
        self.analyze_traffic()
    
    def analyze_traffic(self):
        """Analyze captured traffic"""
        print("\n=== Traffic Analysis ===")
        print(f"Unique CAN IDs: {len(self.messages)}")
        
        for arb_id, msgs in sorted(self.messages.items()):
            print(f"\nID: {hex(arb_id)}")
            print(f"  Message count: {len(msgs)}")
            
            # Calculate frequency
            if len(msgs) > 1:
                timestamps = [m['timestamp'] for m in msgs]
                intervals = [timestamps[i+1] - timestamps[i] 
                           for i in range(len(timestamps)-1)]
                if intervals:
                    avg_interval = statistics.mean(intervals)
                    frequency = 1 / avg_interval if avg_interval > 0 else 0
                    print(f"  Frequency: ~{frequency:.2f} Hz")
            
            # Show data patterns
            data_samples = [m['data'].hex() for m in msgs[:5]]
            print(f"  Sample data: {data_samples}")
    
    def detect_anomalies(self, baseline_duration=30, test_duration=10):
        """Detect anomalies by comparing to baseline"""
        print("Establishing baseline...")
        baseline_ids = set()
        
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        start_time = time.time()
        
        # Capture baseline
        while time.time() - start_time < baseline_duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                baseline_ids.add(msg.arbitration_id)
        
        print(f"Baseline: {len(baseline_ids)} unique IDs")
        print("\nMonitoring for anomalies...")
        
        # Monitor for new IDs
        start_time = time.time()
        anomalies = []
        
        while time.time() - start_time < test_duration:
            msg = bus.recv(timeout=1.0)
            if msg and msg.arbitration_id not in baseline_ids:
                anomaly = {
                    'id': hex(msg.arbitration_id),
                    'data': msg.data.hex(),
                    'timestamp': msg.timestamp
                }
                anomalies.append(anomaly)
                print(f"ANOMALY: New ID {hex(msg.arbitration_id)} - {msg.data.hex()}")
        
        bus.shutdown()
        return anomalies

# Usage
analyzer = CANAnalyzer('vcan0')
analyzer.capture_and_analyze(duration=20)

# Detect anomalies
# analyzer.detect_anomalies(baseline_duration=30, test_duration=10)
```

### 5. Replay Attack Testing

Capture and replay CAN messages:

```python
import can
import time
import pickle

class CANReplay:
    def __init__(self, interface='vcan0'):
        self.interface = interface
    
    def capture_session(self, duration=30, filename='capture.pkl'):
        """Capture a session for replay"""
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        captured = []
        start_time = time.time()
        
        print(f"Capturing session for {duration} seconds...")
        
        try:
            first_timestamp = None
            while time.time() - start_time < duration:
                msg = bus.recv(timeout=1.0)
                if msg:
                    if first_timestamp is None:
                        first_timestamp = msg.timestamp
                    
                    captured.append({
                        'arbitration_id': msg.arbitration_id,
                        'data': msg.data,
                        'relative_time': msg.timestamp - first_timestamp,
                        'is_extended_id': msg.is_extended_id
                    })
        except KeyboardInterrupt:
            print("\nCapture stopped")
        finally:
            bus.shutdown()
        
        # Save to file
        with open(filename, 'wb') as f:
            pickle.dump(captured, f)
        
        print(f"Captured {len(captured)} messages to {filename}")
        return captured
    
    def replay_session(self, filename='capture.pkl', speed=1.0):
        """Replay captured session"""
        # Load captured data
        with open(filename, 'rb') as f:
            captured = pickle.load(f)
        
        print(f"Replaying {len(captured)} messages at {speed}x speed...")
        
        bus = can.interface.Bus(channel=self.interface, bustype='socketcan')
        start_time = time.time()
        
        try:
            for i, msg_data in enumerate(captured):
                # Wait for correct timing
                target_time = msg_data['relative_time'] / speed
                while time.time() - start_time < target_time:
                    time.sleep(0.001)
                
                # Send message
                msg = can.Message(
                    arbitration_id=msg_data['arbitration_id'],
                    data=msg_data['data'],
                    is_extended_id=msg_data['is_extended_id']
                )
                bus.send(msg)
                
                if (i + 1) % 100 == 0:
                    print(f"Replayed {i + 1}/{len(captured)} messages")
        
        except KeyboardInterrupt:
            print("\nReplay stopped")
        finally:
            bus.shutdown()
        
        print("Replay complete")

# Usage
replayer = CANReplay('vcan0')

# Capture a session
# replayer.capture_session(duration=20, filename='test_session.pkl')

# Replay the session
# replayer.replay_session(filename='test_session.pkl', speed=1.0)
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=vcan0

# Set logging level
export S800_LOG_LEVEL=DEBUG

# Set output directory for captures
export S800_OUTPUT_DIR=./captures
```

### Configuration File

Create `s800_config.json`:

```json
{
  "interfaces": {
    "primary": "vcan0",
    "secondary": "can0"
  },
  "capture": {
    "auto_save": true,
    "output_dir": "./captures",
    "format": "pickle"
  },
  "fuzzing": {
    "delay_ms": 10,
    "max_iterations": 10000,
    "target_ids": ["0x123", "0x456", "0x789"]
  },
  "analysis": {
    "anomaly_threshold": 0.95,
    "baseline_duration": 30
  }
}
```

## Common Patterns

### Security Assessment Workflow

```python
import os

# Complete security assessment
def perform_security_assessment(interface='vcan0'):
    """Full vehicle network security assessment"""
    
    # Step 1: Baseline capture
    print("=== Phase 1: Baseline Capture ===")
    analyzer = CANAnalyzer(interface)
    analyzer.capture_and_analyze(duration=60)
    
    # Step 2: ID enumeration
    print("\n=== Phase 2: ID Enumeration ===")
    scanner = scan_can_bus(interface, duration=30)
    
    # Step 3: Anomaly detection
    print("\n=== Phase 3: Anomaly Detection ===")
    anomalies = analyzer.detect_anomalies(
        baseline_duration=30,
        test_duration=30
    )
    
    # Step 4: Targeted fuzzing of discovered IDs
    print("\n=== Phase 4: Fuzzing ===")
    fuzzer = CANFuzzer(interface)
    for msg in scanner[:5]:  # Test first 5 IDs
        arb_id = int(msg['arbitration_id'], 16)
        print(f"Fuzzing ID {hex(arb_id)}")
        fuzzer.fuzz_targeted(arb_id, num_iterations=100)
    
    # Step 5: Generate report
    print("\n=== Assessment Complete ===")
    print(f"Total unique IDs found: {len(scanner)}")
    print(f"Anomalies detected: {len(anomalies)}")

# Run assessment
# perform_security_assessment('vcan0')
```

## Troubleshooting

### CAN Interface Not Available

```bash
# Check if interface exists
ip link show vcan0

# Recreate virtual interface
sudo ip link delete vcan0
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# For physical interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

### Permission Errors

```bash
# Add user to dialout group for serial access
sudo usermod -a -G dialout $USER

# Set permissions for CAN devices
sudo chmod 666 /dev/ttyUSB0
```

### Python CAN Errors

```python
# Check available interfaces
import can
print(can.detect_available_configs())

# Test connection
try:
    bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
    print("Connection successful")
    bus.shutdown()
except Exception as e:
    print(f"Error: {e}")
```

### No Messages Received

```bash
# Generate test traffic on vcan0
cangen vcan0 -g 100 -I 123 -L 8

# Monitor with candump
candump vcan0

# Check kernel modules
lsmod | grep can
```

## Safety and Legal Considerations

**WARNING**: Unauthorized testing of vehicle systems is illegal and dangerous.

- Only test on vehicles you own or have explicit permission to test
- Never test on public roads or operational vehicles
- Use isolated test environments whenever possible
- Understand that manipulating vehicle networks can cause safety issues
- Follow all applicable laws and regulations
- Consider using vehicle network simulators for initial testing

## Additional Resources

- Use `candump` for quick traffic monitoring: `candump vcan0`
- Use `cansend` for manual message injection: `cansend vcan0 123#1122334455667788`
- Enable CAN logging: `candump -l vcan0`
- Analyze with Wireshark (with SocketCAN plugin)
