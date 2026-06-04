---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, CAN bus fuzzing, and automotive protocol analysis
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - fuzz automotive protocols
  - test vehicle ECU security
  - sniff CAN messages
  - perform vehicle penetration testing
  - use S800 framework
  - analyze automotive network vulnerabilities
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive framework for testing and analyzing vehicle network security. It provides tools for CAN bus analysis, protocol fuzzing, ECU testing, and automotive network penetration testing. The framework supports multiple automotive protocols including CAN, CAN-FD, LIN, and FlexRay.

**Note**: This is a test framework. Use only on authorized test vehicles or in laboratory environments. Unauthorized testing on production vehicles may be illegal and dangerous.

## Installation

### Prerequisites

```bash
# Linux system with CAN support
sudo apt-get update
sudo apt-get install can-utils python3 python3-pip git

# Load CAN kernel modules
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

# Setup virtual CAN for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Hardware Setup

For physical CAN testing, you'll need:
- CAN adapter (e.g., CANable, Kvaser, PEAK)
- OBD-II cable or direct CAN connection
- Target vehicle or ECU test bench

```bash
# Configure physical CAN interface (replace can0 with your interface)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

## Core Components

### 1. CAN Bus Sniffer

Capture and analyze CAN traffic:

```python
#!/usr/bin/env python3
import can
import sys

def sniff_can_traffic(interface='vcan0', duration=10):
    """Sniff CAN bus traffic"""
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        print(f"[*] Sniffing on {interface} for {duration} seconds...")
        
        message_log = {}
        
        for msg in bus:
            arb_id = msg.arbitration_id
            data = msg.data.hex()
            
            if arb_id not in message_log:
                message_log[arb_id] = []
            
            message_log[arb_id].append(data)
            print(f"ID: 0x{arb_id:03X} | Data: {data} | DLC: {msg.dlc}")
            
        bus.shutdown()
        return message_log
        
    except Exception as e:
        print(f"[-] Error: {e}")
        sys.exit(1)

if __name__ == "__main__":
    sniff_can_traffic('vcan0', 60)
```

### 2. CAN Message Injection

Send custom CAN messages for testing:

```python
#!/usr/bin/env python3
import can
import time

def send_can_message(interface, arb_id, data):
    """Send a single CAN message"""
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        
        # Create message
        msg = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        bus.send(msg)
        print(f"[+] Sent: ID=0x{arb_id:03X}, Data={data.hex()}")
        bus.shutdown()
        
    except can.CanError as e:
        print(f"[-] CAN Error: {e}")

def inject_replay_attack(interface, target_id, captured_data):
    """Replay captured CAN messages"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Replaying {len(captured_data)} messages...")
    for data in captured_data:
        msg = can.Message(
            arbitration_id=target_id,
            data=bytes.fromhex(data),
            is_extended_id=False
        )
        bus.send(msg)
        time.sleep(0.01)  # 10ms delay
    
    bus.shutdown()
    print("[+] Replay complete")

# Example usage
if __name__ == "__main__":
    # Send door unlock message (example)
    send_can_message('vcan0', 0x123, bytes([0x01, 0x02, 0x03, 0x04]))
    
    # Replay attack
    captured = ['01020304', '05060708', '090a0b0c']
    inject_replay_attack('vcan0', 0x456, captured)
```

### 3. CAN Fuzzer

Fuzz CAN bus to discover vulnerabilities:

```python
#!/usr/bin/env python3
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface, target_ids=None):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.target_ids = target_ids or range(0x000, 0x7FF)
        
    def random_fuzz(self, duration=60, delay=0.1):
        """Send random CAN messages"""
        print(f"[*] Starting random fuzzing for {duration} seconds...")
        start_time = time.time()
        count = 0
        
        while time.time() - start_time < duration:
            arb_id = random.choice(self.target_ids)
            data_len = random.randint(0, 8)
            data = bytes([random.randint(0, 255) for _ in range(data_len)])
            
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                count += 1
                if count % 100 == 0:
                    print(f"[*] Sent {count} messages...")
            except can.CanError as e:
                print(f"[-] Error sending: {e}")
            
            time.sleep(delay)
        
        print(f"[+] Fuzzing complete. Total messages: {count}")
    
    def bit_flip_fuzz(self, base_id, base_data, iterations=1000):
        """Fuzz by flipping bits in known good message"""
        print(f"[*] Starting bit-flip fuzzing...")
        
        for i in range(iterations):
            data = bytearray(base_data)
            
            # Flip random bit
            byte_pos = random.randint(0, len(data) - 1)
            bit_pos = random.randint(0, 7)
            data[byte_pos] ^= (1 << bit_pos)
            
            msg = can.Message(
                arbitration_id=base_id,
                data=bytes(data),
                is_extended_id=False
            )
            
            self.bus.send(msg)
            time.sleep(0.01)
            
            if (i + 1) % 100 == 0:
                print(f"[*] Iteration {i + 1}/{iterations}")
        
        print("[+] Bit-flip fuzzing complete")
    
    def sequential_id_scan(self, data=b'\x00\x00\x00\x00'):
        """Scan all CAN IDs sequentially"""
        print("[*] Scanning CAN ID space...")
        
        for arb_id in range(0x000, 0x7FF):
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                if arb_id % 0x100 == 0:
                    print(f"[*] Progress: 0x{arb_id:03X}")
            except can.CanError:
                pass
            
            time.sleep(0.05)
        
        print("[+] ID scan complete")
    
    def close(self):
        self.bus.shutdown()

# Example usage
if __name__ == "__main__":
    fuzzer = CANFuzzer('vcan0')
    
    # Random fuzzing
    fuzzer.random_fuzz(duration=30, delay=0.05)
    
    # Bit-flip fuzzing on known message
    fuzzer.bit_flip_fuzz(0x123, b'\x01\x02\x03\x04\x05', iterations=500)
    
    # ID space scan
    fuzzer.sequential_id_scan()
    
    fuzzer.close()
```

