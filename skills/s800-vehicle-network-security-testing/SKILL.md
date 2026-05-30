---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, replay, and diagnostic capabilities
triggers:
  - test vehicle network security
  - perform automotive CAN bus testing
  - fuzz vehicle network protocols
  - analyze automotive security vulnerabilities
  - test CAN bus messages
  - simulate vehicle network attacks
  - perform automotive penetration testing
  - replay vehicle network traffic
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive security testing framework designed for automotive vehicle networks. It supports multiple automotive protocols including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay. The framework enables security researchers and automotive engineers to perform penetration testing, fuzzing, traffic analysis, and vulnerability assessment on vehicle networks.

**Key Capabilities:**
- CAN/LIN/FlexRay protocol support
- Message fuzzing and injection
- Traffic capture and replay
- Diagnostic service testing (UDS/KWP2000)
- ECU simulation and emulation
- Network traffic analysis
- Security vulnerability detection

## Installation

### Prerequisites

Ensure you have the required hardware interface (CAN adapter, OBD-II dongle, or similar):

```bash
# Install system dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install can-utils python3-dev python3-pip git

# Enable SocketCAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Framework Installation

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Set up virtual CAN for testing (optional)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Configuration

```bash
# Configure physical CAN interface (example for can0 at 500kbps)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface is up
ip -details link show can0
```

## Configuration

### Framework Configuration File

Create a configuration file `config.json`:

```json
{
  "interface": "can0",
  "bitrate": 500000,
  "protocol": "CAN",
  "logging": {
    "level": "INFO",
    "output": "logs/s800.log"
  },
  "fuzzing": {
    "iterations": 10000,
    "delay_ms": 10,
    "randomize_data": true
  },
  "security": {
    "max_retries": 3,
    "timeout_ms": 1000
  }
}
```

### Environment Variables

```bash
# Set interface
export S800_INTERFACE=can0

# Set log level
export S800_LOG_LEVEL=DEBUG

# Set output directory
export S800_OUTPUT_DIR=./results
```

## Core Usage Patterns

### Basic CAN Message Sniffing

```python
from s800.can import CANInterface
from s800.sniffer import CANSniffer

# Initialize CAN interface
interface = CANInterface(interface='can0', bitrate=500000)

# Create sniffer
sniffer = CANSniffer(interface)

# Start capturing
sniffer.start()

# Capture for 30 seconds
import time
time.sleep(30)

# Stop and save
sniffer.stop()
sniffer.save_to_file('capture.log')

# Analyze captured messages
for msg in sniffer.get_messages():
    print(f"ID: {msg.arbitration_id:03X}, Data: {msg.data.hex()}")
```

### CAN Message Injection

```python
from s800.can import CANInterface, CANMessage

# Initialize interface
interface = CANInterface('can0')

# Create and send a single message
msg = CANMessage(
    arbitration_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    is_extended=False
)
interface.send(msg)

# Send periodic messages
interface.send_periodic(msg, period_ms=100)

# Stop periodic transmission
interface.stop_periodic(0x123)
```

### Protocol Fuzzing

```python
from s800.fuzzer import CANFuzzer
from s800.can import CANInterface

# Initialize fuzzer
interface = CANInterface('can0')
fuzzer = CANFuzzer(interface)

# Fuzz specific CAN ID with random data
fuzzer.fuzz_id(
    arbitration_id=0x7DF,  # OBD-II functional address
    iterations=1000,
    delay_ms=10,
    data_length=8
)

# Fuzz with mutation strategy
fuzzer.fuzz_with_mutations(
    base_message=CANMessage(0x123, [0x00] * 8),
    mutations=['bit_flip', 'byte_increment', 'random'],
    iterations=5000
)

# Smart fuzzing based on captured traffic
fuzzer.smart_fuzz(
    capture_file='normal_traffic.log',
    target_ids=[0x100, 0x200, 0x300],
    anomaly_detection=True
)
```

### UDS Diagnostic Testing

```python
from s800.diagnostics import UDSClient
from s800.can import CANInterface

