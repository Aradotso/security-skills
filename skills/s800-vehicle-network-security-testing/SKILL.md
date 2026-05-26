---
name: s800-vehicle-network-security-testing
description: Security testing framework for automotive vehicle networks (CAN, LIN, FlexRay) with fuzzing and protocol analysis capabilities
triggers:
  - test vehicle network security
  - scan CAN bus vulnerabilities
  - fuzz automotive protocols
  - analyze vehicle network traffic
  - test ECU security
  - perform automotive penetration testing
  - S800 security framework
  - vehicle network fuzzing
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for automotive vehicle networks including CAN (Controller Area Network), LIN (Local Interconnect Network), and FlexRay protocols. It provides tools for security assessment, fuzzing, traffic analysis, and vulnerability detection in Electronic Control Units (ECUs) and vehicle communication systems.

**Key Features:**
- CAN bus fuzzing and packet injection
- Protocol analysis for automotive networks
- ECU vulnerability scanning
- Traffic monitoring and replay attacks
- Support for multiple vehicle network protocols
- Automated security testing workflows

## Installation

### Prerequisites

```bash
# Install required dependencies
sudo apt-get update
sudo apt-get install can-utils python3 python3-pip git

# Install SocketCAN kernel modules (Linux)
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Clone and Setup

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

### Hardware Setup

For physical CAN bus testing:

```bash
# Configure CAN interface (e.g., using CAN-USB adapter)
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Verify interface is active
ip -details link show can0
```

## Core Components

### 1. CAN Bus Testing

#### Basic CAN Frame Sending

```python
import can
import os

# Initialize CAN bus connection
bus = can.interface.Bus(channel='vcan0', bustype='socketcan')

# Send a single CAN frame
msg = can.Message(
    arbitration_id=0x123,
    data=[0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88],
    is_extended_id=False
)
bus.send(msg)

# Receive CAN messages
for message in bus:
    print(f"ID: {hex(message.arbitration_id)} Data: {message.data.hex()}")
    if message.arbitration_id == 0x456:
        break
```

#### CAN Traffic Monitoring

```python
import can
import time

