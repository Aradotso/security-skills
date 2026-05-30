---
name: s800-vehicle-network-security-testing
description: Framework for testing and analyzing vehicle network security, focusing on CAN bus and automotive communication protocols
triggers:
  - test vehicle network security
  - analyze CAN bus traffic
  - perform automotive security testing
  - scan vehicle communication protocols
  - use S800 framework
  - test car network vulnerabilities
  - simulate vehicle network attacks
  - audit automotive ECU security
---

# S800 Vehicle Network Security Testing Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

S800 is a security testing framework designed for analyzing and testing vehicle network communications, with primary focus on Controller Area Network (CAN) bus protocols. It provides tools for monitoring, fuzzing, and security assessment of automotive electronic control units (ECUs) and in-vehicle networks.

**Key Capabilities:**
- CAN bus traffic monitoring and analysis
- Protocol fuzzing for automotive networks
- ECU communication testing
- Vulnerability detection in vehicle networks
- Replay attack simulation
- Network packet crafting and injection

## Installation

### Prerequisites

```bash
# Install required dependencies
sudo apt-get update
sudo apt-get install -y python3 python3-pip can-utils

# Install SocketCAN kernel modules
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Setup Virtual CAN Interface

```bash
# Create virtual CAN interface for testing
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Verify interface is up
ifconfig vcan0
```

### Clone and Install

```bash
git clone https://github.com/zhu-zhu666/S800-Vehicle-Network-Security-Testing-Framework.git
cd S800-Vehicle-Network-Security-Testing-Framework

# Install Python dependencies
pip3 install -r requirements.txt

# Make scripts executable
chmod +x *.py
```

## Core Components

### 1. CAN Bus Sniffer

Monitor and capture CAN bus traffic:

```python
import can
import os

def sniff_can_traffic(interface='vcan0', duration=10):
    """
    Capture CAN bus traffic for analysis
    
    Args:
        interface: CAN interface name
        duration: Capture duration in seconds
    """
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Sniffing on {interface} for {duration} seconds...")
    messages = []
    
    try:
        import time
        start_time = time.time()
        
        while time.time() - start_time < duration:
            msg = bus.recv(timeout=1.0)
            if msg:
                messages.append({
                    'id': hex(msg.arbitration_id),
                    'data': msg.data.hex(),
                    'timestamp': msg.timestamp,
                    'dlc': msg.dlc
                })
                print(f"ID: {hex(msg.arbitration_id)} Data: {msg.data.hex()}")
                
    except KeyboardInterrupt:
        print("\n[!] Capture interrupted")
    finally:
        bus.shutdown()
    
    return messages

# Usage
captured = sniff_can_traffic('vcan0', duration=30)
```

### 2. CAN Frame Injection

Send crafted CAN frames:

```python
import can

def send_can_frame(interface, arb_id, data):
    """
    Send a CAN frame to the bus
    
    Args:
        interface: CAN interface (e.g., 'can0', 'vcan0')
        arb_id: Arbitration ID (0x000 - 0x7FF for standard, up to 0x1FFFFFFF for extended)
        data: Byte array or hex string of payload
    """
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Convert hex string to bytes if needed
    if isinstance(data, str):
        data = bytes.fromhex(data)
    
    msg = can.Message(
        arbitration_id=arb_id,
        data=data,
        is_extended_id=False
    )
    
    try:
        bus.send(msg)
        print(f"[+] Sent frame - ID: {hex(arb_id)}, Data: {data.hex()}")
    except can.CanError as e:
        print(f"[-] Failed to send frame: {e}")
    finally:
        bus.shutdown()

# Example: Send door unlock command (example payload)
send_can_frame('vcan0', 0x123, bytes([0x01, 0x02, 0x03, 0x04]))
```

### 3. Fuzzing Engine

Automated fuzzing of CAN messages:

```python
import can
import random
import time

