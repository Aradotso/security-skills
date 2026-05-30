---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing and attack simulation capabilities
triggers:
  - test vehicle network security
  - automotive CAN bus testing
  - vehicle network fuzzing
  - S800 security framework
  - test car network protocols
  - simulate vehicle network attacks
  - automotive security testing framework
  - CAN bus penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for automotive security researchers and penetration testers. It provides tools for testing CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols commonly found in modern vehicles. The framework enables security assessment through fuzzing, packet injection, sniffing, and attack simulation.

**Warning**: This is a test framework. Only use on authorized systems in controlled environments. Never test on production vehicles or public roads.

## Installation

### Prerequisites

- Python 3.7+
- CAN interface hardware (SocketCAN compatible, CANable, PCAN, etc.)
- Root/administrator privileges for hardware access

### Basic Installation

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework
pip install -r requirements.txt
```

### Setup CAN Interface (Linux)

```bash
# Load kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Setup virtual CAN for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# For physical CAN devices (example with slcan)
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0
```

## Core Components

### 1. CAN Bus Sniffer

Monitor and capture CAN traffic:

```python
from s800.can_sniffer import CANSniffer

# Initialize sniffer
sniffer = CANSniffer(interface='vcan0', bitrate=500000)

# Start capturing
sniffer.start_capture(
    duration=60,  # seconds
    output_file='capture.log',
    filter_ids=[0x123, 0x456]  # Optional: filter specific CAN IDs
)

# Real-time monitoring with callback
def on_message(msg):
    print(f"ID: {hex(msg.arbitration_id)}, Data: {msg.data.hex()}")

sniffer.monitor(callback=on_message)
```

### 2. CAN Fuzzer

Automated fuzzing of CAN messages:

```python
from s800.fuzzer import CANFuzzer

fuzzer = CANFuzzer(interface='vcan0')

# Basic fuzzing - random data generation
fuzzer.fuzz_random(
    can_id=0x123,
    duration=300,  # seconds
    delay=0.1      # seconds between messages
)

# Smart fuzzing - mutation-based
fuzzer.fuzz_mutate(
    base_messages='capture.log',  # Use captured traffic
    mutation_rate=0.3,
    targets=[0x123, 0x456, 0x789]
)

# Bit-flip fuzzing
fuzzer.fuzz_bitflip(
    can_id=0x123,
    base_data=b'\x00\x01\x02\x03\x04\x05\x06\x07',
    flip_positions='all'  # or specific bit positions
)
```

### 3. Packet Injection

Send crafted CAN messages:

```python
from s800.injector import CANInjector

injector = CANInjector(interface='vcan0')

# Single message injection
injector.send_message(
    can_id=0x123,
    data=b'\xDE\xAD\xBE\xEF\x00\x00\x00\x00',
    extended=False
)

# Replay attack
injector.replay_from_file(
    capture_file='capture.log',
    loop=True,
    speed_multiplier=1.0
)

# Periodic message injection
injector.send_periodic(
    can_id=0x456,
    data=b'\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF',
    interval=0.01  # 10ms
)
```

### 4. Attack Simulation

Pre-configured attack scenarios:

```python
from s800.attacks import AttackSimulator

simulator = AttackSimulator(interface='vcan0')

# DoS attack - bus flooding
simulator.dos_flood(
    can_id=0x000,
    priority='highest',
    duration=10
)

# Message suppression attack
simulator.suppress_message(
    target_id=0x123,
    duration=30
)

# ECU impersonation
simulator.impersonate_ecu(
    ecu_id=0x456,
    message_pattern='capture.log',
    modification_func=lambda data: bytes([b ^ 0xFF for b in data])
)

# UDS diagnostic attack
simulator.uds_scan(
    ecu_ids=range(0x700, 0x800),
    services=[0x10, 0x27, 0x3E]  # DiagnosticSessionControl, SecurityAccess, TesterPresent
)
```

## Configuration

### Configuration File (s800_config.yaml)

```yaml
interfaces:
  primary:
    type: socketcan
    device: vcan0
    bitrate: 500000
  
  backup:
    type: pcan
    device: PCAN_USBBUS1
    bitrate: 250000

logging:
  level: INFO
  output: logs/s800.log
  format: detailed

fuzzing:
  default_duration: 300
  min_delay: 0.001
  max_delay: 1.0
  save_crashes: true
  crash_dir: crashes/

security:
  rate_limit: 1000  # messages per second
  whitelist_ids: [0x100, 0x200, 0x300]
  blacklist_ids: []
```

### Load Configuration

```python
from s800.config import Config

config = Config.load('s800_config.yaml')
sniffer = CANSniffer(
    interface=config.interfaces.primary.device,
    bitrate=config.interfaces.primary.bitrate
)
```

## Common Patterns

### Pattern 1: Baseline + Fuzz + Compare

```python
from s800 import CANSniffer, CANFuzzer, Analyzer

# Step 1: Capture baseline behavior
sniffer = CANSniffer(interface='vcan0')
sniffer.start_capture(duration=120, output_file='baseline.log')

# Step 2: Perform fuzzing
fuzzer = CANFuzzer(interface='vcan0')
fuzzer.fuzz_random(can_id=0x123, duration=300, output_file='fuzz.log')

# Step 3: Analyze differences
analyzer = Analyzer()
differences = analyzer.compare_captures('baseline.log', 'fuzz.log')
print(f"Anomalies detected: {differences}")
```

### Pattern 2: Reverse Engineering CAN Protocol

```python
from s800 import CANSniffer, ProtocolAnalyzer

# Capture during specific actions
sniffer = CANSniffer(interface='vcan0')

