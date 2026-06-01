---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, replay, and analysis capabilities
triggers:
  - test vehicle CAN bus security
  - analyze automotive network traffic
  - fuzz vehicle ECU communications
  - replay CAN bus messages
  - scan for vehicle network vulnerabilities
  - monitor automotive network protocols
  - simulate vehicle network attacks
  - test ECU security posture
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for fuzzing, message replay, traffic analysis, and vulnerability assessment of Electronic Control Units (ECUs) in vehicles.

**Primary use cases:**
- Security testing of automotive ECUs
- CAN bus traffic monitoring and analysis
- Protocol fuzzing for vulnerability discovery
- Message injection and replay attacks
- Network topology discovery
- Security compliance validation

## Installation

### Prerequisites

```bash
# Required hardware: CAN interface (USB-to-CAN adapter)
# Supported adapters: SocketCAN, PCAN, IXXAT, Kvaser

# Install dependencies (Linux)
sudo apt-get update
sudo apt-get install can-utils python3-pip python3-dev

# Load kernel modules for SocketCAN
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Framework Setup

```bash
# Clone repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Setup virtual CAN for testing (optional)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Configuration

```bash
# Configure physical CAN interface (example for can0)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ifconfig can0
candump can0
```

## Core Components

### 1. CAN Bus Scanner

Discover active CAN IDs and analyze traffic patterns.

```python
from s800.scanner import CANScanner
from s800.interface import CANInterface

# Initialize CAN interface
interface = CANInterface(channel='can0', bustype='socketcan', bitrate=500000)

# Create scanner instance
scanner = CANScanner(interface)

# Scan for active CAN IDs (passive monitoring)
active_ids = scanner.scan(duration=30, mode='passive')
print(f"Discovered {len(active_ids)} active CAN IDs: {active_ids}")

# Analyze traffic statistics
stats = scanner.get_statistics()
for can_id, info in stats.items():
    print(f"ID 0x{can_id:03X}: {info['count']} messages, "
          f"interval: {info['avg_interval']:.3f}s")
```

### 2. Message Fuzzing

Fuzz CAN messages to discover vulnerabilities.

```python
from s800.fuzzer import CANFuzzer
from s800.payloads import PayloadGenerator

# Initialize fuzzer
fuzzer = CANFuzzer(interface)

# Basic fuzzing - random data
fuzzer.fuzz_random(
    target_id=0x123,
    duration=60,
    interval=0.1,
    data_length=8
)

# Smart fuzzing with payload generator
payload_gen = PayloadGenerator()

# Fuzz with specific patterns
fuzzer.fuzz_pattern(
    target_id=0x456,
    payloads=payload_gen.generate([
        'sequential',      # 0x00, 0x01, 0x02...
        'boundary',        # 0x00, 0xFF, 0x7F, 0x80
        'bit_flip',        # Single bit flips
        'known_values'     # Common control values
    ]),
    delay=0.05
)

# Mutation-based fuzzing from captured traffic
baseline = scanner.capture(duration=10)
fuzzer.fuzz_mutation(
    baseline_messages=baseline,
    mutation_rate=0.3,
    iterations=1000
)
```

### 3. Message Replay

Capture and replay CAN traffic for testing.

```python
from s800.replay import CANReplay
from s800.capture import CANCapture

# Capture traffic
capture = CANCapture(interface)
messages = capture.record(
    duration=60,
    filter_ids=[0x100, 0x200, 0x300],  # Optional filter
    output_file='session_001.log'
)

# Replay captured messages
replay = CANReplay(interface)

# Replay with original timing
replay.replay_file(
    'session_001.log',
    preserve_timing=True,
    loop=False
)

# Replay with modified timing
replay.replay_file(
    'session_001.log',
    speedup=2.0,  # 2x faster
    loop=True,
    loop_count=5
)

# Replay single message repeatedly
replay.replay_message(
    can_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    interval=0.1,
    count=100
)
```

### 4. Protocol Analysis

Analyze and decode vehicle network protocols.

