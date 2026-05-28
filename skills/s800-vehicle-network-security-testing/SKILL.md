---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for automotive CAN bus, LIN, and FlexRay protocol analysis and penetration testing
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive penetration testing
  - use S800 framework
  - test vehicle ECU security
  - analyze automotive network protocols
  - scan car network vulnerabilities
  - test FlexRay or LIN protocols
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive vehicle network security testing framework designed for automotive security researchers and penetration testers. It provides tools for analyzing, testing, and securing CAN bus, LIN, and FlexRay protocols commonly found in modern vehicles.

**Key Features:**
- CAN bus traffic analysis and manipulation
- LIN protocol testing
- FlexRay network analysis
- ECU vulnerability scanning
- Message injection and replay attacks
- Protocol fuzzing capabilities
- Real-time network monitoring

## Installation

### Prerequisites

```bash
# Install required dependencies
sudo apt-get update
sudo apt-get install -y python3 python3-pip can-utils git

# Install SocketCAN kernel modules (Linux)
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

### Hardware Setup (Optional)

For real vehicle testing, compatible CAN adapters:
- CANtact
- PEAK PCAN-USB
- Kvaser interfaces
- Arduino with MCP2515 CAN shield

## Configuration

### Basic Configuration

Create a configuration file `config.json`:

```json
{
  "interface": "can0",
  "bitrate": 500000,
  "log_level": "INFO",
  "output_dir": "./logs",
  "database": {
    "enabled": true,
    "path": "./dbc/vehicle.dbc"
  },
  "fuzzing": {
    "delay": 0.01,
    "iterations": 1000
  }
}
```

### DBC File Setup

DBC (Database CAN) files define CAN message formats:

```bash
# Place your vehicle-specific DBC files in the dbc/ directory
mkdir -p dbc
# Example: dbc/toyota_camry_2020.dbc
```

### Environment Variables

```bash
# Set interface name
export S800_CAN_INTERFACE=can0

# Set logging level
export S800_LOG_LEVEL=DEBUG

# Database path
export S800_DBC_PATH=/path/to/vehicle.dbc

# Output directory
export S800_OUTPUT_DIR=/var/log/s800
```

## Core Usage

### CAN Bus Sniffing

```python
from s800.can import CANSniffer
from s800.config import Config

# Initialize configuration
config = Config.load('config.json')

# Create sniffer instance
sniffer = CANSniffer(
    interface=config.interface,
    bitrate=config.bitrate
)

# Start capturing traffic
sniffer.start()

# Capture for 60 seconds
messages = sniffer.capture(duration=60)

# Analyze captured messages
for msg in messages:
    print(f"ID: {msg.arbitration_id:#x} Data: {msg.data.hex()}")

# Stop sniffer
sniffer.stop()
```

### Message Injection

```python
from s800.can import CANInjector
import can

# Initialize injector
injector = CANInjector(interface='can0')

# Create a CAN message
message = can.Message(
    arbitration_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    is_extended_id=False
)

# Send single message
injector.send(message)

# Send repeated messages
injector.send_periodic(
    message=message,
    period=0.1,  # 100ms interval
    duration=10  # for 10 seconds
)
```

### Replay Attack

```python
from s800.attacks import ReplayAttack

# Load captured traffic
attack = ReplayAttack(pcap_file='./logs/capture.pcap')

# Replay all messages
attack.replay(
    interface='can0',
    speed=1.0  # 1.0 = real-time, 2.0 = 2x speed
)

# Replay specific message IDs
attack.replay_filtered(
    interface='can0',
    message_ids=[0x123, 0x456],
    loop=True  # Continuous replay
)
```

### Fuzzing

```python
from s800.fuzzing import CANFuzzer
from s800.fuzzing.strategies import RandomByteFlip

# Initialize fuzzer
fuzzer = CANFuzzer(
    interface='can0',
    strategy=RandomByteFlip()
)

# Target specific message ID
fuzzer.fuzz_message(
    arbitration_id=0x123,
    iterations=1000,
    delay=0.01
)

# Intelligent fuzzing with DBC awareness
fuzzer.fuzz_with_dbc(
    dbc_path='./dbc/vehicle.dbc',
    signal_name='EngineSpeed',
    min_value=0,
    max_value=8000
)
```

## CLI Commands

### Traffic Analysis

```bash
# Capture CAN traffic
python3 s800.py capture --interface can0 --duration 60 --output capture.pcap

# Display live traffic
python3 s800.py monitor --interface can0 --filter 0x100-0x200

# Analyze PCAP file
python3 s800.py analyze --pcap capture.pcap --dbc ./dbc/vehicle.dbc
```

### Security Testing

```bash
# Scan for active ECUs
python3 s800.py scan --interface can0 --range 0x000-0x7FF