# Trigger action in vehicle, then:
messages = sniffer.capture_live(duration=10)

# Analyze patterns
analyzer = ProtocolAnalyzer()
patterns = analyzer.identify_patterns(messages)

for can_id, pattern in patterns.items():
    print(f"CAN ID {hex(can_id)}:")
    print(f"  Frequency: {pattern['frequency']} Hz")
    print(f"  Data pattern: {pattern['structure']}")
    print(f"  Possible function: {pattern['hypothesis']}")
```

### Pattern 3: Security Assessment Workflow

```python
from s800 import SecurityAssessment

assessment = SecurityAssessment(interface='vcan0')

# Run comprehensive security tests
results = assessment.run_full_scan(
    tests=[
        'authentication_bypass',
        'replay_attack',
        'dos_resilience',
        'message_injection',
        'uds_security_access'
    ],
    report_format='html',
    output_file='security_report.html'
)

# Generate compliance report
assessment.generate_compliance_report(
    standards=['ISO 21434', 'SAE J3061'],
    output='compliance.pdf'
)
```

## Command Line Interface

### Sniffing

```bash
# Basic sniffing
python -m s800.cli sniff --interface vcan0 --duration 60 --output capture.log

# Filter specific IDs
python -m s800.cli sniff --interface vcan0 --filter 0x123,0x456 --verbose

# Live display
python -m s800.cli sniff --interface vcan0 --display --format table
```

### Fuzzing

```bash
# Random fuzzing
python -m s800.cli fuzz --interface vcan0 --id 0x123 --duration 300 --type random

# Mutation fuzzing from capture
python -m s800.cli fuzz --interface vcan0 --input capture.log --type mutate --rate 0.3

# Bit-flip fuzzing
python -m s800.cli fuzz --interface vcan0 --id 0x456 --type bitflip --base-data DEADBEEF
```

### Attack Simulation

```bash
# DoS attack
python -m s800.cli attack --type dos --interface vcan0 --id 0x000 --duration 10

# Replay attack
python -m s800.cli attack --type replay --interface vcan0 --input capture.log --loop

# UDS scan
python -m s800.cli attack --type uds-scan --interface vcan0 --ecu-range 0x700-0x7FF
```

## Advanced Usage

### Custom Attack Development

```python
from s800.core import BaseAttack

class CustomECUAttack(BaseAttack):
    def __init__(self, interface):
        super().__init__(interface)
    
    def execute(self, target_id, payload):
        # Custom attack logic
        self.logger.info(f"Executing custom attack on {hex(target_id)}")
        
        # Multi-stage attack
        self.send_message(target_id, b'\x01\x02\x03')
        self.wait(0.1)
        self.send_message(target_id, payload)
        
        # Monitor response
        response = self.wait_for_response(target_id + 8, timeout=1.0)
        return response

# Use custom attack
attack = CustomECUAttack(interface='vcan0')
result = attack.execute(target_id=0x7E0, payload=b'\x27\x01\x00\x00')
```

### Scripted Test Scenarios

```python
from s800 import TestScenario

scenario = TestScenario('vehicle_unlock_test')

# Define test steps
scenario.add_step('baseline', lambda: sniffer.capture(30))
scenario.add_step('trigger_unlock', lambda: injector.send_message(0x456, b'\x01'))
scenario.add_step('verify_response', lambda: analyzer.check_unlock_response())

# Run scenario
results = scenario.execute(iterations=100)
scenario.generate_report('unlock_test_report.pdf')
```

## Troubleshooting

### CAN Interface Not Found

```python
from s800.utils import list_interfaces

# List available interfaces
interfaces = list_interfaces()
print(f"Available CAN interfaces: {interfaces}")

# Verify interface is up
from s800.utils import verify_interface
if not verify_interface('vcan0'):
    print("Interface not ready. Run: sudo ip link set up vcan0")
```

### Permission Denied

```bash
# Add user to dialout group (Linux)
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python -m s800.cli sniff --interface can0
```

### No Messages Received

```python
from s800.diagnostics import CANDiagnostics

diag = CANDiagnostics(interface='vcan0')

# Check bus status
status = diag.check_bus_health()
print(f"Bus state: {status['state']}")
print(f"Error counters: {status['errors']}")
print(f"Bitrate: {status['bitrate']}")

# Test loopback
diag.test_loopback()
```

### High Error Rates

```python
# Adjust bitrate
from s800.utils import set_bitrate

set_bitrate('can0', bitrate=250000)  # Try different rates: 125k, 250k, 500k, 1M

# Enable error monitoring
sniffer = CANSniffer(interface='can0', error_monitoring=True)
errors = sniffer.get_error_stats()
print(f"Bus errors: {errors}")
```

## Environment Variables

```bash
# Set default CAN interface
export S800_DEFAULT_INTERFACE=vcan0

# Set log level
export S800_LOG_LEVEL=DEBUG

# Set output directory
export S800_OUTPUT_DIR=/var/log/s800

# Enable crash logging
export S800_CRASH_LOGGING=true
```

## Best Practices

1. **Always test in isolated environments** - use virtual CAN or isolated test benches
2. **Log everything** - maintain detailed logs for analysis and compliance
3. **Rate limiting** - avoid overwhelming the CAN bus (max ~1000 msgs/sec)
4. **Baseline first** - capture normal behavior before fuzzing
5. **Incremental testing** - start with single messages, then complex scenarios
6. **Monitor system state** - watch for ECU resets, error frames, bus-off conditions

## Safety Warnings

- Never use on production vehicles
- Never test while vehicle is in motion
- Always have emergency shutdown procedures
- Understand legal implications of automotive security testing
- Follow responsible disclosure practices for vulnerabilities
