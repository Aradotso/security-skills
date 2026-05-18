```markdown
---
name: security-risk-awareness
description: Recognize potentially malicious software distribution repositories disguised as legitimate security software
triggers:
  - analyze this security software repository
  - is this antivirus download safe
  - check if this software is legitimate
  - verify security software authenticity
  - evaluate this keygen or crack repository
  - identify malware distribution sites
  - assess software piracy risks
  - detect fake security software
---

# Security Risk Awareness: Identifying Malicious Software Distribution

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ Critical Security Warning

This repository exhibits multiple indicators of a malicious or fraudulent software distribution attempt:

### Red Flags Identified

1. **Piracy Keywords**: "Keygen", "Pre-Activated", "Loader", "Serial", "Crack"
2. **Trademark Abuse**: Unauthorized distribution of commercial Avast software
3. **Suspicious Naming**: Generic username with commercial product claims
4. **No Source Code**: C++ project with no README or visible code
5. **Artificial Engagement**: Suspicious star growth pattern (5 stars/day)
6. **Future Date**: Created date in 2026 (timestamp manipulation)
7. **License Violation**: No license assertion for commercial software

## What This Repository Actually Represents

This is NOT a legitimate Avast Premium Security distribution. It represents one of the following:

- **Malware Distribution**: Downloads may contain trojans, ransomware, or spyware
- **Credential Theft**: Setup files may harvest system information or passwords
- **Cryptocurrency Miners**: Hidden background miners consuming system resources
- **Botnet Recruitment**: Remote access trojans (RATs) for DDoS or other attacks
- **Adware/PUPs**: Potentially Unwanted Programs bundling unwanted software

## How to Identify Similar Threats

### Repository Analysis Checklist

```python
def analyze_repository_risk(repo_data):
    """
    Analyze repository for malware distribution indicators
    """
    risk_score = 0
    flags = []
    
    # Check for piracy keywords
    piracy_terms = ['keygen', 'crack', 'serial', 'loader', 
                    'pre-activated', 'activation', 'license key']
    description_lower = repo_data['description'].lower()
    
    for term in piracy_terms:
        if term in description_lower:
            risk_score += 15
            flags.append(f"Piracy keyword detected: {term}")
    
    # Check for commercial software names
    commercial_products = ['avast', 'norton', 'kaspersky', 
                          'mcafee', 'bitdefender', 'windows']
    for product in commercial_products:
        if product in description_lower:
            risk_score += 10
            flags.append(f"Commercial product name: {product}")
    
    # Check for missing content
    if not repo_data.get('readme'):
        risk_score += 20
        flags.append("No README file")
    
    # Check for suspicious star growth
    days_old = (datetime.now() - repo_data['created_at']).days
    if days_old > 0:
        stars_per_day = repo_data['stars'] / days_old
        if stars_per_day > 2:
            risk_score += 15
            flags.append(f"Suspicious star growth: {stars_per_day:.1f}/day")
    
    # Check for mismatched language/content
    if repo_data['language'] and not repo_data.get('source_files'):
        risk_score += 10
        flags.append("Language claimed but no source code")
    
    return {
        'risk_score': risk_score,
        'risk_level': 'CRITICAL' if risk_score > 50 else 
                     'HIGH' if risk_score > 30 else 
                     'MEDIUM' if risk_score > 15 else 'LOW',
        'flags': flags,
        'recommendation': 'AVOID' if risk_score > 30 else 'CAUTION'
    }
```

### Safe Software Acquisition Practices

```bash
# ✅ CORRECT: Download from official sources
# Avast Official Website
curl -O https://www.avast.com/download-premium-security

# ✅ CORRECT: Use official package managers
# Windows (via Chocolatey)
choco install avast-free-antivirus

# macOS (via Homebrew Cask)
brew install --cask avast-security

# ❌ WRONG: Never download from:
# - GitHub repos claiming "pre-activated" software
# - File sharing sites with "keygens"
# - URLs with multiple redirects
# - Sites requiring surveys before download
```

## Verification Techniques

### Check File Hashes

```python
import hashlib
import requests

