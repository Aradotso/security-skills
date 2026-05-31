---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, sniffing, and attack simulation capabilities
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - fuzz automotive protocols
  - simulate vehicle network attacks
  - capture CAN messages
  - test ECU vulnerabilities
  - audit vehicle communication security
  - perform automotive penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive security testing framework designed for automotive vehicle networks. It supports testing and analyzing protocols like CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay. The framework enables security researchers and automotive engineers to:

- Sniff and capture vehicle network traffic
- Perform protocol fuzzing and attack simulation
- Test ECU (Electronic Control Unit) vulnerabilities
- Replay and modify network messages
- Analyze network behavior and anomalies

**Note**: This is a test framework. Ensure you have proper authorization before testing any vehicle systems. Unauthorized access to vehicle networks may be illegal.

## Installation

### Prerequisites

- Python 3.7+
- Hardware interface (e.g., CANtact, Kvaser, PCAN adapter)
- Linux system with SocketCAN support (recommended)

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Install system-level CAN utilities (Linux)
sudo apt-get install can-utils

# Set up SocketCAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

### Hardware Configuration

```bash
# Verify CAN interface
ifconfig can0

# Test with candump
candump can0

# Send test frame
cansend can0 123#DEADBEEF
```

## Core Components

### 1. Traffic Sniffing

Capture and analyze CAN bus traffic:

```python
from s800.sniffer import CANSniffer
from s800.protocols import CAN

# Initialize sniffer
sniffer = CANSniffer(interface='can0', bitrate=500000)

# Start capturing
sniffer.start()

# Capture with filter
sniffer.capture(
    filter_id=0x123,
    duration=60,
    output_file='capture.log'
)

# Real-time analysis
def message_handler(msg):
    print(f"ID: {msg.arbitration_id:#x}, Data: {msg.data.hex()}")
    if msg.arbitration_id == 0x7DF:  # OBD diagnostic request
        print("Diagnostic message detected")

sniffer.add_callback(message_handler)
sniffer.start()
```

### 2. Message Replay

Replay captured traffic:

```python
from s800.replay import MessageReplayer
from s800.parser import LogParser

# Parse captured log
parser = LogParser('capture.log')
messages = parser.parse()

# Initialize replayer
replayer = MessageReplayer(interface='can0')

# Replay messages
replayer.replay(
    messages=messages,
    timing='original',  # or 'fast', 'slow'
    loop=False
)

# Replay with modifications
for msg in messages:
    if msg.arbitration_id == 0x123:
        msg.data = bytearray([0xFF] * 8)  # Modify payload
    replayer.send(msg)
```

### 3. Protocol Fuzzing

Fuzz CAN messages to test ECU robustness:

```python
from s800.fuzzer import CANFuzzer
from s800.fuzzer.strategies import RandomStrategy, MutationStrategy

# Initialize fuzzer
fuzzer = CANFuzzer(interface='can0')

# Random fuzzing
fuzzer.fuzz_random(
    id_range=(0x100, 0x7FF),
    data_length=8,
    count=1000,
    delay=0.01  # seconds between messages
)

# Mutation-based fuzzing
base_messages = [
    {'id': 0x123, 'data': b'\x01\x02\x03\x04\x05\x06\x07\x08'},
    {'id': 0x456, 'data': b'\xAA\xBB\xCC\xDD\xEE\xFF\x00\x11'}
]

strategy = MutationStrategy(
    bit_flip_prob=0.1,
    byte_flip_prob=0.05,
    boundary_values=True
)

fuzzer.fuzz_mutation(
    base_messages=base_messages,
    strategy=strategy,
    iterations=5000
)

# Monitor for crashes/anomalies
def anomaly_detector(msg, response):
    if response is None:
        print(f"No response for ID {msg.arbitration_id:#x}")
    elif len(response.data) != 8:
        print(f"Abnormal response length: {len(response.data)}")

fuzzer.add_monitor(anomaly_detector)
```

### 4. Attack Simulation

Simulate common vehicle network attacks:

```python
from s800.attacks import (
    DosAttack,
    InjectionAttack,
    ReplayAttack,
    MitMAttack
)

# DoS attack - flood the bus
dos = DosAttack(interface='can0')
dos.flood(
    target_id=0x123,
    priority='high',  # Uses high-priority arbitration ID
    duration=10
)

# Message injection
injector = InjectionAttack(interface='can0')

# Inject spoofed messages
injector.inject(
    arbitration_id=0x200,
    data=b'\x00\x64\x00\x00\x00\x00\x00\x00',  # Spoofed speed value
    interval=0.1,
    count=100
)

# Man-in-the-Middle
mitm = MitMAttack(interface='can0')

# Intercept and modify
def modify_handler(msg):
    if msg.arbitration_id == 0x180:
        # Modify steering angle data
        msg.data = bytearray([msg.data[0] | 0x10] + list(msg.data[1:]))
    return msg

mitm.start(callback=modify_handler)
```

### 5. Diagnostic Testing (UDS)

Test Unified Diagnostic Services (ISO 14229):

