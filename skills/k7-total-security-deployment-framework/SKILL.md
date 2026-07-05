---
name: k7-total-security-deployment-framework
description: Deploy and configure K7 Total Security 16.0.1195 multi-platform endpoint protection with AI-powered threat analysis
triggers:
  - "configure K7 Total Security deployment"
  - "set up K7 endpoint protection policies"
  - "create K7 security profile for enterprise"
  - "integrate K7 with OpenAI threat analysis"
  - "deploy K7 Total Security across multiple platforms"
  - "troubleshoot K7 security agent configuration"
  - "automate K7 policy distribution"
  - "configure K7 USB filtering and web protection"
---

# K7 Total Security Deployment Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

K7 Total Security 16.0.1195 is a unified endpoint protection framework supporting multi-platform deployment (Windows, macOS, Linux) with AI-powered threat detection, signatureless behavioral analysis, and centralized policy management. This skill covers configuration, deployment automation, and integration patterns.

## Installation

### Prerequisites

- Administrative access on target endpoints
- Network connectivity to management console (default port 4451)
- Valid K7 Total Security license
- (Optional) OpenAI or Claude API credentials for AI-powered analysis

### Console Installation

```bash
# Download and install management console
curl -O https://download.k7computing.com/k7-console-16.0.1195.tar.gz
tar -xzf k7-console-16.0.1195.tar.gz
cd k7-console

# Install with default settings
sudo ./install.sh --silent --accept-license

# Verify installation
k7-console --version
# Expected: K7 Total Security Console 16.0.1195
```

### Agent Deployment

```bash
# Windows (PowerShell as Administrator)
Invoke-WebRequest -Uri "https://download.k7computing.com/k7-agent-win-16.0.1195.msi" -OutFile "k7-agent.msi"
msiexec /i k7-agent.msi /quiet SERVERURL="https://console.example.com:4451"

# macOS
curl -O https://download.k7computing.com/k7-agent-mac-16.0.1195.pkg
sudo installer -pkg k7-agent-mac-16.0.1195.pkg -target /

# Linux (Debian/Ubuntu)
wget https://download.k7computing.com/k7-agent-linux-16.0.1195.deb
sudo dpkg -i k7-agent-linux-16.0.1195.deb
```

## Core Configuration

### Profile Structure

Security profiles are defined in YAML and applied to endpoints via the console or API.

```yaml
# enterprise_strict.yaml
profile_name: "Enterprise Strict"
version: "16.0.1195"
enforcement_level: "strict"

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "block_and_alert"
    exceptions:
      - path: "/opt/approved-tools/*"
      - hash: "sha256:a3f9c82..."
  
  memory_scanner:
    state: enabled
    sensitivity: "maximum"
    scan_interval_seconds: 180
    exclude_processes:
      - "vmware-vmx"
      - "VirtualBox"
  
  usb_filter:
    state: enabled
    policy: "block_all"
    allow_list:
      - vendor_id: "0x0781"
        product_id: "0x5581"
        serial: "ABC123DEF456"
  
  web_filter:
    state: enabled
    ssl_inspection: true
    categories_blocked:
      - "malware_distribution"
      - "phishing"
      - "cryptomining_scripts"
      - "peer_to_peer"
    allow_list:
      - domain: "*.corp.internal"

network_protection:
  firewall_mode: "application_aware"
  block_outbound_smtp: true
  dns_filtering: true
  dns_servers:
    - "1.1.1.1"
    - "8.8.8.8"

ai_integration:
  incident_analysis:
    provider: "openai"
    api_key_env: "K7_OPENAI_API_KEY"
    model: "gpt-4-turbo"
    max_tokens: 2000
    temperature: 0.3
    endpoint: "https://api.openai.com/v1/chat/completions"
  
  automated_response:
    provider: "claude"
    api_key_env: "K7_CLAUDE_API_KEY"
    model: "claude-3-opus-20240229"
    max_tokens: 1500
    endpoint: "https://api.anthropic.com/v1/messages"

logging:
  verbose: true
  retention_days: 365
  forward_to:
    - type: "syslog"
      host: "siem.example.com"
      port: 514
      protocol: "tcp"
    - type: "elasticsearch"
      url: "https://elk.example.com:9200"
      index: "k7-security-events"
      auth_token_env: "K7_ELK_TOKEN"

notifications:
  email:
    enabled: true
    smtp_server: "smtp.example.com:587"
    from: "security@example.com"
    recipients:
      - "soc@example.com"
    tls: true
    auth_user_env: "K7_SMTP_USER"
    auth_pass_env: "K7_SMTP_PASS"
  
  webhook:
    enabled: true
    url_env: "K7_WEBHOOK_URL"
    method: "POST"
    headers:
      Content-Type: "application/json"
    retry_attempts: 3
    timeout_seconds: 10
```

### Applying Profiles

