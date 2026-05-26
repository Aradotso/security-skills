---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, replay, and analysis capabilities
triggers:
  - test vehicle network security
  - automotive CAN bus testing
  - vehicle network fuzzing
  - s800 security framework
  - CAN bus penetration testing
  - automotive security analysis
  - vehicle ECU testing
  - FlexRay security testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for fuzzing, message replay, traffic analysis, and vulnerability assessment of automotive Electronic Control Units (ECUs).

**Note**: This is a testing framework. Only use on vehicles/networks you own or have explicit authorization to test.

## Installation

### Prerequisites

```bash
# Linux dependencies
sudo apt-get update
sudo apt-get install -y can-utils python3-can python3-pip libusb-1.0-0-dev

# Python dependencies
pip3 install python-can cantools j1939 isotp
```

### Framework Setup

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework
pip3 install -r requirements.txt
python3 setup.py install
```

### Hardware Interface Setup

```bash
# Configure SocketCAN interface (for CAN adapters)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ifconfig can0
```

## Core Components

### 1. CAN Bus Sniffer

Capture and analyze CAN bus traffic:

```python
from s800.canbus import CANSniffer
from s800.utils import Logger

# Initialize sniffer
sniffer = CANSniffer(interface='can0', bitrate=500000)

# Start passive monitoring
sniffer.start_capture(
    duration=60,  # seconds
    output_file='capture.log',
    filter_ids=[0x123, 0x456]  # Optional: specific CAN IDs
)

# Analyze captured data
analysis = sniffer.analyze_traffic()
print(f"Unique IDs: {analysis['unique_ids']}")
print(f"Message rate: {analysis['messages_per_second']}")
print(f"Anomalies detected: {analysis['anomalies']}")
```

### 2. Message Fuzzer

Fuzz CAN messages to identify vulnerabilities:

```python
from s800.fuzzing import CANFuzzer
from s800.payloads import FuzzPayload

# Initialize fuzzer
fuzzer = CANFuzzer(interface='can0')

# Bit-flip fuzzing
fuzzer.bit_flip_fuzz(
    can_id=0x123,
    base_payload=b'\x01\x02\x03\x04\x05\x06\x07\x08',
    iterations=1000,
    delay=0.01  # seconds between messages
)

# Random payload fuzzing
fuzzer.random_fuzz(
    can_id_range=(0x100, 0x200),
    payload_length=8,
    iterations=5000,
    log_responses=True
)

# Mutation-based fuzzing
base_messages = sniffer.get_baseline_messages()
fuzzer.mutation_fuzz(
    base_messages=base_messages,
    mutation_rate=0.3,
    iterations=2000
)
```

### 3. Replay Attacks

Replay captured CAN messages:

```python
from s800.replay import MessageReplay

# Load captured messages
replay = MessageReplay(interface='can0')
replay.load_capture('capture.log')

# Basic replay
replay.replay_all(
    speed_multiplier=1.0,  # Real-time
    loop=False
)

# Selective replay
replay.replay_filtered(
    can_ids=[0x123, 0x456],
    time_range=(10.0, 30.0),  # seconds from capture start
    speed_multiplier=2.0  # 2x speed
)

# Modified replay
replay.replay_modified(
    can_id=0x123,
    payload_modifier=lambda p: bytes([b ^ 0xFF for b in p]),  # Invert bits
    interval=0.1
)
```

### 4. Diagnostic Services (UDS)

Interact with ECUs using UDS protocol:

```python
from s800.diagnostics import UDSClient
from s800.diagnostics.services import *

# Initialize UDS client
uds = UDSClient(
    interface='can0',
    request_id=0x7DF,  # Broadcast or specific ECU
    response_id=0x7E8
)

# Read ECU information
ecu_info = uds.read_data_by_identifier(0xF190)  # VIN
print(f"VIN: {ecu_info.decode('ascii')}")

