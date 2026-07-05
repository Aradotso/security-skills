---
name: k7-total-security-configuration
description: Deploy and configure K7 Total Security 16.0.1195 unified defense framework with AI-powered threat detection
triggers:
  - "configure K7 Total Security profile"
  - "deploy K7 security policy across endpoints"
  - "set up K7 protection layers"
  - "integrate K7 with OpenAI threat analysis"
  - "create K7 security configuration"
  - "apply K7 endpoint protection profile"
  - "troubleshoot K7 security deployment"
  - "configure K7 multi-platform security"
---

# K7 Total Security Configuration Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

K7 Total Security 16.0.1195 is a unified defense framework providing multi-layered endpoint protection across Windows, macOS, and Linux platforms. This skill covers configuration management, profile deployment, AI integration for threat analysis, and operational management of the security ecosystem.

## Installation

### Prerequisites

- Administrative credentials for target endpoints
- Network connectivity to managed devices (port 4451 default)
- Valid K7 Total Security license
- Authentication certificates (PEM format) for remote management

### CLI Installation

```bash
# Download and install the K7 console management tool
curl -O https://k7computing.com/downloads/k7-console-latest.tar.gz
tar -xzf k7-console-latest.tar.gz
cd k7-console
sudo ./install.sh

# Verify installation
k7-console --version
# Expected: K7 Console Manager v16.0.1195
```

### Agent Deployment

```bash
# Deploy agent to remote endpoint
k7-console --deploy-agent \
  --target 192.168.1.100 \
  --platform windows \
  --auth-file ~/.k7/admin.pem \
  --install-path "C:\\Program Files\\K7\\Agent"
```

## Core Configuration

### Profile Structure

K7 profiles are defined in YAML with these key sections:

```yaml
profile_name: "CustomSecurityProfile"
version: "16.0.1195"
enforcement_level: "adaptive"  # adaptive | strict | monitor

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "quarantine"  # quarantine | block | alert
    exceptions: []
  
  memory_scanner:
    state: enabled
    sensitivity: "high"  # low | medium | high
    scan_interval_seconds: 300
  
  usb_filter:
    state: enabled
    policy: "read_only_for_unknown"  # block_all | read_only_for_unknown | allow_all
    allow_list: []
  
  web_filter:
    state: enabled
    categories_blocked: []
    allow_list: []

ai_integration:
  incident_analysis:
    provider: "openai"  # openai | claude | none
    model: "gpt-4-turbo"
    max_tokens: 2000
    temperature: 0.3
  
  automated_response:
    provider: "claude"
    model: "claude-3-opus-20240229"
    webhook_retries: 3

logging:
  verbose: false
  retention_days: 90
  forward_to: []

notifications:
  email:
    enabled: false
    recipients: []
  webhook:
    enabled: false
    url: ""
```

### Basic Profile Example

```yaml
profile_name: "developer_workstation"
version: "16.0.1195"
enforcement_level: "adaptive"

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "alert"
    exceptions:
      - path: "/home/*/projects/*"
      - path: "/opt/vscode/*"
      - path: "C:\\DevTools\\*"
  
  memory_scanner:
    state: enabled
    sensitivity: "medium"
    scan_interval_seconds: 600
  
  usb_filter:
    state: enabled
    policy: "read_only_for_unknown"
    allow_list:
      - vendor_id: "0x0781"  # SanDisk
      - vendor_id: "0x8564"  # Transcend
  
  web_filter:
    state: enabled
    categories_blocked:
      - "malware_distribution"
      - "phishing"
      - "cryptomining_scripts"
    allow_list:
      - domain: "*.github.com"
      - domain: "*.npmjs.org"

ai_integration:
  incident_analysis:
    provider: "openai"
    model: "gpt-4-turbo"
    max_tokens: 1500
    temperature: 0.2

logging:
  verbose: true
  retention_days: 30
  forward_to:
    - syslog_server: "logs.internal:514"

notifications:
  webhook:
    enabled: true
    url: "${SLACK_WEBHOOK_URL}"
```

## CLI Commands

### Profile Management

```bash
# Apply profile to single endpoint
k7-console --apply-profile profile.yaml \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem \
  --log-level verbose

# Apply profile to multiple endpoints
k7-console --apply-profile profile.yaml \
  --target-list endpoints.txt \
  --auth-file ~/.k7/admin.pem \
  --parallel 5

# Validate profile syntax
k7-console --validate-profile profile.yaml

# Export current profile from endpoint
k7-console --export-profile \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem \
  --output current_profile.yaml
```

### Status and Monitoring

```bash
# Check protection status
k7-console --status \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem

# View recent alerts
k7-console --alerts \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem \
  --last 24h \
  --format json

# Test connectivity to endpoint
k7-console --ping \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem
```

### Policy Operations

