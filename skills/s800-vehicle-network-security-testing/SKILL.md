---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, focusing on CAN bus and automotive protocol vulnerabilities
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - scan automotive network protocols
  - perform vehicle penetration testing
  - test car ECU security
  - fuzzing automotive networks
  - vehicle security assessment
  - automotive protocol testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive framework for testing and assessing security vulnerabilities in vehicle networks, particularly focusing on CAN (Controller Area Network) bus systems and other automotive communication protocols. It enables security researchers and automotive engineers to perform penetration testing, fuzzing, and vulnerability analysis on automotive electronic control units (ECUs) and network communications.

## Installation

### Prerequisites

```bash
# Required dependencies for CAN interface support
sudo apt-get update
sudo apt-get install can-utils python3-can python3-pip

# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Setup Virtual CAN Interface (for testing)

```bash
# Create virtual CAN interface
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Verify interface
ifconfig vcan0
```

### Framework Installation

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework
pip3 install -r requirements.txt
```

## Configuration

### Basic Configuration File (config.json)

```json
{
  "interface": "can0",
  "bitrate": 500000,
  "log_level": "INFO",
  "output_dir": "./results",
  "timeout": 30,
  "retry_count": 3,
  "protocols": {
    "can": {
      "enabled": true,
      "extended_id": false
    },
    "uds": {
      "enabled": true,
      "timeout": 5
    },
    "obd2": {
      "enabled": true,
      "mode": "auto"
    }
  }
}
```

### Hardware Interface Setup

```python
# Connect to physical CAN interface
import can

def setup_can_interface(interface="can0", bitrate=500000):
    """Configure CAN interface for vehicle testing"""
    bus = can.interface.Bus(
        channel=interface,
        bustype='socketcan',
        bitrate=bitrate
    )
    return bus

# Example usage
bus = setup_can_interface()
```

## Core Features

### 1. CAN Bus Sniffing

```python
import can
import time

def sniff_can_traffic(interface="vcan0", duration=10):
    """Capture and analyze CAN bus traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Sniffing CAN traffic on {interface} for {duration} seconds...")
    
    messages = []
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append(msg)
            print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
    
    bus.shutdown()
    return messages

# Run sniffer
captured = sniff_can_traffic(duration=30)
```

### 2. CAN Message Injection

```python
import can

def inject_can_message(interface, arbitration_id, data):
    """Send crafted CAN messages to the bus"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    message = can.Message(
        arbitration_id=arbitration_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(message)
        print(f"Sent: ID=0x{arbitration_id:03X}, Data={data.hex()}")
        return True
    except can.CanError as e:
        print(f"Error sending message: {e}")
        return False
    finally:
        bus.shutdown()

# Example: Send diagnostic request
inject_can_message("can0", 0x7DF, bytes([0x02, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]))
```

### 3. Fuzzing Automotive Protocols

```python
import can
import random
import time

def fuzz_can_ids(interface, id_range=(0x000, 0x7FF), count=1000):
    """Fuzz CAN arbitration IDs with random data"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    results = []
    
    for i in range(count):
        arb_id = random.randint(id_range[0], id_range[1])
        data = bytes([random.randint(0, 255) for _ in range(8)])
        
        message = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(message)
            results.append({
                "id": arb_id,
                "data": data.hex(),
                "status": "sent"
            })
            time.sleep(0.01)  # Prevent bus flooding
        except Exception as e:
            results.append({
                "id": arb_id,
                "error": str(e)
            })
    
    bus.shutdown()
    return results

# Start fuzzing campaign
fuzz_results = fuzz_can_ids("vcan0", count=500)
```

### 4. UDS (Unified Diagnostic Services) Testing

