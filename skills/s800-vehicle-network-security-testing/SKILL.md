---
name: s800-vehicle-network-security-testing
description: Test and analyze vehicle network security vulnerabilities for CAN, LIN, and automotive protocols
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - perform automotive network testing
  - scan vehicle communication protocols
  - test automotive ECU security
  - analyze vehicle network traffic
  - perform CAN bus security assessment
  - test automotive protocol vulnerabilities
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive vehicle network security testing framework designed for penetration testing and vulnerability assessment of automotive networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and other vehicle communication protocols. The framework enables security researchers and automotive engineers to analyze ECU (Electronic Control Unit) behavior, detect anomalies, and identify security vulnerabilities in vehicle networks.

**Key Capabilities:**
- CAN bus traffic capture and analysis
- Message injection and replay attacks
- Fuzzing automotive protocols
- ECU fingerprinting and enumeration
- Anomaly detection in vehicle networks
- Protocol reverse engineering
- Real-time traffic monitoring

## Installation

### Prerequisites

```bash
# Install required system dependencies
sudo apt-get update
sudo apt-get install -y python3 python3-pip can-utils

# Enable SocketCAN kernel modules
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

### Hardware Setup (Physical CAN Interface)

```bash
# For physical CAN interfaces (e.g., SocketCAN compatible hardware)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ip link show can0
```

## Configuration

### Framework Configuration

Create a configuration file `config.json`:

```json
{
  "interface": "vcan0",
  "bitrate": 500000,
  "logging": {
    "level": "INFO",
    "output": "logs/s800.log"
  },
  "fuzzing": {
    "duration": 60,
    "delay": 0.01,
    "seed": null
  },
  "capture": {
    "output_dir": "captures/",
    "format": "candump"
  }
}
```

### Environment Variables

```bash
# Set interface
export S800_INTERFACE=can0

# Set log level
export S800_LOG_LEVEL=DEBUG

# Set capture directory
export S800_CAPTURE_DIR=/path/to/captures
```

## Core Usage Patterns

### Traffic Capture and Analysis

```python
#!/usr/bin/env python3
from s800.capture import CANCapture
from s800.analysis import TrafficAnalyzer

# Initialize capture on CAN interface
capture = CANCapture(interface='vcan0')

# Start capturing traffic
print("Starting CAN traffic capture...")
capture.start()

# Capture for 30 seconds
import time
time.sleep(30)

# Stop and save capture
messages = capture.stop()
capture.save('capture_session.log')

# Analyze captured traffic
analyzer = TrafficAnalyzer(messages)
stats = analyzer.get_statistics()

print(f"Total messages: {stats['total']}")
print(f"Unique CAN IDs: {stats['unique_ids']}")
print(f"Message frequency: {stats['frequency']}")
```

### Message Injection

```python
#!/usr/bin/env python3
from s800.injection import CANInjector

# Initialize injector
injector = CANInjector(interface='vcan0')

# Single message injection
injector.send(
    can_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08]
)

# Periodic message injection (every 100ms)
injector.send_periodic(
    can_id=0x456,
    data=[0xAA, 0xBB, 0xCC, 0xDD],
    interval=0.1,
    duration=10.0
)

# Burst injection
injector.send_burst(
    can_id=0x789,
    data=[0xFF, 0x00, 0xFF, 0x00],
    count=100,
    delay=0.001
)
```

### Fuzzing

```python
#!/usr/bin/env python3
from s800.fuzzing import CANFuzzer

# Initialize fuzzer
fuzzer = CANFuzzer(interface='vcan0')

# Random fuzzing on specific CAN ID
fuzzer.fuzz_random(
    can_id=0x200,
    duration=60,  # seconds
    delay=0.01    # 10ms between messages
)

# Sequential fuzzing (iterate through values)
fuzzer.fuzz_sequential(
    can_id=0x300,
    start_value=0x00,
    end_value=0xFF,
    byte_position=0,
    delay=0.05
)