```bash
# Add exception to existing profile
k7-console --add-exception \
  --target 192.168.1.50 \
  --layer executable_guard \
  --path "/opt/custom-app/*" \
  --auth-file ~/.k7/admin.pem

# Update scan sensitivity
k7-console --update-layer \
  --target 192.168.1.50 \
  --layer memory_scanner \
  --set sensitivity=high \
  --auth-file ~/.k7/admin.pem

# Force immediate scan
k7-console --scan-now \
  --target 192.168.1.50 \
  --type full \
  --auth-file ~/.k7/admin.pem
```

## AI Integration Configuration

### OpenAI Setup

```yaml
ai_integration:
  incident_analysis:
    provider: "openai"
    endpoint: "https://api.openai.com/v1/chat/completions"
    model: "gpt-4-turbo-2024"
    max_tokens: 2000
    temperature: 0.3
    api_key: "${OPENAI_API_KEY}"
    
    system_prompt: |
      You are a cybersecurity analyst. Analyze the following threat event
      and provide: 1) threat classification, 2) severity (1-10),
      3) recommended action, 4) detailed explanation.
```

### Claude Setup

```yaml
ai_integration:
  automated_response:
    provider: "claude"
    endpoint: "https://api.anthropic.com/v1/messages"
    model: "claude-3-opus-20240229"
    max_tokens: 1500
    api_key: "${ANTHROPIC_API_KEY}"
    webhook_retries: 3
    
    response_templates:
      - threat_type: "memory_injection"
        action: "terminate_process"
      - threat_type: "suspicious_network"
        action: "revoke_network_access"
```

### Environment Variables

```bash
# Set API keys securely
export OPENAI_API_KEY="sk-proj-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."

# Apply profile with environment variable substitution
k7-console --apply-profile profile.yaml \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem \
  --expand-env-vars
```

## Common Patterns

### Enterprise Multi-Platform Deployment

```yaml
profile_name: "enterprise_standard"
version: "16.0.1195"
enforcement_level: "strict"

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "quarantine"
    exceptions:
      - path: "C:\\Program Files\\CompanyApp\\*"
      - path: "/opt/company-tools/*"
      - path: "/Applications/CompanyApp.app/*"
  
  memory_scanner:
    state: enabled
    sensitivity: "high"
    scan_interval_seconds: 180
  
  usb_filter:
    state: enabled
    policy: "block_all"
    allow_list:
      - vendor_id: "0x046D"  # Logitech peripherals
      - serial_number: "APPROVED-001"
  
  web_filter:
    state: enabled
    categories_blocked:
      - "malware_distribution"
      - "phishing"
      - "adult_content"
      - "gambling"
      - "cryptomining_scripts"
    allow_list:
      - domain: "*.company.internal"

logging:
  verbose: true
  retention_days: 365
  forward_to:
    - elasticsearch: "https://${ES_HOST}:9200"
      index: "k7-security-events"
    - syslog_server: "${SYSLOG_HOST}:514"

notifications:
  email:
    enabled: true
    recipients:
      - "secops@company.com"
      - "it-alerts@company.com"
  webhook:
    enabled: true
    url: "${TEAMS_WEBHOOK_URL}"
```

### Development Environment Profile

```yaml
profile_name: "dev_environment"
version: "16.0.1195"
enforcement_level: "monitor"

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "alert"
    exceptions:
      - path: "/home/*/dev/*"
      - path: "/home/*/.npm/*"
      - path: "/home/*/.cargo/*"
      - path: "C:\\Users\\*\\source\\*"
      - path: "C:\\Users\\*\\.vscode\\*"
  
  memory_scanner:
    state: enabled
    sensitivity: "low"
    scan_interval_seconds: 1800
  
  usb_filter:
    state: disabled
  
  web_filter:
    state: enabled
    categories_blocked:
      - "malware_distribution"
      - "phishing"
    allow_list:
      - domain: "*"  # Allow all, block only known threats

logging:
  verbose: false
  retention_days: 7
  forward_to: []

notifications:
  email:
    enabled: false
  webhook:
    enabled: false
```

### Batch Deployment Script

```bash
#!/bin/bash
# deploy_k7_fleet.sh

PROFILE="enterprise_standard.yaml"
AUTH_FILE="~/.k7/admin.pem"
ENDPOINT_LIST="endpoints.txt"

# Validate profile
echo "Validating profile..."
k7-console --validate-profile "$PROFILE"
if [ $? -ne 0 ]; then
  echo "Profile validation failed"
  exit 1
fi

# Deploy to all endpoints
echo "Deploying to fleet..."
k7-console --apply-profile "$PROFILE" \
  --target-list "$ENDPOINT_LIST" \
  --auth-file "$AUTH_FILE" \
  --parallel 10 \
  --log-level info \
  --timeout 300 \
  --retry-failures 2 \
  > deployment_$(date +%Y%m%d_%H%M%S).log

# Generate report
echo "Generating deployment report..."
k7-console --fleet-status \
  --target-list "$ENDPOINT_LIST" \
  --auth-file "$AUTH_FILE" \
  --format html \
  --output fleet_status.html
```

## Troubleshooting

