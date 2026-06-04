---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, supporting CAN bus and automotive protocol testing
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive security testing
  - use S800 framework
  - test car network vulnerabilities
  - scan vehicle communication protocols
  - fuzzing automotive networks
  - capture CAN bus messages
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and security analysis of automotive networks, primarily focusing on CAN (Controller Area Network) bus protocols. The framework provides tools for capturing, analyzing, replaying, and fuzzing vehicle network traffic to identify security vulnerabilities in modern vehicles.

**Key Capabilities:**
- CAN bus message capture and analysis
- Network traffic replay attacks
- Protocol fuzzing for vulnerability discovery
- ECU (Electronic Control Unit) enumeration
- Diagnostic protocol testing (UDS, OBD-II)
- Real-time monitoring and filtering

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can cantools pyserial

# For hardware support (SocketCAN on Linux)
sudo apt-get install can-utils

# Load kernel modules for CAN interface
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Setup S800

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install dependencies
pip install -r requirements.txt

# Set up virtual CAN interface (for testing without hardware)
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Configuration

For physical CAN bus testing, configure your CAN interface:

```bash
# Configure real CAN interface (adjust bitrate as needed)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface
ip -details link show can0
```

## Core Features

### 1. CAN Bus Message Capture

Capture and log CAN bus messages for analysis:

```python
import can
import time

def capture_can_traffic(interface='vcan0', duration=10):
    """Capture CAN messages for specified duration"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    print(f"Capturing on {interface} for {duration} seconds...")
    start_time = time.time()
    
    try:
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                messages.append({
                    'timestamp': msg.timestamp,
                    'arbitration_id': hex(msg.arbitration_id),
                    'data': msg.data.hex(),
                    'dlc': msg.dlc
                })
                print(f"ID: {hex(msg.arbitration_id)} Data: {msg.data.hex()}")
    finally:
        bus.shutdown()
    
    return messages

# Usage
captured = capture_can_traffic('can0', duration=30)
```

### 2. Message Replay Attack

Replay captured CAN messages to test vehicle responses:

```python
import can

def replay_can_messages(interface='vcan0', messages_file='captured.log'):
    """Replay CAN messages from log file"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    try:
        with open(messages_file, 'r') as f:
            for line in f:
                # Parse format: timestamp,arbitration_id,data
                parts = line.strip().split(',')
                if len(parts) >= 3:
                    arb_id = int(parts[1], 16)
                    data = bytes.fromhex(parts[2])
                    
                    msg = can.Message(
                        arbitration_id=arb_id,
                        data=data,
                        is_extended_id=False
                    )
                    
                    bus.send(msg)
                    print(f"Sent: ID={hex(arb_id)} Data={data.hex()}")
                    time.sleep(0.01)  # Delay between messages
    finally:
        bus.shutdown()

# Usage
replay_can_messages('can0', 'unlock_sequence.log')
```

### 3. CAN Bus Fuzzing

Fuzz CAN messages to discover vulnerabilities:

```python
import can
import random

def fuzz_can_id(interface='vcan0', target_id=0x123, iterations=1000):
    """Fuzz a specific CAN ID with random payloads"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Fuzzing ID {hex(target_id)} with {iterations} iterations...")
    
    try:
        for i in range(iterations):
            # Generate random 8-byte payload
            data = bytes([random.randint(0, 255) for _ in range(8)])
            
            msg = can.Message(
                arbitration_id=target_id,
                data=data,
                is_extended_id=False
            )
            
            bus.send(msg)
            
            if i % 100 == 0:
                print(f"Progress: {i}/{iterations} - Last: {data.hex()}")
            
            time.sleep(0.005)
    except KeyboardInterrupt:
        print("\nFuzzing stopped by user")
    finally:
        bus.shutdown()

# Fuzz specific ECU
fuzz_can_id('can0', target_id=0x7E0, iterations=5000)
```

### 4. UDS Diagnostic Testing

Test Unified Diagnostic Services (UDS) protocol:

```python
import can

class UDSScanner:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.timeout = 1.0
    
    def send_uds_request(self, ecu_id, service, data=b''):
        """Send UDS request and wait for response"""
        # UDS request format: [service_id, ...data]
        request_data = bytes([service]) + data
        
        msg = can.Message(
            arbitration_id=ecu_id,
            data=request_data,
            is_extended_id=False
        )
        
        self.bus.send(msg)
        
        # Wait for response (typically ECU_ID + 0x08)
        response_id = ecu_id + 0x08
        start_time = time.time()
        
        while time.time() - start_time < self.timeout:
            resp = self.bus.recv(timeout=0.1)
            if resp and resp.arbitration_id == response_id:
                return resp.data
        
        return None
    
    def scan_ecus(self, id_range=range(0x700, 0x800)):
        """Scan for active ECUs using DiagnosticSessionControl"""
        active_ecus = []
        
        for ecu_id in id_range:
            # Service 0x10: DiagnosticSessionControl, 0x01: Default session
            response = self.send_uds_request(ecu_id, 0x10, b'\x01')
            
            if response and response[0] == 0x50:  # Positive response
                active_ecus.append(hex(ecu_id))
                print(f"Found ECU at {hex(ecu_id)}")
        
        return active_ecus
    
    def read_dtc(self, ecu_id):
        """Read Diagnostic Trouble Codes"""
        # Service 0x19: ReadDTCInformation, 0x02: reportDTCByStatusMask
        response = self.send_uds_request(ecu_id, 0x19, b'\x02\xFF')
        return response
    
    def close(self):
        self.bus.shutdown()

# Usage
scanner = UDSScanner('can0')
ecus = scanner.scan_ecus()
print(f"Discovered ECUs: {ecus}")

for ecu in ecus:
    dtcs = scanner.read_dtc(int(ecu, 16))
    if dtcs:
        print(f"DTCs for {ecu}: {dtcs.hex()}")

scanner.close()
```

