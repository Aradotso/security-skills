---
name: bitdefender-total-security-workflow
description: Workflow automation and documentation for Bitdefender Total Security on Windows — scan scheduling, quarantine management, and maintenance procedures
triggers:
  - automate bitdefender scans
  - schedule bitdefender total security
  - manage bitdefender quarantine
  - bitdefender workflow setup
  - bitdefender exclusion management
  - automate antivirus maintenance
  - bitdefender powershell commands
  - setup bitdefender automation
---

# Bitdefender Total Security Workflow

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill enables AI coding agents to help developers automate and document **Bitdefender Total Security** workflows on Windows using PowerShell, scheduled tasks, and workflow automation patterns.

## What This Project Does

**Bitdefender-Total-Security-2026** provides workflow automation templates for:

- **Scan scheduling** — Automated full, quick, and custom scans
- **Quarantine review** — Periodic checks and restoration workflows
- **Exclusion management** — Documented folder/file exclusions with audit trail
- **Subscription tracking** — License renewal and version logging
- **Security maintenance** — Post-update validation checklists

This is a workflow documentation and automation project, not a Bitdefender API wrapper.

## Installation

### Project Setup

Clone the workflow repository:

```powershell
git clone https://github.com/Forwardmetier57/Bitdefender-Total-Security-2026.git
cd Bitdefender-Total-Security-2026
```

### PowerShell Execution Policy

Enable script execution (if needed):

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Bitdefender CLI Tools

Bitdefender Total Security includes command-line utilities in:

```
C:\Program Files\Bitdefender\Bitdefender Security\
```

Key executable: `bdagent.exe`

## Key Commands and Automation

### Manual Scan Triggers

**Quick Scan** (PowerShell):

```powershell
Start-Process "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe" -ArgumentList "/scan quick"
```

**Full Scan**:

```powershell
Start-Process "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe" -ArgumentList "/scan full"
```

**Custom Scan** (specific path):

```powershell
$targetPath = "C:\Users\$env:USERNAME\Downloads"
Start-Process "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe" -ArgumentList "/scan `"$targetPath`""
```

### Scheduled Scan Automation

Create a scheduled task for weekly full scan:

```powershell
# Define scan script
$scanScript = @'
$logFile = "C:\BitdefenderWorkflow\Logs\scan-$(Get-Date -Format 'yyyy-MM-dd').log"
$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
"[$timestamp] Starting full scan" | Out-File $logFile -Append

Start-Process "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe" -ArgumentList "/scan full" -Wait

"[$timestamp] Scan completed" | Out-File $logFile -Append
'@

# Save script
New-Item -Path "C:\BitdefenderWorkflow\Scripts" -ItemType Directory -Force
$scanScript | Out-File "C:\BitdefenderWorkflow\Scripts\weekly-full-scan.ps1" -Encoding UTF8

# Create scheduled task
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-File C:\BitdefenderWorkflow\Scripts\weekly-full-scan.ps1"
$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 2am
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries

Register-ScheduledTask -TaskName "BitdefenderWeeklyScan" -Action $action -Trigger $trigger -Principal $principal -Settings $settings
```

### Quarantine Management

**List quarantined items** (via log parsing):

```powershell
# Bitdefender quarantine location
$quarantinePath = "C:\ProgramData\Bitdefender\Desktop\Quarantine"

# Check quarantine folder
Get-ChildItem $quarantinePath -Recurse | Select-Object Name, LastWriteTime, Length | Format-Table
```

**Quarantine review script**:

```powershell
# Create quarantine review log
$reviewLog = "C:\BitdefenderWorkflow\Logs\quarantine-review-$(Get-Date -Format 'yyyy-MM-dd').csv"

$quarantineItems = Get-ChildItem "C:\ProgramData\Bitdefender\Desktop\Quarantine" -Recurse -ErrorAction SilentlyContinue

