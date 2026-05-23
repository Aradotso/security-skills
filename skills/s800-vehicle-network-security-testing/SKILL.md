---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, including CAN bus, OBD-II, and automotive protocol fuzzing
triggers:
  - test vehicle network security
  - analyze CAN bus communication
  - fuzz automotive protocols
  - scan OBD-II vulnerabilities
  - test vehicle ECU security
  - simulate vehicle network attacks
  - audit automotive network security
  - test car controller security
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800-Vehicle-Network-Security-Testing-Framework is a comprehensive toolkit for testing and analyzing security vulnerabilities in vehicle networks. It supports testing CAN (Controller Area Network) bus communications, OBD-II protocols, ECU (Electronic Control Unit) security, and automotive network fuzzing. The framework enables security researchers and automotive engineers to identify vulnerabilities in vehicle communication systems.

**Key Capabilities:**
- CAN bus message capture and replay
- OBD-II protocol analysis and testing
- ECU security vulnerability scanning
- Automotive protocol fuzzing
- Network traffic simulation and injection
- Vehicle network monitoring and logging

## Installation

### Prerequisites

```bash
# Install required system dependencies
sudo apt-get update
sudo apt-get install -y python3 python3-pip can-utils

# Load kernel modules for CAN interface
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Set up virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For physical vehicle testing, you'll need:
- CAN bus interface (e.g., USB2CAN, CANable, Kvaser)
- OBD-II adapter compatible with SocketCAN
- Connection to vehicle's OBD-II port or CAN bus

```bash
# Configure physical CAN interface (example for slcan devices)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0

# Set CAN bitrate (common automotive rate: 500 kbps)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Configuration

### Configuration File Structure

Create a configuration file `config.json`:

```json
{
  "interface": "can0",
  "baudrate": 500000,
  "logging": {
    "enabled": true,
    "path": "./logs",
    "format": "csv"
  },
  "fuzzing": {
    "target_ids": ["0x7DF", "0x7E0", "0x7E8"],
    "delay_ms": 100,
    "iterations": 1000
  },
  "filtering": {
    "whitelist": [],
    "blacklist": ["0x000"]
  }
}
```

### Environment Variables

```bash
# Set CAN interface
export S800_CAN_INTERFACE=can0

# Set logging directory
export S800_LOG_DIR=/var/log/s800

# Set test mode (prevents actual vehicle commands)
export S800_TEST_MODE=1

# Set database path for known CAN IDs
export S800_DB_PATH=./vehicle_db.json
```

## Core Modules and Usage

### 1. CAN Bus Sniffer

Capture and analyze CAN bus traffic:

```python
#!/usr/bin/env python3
import can
import time

# Initialize CAN bus interface
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

# Capture CAN messages
def sniff_can_traffic(duration=10):
    """Capture CAN messages for specified duration"""
    messages = []
    start_time = time.time()
    
    print(f"[*] Sniffing CAN traffic for {duration} seconds...")
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg is not None:
            messages.append({
                'timestamp': msg.timestamp,
                'arbitration_id': hex(msg.arbitration_id),
                'data': msg.data.hex(),
                'dlc': msg.dlc
            })
            print(f"[+] ID: {hex(msg.arbitration_id)} Data: {msg.data.hex()}")
    
    return messages

# Run sniffer
captured = sniff_can_traffic(duration=30)
print(f"[*] Captured {len(captured)} messages")
```

### 2. CAN Message Replay

Replay captured CAN messages for testing:

```python
#!/usr/bin/env python3
import can
import json
import time

def replay_can_messages(log_file, interface='vcan0'):
    """Replay CAN messages from log file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    with open(log_file, 'r') as f:
        messages = json.load(f)
    
    print(f"[*] Replaying {len(messages)} messages...")
    
    for msg_data in messages:
        msg = can.Message(
            arbitration_id=int(msg_data['arbitration_id'], 16),
            data=bytes.fromhex(msg_data['data']),
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[+] Sent: {msg}")
            time.sleep(0.01)  # Small delay between messages
        except can.CanError as e:
            print(f"[-] Error sending message: {e}")
    
    bus.shutdown()

# Usage
replay_can_messages('captured_traffic.json', interface='can0')
```

### 3. OBD-II Scanner

