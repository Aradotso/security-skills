---
name: avast-premium-security-awareness
description: Recognize and avoid potentially malicious software distribution repositories disguised as legitimate security tools
triggers:
  - how do I identify fake security software repositories
  - what are signs of malware distribution on github
  - is this avast repository legitimate
  - how to verify official antivirus downloads
  - detecting pirated software scams
  - identifying keygen and crack distribution sites
  - how to spot fake premium security installers
  - what makes a github project look suspicious
---

# Avast Premium Security Awareness

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ Critical Security Warning

This repository **viceofficialtower74/Avast-Premium-Security-Windows-Latest** exhibits multiple red flags indicating it is **NOT** a legitimate security project and is likely malicious:

### Red Flags Identified

1. **Unauthorized Distribution**: Avast does not distribute official software through random GitHub repositories
2. **Piracy Keywords**: Contains "keygen", "crack", "loader", "pre-activated" - common malware indicators
3. **Suspicious Metrics**: 68 stars with 6 stars/day growth pattern suggests artificial promotion
4. **No Code**: Claims to be C++ but contains no actual source code or README
5. **Trademark Abuse**: Unauthorized use of Avast branding
6. **License Violation**: "NOASSERTION" license for commercial software

## What This Project Actually Represents

This is a **malware distribution vector** disguising itself as legitimate security software. Common payloads include:

- Trojans and backdoors
- Cryptocurrency miners
- Information stealers (passwords, credentials)
- Ransomware
- Botnet clients
- Browser hijackers

## How to Identify Similar Threats

### GitHub Repository Red Flags

```bash
# Check repository metadata
# Suspicious indicators:
# - Recent creation date with high star velocity
# - No actual source code despite language claims
# - Topics mixing legitimate tools with "keygen", "crack", "loader"
# - Generic username patterns
# - No commit history or minimal commits
```

### Filename Patterns to Avoid

```
❌ Setup.exe
❌ Crack.exe
❌ Keygen.exe
❌ Loader.exe
❌ Patch.exe
❌ Activator.exe
❌ *-Pre-Activated.exe
❌ Serial-Generator.exe
```

### Common Malware Distribution Topics

```yaml
# These topic combinations are red flags:
suspicious_topics:
  - "keygen" + commercial_software_name
  - "crack" + "premium" + "license"
  - "loader" + "activation"
  - "pre-activated" + "full-version"
  - legitimate_tool + "serial"
```

## Safe Security Software Practices

### Verify Official Sources

```python
# Example: Checking official sources
official_sources = {
    "Avast": "https://www.avast.com",
    "Norton": "https://www.norton.com",
    "Bitdefender": "https://www.bitdefender.com",
    "Kaspersky": "https://www.kaspersky.com"
}

def is_official_source(url):
    """Verify if URL matches official vendor domain"""
    from urllib.parse import urlparse
    domain = urlparse(url).netloc
    
    # GitHub is NOT an official source for commercial antivirus
    if 'github.com' in domain:
        return False
    
    # Check against known official domains
    for vendor, official_url in official_sources.items():
        official_domain = urlparse(official_url).netloc
        if domain == official_domain:
            return True
    
    return False
```

### Download Verification Steps

```bash
#!/bin/bash
# Always verify downloaded security software

# 1. Download only from official vendor website
OFFICIAL_URL="https://www.avast.com/download"

# 2. Verify digital signature (Windows)
# signtool verify /pa /v installer.exe

# 3. Check file hash against official checksums
# sha256sum installer.exe
# Compare with vendor-provided hash

# 4. Scan with existing antivirus before running
# Never disable antivirus to install "cracked" security software
```

### Code Signature Verification (PowerShell)