$quarantineItems | Select-Object `
    @{Name="FileName";Expression={$_.Name}},
    @{Name="QuarantinedDate";Expression={$_.LastWriteTime}},
    @{Name="SizeKB";Expression={[math]::Round($_.Length/1KB,2)}},
    @{Name="Action";Expression={"Pending Review"}},
    @{Name="Notes";Expression={""}} | 
Export-Csv $reviewLog -NoTypeInformation

Write-Host "Quarantine review log created: $reviewLog"
```

### Exclusion Management

**Document exclusion with audit trail**:

```powershell
function Add-BitdefenderExclusion {
    param(
        [Parameter(Mandatory)]
        [string]$Path,
        
        [Parameter(Mandatory)]
        [string]$Reason,
        
        [string]$RequestedBy = $env:USERNAME
    )
    
    # Log exclusion
    $exclusionLog = "C:\BitdefenderWorkflow\Logs\exclusions.csv"
    
    $entry = [PSCustomObject]@{
        Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        Path = $Path
        Reason = $Reason
        RequestedBy = $RequestedBy
        Status = "Added"
    }
    
    # Append to log
    $entry | Export-Csv $exclusionLog -Append -NoTypeInformation
    
    Write-Host "Exclusion documented: $Path"
    Write-Host "Manual step: Add exclusion in Bitdefender UI (Protection > Antivirus > Manage Exclusions)"
}

# Usage
Add-BitdefenderExclusion -Path "C:\Dev\Projects\MyApp" -Reason "Development environment - false positive on build artifacts"
```

### Subscription and Version Tracking

**Log current version and subscription status**:

```powershell
function Get-BitdefenderStatus {
    $statusLog = "C:\BitdefenderWorkflow\Logs\status-$(Get-Date -Format 'yyyy-MM-dd').json"
    
    # Get Bitdefender version from registry
    $bdVersion = (Get-ItemProperty -Path "HKLM:\SOFTWARE\Bitdefender\Bitdefender Desktop" -Name "Version" -ErrorAction SilentlyContinue).Version
    
    # Check service status
    $bdService = Get-Service -Name "VSSERV" -ErrorAction SilentlyContinue
    
    $status = [PSCustomObject]@{
        Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        Version = $bdVersion
        ServiceStatus = $bdService.Status
        ServiceStartType = $bdService.StartType
        LastCheck = (Get-Date)
    }
    
    $status | ConvertTo-Json | Out-File $statusLog
    
    return $status
}

# Usage
Get-BitdefenderStatus | Format-List
```

## Configuration

### Workflow Directory Structure

Recommended folder layout:

```
C:\BitdefenderWorkflow\
├── Scripts\
│   ├── weekly-full-scan.ps1
│   ├── quarantine-review.ps1
│   └── status-check.ps1
├── Logs\
│   ├── scan-YYYY-MM-DD.log
│   ├── quarantine-review-YYYY-MM-DD.csv
│   ├── exclusions.csv
│   └── status-YYYY-MM-DD.json
└── Config\
    └── workflow-settings.json
```

**Initialize workflow structure**:

```powershell
$workflowRoot = "C:\BitdefenderWorkflow"
@("Scripts", "Logs", "Config") | ForEach-Object {
    New-Item -Path "$workflowRoot\$_" -ItemType Directory -Force
}
```

### Workflow Settings File

**Config/workflow-settings.json**:

```json
{
  "scanSchedule": {
    "fullScan": "Weekly - Sunday 2:00 AM",
    "quickScan": "Daily - 12:00 PM"
  },
  "quarantineReview": {
    "frequency": "Weekly",
    "retentionDays": 30
  },
  "exclusions": {
    "requireApproval": true,
    "documentationRequired": true
  },
  "logging": {
    "retentionDays": 90,
    "archivePath": "C:\\BitdefenderWorkflow\\Archive"
  }
}
```

## Real-World Patterns

### Pattern 1: Post-Update Validation

