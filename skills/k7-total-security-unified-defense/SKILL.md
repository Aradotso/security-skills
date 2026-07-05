---
name: k7-total-security-unified-defense
description: Deploy and configure K7 Total Security 16.0.1195 multi-layered endpoint protection with AI-powered threat detection
triggers:
  - configure K7 Total Security protection profiles
  - set up K7 endpoint defense policies
  - deploy K7 security framework across platforms
  - integrate K7 with AI threat analysis
  - troubleshoot K7 Total Security deployment
  - create custom K7 security rules
  - manage K7 multi-platform protection
  - configure K7 web and USB filtering
---

# K7 Total Security Unified Defense Framework

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

K7 Total Security 16.0.1195 is a unified endpoint protection framework providing multi-layered defense across Windows, macOS, and Linux platforms. The system employs signatureless AI detection, behavioral monitoring, and centralized policy management with external AI integration (OpenAI/Claude) for threat analysis.

**Key Capabilities:**
- Cross-platform security profile deployment (Windows, macOS, Linux)
- Signatureless AI-based threat detection
- Modular protection layers (executable guard, memory scanner, USB/web filters)
- AI-powered incident analysis via OpenAI GPT and Claude APIs
- Centralized management console with remote policy application
- SIEM integration and webhook alerting

## Installation

### Prerequisites
- Administrative/root access on target systems
- Network connectivity for central management
- Optional: API keys for OpenAI/Claude integration

### Installation Methods

**Windows:**
```powershell
# Download and install agent
Invoke-WebRequest -Uri "https://k7-releases.example/K7-Agent-16.0.1195-x64.msi" -OutFile "K7-Agent.msi"
msiexec /i K7-Agent.msi /quiet /log install.log MANAGEMENT_SERVER="10.0.1.50"
```

**macOS:**
```bash
# Install via DMG or Homebrew
brew tap k7security/tap
brew install k7-total-security

# Configure management server
sudo k7-agent --set-server 10.0.1.50 --enroll
```

**Linux (Ubuntu/Debian):**
```bash
# Add repository and install
wget -qO - https://apt.k7security.example/pubkey.gpg | sudo apt-key add -
echo "deb https://apt.k7security.example/ stable main" | sudo tee /etc/apt/sources.list.d/k7.list
sudo apt update && sudo apt install k7-total-security

# Configure
sudo k7-agent configure --server 10.0.1.50
```

## Configuration

### Profile Structure

Security profiles are defined in YAML format and applied via the management console or CLI:

```yaml
profile_name: "production_strict"
version: "16.0.1195"
enforcement_level: "strict"  # strict | adaptive | permissive

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "quarantine"  # quarantine | block | alert
    exceptions:
      - path: "/opt/approved-tools/*"
      - hash: "sha256:abc123..."
    
  memory_scanner:
    state: enabled
    sensitivity: "high"  # low | medium | high
    scan_interval_seconds: 300
    
  usb_filter:
    state: enabled
    policy: "block_all"  # allow_all | read_only | block_all | read_only_for_unknown
    allow_list:
      - vendor_id: "0x0781"  # SanDisk
      - serial: "ABC-12345"
    
  web_filter:
    state: enabled
    categories_blocked:
      - "malware_distribution"
      - "phishing"
      - "cryptomining_scripts"
      - "c2_servers"
    allow_list:
      - domain: "*.trusted-cdn.com"
      - ip_range: "10.0.0.0/8"

ai_integration:
  incident_analysis:
    provider: "openai"  # openai | claude | none
    model: "gpt-4-turbo"
    endpoint: "https://api.openai.com/v1/chat/completions"
    api_key_env: "K7_OPENAI_API_KEY"
    max_tokens: 2000
    temperature: 0.3
    
  automated_response:
    provider: "claude"
    model: "claude-3-opus-20240229"
    endpoint: "https://api.anthropic.com/v1/messages"
    api_key_env: "K7_CLAUDE_API_KEY"
    webhook_retries: 3

logging:
  verbose: true
  retention_days: 90
  forward_to:
    - type: "syslog"
      server: "10.0.1.50:514"
    - type: "elasticsearch"
      url: "https://logs.example.io:9200"
      index_prefix: "k7-security"

notifications:
  email:
    enabled: true
    smtp_server: "smtp.example.com:587"
    from: "k7-alerts@example.com"
    recipients:
      - "secops@example.com"
  webhook:
    enabled: true
    url_env: "K7_SLACK_WEBHOOK"
    format: "slack"  # slack | discord | teams | generic
```

