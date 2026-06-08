---
name: security-investigator-automation
description: Automated security investigations using Microsoft Sentinel, Defender XDR, GitHub Copilot Agent Skills, and MCP servers for natural language threat hunting
triggers:
  - "investigate security incident with Sentinel"
  - "automate threat hunting in Microsoft Defender"
  - "run security investigation with Copilot"
  - "analyze IoC with threat intelligence"
  - "create security investigation agent skill"
  - "query Sentinel with natural language"
  - "generate security investigation report"
  - "hunt threats across Microsoft security stack"
---

# security-investigator-automation

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Automated security investigation framework combining GitHub Copilot Agent Skills, VS Code, and Model Context Protocol (MCP) servers to enable natural language security investigations across Microsoft Sentinel, Defender XDR, Graph API, and threat intelligence platforms. Includes 25 specialized investigation workflows with KQL queries, threat intelligence enrichment, and automated HTML report generation.

## Installation

```powershell
# Clone repository
git clone https://github.com/SCStelz/security-investigator.git
cd security-investigator
code .

# Set up Python virtual environment
python -m venv .venv

# Windows
.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate

# Install dependencies (hash-verified recommended)
pip install --require-hashes -r requirements.lock
# OR without hash verification
pip install -r requirements.txt
```

## Configuration

### 1. Sentinel and Azure Configuration

```powershell
# Copy template
copy config.json.template config.json
```

Edit `config.json`:

```json
{
  "workspace_id": "your-sentinel-workspace-id",
  "tenant_id": "your-azure-tenant-id",
  "subscription_id": "your-azure-subscription-id",
  "resource_group": "your-resource-group",
  "workspace_name": "your-sentinel-workspace-name"
}
```

### 2. Threat Intelligence API Keys

```powershell
# Copy template
copy .env.template .env
```

Edit `.env`:

```bash
IPINFO_TOKEN=your_ipinfo_token
ABUSEIPDB_API_KEY=your_abuseipdb_key
VPNAPI_KEY=your_vpnapi_key
SHODAN_API_KEY=your_shodan_key
```

### 3. MCP Server Configuration

```powershell
# Copy template
copy .vscode\mcp.json.template .vscode\mcp.json
```

The template includes pre-configured MCP servers:
- **@sentinel-datalake** - Sentinel log analytics
- **@sentinel-triage** - Defender XDR Advanced Hunting
- **@graph-api** - Entra ID/Graph API queries
- **@kql-search** - Schema documentation
- **@microsoft-learn** - Documentation reference
- **@azure-mcp-server** - Azure Resource Manager + Monitor
- **@sentinel-graph** - Relationship mapping (private preview)

Custom local MCP apps (geomap, heatmap, incident-comment) are also pre-configured.

## Core Workflow

### 1. Start Investigation with Copilot Chat

Open VS Code Copilot Chat (Ctrl+Shift+I) in **Agent mode**.

**Recommended first command:**
```
"Run a threat pulse scan"
```

This executes the **threat-pulse** skill which scans 9 security domains and provides prioritized findings with drill-down recommendations.

### 2. Investigation Skills

The system auto-detects skills based on keywords:

#### Core Investigations

**User Investigation:**
```
"Investigate user@domain.com for the last 7 days"
```

Generates comprehensive user analysis:
- Entra ID sign-in activity with anomaly detection
- MFA status and authentication methods
- Risky sign-ins and Identity Protection alerts
- Device registrations and compliance
- Audit log changes (role assignments, permissions)
- Associated incidents and alerts
- HTML report with IP enrichment

**Incident Investigation:**
```
"Analyze incident 12345"
```

Deep-dive incident analysis:
- Criticality assessment (CRITICAL/HIGH/MEDIUM/LOW)
- Entity extraction (users, devices, IPs, processes)
- Recursive entity investigation
- MITRE ATT&CK mapping
- Evidence correlation across Sentinel and Defender XDR

**IoC Investigation:**
```
"Is this IP malicious? 203.0.113.42"
```

Multi-source threat intelligence:
- Defender Threat Intelligence API
- Sentinel ThreatIntelligenceIndicator table
- AbuseIPDB reputation scores
- Shodan port/service/CVE data
- VPN detection (vpnapi.io)
- Geolocation with IPInfo
- Organizational exposure assessment

**Device Investigation:**
```
"Investigate computer WIN-CLIENT01"
```

Endpoint security analysis:
- Defender alerts and vulnerabilities
- Device compliance status
- Logged-on users and sessions
- Process execution (DeviceProcessEvents)
- Network connections (DeviceNetworkEvents)
- File operations (DeviceFileEvents)