def monitor_can_traffic(interface='vcan0', duration=10):
    """Monitor CAN bus traffic for specified duration"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"Monitoring {interface} for {duration} seconds...")
    start_time = time.time()
    message_count = {}
    
    while (time.time() - start_time) < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            arb_id = hex(msg.arbitration_id)
            message_count[arb_id] = message_count.get(arb_id, 0) + 1
            print(f"[{time.time():.2f}] ID: {arb_id} | Data: {msg.data.hex()}")
    
    print("\n=== Traffic Summary ===")
    for msg_id, count in sorted(message_count.items()):
        print(f"{msg_id}: {count} messages")
    
    bus.shutdown()

# Run monitoring
monitor_can_traffic(interface='can0', duration=30)
```

### 2. CAN Fuzzing

#### Random CAN Frame Fuzzer

```python
import can
import random
import time

class CANFuzzer:
    def __init__(self, interface='vcan0', target_ids=None):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.target_ids = target_ids or range(0x000, 0x7FF)
    
    def fuzz_random(self, count=1000, delay=0.01):
        """Send random CAN frames to test ECU resilience"""
        print(f"Starting fuzzing attack with {count} frames...")
        
        for i in range(count):
            # Randomize arbitration ID
            arb_id = random.choice(self.target_ids)
            
            # Randomize data length (0-8 bytes for standard CAN)
            data_len = random.randint(0, 8)
            data = [random.randint(0, 255) for _ in range(data_len)]
            
            msg = can.Message(
                arbitration_id=arb_id,
                data=data,
                is_extended_id=False
            )
            
            try:
                self.bus.send(msg)
                if i % 100 == 0:
                    print(f"Sent {i} frames...")
            except can.CanError as e:
                print(f"Error sending frame: {e}")
            
            time.sleep(delay)
        
        print("Fuzzing complete")
        self.bus.shutdown()

# Example usage
fuzzer = CANFuzzer(interface='can0', target_ids=[0x100, 0x200, 0x300])
fuzzer.fuzz_random(count=5000, delay=0.005)
```

#### Intelligent Mutation Fuzzer

```python
import can
import time

class MutationFuzzer:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.baseline_messages = []
    
    def capture_baseline(self, duration=10):
        """Capture normal traffic patterns"""
        print(f"Capturing baseline traffic for {duration}s...")
        start = time.time()
        
        while (time.time() - start) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.baseline_messages.append({
                    'id': msg.arbitration_id,
                    'data': list(msg.data)
                })
        
        print(f"Captured {len(self.baseline_messages)} baseline messages")
    
    def mutate_data(self, data):
        """Apply mutations to CAN data"""
        mutated = data.copy()
        mutation_type = random.choice(['bit_flip', 'byte_replace', 'boundary'])
        
        if mutation_type == 'bit_flip' and mutated:
            idx = random.randint(0, len(mutated) - 1)
            bit = random.randint(0, 7)
            mutated[idx] ^= (1 << bit)
        elif mutation_type == 'byte_replace' and mutated:
            idx = random.randint(0, len(mutated) - 1)
            mutated[idx] = random.randint(0, 255)
        elif mutation_type == 'boundary':
            mutated = [0xFF] * len(mutated)
        
        return mutated
    
    def fuzz_mutations(self, iterations=1000):
        """Fuzz based on captured baseline"""
        if not self.baseline_messages:
            print("No baseline captured. Run capture_baseline() first.")
            return
        
        print(f"Starting mutation fuzzing with {iterations} iterations...")
        
        for i in range(iterations):
            base_msg = random.choice(self.baseline_messages)
            mutated_data = self.mutate_data(base_msg['data'])
            
            msg = can.Message(
                arbitration_id=base_msg['id'],
                data=mutated_data,
                is_extended_id=False
            )
            
            self.bus.send(msg)
            time.sleep(0.01)
            
            if i % 100 == 0:
                print(f"Sent {i} mutated frames...")
        
        print("Mutation fuzzing complete")

# Example usage
import random
mfuzzer = MutationFuzzer(interface='can0')
mfuzzer.capture_baseline(duration=15)
mfuzzer.fuzz_mutations(iterations=2000)
```

### 3. Replay Attacks

```python
import can
import time
import pickle

class CANReplay:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.recorded_traffic = []
    
    def record(self, duration=30, output_file='can_capture.pkl'):
        """Record CAN traffic to file"""
        print(f"Recording for {duration} seconds...")
        start = time.time()
        
        while (time.time() - start) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.recorded_traffic.append({
                    'timestamp': time.time(),
                    'id': msg.arbitration_id,
                    'data': list(msg.data),
                    'is_extended': msg.is_extended_id
                })
        
        with open(output_file, 'wb') as f:
            pickle.dump(self.recorded_traffic, f)
        
        print(f"Recorded {len(self.recorded_traffic)} messages to {output_file}")
    
    def replay(self, input_file='can_capture.pkl', speed_multiplier=1.0):
        """Replay recorded CAN traffic"""
        with open(input_file, 'rb') as f:
            traffic = pickle.load(f)
        
        print(f"Replaying {len(traffic)} messages...")
        
        if not traffic:
            return
        
        base_time = traffic[0]['timestamp']
        start_replay = time.time()
        
        for entry in traffic:
            # Calculate delay
            original_delay = entry['timestamp'] - base_time
            target_time = start_replay + (original_delay / speed_multiplier)
            
            sleep_time = target_time - time.time()
            if sleep_time > 0:
                time.sleep(sleep_time)
            
            msg = can.Message(
                arbitration_id=entry['id'],
                data=entry['data'],
                is_extended_id=entry['is_extended']
            )
            self.bus.send(msg)
        
        print("Replay complete")
        self.bus.shutdown()

# Example: Record and replay attack
replay = CANReplay(interface='can0')
replay.record(duration=60, output_file='door_unlock.pkl')
# Later, replay the captured unlock sequence
replay.replay(input_file='door_unlock.pkl', speed_multiplier=1.0)
```

### 4. ECU Identification and Scanning

```python
import can
import time

class ECUScanner:
    def __init__(self, interface='vcan0'):
        self.bus = can.interface.Bus(channel=interface, bustype='socketcan')
        self.active_ids = set()
    
    def scan_active_ids(self, duration=30):
        """Identify active CAN IDs on the bus"""
        print(f"Scanning for active ECUs for {duration}s...")
        start = time.time()
        
        while (time.time() - start) < duration:
            msg = self.bus.recv(timeout=1.0)
            if msg:
                self.active_ids.add(msg.arbitration_id)
        
        print(f"\nFound {len(self.active_ids)} active CAN IDs:")
        for can_id in sorted(self.active_ids):
            print(f"  0x{can_id:03X}")
        
        return self.active_ids
    
    def probe_diagnostics(self, ecu_ids=None):
        """Probe ECUs for diagnostic services (UDS)"""
        if ecu_ids is None:
            ecu_ids = [0x7DF]  # Broadcast diagnostic ID
        
        # UDS service IDs
        services = {
            0x10: "Diagnostic Session Control",
            0x11: "ECU Reset",
            0x22: "Read Data By Identifier",
            0x27: "Security Access",
            0x3E: "Tester Present"
        }
        
        results = {}
        
        for ecu_id in ecu_ids:
            print(f"\nProbing ECU 0x{ecu_id:03X}...")
            results[ecu_id] = {}
            
            for service_id, service_name in services.items():
                # Send diagnostic request
                msg = can.Message(
                    arbitration_id=ecu_id,
                    data=[0x02, service_id, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00],
                    is_extended_id=False
                )
                
                self.bus.send(msg)
                
                # Wait for response
                timeout = time.time() + 0.5
                while time.time() < timeout:
                    response = self.bus.recv(timeout=0.1)
                    if response and response.arbitration_id == (ecu_id + 0x08):
                        results[ecu_id][service_id] = {
                            'name': service_name,
                            'response': list(response.data)
                        }
                        print(f"  ✓ {service_name}: Response received")
                        break
                else:
                    print(f"  ✗ {service_name}: No response")
        
        return results

# Example usage
scanner = ECUScanner(interface='can0')
active_ids = scanner.scan_active_ids(duration=20)
diag_results = scanner.probe_diagnostics(ecu_ids=[0x7E0, 0x7E1, 0x7DF])
```

## Configuration

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Set CAN bitrate
export S800_CAN_BITRATE=500000

# Enable verbose logging
export S800_DEBUG=1

# Set output directory for captures
export S800_OUTPUT_DIR=/var/log/s800
```

### Configuration File (s800_config.json)

```json
{
  "interfaces": {
    "primary": "can0",
    "secondary": "can1",
    "virtual": "vcan0"
  },
  "fuzzing": {
    "default_delay_ms": 10,
    "max_iterations": 10000,
    "target_ids": [256, 512, 768]
  },
  "logging": {
    "level": "INFO",
    "output_dir": "./logs",
    "save_pcap": true
  },
  "security": {
    "enable_safety_checks": true,
    "blacklist_ids": [100, 101],
    "rate_limit_ms": 5
  }
}
```

## Common Patterns

### Pattern 1: Comprehensive Security Audit

```python
import can
import time
import json

class VehicleSecurityAudit:
    def __init__(self, interface='can0'):
        self.interface = interface
        self.results = {
            'active_ids': [],
            'vulnerabilities': [],
            'recommendations': []
        }
    
    def run_full_audit(self):
        """Execute complete security audit"""
        print("=== Starting Vehicle Network Security Audit ===\n")
        
        # Step 1: Scan for active IDs
        scanner = ECUScanner(self.interface)
        self.results['active_ids'] = list(scanner.scan_active_ids(duration=30))
        
        # Step 2: Test for replay vulnerabilities
        print("\n[*] Testing replay attack vulnerability...")
        self.test_replay_vulnerability()
        
        # Step 3: Fuzzing stress test
        print("\n[*] Running fuzzing stress test...")
        self.fuzzing_stress_test()
        
        # Step 4: Generate report
        self.generate_report()
    
    def test_replay_vulnerability(self):
        """Check if network accepts replayed messages"""
        # Implementation details...
        self.results['vulnerabilities'].append({
            'type': 'Replay Attack',
            'severity': 'High',
            'description': 'Network accepts replayed messages without validation'
        })
    
    def fuzzing_stress_test(self):
        """Test ECU resilience to malformed packets"""
        fuzzer = CANFuzzer(self.interface)
        try:
            fuzzer.fuzz_random(count=1000, delay=0.01)
            self.results['vulnerabilities'].append({
                'type': 'Fuzzing Resilience',
                'severity': 'Medium',
                'description': 'Some ECUs may crash under fuzzing'
            })
        except Exception as e:
            print(f"Fuzzing error: {e}")
    
    def generate_report(self):
        """Generate JSON audit report"""
        with open('security_audit_report.json', 'w') as f:
            json.dump(self.results, f, indent=2)
        print("\n=== Audit Complete ===")
        print(f"Report saved to security_audit_report.json")

# Run audit
audit = VehicleSecurityAudit(interface='can0')
audit.run_full_audit()
```

### Pattern 2: Safe Testing with Virtual CAN

```bash
#!/bin/bash
# setup_test_environment.sh

# Create virtual CAN interfaces
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Start CAN simulator in background
candump vcan0 &
CANDUMP_PID=$!

# Run tests
python3 test_fuzzing.py --interface vcan0

# Cleanup
kill $CANDUMP_PID
sudo ip link delete vcan0
```

## Troubleshooting

### Issue: Permission Denied on CAN Interface

```bash
# Add user to required groups
sudo usermod -a -G dialout $USER
sudo usermod -a -G can $USER

# Set interface permissions
sudo chmod 666 /dev/can0

# Reboot or re-login for group changes to take effect
```

### Issue: CAN Interface Not Found

```bash
# List available CAN interfaces
ip link show | grep can

# Verify kernel modules loaded
lsmod | grep can

# Load required modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan

# Check dmesg for hardware errors
dmesg | grep -i can
```

### Issue: Bus-Off State

```python
import can
import subprocess

def reset_can_interface(interface='can0'):
    """Reset CAN interface if in bus-off state"""
    try:
        subprocess.run(['sudo', 'ip', 'link', 'set', 'down', interface], check=True)
        subprocess.run(['sudo', 'ip', 'link', 'set', 'up', interface], check=True)
        print(f"Interface {interface} reset successfully")
    except subprocess.CalledProcessError as e:
        print(f"Error resetting interface: {e}")

# Use when encountering bus-off errors
reset_can_interface('can0')
```

### Issue: High Bus Load Causing Packet Loss

```python
import can

# Implement filtering to reduce load
def setup_filtered_bus(interface='can0', filter_ids=None):
    """Setup CAN bus with ID filtering"""
    if filter_ids:
        filters = [{"can_id": can_id, "can_mask": 0x7FF} 
                   for can_id in filter_ids]
        bus = can.interface.Bus(
            channel=interface,
            bustype='socketcan',
            can_filters=filters
        )
    else:
        bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    return bus

# Only receive specific IDs
bus = setup_filtered_bus(interface='can0', filter_ids=[0x100, 0x200, 0x300])
```

## Best Practices

1. **Always test on isolated networks first** - Use virtual CAN (vcan) or isolated physical networks
2. **Implement rate limiting** - Avoid overwhelming vehicle networks with excessive traffic
3. **Log all activities** - Maintain detailed logs for audit and analysis
4. **Use safety checks** - Never test critical systems (braking, steering) without proper safety measures
5. **Follow responsible disclosure** - Report vulnerabilities to manufacturers before public disclosure

## Legal and Safety Warning

⚠️ **IMPORTANT**: Unauthorized testing on vehicle networks may be illegal and dangerous. Only perform security testing on:
- Your own vehicles with full understanding of risks
- Test benches and simulators
- Systems where you have explicit written authorization

Always prioritize safety and comply with local laws and regulations.