```bash
# Apply to single endpoint
k7-console apply-profile \
  --profile enterprise_strict.yaml \
  --target 192.168.1.100 \
  --auth-cert /etc/k7/admin.pem \
  --log-level verbose

# Apply to endpoint group
k7-console apply-profile \
  --profile enterprise_strict.yaml \
  --group "engineering-team" \
  --auth-cert /etc/k7/admin.pem \
  --schedule "2024-01-15T02:00:00Z"

# Validate profile before deployment
k7-console validate-profile --file enterprise_strict.yaml
```

## API Integration

### Python Client Example

```python
import os
import requests
from typing import Dict, List

class K7SecurityClient:
    def __init__(self, base_url: str, cert_path: str):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.cert = cert_path
        self.session.verify = True
    
    def apply_profile(self, profile_data: Dict, targets: List[str]) -> Dict:
        """Apply security profile to multiple endpoints"""
        payload = {
            "profile": profile_data,
            "targets": targets,
            "async": True
        }
        response = self.session.post(
            f"{self.base_url}/api/v1/profiles/apply",
            json=payload,
            timeout=30
        )
        response.raise_for_status()
        return response.json()
    
    def get_endpoint_status(self, endpoint_id: str) -> Dict:
        """Retrieve current protection status for endpoint"""
        response = self.session.get(
            f"{self.base_url}/api/v1/endpoints/{endpoint_id}/status",
            timeout=10
        )
        response.raise_for_status()
        return response.json()
    
    def query_threats(self, start_time: str, severity: str = "high") -> List[Dict]:
        """Query threat events from centralized log"""
        params = {
            "start": start_time,
            "severity": severity,
            "limit": 1000
        }
        response = self.session.get(
            f"{self.base_url}/api/v1/threats/query",
            params=params,
            timeout=30
        )
        response.raise_for_status()
        return response.json()["events"]
    
    def trigger_ai_analysis(self, event_id: str) -> Dict:
        """Request AI analysis for specific threat event"""
        payload = {"event_id": event_id}
        response = self.session.post(
            f"{self.base_url}/api/v1/ai/analyze",
            json=payload,
            timeout=60
        )
        response.raise_for_status()
        return response.json()

# Usage
client = K7SecurityClient(
    base_url=os.environ["K7_CONSOLE_URL"],
    cert_path="/etc/k7/admin.pem"
)

# Deploy profile to engineering workstations
profile = {
    "profile_name": "Engineering Developer",
    "enforcement_level": "balanced",
    "protection_layers": {
        "executable_guard": {
            "state": "enabled",
            "exceptions": ["/opt/dev-tools/*"]
        }
    }
}

result = client.apply_profile(
    profile_data=profile,
    targets=["192.168.1.10", "192.168.1.11", "192.168.1.12"]
)
print(f"Deployment job ID: {result['job_id']}")

# Check endpoint status
status = client.get_endpoint_status("192.168.1.10")
print(f"Protection active: {status['protection_active']}")
print(f"Last scan: {status['last_scan_timestamp']}")

# Query recent high-severity threats
threats = client.query_threats(
    start_time="2024-01-01T00:00:00Z",
    severity="critical"
)
for threat in threats:
    print(f"Threat: {threat['type']} on {threat['endpoint_id']}")
    
    # Request AI analysis
    analysis = client.trigger_ai_analysis(threat["event_id"])
    print(f"AI recommendation: {analysis['recommendation']}")
```

### Webhook Integration for Slack Notifications

```python
import os
import hmac
import hashlib
from flask import Flask, request, jsonify

app = Flask(__name__)

WEBHOOK_SECRET = os.environ["K7_WEBHOOK_SECRET"]
SLACK_URL = os.environ["SLACK_WEBHOOK_URL"]

@app.route("/k7-webhook", methods=["POST"])
def handle_k7_event():
    # Verify signature
    signature = request.headers.get("X-K7-Signature")
    body = request.get_data()
    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(signature, expected):
        return jsonify({"error": "Invalid signature"}), 403
    
    event = request.json
    
    # Format Slack message
    slack_payload = {
        "text": f"🚨 K7 Security Alert: {event['threat_type']}",
        "blocks": [
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*Threat Detected*\n*Type:* {event['threat_type']}\n*Endpoint:* {event['endpoint_id']}\n*Severity:* {event['severity']}"
                }
            },
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*AI Analysis:*\n{event.get('ai_recommendation', 'Pending analysis...')}"
                }
            }
        ]
    }
    
    # Forward to Slack
    requests.post(SLACK_URL, json=slack_payload)
    
    return jsonify({"status": "processed"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

## Common Deployment Patterns

### Mass Deployment via Ansible

```yaml
# k7-deploy.yml
---
- name: Deploy K7 Total Security to fleet
  hosts: all_endpoints
  become: yes
  vars:
    k7_console_url: "https://console.example.com:4451"
    k7_profile: "{{ lookup('file', 'profiles/{{ inventory_hostname_short }}.yaml') }}"
  
  tasks:
    - name: Install K7 agent (Debian/Ubuntu)
      apt:
        deb: "https://download.k7computing.com/k7-agent-linux-16.0.1195.deb"
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Configure agent connection
      template:
        src: agent-config.j2
        dest: /etc/k7/agent.conf
        mode: '0600'
      notify: restart k7-agent
    
    - name: Apply security profile
      command: >
        k7-console apply-profile
        --profile {{ k7_profile }}
        --target {{ ansible_default_ipv4.address }}
        --auth-cert /etc/k7/admin.pem
      delegate_to: localhost
  
  handlers:
    - name: restart k7-agent
      service:
        name: k7-agent
        state: restarted