# Fuzz specific message ID
python3 s800.py fuzz --interface can0 --id 0x123 --iterations 1000

# Replay attack
python3 s800.py replay --pcap capture.pcap --interface can0 --speed 1.0
```

### Protocol Testing

```bash
# Test UDS diagnostics
python3 s800.py uds --interface can0 --ecu 0x7E0 --service ReadDTC

# Test XCP protocol
python3 s800.py xcp --interface can0 --connect --slave 0x100
```

## Advanced Patterns

### ECU Discovery and Enumeration

```python
from s800.discovery import ECUScanner

# Scan for active ECUs
scanner = ECUScanner(interface='can0')
ecus = scanner.scan(id_range=(0x000, 0x7FF))

for ecu in ecus:
    print(f"Found ECU: ID={ecu.id:#x}, Type={ecu.type}")
    
    # Try to identify ECU via UDS
    if ecu.supports_uds():
        vin = ecu.read_vin()
        print(f"  VIN: {vin}")
```

### Message Filtering and Analysis

```python
from s800.analyzer import MessageAnalyzer
from s800.filters import IDFilter, FrequencyFilter

# Create analyzer with filters
analyzer = MessageAnalyzer(pcap_file='capture.pcap')

# Filter by ID range
id_filter = IDFilter(min_id=0x100, max_id=0x200)
messages = analyzer.filter(id_filter)

# Find periodic messages
freq_filter = FrequencyFilter(min_frequency=10, max_frequency=100)
periodic = analyzer.find_periodic(freq_filter)

# Statistical analysis
stats = analyzer.get_statistics()
print(f"Unique IDs: {stats.unique_ids}")
print(f"Total messages: {stats.total_count}")
print(f"Average rate: {stats.avg_rate} msg/s")
```

### Denial of Service Testing

```python
from s800.attacks import DoSAttack

# Bus flooding attack
dos = DoSAttack(interface='can0')

# Flood with high-priority messages
dos.flood(
    arbitration_id=0x000,  # Highest priority
    data=[0xFF] * 8,
    rate=1000  # messages per second
)

# Selective DoS on specific ECU
dos.target_ecu(
    target_id=0x123,
    duration=30
)
```

### Custom Protocol Handler

```python
from s800.protocols import BaseProtocol

class CustomProtocol(BaseProtocol):
    def __init__(self, interface):
        super().__init__(interface)
        self.name = "CustomProto"
    
    def parse_message(self, message):
        """Parse custom protocol message"""
        if message.arbitration_id & 0x700 == 0x600:
            return {
                'type': 'custom',
                'command': message.data[0],
                'payload': message.data[1:]
            }
        return None
    
    def build_message(self, command, payload):
        """Build custom protocol message"""
        return can.Message(
            arbitration_id=0x600,
            data=[command] + list(payload),
            is_extended_id=False
        )

# Use custom protocol
protocol = CustomProtocol(interface='can0')
msg = protocol.build_message(0x01, [0x02, 0x03])
protocol.send(msg)
```

## Troubleshooting

### CAN Interface Issues

```bash
# Check if interface is up
ip link show can0

# Bring interface up with correct bitrate
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Check for errors
ip -details -statistics link show can0

# Reset interface
sudo ip link set down can0
sudo ip link set up can0
```

### Permission Errors

```bash
# Add user to dialout group for device access
sudo usermod -a -G dialout $USER

# Set capabilities for raw socket access
sudo setcap cap_net_raw+ep /usr/bin/python3

# Or run with sudo (not recommended for production)
sudo python3 s800.py capture --interface can0
```

### No Traffic Detected

```python
# Verify interface is receiving
from s800.diagnostics import InterfaceDiagnostics

diag = InterfaceDiagnostics('can0')
if not diag.is_active():
    print("Interface not active or no traffic")
    print(f"Error state: {diag.get_error_state()}")

# Check termination resistor (120Ω required on CAN bus)
# Verify physical connections
```

### DBC Parsing Errors

```python
from s800.dbc import DBCParser

try:
    parser = DBCParser('vehicle.dbc')
    db = parser.parse()
except Exception as e:
    print(f"DBC parse error: {e}")
    # Use generic parsing without DBC
    from s800.can import GenericParser
    parser = GenericParser()
```

## Security Considerations

⚠️ **Warning**: This framework is for authorized security testing only. Unauthorized access to vehicle networks is illegal and dangerous.

- Always obtain written permission before testing
- Never test on vehicles in motion
- Use isolated test environments when possible
- Keep safety systems (airbags, brakes) disconnected during testing
- Follow responsible disclosure practices for vulnerabilities

## Additional Resources

- CAN bus specification: ISO 11898
- UDS protocol: ISO 14229
- Automotive Ethernet: IEEE 802.1
- DBC file format documentation
