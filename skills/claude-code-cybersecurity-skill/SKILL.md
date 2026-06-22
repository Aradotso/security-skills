---
name: claude-code-cybersecurity-skill
description: Use and extend 15 production-quality Claude Code Skills for cybersecurity operations including offensive security, defensive operations, reverse engineering, threat hunting, CSOC automation, and more
triggers:
  - "help me use the Claude cybersecurity skills"
  - "install the security skills for Claude Code"
  - "show me how to activate a specific cybersecurity skill"
  - "generate detection rules using the threat hunting skill"
  - "create an incident response playbook with Claude skills"
  - "audit this system using the blue team defense skill"
  - "analyze this malware with the malware analysis skill"
  - "how do I extend the cybersecurity skills collection"
---

# Claude Code CyberSecurity Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

The Claude Code CyberSecurity Skill collection is a comprehensive set of 15 production-quality SKILL.md files that transform Claude Code into a cybersecurity expert. Each skill provides structured methodology, decision frameworks, ready-to-run commands, and output templates for specific security domains.

**What this collection provides:**
- **Skill-based expertise** — Claude reads SKILL.md files at conversation start to gain domain knowledge
- **Native Claude Code integration** — Skills leverage Claude's bash, file reading, and analysis capabilities
- **Structured workflows** — Step-by-step procedures for reconnaissance, analysis, hunting, and defense
- **Output templates** — Exact formats for YARA rules, Sigma rules, SIEM queries, and reports
- **Automation scripts** — Python utilities included with each skill for complex operations
- **Authorization gates** — Built-in compliance checks for offensive security skills

**Skill domains covered:**
1. Recon & OSINT
2. Vulnerability Scanning
3. Exploit Development
4. Reverse Engineering
5. Malware Analysis
6. Threat Hunting
7. Incident Response
8. Network Security
9. Web Security
10. Cloud Security
11. CSOC Automation
12. Log Analysis & SIEM
13. Cryptographic Analysis
14. Red Team Operations
15. Blue Team Defense

## Installation

### Clone the Repository

```bash
git clone https://github.com/Masriyan/Claude-Code-CyberSecurity-Skill.git
cd Claude-Code-CyberSecurity-Skill
```

### Install Skills Globally (Recommended)

Claude Code loads skills from `~/.claude/skills/` (global) or `./.claude/skills/` (project-specific).

```bash
# Create global skills directory
mkdir -p ~/.claude/skills

# Copy all skills
cp -r skills/* ~/.claude/skills/

# Verify installation
ls ~/.claude/skills/
```

### Install Skills via Symlink (Development Mode)

For active development or testing skill modifications:

```bash
# Symlink individual skills
ln -sf "$(pwd)/skills/01-recon-osint" ~/.claude/skills/
ln -sf "$(pwd)/skills/06-threat-hunting" ~/.claude/skills/

# Or symlink all skills
for skill in skills/*/; do
  ln -sf "$(pwd)/$skill" ~/.claude/skills/
done
```

### Install Skills per Project

For project-specific skills:

```bash
# In your project directory
mkdir -p ./.claude/skills
cp -r /path/to/Claude-Code-CyberSecurity-Skill/skills/06-threat-hunting ./.claude/skills/
```

## Skill Structure

Each skill follows this directory structure:

```
skills/XX-skill-name/
├── SKILL.md           # Main skill definition (read by Claude)
├── scripts/           # Python automation scripts
│   └── tool.py
├── examples/          # Sample inputs/outputs
│   └── example.txt
└── resources/         # Reference data (CVE lists, ATT&CK mappings, etc.)
    └── reference.json
```

### SKILL.md Anatomy

```yaml
---
name: skill-identifier
description: One-line description of what this skill does
tags: [security, domain, specialty]
---

# Skill Name

## Activation Triggers
- "phrase that should activate this skill"
- "another natural language trigger"

## Methodology
Step-by-step procedures Claude follows natively.

## Output Templates
Exact formats for artifacts Claude generates.

## Script Usage
When and how to use included automation scripts.

## Authorization Requirements (offensive skills only)
Required confirmations before proceeding.
```

