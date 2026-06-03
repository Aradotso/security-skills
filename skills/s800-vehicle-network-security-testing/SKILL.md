---
name: s800-vehicle-network-security-testing
description: Vehicle network security testing framework for CAN bus, LIN, FlexRay and automotive protocol vulnerability assessment
triggers:
  - test vehicle network security
  - scan CAN bus for vulnerabilities
  - perform automotive security testing
  - analyze vehicle network protocols
  - test car network security with S800
  - run automotive penetration tests
  - check vehicle ECU security
  - fuzz automotive protocols
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive security testing framework designed for automotive network protocols including CAN (Controller Area Network), LIN (Local Interconnect Network), FlexRay, and other vehicle communication systems. It enables security researchers and automotive engineers to identify vulnerabilities, perform penetration testing, and validate security controls in vehicle networks.

**Note**: This framework is for authorized security testing and research only. Always obtain proper authorization before testing vehicle systems.

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can cantools
pip install pyserial scapy

# For hardware interface support
sudo apt-get install can-utils
```

### Hardware Requirements

- CAN interface adapter (SocketCAN compatible, USB2CAN, PCAN, etc.)
- Connection to vehicle OBD-II port or direct ECU access
- Appropriate testing environment (bench setup or isolated vehicle)

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Set up CAN interface (Linux)
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# For physical CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Components

### 1. CAN Bus Scanner

Scan and monitor CAN bus traffic for security analysis:

```python
import can
import time

class CANScanner:
    def __init__(self, interface='can0', bitrate=500000):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.message_log = []
    
    def scan(self, duration=10):
        """Scan CAN bus for specified duration"""
        print(f"[*] Scanning CAN bus for {duration} seconds...")
        start_time = time.time()
        
        while (time.time() - start_time) < duration:
            message = self.bus.recv(timeout=1.0)
            if message:
                self.log_message(message)
                self.analyze_message(message)
        
        return self.message_log
    
    def log_message(self, msg):
        """Log CAN message with timestamp"""
        entry = {
            'timestamp': msg.timestamp,
            'arbitration_id': hex(msg.arbitration_id),
            'data': msg.data.hex(),
            'dlc': msg.dlc
        }
        self.message_log.append(entry)
        print(f"[+] ID: {entry['arbitration_id']} Data: {entry['data']}")
    
    def analyze_message(self, msg):
        """Perform basic security analysis"""
        # Check for diagnostic messages (OBD-II range 0x7E0-0x7EF)
        if 0x7E0 <= msg.arbitration_id <= 0x7EF:
            print(f"[!] Diagnostic message detected: {hex(msg.arbitration_id)}")
        
        # Check for high-priority messages
        if msg.arbitration_id < 0x100:
            print(f"[!] High-priority message: {hex(msg.arbitration_id)}")

# Usage
scanner = CANScanner(interface='can0')
messages = scanner.scan(duration=30)
```

### 2. Fuzzing Engine

Fuzz vehicle network protocols to discover vulnerabilities:

```python
import can
import random
import itertools

