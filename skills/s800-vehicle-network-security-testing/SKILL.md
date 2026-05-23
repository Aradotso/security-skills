---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, focusing on CAN bus and automotive protocol vulnerabilities
triggers:
  - test vehicle network security
  - analyze CAN bus vulnerabilities
  - perform automotive security testing
  - use S800 framework
  - scan vehicle networks for vulnerabilities
  - test automotive protocol security
  - fuzzing vehicle CAN messages
  - analyze vehicle ECU security
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a vehicle network security testing framework designed for penetration testing and vulnerability assessment of automotive networks, particularly CAN (Controller Area Network) bus systems. It provides tools for sniffing, fuzzing, replaying, and analyzing vehicle network traffic to identify security weaknesses in Electronic Control Units (ECUs) and vehicle communication protocols.

**Key Capabilities:**
- CAN bus message sniffing and logging
- Protocol fuzzing and anomaly injection
- Message replay attacks
- ECU fingerprinting and enumeration
- Traffic analysis and pattern detection
- DoS testing against vehicle systems
- UDS (Unified Diagnostic Services) exploitation

## Installation

### Prerequisites

```bash
# Install Python dependencies
pip install python-can cantools

# On Linux, install SocketCAN utilities
sudo apt-get install can-utils

# Load kernel modules for CAN interface
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Setup Virtual CAN Interface (for testing)

```bash
# Create virtual CAN interface
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Verify interface
ifconfig vcan0
```

### Hardware Setup (for real vehicle testing)

```bash
# Setup physical CAN interface (e.g., CAN-USB adapter)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Common bitrates: 125000, 250000, 500000, 1000000
```

## Core Components

### 1. CAN Bus Sniffer

Capture and log CAN bus traffic:

```python
import can
import time
from datetime import datetime