```python
from s800.diagnostics import UDSClient
from s800.diagnostics.services import *

# Initialize UDS client
client = UDSClient(
    interface='can0',
    request_id=0x7DF,   # Broadcast diagnostic request
    response_id=0x7E8   # ECU response
)

# Start diagnostic session
client.start_session(session_type=SessionType.EXTENDED)

# Read DTC (Diagnostic Trouble Codes)
dtcs = client.read_dtc()
for dtc in dtcs:
    print(f"DTC: {dtc.code}, Status: {dtc.status}")

# Security access test
seed = client.request_seed(level=0x01)
key = calculate_key(seed)  # Implement key algorithm
client.send_key(key)

# Read data by identifier
vin = client.read_data_by_id(identifier=0xF190)  # VIN
print(f"VIN: {vin.decode()}")

# Memory read
data = client.read_memory(
    address=0x10000,
    length=256
)

# ECU reset
client.ecu_reset(reset_type=ResetType.HARD)
```

## Configuration

### Framework Configuration

Create `config.yaml`:

```yaml
# CAN Interface Configuration
can:
  interface: can0
  bitrate: 500000
  fd_mode: false
  extended_id: false

# Logging
logging:
  level: INFO
  file: s800.log
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"

# Fuzzing
fuzzer:
  default_delay: 0.01
  max_iterations: 10000
  monitor_responses: true
  timeout: 1.0

# Attack Simulation
attacks:
  dos:
    max_rate: 1000  # messages per second
    priority_escalation: true
  
  injection:
    verify_injection: true
    stealth_mode: false

# Diagnostics
uds:
  timeout: 2.0
  retry_count: 3
  security_key_algo: ${SECURITY_KEY_ALGO}  # Reference env var
```

Load configuration:

```python
from s800.config import Config

config = Config.load('config.yaml')
sniffer = CANSniffer(
    interface=config.can.interface,
    bitrate=config.can.bitrate
)
```

## Common Patterns

### Pattern 1: Baseline Capture and Analysis

```python
from s800.sniffer import CANSniffer
from s800.analysis import TrafficAnalyzer

# Capture normal traffic baseline
sniffer = CANSniffer(interface='can0')
baseline = sniffer.capture(duration=300, output_file='baseline.log')

# Analyze patterns
analyzer = TrafficAnalyzer()
analyzer.load('baseline.log')

# Get statistics
stats = analyzer.get_statistics()
print(f"Unique IDs: {stats['unique_ids']}")
print(f"Message rate: {stats['msg_per_second']}")
print(f"Most frequent: {stats['top_ids']}")

# Detect anomalies in new capture
sniffer.capture(duration=60, output_file='test.log')
analyzer.compare('baseline.log', 'test.log')
anomalies = analyzer.detect_anomalies(threshold=0.95)
```

### Pattern 2: Automated Vulnerability Scanning

```python
from s800.scanner import VulnerabilityScanner
from s800.reports import HTMLReport

# Initialize scanner
scanner = VulnerabilityScanner(interface='can0')

# Scan for common vulnerabilities
results = scanner.scan_all(
    tests=[
        'unauthenticated_diag',
        'weak_security_access',
        'replay_vulnerability',
        'dos_susceptibility',
        'injection_detection'
    ]
)

# Generate report
report = HTMLReport(results)
report.save('vulnerability_report.html')

# Check specific vulnerability
if scanner.check_replay_vulnerability(target_id=0x123):
    print("Replay attack possible on ID 0x123")
```

### Pattern 3: Custom Message Analysis

```python
from s800.sniffer import CANSniffer
from s800.decoder import MessageDecoder

# Define message structure
decoder = MessageDecoder()
decoder.add_signal(
    name='VehicleSpeed',
    start_bit=0,
    length=16,
    scale=0.01,
    offset=0,
    unit='km/h'
)
decoder.add_signal(
    name='EngineRPM',
    start_bit=16,
    length=16,
    scale=0.25,
    offset=0,
    unit='rpm'
)

# Decode messages
sniffer = CANSniffer(interface='can0')

def decode_handler(msg):
    if msg.arbitration_id == 0x300:
        values = decoder.decode(msg.data)
        print(f"Speed: {values['VehicleSpeed']} km/h")
        print(f"RPM: {values['EngineRPM']} rpm")

sniffer.add_callback(decode_handler)
sniffer.start()
```

## Troubleshooting

### Interface Not Found

```bash
# Check available interfaces
ip link show

# Bring up interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Check for errors
dmesg | grep can
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python test_script.py
```

### No Messages Received

```python
# Verify bitrate matches vehicle network
# Common rates: 125k, 250k, 500k, 1M
sniffer = CANSniffer(interface='can0', bitrate=250000)

# Enable promiscuous mode
sniffer.set_filter(None)  # Accept all messages

# Check hardware connection
import can
bus = can.interface.Bus(channel='can0', bustype='socketcan')
print(bus.state)  # Should be 'ACTIVE'
```

### Bus Off Error

```bash
# Reset the interface
sudo ip link set down can0
sudo ip link set up can0

# Check error counters
cat /sys/class/net/can0/statistics/rx_errors
```

## Security Considerations

- **Authorization**: Only test systems you own or have explicit permission to test
- **Isolation**: Use isolated test benches; never test on production vehicles without safety measures
- **Backup**: Always maintain ability to restore original ECU firmware
- **Safety**: Disable critical systems (brakes, steering) during aggressive testing
- **Legal**: Understand local laws regarding vehicle network access

## Environment Variables

```bash
# Set security key algorithm path
export SECURITY_KEY_ALGO=/path/to/key_algorithm.py

# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Enable debug logging
export S800_DEBUG=1

# Set output directory
export S800_OUTPUT_DIR=/var/log/s800
```