### Environment Variables

Set these in your deployment environment:

```bash
# AI Integration
export K7_OPENAI_API_KEY="sk-..."
export K7_CLAUDE_API_KEY="sk-ant-..."

# Webhook Notifications
export K7_SLACK_WEBHOOK="https://hooks.slack.com/services/..."

# Management Server
export K7_MGMT_SERVER="10.0.1.50"
export K7_MGMT_PORT="4451"
export K7_AUTH_CERT="/path/to/admin.pem"
```

## CLI Commands

### k7-console (Management Console)

```bash
# Apply profile to remote endpoint
k7-console --apply-profile production_strict.yaml \
           --target 192.168.8.105 \
           --auth-file $K7_AUTH_CERT \
           --log-level verbose \
           --timeout 120

# Bulk deployment to multiple hosts
k7-console --apply-profile production_strict.yaml \
           --target-file hosts.txt \
           --parallel 10 \
           --auth-file $K7_AUTH_CERT

# Query endpoint status
k7-console --query-status --target 192.168.8.105 \
           --auth-file $K7_AUTH_CERT \
           --output json

# Force policy sync
k7-console --sync-policy --target 192.168.8.105 \
           --auth-file $K7_AUTH_CERT

# View active threats
k7-console --list-threats --target 192.168.8.105 \
           --since "2026-01-01" \
           --severity high
```

### k7-agent (Local Agent)

```bash
# Check agent status
k7-agent status

# Manual scan
k7-agent scan --path /home/user/Downloads --deep

# Update threat definitions
k7-agent update --force

# View quarantine
k7-agent quarantine list

# Restore quarantined file
k7-agent quarantine restore --id QID-12345

# Generate diagnostic report
k7-agent diagnostics --output /tmp/k7-diag.zip

# Configure local settings
k7-agent configure --memory-limit 2G --scan-threads 4
```

## Code Examples

### Python: Policy Deployment Automation

```python
import yaml
import subprocess
import os
from pathlib import Path

class K7PolicyManager:
    def __init__(self, auth_cert_path: str, mgmt_server: str = None):
        self.auth_cert = auth_cert_path
        self.mgmt_server = mgmt_server or os.getenv('K7_MGMT_SERVER')
        
    def apply_profile(self, profile_path: str, targets: list[str], 
                     timeout: int = 120) -> dict:
        """Apply security profile to multiple endpoints."""
        results = {}
        
        for target in targets:
            cmd = [
                'k7-console',
                '--apply-profile', profile_path,
                '--target', target,
                '--auth-file', self.auth_cert,
                '--timeout', str(timeout),
                '--output', 'json'
            ]
            
            try:
                result = subprocess.run(
                    cmd, 
                    capture_output=True, 
                    text=True, 
                    check=True
                )
                results[target] = {
                    'success': True,
                    'output': result.stdout
                }
            except subprocess.CalledProcessError as e:
                results[target] = {
                    'success': False,
                    'error': e.stderr
                }
                
        return results
    
    def generate_profile(self, name: str, enforcement: str = 'adaptive',
                        ai_provider: str = 'openai') -> str:
        """Generate a security profile from template."""
        profile = {
            'profile_name': name,
            'version': '16.0.1195',
            'enforcement_level': enforcement,
            'protection_layers': {
                'executable_guard': {
                    'state': 'enabled',
                    'action_on_threat': 'quarantine'
                },
                'memory_scanner': {
                    'state': 'enabled',
                    'sensitivity': 'high',
                    'scan_interval_seconds': 300
                },
                'usb_filter': {
                    'state': 'enabled',
                    'policy': 'read_only_for_unknown'
                },
                'web_filter': {
                    'state': 'enabled',
                    'categories_blocked': [
                        'malware_distribution',
                        'phishing',
                        'cryptomining_scripts'
                    ]
                }
            },
            'ai_integration': {
                'incident_analysis': {
                    'provider': ai_provider,
                    'model': 'gpt-4-turbo' if ai_provider == 'openai' else 'claude-3-opus-20240229',
                    'api_key_env': f'K7_{ai_provider.upper()}_API_KEY',
                    'max_tokens': 2000
                }
            }
        }
        
        output_path = f'/tmp/k7-profile-{name}.yaml'
        with open(output_path, 'w') as f:
            yaml.dump(profile, f, default_flow_style=False)
            
        return output_path

# Usage
manager = K7PolicyManager(auth_cert_path='/etc/k7/admin.pem')

# Generate profile
profile_path = manager.generate_profile(
    name='dev_workstations',
    enforcement='adaptive',
    ai_provider='openai'
)

# Deploy to endpoints
targets = ['192.168.1.10', '192.168.1.11', '192.168.1.12']
results = manager.apply_profile(profile_path, targets)

for target, result in results.items():
    if result['success']:
        print(f"✓ {target}: Profile applied successfully")
    else:
        print(f"✗ {target}: {result['error']}")
```