### 4. UDS (Unified Diagnostic Services) Testing

Test ECU diagnostic services:

```python
#!/usr/bin/env python3
import can
import time

class UDSTester:
    def __init__(self, interface, request_id=0x7DF, response_id=0x7E8):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.request_id = request_id
        self.response_id = response_id
    
    def send_uds_request(self, service, data=None):
        """Send UDS request and wait for response"""
        payload = [service]
        if data:
            payload.extend(data)
        
        # Pad to 8 bytes
        while len(payload) < 8:
            payload.append(0x00)
        
        msg = can.Message(
            arbitration_id=self.request_id,
            data=bytes(payload),
            is_extended_id=False
        )
        
        self.bus.send(msg)
        print(f"[>] Sent UDS: Service=0x{service:02X}, Data={bytes(payload).hex()}")
        
        # Wait for response
        timeout = time.time() + 2
        while time.time() < timeout:
            response = self.bus.recv(timeout=0.5)
            if response and response.arbitration_id == self.response_id:
                print(f"[<] Response: {response.data.hex()}")
                return response.data
        
        print("[-] No response received")
        return None
    
    def read_dtc(self):
        """Read Diagnostic Trouble Codes"""
        print("[*] Reading DTCs...")
        # Service 0x19 (ReadDTCInformation), Sub-function 0x02
        return self.send_uds_request(0x19, [0x02, 0xFF])
    
    def read_data_by_id(self, data_id):
        """Read data by identifier"""
        print(f"[*] Reading data identifier 0x{data_id:04X}...")
        # Service 0x22 (ReadDataByIdentifier)
        return self.send_uds_request(0x22, [data_id >> 8, data_id & 0xFF])
    
    def enter_diagnostic_session(self, session_type=0x03):
        """Enter diagnostic session"""
        print(f"[*] Entering diagnostic session type 0x{session_type:02X}...")
        # Service 0x10 (DiagnosticSessionControl)
        return self.send_uds_request(0x10, [session_type])
    
    def security_access(self, level=0x01):
        """Request seed for security access"""
        print(f"[*] Requesting security access level 0x{level:02X}...")
        # Service 0x27 (SecurityAccess)
        return self.send_uds_request(0x27, [level])
    
    def close(self):
        self.bus.shutdown()

# Example usage
if __name__ == "__main__":
    tester = UDSTester('vcan0')
    
    # Enter extended diagnostic session
    tester.enter_diagnostic_session(0x03)
    
    # Read VIN (Data ID 0xF190)
    tester.read_data_by_id(0xF190)
    
    # Read DTCs
    tester.read_dtc()
    
    # Request security access
    tester.security_access(0x01)
    
    tester.close()
```

### 5. Traffic Analysis and Pattern Detection

Analyze captured CAN traffic:

```python
#!/usr/bin/env python3
import can
from collections import defaultdict
import statistics

class CANAnalyzer:
    def __init__(self, interface):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.traffic_log = defaultdict(list)
        self.timing_data = defaultdict(list)
    
    def capture_traffic(self, duration=60):
        """Capture and analyze traffic patterns"""
        print(f"[*] Capturing traffic for {duration} seconds...")
        
        last_timestamp = defaultdict(float)
        
        self.bus.set_filters([])  # Clear filters, capture all
        
        for msg in self.bus:
            arb_id = msg.arbitration_id
            timestamp = msg.timestamp
            
            # Store data
            self.traffic_log[arb_id].append(msg.data.hex())
            
            # Calculate timing
            if arb_id in last_timestamp:
                interval = timestamp - last_timestamp[arb_id]
                self.timing_data[arb_id].append(interval)
            
            last_timestamp[arb_id] = timestamp
        
        self.bus.shutdown()
    
    def generate_report(self):
        """Generate analysis report"""
        print("\n=== CAN Traffic Analysis Report ===\n")
        
        for arb_id in sorted(self.traffic_log.keys()):
            messages = self.traffic_log[arb_id]
            unique_messages = set(messages)
            
            print(f"ID: 0x{arb_id:03X}")
            print(f"  Total Messages: {len(messages)}")
            print(f"  Unique Payloads: {len(unique_messages)}")
            
            if arb_id in self.timing_data and self.timing_data[arb_id]:
                intervals = self.timing_data[arb_id]
                avg_interval = statistics.mean(intervals)
                print(f"  Avg Interval: {avg_interval*1000:.2f} ms")
                print(f"  Frequency: {1/avg_interval:.2f} Hz")
            
            # Show sample payloads
            print(f"  Sample Payloads:")
            for payload in list(unique_messages)[:3]:
                print(f"    {payload}")
            print()
    
    def detect_anomalies(self):
        """Detect unusual patterns"""
        print("\n=== Anomaly Detection ===\n")
        
        for arb_id, messages in self.traffic_log.items():
            # Check for single-occurrence messages (suspicious)
            if len(messages) == 1:
                print(f"[!] ID 0x{arb_id:03X}: Single message (potential injection)")
                print(f"    Data: {messages[0]}")
            
            # Check for timing anomalies
            if arb_id in self.timing_data:
                intervals = self.timing_data[arb_id]
                if len(intervals) > 10:
                    avg = statistics.mean(intervals)
                    stdev = statistics.stdev(intervals)
                    
                    # Flag if high variance
                    if stdev / avg > 0.5:
                        print(f"[!] ID 0x{arb_id:03X}: High timing variance (CV={stdev/avg:.2f})")

# Example usage
if __name__ == "__main__":
    analyzer = CANAnalyzer('vcan0')
    analyzer.capture_traffic(duration=30)
    analyzer.generate_report()
    analyzer.detect_anomalies()
```

## Configuration

### Environment Variables

```bash
# CAN interface configuration
export S800_CAN_INTERFACE=can0
export S800_CAN_BITRATE=500000

# UDS configuration
export S800_UDS_REQUEST_ID=0x7DF
export S800_UDS_RESPONSE_ID=0x7E8

# Logging
export S800_LOG_LEVEL=INFO
export S800_LOG_FILE=/var/log/s800/can_traffic.log

# Safety limits
export S800_MAX_FUZZ_RATE=100  # messages per second
export S800_ENABLE_SAFETY_CHECK=true
```

### Configuration File (config.json)

```json
{
  "interfaces": {
    "primary": "can0",
    "secondary": "vcan0"
  },
  "can_settings": {
    "bitrate": 500000,
    "sample_point": 0.75,
    "sjw": 1
  },
  "uds": {
    "request_id": "0x7DF",
    "response_id": "0x7E8",
    "timeout": 2000
  },
  "fuzzing": {
    "max_rate": 100,
    "target_ids": ["0x100-0x200", "0x300-0x400"],
    "excluded_ids": ["0x7DF", "0x7E8"]
  },
  "logging": {
    "level": "INFO",
    "file": "/var/log/s800/activity.log",
    "rotate": true
  }
}
```

## Common Testing Patterns

### Pattern 1: Baseline Traffic Capture

```python
# Capture normal vehicle operation
def establish_baseline(interface, duration=300):
    """Capture baseline traffic during normal operation"""
    analyzer = CANAnalyzer(interface)
    
    print("[*] Establishing baseline...")
    print("[*] Perform normal vehicle operations now")
    
    analyzer.capture_traffic(duration)
    analyzer.generate_report()
    
    # Save baseline
    with open('baseline.json', 'w') as f:
        json.dump(analyzer.traffic_log, f)
    
    return analyzer.traffic_log
```

### Pattern 2: Differential Analysis

```python
# Compare traffic before/after action
def differential_analysis(interface, action_description):
    """Capture traffic before and after an action"""
    print(f"[*] Starting differential analysis: {action_description}")
    
    # Before
    print("[*] Capturing BEFORE state (10s)...")
    before = capture_snapshot(interface, 10)
    
    input("Press Enter after performing action...")
    
    # After
    print("[*] Capturing AFTER state (10s)...")
    after = capture_snapshot(interface, 10)
    
    # Compare
    new_ids = set(after.keys()) - set(before.keys())
    changed_ids = {
        id: after[id] for id in after 
        if id in before and after[id] != before[id]
    }
    
    print(f"\n[+] New IDs: {[f'0x{id:03X}' for id in new_ids]}")
    print(f"[+] Changed IDs: {[f'0x{id:03X}' for id in changed_ids.keys()]}")
    
    return new_ids, changed_ids

def capture_snapshot(interface, duration):
    analyzer = CANAnalyzer(interface)
    analyzer.capture_traffic(duration)
    return analyzer.traffic_log
```