# Security access (seed/key)
seed = uds.security_access_request_seed(level=0x01)
key = calculate_key(seed)  # Implement your key algorithm
uds.security_access_send_key(level=0x02, key=key)

# Read diagnostic trouble codes
dtcs = uds.read_dtc_information()
for dtc in dtcs:
    print(f"DTC: {dtc['code']} - {dtc['status']}")

# Write data (if authorized)
uds.write_data_by_identifier(0x0110, b'\x01\x02\x03\x04')
```

### 5. ECU Simulator

Create virtual ECUs for testing:

```python
from s800.simulation import ECUSimulator

# Create simulated ECU
ecu = ECUSimulator(
    interface='vcan0',  # Virtual CAN interface
    ecu_id=0x7E8,
    response_to=0x7DF
)

# Define response behaviors
@ecu.on_message(can_id=0x123)
def handle_request(msg):
    # Custom logic
    return {
        'can_id': 0x456,
        'data': b'\xAA\xBB\xCC\xDD\x00\x00\x00\x00'
    }

# Add UDS service support
ecu.add_uds_service(
    service_id=0x22,  # ReadDataByIdentifier
    did=0xF190,
    response=b'TEST_VIN_12345678'
)

# Start simulator
ecu.start()
```

## Configuration

### Framework Configuration

Create `s800_config.yaml`:

```yaml
# Network interfaces
interfaces:
  can0:
    type: socketcan
    bitrate: 500000
    protocol: CAN
  can1:
    type: socketcan
    bitrate: 250000
    protocol: CAN-FD

# Logging
logging:
  level: INFO
  output_dir: ./logs
  format: json

# Fuzzing defaults
fuzzing:
  max_iterations: 10000
  delay_ms: 10
  stop_on_error: false
  blacklist_ids: [0x000, 0x7FF]

# Security
security:
  require_auth: true
  allowed_operations: [read, fuzz, replay]
  restricted_ids: [0x600, 0x601, 0x602]

# Analysis
analysis:
  anomaly_detection: true
  baseline_threshold: 0.15
  report_format: html
```

Load configuration:

```python
from s800.config import load_config

config = load_config('s800_config.yaml')
sniffer = CANSniffer(
    interface=config['interfaces']['can0']['type'],
    bitrate=config['interfaces']['can0']['bitrate']
)
```

## Common Testing Patterns

### Full Security Assessment

```python
from s800.assessment import SecurityAssessment

# Initialize assessment
assessment = SecurityAssessment(
    interface='can0',
    target_ecus=[0x7E8, 0x7E9, 0x7EA]
)

# Phase 1: Reconnaissance
assessment.discover_ecus()
assessment.enumerate_services()
assessment.identify_protocol_version()

# Phase 2: Vulnerability scanning
assessment.test_authentication_bypass()
assessment.test_message_injection()
assessment.test_dos_vulnerabilities()

# Phase 3: Exploitation attempts
assessment.test_unauthorized_write()
assessment.test_firmware_manipulation()

# Generate report
report = assessment.generate_report(
    format='html',
    output='security_assessment.html',
    include_recommendations=True
)
```

### Traffic Baseline and Anomaly Detection

```python
from s800.analysis import TrafficAnalyzer

analyzer = TrafficAnalyzer(interface='can0')

# Establish baseline (normal operation)
print("Recording baseline - operate vehicle normally for 60 seconds")
baseline = analyzer.record_baseline(duration=60)
analyzer.save_baseline('baseline.json')

# Monitor for anomalies
analyzer.start_monitoring(
    baseline='baseline.json',
    callback=lambda anomaly: print(f"Anomaly detected: {anomaly}")
)

# Statistical analysis
stats = analyzer.get_statistics()
print(f"Average message rate: {stats['avg_rate']}")
print(f"Standard deviation: {stats['std_dev']}")
print(f"Suspicious patterns: {stats['suspicious']}")
```

### Automated Fuzzing Campaign

```python
from s800.campaigns import FuzzingCampaign