## Using the Skills

### Implicit Activation (Natural Language)

Claude automatically activates the relevant skill based on your request:

```python
# Example: User prompt activates Skill 06 (Threat Hunting)
"""
Extract IOCs from this phishing email and map the TTPs to MITRE ATT&CK.
Generate Splunk SPL queries to hunt for this activity in our environment.
"""

# Claude reads skills/06-threat-hunting/SKILL.md and follows its methodology
# Output includes:
# - IOC extraction (IPs, domains, hashes)
# - ATT&CK technique mapping
# - Ready-to-run Splunk queries
# - Hunting playbook
```

### Explicit Skill Invocation

Force activation of a specific skill:

```python
# Example: Explicitly invoke Malware Analysis skill
"""
Use the malware-analysis skill to analyze this PE file.
Generate YARA rules and extract all embedded strings.
"""
```

### Multi-Skill Workflows

Chain multiple skills for complex operations:

```python
# Example: Red Team → Blue Team workflow
"""
First, use the red-team-ops skill to design a credential dumping attack.
Then, use the blue-team-defense skill to create detection rules for it.
Finally, use the log-analysis skill to write Sentinel KQL queries.
"""
```

## Key Skills Reference

### Skill 01: Recon & OSINT

**Activation triggers:**
- "enumerate subdomains for example.com"
- "fingerprint the web stack on this target"
- "run WHOIS and DNS analysis"

**Example workflow:**

```bash
# Claude uses bash tool to run reconnaissance
subfinder -d example.com -o subdomains.txt
nmap -sV -p 80,443 -iL subdomains.txt
whatweb https://example.com

# Claude analyzes output and generates structured report
```

**Output template:**

```markdown
## Reconnaissance Report: example.com

### Subdomains Discovered (12)
- www.example.com (Apache 2.4.54, WordPress 6.1)
- api.example.com (Nginx 1.21.6, REST API)
- ...

### Technology Stack
- Web Server: Apache 2.4.54
- CMS: WordPress 6.1
- Plugins: Yoast SEO 19.3, Contact Form 7
- DNS: Cloudflare (1.1.1.1)

### Findings
- WordPress version outdated (CVE-2022-21661)
- TLS 1.0 enabled (deprecated)
```

### Skill 06: Threat Hunting

**Activation triggers:**
- "map these TTPs to MITRE ATT&CK"
- "generate Sigma rules for lateral movement"
- "write Splunk queries to hunt for DCSync"

**Example: Generate SIEM query from IOC**

```python
# User provides IOC
"""
Hunt for PowerShell Empire C2 beaconing to 203.0.113.42 on port 8080
"""

# Claude generates Splunk SPL query (from Skill 06 methodology)
```

**Claude output:**

```spl
index=proxy OR index=firewall
| search dest_ip="203.0.113.42" dest_port=8080
| stats count by src_ip, dest_ip, dest_port, bytes_out
| where count > 100
| eval beacon_interval=round(count/3600, 2)
| where beacon_interval > 0.5 AND beacon_interval < 5
| table src_ip, dest_ip, count, beacon_interval
```

**Sentinel KQL equivalent:**

```kql
CommonSecurityLog
| where DestinationIP == "203.0.113.42" and DestinationPort == 8080
| summarize Count=count(), TotalBytes=sum(SentBytes) by SourceIP, DestinationIP, DestinationPort
| where Count > 100
| extend BeaconInterval = Count / 3600.0
| where BeaconInterval > 0.5 and BeaconInterval < 5
| project SourceIP, DestinationIP, Count, BeaconInterval
```

### Skill 05: Malware Analysis

**Activation triggers:**
- "generate YARA rules from this malware sample"
- "extract IOCs from this PE file"
- "analyze this PowerShell script for obfuscation"

**Example: YARA rule generation**

```python
# User provides malware hash or file path
"""
Generate YARA rules for the ransomware sample at /tmp/sample.bin
Focus on unique strings and API call patterns.
"""
```

**Claude output (using Skill 05 template):**