# Smart fuzzing with callback
def monitor_response(message):
    """Monitor for unusual responses"""
    if message.arbitration_id == 0x7DF:  # Diagnostic response
        print(f"Response detected: {message.data.hex()}")

fuzzer.fuzz_with_monitor(
    target_id=0x7E0,  # Diagnostic request
    monitor_ids=[0x7E8, 0x7DF],
    callback=monitor_response,
    duration=120
)
```

### ECU Fingerprinting

```python
#!/usr/bin/env python3
from s800.enumeration import ECUEnumerator

# Initialize enumerator
enumerator = ECUEnumerator(interface='vcan0')

# Scan for active ECUs
print("Scanning for ECUs...")
ecus = enumerator.scan_network(timeout=10)

for ecu in ecus:
    print(f"ECU ID: 0x{ecu['id']:03X}")
    print(f"  Message count: {ecu['count']}")
    print(f"  Frequency: {ecu['frequency']} Hz")
    print(f"  Data pattern: {ecu['pattern']}")

# UDS (Unified Diagnostic Services) enumeration
uds_responses = enumerator.scan_uds_services(
    diagnostic_id=0x7E0,
    response_id=0x7E8
)

for service in uds_responses:
    print(f"Service 0x{service:02X}: Available")
```

### Replay Attacks

```python
#!/usr/bin/env python3
from s800.replay import CANReplay

# Load captured traffic
replay = CANReplay(interface='vcan0')
replay.load_capture('capture_session.log')

# Replay with original timing
replay.replay_with_timing()

# Replay at different speed (2x faster)
replay.replay_with_speed(speed=2.0)

# Replay specific CAN ID only
replay.replay_filtered(
    can_ids=[0x123, 0x456],
    loop=True,
    count=10
)

# Modify and replay
def modify_message(msg):
    """Modify data before replay"""
    if msg.arbitration_id == 0x200:
        # Increment first byte
        data = list(msg.data)
        data[0] = (data[0] + 1) % 256
        msg.data = bytes(data)
    return msg

replay.replay_with_modifier(modifier=modify_message)
```

### Anomaly Detection

```python
#!/usr/bin/env python3
from s800.detection import AnomalyDetector
from s800.capture import CANCapture

# Train on baseline traffic
detector = AnomalyDetector()

# Capture baseline (normal operation)
print("Capturing baseline traffic...")
capture = CANCapture(interface='vcan0')
capture.start()
time.sleep(300)  # 5 minutes of normal traffic
baseline = capture.stop()

# Train detector
detector.train(baseline)
detector.save_model('baseline_model.pkl')

# Monitor for anomalies in real-time
def anomaly_callback(message, score):
    if score > 0.8:  # High anomaly score
        print(f"ANOMALY: ID 0x{message.arbitration_id:03X}")
        print(f"  Data: {message.data.hex()}")
        print(f"  Score: {score:.2f}")

detector.monitor_realtime(
    interface='vcan0',
    callback=anomaly_callback,
    threshold=0.8
)
```

## CLI Commands

### Traffic Capture

```bash
# Capture CAN traffic to file
s800 capture -i can0 -o traffic.log -d 60

# Capture with filtering
s800 capture -i can0 --filter "0x100-0x1FF" -o filtered.log

# Real-time display
s800 capture -i can0 --display --format candump
```

### Analysis

```bash
# Analyze captured traffic
s800 analyze traffic.log --stats

# Generate frequency analysis
s800 analyze traffic.log --frequency --graph

# Extract unique CAN IDs
s800 analyze traffic.log --extract-ids

# Protocol detection
s800 analyze traffic.log --detect-protocol
```

### Injection

```bash
# Send single CAN message
s800 send -i can0 --id 0x123 --data "01 02 03 04 05 06 07 08"

# Periodic transmission
s800 send -i can0 --id 0x456 --data "AA BB CC DD" --interval 100 --count 100

# Send from file
s800 send -i can0 --file messages.txt
```

### Fuzzing

```bash
# Random fuzzing
s800 fuzz -i can0 --id 0x200 --random --duration 60