def fuzz_can_bus(interface, target_id=None, iterations=1000):
    """
    Fuzz CAN bus with random or targeted messages
    
    Args:
        interface: CAN interface name
        target_id: Specific arbitration ID to fuzz (None for random)
        iterations: Number of fuzz iterations
    """
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print(f"[*] Starting fuzzing on {interface}")
    print(f"[*] Target ID: {hex(target_id) if target_id else 'Random'}")
    
    for i in range(iterations):
        # Generate random or targeted arbitration ID
        if target_id:
            arb_id = target_id
        else:
            arb_id = random.randint(0x000, 0x7FF)
        
        # Generate random data (1-8 bytes)
        data_len = random.randint(1, 8)
        data = bytes([random.randint(0, 255) for _ in range(data_len)])
        
        msg = can.Message(
            arbitration_id=arb_id,
            data=data,
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            if i % 100 == 0:
                print(f"[*] Sent {i} fuzz messages...")
        except can.CanError as e:
            print(f"[-] Error at iteration {i}: {e}")
        
        time.sleep(0.01)  # Small delay to prevent bus flooding
    
    bus.shutdown()
    print(f"[+] Fuzzing complete: {iterations} messages sent")

# Fuzz specific ECU
fuzz_can_bus('vcan0', target_id=0x7DF, iterations=500)
```

### 4. Replay Attack Simulator

Record and replay CAN traffic:

```python
import can
import time
import pickle

def record_can_session(interface, duration, output_file):
    """
    Record CAN traffic to file for later replay
    """
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    messages = []
    
    print(f"[*] Recording {duration} seconds of CAN traffic...")
    start_time = time.time()
    
    while time.time() - start_time < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            messages.append({
                'id': msg.arbitration_id,
                'data': list(msg.data),
                'timestamp': msg.timestamp,
                'is_extended': msg.is_extended_id
            })
    
    bus.shutdown()
    
    # Save to file
    with open(output_file, 'wb') as f:
        pickle.dump(messages, f)
    
    print(f"[+] Recorded {len(messages)} messages to {output_file}")

def replay_can_session(interface, input_file, speed=1.0):
    """
    Replay recorded CAN traffic
    
    Args:
        interface: CAN interface
        input_file: Recorded session file
        speed: Playback speed multiplier (1.0 = normal, 2.0 = 2x speed)
    """
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    # Load recorded messages
    with open(input_file, 'rb') as f:
        messages = pickle.load(f)
    
    print(f"[*] Replaying {len(messages)} messages at {speed}x speed...")
    
    start_time = messages[0]['timestamp']
    replay_start = time.time()
    
    for msg_data in messages:
        # Calculate timing
        original_offset = msg_data['timestamp'] - start_time
        target_time = replay_start + (original_offset / speed)
        
        # Wait until target time
        while time.time() < target_time:
            time.sleep(0.001)
        
        # Send message
        msg = can.Message(
            arbitration_id=msg_data['id'],
            data=bytes(msg_data['data']),
            is_extended_id=msg_data['is_extended']
        )
        
        bus.send(msg)
    
    bus.shutdown()
    print("[+] Replay complete")

# Record and replay workflow
record_can_session('vcan0', duration=10, output_file='session.pkl')
replay_can_session('vcan0', input_file='session.pkl', speed=1.0)
```

### 5. UDS (Unified Diagnostic Services) Scanner

Scan for diagnostic services:

```python
import can
import time

def uds_scan(interface, start_id=0x700, end_id=0x7FF):
    """
    Scan for ECUs responding to UDS diagnostic requests
    """
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    responding_ecus = []
    
    print("[*] Scanning for UDS-capable ECUs...")
    
    for arb_id in range(start_id, end_id + 1):
        # Send UDS session control request (0x10 0x01)
        msg = can.Message(
            arbitration_id=arb_id,
            data=[0x02, 0x10, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00],
            is_extended_id=False
        )
        
        try:
            bus.send(msg)
            time.sleep(0.05)
            
            # Check for response
            response = bus.recv(timeout=0.1)
            if response and response.arbitration_id == arb_id + 0x08:
                print(f"[+] Found ECU at ID {hex(arb_id)}")
                responding_ecus.append(arb_id)
        except:
            pass
    
    bus.shutdown()
    print(f"[+] Scan complete. Found {len(responding_ecus)} ECUs")
    return responding_ecus

# Scan for ECUs
ecus = uds_scan('vcan0')
```

## Configuration

### CAN Interface Setup

Configure hardware CAN interfaces:

```bash
# Set bitrate for physical CAN interface
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# Common bitrates: 125000, 250000, 500000, 1000000
```

### Environment Variables

```bash
# Set default CAN interface
export S800_CAN_INTERFACE=can0

# Set logging level
export S800_LOG_LEVEL=DEBUG

# Output directory for captured data
export S800_OUTPUT_DIR=/var/log/s800
```

## Common Testing Patterns

### Pattern 1: Baseline Traffic Analysis

```python
def analyze_baseline(interface, duration=60):
    """Capture and analyze normal CAN traffic patterns"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    id_stats = {}
    
    start = time.time()
    while time.time() - start < duration:
        msg = bus.recv(timeout=1.0)
        if msg:
            arb_id = msg.arbitration_id
            if arb_id not in id_stats:
                id_stats[arb_id] = {'count': 0, 'data_samples': []}
            
            id_stats[arb_id]['count'] += 1
            id_stats[arb_id]['data_samples'].append(msg.data.hex())
    
    bus.shutdown()
    
    # Print statistics
    print("\n=== Traffic Analysis ===")
    for arb_id, stats in sorted(id_stats.items()):
        print(f"ID {hex(arb_id)}: {stats['count']} messages")
    
    return id_stats
```

### Pattern 2: Anomaly Detection

```python
def detect_anomalies(interface, baseline, threshold=3):
    """Detect traffic deviating from baseline"""
    bus = can.interface.Bus(channel=interface, bustype='socketcan')
    
    print("[*] Monitoring for anomalies...")
    while True:
        msg = bus.recv(timeout=1.0)
        if msg:
            arb_id = msg.arbitration_id
            
            # Check if ID is in baseline
            if arb_id not in baseline:
                print(f"[!] ANOMALY: Unknown ID {hex(arb_id)}")
            else:
                # Check data pattern
                if msg.data.hex() not in baseline[arb_id]['data_samples'][:threshold]:
                    print(f"[!] ANOMALY: Unusual data on ID {hex(arb_id)}: {msg.data.hex()}")
```

## Troubleshooting

### Permission Denied

```bash
# Add user to necessary groups
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER

# Reboot or re-login for changes to take effect
```

### Interface Not Found

```bash
# List available CAN interfaces
ip link show type can

# Check if SocketCAN modules are loaded
lsmod | grep can

# Load modules if missing
sudo modprobe can
sudo modprobe can_raw
sudo modprobe vcan
```

### Bus-Off Errors

```bash
# Reset CAN interface
sudo ip link set can0 down
sudo ip link set can0 up

# Check error counters
ip -details -statistics link show can0
```

### No Traffic Captured

- Verify interface is UP: `ip link show`
- Check bitrate matches network: `ip -details link show can0`
- Confirm physical connections for hardware interfaces
- Use `candump` to verify traffic: `candump vcan0`

## Safety and Legal Warnings

⚠️ **IMPORTANT**: This framework is for authorized security testing only.

- Only test on vehicles you own or have explicit permission to test
- Testing on unauthorized vehicles may violate laws and safety regulations
- Improper CAN bus manipulation can cause vehicle malfunctions or safety hazards
- Always perform testing in a controlled, safe environment
- Disconnect critical systems before testing when possible