### Bash: Automated Threat Response

```bash
#!/bin/bash
# k7-threat-monitor.sh - Monitor and respond to threats

K7_AUTH_CERT="${K7_AUTH_CERT:-/etc/k7/admin.pem}"
ALERT_WEBHOOK="${K7_SLACK_WEBHOOK}"

monitor_threats() {
    local target="$1"
    
    # Query for high-severity threats in last hour
    threats=$(k7-console --list-threats \
        --target "$target" \
        --auth-file "$K7_AUTH_CERT" \
        --since "1 hour ago" \
        --severity high \
        --output json)
    
    if [ "$(echo "$threats" | jq '.count')" -gt 0 ]; then
        echo "⚠️  High-severity threats detected on $target"
        
        # Send webhook alert
        send_alert "$target" "$threats"
        
        # Trigger AI analysis
        analyze_with_ai "$target" "$threats"
        
        # Auto-isolate if critical
        if [ "$(echo "$threats" | jq '.critical_count')" -gt 0 ]; then
            isolate_endpoint "$target"
        fi
    fi
}

send_alert() {
    local target="$1"
    local threats="$2"
    
    curl -X POST "$ALERT_WEBHOOK" \
        -H 'Content-Type: application/json' \
        -d @- <<EOF
{
    "text": "K7 Security Alert",
    "attachments": [{
        "color": "danger",
        "title": "Threats detected on $target",
        "text": "$(echo "$threats" | jq -r '.summary')",
        "fields": [
            {
                "title": "Severity",
                "value": "HIGH",
                "short": true
            },
            {
                "title": "Count",
                "value": "$(echo "$threats" | jq '.count')",
                "short": true
            }
        ]
    }]
}
EOF
}

analyze_with_ai() {
    local target="$1"
    local threats="$2"
    
    # Trigger AI analysis via K7 console
    k7-console --analyze-threat \
        --target "$target" \
        --auth-file "$K7_AUTH_CERT" \
        --threat-data "$threats" \
        --ai-provider openai \
        --output /var/log/k7/analysis-$(date +%s).json
}

isolate_endpoint() {
    local target="$1"
    
    echo "🔒 Isolating endpoint $target"
    
    k7-console --isolate \
        --target "$target" \
        --auth-file "$K7_AUTH_CERT" \
        --reason "Critical threat detected - automated response"
}

# Monitor list of endpoints
ENDPOINTS_FILE="${1:-/etc/k7/monitored-hosts.txt}"

while IFS= read -r endpoint; do
    monitor_threats "$endpoint"
done < "$ENDPOINTS_FILE"
```

### PowerShell: Windows Fleet Management