Scan for supported OBD-II PIDs:

```python
#!/usr/bin/env python3
import can
import time

class OBD2Scanner:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.obd_request_id = 0x7DF  # Broadcast ID
        self.obd_response_id = 0x7E8  # ECU response ID
    
    def send_obd_request(self, mode, pid):
        """Send OBD-II request"""
        data = [mode, pid, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
        msg = can.Message(
            arbitration_id=self.obd_request_id,
            data=data,
            is_extended_id=False
        )
        self.bus.send(msg)
    
    def receive_response(self, timeout=1.0):
        """Receive OBD-II response"""
        msg = self.bus.recv(timeout=timeout)
        if msg and msg.arbitration_id == self.obd_response_id:
            return msg.data
        return None
    
    def scan_supported_pids(self):
        """Scan for supported PIDs in Mode 01"""
        supported_pids = []
        
        print("[*] Scanning for supported PIDs...")
        
        # Test common PIDs
        common_pids = [0x00, 0x01, 0x05, 0x0C, 0x0D, 0x0E, 0x0F, 
                       0x10, 0x11, 0x1C, 0x1F, 0x20, 0x21, 0x2F]
        
        for pid in common_pids:
            self.send_obd_request(mode=0x01, pid=pid)
            response = self.receive_response()
            
            if response:
                supported_pids.append({
                    'pid': hex(pid),
                    'response': response.hex()
                })
                print(f"[+] PID {hex(pid)}: Supported")
            
            time.sleep(0.1)
        
        return supported_pids
    
    def close(self):
        self.bus.shutdown()

# Usage
scanner = OBD2Scanner(interface='can0')
supported = scanner.scan_supported_pids()
print(f"[*] Found {len(supported)} supported PIDs")
scanner.close()
```

### 4. CAN Fuzzer

Fuzz CAN messages to discover vulnerabilities:

```python
#!/usr/bin/env python3
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0', target_ids=None):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.target_ids = target_ids or [0x7DF, 0x7E0, 0x7E8]
    
    def generate_random_payload(self, length=8):
        """Generate random CAN payload"""
        return [random.randint(0, 255) for _ in range(length)]
    
    def fuzz_sequential(self, iterations=1000, delay=0.01):
        """Sequential fuzzing of target IDs"""
        print(f"[*] Starting sequential fuzzing ({iterations} iterations)...")
        
        for i in range(iterations):
            for target_id in self.target_ids:
                payload = self.generate_random_payload()
                msg = can.Message(
                    arbitration_id=target_id,
                    data=payload,
                    is_extended_id=False
                )
                
                try:
                    self.bus.send(msg)
                    print(f"[{i}] Sent to {hex(target_id)}: {bytes(payload).hex()}")
                except can.CanError as e:
                    print(f"[-] Error: {e}")
                
                time.sleep(delay)
    
    def fuzz_random_ids(self, iterations=1000, delay=0.01):
        """Fuzz with random CAN IDs"""
        print(f"[*] Starting random ID fuzzing ({iterations} iterations)...")
        
        for i in range(iterations):
            random_id = random.randint(0x000, 0x7FF)
            payload = self.generate_random_payload()
            
            msg = can.Message(
                arbitration_id=random_id,
                data=payload,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[{i}] ID: {hex(random_id)} Data: {bytes(payload).hex()}")
            except can.CanError as e:
                print(f"[-] Error: {e}")
            
            time.sleep(delay)
    
    def close(self):
        self.bus.shutdown()

# Usage
fuzzer = CANFuzzer(interface='vcan0', target_ids=[0x7DF, 0x7E0])
fuzzer.fuzz_sequential(iterations=100, delay=0.05)
fuzzer.close()
```

### 5. ECU Identification

Identify and fingerprint ECUs on the network:

```python
#!/usr/bin/env python3
import can
import time

class ECUIdentifier:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.discovered_ecus = {}
    
    def scan_network(self, duration=30):
        """Scan network and identify active ECUs"""
        print(f"[*] Scanning network for {duration} seconds...")
        start_time = time.time()
        
        while time.time() - start_time < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                arb_id = msg.arbitration_id
                
                if arb_id not in self.discovered_ecus:
                    self.discovered_ecus[arb_id] = {
                        'first_seen': msg.timestamp,
                        'message_count': 1,
                        'data_samples': [msg.data.hex()]
                    }
                    print(f"[+] New ECU detected: {hex(arb_id)}")
                else:
                    self.discovered_ecus[arb_id]['message_count'] += 1
                    if len(self.discovered_ecus[arb_id]['data_samples']) < 5:
                        self.discovered_ecus[arb_id]['data_samples'].append(
                            msg.data.hex()
                        )
        
        return self.discovered_ecus
    
    def request_ecu_info(self, ecu_id):
        """Request diagnostic information from ECU"""
        # UDS (Unified Diagnostic Services) request
        msg = can.Message(
            arbitration_id=ecu_id,
            data=[0x02, 0x09, 0x02, 0x00, 0x00, 0x00, 0x00, 0x00],
            is_extended_id=False
        )
        
        self.bus.send(msg)
        response = self.bus.recv(timeout=1.0)
        
        if response:
            return response.data.hex()
        return None
    
    def close(self):
        self.bus.shutdown()

# Usage
identifier = ECUIdentifier(interface='can0')
ecus = identifier.scan_network(duration=60)
print(f"[*] Discovered {len(ecus)} ECUs")
for ecu_id, info in ecus.items():
    print(f"  {hex(ecu_id)}: {info['message_count']} messages")
identifier.close()
```

## Common Patterns

### Safe Testing Workflow

```python
#!/usr/bin/env python3
import os
import can
import json
from datetime import datetime

def safe_testing_wrapper(test_function, interface='vcan0'):
    """Wrapper for safe testing with logging"""
    # Check if in test mode
    test_mode = os.getenv('S800_TEST_MODE', '0') == '1'
    
    if not test_mode:
        response = input("[!] Not in test mode. Continue with live vehicle? (yes/no): ")
        if response.lower() != 'yes':
            print("[*] Test cancelled")
            return
    
    # Create log directory
    log_dir = os.getenv('S800_LOG_DIR', './logs')
    os.makedirs(log_dir, exist_ok=True)
    
    # Initialize logging
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    log_file = f"{log_dir}/test_{timestamp}.json"
    
    print(f"[*] Starting test (logging to {log_file})...")
    
    try:
        result = test_function(interface)
        
        # Save results
        with open(log_file, 'w') as f:
            json.dump(result, f, indent=2)
        
        print(f"[*] Test completed successfully")
        return result
    
    except Exception as e:
        print(f"[-] Test failed: {e}")
        with open(f"{log_dir}/error_{timestamp}.txt", 'w') as f:
            f.write(str(e))
        raise

# Usage
def my_test(interface):
    # Your test code here
    return {"status": "completed"}

safe_testing_wrapper(my_test, interface='vcan0')
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check if interface exists
ip link show can0

# If using virtual CAN for testing
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# For physical interfaces, check device connection
dmesg | grep -i can
lsusb  # For USB CAN adapters
```

### Permission Denied

```bash
# Add user to dialout group for USB device access
sudo usermod -a -G dialout $USER

# Add user to can group (if exists)
sudo usermod -a -G can $USER

# Logout and login for changes to take effect
```

### No Messages Received

```python
# Verify CAN bus is active
import can

bus = can.interface.Bus(channel='can0', bustype='socketcan')
print(f"Bus state: {bus.state}")

# Check for errors
try:
    msg = bus.recv(timeout=5.0)
    if msg:
        print(f"Received: {msg}")
    else:
        print("No messages in 5 seconds - check vehicle is powered on")
except can.CanError as e:
    print(f"CAN Error: {e}")
```

### High Bus Load

```bash
# Monitor CAN bus load
candump -t A can0 | pv -l > /dev/null

# Filter specific IDs only
candump can0,7DF:7FF  # Only IDs 0x7DF to 0x7FF
```

## Safety Warnings

**CRITICAL**: This framework interacts with vehicle systems. Improper use can cause:
- Vehicle malfunction
- Loss of control
- Physical damage
- Safety hazards

Always:
- Test on virtual/bench setups first (vcan0)
- Use test mode when possible
- Have emergency stop procedures
- Never test on moving vehicles
- Follow automotive testing standards
- Comply with local regulations
