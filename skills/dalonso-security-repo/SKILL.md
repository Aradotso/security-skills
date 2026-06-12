---
name: dalonso-security-repo
description: Microsoft Security threat hunting queries, KQL examples, Sentinel workbooks, and security analytics notebooks
triggers:
  - how do I use KQL queries from Dalonso Security Repo
  - show me threat hunting examples for Microsoft Sentinel
  - help me with Microsoft security analytics notebooks
  - how to implement Sentinel workbooks from Dalonso repo
  - find KQL queries for threat detection
  - use David Alonso security resources for threat hunting
  - implement Microsoft Defender threat hunting queries
  - analyze security data with Dalonso notebooks
---

# Dalonso Security Repo

> Skill by [ara.so](https://ara.so) — Security Skills collection.

A comprehensive collection of Microsoft Security resources including KQL queries, threat hunting use cases, Microsoft Sentinel workbooks, and Jupyter notebooks for security analytics and investigation workflows.

## What This Repository Provides

The Dalonso Security Repo is a curated knowledge base for Microsoft Security technologies:

- **Threat Hunting Use Cases**: Real-world KQL (Kusto Query Language) queries for proactive threat detection
- **Microsoft Sentinel Workbooks**: Custom visualization and analysis dashboards
- **Security Analytics Notebooks**: Jupyter notebooks for advanced investigation and automation
- **Practical Examples**: Production-ready queries and workflows for security operations

## Installation

Clone the repository to access all resources:

```bash
git clone https://github.com/davidalonsod/Dalonso-Security-Repo.git
cd Dalonso-Security-Repo
```

For Jupyter notebooks, install required dependencies:

```bash
pip install jupyter msticpy pandas azure-monitor-query azure-identity
```

## Repository Structure

The repository is organized by security tool and use case:

```
Dalonso-Security-Repo/
├── Threat-Hunting/          # KQL queries for threat hunting
├── Sentinel-Workbooks/      # Custom Sentinel workbooks (JSON)
├── Notebooks/               # Jupyter notebooks (.ipynb)
├── Detection-Rules/         # Analytics rules and detections
└── Investigation-Queries/   # Ad-hoc investigation KQL
```

## Working with KQL Queries

### Basic KQL Query Structure

KQL queries in this repo follow standard Microsoft Sentinel/Log Analytics syntax:

```kql
// Example: Detecting suspicious PowerShell execution
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4688
| where Process has "powershell.exe"
| where CommandLine has_any ("bypass", "encoded", "hidden")
| project TimeGenerated, Computer, Account, CommandLine, ParentProcessName
| order by TimeGenerated desc
```

### Using Queries in Microsoft Sentinel

1. Navigate to **Microsoft Sentinel** > **Logs**
2. Copy the KQL query from the repository
3. Paste into the query editor
4. Adjust time ranges and parameters as needed
5. Run and analyze results

### Threat Hunting Query Pattern

```kql
// Pattern: Multi-stage threat hunting
let suspiciousIPs = datatable(IPAddress:string)
[
    "192.0.2.1",
    "198.51.100.1"
];
let timeframe = 7d;
CommonSecurityLog
| where TimeGenerated > ago(timeframe)
| where DestinationIP in (suspiciousIPs)
| join kind=inner (
    SigninLogs
    | where TimeGenerated > ago(timeframe)
    | where ResultType == "0"
) on $left.SourceIP == $right.IPAddress
| project TimeGenerated, SourceIP, DestinationIP, UserPrincipalName, DeviceDetail
| summarize count() by SourceIP, UserPrincipalName
```

## Working with Sentinel Workbooks

### Importing a Workbook

1. Download the workbook JSON file from the repository
2. In **Microsoft Sentinel** > **Workbooks** > **Add workbook**
3. Click **Edit** > **Advanced Editor** (</> icon)
4. Replace content with the downloaded JSON
5. Click **Apply** > **Save**

### Workbook JSON Structure

```json
{
  "version": "Notebook/1.0",
  "items": [
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "SecurityEvent\n| summarize count() by Computer\n| top 10 by count_",
        "size": 0,
        "title": "Top 10 Event Sources"
      }
    }
  ]
}
```

## Using Security Analytics Notebooks

### Setting Up Jupyter Environment

```python
# Install required packages
# pip install msticpy azure-monitor-query azure-identity pandas matplotlib

import msticpy as mp
from msticpy.data import QueryProvider
from azure.identity import AzureCliCredential
import pandas as pd
import os

# Initialize MSTICPy
mp.init_notebook()
```

### Connecting to Microsoft Sentinel

```python
# Authenticate using Azure CLI or environment variables
qry_prov = QueryProvider("MSSentinel")

# Configure workspace
workspace_id = os.getenv("SENTINEL_WORKSPACE_ID")
tenant_id = os.getenv("AZURE_TENANT_ID")

qry_prov.connect(
    connection_str=f"loganalytics://code().tenant('{tenant_id}').workspace('{workspace_id}')"
)
```

### Running Threat Hunting Queries

```python
# Example: Hunt for lateral movement
query = """
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4624
| where LogonType == 3
| summarize LogonCount = count() by Account, Computer, IpAddress
| where LogonCount > 10
| order by LogonCount desc
"""

results = qry_prov.exec_query(query)
print(f"Found {len(results)} potential lateral movement events")
results.head(10)
```

### Data Analysis Pattern

```python
# Analyze authentication anomalies
import matplotlib.pyplot as plt

def analyze_auth_patterns(dataframe):
    """Analyze authentication patterns for anomalies"""
    
    # Group by hour
    dataframe['Hour'] = pd.to_datetime(dataframe['TimeGenerated']).dt.hour
    hourly_counts = dataframe.groupby('Hour').size()
    
    # Plot distribution
    plt.figure(figsize=(12, 6))
    hourly_counts.plot(kind='bar')
    plt.title('Authentication Events by Hour')
    plt.xlabel('Hour of Day')
    plt.ylabel('Event Count')
    plt.show()
    
    # Identify anomalies (simple threshold)
    mean_count = hourly_counts.mean()
    std_count = hourly_counts.std()
    threshold = mean_count + (2 * std_count)
    
    anomalies = hourly_counts[hourly_counts > threshold]
    return anomalies

# Use the function
auth_data = qry_prov.exec_query("SigninLogs | where TimeGenerated > ago(7d)")
anomalies = analyze_auth_patterns(auth_data)
print(f"Anomalous hours: {anomalies.index.tolist()}")
```

## Common Detection Patterns

### Credential Access Detection

```kql
// Detect LSASS memory access
SecurityEvent
| where TimeGenerated > ago(1h)
| where EventID == 4656
| where ObjectName has "lsass.exe"
| where AccessMask has "0x1010"
| project TimeGenerated, Computer, SubjectUserName, ProcessName, ObjectName
```

### Persistence Mechanism Detection

```kql
// Monitor registry run keys
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID in (4657, 4663)
| where ObjectName has_any (
    "\\Software\\Microsoft\\Windows\\CurrentVersion\\Run",
    "\\Software\\Microsoft\\Windows\\CurrentVersion\\RunOnce"
)
| project TimeGenerated, Computer, Account, ObjectName, ProcessName
```

### Command and Control Detection

```kql
// Detect beaconing behavior
CommonSecurityLog
| where TimeGenerated > ago(24h)
| summarize ConnectionCount = count(), 
            AvgBytes = avg(SentBytes + ReceivedBytes),
            TimeVariance = stdev(bin(TimeGenerated, 1m))
            by SourceIP, DestinationIP
| where ConnectionCount > 50
| where TimeVariance < 5  // Consistent timing
| where AvgBytes < 1000   // Small payloads
```

## Configuration

### Environment Variables

Set up authentication for notebooks:

```bash
export AZURE_TENANT_ID="your-tenant-id"
export SENTINEL_WORKSPACE_ID="your-workspace-id"
export AZURE_CLIENT_ID="your-service-principal-id"
export AZURE_CLIENT_SECRET="your-service-principal-secret"
```

### MSTICPy Configuration

Create `msticpyconfig.yaml` in your working directory:

```yaml
AzureSentinel:
  Workspaces:
    Default:
      WorkspaceId: ${SENTINEL_WORKSPACE_ID}
      TenantId: ${AZURE_TENANT_ID}

TIProviders:
  VirusTotal:
    Args:
      AuthKey: ${VT_API_KEY}
    Primary: true
    Provider: "VirusTotal"
```

## Troubleshooting

### KQL Query Timeout

If queries timeout, optimize with:

```kql
// Use time filters early
| where TimeGenerated > ago(1h)
// Limit columns
| project TimeGenerated, Computer, Account
// Use summarize instead of distinct when possible
| summarize count() by Computer
```

### Notebook Authentication Issues

```python
# Try interactive authentication
from azure.identity import InteractiveBrowserCredential

credential = InteractiveBrowserCredential()
qry_prov.connect(credential=credential)
```

### Missing Data in Queries

Verify data connectors are enabled and tables exist:

```kql
// List available tables
search *
| distinct $table
| order by $table asc
```

### Performance Optimization

```kql
// Use materialize() for reusable subqueries
let suspiciousEvents = materialize(
    SecurityEvent
    | where TimeGenerated > ago(24h)
    | where EventID in (4624, 4625, 4688)
);
suspiciousEvents
| where EventID == 4624
| summarize count() by Computer
```

## Best Practices

1. **Always use time filters** early in queries to reduce data scanned
2. **Test queries on small timeframes** before expanding scope
3. **Document custom queries** with comments explaining detection logic
4. **Version control workbooks** by exporting JSON regularly
5. **Use environment variables** for sensitive configuration
6. **Leverage MSTICPy** for complex investigation workflows
7. **Schedule notebooks** for automated threat hunting runs

## Additional Resources

- [David Alonso LinkedIn](https://www.linkedin.com/in/david-alonso-dominguez/)
- [Microsoft Sentinel Documentation](https://docs.microsoft.com/azure/sentinel/)
- [KQL Reference](https://docs.microsoft.com/azure/data-explorer/kusto/query/)
- [MSTICPy Documentation](https://msticpy.readthedocs.io/)
