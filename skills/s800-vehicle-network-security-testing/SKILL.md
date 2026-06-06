---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, focusing on CAN bus and automotive protocol testing
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive security testing
  - scan vehicle network vulnerabilities
  - use S800 framework for car hacking
  - test automotive protocols
  - fuzz vehicle CAN messages
  - monitor vehicle network communications
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

S800 is a vehicle network security testing framework designed for penetration testing and security analysis of automotive networks, particularly CAN (Controller Area Network) bus systems. It provides tools for traffic monitoring, fuzzing, replay attacks, and vulnerability discovery in vehicle electronic control units (ECUs).

## Installation

### Prerequisites

- Python 3.7+
- SocketCAN support (Linux kernel 2.6.25+)
- CAN interface hardware (USB-to-CAN adapter, OBD-II dongle, etc.)

### Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Or install individual packages
pip install python-can cantools pyserial
```

### Hardware Setup

```bash
# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Set up virtual CAN interface (for testing without hardware)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Set up physical CAN interface (replace can0 with your interface)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Functionality

### CAN Bus Traffic Monitoring

Monitor and log CAN bus messages in real-time:

```python
import can

def monitor_can_traffic(interface='can0', duration=10):
    """Monitor CAN bus traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Monitoring {interface} for {duration} seconds...")
    
    try:
        for msg in bus:
            print(f"ID: {msg.arbitration_id:03X} Data: {msg.data.hex()}")
            
            if duration and bus.recv(timeout=duration) is None:
                break
    except KeyboardInterrupt:
        print("\nMonitoring stopped")
    finally:
        bus.shutdown()

# Usage
monitor_can_traffic('vcan0', duration=30)
```

### CAN Message Analysis

Parse and decode CAN messages:

```python
import can
import cantools

def analyze_can_messages(interface='can0', dbc_file=None):
    """Analyze CAN messages using DBC database"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Load DBC file if provided
    db = None
    if dbc_file:
        db = cantools.database.load_file(dbc_file)
    
    for message in bus:
        print(f"\n[{message.timestamp}] CAN ID: 0x{message.arbitration_id:03X}")
        print(f"Data: {message.data.hex()}")
        print(f"DLC: {message.dlc}")
        
        # Decode if DBC available
        if db:
            try:
                decoded = db.decode_message(message.arbitration_id, message.data)
                print(f"Decoded: {decoded}")
            except Exception as e:
                print(f"Decode error: {e}")

# Usage
analyze_can_messages('vcan0', dbc_file='vehicle.dbc')
```

### CAN Fuzzing

Fuzz CAN messages to discover vulnerabilities:

```python
import can
import random
import time

def fuzz_can_messages(interface='can0', target_id=None, iterations=1000):
    """Fuzz CAN bus with random messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Starting fuzzing on {interface}")
    print(f"Target ID: {target_id if target_id else 'All IDs'}")
    print(f"Iterations: {iterations}")
    
    for i in range(iterations):
        # Generate random CAN ID if not specified
        can_id = target_id if target_id else random.randint(0, 0x7FF)
        
        # Generate random data (0-8 bytes)
        data_length = random.randint(1, 8)
        data = bytes([random.randint(0, 0xFF) for _ in range(data_length)])
        
        # Create and send message
        msg = can.Message(
            arbitration_id=can_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            print(f"[{i+1}/{iterations}] Sent ID: 0x{can_id:03X} Data: {data.hex()}")
            time.sleep(0.01)  # Rate limiting
        except can.CanError as e:
            print(f"Error sending message: {e}")
    
    bus.shutdown()

# Usage - fuzz specific ID
fuzz_can_messages('vcan0', target_id=0x123, iterations=500)

# Usage - fuzz all IDs
fuzz_can_messages('vcan0', iterations=1000)
```

### Replay Attacks

Capture and replay CAN traffic:

```python
import can
import time
import pickle

def capture_can_traffic(interface='can0', duration=30, output_file='captured.pkl'):
    """Capture CAN traffic to file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    print(f"Capturing traffic for {duration} seconds...")
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append({
                'id': msg.arbitration_id,
                'data': msg.data,
                'timestamp': msg.timestamp,
                'is_extended': msg.is_extended_id
            })
            print(f"Captured: ID 0x{msg.arbitration_id:03X}")
    
    # Save to file
    with open(output_file, 'wb') as f:
        pickle.dump(messages, f)
    
    print(f"Captured {len(messages)} messages to {output_file}")
    bus.shutdown()

def replay_can_traffic(interface='can0', input_file='captured.pkl', speed=1.0):
    """Replay captured CAN traffic"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Load captured messages
    with open(input_file, 'rb') as f:
        messages = pickle.load(f)
    
    print(f"Replaying {len(messages)} messages at {speed}x speed...")
    
    for i, msg_data in enumerate(messages):
        msg = can.Message(
            arbitration_id=msg_data['id'],
            data=msg_data['data'],
            is_extended_id=msg_data['is_extended']
        )
        
        bus.send(msg)
        print(f"[{i+1}/{len(messages)}] Replayed: ID 0x{msg.arbitration_id:03X}")
        
        # Timing between messages
        if i < len(messages) - 1:
            delay = (messages[i+1]['timestamp'] - msg_data['timestamp']) / speed
            time.sleep(max(0, delay))
    
    print("Replay complete")
    bus.shutdown()

# Usage
capture_can_traffic('vcan0', duration=60, output_file='attack.pkl')
replay_can_traffic('vcan0', input_file='attack.pkl', speed=1.0)
```