# Initialize UDS client
interface = CANInterface('can0')
uds_client = UDSClient(
    interface=interface,
    request_id=0x7E0,   # Tester -> ECU
    response_id=0x7E8   # ECU -> Tester
)

# Start diagnostic session
uds_client.start_session(session_type=0x03)  # Extended diagnostic

# Read DTC (Diagnostic Trouble Codes)
dtcs = uds_client.read_dtc()
for dtc in dtcs:
    print(f"DTC: {dtc['code']}, Status: {dtc['status']}")

# Read data by identifier
vin = uds_client.read_data_by_id(0xF190)  # VIN
print(f"VIN: {vin.decode()}")

# Security access (seed/key)
seed = uds_client.request_seed(level=0x01)
key = calculate_key(seed)  # Implement your key algorithm
uds_client.send_key(key)

# Write data (requires security access)
uds_client.write_data_by_id(0x1234, b'\x00\x01\x02\x03')
```

### Traffic Replay Attack

```python
from s800.replay import TrafficReplayer
from s800.can import CANInterface

# Load captured traffic
replayer = TrafficReplayer('captured_traffic.log')

# Replay with original timing
interface = CANInterface('can0')
replayer.replay(
    interface=interface,
    preserve_timing=True,
    loop=False
)

# Replay with modifications
replayer.replay_modified(
    interface=interface,
    speed_multiplier=2.0,  # 2x speed
    filter_ids=[0x100, 0x200],  # Only replay these IDs
    modify_callback=lambda msg: msg  # Custom modification
)

# Replay attack scenario
replayer.replay_attack(
    interface=interface,
    target_id=0x123,
    inject_interval=10,  # Inject every 10th message
    inject_data=[0xFF] * 8
)
```

### ECU Simulation

```python
from s800.simulator import ECUSimulator
from s800.can import CANInterface

# Create ECU simulator
interface = CANInterface('can0')
ecu = ECUSimulator(interface, ecu_id=0x7E8)

# Define response handlers
@ecu.register_handler(service=0x22)  # ReadDataByIdentifier
def handle_read_data(request):
    did = (request.data[1] << 8) | request.data[2]
    if did == 0xF190:  # VIN
        return b'\x62\xF1\x90' + b'WAUZZZ8V0AA000001'
    return None

@ecu.register_handler(service=0x10)  # DiagnosticSessionControl
def handle_session_control(request):
    session_type = request.data[1]
    return bytes([0x50, session_type, 0x00, 0x32, 0x01, 0xF4])

# Start simulator
ecu.start()

# Run until stopped
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    ecu.stop()
```

### Security Scanning

```python
from s800.scanner import SecurityScanner
from s800.can import CANInterface

# Initialize security scanner
interface = CANInterface('can0')
scanner = SecurityScanner(interface)

# Scan for active ECUs
ecus = scanner.scan_ecus(
    id_range=(0x700, 0x7FF),
    timeout_ms=100
)
print(f"Found {len(ecus)} ECUs: {[hex(id) for id in ecus]}")

# Test for diagnostic vulnerabilities
vulns = scanner.test_diagnostic_security(
    target_ids=ecus,
    tests=['unauthorized_access', 'session_hijacking', 'seed_predictability']
)

for vuln in vulns:
    print(f"[{vuln.severity}] {vuln.description} on ECU {hex(vuln.ecu_id)}")

# Brute force security access
scanner.bruteforce_security_access(
    ecu_id=0x7E0,
    level=0x01,
    key_space=range(0x00000000, 0x0000FFFF),
    threads=4
)
```

## CLI Commands

### Basic Sniffing

```bash
# Sniff CAN traffic
python s800_cli.py sniff --interface can0 --duration 60 --output capture.log

# Sniff with filtering
python s800_cli.py sniff --interface can0 --filter "0x100-0x200" --format candump
```

### Message Injection

```bash
# Send single message
python s800_cli.py send --interface can0 --id 0x123 --data "01020304"

# Send periodic messages
python s800_cli.py send --interface can0 --id 0x123 --data "01020304" --periodic 100
```

### Fuzzing

```bash
# Fuzz specific ID
python s800_cli.py fuzz --interface can0 --target-id 0x7DF --iterations 10000