```powershell
# K7-FleetManager.ps1

class K7FleetManager {
    [string]$AuthCert
    [string]$MgmtServer
    
    K7FleetManager([string]$authCertPath) {
        $this.AuthCert = $authCertPath
        $this.MgmtServer = $env:K7_MGMT_SERVER
    }
    
    [hashtable] GetEndpointStatus([string[]]$targets) {
        $results = @{}
        
        foreach ($target in $targets) {
            $cmd = @(
                'k7-console',
                '--query-status',
                '--target', $target,
                '--auth-file', $this.AuthCert,
                '--output', 'json'
            )
            
            try {
                $output = & $cmd[0] $cmd[1..($cmd.Length-1)] 2>&1 | ConvertFrom-Json
                $results[$target] = @{
                    Success = $true
                    Status = $output
                }
            }
            catch {
                $results[$target] = @{
                    Success = $false
                    Error = $_.Exception.Message
                }
            }
        }
        
        return $results
    }
    
    [void] DeployProfileByGroup([hashtable]$groupProfiles) {
        foreach ($group in $groupProfiles.Keys) {
            $profilePath = $groupProfiles[$group].Profile
            $targets = $groupProfiles[$group].Endpoints
            
            Write-Host "Deploying profile '$profilePath' to $($targets.Count) endpoints in group '$group'"
            
            $cmd = @(
                'k7-console',
                '--apply-profile', $profilePath,
                '--target-file', ($targets -join "`n" | Out-File -FilePath "$env:TEMP\k7-targets-$group.txt" -PassThru).FullName,
                '--auth-file', $this.AuthCert,
                '--parallel', '10'
            )
            
            & $cmd[0] $cmd[1..($cmd.Length-1)]
        }
    }
    
    [object[]] AuditCompliance([string[]]$targets, [string]$requiredProfile) {
        $nonCompliant = @()
        
        $statuses = $this.GetEndpointStatus($targets)
        
        foreach ($target in $statuses.Keys) {
            if ($statuses[$target].Success) {
                $currentProfile = $statuses[$target].Status.active_profile
                if ($currentProfile -ne $requiredProfile) {
                    $nonCompliant += [PSCustomObject]@{
                        Endpoint = $target
                        CurrentProfile = $currentProfile
                        RequiredProfile = $requiredProfile
                    }
                }
            }
        }
        
        return $nonCompliant
    }
}

# Usage
$manager = [K7FleetManager]::new("C:\K7\admin.pem")

# Deploy profiles by organizational groups
$groupProfiles = @{
    'Engineering' = @{
        Profile = 'C:\K7\profiles\engineering-adaptive.yaml'
        Endpoints = @('10.0.1.10', '10.0.1.11', '10.0.1.12')
    }
    'Finance' = @{
        Profile = 'C:\K7\profiles\finance-strict.yaml'
        Endpoints = @('10.0.2.10', '10.0.2.11')
    }
}

$manager.DeployProfileByGroup($groupProfiles)

# Audit compliance
$nonCompliant = $manager.AuditCompliance(
    @('10.0.1.10', '10.0.1.11', '10.0.2.10'),
    'production_strict'
)

if ($nonCompliant.Count -gt 0) {
    Write-Warning "Found $($nonCompliant.Count) non-compliant endpoints:"
    $nonCompliant | Format-Table
}
```

## Common Patterns

### Pattern 1: Zero-Trust USB Policy

```yaml
profile_name: "zero_trust_usb"
version: "16.0.1195"

protection_layers:
  usb_filter:
    state: enabled
    policy: "block_all"
    allow_list:
      # Only allow specific approved devices by serial
      - vendor_id: "0x0781"
        product_id: "0x5581"
        serial: "APPROVED-SN-001"
      - vendor_id: "0x13FE"
        product_id: "0x4200"
        serial: "APPROVED-SN-002"
    
    auto_scan: true
    mount_read_only: true
    log_all_attempts: true

notifications:
  webhook:
    enabled: true
    url_env: "K7_USB_ALERT_WEBHOOK"
    notify_on:
      - "unauthorized_usb_attempt"
      - "usb_scan_threat_found"
```

### Pattern 2: Developer Workstation (Balanced)

```yaml
profile_name: "developer_balanced"
version: "16.0.1195"
enforcement_level: "adaptive"

protection_layers:
  executable_guard:
    state: enabled
    action_on_threat: "alert"  # Alert only, don't block
    exceptions:
      - path: "/home/*/.npm/*"
      - path: "/opt/nodejs/*"
      - path: "/usr/local/bin/docker*"
      - path: "C:\\Program Files\\Docker\\*"
      - path: "C:\\Users\\*\\AppData\\Local\\Programs\\*"
    
  memory_scanner:
    state: enabled
    sensitivity: "medium"
    scan_interval_seconds: 600  # Less frequent
    exclude_processes:
      - "node"
      - "python"
      - "docker"
    
  web_filter:
    state: enabled
    categories_blocked:
      - "malware_distribution"
      - "phishing"
    allow_list:
      - domain: "*.npmjs.com"
      - domain: "*.github.com"
      - domain: "*.docker.io"