### 5. Real-time Monitoring with Filters

Monitor specific CAN IDs or patterns:

```python
import can

def monitor_with_filter(interface='vcan0', filter_ids=None):
    """Monitor CAN bus with optional ID filtering"""
    filters = None
    
    if filter_ids:
        # Create filter list
        filters = [{"can_id": id, "can_mask": 0x7FF} for id in filter_ids]
    
    bus = can.interface.Bus(
        channel=interface,
        bustype='socketcan',
        can_filters=filters
    )
    
    print(f"Monitoring {interface}...")
    print(f"Filters: {filter_ids if filter_ids else 'None (all messages)'}")
    
    try:
        for msg in bus:
            print(f"[{msg.timestamp:.6f}] ID: {hex(msg.arbitration_id)} "
                  f"DLC: {msg.dlc} Data: {msg.data.hex()}")
    except KeyboardInterrupt:
        print("\nMonitoring stopped")
    finally:
        bus.shutdown()

# Monitor specific IDs
monitor_with_filter('can0', filter_ids=[0x123, 0x456, 0x7E0])
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Set default bitrate
export S800_CAN_BITRATE=500000

# Enable verbose logging
export S800_VERBOSE=1

# Set capture output directory
export S800_OUTPUT_DIR=/var/log/s800
```

### Configuration File

Create `s800_config.json`:

```json
{
  "interface": "can0",
  "bitrate": 500000,
  "timeout": 1.0,
  "output_directory": "./captures",
  "fuzzing": {
    "delay_ms": 5,
    "max_iterations": 10000
  },
  "uds": {
    "ecu_scan_range": [1760, 2048],
    "default_timeout": 2.0
  },
  "filters": {
    "critical_ids": [291, 1110, 2016]
  }
}
```

## Common Testing Patterns

### Security Assessment Workflow

```python
import can
import json
import time
from datetime import datetime

class VehicleSecurityAssessment:
    def __init__(self, interface='can0', config_file='s800_config.json'):
        with open(config_file) as f:
            self.config = json.load(f)
        
        self.interface = interface
        self.results = {
            'timestamp': datetime.now().isoformat(),
            'ecus': [],
            'vulnerabilities': [],
            'traffic_analysis': {}
        }
    
    def run_full_assessment(self):
        """Run complete security assessment"""
        print("[1/4] Scanning for ECUs...")
        self.scan_ecus()
        
        print("[2/4] Capturing baseline traffic...")
        self.capture_baseline()
        
        print("[3/4] Testing UDS services...")
        self.test_uds_services()
        
        print("[4/4] Fuzzing critical IDs...")
        self.fuzz_critical_ids()
        
        self.save_results()
    
    def scan_ecus(self):
        scanner = UDSScanner(self.interface)
        ecu_range = range(*self.config['uds']['ecu_scan_range'])
        self.results['ecus'] = scanner.scan_ecus(ecu_range)
        scanner.close()
    
    def capture_baseline(self, duration=60):
        messages = capture_can_traffic(self.interface, duration)
        
        # Analyze message frequency
        id_counts = {}
        for msg in messages:
            arb_id = msg['arbitration_id']
            id_counts[arb_id] = id_counts.get(arb_id, 0) + 1
        
        self.results['traffic_analysis'] = id_counts
    
    def test_uds_services(self):
        # Test for unauthorized access to sensitive services
        sensitive_services = [0x27, 0x31, 0x34, 0x36]  # Security, routine, download
        
        for ecu in self.results['ecus']:
            for service in sensitive_services:
                # Test without authentication
                # Implementation specific to framework
                pass
    
    def fuzz_critical_ids(self):
        critical_ids = self.config['filters']['critical_ids']
        
        for can_id in critical_ids:
            try:
                fuzz_can_id(self.interface, can_id, iterations=1000)
            except Exception as e:
                self.results['vulnerabilities'].append({
                    'id': hex(can_id),
                    'issue': str(e)
                })
    
    def save_results(self):
        filename = f"assessment_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(filename, 'w') as f:
            json.dump(self.results, f, indent=2)
        print(f"Results saved to {filename}")

# Run assessment
assessment = VehicleSecurityAssessment('can0')
assessment.run_full_assessment()
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify kernel modules
lsmod | grep can

# Reload modules if needed
sudo modprobe -r vcan
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to dialout group for serial access
sudo usermod -a -G dialout $USER

# Set permissions for CAN interface
sudo chmod 666 /dev/can0
```

### No Messages Received

```python
# Verify interface is up and configured
import can

try:
    bus = can.interface.Bus(channel='can0', bustype='socketcan')
    print("Interface connected successfully")
    
    # Check for any traffic
    msg = bus.recv(timeout=5.0)
    if msg:
        print(f"Received: {msg}")
    else:
        print("No traffic detected - check connections and bitrate")
    
    bus.shutdown()
except Exception as e:
    print(f"Error: {e}")
```

### Bitrate Mismatch

```bash
# Common automotive bitrates: 125000, 250000, 500000, 1000000
# Try different bitrates if no traffic detected

sudo ip link set can0 down
sudo ip link set can0 type can bitrate 250000
sudo ip link set can0 up
```

## Safety Warnings

⚠️ **IMPORTANT**: Vehicle network testing can affect safety-critical systems.

- Always test on isolated test benches or simulation environments first
- Never test on public roads or production vehicles without authorization
- Understand the implications of each test before execution
- Keep emergency stop procedures ready
- Document all tests and changes thoroughly
- Comply with local regulations and manufacturer guidelines