### Connection Issues

```bash
# Test network connectivity
k7-console --ping \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem \
  --verbose

# Check firewall rules (ensure port 4451 is open)
sudo ufw allow 4451/tcp  # Linux
netsh advfirewall firewall add rule name="K7 Management" dir=in action=allow protocol=TCP localport=4451  # Windows

# Verify certificate permissions
chmod 600 ~/.k7/admin.pem
```

### Profile Application Failures

```bash
# Check profile syntax
k7-console --validate-profile profile.yaml

# View detailed error logs
k7-console --apply-profile profile.yaml \
  --target 192.168.1.50 \
  --auth-file ~/.k7/admin.pem \
  --log-level verbose 2>&1 | tee apply.log

# Common issues:
# - Invalid YAML indentation
# - Unsupported parameter values
# - Platform-specific path format mismatches
```

### Agent Not Responding

```bash
# Restart agent service
# Linux/macOS:
sudo systemctl restart k7-agent

# Windows (PowerShell as Admin):
Restart-Service -Name "K7SecurityAgent"

# Check agent logs
# Linux: /var/log/k7/agent.log
# macOS: /Library/Logs/K7/agent.log
# Windows: C:\ProgramData\K7\Logs\agent.log
tail -f /var/log/k7/agent.log
```

### Performance Issues

```yaml
# Adjust scan frequency for better performance
protection_layers:
  memory_scanner:
    scan_interval_seconds: 900  # Increase from 300
    sensitivity: "medium"  # Reduce from high

# Exclude high-activity directories
protection_layers:
  executable_guard:
    exceptions:
      - path: "/var/lib/docker/*"
      - path: "C:\\Windows\\Temp\\*"
```

### AI Integration Failures

```bash
# Test API connectivity
curl -X POST https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4-turbo","messages":[{"role":"user","content":"test"}],"max_tokens":5}'

# Check API key format
echo $OPENAI_API_KEY | wc -c  # Should be ~51 chars for OpenAI
echo $ANTHROPIC_API_KEY | wc -c  # Should be ~108 chars for Claude

# Enable debug logging for AI calls
ai_integration:
  debug_mode: true
  log_requests: true
  log_responses: true
```

### USB Filter Issues

```bash
# List connected USB devices with vendor IDs
lsusb  # Linux/macOS
Get-PnpDevice -Class USB | Select-Object FriendlyName, InstanceId  # Windows PowerShell

# Add device to allow list
k7-console --add-exception \
  --target 192.168.1.50 \
  --layer usb_filter \
  --vendor-id "0x0781" \
  --description "SanDisk Approved" \
  --auth-file ~/.k7/admin.pem
```

## Advanced Configuration

### Custom Threat Response Actions

```yaml
ai_integration:
  automated_response:
    provider: "claude"
    model: "claude-3-opus-20240229"
    
    custom_actions:
      - name: "isolate_endpoint"
        trigger: "severity >= 9"
        script: |
          #!/bin/bash
          # Disable all network interfaces except management
          for iface in $(ip link show | grep -v "lo\|k7mgmt" | awk -F: '{print $2}'); do
            ip link set $iface down
          done
          
      - name: "capture_memory_dump"
        trigger: "threat_type == 'memory_injection'"
        script: |
          #!/bin/bash
          timestamp=$(date +%Y%m%d_%H%M%S)
          gcore -o "/var/log/k7/dumps/dump_${timestamp}" ${THREAT_PID}
```

### Multi-Tier Policy Inheritance

```yaml
# base_policy.yaml
profile_name: "base_security"
version: "16.0.1195"
enforcement_level: "adaptive"

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "quarantine"
  memory_scanner:
    state: enabled
    sensitivity: "medium"

---
# dev_override.yaml (extends base_policy.yaml)
extends: "base_policy.yaml"
profile_name: "dev_security"

protection_layers:
  executable_guard:
    action_on_threat: "alert"  # Override
    exceptions:  # Add to base
      - path: "/home/*/dev/*"
```

## Security Best Practices

1. **Never commit API keys or certificates to version control**
   ```bash
   # Use .gitignore
   echo "*.pem" >> .gitignore
   echo ".env" >> .gitignore
   echo "*credentials*" >> .gitignore
   ```

2. **Rotate authentication certificates regularly**
   ```bash
   k7-console --generate-cert \
     --output ~/.k7/admin_$(date +%Y%m).pem \
     --validity-days 30
   ```

3. **Use least privilege profiles for development**
   - Start with `enforcement_level: "monitor"`
   - Gradually increase to `"adaptive"` then `"strict"`

4. **Log all configuration changes**
   ```bash
   k7-console --apply-profile profile.yaml ... \
     | tee -a /var/log/k7/config_changes.log
   ```

5. **Test profiles in isolated environment first**
   ```bash
   # Deploy to test VM before production
   k7-console --apply-profile new_profile.yaml \
     --target test-vm.local \
     --auth-file ~/.k7/admin.pem
   ```
