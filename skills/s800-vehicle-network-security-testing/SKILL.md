---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing and attack simulation capabilities
triggers:
  - test vehicle network security
  - perform CAN bus fuzzing
  - simulate automotive network attacks
  - test car network vulnerabilities
  - analyze vehicle communication protocols
  - run S800 security tests
  - fuzz automotive ECU communications
  - test in-vehicle network security
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for security assessment, fuzzing, traffic analysis, and attack simulation on in-vehicle communication systems.

**Key capabilities:**
- CAN bus message injection and sniffing
- Protocol fuzzing for ECU testing
- Replay attacks and message manipulation
- Traffic analysis and logging
- DoS attack simulation
- Diagnostic protocol testing (UDS, OBD-II)

## Installation

### Prerequisites

```bash
# Install required system packages (Linux)
sudo apt-get update
sudo apt-get install can-utils python3 python3-pip git

# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Setup virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup (Optional)

For real vehicle testing, connect CAN interface hardware:

```bash
# Configure physical CAN interface (e.g., can0)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Configuration

### Network Interface Configuration

Create `config.ini`:

```ini
[NETWORK]
interface = vcan0
bitrate = 500000
protocol = CAN

[FUZZING]
iterations = 10000
timeout = 5
random_seed = 42

[LOGGING]
log_level = INFO
log_file = s800_test.log
pcap_output = true

[TARGETS]
ecu_ids = 0x100,0x200,0x300,0x7E0
```

### Environment Variables

```bash
export S800_INTERFACE=vcan0
export S800_LOG_LEVEL=DEBUG
export S800_OUTPUT_DIR=./results
```

## Core Usage Patterns

### CAN Bus Sniffing

```python
#!/usr/bin/env python3
from s800.can_sniffer import CANSniffer
from s800.utils import setup_logging

# Initialize logger
logger = setup_logging('can_capture')

# Create sniffer instance
sniffer = CANSniffer(interface='vcan0')

# Start capturing with filter
sniffer.start(
    filter_ids=[0x100, 0x7E0],
    duration=60,  # Capture for 60 seconds
    output_file='capture.pcap'
)

# Analyze captured traffic
stats = sniffer.get_statistics()
print(f"Total frames: {stats['total_frames']}")
print(f"Unique IDs: {stats['unique_ids']}")
print(f"Frame rate: {stats['avg_rate']} fps")
```

### Message Injection

```python
#!/usr/bin/env python3
from s800.can_injector import CANInjector
from s800.message import CANMessage

# Initialize injector
injector = CANInjector(interface='vcan0')

# Create and send single message
msg = CANMessage(
    arb_id=0x100,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    extended=False
)
injector.send(msg)

# Send periodic messages
injector.send_periodic(
    arb_id=0x200,
    data=[0xFF, 0x00, 0xFF, 0x00],
    interval=0.1,  # 100ms
    duration=10.0   # Send for 10 seconds
)
```

### Fuzzing ECU Communications

```python
#!/usr/bin/env python3
from s800.fuzzer import CANFuzzer
from s800.fuzzer.strategies import RandomStrategy, MutationStrategy

# Initialize fuzzer
fuzzer = CANFuzzer(
    interface='vcan0',
    target_ids=[0x7E0, 0x7E8]  # OBD-II diagnostic IDs
)

# Configure fuzzing strategy
fuzzer.add_strategy(RandomStrategy(
    min_dlc=0,
    max_dlc=8,
    rate=100  # Messages per second
))

# Start fuzzing with monitoring
fuzzer.start(
    duration=300,  # 5 minutes
    monitor_responses=True,
    crash_detection=True,
    callback=lambda msg: print(f"Response: {msg}")
)

# Get fuzzing results
results = fuzzer.get_results()
print(f"Messages sent: {results['sent']}")
print(f"Responses received: {results['responses']}")
print(f"Anomalies detected: {results['anomalies']}")
```

### Replay Attack Simulation

