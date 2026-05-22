---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for CAN bus, LIN, FlexRay, and automotive protocol penetration testing
triggers:
  - test vehicle CAN bus security
  - perform automotive network penetration testing
  - simulate vehicle network attacks
  - analyze car network protocols
  - fuzz automotive ECU messages
  - scan vehicle security vulnerabilities
  - test in-vehicle network security
  - perform CAN bus fuzzing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

S800 is a comprehensive security testing framework designed for automotive and vehicle network security assessment. It supports testing of CAN bus, LIN bus, FlexRay, and other in-vehicle network protocols, enabling security researchers and automotive engineers to identify vulnerabilities, perform penetration testing, and validate security controls in modern vehicles.

## What It Does

- **CAN Bus Testing**: Sniff, inject, and fuzz CAN frames
- **Protocol Analysis**: Support for CAN, LIN, FlexRay, and automotive Ethernet
- **Fuzzing**: Generate and send malformed messages to test ECU resilience
- **Replay Attacks**: Capture and replay vehicle network traffic
- **DoS Testing**: Test denial-of-service vulnerabilities in vehicle networks
- **UDS/OBD-II**: Diagnostic protocol security testing
- **Man-in-the-Middle**: Intercept and modify in-vehicle communications

## Installation

### Prerequisites

```bash
# Install required hardware drivers (SocketCAN on Linux)
sudo apt-get update
sudo apt-get install can-utils python3-can

# Load kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Install S800 Framework

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip3 install -r requirements.txt

# Install the framework
python3 setup.py install
```

### Hardware Setup

```bash
# Setup virtual CAN interface (for testing)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Setup physical CAN interface (e.g., CAN-USB adapter)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Configuration

### Basic Configuration File

Create `config.yaml`:

```yaml
# S800 Configuration
interface:
  type: socketcan  # socketcan, kvaser, pcan, vector
  channel: can0    # or vcan0 for virtual
  bitrate: 500000  # 125000, 250000, 500000, 1000000

logging:
  level: INFO
  output: logs/s800.log
  capture: true

testing:
  timeout: 5
  retry: 3
  fuzzing:
    iterations: 10000
    seed: 42

targets:
  - ecu_id: 0x7DF  # OBD-II broadcast
    services:
      - 0x09  # Request vehicle info
      - 0x22  # Read data by ID
  - ecu_id: 0x720  # Example ECU
    name: "Engine Control Unit"
```

### Environment Variables

```bash
# Set interface
export S800_INTERFACE=can0
export S800_BITRATE=500000

# Logging
export S800_LOG_LEVEL=DEBUG
export S800_LOG_FILE=/var/log/s800/testing.log

# Database for traffic capture
export S800_DB_PATH=/var/s800/captures.db
```

## Key Commands & API

### CLI Usage

```bash
# Sniff CAN bus traffic
s800 sniff --interface can0 --duration 60 --output capture.log

# Replay captured traffic
s800 replay --input capture.log --interface can0 --loop 10

# Fuzz specific CAN ID
s800 fuzz --target 0x720 --iterations 5000 --interface can0

# UDS scanner
s800 uds-scan --target 0x7E0 --services all --interface can0

# DoS attack simulation
s800 dos --target 0x123 --rate 1000 --duration 30 --interface can0

# Diagnostic session
s800 diag --target 0x7E0 --session extended --interface can0
```

### Python API - Sniffing

```python
from s800 import CANInterface, Sniffer

# Initialize interface
interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

# Create sniffer
sniffer = Sniffer(interface)

# Sniff with callback
def on_message(msg):
    print(f"ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")

sniffer.sniff(callback=on_message, duration=30)

# Sniff with filter
sniffer.sniff(
    filter_ids=[0x720, 0x730],
    callback=on_message,
    duration=60
)

interface.disconnect()
```

### Python API - Message Injection

```python
from s800 import CANInterface, Message

interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

# Send single message
msg = Message(
    arbitration_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    is_extended_id=False
)
interface.send(msg)

# Send periodic message (every 100ms)
interface.send_periodic(msg, period=0.1, duration=10)

# Burst send
for i in range(100):
    msg.data = [i & 0xFF] * 8
    interface.send(msg)

interface.disconnect()
```

### Python API - Fuzzing

```python
from s800 import Fuzzer, CANInterface

interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

# Create fuzzer
fuzzer = Fuzzer(interface)

# Simple fuzzing on CAN ID
fuzzer.fuzz_id(
    target_id=0x720,
    iterations=10000,
    delay=0.01,  # 10ms between messages
    callback=lambda msg, resp: print(f"Sent: {msg}, Response: {resp}")
)

# Data field fuzzing
fuzzer.fuzz_data_field(
    target_id=0x730,
    field_index=2,  # Fuzz 3rd byte
    min_val=0x00,
    max_val=0xFF,
    iterations=256
)

# DLC (Data Length Code) fuzzing
fuzzer.fuzz_dlc(
    target_id=0x740,
    iterations=16
)