```yara
rule Ransomware_Sample_202406 {
    meta:
        author = "Claude Code + Malware Analysis Skill"
        date = "2024-06-22"
        description = "Detects ransomware sample based on unique strings and APIs"
        hash = "a1b2c3d4e5f6..."
        
    strings:
        $str1 = "YOUR FILES HAVE BEEN ENCRYPTED" ascii wide
        $str2 = "bitcoin:" ascii wide
        $api1 = "CryptEncrypt" ascii
        $api2 = "CreateFileW" ascii
        $api3 = "WriteFile" ascii
        
    condition:
        uint16(0) == 0x5A4D and
        filesize < 5MB and
        all of ($str*) and
        2 of ($api*)
}
```

### Skill 15: Blue Team Defense

**Activation triggers:**
- "harden this Ubuntu server following CIS benchmarks"
- "audit this Dockerfile for security issues"
- "generate a Windows hardening checklist"

**Example: Linux hardening automation**

```bash
# Claude uses included scripts/harden_linux.py
# User prompt:
"""
Use the blue-team-defense skill to harden this Ubuntu 24.04 server.
Apply CIS Level 1 recommendations.
"""
```

**Claude-generated hardening script:**

```bash
#!/bin/bash
# Ubuntu 24.04 CIS Level 1 Hardening
# Generated by Claude Code + Blue Team Defense Skill

# 1. Disable unused filesystems
echo "install cramfs /bin/true" >> /etc/modprobe.d/cramfs.conf
echo "install freevxfs /bin/true" >> /etc/modprobe.d/freevxfs.conf
echo "install jffs2 /bin/true" >> /etc/modprobe.d/jffs2.conf

# 2. Configure auditd
apt-get install -y auditd
systemctl enable auditd
cat > /etc/audit/rules.d/cis.rules <<EOF
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins
-a always,exit -F arch=b64 -S execve -k exec
EOF
auditctl -R /etc/audit/rules.d/cis.rules

# 3. SSH hardening
sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^X11Forwarding.*/X11Forwarding no/' /etc/ssh/sshd_config
systemctl reload sshd

# 4. Enable firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw --force enable

echo "[+] CIS Level 1 hardening complete"
```

## Configuration

### Global Skill Configuration

Create `~/.claude/config.yaml` to customize skill behavior:

```yaml
skills:
  enabled: true
  directories:
    - ~/.claude/skills
    - /opt/security-skills
  
  # Offensive skills require authorization
  authorization_required:
    - exploit-development
    - red-team-ops
  
  # Auto-load these skills for all sessions
  always_active:
    - log-analysis
    - threat-hunting
    - blue-team-defense
```

### Project-Specific Configuration

Create `.claude/config.yaml` in your project root:

```yaml
skills:
  directories:
    - ./.claude/skills
  
  # Only activate these skills for this project
  project_skills:
    - web-security
    - cloud-security
```

### Environment Variables for Scripts

Many skill scripts require external services or tools:

```bash
# ~/.bashrc or ~/.zshrc
export VIRUSTOTAL_API_KEY="${VIRUSTOTAL_API_KEY}"
export SHODAN_API_KEY="${SHODAN_API_KEY}"
export CENSYS_API_ID="${CENSYS_API_ID}"
export CENSYS_API_SECRET="${CENSYS_API_SECRET}"
export MISP_URL="${MISP_URL}"
export MISP_KEY="${MISP_KEY}"
```

## Common Patterns

### Pattern 1: IOC Extraction + Threat Intel Enrichment

```python
# User workflow
"""
Extract IOCs from this malware report PDF.
Enrich with VirusTotal and MISP.
Generate Sigma rules for detection.
"""

# Claude uses Skill 05 (Malware Analysis) + Skill 06 (Threat Hunting)
# Steps:
# 1. Extract IOCs using bash + grep/regex
# 2. Call scripts/vt_lookup.py with extracted hashes
# 3. Generate Sigma rule using Skill 06 template
```

**Example script execution:**