def sniff_can_traffic(interface='vcan0', duration=60):
    """Sniff CAN bus traffic for specified duration"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Sniffing on {interface} for {duration} seconds...")
    start_time = time.time()
    messages = []
    
    try:
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                timestamp = datetime.now().strftime("%H:%M:%S.%f")
                print(f"[{timestamp}] ID: 0x{msg.arbitration_id:03X} Data: {msg.data.hex()}")
                messages.append({
                    'timestamp': timestamp,
                    'id': msg.arbitration_id,
                    'data': msg.data.hex(),
                    'dlc': msg.dlc
                })
    except KeyboardInterrupt:
        print("\n[!] Sniffing stopped by user")
    finally:
        bus.shutdown()
    
    return messages

# Save captured traffic
def save_traffic_log(messages, filename='can_capture.log'):
    with open(filename, 'w') as f:
        for msg in messages:
            f.write(f"{msg['timestamp']},{msg['id']:03X},{msg['data']},{msg['dlc']}\n")
    print(f"[+] Saved {len(messages)} messages to {filename}")
```

### 2. CAN Message Fuzzer

Fuzz CAN messages to discover vulnerabilities:

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.known_ids = []
    
    def fuzz_random_id(self, count=100, delay=0.01):
        """Send random CAN messages with random IDs"""
        print(f"[*] Fuzzing with {count} random messages...")
        
        for i in range(count):
            arb_id = random.randint(0x000, 0x7FF)  # Standard CAN ID range
            data = bytes([random.randint(0, 255) for _ in range(8)])
            
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[{i+1}] Sent ID: 0x{arb_id:03X} Data: {data.hex()}")
                time.sleep(delay)
            except can.CanError as e:
                print(f"[!] Error sending message: {e}")
    
    def fuzz_known_id(self, target_id, count=100, delay=0.01):
        """Fuzz a specific CAN ID with random data"""
        print(f"[*] Fuzzing ID 0x{target_id:03X} with {count} variants...")
        
        for i in range(count):
            data = bytes([random.randint(0, 255) for _ in range(8)])
            
            msg = can.Message(
                arbitration_id=target_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[{i+1}] Data: {data.hex()}")
                time.sleep(delay)
            except can.CanError as e:
                print(f"[!] Error: {e}")
    
    def fuzz_byte_position(self, target_id, base_data, position, count=256):
        """Fuzz specific byte position in a message"""
        print(f"[*] Fuzzing byte position {position} in ID 0x{target_id:03X}...")
        
        for value in range(count):
            data = list(base_data)
            data[position] = value
            
            msg = can.Message(
                arbitration_id=target_id,
                data=bytes(data),
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[{value}] Position {position}: 0x{value:02X}")
                time.sleep(0.01)
            except can.CanError as e:
                print(f"[!] Error: {e}")
    
    def cleanup(self):
        self.bus.shutdown()

# Usage
fuzzer = CANFuzzer(interface='vcan0')
fuzzer.fuzz_random_id(count=50)
fuzzer.fuzz_known_id(target_id=0x123, count=100)
fuzzer.cleanup()
```

### 3. Message Replay Attack

Replay captured CAN messages:

```python
import can
import time

class CANReplayer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    def replay_from_log(self, logfile, speed_multiplier=1.0):
        """Replay messages from captured log file"""
        print(f"[*] Replaying messages from {logfile}...")
        
        messages = []
        with open(logfile, 'r') as f:
            for line in f:
                parts = line.strip().split(',')
                if len(parts) >= 3:
                    arb_id = int(parts[1], 16)
                    data = bytes.fromhex(parts[2])
                    messages.append((arb_id, data))
        
        for arb_id, data in messages:
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[>] Replayed ID: 0x{arb_id:03X} Data: {data.hex()}")
                time.sleep(0.01 / speed_multiplier)
            except can.CanError as e:
                print(f"[!] Error: {e}")
    
    def replay_single(self, arb_id, data, count=10, interval=0.1):
        """Replay a single message multiple times"""
        print(f"[*] Replaying ID 0x{arb_id:03X} {count} times...")
        
        msg = can.Message(
            arbitration_id=arb_id,
            data=bytes.fromhex(data) if isinstance(data, str) else data,
            is_extended_id=False
        )
        
        for i in range(count):
            try:
                self.bus.send(msg)
                print(f"[{i+1}] Sent")
                time.sleep(interval)
            except can.CanError as e:
                print(f"[!] Error: {e}")
    
    def cleanup(self):
        self.bus.shutdown()

# Usage
replayer = CANReplayer(interface='vcan0')
replayer.replay_single(arb_id=0x123, data='0102030405060708', count=5)
replayer.cleanup()
```

### 4. UDS (Unified Diagnostic Services) Scanner

Scan for diagnostic services:

```python
import can
import time

class UDSScanner:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.diagnostic_ids = range(0x7E0, 0x7E8)  # Common diagnostic IDs
    
    def scan_services(self, target_id=0x7E0, response_id=0x7E8):
        """Scan for available UDS services"""
        print(f"[*] Scanning UDS services on ID 0x{target_id:03X}...")
        
        # Common UDS services
        services = {
            0x10: "Diagnostic Session Control",
            0x11: "ECU Reset",
            0x14: "Clear Diagnostic Information",
            0x19: "Read DTC Information",
            0x22: "Read Data By Identifier",
            0x23: "Read Memory By Address",
            0x27: "Security Access",
            0x28: "Communication Control",
            0x2E: "Write Data By Identifier",
            0x31: "Routine Control",
            0x34: "Request Download",
            0x35: "Request Upload",
            0x36: "Transfer Data",
            0x37: "Request Transfer Exit",
            0x3E: "Tester Present"
        }
        
        for service_id, service_name in services.items():
            # Send UDS request
            data = bytes([0x02, service_id, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00])
            msg = can.Message(
                arbitration_id=target_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                time.sleep(0.1)
                
                # Check for response
                response = self.bus.recv(timeout=0.5)
                if response and response.arbitration_id == response_id:
                    if response.data[1] == service_id + 0x40:
                        print(f"[+] Service 0x{service_id:02X} ({service_name}): SUPPORTED")
                    elif response.data[1] == 0x7F:
                        print(f"[-] Service 0x{service_id:02X} ({service_name}): NOT SUPPORTED")
            except can.CanError as e:
                print(f"[!] Error: {e}")
    
    def send_tester_present(self, target_id=0x7E0, interval=2.0, duration=60):
        """Send Tester Present messages to keep diagnostic session alive"""
        print(f"[*] Sending Tester Present for {duration} seconds...")
        
        start_time = time.time()
        while time.time() - start_time < duration:
            data = bytes([0x02, 0x3E, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00])
            msg = can.Message(
                arbitration_id=target_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                print(f"[>] Tester Present sent")
                time.sleep(interval)
            except can.CanError as e:
                print(f"[!] Error: {e}")
    
    def cleanup(self):
        self.bus.shutdown()

# Usage
uds = UDSScanner(interface='vcan0')
uds.scan_services(target_id=0x7E0, response_id=0x7E8)
uds.cleanup()
```

### 5. Traffic Analyzer

Analyze patterns in CAN traffic:

```python
import can
from collections import defaultdict
import time

class TrafficAnalyzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.message_counts = defaultdict(int)
        self.message_data = defaultdict(list)
    
    def analyze_traffic(self, duration=60):
        """Analyze CAN traffic patterns"""
        print(f"[*] Analyzing traffic for {duration} seconds...")
        
        start_time = time.time()
        try:
            while time.time() - start_time < duration:
                msg = self.bus.recv(timeout=1.0)
                if msg:
                    self.message_counts[msg.arbitration_id] += 1
                    self.message_data[msg.arbitration_id].append(msg.data.hex())
        except KeyboardInterrupt:
            print("\n[!] Analysis stopped by user")
        
        self.print_statistics()
    
    def print_statistics(self):
        """Print traffic statistics"""
        print("\n[*] Traffic Statistics:")
        print("-" * 50)
        
        sorted_ids = sorted(self.message_counts.items(), key=lambda x: x[1], reverse=True)
        
        for arb_id, count in sorted_ids:
            unique_data = len(set(self.message_data[arb_id]))
            print(f"ID 0x{arb_id:03X}: {count} messages, {unique_data} unique payloads")
            
            if unique_data <= 5:
                print(f"  Payloads: {set(self.message_data[arb_id])}")
    
    def detect_periodic_messages(self, duration=30):
        """Detect periodically transmitted messages"""
        print(f"[*] Detecting periodic messages for {duration} seconds...")
        
        timestamps = defaultdict(list)
        start_time = time.time()
        
        try:
            while time.time() - start_time < duration:
                msg = self.bus.recv(timeout=1.0)
                if msg:
                    timestamps[msg.arbitration_id].append(time.time())
        except KeyboardInterrupt:
            print("\n[!] Detection stopped by user")
        
        print("\n[*] Periodic Messages:")
        for arb_id, times in timestamps.items():
            if len(times) > 2:
                intervals = [times[i+1] - times[i] for i in range(len(times)-1)]
                avg_interval = sum(intervals) / len(intervals)
                print(f"ID 0x{arb_id:03X}: ~{avg_interval*1000:.2f}ms interval")
    
    def cleanup(self):
        self.bus.shutdown()

# Usage
analyzer = TrafficAnalyzer(interface='vcan0')
analyzer.analyze_traffic(duration=30)
analyzer.detect_periodic_messages(duration=30)
analyzer.cleanup()
```

## Common Testing Workflows

### Full Reconnaissance Scan

```python
def vehicle_recon(interface='vcan0'):
    """Perform comprehensive vehicle network reconnaissance"""
    
    print("[*] Starting Vehicle Network Reconnaissance")
    
    # 1. Sniff normal traffic
    print("\n[STEP 1] Capturing baseline traffic...")
    messages = sniff_can_traffic(interface, duration=30)
    save_traffic_log(messages, 'baseline.log')
    
    # 2. Analyze traffic patterns
    print("\n[STEP 2] Analyzing traffic patterns...")
    analyzer = TrafficAnalyzer(interface)
    analyzer.analyze_traffic(duration=30)
    analyzer.detect_periodic_messages(duration=20)
    analyzer.cleanup()
    
    # 3. Scan for UDS services
    print("\n[STEP 3] Scanning for diagnostic services...")
    uds = UDSScanner(interface)
    for diag_id in range(0x7E0, 0x7E8):
        uds.scan_services(target_id=diag_id, response_id=diag_id+8)
    uds.cleanup()
    
    print("\n[+] Reconnaissance complete!")

# Run reconnaissance
vehicle_recon(interface='vcan0')
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=vcan0

# Set default bitrate
export S800_BITRATE=500000

# Enable verbose logging
export S800_VERBOSE=1

# Set log directory
export S800_LOG_DIR=/var/log/s800
```

### Configuration File (config.json)

```json
{
  "interface": "vcan0",
  "bitrate": 500000,
  "logging": {
    "enabled": true,
    "directory": "/var/log/s800",
    "level": "INFO"
  },
  "fuzzing": {
    "default_delay": 0.01,
    "max_messages": 1000,
    "blacklist_ids": [0x000, 0x7FF]
  },
  "uds": {
    "diagnostic_range": [0x7E0, 0x7E7],
    "response_offset": 8,
    "timeout": 0.5
  }
}
```

## Troubleshooting

### CAN Interface Not Found

```bash
# Check available interfaces
ip link show

# Verify SocketCAN modules loaded
lsmod | grep can

# Reload modules
sudo modprobe -r vcan
sudo modprobe vcan
```

### Permission Denied

```bash
# Add user to required groups
sudo usermod -a -G dialout $USER
sudo usermod -a -G can $USER

# Or run with sudo (not recommended for production)
sudo python test_script.py
```

### No Messages Received

```python
# Verify bus is active
def check_bus_activity(interface='vcan0', timeout=5):
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    msg = bus.recv(timeout=timeout)
    if msg:
        print(f"[+] Bus is active: {msg}")
    else:
        print(f"[!] No activity detected on {interface}")
    bus.shutdown()
```

### High CPU Usage During Fuzzing

```python
# Add delays between messages
fuzzer.fuzz_random_id(count=100, delay=0.05)  # Increase delay

# Reduce message count
fuzzer.fuzz_known_id(target_id=0x123, count=50)  # Fewer messages
```

## Safety Warnings

**⚠️ CRITICAL SAFETY NOTICE:**
- Never test on a moving vehicle
- Always use isolated test environments or virtual CAN interfaces
- Testing on real vehicles can cause unpredictable behavior
- Ensure emergency stop procedures are in place
- Comply with all local laws and regulations
- Obtain proper authorization before testing

## References

- CAN Bus Protocol: ISO 11898
- UDS Protocol: ISO 14229
- OBD-II Standards: SAE J1979
- SocketCAN Documentation: https://www.kernel.org/doc/html/latest/networking/can.html