#### Authentication & Access

**Authentication Tracing:**
```
"Trace authentication for SessionId abc123"
```

Forensic authentication chain analysis:
- SessionId correlation across sign-ins
- Token reuse vs interactive MFA events
- Geographic anomaly detection
- Authentication method changes

**Conditional Access Policy Investigation:**
```
"Investigate Conditional Access failures for user@domain.com"
```

Policy forensics:
- CA policy state changes (enabled/disabled)
- Sign-in failure correlation
- Device compliance bypass detection
- Policy evaluation timeline

#### Behavioral Analysis

**Scope Drift Detection (Device):**
```
"Detect device drift for WIN-SERVER01"
```

Baseline deviation analysis:
- Configurable baseline window (default 30 days)
- 5-dimension Drift Score: Volume, Processes, Accounts, Command Chains, Code Signing
- Heartbeat corroboration for uptime validation
- Fleet-wide or single-device analysis

**Scope Drift Detection (SPN):**
```
"Detect scope drift for service principal app-id-123"
```

Service principal behavioral change:
- 90-day baseline vs 7-day comparison
- Correlated with AuditLogs, SecurityAlert, DeviceNetworkEvents
- New resource access patterns
- Permission elevation detection

#### Posture & Exposure

**Threat Pulse (Quick Scan):**
```
"Run a morning threat pulse"
```

Broad-spectrum scan with color-coded verdicts:
- 🔴 Escalate - Critical findings requiring immediate action
- 🟠 Investigate - Medium-risk anomalies
- 🟡 Monitor - Low-risk observations
- ✅ Clear - No issues detected

Covers:
- Active incidents (Sentinel + Defender XDR)
- Identity risks (humans + non-human identities)
- Endpoint threats
- Exposure management (Defender EASM)
- Email threats (Defender for Office 365)
- UEBA anomalies
- Authentication spray attacks
- Privileged operations
- Recent CVEs in environment

**Exposure Management:**
```
"Run exposure management scan"
```

Attack surface analysis:
- Defender EASM integration
- Internet-facing assets
- Shadow IT discovery
- SSL/TLS certificate inventory
- Domain and subdomain enumeration

**Identity Posture:**
```
"Analyze identity posture"
```

Entra ID security assessment:
- Guest user risk
- Privileged role assignments
- MFA coverage gaps
- Legacy authentication usage
- Inactive high-privilege accounts

## Python Utilities

### Generate Investigation Report

```python
from report_generator import generate_investigation_report

# Generate HTML report from investigation data
report_data = {
    "user": "user@domain.com",
    "investigation_period": "7 days",
    "sign_ins": [...],  # Sign-in data from Sentinel
    "alerts": [...],    # Security alerts
    "devices": [...]    # Device registrations
}

generate_investigation_report(
    data=report_data,
    output_file="investigation_report.html"
)
```

### IP Enrichment

```python
from enrich_ips import enrich_ip_address

# Enrich IP with threat intelligence
enrichment = enrich_ip_address("203.0.113.42")

print(f"Country: {enrichment['country']}")
print(f"ISP: {enrichment['isp']}")
print(f"Is VPN: {enrichment['is_vpn']}")
print(f"Abuse Score: {enrichment['abuse_confidence_score']}")
print(f"Shodan Services: {enrichment['shodan_services']}")
print(f"Associated CVEs: {enrichment['shodan_vulns']}")
```

### Batch Report Generation from JSON

```python
# Generate report from JSON investigation output
python generate_report_from_json.py investigation_output.json

# Creates HTML report with:
# - Executive summary
# - Timeline visualization
# - Risk scoring matrix
# - Enriched IP addresses with threat intel
# - MITRE ATT&CK mapping
# - Recommended actions
```

## KQL Query Patterns

The framework includes a verified query library in `queries/` directory:

### User Activity Query

```kql
// queries/user_signins.kql
SigninLogs
| where TimeGenerated > ago(7d)
| where UserPrincipalName == "user@domain.com"
| extend GeoLocation = strcat(Location.city, ", ", Location.countryOrRegion)
| extend RiskState = case(
    RiskLevelDuringSignIn == "high", "🔴 High Risk",
    RiskLevelDuringSignIn == "medium", "🟠 Medium Risk",
    RiskLevelDuringSignIn == "low", "🟡 Low Risk",
    "✅ No Risk"
)
| project
    TimeGenerated,
    UserPrincipalName,
    AppDisplayName,
    IPAddress,
    GeoLocation,
    RiskState,
    AuthenticationRequirement,
    ConditionalAccessStatus,
    DeviceDetail.displayName
| order by TimeGenerated desc
```

