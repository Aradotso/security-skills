---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for automotive CAN bus and protocol analysis
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive security testing
  - use S800 framework
  - test vehicle communication protocols
  - scan automotive network vulnerabilities
  - fuzzing CAN messages
  - vehicle penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a specialized framework for testing security vulnerabilities in vehicle networks, particularly focusing on CAN (Controller Area Network) bus and other automotive communication protocols. It provides tools for traffic analysis, fuzzing, packet injection, and vulnerability assessment of in-vehicle networks.

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can cantools socketcan pyserial

# For Linux CAN support
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Clone and Setup

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install requirements
pip install -r requirements.txt

# Setup virtual CAN for testing (Linux)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For physical CAN interfaces:
```bash
# Configure real CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Components

### CAN Bus Communication

```python
import can

# Initialize CAN bus connection
def setup_can_interface(channel='vcan0', bustype='socketcan', bitrate=500000):
    """Setup CAN bus interface"""
    bus = can.interface.Bus(
        channel=channel,
        bustype=bustype,
        bitrate=bitrate
    )
    return bus

# Send CAN message
def send_can_message(bus, arbitration_id, data):
    """Send a CAN message"""
    msg = can.Message(
        arbitration_id=arbitration_id,
        data=data,
        is_extended_id=False
    )
    bus.send(msg)
    print(f"Sent: ID={hex(arbitration_id)}, Data={data.hex()}")

# Example usage
bus = setup_can_interface('vcan0')
send_can_message(bus, 0x123, bytes([0x01, 0x02, 0x03, 0x04]))
```

### Traffic Sniffing

```python
import can
import time