campaign = FuzzingCampaign(
    interface='can0',
    name='ECU_Stability_Test'
)

# Add fuzzing strategies
campaign.add_strategy('bit_flip', can_ids=[0x100, 0x101], iterations=1000)
campaign.add_strategy('random', can_id_range=(0x200, 0x2FF), iterations=5000)
campaign.add_strategy('boundary', can_ids=[0x300], iterations=500)

# Set monitoring
campaign.monitor_ecu_health([0x7E8, 0x7E9])
campaign.set_crash_detection(timeout=5.0)

# Execute campaign
results = campaign.execute(
    auto_recovery=True,
    continue_on_crash=False,
    save_crashes=True
)

# Analyze results
print(f"Total messages sent: {results['total_sent']}")
print(f"Crashes detected: {results['crashes']}")
print(f"Reproducible bugs: {results['reproducible']}")
```

## Advanced Features

### Custom Protocol Implementation

```python
from s800.protocols import BaseProtocol

class CustomProtocol(BaseProtocol):
    def __init__(self, interface):
        super().__init__(interface)
        self.protocol_name = "CustomAutomotive"
    
    def encode_message(self, msg_type, payload):
        # Custom encoding logic
        header = msg_type.to_bytes(1, 'big')
        length = len(payload).to_bytes(1, 'big')
        checksum = self.calculate_checksum(header + length + payload)
        return header + length + payload + checksum
    
    def decode_message(self, raw_data):
        # Custom decoding logic
        return {
            'type': raw_data[0],
            'length': raw_data[1],
            'payload': raw_data[2:-1],
            'checksum': raw_data[-1]
        }
```

### DBC File Integration

```python
from s800.formats import DBCParser

# Load DBC file (CAN database)
dbc = DBCParser('vehicle.dbc')

# Decode message using DBC
raw_msg = {'can_id': 0x123, 'data': b'\x01\x02\x03\x04\x05\x06\x07\x08'}
decoded = dbc.decode_message(raw_msg)
print(f"Speed: {decoded['Speed']} km/h")
print(f"RPM: {decoded['EngineRPM']} rpm")

# Encode message using DBC
msg = dbc.encode_message(
    'EngineData',
    {'Speed': 80, 'EngineRPM': 3000}
)
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Bring up interface manually
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up

# Check for errors
dmesg | grep can
```

### Permission Denied

```bash
# Add user to required groups
sudo usermod -aG dialout $USER
sudo usermod -aG can $USER

# Reload groups
newgrp dialout

# Or run with sudo (not recommended for production)
sudo python3 your_script.py
```

### No Messages Received

```python
# Verify interface is active
from s800.utils import verify_interface

if not verify_interface('can0'):
    print("Interface not ready")

# Check bitrate matches network
# Common automotive bitrates: 125000, 250000, 500000, 1000000

# Enable promiscuous mode
sniffer = CANSniffer(interface='can0', promiscuous=True)
```

### High CPU Usage During Fuzzing

```python
# Reduce fuzzing rate
fuzzer.set_throttle(messages_per_second=100)

# Use batch mode
fuzzer.enable_batch_mode(batch_size=50, batch_delay=0.1)

# Limit iterations
fuzzer.set_max_iterations(1000)
```

## Environment Variables

```bash
# Interface configuration
export S800_INTERFACE=can0
export S800_BITRATE=500000

# Logging
export S800_LOG_LEVEL=DEBUG
export S800_LOG_DIR=/var/log/s800

# Security
export S800_AUTH_TOKEN=your_auth_token_here
export S800_RESTRICTED_MODE=true

# Performance
export S800_MAX_THREADS=4
export S800_BUFFER_SIZE=1024
```

## Safety Warnings

- **Never test on production vehicles** without proper authorization
- **Always use in controlled environments** (test benches, simulators)
- **Fuzzing can cause ECU crashes** - ensure safe shutdown procedures
- **Some operations may be illegal** without proper authorization
- **Keep emergency stop mechanisms** readily available