```powershell
# Verify digital signature of downloaded file
function Verify-Signature {
    param([string]$FilePath)
    
    $signature = Get-AuthenticodeSignature -FilePath $FilePath
    
    if ($signature.Status -eq 'Valid') {
        Write-Host "✓ Valid signature from: $($signature.SignerCertificate.Subject)"
        
        # Verify it's actually from the vendor
        $expectedPublisher = "Avast Software s.r.o."
        if ($signature.SignerCertificate.Subject -match $expectedPublisher) {
            Write-Host "✓ Verified official publisher"
            return $true
        } else {
            Write-Warning "⚠ Signature valid but unexpected publisher"
            return $false
        }
    } else {
        Write-Error "✗ Invalid or missing signature: $($signature.Status)"
        return $false
    }
}

# Usage
Verify-Signature -FilePath ".\downloaded_installer.exe"
```

## Detection and Response

### If You've Already Downloaded

```python
# Immediate response steps
import os
import subprocess
import hashlib

def emergency_response(suspicious_file):
    """Steps to take if malware is suspected"""
    
    # 1. DO NOT EXECUTE the file
    print("[!] Do not run the suspicious file")
    
    # 2. Calculate hash for reporting
    with open(suspicious_file, 'rb') as f:
        file_hash = hashlib.sha256(f.read()).hexdigest()
    print(f"[i] File SHA256: {file_hash}")
    
    # 3. Upload to VirusTotal (or similar) for analysis
    print("[i] Submit hash to VirusTotal.com for analysis")
    
    # 4. Delete the file securely
    try:
        os.remove(suspicious_file)
        print("[✓] File deleted")
    except Exception as e:
        print(f"[!] Could not delete: {e}")
    
    # 5. Run full system scan
    print("[i] Running full system antivirus scan...")
    
    # 6. Check for persistence mechanisms
    print("[i] Check startup items and scheduled tasks")
    
    return file_hash
```

### System Cleanup Commands

```bash
#!/bin/bash
# Linux/macOS cleanup

# Check for suspicious processes
ps aux | grep -i "avast\|keygen\|crack\|loader"

# Check recently modified files
find /tmp -type f -mtime -1

# Check for persistence
crontab -l
cat ~/.bash_profile
cat ~/.bashrc

# Network connections
netstat -an | grep ESTABLISHED
```

```powershell
# Windows cleanup
# Check running processes
Get-Process | Where-Object {$_.ProcessName -match "setup|crack|keygen|loader"}

# Check scheduled tasks
Get-ScheduledTask | Where-Object {$_.Date -gt (Get-Date).AddDays(-1)}

# Check startup items
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location

# Check recent network connections
Get-NetTCPConnection | Where-Object {$_.State -eq "Established"}
```

## Legitimate Alternatives

### Official Free Antivirus Options

```yaml
legitimate_free_options:
  - name: "Windows Defender"
    source: "Built into Windows 10/11"
    url: "ms-settings:windowsdefender"
    
  - name: "Avast Free Antivirus"
    source: "Official website only"
    url: "https://www.avast.com/free-antivirus-download"
    
  - name: "Bitdefender Antivirus Free"
    source: "Official website only"
    url: "https://www.bitdefender.com/solutions/free.html"
```

### Open Source Security Tools (Actually Legitimate)

```bash
# ClamAV - Open source antivirus
# Official repo: https://github.com/Cisco-Talos/clamav
sudo apt install clamav
clamscan -r /home

# YARA - Malware identification
# Official repo: https://github.com/VirusTotal/yara
pip install yara-python
```

## Reporting Malicious Repositories

```bash
# Report to GitHub
# URL: https://github.com/contact/report-abuse
# Select: "This repository contains malware or is being used for phishing"

# Report to security vendors
# - Microsoft: https://www.microsoft.com/en-us/wdsi/filesubmission
# - Google Safe Browsing: https://safebrowsing.google.com/safebrowsing/report_badware/
```

## Key Takeaways

1. **Never download security software from GitHub** unless it's official open-source projects (e.g., ClamAV)
2. **Always use official vendor websites** for commercial antivirus software
3. **"Cracked" security software is an oxymoron** - it's designed to bypass protection
4. **Free legitimate alternatives exist** - never risk malware for "premium" features
5. **Verify digital signatures** before running any security software
6. **High star counts can be faked** - check commit history and code quality

This skill helps identify and avoid malware distribution disguised as legitimate security tools.