def verify_download(file_path, official_hash=None):
    """
    Verify downloaded file against official hash
    """
    # Calculate SHA256 hash
    sha256_hash = hashlib.sha256()
    with open(file_path, "rb") as f:
        for byte_block in iter(lambda: f.read(4096), b""):
            sha256_hash.update(byte_block)
    
    calculated_hash = sha256_hash.hexdigest()
    
    print(f"File: {file_path}")
    print(f"SHA256: {calculated_hash}")
    
    if official_hash:
        if calculated_hash == official_hash:
            print("✅ Hash verified - file is authentic")
            return True
        else:
            print("❌ Hash mismatch - file may be compromised")
            return False
    else:
        print("⚠️  No official hash provided for verification")
        return None
```

### VirusTotal API Check

```python
import os
import requests
import time

def check_file_virustotal(file_path):
    """
    Submit file to VirusTotal for analysis
    Requires VIRUSTOTAL_API_KEY environment variable
    """
    api_key = os.environ.get('VIRUSTOTAL_API_KEY')
    if not api_key:
        raise ValueError("VIRUSTOTAL_API_KEY not set")
    
    # Calculate file hash
    sha256_hash = hashlib.sha256()
    with open(file_path, 'rb') as f:
        sha256_hash.update(f.read())
    file_hash = sha256_hash.hexdigest()
    
    # Check existing analysis
    url = f"https://www.virustotal.com/api/v3/files/{file_hash}"
    headers = {"x-apikey": api_key}
    
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        data = response.json()
        stats = data['data']['attributes']['last_analysis_stats']
        
        print(f"\nVirusTotal Analysis for: {file_path}")
        print(f"Malicious detections: {stats['malicious']}")
        print(f"Suspicious detections: {stats['suspicious']}")
        print(f"Undetected: {stats['undetected']}")
        
        if stats['malicious'] > 0:
            print("❌ MALWARE DETECTED - Do not execute")
            return False
        else:
            print("✅ No malware detected")
            return True
    else:
        print("File not in VirusTotal database - manual upload required")
        return None
```

## Legitimate Alternatives

### Official Security Software Sources

```yaml
Antivirus Solutions:
  Avast Official:
    url: https://www.avast.com
    free_version: true
    
  Windows Defender:
    included_with: Windows 10/11
    cost: free
    
  Malwarebytes:
    url: https://www.malwarebytes.com
    free_trial: 14 days

Open Source Security Tools:
  ClamAV:
    repo: https://github.com/Cisco-Talos/clamav
    license: GPL-2.0
    package: clamav
    
  OSSEC:
    repo: https://github.com/ossec/ossec-hids
    license: GPL-2.0
    type: Host-based IDS
```

## Response Actions

### If You've Downloaded Suspicious Software

```bash
#!/bin/bash
# Emergency response script

# 1. Disconnect from network immediately
# (Manual action required)

# 2. Do NOT execute the downloaded file
echo "❌ Do not run any .exe, .msi, or .bat files from untrusted sources"

# 3. Quarantine the file
QUARANTINE_DIR="$HOME/quarantine"
mkdir -p "$QUARANTINE_DIR"
# Move suspicious file (don't run this automatically)
# mv suspicious_file.exe "$QUARANTINE_DIR/"

# 4. Run legitimate security scan
# Windows Defender quick scan
powershell.exe -Command "Start-MpScan -ScanType QuickScan"

# 5. Check for indicators of compromise
netstat -an | grep ESTABLISHED  # Check active connections
ps aux | grep -i suspicious    # Check running processes

# 6. Report the repository
echo "Report to: GitHub Support, VirusTotal, Anti-Phishing Working Group"
```

## Educational Use

This skill helps AI agents recognize and warn users about:

- Software piracy attempts disguised as repositories
- Malware distribution infrastructure
- Social engineering tactics in repository descriptions
- Trademark and license violations
- Credential theft mechanisms

## Conclusion

**Never download or execute files from repositories claiming to offer:**
- Pre-activated commercial software
- Keygens, cracks, or loaders
- "Full version" of paid software for free
- Suspicious star growth patterns
- No visible source code despite claiming a programming language

**Always:**
- Use official vendor websites
- Verify digital signatures
- Check file hashes against official sources
- Use VirusTotal or similar services
- Keep legitimate antivirus software updated

```