After Bitdefender updates, verify protection is active:

```powershell
function Test-BitdefenderProtection {
    Write-Host "=== Bitdefender Post-Update Check ===" -ForegroundColor Cyan
    
    # Check service
    $service = Get-Service -Name "VSSERV"
    Write-Host "Service Status: $($service.Status)" -ForegroundColor $(if($service.Status -eq "Running"){"Green"}else{"Red"})
    
    # Check real-time protection (example using WMI)
    $realTimeProtection = Get-WmiObject -Namespace "root\cimv2\security\microsoftdefender" -Class MSFT_MpPreference -ErrorAction SilentlyContinue
    
    # Log results
    $checkLog = "C:\BitdefenderWorkflow\Logs\protection-check-$(Get-Date -Format 'yyyy-MM-dd').log"
    @"
Protection Check - $(Get-Date)
Service: $($service.Status)
Last Update: $(Get-Date)
"@ | Out-File $checkLog
    
    Write-Host "Check logged to: $checkLog"
}

# Run post-update
Test-BitdefenderProtection
```

### Pattern 2: False Positive Workflow

Handle false positive detections with documentation:

```powershell
function Resolve-FalsePositive {
    param(
        [Parameter(Mandatory)]
        [string]$FilePath,
        
        [Parameter(Mandatory)]
        [string]$ThreatName,
        
        [string]$Justification
    )
    
    $falsePositiveLog = "C:\BitdefenderWorkflow\Logs\false-positives.csv"
    
    $entry = [PSCustomObject]@{
        Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        FilePath = $FilePath
        ThreatName = $ThreatName
        Justification = $Justification
        Reviewer = $env:USERNAME
        Action = "Exclusion Requested"
    }
    
    $entry | Export-Csv $falsePositiveLog -Append -NoTypeInformation
    
    Write-Host @"
False Positive Documented:
  File: $FilePath
  Threat: $ThreatName
  Next Steps:
    1. Review in Bitdefender UI (Protection > Quarantine)
    2. Restore file if safe
    3. Add exclusion via Antivirus > Manage Exclusions
    4. Update exclusions.csv log
"@
}

# Usage
Resolve-FalsePositive -FilePath "C:\Dev\MyApp\build\app.exe" -ThreatName "Gen:Variant.Zusy.123456" -Justification "Custom build script - verified safe"
```

### Pattern 3: Maintenance Dashboard

Generate a weekly maintenance report:

```powershell
function New-MaintenanceReport {
    $reportPath = "C:\BitdefenderWorkflow\Reports\maintenance-$(Get-Date -Format 'yyyy-MM-dd').html"
    
    # Gather data
    $bdStatus = Get-BitdefenderStatus
    $recentScans = Get-ChildItem "C:\BitdefenderWorkflow\Logs\scan-*.log" | Select-Object -First 7
    $quarantineCount = (Get-ChildItem "C:\ProgramData\Bitdefender\Desktop\Quarantine" -Recurse -ErrorAction SilentlyContinue).Count
    $exclusions = Import-Csv "C:\BitdefenderWorkflow\Logs\exclusions.csv" -ErrorAction SilentlyContinue
    
    # Build HTML
    $html = @"
<!DOCTYPE html>
<html>
<head><title>Bitdefender Maintenance Report</title></head>
<body>
<h1>Bitdefender Maintenance Report</h1>
<p>Generated: $(Get-Date)</p>
<h2>Protection Status</h2>
<ul>
<li>Version: $($bdStatus.Version)</li>
<li>Service: $($bdStatus.ServiceStatus)</li>
</ul>
<h2>Recent Scans</h2>
<p>Scans in last 7 days: $($recentScans.Count)</p>
<h2>Quarantine</h2>
<p>Items in quarantine: $quarantineCount</p>
<h2>Exclusions</h2>
<p>Total exclusions: $($exclusions.Count)</p>
</body>
</html>
"@
    
    New-Item -Path "C:\BitdefenderWorkflow\Reports" -ItemType Directory -Force
    $html | Out-File $reportPath
    
    Write-Host "Report generated: $reportPath"
}

# Generate report
New-MaintenanceReport
```