# Smart fuzzing
python s800_cli.py fuzz --interface can0 --capture normal.log --smart --anomaly-detect
```

### Diagnostic Operations

```bash
# Read VIN
python s800_cli.py uds --interface can0 --ecu 0x7E0 --read-vin

# Read DTCs
python s800_cli.py uds --interface can0 --ecu 0x7E0 --read-dtc

# Custom diagnostic request
python s800_cli.py uds --interface can0 --ecu 0x7E0 --request "10 03"
```

### Replay Attacks

```bash
# Replay traffic
python s800_cli.py replay --interface can0 --input capture.log --preserve-timing

# Replay with speed modification
python s800_cli.py replay --interface can0 --input capture.log --speed 2.0
```

## Troubleshooting

### CAN Interface Not Found

```python
# Check available interfaces
import os
interfaces = os.listdir('/sys/class/net/')
can_interfaces = [i for i in interfaces if i.startswith('can')]
print(f"Available CAN interfaces: {can_interfaces}")

# Verify interface is up
from s800.utils import check_interface
if not check_interface('can0'):
    print("Interface can0 is not up. Run: sudo ip link set up can0")
```

### Permission Denied

```bash
# Add user to dialout/can group
sudo usermod -a -G dialout $USER
sudo usermod -a -G can $USER

# Set capabilities for raw socket access
sudo setcap cap_net_raw+ep /usr/bin/python3
```

### No Messages Received

```python
from s800.diagnostics import InterfaceTest

# Test interface connectivity
tester = InterfaceTest('can0')

# Check for bus activity
if not tester.detect_activity(timeout=5):
    print("No CAN bus activity detected")
    print("- Check physical connections")
    print("- Verify bitrate matches network")
    print("- Check termination resistors")

# Test loopback
tester.loopback_test()
```

### Bitrate Mismatch

```python
from s800.utils import detect_bitrate

# Auto-detect bitrate
detected = detect_bitrate('can0', candidates=[125000, 250000, 500000, 1000000])
print(f"Detected bitrate: {detected}")

# Reconfigure interface
import subprocess
subprocess.run(['sudo', 'ip', 'link', 'set', 'can0', 'type', 'can', 'bitrate', str(detected)])
subprocess.run(['sudo', 'ip', 'link', 'set', 'up', 'can0'])
```

## Advanced Patterns

### Custom Protocol Handler

```python
from s800.protocol import ProtocolHandler

class CustomProtocolHandler(ProtocolHandler):
    def __init__(self, interface):
        super().__init__(interface)
        
    def send_request(self, service, data):
        msg = self.build_message(service, data)
        self.interface.send(msg)
        return self.wait_for_response(timeout_ms=1000)
    
    def parse_response(self, msg):
        if msg.data[0] == 0x7F:  # Negative response
            return {
                'status': 'error',
                'nrc': msg.data[2],
                'description': self.get_nrc_description(msg.data[2])
            }
        return {'status': 'success', 'data': msg.data[1:]}
```

### Multi-Interface Testing

```python
from s800.can import CANInterface
import threading

def monitor_interface(interface_name):
    interface = CANInterface(interface_name)
    sniffer = CANSniffer(interface)
    sniffer.start()
    # Monitor logic here

# Monitor multiple interfaces simultaneously
threads = []
for iface in ['can0', 'can1', 'can2']:
    t = threading.Thread(target=monitor_interface, args=(iface,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
```

## Best Practices

1. **Always test on isolated networks first** - Use virtual CAN or bench setups
2. **Implement rate limiting** - Prevent bus flooding and ECU damage
3. **Log all operations** - Maintain audit trails for security testing
4. **Validate responses** - Check for negative responses and errors
5. **Use environment variables** - Keep sensitive configuration out of code
6. **Handle exceptions gracefully** - Network operations can fail unexpectedly

## Safety Warnings

⚠️ **WARNING**: This framework can send arbitrary messages to vehicle networks. Improper use may cause:
- ECU malfunctions
- Vehicle safety system failures
- Physical damage to components
- Violation of local laws

Always obtain proper authorization before testing on production vehicles.