```python
#!/usr/bin/env python3
from s800.replay import ReplayAttack
from s800.pcap_parser import parse_pcap

# Load captured traffic
messages = parse_pcap('legitimate_traffic.pcap')

# Initialize replay attack
replay = ReplayAttack(interface='vcan0')

# Replay with modifications
replay.replay(
    messages=messages,
    speed_multiplier=2.0,  # 2x speed
    repeat=5,
    modify_callback=lambda msg: modify_message(msg)
)

def modify_message(msg):
    """Modify message data for testing"""
    if msg.arb_id == 0x200:
        # Increment first byte
        msg.data[0] = (msg.data[0] + 1) % 256
    return msg
```

### UDS Diagnostic Testing

```python
#!/usr/bin/env python3
from s800.protocols.uds import UDSClient
from s800.protocols.uds.services import *

# Connect to ECU
client = UDSClient(
    interface='vcan0',
    request_id=0x7E0,
    response_id=0x7E8
)

# Start diagnostic session
client.start_diagnostic_session(SESSION_TYPE_EXTENDED)

# Read DTC (Diagnostic Trouble Codes)
dtcs = client.read_dtc()
for dtc in dtcs:
    print(f"DTC: {dtc['code']} - Status: {dtc['status']}")

# Read data by identifier
vin = client.read_data_by_id(0xF190)  # VIN identifier
print(f"Vehicle VIN: {vin.decode('ascii')}")

# Security access (seed/key)
seed = client.request_seed(level=0x01)
key = calculate_key(seed)  # Custom key algorithm
client.send_key(key)

# Write data (requires security access)
client.write_data_by_id(0x1234, b'\x00\x01\x02\x03')
```

### DoS Attack Testing

```python
#!/usr/bin/env python3
from s800.attacks import DoSAttack
from s800.monitoring import BusMonitor

# Initialize monitor to observe impact
monitor = BusMonitor(interface='vcan0')
monitor.start_background()

# Configure DoS attack
dos = DoSAttack(interface='vcan0')

# Bus flooding attack
dos.bus_flood(
    arb_id=0x000,  # Highest priority
    data=[0xFF] * 8,
    duration=30,    # 30 seconds
    max_rate=True   # Maximum throughput
)

# Check bus health after attack
health = monitor.get_bus_health()
print(f"Bus utilization: {health['utilization']}%")
print(f"Error frames: {health['error_count']}")
print(f"Lost messages: {health['lost_messages']}")
```

## CLI Commands

### Basic Sniffing

```bash
# Capture CAN traffic
python3 -m s800.cli sniffer --interface vcan0 --output capture.log

# With filtering
python3 -m s800.cli sniffer -i vcan0 --filter 0x100,0x200 --pcap output.pcap
```

### Send Messages

```bash
# Send single message
python3 -m s800.cli send -i vcan0 --id 0x100 --data 01020304

# Send from file
python3 -m s800.cli send -i vcan0 --file messages.txt
```

### Fuzzing

```bash
# Random fuzzing
python3 -m s800.cli fuzz -i vcan0 --target 0x7E0 --duration 300 --strategy random

# Mutation-based fuzzing
python3 -m s800.cli fuzz -i vcan0 --target 0x7E0 --strategy mutation --corpus corpus.pcap
```

### Replay Traffic

```bash
# Replay captured traffic
python3 -m s800.cli replay -i vcan0 --pcap traffic.pcap --speed 1.0

# Replay with loop
python3 -m s800.cli replay -i vcan0 --pcap traffic.pcap --loop 10
```

### UDS Scanner

```bash
# Scan for ECUs
python3 -m s800.cli uds-scan -i vcan0 --range 0x700-0x7FF

# Enumerate services
python3 -m s800.cli uds-enum -i vcan0 --ecu 0x7E0
```

## Advanced Patterns

### Custom Fuzzing Strategy