ai_integration:
  incident_analysis:
    provider: "openai"
    model: "gpt-4-turbo"
    api_key_env: "K7_OPENAI_API_KEY"
```

### Pattern 3: Multi-Site SIEM Integration

```yaml
profile_name: "enterprise_siem"
version: "16.0.1195"

logging:
  verbose: true
  retention_days: 365
  forward_to:
    # Primary SIEM
    - type: "elasticsearch"
      url: "https://siem-primary.corp.example:9200"
      index_prefix: "k7-security"
      auth_env: "K7_ES_PRIMARY_TOKEN"
      
    # Backup SIEM
    - type: "elasticsearch"
      url: "https://siem-backup.corp.example:9200"
      index_prefix: "k7-security"
      auth_env: "K7_ES_BACKUP_TOKEN"
      
    # Syslog for legacy systems
    - type: "syslog"
      server: "10.0.10.50:514"
      protocol: "tcp"
      format: "rfc5424"
      
  enrichment:
    include_system_info: true
    include_user_context: true
    include_process_tree: true
```

## Troubleshooting

### Agent Not Connecting to Management Server

**Symptom:** Agent shows "Disconnected" status

```bash
# Check network connectivity
k7-agent diagnostics --network-test

# Verify certificate
openssl verify -CAfile /etc/k7/ca.crt $K7_AUTH_CERT

# Test management server reachability
nc -zv $K7_MGMT_SERVER 4451

# Check agent logs
k7-agent logs --tail 100 --level error

# Force re-enrollment
sudo k7-agent configure --server $K7_MGMT_SERVER --force-enroll
```

### High Memory Usage

**Symptom:** Agent consuming excessive RAM

```bash
# Check current memory usage
k7-agent status --verbose | grep Memory

# Adjust memory limits
k7-agent configure --memory-limit 1G --scan-threads 2

# Exclude large directories from real-time scanning
# Edit profile and add exceptions
k7-agent configure --exclude-path /mnt/large-storage
```

### Profile Application Fails

**Symptom:** `k7-console --apply-profile` returns error

```bash
# Validate profile syntax
k7-console --validate-profile production.yaml

# Check permissions
ls -l production.yaml
stat $K7_AUTH_CERT

# Test with verbose output
k7-console --apply-profile production.yaml \
           --target 192.168.1.10 \
           --auth-file $K7_AUTH_CERT \
           --log-level debug \
           --dry-run  # Test without applying

# Check target agent version compatibility
k7-console --query-status --target 192.168.1.10 \
           --auth-file $K7_AUTH_CERT | grep version
```

### AI Integration Not Working

**Symptom:** Threat analysis not invoking AI

```bash
# Verify API key is set
echo $K7_OPENAI_API_KEY | wc -c  # Should be >0

# Test API connectivity
curl -H "Authorization: Bearer $K7_OPENAI_API_KEY" \
     https://api.openai.com/v1/models

# Check AI integration logs
k7-console --query-logs --target 192.168.1.10 \
           --auth-file $K7_AUTH_CERT \
           --filter "ai_integration" \
           --since "1 hour ago"

# Verify profile has correct AI settings
grep -A10 "ai_integration:" production.yaml
```

### USB Filter Blocking Legitimate Device

**Symptom:** Approved USB device is blocked

```bash
# List currently connected USB devices
lsusb  # Linux
system_profiler SPUSBDataType  # macOS

# Get vendor/product IDs
# Add to allow_list in profile
k7-agent usb list --verbose

# Temporarily allow device (until next policy sync)
k7-agent usb allow --vendor-id 0x0781 --serial ABC123

# Update profile and re-apply
# Edit yaml file, then:
k7-console --apply-profile updated.yaml --target <host>
```

## Best Practices

1. **Always use environment variables for secrets** — Never hardcode API keys or credentials in profiles
2. **Test profiles in dry-run mode** before deploying to production
3. **Use adaptive enforcement** for developer workstations, strict for production servers
4. **Enable verbose logging** initially, reduce after stabilization
5. **Set up automated compliance audits** to detect configuration drift
6. **Integrate with existing SIEM** for centralized monitoring
7. **Define clear exception paths** for development tools and CI/CD systems
8. **Implement AI analysis** for high-severity threats to reduce response time
9. **Regularly update threat definitions** via `k7-agent update`
10. **Backup profile configurations** and version control them in Git