class VehicleFuzzer:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.target_ids = []
    
    def fuzz_random(self, arbitration_id, count=100, delay=0.01):
        """Send random fuzzing payloads"""
        print(f"[*] Fuzzing ID {hex(arbitration_id)} with {count} random payloads")
        
        for i in range(count):
            # Generate random data (0-8 bytes)
            data_length = random.randint(0, 8)
            data = [random.randint(0, 255) for _ in range(data_length)]
            
            msg = can.Message(
                arbitration_id=arbitration_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[+] Sent: {hex(arbitration_id)} -> {bytes(data).hex()}")
                time.sleep(delay)
            except can.CanError as e:
                print(f"[!] Error sending message: {e}")
    
    def fuzz_sequential(self, arbitration_id, byte_position=0):
        """Fuzz specific byte position with sequential values"""
        print(f"[*] Sequential fuzzing byte {byte_position} of ID {hex(arbitration_id)}")
        
        base_data = [0x00] * 8
        
        for value in range(256):
            base_data[byte_position] = value
            msg = can.Message(
                arbitration_id=arbitration_id,
                data=base_data,
                is_extended_id=False
            )
            
            self.bus.send(msg)
            time.sleep(0.01)
    
    def fuzz_diagnostic_session(self):
        """Fuzz UDS diagnostic session control"""
        diagnostic_id = 0x7E0
        session_types = [0x01, 0x02, 0x03, 0x04, 0x40, 0x7F]
        
        for session in session_types:
            # UDS: 0x10 (Diagnostic Session Control) + session type
            data = [0x02, 0x10, session, 0x00, 0x00, 0x00, 0x00, 0x00]
            msg = can.Message(arbitration_id=diagnostic_id, data=data)
            
            print(f"[*] Testing session type: {hex(session)}")
            self.bus.send(msg)
            time.sleep(0.1)

# Usage
fuzzer = VehicleFuzzer(interface='can0')
fuzzer.fuzz_random(arbitration_id=0x123, count=50)
fuzzer.fuzz_diagnostic_session()
```

### 3. Replay Attack Tool

Capture and replay CAN messages for security testing:

```python
import can
import pickle
import time

class ReplayAttack:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.captured_messages = []
    
    def capture(self, duration=10, output_file='capture.pkl'):
        """Capture CAN messages for replay"""
        print(f"[*] Capturing messages for {duration} seconds...")
        start_time = time.time()
        
        while (time.time() - start_time) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.captured_messages.append({
                    'timestamp': msg.timestamp,
                    'arbitration_id': msg.arbitration_id,
                    'data': list(msg.data),
                    'is_extended_id': msg.is_extended_id
                })
        
        # Save to file
        with open(output_file, 'wb') as f:
            pickle.dump(self.captured_messages, f)
        
        print(f"[+] Captured {len(self.captured_messages)} messages to {output_file}")
    
    def replay(self, input_file='capture.pkl', repeat=1, speed=1.0):
        """Replay captured messages"""
        with open(input_file, 'rb') as f:
            messages = pickle.load(f)
        
        print(f"[*] Replaying {len(messages)} messages (repeat={repeat}, speed={speed}x)")
        
        for iteration in range(repeat):
            last_timestamp = None
            
            for msg_data in messages:
                # Calculate delay based on original timing
                if last_timestamp is not None:
                    delay = (msg_data['timestamp'] - last_timestamp) / speed
                    time.sleep(max(0, delay))
                
                msg = can.Message(
                    arbitration_id=msg_data['arbitration_id'],
                    data=msg_data['data'],
                    is_extended_id=msg_data['is_extended_id']
                )
                
                self.bus.send(msg)
                last_timestamp = msg_data['timestamp']
            
            print(f"[+] Completed replay iteration {iteration + 1}/{repeat}")

# Usage
replay = ReplayAttack(interface='can0')
replay.capture(duration=20, output_file='door_unlock.pkl')
replay.replay(input_file='door_unlock.pkl', repeat=3)
```

### 4. UDS (Unified Diagnostic Services) Tester

Test diagnostic protocol security:

```python
import can
import time

class UDSTester:
    def __init__(self, interface='can0', tx_id=0x7E0, rx_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.tx_id = tx_id  # Tester ID
        self.rx_id = rx_id  # ECU response ID
    
    def send_uds_request(self, service, data=None):
        """Send UDS request and wait for response"""
        if data is None:
            data = []
        
        # Build request: [length, service, ...data]
        request = [len(data) + 1, service] + data
        request += [0x00] * (8 - len(request))  # Pad to 8 bytes
        
        msg = can.Message(arbitration_id=self.tx_id, data=request)
        self.bus.send(msg)
        
        # Wait for response
        start_time = time.time()
        while (time.time() - start_time) < 2.0:
            response = self.bus.recv(timeout=0.5)
            if response and response.arbitration_id == self.rx_id:
                return response
        
        return None
    
    def read_dtc(self):
        """Read Diagnostic Trouble Codes (0x19 service)"""
        print("[*] Reading DTCs...")
        response = self.send_uds_request(service=0x19, data=[0x02, 0xFF])
        
        if response:
            print(f"[+] Response: {response.data.hex()}")
            return response.data
        else:
            print("[!] No response received")
            return None
    
    def security_access_test(self):
        """Test security access (0x27 service)"""
        print("[*] Testing security access...")
        
        # Request seed (0x27 0x01)
        response = self.send_uds_request(service=0x27, data=[0x01])
        
        if response:
            print(f"[+] Seed response: {response.data.hex()}")
            
            if response.data[1] == 0x67:  # Positive response
                seed = response.data[2:6]
                print(f"[+] Received seed: {seed.hex()}")
                
                # Attempt key calculation (simplified example)
                key = self.calculate_key(seed)
                
                # Send key (0x27 0x02 + key)
                key_response = self.send_uds_request(service=0x27, data=[0x02] + list(key))
                
                if key_response and key_response.data[1] == 0x67:
                    print("[+] Security access granted!")
                    return True
                else:
                    print("[!] Security access denied")
                    return False
        
        return False
    
    def calculate_key(self, seed):
        """Calculate security key from seed (implementation-specific)"""
        # This is a placeholder - real implementation depends on ECU algorithm
        # Common algorithms: XOR, polynomial transforms, etc.
        return bytes([s ^ 0xAA for s in seed])
    
    def read_memory(self, address, size):
        """Read memory by address (0x23 service)"""
        print(f"[*] Reading {size} bytes from address {hex(address)}")
        
        # Build memory address and size parameters
        addr_bytes = address.to_bytes(4, byteorder='big')
        size_bytes = size.to_bytes(2, byteorder='big')
        
        response = self.send_uds_request(
            service=0x23,
            data=list(addr_bytes) + list(size_bytes)
        )
        
        if response:
            print(f"[+] Memory data: {response.data.hex()}")
            return response.data
        
        return None

# Usage
uds = UDSTester(interface='can0', tx_id=0x7E0, rx_id=0x7E8)
uds.read_dtc()
uds.security_access_test()
uds.read_memory(address=0x00010000, size=16)
```

## Configuration

### Environment Variables

```bash
# Set CAN interface
export S800_CAN_INTERFACE=can0
export S800_CAN_BITRATE=500000

# Diagnostic IDs
export S800_DIAG_TX_ID=0x7E0
export S800_DIAG_RX_ID=0x7E8

# Logging
export S800_LOG_LEVEL=DEBUG
export S800_LOG_FILE=/var/log/s800/testing.log
```

### Configuration File (s800_config.json)

```json
{
  "interfaces": {
    "can0": {
      "type": "socketcan",
      "bitrate": 500000,
      "fd": false
    },
    "can1": {
      "type": "socketcan",
      "bitrate": 250000,
      "fd": false
    }
  },
  "diagnostic": {
    "default_tx_id": "0x7E0",
    "default_rx_id": "0x7E8",
    "timeout": 2.0
  },
  "fuzzing": {
    "max_iterations": 1000,
    "delay_between_messages": 0.01,
    "blacklist_ids": ["0x000", "0x7FF"]
  },
  "security": {
    "enable_safeguards": true,
    "max_messages_per_second": 100
  }
}
```

## Common Testing Patterns

### Full Security Assessment

```python
import json
from datetime import datetime

class VehicleSecurityAssessment:
    def __init__(self, interface='can0', config_file='s800_config.json'):
        with open(config_file, 'r') as f:
            self.config = json.load(f)
        
        self.interface = interface
        self.scanner = CANScanner(interface)
        self.fuzzer = VehicleFuzzer(interface)
        self.uds = UDSTester(interface)
        self.results = []
    
    def run_full_assessment(self):
        """Execute comprehensive security assessment"""
        print("[*] Starting vehicle security assessment...")
        
        # Phase 1: Network Discovery
        print("\n[Phase 1] Network Discovery")
        messages = self.scanner.scan(duration=60)
        active_ids = set([msg['arbitration_id'] for msg in messages])
        print(f"[+] Discovered {len(active_ids)} active CAN IDs")
        
        # Phase 2: Diagnostic Testing
        print("\n[Phase 2] Diagnostic Protocol Testing")
        dtc_data = self.uds.read_dtc()
        security_result = self.uds.security_access_test()
        
        self.results.append({
            'test': 'security_access',
            'result': 'vulnerable' if security_result else 'secure',
            'timestamp': datetime.now().isoformat()
        })
        
        # Phase 3: Fuzzing (controlled)
        print("\n[Phase 3] Controlled Fuzzing")
        for arb_id in list(active_ids)[:5]:  # Test first 5 IDs
            print(f"[*] Fuzzing ID {arb_id}")
            self.fuzzer.fuzz_random(
                arbitration_id=int(arb_id, 16),
                count=50,
                delay=0.1
            )
        
        # Generate report
        self.generate_report()
    
    def generate_report(self):
        """Generate security assessment report"""
        report_file = f"security_report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        
        with open(report_file, 'w') as f:
            json.dump(self.results, f, indent=2)
        
        print(f"\n[+] Report saved to {report_file}")

# Usage
assessment = VehicleSecurityAssessment(interface='can0')
assessment.run_full_assessment()
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Bring up interface
sudo ip link set can0 up type can bitrate 500000

# Verify with candump
candump can0
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Set up udev rules for CAN devices
echo 'KERNEL=="can*", MODE="0666"' | sudo tee /etc/udev/rules.d/99-can.rules
sudo udevadm control --reload-rules
```

### No Messages Received

```python
# Check bus status
def check_bus_status(interface='can0'):
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        msg = bus.recv(timeout=5.0)
        if msg:
            print(f"[+] Bus active, received: {msg}")
        else:
            print("[!] No messages received - check connections")
        bus.shutdown()
    except Exception as e:
        print(f"[!] Error: {e}")
```

### Bitrate Mismatch

```bash
# Common automotive bitrates
# High-speed CAN: 500 kbps or 1 Mbps
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 type can bitrate 1000000

# Low-speed CAN: 125 kbps or 250 kbps
sudo ip link set can0 type can bitrate 125000
```

## Safety Considerations

Always implement safety checks when testing vehicle systems:

```python
class SafetyWrapper:
    def __init__(self, interface='can0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.critical_ids = [0x000, 0x100]  # Define critical safety IDs
        self.max_rate = 100  # messages per second
    
    def safe_send(self, msg):
        """Send message with safety checks"""
        if msg.arbitration_id in self.critical_ids:
            print(f"[!] WARNING: Attempting to send to critical ID {hex(msg.arbitration_id)}")
            response = input("Continue? (yes/no): ")
            if response.lower() != 'yes':
                print("[*] Message blocked for safety")
                return False
        
        self.bus.send(msg)
        return True
```

**Always test in isolated environments before live vehicle testing.**