### Device Process Baseline

```kql
// queries/device_process_baseline.kql
let BaselineWindow = 30d;
let AnalysisWindow = 7d;
let DeviceName = "WIN-CLIENT01";
let Baseline = DeviceProcessEvents
| where TimeGenerated between (ago(BaselineWindow) .. ago(AnalysisWindow))
| where DeviceName == DeviceName
| summarize
    BaselineProcesses = make_set(FileName),
    BaselineAccounts = make_set(AccountName),
    BaselineVolume = count()
by DeviceName;
let Recent = DeviceProcessEvents
| where TimeGenerated > ago(AnalysisWindow)
| where DeviceName == DeviceName
| summarize
    RecentProcesses = make_set(FileName),
    RecentAccounts = make_set(AccountName),
    RecentVolume = count()
by DeviceName;
Baseline
| join kind=inner Recent on DeviceName
| extend
    NewProcesses = set_difference(RecentProcesses, BaselineProcesses),
    NewAccounts = set_difference(RecentAccounts, BaselineAccounts),
    VolumeChange = (todouble(RecentVolume) - todouble(BaselineVolume)) / todouble(BaselineVolume) * 100
| extend DriftScore = case(
    array_length(NewProcesses) > 10 or VolumeChange > 50, 90,
    array_length(NewProcesses) > 5 or VolumeChange > 25, 70,
    array_length(NewProcesses) > 0 or VolumeChange > 10, 40,
    0
)
| project
    DeviceName,
    DriftScore,
    NewProcessCount = array_length(NewProcesses),
    NewProcesses,
    VolumeChange = round(VolumeChange, 2),
    Verdict = case(
        DriftScore >= 80, "🔴 High Drift - Investigate",
        DriftScore >= 50, "🟠 Medium Drift - Monitor",
        DriftScore >= 20, "🟡 Low Drift - Baseline Update",
        "✅ Normal Behavior"
    )
```

### Incident Entity Extraction

```kql
// queries/incident_entities.kql
SecurityIncident
| where IncidentNumber == "12345"
| mv-expand AlertIds
| join kind=inner (
    SecurityAlert
    | mv-expand Entities
    | extend EntityType = tostring(Entities.Type)
    | extend EntityValue = case(
        EntityType == "account", tostring(Entities.Name),
        EntityType == "host", tostring(Entities.HostName),
        EntityType == "ip", tostring(Entities.Address),
        EntityType == "process", tostring(Entities.ProcessId),
        EntityType == "file", tostring(Entities.Name),
        tostring(Entities)
    )
) on $left.AlertIds == $right.SystemAlertId
| summarize
    EntityCount = count(),
    Entities = make_set(EntityValue)
by IncidentNumber, EntityType
| order by EntityCount desc
```

## MCP Server Interaction Patterns

### Query Sentinel Data Lake

```
@sentinel-datalake execute the user sign-in query for user@domain.com
```

The agent will:
1. Load appropriate query from `queries/` directory
2. Replace parameters (UPN, time range)
3. Execute via Sentinel Data Lake MCP server
4. Parse results and apply skill-specific analysis

### Advanced Hunting (Defender XDR)

```
@sentinel-triage hunt for process execution on device WIN-CLIENT01
```

Uses DeviceProcessEvents, DeviceNetworkEvents, DeviceFileEvents across Defender for Endpoint.

### Graph API Queries

```
@graph-api get user details for user@domain.com including risk state and authentication methods
```

Queries Entra ID for:
- User profile and attributes
- Sign-in activity
- Risk detections (Identity Protection)
- Authentication methods
- Device registrations
- Group memberships
- Role assignments

## Skill Development

Create custom investigation skills in `.github/skills/`:

```markdown
---
name: custom-investigation
description: Custom security investigation workflow
triggers:
  - "custom investigation"
  - "run custom hunt"
---

# Custom Investigation Skill

## Objective
[Describe what this skill investigates]

## Detection Logic
[KQL queries and analysis patterns]

## Risk Assessment
- 🔴 CRITICAL: [conditions]
- 🟠 HIGH: [conditions]
- 🟡 MEDIUM: [conditions]
- ✅ LOW: [conditions]

## Workflow

1. **Data Collection**
   - Query: `queries/custom_query.kql`
   - Sources: [Sentinel tables]

2. **Analysis**
   - [Analysis steps]

3. **Enrichment**
   - [Threat intel sources]

4. **Reporting**
   - [Output format]

## Example

User: "Run custom investigation for user@domain.com"
Agent: [Expected workflow execution]
```

