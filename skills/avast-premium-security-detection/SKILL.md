---
name: avast-premium-security-detection
description: Detect and analyze potentially fraudulent software distribution repositories masquerading as legitimate security software
triggers:
  - analyze this repository for security software fraud
  - check if this is a legitimate antivirus distribution
  - detect fake software activation repos
  - identify malware distribution patterns
  - verify antivirus software authenticity
  - scan for piracy and malware indicators
  - investigate suspicious security software claims
  - validate software licensing claims
---

# Avast Premium Security Detection

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This skill helps identify repositories that fraudulently claim to distribute commercial security software with "cracks," "keygens," or "pre-activated" licenses. Such repositories are common vectors for malware distribution, credential theft, and social engineering attacks.

## Red Flags to Identify

### Repository Indicators

**High-Risk Patterns:**
- Claims of "Full Version," "Cracked," "Pre-Activated," "Keygen," "Loader," or "Serial"
- Commercial software offered for free with activation tools
- Repository language doesn't match claimed software (e.g., C++ for a binary-only product)
- Excessive use of star emojis and promotional language
- No actual source code or only obfuscated binaries
- Artificially inflated stars (6 stars/day for new repo is suspicious)
- No legitimate license (NOASSERTION)
- Topics mixing legitimate terms with piracy terms

**Specific to This Case:**
```
Description keywords: "Keygen", "Pre-Activated", "Loader", "Serial"
Language mismatch: Listed as C++ but Avast is proprietary
No README or minimal documentation
Topics include: "retdec" (reverse engineering tool)
```

## Analysis Framework

### 1. Repository Metadata Check

```python
import os
import re
from datetime import datetime

def analyze_repo_metadata(repo_data):
    """Analyze repository metadata for fraud indicators"""
    
    red_flags = []
    risk_score = 0
    
    # Check description for piracy keywords
    piracy_keywords = [
        'keygen', 'crack', 'pre-activated', 'loader', 
        'serial', 'license key', 'activation', 'full version'
    ]
    
    description_lower = repo_data.get('description', '').lower()
    found_keywords = [kw for kw in piracy_keywords if kw in description_lower]
    
    if found_keywords:
        red_flags.append(f"Piracy keywords found: {', '.join(found_keywords)}")
        risk_score += len(found_keywords) * 20
    
    # Check for commercial software names
    commercial_products = ['avast', 'norton', 'mcafee', 'kaspersky', 'bitdefender']
    if any(prod in description_lower for prod in commercial_products):
        red_flags.append("Claims to distribute commercial security software")
        risk_score += 30
    
    # Check stars velocity (suspicious growth)
    stars = repo_data.get('stars', 0)
    created = datetime.fromisoformat(repo_data.get('created_at', '').replace('Z', '+00:00'))
    days_old = (datetime.now(created.tzinfo) - created).days
    stars_per_day = stars / max(days_old, 1)
    
    if stars_per_day > 3 and days_old < 30:
        red_flags.append(f"Suspicious star growth: {stars_per_day:.1f} stars/day")
        risk_score += 25
    
    # Check for missing README
    if not repo_data.get('has_readme', True):
        red_flags.append("No README or minimal documentation")
        risk_score += 15
    
    return {
        'risk_score': min(risk_score, 100),
        'risk_level': 'CRITICAL' if risk_score >= 70 else 'HIGH' if risk_score >= 40 else 'MEDIUM',
        'red_flags': red_flags
    }

# Example usage
repo_info = {
    'description': '⭐️ Avast Premium Security 2026 | Full Version Installer v26 | Setup Keygen Activation',
    'language': 'C++',
    'stars': 68,
    'created_at': '2026-05-06T07:04:44Z',
    'has_readme': False
}

analysis = analyze_repo_metadata(repo_info)
print(f"Risk Level: {analysis['risk_level']}")
print(f"Risk Score: {analysis['risk_score']}/100")
for flag in analysis['red_flags']:
    print(f"  ⚠️  {flag}")
```

### 2. Content Pattern Detection

```python
import re

def detect_malicious_patterns(file_content, filename):
    """Detect patterns commonly found in malware distribution"""
    
    patterns = {
        'download_redirects': r'(bit\.ly|tinyurl|short\.io|mediafire|mega\.nz)',
        'credential_harvest': r'(password|username|email).*?input',
        'obfuscation': r'(eval\(|exec\(|fromCharCode|atob\()',
        'suspicious_domains': r'https?://[a-z0-9-]+\.(tk|ml|ga|cf|gq)\b',
        'binary_download': r'\.(exe|msi|dll|scr|bat|vbs|ps1)\b'
    }
    
    findings = {}
    
    for pattern_name, pattern in patterns.items():
        matches = re.findall(pattern, file_content, re.IGNORECASE)
        if matches:
            findings[pattern_name] = {
                'count': len(matches),
                'samples': matches[:3]  # First 3 matches
            }
    
    return findings

# Check for typical malware distribution script
example_suspicious_script = """
window.location.href = "https://example.tk/download/avast-crack.exe";
// Real antivirus downloads don't redirect to free domains
"""

results = detect_malicious_patterns(example_suspicious_script, "download.html")
if results:
    print("⚠️  MALICIOUS PATTERNS DETECTED:")
    for pattern, data in results.items():
        print(f"  - {pattern}: {data['count']} occurrences")
```

### 3. Legitimate Software Verification