```bash
# Claude extracts IOCs
cat malware_report.txt | grep -oE '([0-9a-f]{64})' > hashes.txt
cat malware_report.txt | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' > ips.txt

# Claude runs VT enrichment script
python ~/.claude/skills/05-malware-analysis/scripts/vt_lookup.py \
  --hashes hashes.txt \
  --api-key "${VIRUSTOTAL_API_KEY}" \
  --output vt_results.json
```

### Pattern 2: Detection Rule Development Workflow

```python
# User scenario
"""
I need to detect Kerberoasting attacks in our Windows environment.
Generate detection rules for Splunk, Sentinel, and Sigma.
"""

# Claude uses Skill 06 (Threat Hunting) + Skill 12 (Log Analysis)
```

**Claude output (Sigma rule):**

```yaml
title: Kerberoasting - Service Ticket Request (RC4)
id: 496a0e47-0a33-4dca-b009-9e6ca3591f39
status: stable
description: Detects Kerberoasting by monitoring for service ticket requests with RC4 encryption
author: Claude Code + Threat Hunting Skill
references:
    - https://attack.mitre.org/techniques/T1558/003/
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 4769
        TicketEncryptionType: '0x17'
        ServiceName: '*$'
    filter:
        ServiceName: 'krbtgt'
    condition: selection and not filter
falsepositives:
    - Legitimate service accounts using RC4
level: high
tags:
    - attack.credential_access
    - attack.t1558.003
```

**Converted to Splunk SPL:**

```spl
index=windows EventCode=4769 Ticket_Encryption_Type=0x17 Service_Name!="krbtgt" Service_Name="*$"
| stats count by ComputerName, Account_Name, Service_Name
| where count > 5
| table _time, ComputerName, Account_Name, Service_Name, count
```

### Pattern 3: Incident Response Workflow

```python
# User scenario
"""
We detected ransomware on DESKTOP-ABC123.
Use the incident-response skill to guide me through containment and analysis.
"""

# Claude uses Skill 07 (Incident Response)
# Follows PICERL methodology:
# 1. Preparation
# 2. Identification
# 3. Containment
# 4. Eradication
# 5. Recovery
# 6. Lessons Learned
```

**Claude-generated IR playbook:**

```markdown
## Ransomware Incident Response: DESKTOP-ABC123

### Phase 1: Preparation (Pre-Incident)
✅ IR team contact list ready
✅ Forensic tools available (Volatility, FTK Imager)
✅ Network isolation procedure documented

### Phase 2: Identification
**Execute these commands on a live forensics workstation:**

```bash
# Capture memory dump (requires admin privileges)
DumpIt.exe /OUTPUT C:\Memory\DESKTOP-ABC123.raw

# Collect triage data
Get-Process | Export-Csv C:\IR\processes.csv
Get-NetTCPConnection | Export-Csv C:\IR\network.csv
Get-ScheduledTask | Export-Csv C:\IR\tasks.csv

# Check for common ransomware artifacts
Get-ChildItem C:\Users\*\Desktop\*README*.txt
Get-ChildItem C:\Users\*\Documents\*.locked
```

**Analyze with Volatility:**

```bash
python vol.py -f DESKTOP-ABC123.raw windows.psscan
python vol.py -f DESKTOP-ABC123.raw windows.netscan
python vol.py -f DESKTOP-ABC123.raw windows.malfind
```

### Phase 3: Containment
```powershell
# Immediate network isolation
Disable-NetAdapter -Name "Ethernet" -Confirm:$false

# Disable scheduled tasks
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"} | Disable-ScheduledTask

# Kill malicious processes (example)
Stop-Process -Name "ransomware.exe" -Force
```

## Troubleshooting

### Skills Not Loading

**Symptom:** Claude doesn't seem to have skill knowledge.

**Solution:**

```bash
# Verify skills directory exists
ls -la ~/.claude/skills/

# Check SKILL.md files are present
find ~/.claude/skills/ -name "SKILL.md"

# Ensure correct permissions
chmod -R 755 ~/.claude/skills/

# Restart Claude Code session
```

### Script Execution Fails

**Symptom:** Python scripts throw import errors.

**Solution:**

```bash
# Install required dependencies
cd ~/.claude/skills/XX-skill-name
pip install -r requirements.txt