### ECU Discovery

Scan for active ECUs on the network:

```python
import can
import time

def discover_ecus(interface='can0', scan_range=(0x700, 0x7FF), timeout=5):
    """Discover ECUs using UDS diagnostic requests"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    discovered = []
    
    print(f"Scanning CAN IDs from 0x{scan_range[0]:03X} to 0x{scan_range[1]:03X}")
    
    for can_id in range(scan_range[0], scan_range[1] + 1):
        # Send diagnostic request (UDS Session Control)
        request = can.Message(
            arbitration_id=can_id,
            data=[0x02, 0x10, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00],
            is_extended_id=False
        )
        
        bus.send(request)
        
        # Listen for response
        start = time.time()
        while time.time() - start < timeout:
            response = bus.recv(timeout=0.1)
            if response and response.arbitration_id == can_id + 0x08:
                discovered.append({
                    'request_id': can_id,
                    'response_id': response.arbitration_id,
                    'data': response.data.hex()
                })
                print(f"Found ECU: Request 0x{can_id:03X} -> Response 0x{response.arbitration_id:03X}")
                break
    
    print(f"\nDiscovered {len(discovered)} ECUs")
    bus.shutdown()
    return discovered

# Usage
ecus = discover_ecus('vcan0', scan_range=(0x700, 0x780))
```

### Message Injection

Inject specific CAN messages:

```python
import can

def inject_message(interface='can0', can_id=0x123, data=None, count=1, interval=0.1):
    """Inject CAN messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    if data is None:
        data = [0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
    
    msg = can.Message(
        arbitration_id=can_id,
        data=data,
        is_extended_id=False
    )
    
    for i in range(count):
        bus.send(msg)
        print(f"Injected [{i+1}/{count}]: ID 0x{can_id:03X} Data: {bytes(data).hex()}")
        if i < count - 1:
            time.sleep(interval)
    
    bus.shutdown()

# Usage - inject single message
inject_message('vcan0', can_id=0x456, data=[0xDE, 0xAD, 0xBE, 0xEF])

# Usage - inject multiple messages
inject_message('vcan0', can_id=0x123, data=[0x01, 0x02, 0x03], count=10, interval=0.5)
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Set default bitrate
export S800_CAN_BITRATE=500000

# Set logging level
export S800_LOG_LEVEL=DEBUG

# Set output directory
export S800_OUTPUT_DIR=./results
```

### Configuration File

Create `s800_config.json`:

```json
{
  "interface": "can0",
  "bitrate": 500000,
  "logging": {
    "level": "INFO",
    "output": "./logs/s800.log"
  },
  "fuzzing": {
    "default_iterations": 1000,
    "rate_limit_ms": 10
  },
  "scanning": {
    "timeout": 5,
    "retry_count": 3
  }
}
```

## Common Testing Patterns

### Complete Security Assessment

```python
import can
import json
import os

class VehicleSecurityTester:
    def __init__(self, interface='can0'):
        self.interface = interface
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.results = {}
    
    def run_full_assessment(self):
        """Run complete security assessment"""
        print("Starting vehicle security assessment...")
        
        # 1. Passive monitoring
        self.results['baseline'] = self.monitor_baseline(duration=30)
        
        # 2. ECU discovery
        self.results['ecus'] = self.discover_ecus()
        
        # 3. Fuzzing test
        self.results['fuzz_results'] = self.fuzz_test(iterations=500)
        
        # 4. Generate report
        self.generate_report()
    
    def monitor_baseline(self, duration=30):
        """Establish baseline traffic"""
        messages = {}
        start = time.time()
        
        while time.time() - start < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                can_id = msg.arbitration_id
                if can_id not in messages:
                    messages[can_id] = []
                messages[can_id].append(msg.data.hex())
        
        return messages
    
    def discover_ecus(self):
        # Implementation from discover_ecus function
        pass
    
    def fuzz_test(self, iterations=500):
        # Implementation from fuzz_can_messages function
        pass
    
    def generate_report(self):
        """Generate assessment report"""
        report_file = f"assessment_{int(time.time())}.json"
        with open(report_file, 'w') as f:
            json.dump(self.results, f, indent=2)
        print(f"Report saved to {report_file}")
    
    def cleanup(self):
        self.bus.shutdown()

# Usage
tester = VehicleSecurityTester(interface=os.getenv('S800_CAN_INTERFACE', 'vcan0'))
tester.run_full_assessment()
tester.cleanup()
```

## Troubleshooting

### Permission Denied

```bash
# Add user to dialout group for hardware access
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 your_script.py
```

### CAN Interface Not Found

```bash
# List available interfaces
ip link show

# Bring up interface
sudo ip link set can0 up type can bitrate 500000

# Check interface status
ip -details link show can0
```

### No Messages Received

```bash
# Check if interface is up
ip link show can0

# Monitor with candump
candump can0

# Send test message with cansend
cansend can0 123#DEADBEEF
```

### Rate Limiting Issues

```python
# Add delays between messages
import time

for msg in messages:
    bus.send(msg)
    time.sleep(0.01)  # 10ms delay
```

## Safety Warnings

- Only test on isolated networks or vehicles you own
- Never test on public roads or production vehicles
- Understand legal implications in your jurisdiction
- Use virtual CAN (vcan) for development and testing
- Disconnect from critical systems during fuzzing
- Keep emergency stop mechanisms ready