```python
from s800.analyzer import ProtocolAnalyzer
from s800.decoders import UDSDecoder, OBDDecoder

# Initialize analyzer
analyzer = ProtocolAnalyzer(interface)

# Decode UDS (Unified Diagnostic Services) messages
uds_decoder = UDSDecoder()
messages = capture.get_messages(filter_id=0x7DF)  # OBD-II functional address

for msg in messages:
    decoded = uds_decoder.decode(msg)
    if decoded:
        print(f"Service: {decoded['service']}, "
              f"SubFunction: {decoded.get('subfunction', 'N/A')}, "
              f"Data: {decoded['data']}")

# OBD-II analysis
obd_decoder = OBDDecoder()
obd_response = obd_decoder.decode_response(
    mode=0x01,  # Show current data
    pid=0x0C,   # Engine RPM
    data=[0x1A, 0xF8]
)
print(f"Engine RPM: {obd_response['value']} rpm")

# Identify protocol patterns
patterns = analyzer.identify_patterns(
    messages=messages,
    min_occurrences=10
)
for pattern in patterns:
    print(f"Pattern: {pattern['type']}, Confidence: {pattern['confidence']}")
```

### 5. ECU Fingerprinting

Identify and fingerprint ECUs on the network.

```python
from s800.fingerprint import ECUFingerprinter

fingerprinter = ECUFingerprinter(interface)

# Discover ECUs via diagnostic services
ecus = fingerprinter.discover_ecus(
    methods=['uds', 'obd', 'passive'],
    timeout=30
)

for ecu in ecus:
    print(f"ECU at 0x{ecu['address']:03X}:")
    print(f"  Type: {ecu.get('type', 'Unknown')}")
    print(f"  Vendor: {ecu.get('vendor', 'Unknown')}")
    print(f"  Part Number: {ecu.get('part_number', 'N/A')}")
    print(f"  Software Version: {ecu.get('sw_version', 'N/A')}")

# Read ECU information via UDS
ecu_info = fingerprinter.read_ecu_info(
    address=0x7E8,
    identifiers=[
        0xF187,  # Spare Part Number
        0xF189,  # Application Software Identification
        0xF18A,  # Application Data Identification
        0xF191,  # ECU Hardware Number
    ]
)
```

### 6. Vulnerability Scanner

Automated security assessment.

```python
from s800.scanner import VulnerabilityScanner

vuln_scanner = VulnerabilityScanner(interface)

# Run comprehensive security scan
results = vuln_scanner.scan(
    targets='auto',  # Auto-discover targets
    checks=[
        'authentication',      # Check for missing auth
        'replay_protection',   # Test replay attack resilience
        'fuzzing',            # Basic fuzzing tests
        'dos',                # Denial of Service tests
        'diagnostic_access',  # Unauthorized diagnostic access
        'seed_key',           # Seed-key security
    ],
    timeout=300
)

# Review findings
for finding in results['vulnerabilities']:
    print(f"\n[{finding['severity']}] {finding['title']}")
    print(f"Target: 0x{finding['target_id']:03X}")
    print(f"Description: {finding['description']}")
    print(f"Recommendation: {finding['recommendation']}")

# Generate report
results.export_report('scan_results.json', format='json')
results.export_report('scan_results.html', format='html')
```

## Configuration

### Configuration File (s800_config.yaml)

```yaml
interface:
  type: socketcan
  channel: can0
  bitrate: 500000
  
fuzzer:
  default_interval: 0.01
  max_iterations: 10000
  mutation_rate: 0.2
  
scanner:
  scan_duration: 30
  passive_mode: true
  statistics_enabled: true
  
capture:
  max_buffer_size: 100000
  auto_save: true
  output_dir: ./captures/
  
logging:
  level: INFO
  file: s800.log
  console: true
  
safety:
  emergency_stop_id: 0x7FF
  watchdog_enabled: true
  max_message_rate: 1000  # messages/sec
```

### Load Configuration

```python
from s800.config import Config

# Load configuration
config = Config.load('s800_config.yaml')

# Initialize with config
interface = CANInterface.from_config(config)
fuzzer = CANFuzzer(interface, config=config.fuzzer)
```