### Pattern 3: Replay Attack Testing

```python
def test_replay_attack(interface, target_id, capture_file):
    """Test replay attack resistance"""
    print(f"[*] Testing replay attack on ID 0x{target_id:03X}")
    
    # Load captured messages
    with open(capture_file, 'r') as f:
        captured = json.load(f)
    
    if str(target_id) not in captured:
        print("[-] Target ID not in capture")
        return
    
    messages = captured[str(target_id)]
    
    # Replay with variations
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Original timing
    print("[*] Replaying with original timing...")
    for msg_data in messages:
        msg = can.Message(
            arbitration_id=target_id,
            data=bytes.fromhex(msg_data)
        )
        bus.send(msg)
        time.sleep(0.01)
    
    # Rapid replay
    print("[*] Replaying rapidly...")
    for msg_data in messages * 10:
        msg = can.Message(
            arbitration_id=target_id,
            data=bytes.fromhex(msg_data)
        )
        bus.send(msg)
    
    bus.shutdown()
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify CAN modules loaded
lsmod | grep can

# Load modules if needed
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Check dmesg for errors
dmesg | grep can
```

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Set interface permissions
sudo chmod 666 /dev/ttyUSB0

# Or run with sudo (not recommended for production)
sudo python3 script.py
```

### No Traffic Captured

```python
# Verify interface is up
def check_interface(interface):
    try:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
        print(f"[+] Interface {interface} is accessible")
        bus.shutdown()
        return True
    except Exception as e:
        print(f"[-] Error: {e}")
        return False

# Check bitrate matches
# Most vehicles use 500kbps, some use 250kbps
sudo ip link set can0 type can bitrate 250000
```

### Message Injection Fails

```python
# Check for bus-off state
def check_bus_state(interface):
    import subprocess
    result = subprocess.run(['ip', '-details', 'link', 'show', interface], 
                          capture_output=True, text=True)
    print(result.stdout)
    
    if 'BUS-OFF' in result.stdout:
        print("[!] Bus in BUS-OFF state, restarting...")
        subprocess.run(['sudo', 'ip', 'link', 'set', 'down', interface])
        subprocess.run(['sudo', 'ip', 'link', 'set', 'up', interface])
```

### High CPU Usage During Fuzzing

```python
# Implement rate limiting
def rate_limited_fuzz(interface, max_rate=50):
    """Fuzz with rate limit"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    interval = 1.0 / max_rate
    
    for i in range(1000):
        msg = can.Message(
            arbitration_id=random.randint(0, 0x7FF),
            data=bytes([random.randint(0, 255) for _ in range(8)])
        )
        bus.send(msg)
        time.sleep(interval)  # Rate limit
    
    bus.shutdown()
```

## Safety Considerations

**CRITICAL**: Always implement safety checks when working with vehicle networks:

```python
# Safety wrapper
class SafeCANInterface:
    CRITICAL_IDS = [0x7DF, 0x7E8]  # UDS
    MAX_MESSAGE_RATE = 100  # per second
    
    def __init__(self, interface, enable_safety=True):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.enable_safety = enable_safety
        self.message_count = 0
        self.start_time = time.time()
    
    def send_safe(self, msg):
        """Send with safety checks"""
        if self.enable_safety:
            # Check critical IDs
            if msg.arbitration_id in self.CRITICAL_IDS:
                print("[!] Warning: Sending to critical ID")
                return False
            
            # Rate limiting
            elapsed = time.time() - self.start_time
            rate = self.message_count / elapsed if elapsed > 0 else 0
            
            if rate > self.MAX_MESSAGE_RATE:
                print("[!] Rate limit exceeded")
                time.sleep(0.1)
        
        self.bus.send(msg)
        self.message_count += 1
        return True
```

## Advanced Features

### Multi-Protocol Support

```python
# FlexRay support (requires hardware)
def flexray_test(interface):
    from can.interfaces.vector import VectorBus
    
    bus = VectorBus(channel=0, app_name='S800', bitrate=10000000)
    # FlexRay specific operations
    bus.shutdown()

# LIN support
def lin_test(interface):
    import serial
    ser = serial.Serial(interface, baudrate=19200)
    # LIN frame operations
    ser.close()
```

This skill provides comprehensive coverage of the S800 framework for vehicle network security testing. Remember to always test in authorized environments and follow responsible disclosure practices for any vulnerabilities discovered.