interface.disconnect()
```

### Python API - UDS Testing

```python
from s800 import UDSClient, CANInterface

interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

# Create UDS client
uds = UDSClient(
    interface=interface,
    request_id=0x7E0,  # Tester
    response_id=0x7E8  # ECU response
)

# Start diagnostic session
response = uds.start_session(session_type=0x03)  # Extended diagnostic
print(f"Session Response: {response.hex()}")

# Read data by identifier
data = uds.read_data_by_id(did=0xF190)  # VIN
print(f"VIN: {data.decode('ascii')}")

# Security access
seed = uds.request_seed(level=0x01)
key = calculate_key(seed)  # Implement based on algorithm
uds.send_key(level=0x02, key=key)

# Write data (requires security access)
uds.write_data_by_id(did=0x1234, data=b'\x01\x02\x03\x04')

# Reset ECU
uds.ecu_reset(reset_type=0x01)  # Hard reset

interface.disconnect()
```

### Python API - Replay Attack

```python
from s800 import Replayer, CANInterface

interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

# Load capture file
replayer = Replayer(interface)
replayer.load_capture('captured_traffic.log')

# Replay with original timing
replayer.replay(preserve_timing=True)

# Replay at 2x speed
replayer.replay(speed_factor=2.0)

# Replay in loop
replayer.replay(loop=True, loop_count=10)

# Selective replay (filter by ID)
replayer.replay(
    filter_ids=[0x123, 0x456],
    preserve_timing=False,
    interval=0.01
)

interface.disconnect()
```

### Python API - DoS Testing

```python
from s800 import DoSAttack, CANInterface

interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

dos = DoSAttack(interface)

# Bus flooding
dos.flood_bus(
    rate=1000,  # messages per second
    duration=30  # seconds
)

# Target specific ECU
dos.flood_target(
    target_id=0x720,
    rate=500,
    duration=60,
    data=[0xFF] * 8
)

# Priority inversion attack
dos.priority_inversion(
    high_priority_id=0x100,
    low_priority_id=0x7FF,
    duration=30
)

interface.disconnect()
```

## Common Patterns

### Complete Security Assessment

```python
from s800 import SecurityAssessment, CANInterface

interface = CANInterface(channel='can0', bitrate=500000)
interface.connect()

# Run comprehensive assessment
assessment = SecurityAssessment(interface)

# Discovery phase
assessment.discover_ecus(timeout=60)
assessment.identify_services()

# Vulnerability scanning
assessment.scan_uds_vulnerabilities()
assessment.test_authentication_bypass()
assessment.check_replay_protection()

# Fuzzing campaign
assessment.fuzz_discovered_services(iterations=10000)

# Generate report
report = assessment.generate_report(format='html')
report.save('security_assessment.html')

interface.disconnect()
```

### Man-in-the-Middle

```python
from s800 import MITMProxy, CANInterface

# Setup two interfaces (in/out)
interface_in = CANInterface(channel='can0', bitrate=500000)
interface_out = CANInterface(channel='can1', bitrate=500000)

interface_in.connect()
interface_out.connect()

# Create MITM proxy
proxy = MITMProxy(interface_in, interface_out)

# Modify messages in transit
def modify_callback(msg):
    if msg.arbitration_id == 0x123:
        # Modify speed value
        msg.data[2] = 0x00
    return msg

proxy.set_modifier(modify_callback)

# Start proxying
proxy.start()

# Stop after 60 seconds
import time
time.sleep(60)
proxy.stop()

interface_in.disconnect()
interface_out.disconnect()
```

## Troubleshooting

### Interface Not Found

```bash
# Check available CAN interfaces
ip link show

# Verify SocketCAN modules
lsmod | grep can

# Check permissions
sudo chmod 666 /dev/can*
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo
sudo s800 sniff --interface can0
```

### No Messages Received

```python
# Verify bitrate matches vehicle network
interface = CANInterface(channel='can0', bitrate=500000)  # Try 250000 or 125000

# Check for bus errors
status = interface.get_bus_status()
print(f"TX Errors: {status.tx_errors}, RX Errors: {status.rx_errors}")

# Enable promiscuous mode
interface.set_filters([])  # Accept all messages
```

### Fuzzing Too Aggressive

```python
# Add delays between messages
fuzzer.fuzz_id(
    target_id=0x720,
    iterations=1000,
    delay=0.1,  # 100ms delay
    max_retries=3
)

# Limit message rate
fuzzer.set_rate_limit(max_msgs_per_sec=100)
```

### UDS Negative Response

```python
# Handle negative responses
try:
    response = uds.read_data_by_id(did=0xF190)
except UDSNegativeResponse as e:
    print(f"NRC: 0x{e.nrc:02X} - {e.description}")
    # Common NRCs:
    # 0x13: Incorrect message length
    # 0x31: Request out of range
    # 0x33: Security access denied
```

This skill enables AI coding agents to effectively use the S800 framework for comprehensive vehicle network security testing, from basic CAN sniffing to advanced penetration testing techniques.