# Verify Python version
python3 --version  # Should be 3.10+

# Check script permissions
chmod +x scripts/*.py
```

### API Key Errors

**Symptom:** `VirusTotal API key not found` or similar.

**Solution:**

```bash
# Set environment variables
export VIRUSTOTAL_API_KEY="your-api-key-here"

# Make persistent (add to ~/.bashrc)
echo 'export VIRUSTOTAL_API_KEY="your-api-key-here"' >> ~/.bashrc
source ~/.bashrc

# Verify
echo $VIRUSTOTAL_API_KEY
```

### Offensive Skills Won't Activate

**Symptom:** Claude refuses to assist with Exploit Development or Red Team skills.

**Solution:**

Offensive skills (03, 14) require explicit authorization confirmation:

```python
# Correct workflow
"""
I have written authorization from the target organization.
Scope: 10.0.0.0/24 internal network pentest.
Use the exploit-development skill to create a reverse shell payload.
"""

# Claude will now proceed after confirming authorization
```

## Extending the Skills

### Creating a Custom Skill

Create a new skill directory:

```bash
mkdir -p ~/.claude/skills/16-custom-skill
cd ~/.claude/skills/16-custom-skill
```

Create `SKILL.md`:

```yaml
---
name: custom-security-skill
description: Custom skill for specific security operations
tags: [security, custom, specialty]
---

# Custom Security Skill

## Activation Triggers
- "use my custom security skill"
- "run custom analysis on this target"

## Methodology

### Step 1: Initial Analysis
Claude reads the target file/configuration.

### Step 2: Pattern Matching
Claude applies custom regex patterns or rules.

### Step 3: Output Generation
Claude generates a structured report using the template below.

## Output Template

```markdown
## Custom Analysis Report

### Summary
Brief overview of findings.

### Details
- Finding 1: Description
- Finding 2: Description

### Recommendations
1. Action item 1
2. Action item 2
```

## Script Usage

Use `scripts/custom_tool.py` for complex analysis:

```bash
python scripts/custom_tool.py --input target.conf --output report.json
```
```

Add automation script:

```bash
mkdir scripts
cat > scripts/custom_tool.py << 'EOF'
#!/usr/bin/env python3
"""
Custom Security Analysis Tool
"""
import argparse
import json

def analyze(input_file):
    """Analyze the input file."""
    with open(input_file, 'r') as f:
        data = f.read()
    
    findings = []
    # Add your custom analysis logic here
    
    return findings

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Custom security analysis")
    parser.add_argument('--input', required=True, help='Input file path')
    parser.add_argument('--output', required=True, help='Output JSON path')
    args = parser.parse_args()
    
    results = analyze(args.input)
    
    with open(args.output, 'w') as f:
        json.dump(results, f, indent=2)
    
    print(f"[+] Analysis complete: {len(results)} findings")
EOF

chmod +x scripts/custom_tool.py
```

### Contributing Back to the Project

1. Fork the repository
2. Create a feature branch
3. Add your skill following the structure above
4. Test with Claude Code
5. Submit a pull request

```bash
git checkout -b feature/new-skill
git add skills/16-custom-skill/
git commit -m "Add custom security skill for XYZ analysis"
git push origin feature/new-skill
```

## Additional Resources

**Official documentation:**
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Creating Custom Skills](https://docs.anthropic.com/en/docs/claude-code/skills)

**Skill-specific resources:**
- MITRE ATT&CK Framework: https://attack.mitre.org/
- Sigma Rule Repository: https://github.com/SigmaHQ/sigma
- YARA Rules: https://github.com/Yara-Rules/rules
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks/

**Community:**
- GitHub Issues: https://github.com/Masriyan/Claude-Code-CyberSecurity-Skill/issues
- Security Blog: https://www.security-life.org

---

**Next steps:**
1. Install the skills collection globally
2. Test with a simple prompt: "Use the threat-hunting skill to generate a Sigma rule for lateral movement"
3. Explore individual skill SKILL.md files for domain-specific methodology
4. Extend with your own custom skills as needed