```python
import can
import time

class UDSScanner:
    """Scan for UDS diagnostic services"""
    
    def __init__(self, interface, request_id=0x7DF, response_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.request_id = request_id
        self.response_id = response_id
    
    def send_uds_request(self, service, data=[]):
        """Send UDS service request"""
        payload = [len(data) + 1, service] + data + [0] * (8 - len(data) - 2)
        
        msg = can.Message(
            arbitration_id=self.request_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
    
    def read_response(self, timeout=2.0):
        """Read UDS response"""
        start = time.time()
        
        while time.time() - start < timeout:
            msg = self.bus.recv(timeout=0.1)
            if msg and msg.arbitration_id == self.response_id:
                return msg.data
        
        return None
    
    def scan_services(self, services=[0x10, 0x11, 0x22, 0x27, 0x3E]):
        """Scan for available UDS services"""
        results = {}
        
        for service in services:
            self.send_uds_request(service)
            response = self.read_response()
            
            if response:
                results[f"0x{service:02X}"] = {
                    "available": True,
                    "response": response.hex()
                }
            else:
                results[f"0x{service:02X}"] = {"available": False}
        
        return results
    
    def close(self):
        self.bus.shutdown()

# Scan for diagnostic services
scanner = UDSScanner("can0")
services = scanner.scan_services()
print(f"Available services: {services}")
scanner.close()
```

### 5. Replay Attacks

```python
import can
import time
import json

def record_can_session(interface, duration, output_file):
    """Record CAN traffic for replay"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    messages = []
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append({
                "timestamp": time.time() - start_time,
                "id": msg.arbitration_id,
                "data": list(msg.data)
            })
    
    with open(output_file, 'w') as f:
        json.dump(messages, f, indent=2)
    
    bus.shutdown()
    return len(messages)

def replay_can_session(interface, input_file, speed=1.0):
    """Replay recorded CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    with open(input_file, 'r') as f:
        messages = json.load(f)
    
    start_time = time.time()
    
    for msg_data in messages:
        # Wait for correct timing
        target_time = msg_data["timestamp"] / speed
        while time.time() - start_time < target_time:
            time.sleep(0.001)
        
        msg = can.Message(
            arbitration_id=msg_data["id"],
            data=bytes(msg_data["data"]),
            is_extended_id=False
        )
        
        bus.send(msg)
    
    bus.shutdown()

# Record and replay
record_can_session("can0", duration=60, output_file="session.json")
replay_can_session("can0", "session.json", speed=1.0)
```

## Common Testing Patterns

### ECU Identification

```python
def identify_ecus(interface):
    """Identify active ECUs on the network"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    detected_ids = set()
    
    # Monitor traffic for 30 seconds
    start = time.time()
    while time.time() - start < 30:
        msg = bus.recv(timeout=1.0)
        if msg:
            detected_ids.add(msg.arbitration_id)
    
    bus.shutdown()
    
    print(f"Detected {len(detected_ids)} unique CAN IDs:")
    for can_id in sorted(detected_ids):
        print(f"  0x{can_id:03X}")
    
    return detected_ids
```

### Security Seed-Key Authentication Testing

```python
def test_security_access(interface, ecu_id=0x7E0):
    """Test security access (seed-key) mechanism"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Request seed (Service 0x27, SubFunction 0x01)
    seed_request = can.Message(
        arbitration_id=ecu_id,
        data=bytes([0x02, 0x27, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00])
    )
    
    bus.send(seed_request)
    time.sleep(0.1)
    
    response = bus.recv(timeout=2.0)
    if response:
        print(f"Seed response: {response.data.hex()}")
        # Extract seed and compute key (implementation-specific)
    
    bus.shutdown()
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Bring up interface
sudo ip link set can0 up type can bitrate 500000

# Check kernel modules
lsmod | grep can
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Set interface permissions
sudo chmod 666 /dev/can0
```

### No Traffic Detected

```python
# Verify interface is receiving
import can

bus = can.interface.Bus(channel='can0', bustype='socketcan')
print("Listening for 10 seconds...")

count = 0
for msg in bus:
    count += 1
    print(msg)
    if count >= 10:
        break

if count == 0:
    print("No traffic detected - check physical connections")
```

### Buffer Overruns

```python
# Set appropriate buffer sizes
import can

bus = can.interface.Bus(
    channel='can0',
    bustype='socketcan',
    receive_own_messages=False,
    can_filters=None  # No filtering
)
```

## Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE="can0"

# Set output directory
export S800_OUTPUT_DIR="./test_results"

# Enable debug logging
export S800_DEBUG="1"

# Set CAN bitrate
export S800_BITRATE="500000"
```

## Safety Considerations

**WARNING**: This framework is for authorized security testing only. Testing on vehicles:

- Should only be performed on test benches or with proper authorization
- Can affect vehicle safety systems
- May void warranties or violate regulations
- Requires understanding of automotive safety standards (ISO 26262)

Always disconnect critical safety systems before testing and never test on public roads.
