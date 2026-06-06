---
name: s800-vehicle-network-security-testing
description: Test and analyze vehicle network security vulnerabilities, including CAN bus fuzzing, replay attacks, and protocol analysis for automotive systems.
triggers:
  - "test vehicle network security"
  - "fuzz CAN bus messages"
  - "analyze automotive network traffic"
  - "perform vehicle security assessment"
  - "replay CAN bus attacks"
  - "scan vehicle ECU vulnerabilities"
  - "test automotive security protocols"
  - "vehicle penetration testing"
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive framework for testing and analyzing security vulnerabilities in vehicle networks, particularly CAN (Controller Area Network) bus systems. It provides tools for traffic analysis, fuzzing, replay attacks, and ECU (Electronic Control Unit) vulnerability assessment in automotive environments.

**Key Capabilities:**
- CAN bus traffic capture and analysis
- Protocol fuzzing for automotive networks
- Replay attack simulation
- ECU vulnerability scanning
- Message injection and manipulation
- Real-time network monitoring
- Security assessment reporting

## Installation

### Prerequisites

- Python 3.7+
- SocketCAN support (Linux kernel 2.6.25+)
- CAN interface hardware (USB-to-CAN adapter recommended)
- Root/administrator privileges for network access

### Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Set up CAN interface (Linux)
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Create virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For physical testing with real hardware:

```bash
# Configure physical CAN interface (e.g., can0)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Features

### 1. CAN Bus Traffic Capture

Capture and log CAN bus messages for analysis:

```python
import can
from datetime import datetime

def capture_can_traffic(interface='vcan0', duration=60):
    """Capture CAN traffic for specified duration"""
    bus = can.interface.Bus(interface, bustype='socketcan')
    messages = []
    
    start_time = datetime.now()
    print(f"[*] Capturing traffic on {interface}...")
    
    try:
        while (datetime.now() - start_time).seconds < duration:
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
        print("\n[!] Capture stopped by user")
    finally:
        bus.shutdown()
    
    return messages

# Usage
traffic_log = capture_can_traffic(interface='vcan0', duration=30)
```

### 2. CAN Message Injection

Send crafted CAN messages for testing:

```python
import can
import time

def inject_can_message(interface='vcan0', arb_id=0x123, data=None):
    """Inject a CAN message onto the bus"""
    if data is None:
        data = [0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07]
    
    bus = can.interface.Bus(interface, bustype='socketcan')
    
    try:
        msg = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        bus.send(msg)
        print(f"[+] Sent: ID={hex(arb_id)} Data={bytes(data).hex()}")
    except can.CanError as e:
        print(f"[-] Error sending message: {e}")
    finally:
        bus.shutdown()