# Sequential fuzzing
s800 fuzz -i can0 --id 0x300 --sequential --byte 0 --delay 50

# Smart fuzzing with response monitoring
s800 fuzz -i can0 --id 0x7E0 --monitor 0x7E8 --smart
```

### Scanning

```bash
# Scan for active CAN IDs
s800 scan -i can0 --timeout 10

# UDS service discovery
s800 scan -i can0 --uds --diagnostic-id 0x7E0 --response-id 0x7E8

# ECU enumeration
s800 scan -i can0 --enumerate-ecus --verbose
```

## Common Testing Scenarios

### Door Unlock Test

```python
#!/usr/bin/env python3
from s800.capture import CANCapture
from s800.replay import CANReplay

# Step 1: Capture during legitimate door unlock
print("Capture traffic while unlocking door...")
capture = CANCapture(interface='can0')
capture.start()
input("Press Enter after unlocking door...")
messages = capture.stop()
capture.save('door_unlock.log')

# Step 2: Analyze and identify unlock message
from s800.analysis import TrafficAnalyzer
analyzer = TrafficAnalyzer(messages)
candidates = analyzer.find_unique_messages(threshold=1)

print("Potential unlock messages:")
for msg in candidates:
    print(f"ID: 0x{msg.arbitration_id:03X}, Data: {msg.data.hex()}")

# Step 3: Replay to test
replay = CANReplay(interface='can0')
replay.load_capture('door_unlock.log')
print("Replaying unlock sequence...")
replay.replay_filtered(can_ids=[msg.arbitration_id for msg in candidates])
```

### Speed Spoofing Detection

```python
#!/usr/bin/env python3
from s800.monitoring import SpeedMonitor
from s800.injection import CANInjector

# Monitor actual speed messages
monitor = SpeedMonitor(interface='can0', speed_id=0x244)

# Attempt spoofing
injector = CANInjector(interface='can0')

# Inject fake speed (80 km/h = 0x50)
injector.send_periodic(
    can_id=0x244,
    data=[0x50, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00],
    interval=0.02,  # 50Hz
    duration=5.0
)

# Check if spoofing was effective
results = monitor.get_readings()
print(f"Spoofing effectiveness: {results['spoofed_percentage']}%")
```

## Troubleshooting

### CAN Interface Not Available

```bash
# Check if SocketCAN modules are loaded
lsmod | grep can

# Load manually if needed
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Verify interface exists
ip link show can0
```

### Permission Denied

```bash
# Add user to dialout/can groups
sudo usermod -a -G dialout $USER
sudo usermod -a -G can $USER

# Or run with sudo (not recommended for production)
sudo python3 your_script.py
```

### No Traffic Captured

```python
# Verify interface is up
import os
result = os.system('ip link show can0 | grep "UP"')
if result != 0:
    print("Interface is down, bringing up...")
    os.system('sudo ip link set can0 up')

# Check bitrate matches
os.system('ip -details link show can0')

# Test with candump
os.system('candump can0')
```

### High Packet Loss

```python
# Increase buffer size
from s800.capture import CANCapture

capture = CANCapture(
    interface='can0',
    buffer_size=8192  # Increase from default
)

# Use hardware timestamping if available
capture.enable_hardware_timestamps()
```

## Safety and Legal Considerations

```python
# Always implement safety checks
from s800.safety import SafetyController

safety = SafetyController(interface='can0')

# Set critical message protection
safety.protect_ids([0x100, 0x101, 0x102])  # Brake, steering, etc.

# Enable emergency stop
safety.enable_emergency_stop(trigger_signal='SIGINT')

# Monitor bus load
safety.set_bus_load_limit(80)  # Max 80% bus utilization

# Start testing with safety enabled
with safety.protected_context():
    # Your testing code here
    fuzzer.fuzz_random(can_id=0x500, duration=60)
```

**Warning:** Only test on isolated networks or vehicles you own/have authorization to test. Unauthorized vehicle network testing may be illegal and dangerous.