```

### Dynamic Policy Based on Host Role

```python
def generate_profile_for_role(role: str) -> Dict:
    """Generate K7 profile based on host role"""
    base = {
        "version": "16.0.1195",
        "enforcement_level": "adaptive",
        "logging": {"verbose": True, "retention_days": 90}
    }
    
    role_configs = {
        "web_server": {
            "web_filter": {"state": "disabled"},
            "network_protection": {
                "firewall_mode": "strict",
                "allowed_ports": [80, 443]
            }
        },
        "database_server": {
            "usb_filter": {"state": "enabled", "policy": "block_all"},
            "network_protection": {
                "firewall_mode": "strict",
                "allowed_ports": [3306, 5432]
            }
        },
        "developer_workstation": {
            "executable_guard": {
                "exceptions": [
                    "/opt/dev-tools/*",
                    "/home/*/workspace/*"
                ]
            },
            "memory_scanner": {"sensitivity": "medium"}
        }
    }
    
    base["protection_layers"] = role_configs.get(role, {})
    base["profile_name"] = f"{role.replace('_', ' ').title()} Profile"
    
    return base

# Usage
db_profile = generate_profile_for_role("database_server")
```

## Troubleshooting

### Agent Not Connecting to Console

```bash
# Check agent status
sudo systemctl status k7-agent

# Verify network connectivity
telnet console.example.com 4451

# Check agent logs
sudo tail -f /var/log/k7/agent.log

# Test certificate authentication
openssl s_client -connect console.example.com:4451 \
  -cert /etc/k7/agent.pem -key /etc/k7/agent.key

# Reconfigure console URL
sudo k7-agent --reconfigure --console-url https://new-console.example.com:4451
```

### Profile Application Fails

```bash
# Validate profile syntax
k7-console validate-profile --file myprofile.yaml

# Check for conflicting policies
k7-console profile-diff \
  --current-endpoint 192.168.1.100 \
  --new-profile myprofile.yaml

# Force reapplication
k7-console apply-profile \
  --profile myprofile.yaml \
  --target 192.168.1.100 \
  --force \
  --log-level debug
```

### High CPU Usage from Memory Scanner

```yaml
# Adjust scan frequency and exclusions
memory_scanner:
  state: enabled
  sensitivity: "medium"  # Changed from "high"
  scan_interval_seconds: 600  # Increased from 180
  exclude_processes:
    - "chrome"
    - "firefox"
    - "docker"
  exclude_memory_regions:
    - type: "shared_libraries"
    - path: "/usr/lib/*"
```

### AI Integration Not Responding

```python
import os
import requests

def test_ai_endpoint():
    """Verify AI provider connectivity"""
    # Test OpenAI
    openai_key = os.environ.get("K7_OPENAI_API_KEY")
    if openai_key:
        response = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": f"Bearer {openai_key}"},
            json={
                "model": "gpt-4-turbo",
                "messages": [{"role": "user", "content": "test"}],
                "max_tokens": 5
            },
            timeout=10
        )
        print(f"OpenAI status: {response.status_code}")
    
    # Test Claude
    claude_key = os.environ.get("K7_CLAUDE_API_KEY")
    if claude_key:
        response = requests.post(
            "https://api.anthropic.com/v1/messages",
            headers={
                "x-api-key": claude_key,
                "anthropic-version": "2023-06-01"
            },
            json={
                "model": "claude-3-opus-20240229",
                "messages": [{"role": "user", "content": "test"}],
                "max_tokens": 5
            },
            timeout=10
        )
        print(f"Claude status: {response.status_code}")

test_ai_endpoint()
```

### USB Filter Blocking Legitimate Devices

```bash
# List all connected USB devices with vendor/product IDs
lsusb

# Get device serial number
udevadm info --name=/dev/sdb | grep SERIAL

# Add to profile allowlist
# usb_filter:
#   allow_list:
#     - vendor_id: "0x0781"
#       product_id: "0x5581"
#       serial: "ACTUAL_SERIAL_HERE"
```

## Best Practices

1. **Use environment variables for all secrets** — Never hardcode API keys or credentials
2. **Test profiles in staging** — Validate with `k7-console validate-profile` before production deployment
3. **Enable verbose logging initially** — Reduce to `info` level after stabilization
4. **Implement webhook verification** — Always validate `X-K7-Signature` header
5. **Monitor AI integration costs** — Set `max_tokens` limits to control API usage
6. **Schedule scans during off-hours** — Adjust `scan_interval_seconds` based on workload patterns
7. **Maintain exception lists centrally** — Store in version control, apply via automation
8. **Use role-based profiles** — Don't apply one-size-fits-all policies across heterogeneous environments
