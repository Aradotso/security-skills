---
name: s800-vehicle-network-security-testing
description: Security testing framework for vehicle networks (CAN, LIN, FlexRay) with fuzzing, injection, and monitoring capabilities
triggers:
  - test vehicle network security
  - fuzz CAN bus messages
  - inject automotive network packets
  - monitor vehicle communication protocols
  - scan car network vulnerabilities
  - perform automotive penetration testing
  - analyze CAN bus traffic
  - test ECU security
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and security analysis of automotive communication protocols including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay. It provides tools for message injection, fuzzing, monitoring, and vulnerability assessment of Electronic Control Units (ECUs) and vehicle networks.

**Key Capabilities:**
- CAN bus message injection and replay
- Protocol fuzzing for automotive networks
- Traffic monitoring and logging
- ECU vulnerability scanning
- Message filtering and analysis
- Simulation of attack scenarios

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can cantools pyserial

# For SocketCAN support on Linux
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Setup Virtual CAN Interface (Testing)

```bash
# Create virtual CAN interface
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Verify interface
ifconfig vcan0
```

### Hardware Interfaces

Common hardware for real vehicle testing:
- PEAK PCAN-USB
- Kvaser Leaf Light
- CANable USB adapter
- Arduino with MCP2515 CAN controller

## Core Components

### 1. CAN Message Injection

```python
import can
from s800.injector import CANInjector

# Initialize CAN interface
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')
injector = CANInjector(bus)

# Inject single message
message = can.Message(
    arbitration_id=0x123,
    data=[0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08],
    is_extended_id=False
)
injector.send(message)

# Inject with timing control
injector.send_periodic(message, period=0.1)  # Send every 100ms
```

### 2. Traffic Monitoring

```python
from s800.monitor import CANMonitor

# Start monitoring
monitor = CANMonitor(channel='vcan0', bustype='socketcan')
monitor.start()

# Filter specific CAN IDs
monitor.add_filter(arbitration_id=0x100, mask=0x7FF)

# Log to file
monitor.set_log_file('can_traffic.log')

# Get statistics
stats = monitor.get_statistics()
print(f"Messages received: {stats['total_messages']}")
print(f"Unique IDs: {stats['unique_ids']}")
```

### 3. Fuzzing Engine

```python
from s800.fuzzer import CANFuzzer
from s800.fuzzer.strategies import RandomDataFuzzer, SequentialIDFuzzer

# Random data fuzzing
fuzzer = CANFuzzer(channel='vcan0', bustype='socketcan')

# Fuzz specific CAN ID with random data
fuzzer.fuzz_id(
    arbitration_id=0x200,
    duration=60,  # seconds
    interval=0.01,  # 10ms between messages
    strategy=RandomDataFuzzer()
)

# Sequential ID fuzzing (scan all possible IDs)
fuzzer.scan_ids(
    start_id=0x000,
    end_id=0x7FF,
    data=[0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF],
    callback=lambda resp: print(f"Response: {resp}")
)
```

### 4. Replay Attacks

```python
from s800.replay import CANReplay

# Capture traffic
replay = CANReplay(channel='vcan0', bustype='socketcan')
replay.capture(duration=30, output_file='captured.log')

# Replay captured traffic
replay.replay_from_file(
    'captured.log',
    speed_multiplier=1.0,  # Real-time replay
    loop=False
)

# Replay with modifications
replay.replay_with_transform(
    'captured.log',
    transform=lambda msg: msg._replace(
        data=bytes([b ^ 0xFF for b in msg.data])  # XOR all bytes
    )
)
```

### 5. DBC File Support

```python
from s800.parser import DBCParser

# Load DBC database
dbc = DBCParser('vehicle.dbc')

# Decode message
msg = can.Message(arbitration_id=0x100, data=[...])
decoded = dbc.decode_message(msg)
print(decoded)  # {'EngineSpeed': 2500, 'Throttle': 45.5, ...}

# Encode message
encoded = dbc.encode_message('EngineStatus', {
    'EngineSpeed': 3000,
    'Throttle': 60.0
})
injector.send(encoded)
```

## Configuration

### Configuration File (s800_config.json)

```json
{
  "interface": {
    "channel": "can0",
    "bustype": "socketcan",
    "bitrate": 500000
  },
  "logging": {
    "level": "INFO",
    "file": "s800.log",
    "capture_dir": "./captures"
  },
  "fuzzing": {
    "default_interval": 0.01,
    "max_retries": 3,
    "timeout": 5.0
  },
  "filters": {
    "whitelist_ids": [256, 512, 768],
    "blacklist_ids": []
  },
  "safety": {
    "enable_safety_checks": true,
    "critical_ids": [100, 101, 102],
    "max_injection_rate": 1000
  }
}
```

### Load Configuration