# Example: Send diagnostic request
inject_can_message(
    interface='vcan0',
    arb_id=0x7DF,  # Diagnostic request ID
    data=[0x02, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
)
```

### 3. Replay Attack Simulation

Capture and replay CAN messages:

```python
import can
import time
import pickle

class ReplayAttack:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.captured_messages = []
    
    def capture(self, duration=60):
        """Capture messages for replay"""
        bus = can.interface.Bus(self.interface, bustype='socketcan')
        start_time = time.time()
        
        print(f"[*] Capturing for {duration} seconds...")
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                self.captured_messages.append({
                    'id': msg.arbitration_id,
                    'data': list(msg.data),
                    'timestamp': msg.timestamp
                })
        
        bus.shutdown()
        print(f"[+] Captured {len(self.captured_messages)} messages")
    
    def replay(self, speed_multiplier=1.0):
        """Replay captured messages"""
        if not self.captured_messages:
            print("[-] No messages to replay")
            return
        
        bus = can.interface.Bus(self.interface, bustype='socketcan')
        base_time = self.captured_messages[0]['timestamp']
        
        print(f"[*] Replaying {len(self.captured_messages)} messages...")
        for msg_data in self.captured_messages:
            # Calculate delay
            delay = (msg_data['timestamp'] - base_time) / speed_multiplier
            time.sleep(max(0, delay))
            
            msg = can.Message(
                arbitration_id=msg_data['id'],
                data=msg_data['data']
            )
            bus.send(msg)
        
        bus.shutdown()
        print("[+] Replay complete")
    
    def save(self, filename):
        """Save captured messages"""
        with open(filename, 'wb') as f:
            pickle.dump(self.captured_messages, f)
    
    def load(self, filename):
        """Load captured messages"""
        with open(filename, 'rb') as f:
            self.captured_messages = pickle.load(f)

# Usage
replay = ReplayAttack(interface='vcan0')
replay.capture(duration=30)
replay.save('captured_traffic.pkl')
replay.replay(speed_multiplier=1.0)
```

### 4. CAN Bus Fuzzing

Fuzz CAN messages to discover vulnerabilities:

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.bus = can.interface.Bus(interface, bustype='socketcan')
    
    def fuzz_random(self, count=1000, delay=0.01):
        """Send random CAN messages"""
        print(f"[*] Fuzzing with {count} random messages...")
        
        for i in range(count):
            arb_id = random.randint(0x000, 0x7FF)
            data = [random.randint(0, 255) for _ in range(8)]
            
            msg = can.Message(
                arbitration_id=arb_id,
                data=data
            )
            
            try:
                self.bus.send(msg)
                if i % 100 == 0:
                    print(f"[*] Sent {i}/{count} messages")
            except can.CanError as e:
                print(f"[-] Error: {e}")
            
            time.sleep(delay)
    
    def fuzz_id_range(self, start_id=0x000, end_id=0x7FF, data=None):
        """Fuzz by scanning ID range"""
        if data is None:
            data = [0x00] * 8
        
        print(f"[*] Fuzzing IDs from {hex(start_id)} to {hex(end_id)}...")
        
        for arb_id in range(start_id, end_id + 1):
            msg = can.Message(arbitration_id=arb_id, data=data)
            self.bus.send(msg)
            time.sleep(0.01)
    
    def fuzz_data_field(self, arb_id, byte_position=0, iterations=256):
        """Fuzz specific data byte of a message"""
        print(f"[*] Fuzzing byte {byte_position} of ID {hex(arb_id)}...")
        
        base_data = [0x00] * 8
        
        for value in range(iterations):
            base_data[byte_position] = value
            msg = can.Message(arbitration_id=arb_id, data=base_data)
            self.bus.send(msg)
            time.sleep(0.01)
    
    def cleanup(self):
        self.bus.shutdown()

# Usage
fuzzer = CANFuzzer(interface='vcan0')

# Random fuzzing
fuzzer.fuzz_random(count=500, delay=0.01)

# ID range fuzzing
fuzzer.fuzz_id_range(start_id=0x100, end_id=0x200)

# Targeted data fuzzing
fuzzer.fuzz_data_field(arb_id=0x123, byte_position=0)

fuzzer.cleanup()
```

### 5. ECU Fingerprinting

Identify and enumerate ECUs on the network:

```python
import can
import time

class ECUScanner:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.discovered_ecus = set()
    
    def passive_scan(self, duration=60):
        """Passively listen for ECU traffic"""
        bus = can.interface.Bus(self.interface, bustype='socketcan')
        start_time = time.time()
        
        print(f"[*] Passive scanning for {duration} seconds...")
        
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                self.discovered_ecus.add(msg.arbitration_id)
        
        bus.shutdown()
        print(f"[+] Discovered {len(self.discovered_ecus)} unique IDs")
        return sorted(self.discovered_ecus)
    
    def active_scan(self, start_id=0x000, end_id=0x7FF):
        """Actively probe for ECU responses"""
        bus = can.interface.Bus(self.interface, bustype='socketcan')
        responding_ids = []
        
        print(f"[*] Active scanning from {hex(start_id)} to {hex(end_id)}...")
        
        for arb_id in range(start_id, end_id + 1):
            # Send diagnostic request
            msg = can.Message(
                arbitration_id=arb_id,
                data=[0x02, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
            )
            bus.send(msg)
            
            # Listen for response
            response = bus.recv(timeout=0.1)
            if response:
                responding_ids.append(arb_id)
                print(f"[+] Response from {hex(arb_id)}")
        
        bus.shutdown()
        return responding_ids

# Usage
scanner = ECUScanner(interface='vcan0')
active_ecus = scanner.passive_scan(duration=30)
print(f"Active ECUs: {[hex(id) for id in active_ecus]}")
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=vcan0

# Set default bitrate
export S800_CAN_BITRATE=500000

# Enable verbose logging
export S800_VERBOSE=1

# Set output directory for logs
export S800_OUTPUT_DIR=/var/log/s800
```

### Configuration File (config.json)

```json
{
  "interface": "vcan0",
  "bitrate": 500000,
  "logging": {
    "enabled": true,
    "level": "INFO",
    "output_dir": "./logs"
  },
  "fuzzing": {
    "delay_ms": 10,
    "max_iterations": 10000
  },
  "capture": {
    "buffer_size": 10000,
    "auto_save": true
  }
}
```

## Common Patterns

### Pattern 1: Security Assessment Workflow

```python
import can
import time
import json

def vehicle_security_assessment(interface='vcan0'):
    """Complete security assessment workflow"""
    results = {
        'timestamp': time.time(),
        'interface': interface,
        'findings': []
    }
    
    # Step 1: Passive traffic analysis
    print("[1/4] Passive traffic analysis...")
    scanner = ECUScanner(interface)
    discovered_ids = scanner.passive_scan(duration=60)
    results['discovered_ids'] = [hex(id) for id in discovered_ids]
    
    # Step 2: Active ECU enumeration
    print("[2/4] Active ECU enumeration...")
    responding_ids = scanner.active_scan(start_id=0x000, end_id=0x7FF)
    results['responding_ids'] = [hex(id) for id in responding_ids]
    
    # Step 3: Targeted fuzzing
    print("[3/4] Targeted fuzzing on discovered ECUs...")
    fuzzer = CANFuzzer(interface)
    for ecu_id in responding_ids[:5]:  # Limit to first 5
        fuzzer.fuzz_data_field(arb_id=ecu_id, byte_position=0, iterations=100)
    fuzzer.cleanup()
    
    # Step 4: Generate report
    print("[4/4] Generating report...")
    with open('security_assessment.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    print("[+] Assessment complete. Results saved to security_assessment.json")
    return results
```

### Pattern 2: Real-time Monitoring with Alerts

```python
import can
import threading

class CANMonitor:
    def __init__(self, interface='vcan0'):
        self.interface = interface
        self.running = False
        self.alerts = []
    
    def start(self, alert_rules=None):
        """Start real-time monitoring"""
        if alert_rules is None:
            alert_rules = {
                'unexpected_ids': [0x666, 0x667],  # Suspicious IDs
                'data_patterns': [b'\xff\xff\xff\xff']  # Suspicious data
            }
        
        self.running = True
        bus = can.interface.Bus(self.interface, bustype='socketcan')
        
        print("[*] Monitoring started. Press Ctrl+C to stop.")
        
        try:
            while self.running:
                msg = bus.recv(timeout=1.0)
                if msg:
                    # Check alert rules
                    if msg.arbitration_id in alert_rules['unexpected_ids']:
                        alert = f"ALERT: Unexpected ID {hex(msg.arbitration_id)}"
                        print(f"[!] {alert}")
                        self.alerts.append(alert)
                    
                    for pattern in alert_rules['data_patterns']:
                        if pattern in bytes(msg.data):
                            alert = f"ALERT: Suspicious data pattern in {hex(msg.arbitration_id)}"
                            print(f"[!] {alert}")
                            self.alerts.append(alert)
        except KeyboardInterrupt:
            print("\n[*] Monitoring stopped")
        finally:
            bus.shutdown()
            self.running = False
    
    def stop(self):
        self.running = False

# Usage
monitor = CANMonitor(interface='vcan0')
monitor.start()
```

## Troubleshooting

### Issue: "Network is down" error

```bash
# Check interface status
ip link show vcan0

# Bring interface up
sudo ip link set up vcan0

# For physical interfaces, set bitrate first
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

### Issue: Permission denied

```bash
# Run with sudo or add user to appropriate groups
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

# Log out and back in for changes to take effect
```

### Issue: No messages received

```python
# Verify bus configuration
import can

bus = can.interface.Bus('vcan0', bustype='socketcan')
print(f"Bus state: {bus.state}")
print(f"Channel info: {bus.channel_info}")

# Check for messages with longer timeout
msg = bus.recv(timeout=5.0)
if msg is None:
    print("No traffic detected - verify message source")
```

### Issue: Message buffer overflow

```python
# Use buffered reader with larger buffer
bus = can.interface.Bus('vcan0', bustype='socketcan')
reader = can.BufferedReader()
notifier = can.Notifier(bus, [reader], timeout=0.1)

# Process messages in batches
while True:
    msg = reader.get_message(timeout=1.0)
    if msg:
        # Process message
        pass
```

## Safety Warnings

⚠️ **IMPORTANT**: This framework is for authorized security testing only.

- Never test on vehicles in operation or public roads
- Always use isolated test environments
- Obtain proper authorization before testing
- Be aware that fuzzing can cause unexpected ECU behavior
- Have kill switches and safety measures in place
- Keep detailed logs of all testing activities

## Additional Resources

- CAN Bus specification: ISO 11898
- UDS protocol: ISO 14229
- Automotive security standards: ISO/SAE 21434
- SocketCAN documentation: https://www.kernel.org/doc/Documentation/networking/can.txt