```python
def verify_official_source(product_name, repo_url):
    """Check if repository matches official distribution channels"""
    
    official_sources = {
        'avast': {
            'domains': ['avast.com', 'github.com/avast'],
            'official_repo': 'avast/retdec'  # Their actual open source project
        },
        'norton': {
            'domains': ['norton.com', 'nortonlifelock.com']
        }
    }
    
    product_lower = product_name.lower()
    
    for product, info in official_sources.items():
        if product in product_lower:
            # Check if repo URL contains official domain
            is_official = any(domain in repo_url.lower() for domain in info['domains'])
            
            return {
                'product': product,
                'is_official': is_official,
                'official_channels': info['domains'],
                'warning': None if is_official else f"NOT from official {product} sources"
            }
    
    return {'product': product_name, 'is_official': False, 'warning': 'Unknown product source'}

# Example check
result = verify_official_source('Avast Premium Security', 'github.com/viceofficialtower74/Avast-Premium')
print(f"Official Source: {result['is_official']}")
if result['warning']:
    print(f"⛔ WARNING: {result['warning']}")
    print(f"   Official channels: {', '.join(result['official_channels'])}")
```

## User Warning Template

When detecting fraudulent repositories, provide clear warnings:

```markdown
## ⛔ SECURITY WARNING ⛔

This repository exhibits multiple indicators of fraudulent software distribution:

### Critical Issues Identified:
1. **Piracy Claims**: Advertises "keygen," "pre-activated," and "cracked" software
2. **Commercial Software**: Claims to distribute Avast Premium (commercial product)
3. **Language Mismatch**: Listed as C++ but Avast is proprietary binary software
4. **No Source Code**: No legitimate code present, likely contains malware
5. **Suspicious Growth**: Artificially inflated stars/engagement

### Risks:
- **Malware Distribution**: High probability of containing trojans, ransomware, or spyware
- **Credential Theft**: May harvest passwords, emails, or personal information
- **System Compromise**: Could install backdoors or rootkits
- **Legal Issues**: Software piracy is illegal

### Safe Alternatives:
- Official Avast: https://www.avast.com
- Free alternatives: Windows Defender (built-in), ClamAV (open source)

**DO NOT DOWNLOAD OR RUN ANY FILES FROM THIS REPOSITORY**
```

## Detection Automation

```python
import os
import json

def full_repository_scan(repo_path):
    """Comprehensive scan of a repository for fraud indicators"""
    
    report = {
        'timestamp': datetime.now().isoformat(),
        'repository': repo_path,
        'findings': [],
        'overall_risk': 'UNKNOWN'
    }
    
    # 1. Check for executable files (major red flag)
    executable_extensions = ['.exe', '.msi', '.dll', '.bat', '.scr', '.vbs']
    executables_found = []
    
    for root, dirs, files in os.walk(repo_path):
        for file in files:
            ext = os.path.splitext(file)[1].lower()
            if ext in executable_extensions:
                executables_found.append(os.path.join(root, file))
    
    if executables_found:
        report['findings'].append({
            'type': 'CRITICAL',
            'issue': 'Executable files found',
            'details': f"Found {len(executables_found)} executable files",
            'files': executables_found[:5]  # List first 5
        })
        report['overall_risk'] = 'CRITICAL'
    
    # 2. Check for absence of legitimate source code
    source_extensions = ['.cpp', '.h', '.c', '.py', '.js', '.go']
    source_files = []
    
    for root, dirs, files in os.walk(repo_path):
        for file in files:
            ext = os.path.splitext(file)[1].lower()
            if ext in source_extensions:
                source_files.append(file)
    
    if len(source_files) == 0 and len(executables_found) > 0:
        report['findings'].append({
            'type': 'CRITICAL',
            'issue': 'No source code, only binaries',
            'details': 'Repository contains executables but no source code'
        })
    
    return report

# Usage
# scan_result = full_repository_scan('/path/to/suspicious/repo')
# print(json.dumps(scan_result, indent=2))
```

## Best Practices

### For Users
1. **Never download cracked/pirated software** - extremely high malware risk
2. **Verify official sources** - only download from vendor websites
3. **Check repository authenticity** - look for verified accounts, legitimate code
4. **Use built-in protection** - Windows Defender is free and effective
5. **Report fraudulent repos** - use GitHub's abuse report feature

### For Developers
1. **Implement security scanning** in CI/CD pipelines
2. **Monitor for impersonation** of your software products
3. **Educate users** about official distribution channels
4. **Report violations** to hosting platforms promptly

## Reporting Fraudulent Repositories

```bash
# Report to GitHub
# Visit: https://github.com/contact/report-abuse
# Select: "This repository contains illegal content or violates terms"
# Provide: Repository URL and evidence of malware/piracy

# Report to security communities
# VirusTotal: Upload suspicious files
# abuse.ch: Report malware distribution
```

## Legitimate Avast Open Source

The **real** Avast open source project is RetDec (reverse engineering tool):
- Official repo: `https://github.com/avast/retdec`
- Purpose: Decompiler for binary analysis
- Not related to Avast antivirus product distribution

## Summary

This skill enables detection of fraudulent software distribution schemes that:
- Claim to offer commercial software for free
- Use piracy-related terminology
- Likely distribute malware instead of legitimate software
- Pose significant security and legal risks

Always verify software sources and never trust repositories offering "cracked" commercial products.