def sniff_can_traffic(channel='vcan0', duration=10, filter_id=None):
    """Capture and analyze CAN traffic"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    print(f"Sniffing CAN traffic on {channel} for {duration}s...")
    start_time = time.time()
    messages = []
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            if filter_id is None or msg.arbitration_id == filter_id:
                messages.append(msg)
                print(f"Time: {msg.timestamp:.6f} ID: {hex(msg.arbitration_id)} "
                      f"Data: {msg.data.hex()} DLC: {msg.dlc}")
    
    bus.shutdown()
    return messages

# Capture all traffic
messages = sniff_can_traffic('vcan0', duration=30)

# Capture specific ID
messages = sniff_can_traffic('vcan0', duration=30, filter_id=0x123)
```

### CAN Fuzzing

```python
import can
import random
import time

def fuzz_can_messages(bus, target_id, num_messages=100, delay=0.01):
    """Fuzz CAN messages with random payloads"""
    print(f"Fuzzing CAN ID {hex(target_id)} with {num_messages} messages...")
    
    for i in range(num_messages):
        # Generate random data length (0-8 bytes for standard CAN)
        dlc = random.randint(0, 8)
        data = bytes([random.randint(0, 255) for _ in range(dlc)])
        
        msg = can.Message(
            arbitration_id=target_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[{i+1}/{num_messages}] Sent: {data.hex()}")
        except Exception as e:
            print(f"Error sending message: {e}")
        
        time.sleep(delay)

# Example usage
bus = setup_can_interface('vcan0')
fuzz_can_messages(bus, 0x7DF, num_messages=50)
```

### Replay Attack

```python
import can
import time

def replay_attack(capture_file, channel='vcan0', speed_multiplier=1.0):
    """Replay captured CAN messages"""
    bus = can.interface.Bus(channel=channel, bustype='socketcan')
    
    # Load messages from log file
    messages = []
    with open(capture_file, 'r') as f:
        for line in f:
            # Parse log format: timestamp, id, data
            parts = line.strip().split(',')
            if len(parts) >= 3:
                timestamp = float(parts[0])
                arb_id = int(parts[1], 16)
                data = bytes.fromhex(parts[2])
                messages.append((timestamp, arb_id, data))
    
    print(f"Replaying {len(messages)} messages...")
    
    # Replay with timing
    start_time = time.time()
    first_msg_time = messages[0][0] if messages else 0
    
    for msg_time, arb_id, data in messages:
        # Calculate delay
        target_time = (msg_time - first_msg_time) / speed_multiplier
        current_time = time.time() - start_time
        
        if target_time > current_time:
            time.sleep(target_time - current_time)
        
        msg = can.Message(arbitration_id=arb_id, data=data)
        bus.send(msg)
        print(f"Replayed: ID={hex(arb_id)}, Data={data.hex()}")
    
    bus.shutdown()

# Replay at normal speed
replay_attack('captured_traffic.log', 'vcan0', speed_multiplier=1.0)

# Replay at 2x speed
replay_attack('captured_traffic.log', 'vcan0', speed_multiplier=2.0)
```

### UDS (Unified Diagnostic Services) Testing

```python
import can
import time

class UDSScanner:
    """Scanner for UDS diagnostic services"""
    
    def __init__(self, bus, target_id=0x7DF, response_id=0x7E8):
        self.bus = bus
        self.target_id = target_id
        self.response_id = response_id
    
    def send_uds_request(self, service, data=[]):
        """Send UDS request and wait for response"""
        payload = [service] + data
        msg = can.Message(
            arbitration_id=self.target_id,
            data=payload,
            is_extended_id=False
        )
        
        self.bus.send(msg)
        
        # Wait for response
        start_time = time.time()
        while time.time() - start_time < 1.0:
            response = self.bus.recv(timeout=0.1)
            if response and response.arbitration_id == self.response_id:
                return response.data
        
        return None
    
    def scan_services(self, service_range=range(0x00, 0x100)):
        """Scan for available UDS services"""
        print("Scanning UDS services...")
        available = []
        
        for service in service_range:
            response = self.send_uds_request(service)
            
            if response and len(response) > 0:
                if response[0] != 0x7F:  # Not a negative response
                    available.append(service)
                    print(f"Service {hex(service)}: Available - {response.hex()}")
            
            time.sleep(0.01)
        
        return available

# Example usage
bus = setup_can_interface('vcan0')
scanner = UDSScanner(bus, target_id=0x7DF, response_id=0x7E8)
services = scanner.scan_services()
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=vcan0

# Set CAN bitrate
export S800_CAN_BITRATE=500000

# Enable verbose logging
export S800_VERBOSE=1

# Set log output directory
export S800_LOG_DIR=/var/log/s800
```

### Configuration File (config.json)

```json
{
  "can_interface": "vcan0",
  "bitrate": 500000,
  "logging": {
    "enabled": true,
    "level": "INFO",
    "output_dir": "./logs"
  },
  "fuzzing": {
    "default_iterations": 1000,
    "delay_ms": 10,
    "random_seed": null
  },
  "target_ids": {
    "engine_ecu": "0x7E0",
    "transmission": "0x7E1",
    "abs": "0x7E2"
  }
}
```

### Load Configuration

```python
import json
import os

def load_config(config_file='config.json'):
    """Load configuration from file"""
    with open(config_file, 'r') as f:
        config = json.load(f)
    
    # Override with environment variables
    config['can_interface'] = os.getenv('S800_CAN_INTERFACE', config['can_interface'])
    config['bitrate'] = int(os.getenv('S800_CAN_BITRATE', config['bitrate']))
    
    return config

config = load_config()
bus = setup_can_interface(
    channel=config['can_interface'],
    bitrate=config['bitrate']
)
```

## Common Testing Patterns

### ECU Enumeration

```python
def enumerate_ecus(bus, id_range=range(0x7E0, 0x7F0)):
    """Enumerate ECUs by sending diagnostic requests"""
    active_ecus = []
    
    for ecu_id in id_range:
        # Send read DTC request (service 0x19)
        msg = can.Message(
            arbitration_id=ecu_id,
            data=[0x19, 0x02, 0xFF],
            is_extended_id=False
        )
        
        bus.send(msg)
        
        # Listen for response
        response = bus.recv(timeout=0.5)
        if response and response.arbitration_id == (ecu_id + 0x08):
            active_ecus.append(ecu_id)
            print(f"Found ECU at ID: {hex(ecu_id)}")
    
    return active_ecus
```

### Session Management

```python
def start_diagnostic_session(bus, target_id, session_type=0x03):
    """Start diagnostic session (extended, programming, etc.)"""
    # Service 0x10 - Diagnostic Session Control
    msg = can.Message(
        arbitration_id=target_id,
        data=[0x10, session_type],
        is_extended_id=False
    )
    
    bus.send(msg)
    response = bus.recv(timeout=1.0)
    
    if response and response.data[0] == 0x50:
        print(f"Diagnostic session started: {hex(session_type)}")
        return True
    
    return False
```

## Troubleshooting

### CAN Interface Not Found

```python
import can

def check_can_interfaces():
    """Check available CAN interfaces"""
    try:
        interfaces = can.detect_available_configs()
        print("Available interfaces:")
        for interface in interfaces:
            print(f"  - {interface}")
        return interfaces
    except Exception as e:
        print(f"Error detecting interfaces: {e}")
        return []

# If no interfaces found, setup virtual CAN
# sudo ip link add dev vcan0 type vcan
# sudo ip link set up vcan0
```

### Permission Denied

```bash
# Add user to dialout group for serial devices
sudo usermod -a -G dialout $USER

# Set CAN permissions
sudo chmod 666 /dev/ttyUSB0
```

### No Response from ECU

```python
def diagnose_connection(bus, target_id):
    """Test connectivity to target ECU"""
    # Try tester present service (0x3E)
    msg = can.Message(
        arbitration_id=target_id,
        data=[0x3E, 0x00],
        is_extended_id=False
    )
    
    print(f"Testing connection to {hex(target_id)}...")
    bus.send(msg)
    
    response = bus.recv(timeout=2.0)
    if response:
        print(f"Response received: {response.data.hex()}")
        return True
    else:
        print("No response - check wiring, bitrate, and ECU power")
        return False
```

### Message Timing Issues

```python
import can
import threading

def continuous_tester_present(bus, target_id, interval=2.0):
    """Keep diagnostic session alive with periodic tester present"""
    stop_event = threading.Event()
    
    def send_tester_present():
        while not stop_event.is_set():
            msg = can.Message(
                arbitration_id=target_id,
                data=[0x3E, 0x80],  # Suppress positive response
                is_extended_id=False
            )
            bus.send(msg)
            time.sleep(interval)
    
    thread = threading.Thread(target=send_tester_present)
    thread.start()
    
    return stop_event  # Call stop_event.set() to stop
```

## Safety Warnings

**CRITICAL**: Vehicle network testing can cause unintended behavior. Always:
- Test on isolated bench setups when possible
- Never test on vehicles in operation or on public roads
- Understand the implications of messages before sending
- Have kill switches and safety procedures in place
- Comply with local laws and regulations