Reference the skill in `.github/copilot-instructions.md`:

```markdown
## Skill: custom-investigation
**Triggers:** "custom investigation", "run custom hunt"
**Path:** .github/skills/custom-investigation/SKILL.md
```

## Troubleshooting

### MCP Server Connection Issues

```powershell
# Verify MCP configuration
cat .vscode\mcp.json

# Check logs in VS Code
# View -> Output -> Select "MCP" from dropdown

# Test Sentinel connectivity
az login
az account show
```

### Authentication Errors

```powershell
# Refresh Azure credentials
az logout
az login --tenant your-tenant-id

# Verify Sentinel access
az monitor log-analytics workspace show \
  --resource-group your-rg \
  --workspace-name your-workspace
```

### Python Dependencies

```powershell
# Reinstall with hash verification
pip install --force-reinstall --require-hashes -r requirements.lock

# Check installed packages
pip list

# Verify environment activation
python -c "import sys; print(sys.prefix)"
```

### Missing Threat Intel Data

- **IPInfo:** Free tier allows 50,000 requests/month - check env var `IPINFO_TOKEN`
- **AbuseIPDB:** Free tier allows 1,000 requests/day - check `ABUSEIPDB_API_KEY`
- **Shodan:** Requires paid API key - check `SHODAN_API_KEY`
- **VPNapi:** Free tier for basic checks - check `VPNAPI_KEY`

Enrichment gracefully degrades if APIs are unavailable.

### KQL Query Errors

```
# Test query syntax in Sentinel portal first
# Logs -> Run query manually
# Copy working query to queries/ directory

# Verify table schema with @kql-search
@kql-search show schema for SigninLogs
```

### Report Generation Failures

```python
# Debug report generation
python generate_report_from_json.py investigation_output.json --debug

# Validate JSON structure
import json
with open('investigation_output.json') as f:
    data = json.load(f)
    print(json.dumps(data, indent=2))
```

### Copilot Skill Not Triggering

1. Check trigger keywords in `.github/skills/[skill-name]/SKILL.md`
2. Verify skill is referenced in `.github/copilot-instructions.md`
3. Reload VS Code window (Ctrl+Shift+P -> "Reload Window")
4. Use exact trigger phrases or ask: "What skills do you have access to?"

## Advanced Patterns

### Chained Investigations

```
1. "Run threat pulse scan"
   → Identifies 3 high-risk users

2. "Investigate user1@domain.com for last 7 days"
   → Detects suspicious sign-in from 203.0.113.42

3. "Is this IP malicious? 203.0.113.42"
   → Confirms C2 infrastructure

4. "Hunt for this IP across all devices"
   → @sentinel-datalake query DeviceNetworkEvents
```

### Automated Weekly Reports

```python
# weekly_hunt.py
from datetime import datetime, timedelta
from report_generator import generate_investigation_report

# Query last 7 days
end_date = datetime.utcnow()
start_date = end_date - timedelta(days=7)

# Generate report for all high-risk users
high_risk_users = [...]  # From Identity Protection
for user in high_risk_users:
    generate_investigation_report(
        user=user,
        start_date=start_date,
        end_date=end_date,
        output_file=f"reports/{user}_{end_date.strftime('%Y%m%d')}.html"
    )
```

### Custom Threat Intelligence Integration

```python
# custom_ti_enrichment.py
import requests
import os

def custom_ti_lookup(ioc, ioc_type):
    """Integrate custom threat intel source"""
    api_key = os.getenv('CUSTOM_TI_API_KEY')
    response = requests.get(
        f"https://custom-ti-api.com/v1/{ioc_type}/{ioc}",
        headers={"Authorization": f"Bearer {api_key}"}
    )
    return response.json()

# Use in enrich_ips.py
from custom_ti_enrichment import custom_ti_lookup
enrichment['custom_ti'] = custom_ti_lookup(ip_address, 'ip')
```

## Resources

- **Video Walkthrough:** [YouTube Demo](https://youtu.be/3UFqWA4cmoE?t=1470)
- **Repository:** https://github.com/SCStelz/security-investigator
- **Issues:** https://github.com/SCStelz/security-investigator/issues
- **Copilot Instructions:** `.github/copilot-instructions.md`
- **KQL Query Library:** `queries/` directory
- **Skill Catalog:** `.github/skills/` directory (25 specialized skills)