## Troubleshooting

### Issue: Script cannot find Bitdefender executable

**Check:**

```powershell
# Verify installation path
Test-Path "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe"

# If different, update path in scripts
$bdPath = (Get-ItemProperty "HKLM:\SOFTWARE\Bitdefender\Bitdefender Desktop" -Name "InstallDir").InstallDir
Write-Host "Bitdefender installed at: $bdPath"
```

### Issue: Scheduled task not running

**Debug:**

```powershell
# Check task status
Get-ScheduledTask -TaskName "BitdefenderWeeklyScan" | Get-ScheduledTaskInfo

# View task history in Event Viewer
Get-WinEvent -LogName "Microsoft-Windows-TaskScheduler/Operational" -MaxEvents 20 | 
    Where-Object {$_.Message -like "*BitdefenderWeeklyScan*"} | 
    Format-Table TimeCreated, Message -Wrap
```

### Issue: Access denied to quarantine folder

**Solution:**

```powershell
# Run PowerShell as Administrator
# Or check permissions
Get-Acl "C:\ProgramData\Bitdefender\Desktop\Quarantine" | Format-List
```

### Issue: Logs growing too large

**Cleanup script**:

```powershell
# Archive logs older than 90 days
$archivePath = "C:\BitdefenderWorkflow\Archive"
$retentionDays = 90

Get-ChildItem "C:\BitdefenderWorkflow\Logs\*.log" | 
    Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-$retentionDays)} |
    ForEach-Object {
        Move-Item $_.FullName -Destination $archivePath -Force
    }
```

## Common Use Cases

### Automated Scan Before System Maintenance

```powershell
# Pre-maintenance scan
function Start-PreMaintenanceScan {
    $logFile = "C:\BitdefenderWorkflow\Logs\pre-maintenance-$(Get-Date -Format 'yyyy-MM-dd').log"
    
    "Starting pre-maintenance scan at $(Get-Date)" | Out-File $logFile
    
    Start-Process "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe" -ArgumentList "/scan quick" -Wait
    
    "Scan completed at $(Get-Date)" | Out-File $logFile -Append
    
    # Check if quarantine has new items
    $quarantineCheck = Get-ChildItem "C:\ProgramData\Bitdefender\Desktop\Quarantine" -Recurse -ErrorAction SilentlyContinue |
        Where-Object {$_.LastWriteTime -gt (Get-Date).AddHours(-1)}
    
    if ($quarantineCheck) {
        "WARNING: New items in quarantine!" | Out-File $logFile -Append
        $quarantineCheck | Select-Object Name, LastWriteTime | Out-File $logFile -Append
    }
}
```

### Integration with CI/CD Pipeline

```powershell
# Scan build output before deployment
function Test-BuildArtifacts {
    param([string]$BuildPath)
    
    Write-Host "Scanning build artifacts: $BuildPath"
    
    Start-Process "C:\Program Files\Bitdefender\Bitdefender Security\bdagent.exe" -ArgumentList "/scan `"$BuildPath`"" -Wait
    
    # Check for threats
    $recentQuarantine = Get-ChildItem "C:\ProgramData\Bitdefender\Desktop\Quarantine" -Recurse -ErrorAction SilentlyContinue |
        Where-Object {$_.LastWriteTime -gt (Get-Date).AddMinutes(-5)}
    
    if ($recentQuarantine) {
        throw "Build artifacts flagged by antivirus! Deployment blocked."
    }
    
    Write-Host "Build artifacts clean - deployment can proceed"
}
```

This skill equips AI agents to help developers automate Bitdefender Total Security workflows using PowerShell, scheduled tasks, and documented maintenance procedures.