```python
from s800.config import S800Config

config = S800Config.from_file('s800_config.json')
bus = can.interface.Bus(
    channel=config.interface.channel,
    bustype=config.interface.bustype,
    bitrate=config.interface.bitrate
)
```

## Common Attack Scenarios

### Diagnostic Session Hijacking (UDS)

```python
from s800.protocols.uds import UDSClient

# Connect to ECU
uds = UDSClient(channel='can0', tx_id=0x7E0, rx_id=0x7E8)

# Start diagnostic session
uds.start_diagnostic_session(session_type=0x03)  # Extended diagnostic

# Read DTC (Diagnostic Trouble Codes)
dtc_list = uds.read_dtc()
print(f"DTCs found: {dtc_list}")

# Clear DTCs
uds.clear_dtc()

# Security access bypass attempt
seed = uds.request_seed(level=0x01)
key = compute_key(seed)  # Custom key algorithm
uds.send_key(key)
```

### Denial of Service

```python
from s800.attacks import DOSAttack

# Bus flooding
dos = DOSAttack(channel='can0', bustype='socketcan')

# Flood with high priority messages
dos.flood_bus(
    arbitration_id=0x000,  # Highest priority
    data=[0xFF] * 8,
    rate=10000  # Messages per second
)

# Targeted ECU flooding
dos.target_ecu(
    target_id=0x300,
    duration=10
)
```

### Message Spoofing

```python
from s800.attacks import SpoofingAttack

# Spoof speed sensor
spoof = SpoofingAttack(channel='can0', bustype='socketcan')

# Monitor original messages and inject modified versions
spoof.man_in_the_middle(
    target_id=0x200,
    intercept=True,
    modify=lambda msg: msg._replace(
        data=bytes([0x00, 0x00, 0xFF, 0xFF] + list(msg.data[4:]))
    )
)
```

## CLI Usage

### Basic Commands

```bash
# Monitor CAN traffic
python -m s800 monitor --channel can0 --output traffic.log

# Inject single message
python -m s800 inject --id 0x123 --data "01:02:03:04:05:06:07:08" --channel can0

# Fuzz CAN ID range
python -m s800 fuzz --start 0x000 --end 0x7FF --duration 300 --channel can0

# Replay captured traffic
python -m s800 replay --file captured.log --channel can0 --speed 1.0

# Scan for active ECUs
python -m s800 scan --channel can0 --timeout 5

# Dump DBC definitions
python -m s800 dbc --file vehicle.dbc --export json
```

### Advanced Operations

```bash
# Fuzzing with specific strategy
python -m s800 fuzz --id 0x200 --strategy random --seed 12345 --verbose

# Conditional replay
python -m s800 replay --file captured.log --filter "id >= 0x100 and id <= 0x200"

# Export statistics
python -m s800 monitor --channel can0 --stats --export csv --output stats.csv
```

## Security Best Practices

### Safe Testing Environment

```python
from s800.safety import SafetyWrapper

# Wrap operations with safety checks
safe_injector = SafetyWrapper(injector, config.safety)

# This will be blocked if ID is in critical list
safe_injector.send(can.Message(arbitration_id=0x100, data=[...]))

# Set rate limiting
safe_injector.set_max_rate(100)  # Max 100 messages/second
```

### Environment Variables

```bash
# Hardware interface configuration
export S800_CHANNEL=can0
export S800_BUSTYPE=socketcan
export S800_BITRATE=500000

# Safety settings
export S800_ENABLE_SAFETY=true
export S800_MAX_INJECTION_RATE=1000

# Logging
export S800_LOG_LEVEL=DEBUG
export S800_LOG_FILE=/var/log/s800.log
```

## Troubleshooting

### Interface Not Found

```bash
# Check available interfaces
ip link show

# Bring up interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Check kernel modules
lsmod | grep can
```

### Permission Denied

```bash
# Add user to dialout/can group
sudo usermod -a -G dialout $USER
sudo usermod -a -G can $USER

# Or run with sudo (not recommended for production)
sudo python s800_script.py
```

### No Messages Received

```python
# Verify bus activity
import can
bus = can.interface.Bus(channel='can0', bustype='socketcan')
msg = bus.recv(timeout=5.0)
if msg is None:
    print("No traffic detected - check connections")

# Check for filters
bus.set_filters([])  # Clear all filters
```

### High Bus Load

```python
from s800.utils import BusLoadMonitor

# Monitor bus utilization
load_monitor = BusLoadMonitor(channel='can0', bustype='socketcan')
print(f"Bus load: {load_monitor.get_load()}%")

# Reduce injection rate if load > 70%
if load_monitor.get_load() > 70:
    injector.set_interval(0.05)  # Slow down
```

## Resources

- CAN Protocol: ISO 11898
- UDS Protocol: ISO 14229
- OBD-II: SAE J1979
- DBC file format specification
- Vehicle network security standards (ISO/SAE 21434)