## Common Testing Workflows

### Passive Reconnaissance

```python
from s800 import S800Framework

# Initialize framework
s800 = S800Framework(channel='can0')

# Step 1: Passive traffic analysis
print("[*] Starting passive reconnaissance...")
s800.scanner.scan(duration=120, mode='passive')

# Step 2: Identify periodic messages (likely sensors/status)
periodic = s800.analyzer.find_periodic_messages(tolerance=0.05)

# Step 3: Identify event-driven messages
events = s800.analyzer.find_event_messages()

# Step 4: Build network map
network_map = s800.mapper.create_map(
    periodic_messages=periodic,
    event_messages=events
)
network_map.visualize('network_topology.png')
```

### Active Security Testing

```python
# Step 1: Baseline capture
baseline = s800.capture.record(duration=60, label='baseline')

# Step 2: ECU discovery
ecus = s800.fingerprinter.discover_ecus()

# Step 3: Test each ECU
for ecu in ecus:
    print(f"[*] Testing ECU 0x{ecu['address']:03X}")
    
    # Test diagnostic access
    diag_result = s800.vuln_scanner.test_diagnostic_access(ecu['address'])
    
    # Test authentication
    auth_result = s800.vuln_scanner.test_authentication(ecu['address'])
    
    # Controlled fuzzing
    s800.fuzzer.fuzz_smart(
        target_id=ecu['address'],
        iterations=100,
        monitor_ids=ecu.get('related_ids', [])
    )

# Step 4: Compare with baseline
anomalies = s800.analyzer.compare_with_baseline(baseline)
```

## Troubleshooting

### CAN Interface Issues

```python
from s800.diagnostics import InterfaceDiagnostics

diag = InterfaceDiagnostics()

# Check interface status
status = diag.check_interface('can0')
if not status['up']:
    print(f"Interface down. Error: {status['error']}")
    # Auto-fix common issues
    diag.fix_interface('can0', bitrate=500000)

# Monitor for bus errors
errors = diag.monitor_bus_errors(duration=10)
if errors['error_rate'] > 0.01:
    print(f"High error rate detected: {errors['error_rate']:.2%}")
    print(f"Suggested bitrate: {errors['suggested_bitrate']}")
```

### Permission Errors

```bash
# Add user to required groups (Linux)
sudo usermod -a -G dialout $USER
sudo usermod -a -G can $USER

# Set interface permissions
sudo chmod 666 /dev/can0
```

### Message Filtering

```python
# Filter by ID range
filtered = capture.filter_messages(
    id_range=(0x100, 0x3FF),
    exclude_ids=[0x200, 0x201]
)

# Filter by content pattern
pattern_match = capture.filter_by_pattern(
    pattern=b'\x00\x00.*\xFF',
    regex=True
)
```

## Safety Considerations

**⚠️ IMPORTANT:** This framework is for authorized security testing only.

```python
from s800.safety import SafetyManager

# Initialize safety manager
safety = SafetyManager(interface)

# Set emergency stop
safety.set_emergency_stop(
    trigger_id=0x7FF,
    trigger_data=b'\xFF' * 8
)

# Enable watchdog
safety.enable_watchdog(
    max_rate=1000,  # max messages/sec
    callback=lambda: print("EMERGENCY STOP TRIGGERED")
)

# Use context manager for safe testing
with safety.safe_testing_mode():
    # Perform tests - auto-stops on anomalies
    fuzzer.fuzz_random(target_id=0x123, duration=30)
```

## Environment Variables

```bash
# Optional configuration via environment
export S800_INTERFACE=can0
export S800_BITRATE=500000
export S800_LOG_LEVEL=DEBUG
export S800_OUTPUT_DIR=/var/log/s800/
export S800_ENABLE_SAFETY=true
```

Use in code:

```python
import os
from s800 import S800Framework

s800 = S800Framework(
    channel=os.getenv('S800_INTERFACE', 'vcan0'),
    bitrate=int(os.getenv('S800_BITRATE', '500000'))
)
```