```python
#!/usr/bin/env python3
from s800.fuzzer.base import FuzzingStrategy
from s800.message import CANMessage
import random

class IntelligentFuzzer(FuzzingStrategy):
    """Custom fuzzing strategy targeting specific protocol fields"""
    
    def __init__(self, protocol_spec):
        self.spec = protocol_spec
        self.test_cases = []
        
    def generate_message(self):
        """Generate test case based on protocol knowledge"""
        arb_id = random.choice(self.spec['valid_ids'])
        
        # Build data based on protocol structure
        data = bytearray(8)
        data[0] = random.choice(self.spec['service_ids'])
        data[1] = random.randint(0, 255)  # Subfunction
        
        # Add boundary values for remaining bytes
        for i in range(2, 8):
            data[i] = random.choice([0x00, 0xFF, 0x7F, 0x80])
            
        return CANMessage(arb_id=arb_id, data=data)
    
    def mutate_message(self, original):
        """Intelligent mutation preserving protocol structure"""
        mutated = original.copy()
        
        # Keep service ID, mutate parameters
        mutation_byte = random.randint(1, 7)
        mutated.data[mutation_byte] ^= (1 << random.randint(0, 7))
        
        return mutated

# Use custom strategy
protocol_spec = {
    'valid_ids': [0x7E0, 0x7E1, 0x7E2],
    'service_ids': [0x10, 0x11, 0x22, 0x27, 0x31]
}

fuzzer = CANFuzzer(interface='vcan0')
fuzzer.add_strategy(IntelligentFuzzer(protocol_spec))
fuzzer.start(duration=600)
```

### Traffic Analysis and Anomaly Detection

```python
#!/usr/bin/env python3
from s800.analysis import TrafficAnalyzer
from s800.ml import AnomalyDetector

# Analyze baseline traffic
analyzer = TrafficAnalyzer()
analyzer.load_pcap('baseline_traffic.pcap')

# Extract features
baseline_features = analyzer.extract_features([
    'message_frequency',
    'data_entropy',
    'inter_arrival_time',
    'payload_patterns'
])

# Train anomaly detector
detector = AnomalyDetector(algorithm='isolation_forest')
detector.train(baseline_features)

# Monitor live traffic for anomalies
from s800.can_sniffer import CANSniffer

sniffer = CANSniffer(interface='vcan0')

def check_anomaly(msg):
    features = analyzer.extract_message_features(msg)
    if detector.is_anomaly(features):
        print(f"Anomaly detected: ID={hex(msg.arb_id)}, Data={msg.data.hex()}")
        print(f"Anomaly score: {detector.get_score(features)}")

sniffer.start(callback=check_anomaly)
```

## Troubleshooting

### Permission Denied on CAN Interface

```bash
# Add user to netdev group
sudo usermod -aG dialout $USER
sudo usermod -aG netdev $USER

# Or run with sudo
sudo python3 script.py
```

### Interface Not Found

```bash
# List available interfaces
ip link show

# Bring up interface
sudo ip link set can0 up type can bitrate 500000

# Check interface status
ip -details link show can0
```

### No Messages Received

```python
# Verify interface is receiving
from s800.diagnostics import test_interface

result = test_interface('vcan0')
if not result['rx_ok']:
    print("Interface not receiving. Check connections.")
    print(f"RX errors: {result['rx_errors']}")
```

### High Error Rate

```python
# Adjust bitrate
from s800.utils import set_bitrate

# Try common automotive bitrates
for bitrate in [125000, 250000, 500000, 1000000]:
    set_bitrate('can0', bitrate)
    # Test communication
    if test_connection():
        print(f"Success with bitrate: {bitrate}")
        break
```

### Memory Issues During Long Fuzzing

```python
# Enable batch processing and periodic cleanup
fuzzer = CANFuzzer(
    interface='vcan0',
    batch_size=1000,
    auto_cleanup=True
)

fuzzer.start(
    duration=3600,
    checkpoint_interval=300  # Save state every 5 minutes
)
```

## Safety Warnings

**⚠️ CRITICAL:** Only use on authorized test vehicles or isolated test benches. Unauthorized vehicle network testing is illegal and dangerous.

- Always use isolated test environments
- Never test on public roads or operational vehicles
- Understand that malformed messages can damage ECUs
- Keep emergency shutdown procedures ready
- Follow automotive cybersecurity standards (ISO 21434)
