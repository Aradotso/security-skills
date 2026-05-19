```markdown
---
name: security-awareness-malicious-repository-detection
description: Identify and analyze potentially malicious repositories masquerading as legitimate security software cracks or activators
triggers:
  - detect malicious software repository
  - analyze suspicious crack download repo
  - identify fake antivirus keygen project
  - check if repository contains malware
  - evaluate security software crack legitimacy
  - scan repository for malicious patterns
  - assess fake license key generator risks
  - investigate suspicious activation tool repo
---

# Security Awareness: Malicious Repository Detection

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ CRITICAL SECURITY WARNING

**This repository is HIGHLY SUSPICIOUS and likely MALICIOUS.** It exhibits multiple red flags characteristic of malware distribution:

### Red Flags Present

1. **Fraudulent Purpose**: Claims to provide "cracked" commercial antivirus software
2. **Contradictory Topics**: Tags include both "defender-bypass" and "antivirus-tools" - indicating malicious intent
3. **Suspicious Growth**: 59 stars at 4 stars/day suggests artificial inflation
4. **No README**: Legitimate projects provide documentation
5. **Deceptive Naming**: Uses "MistDuckCount" as owner to appear innocuous
6. **Commercial Software Piracy**: Distributes unauthorized copies of paid software
7. **Keygen/Loader References**: Classic malware distribution terminology

## What This Actually Represents

This is a **malware distribution vector** disguised as:
- Antivirus software crack
- License key generator
- Pre-activated installer

### Likely Payloads

```go
// Common malware patterns in fake crack repositories:

// 1. Information Stealer
func exfiltrateData() {
    // Steals credentials, browser data, crypto wallets
    // Sends to C2 server
}

// 2. Ransomware Dropper
func deployRansomware() {
    // Encrypts user files
    // Demands payment
}

// 3. Remote Access Trojan (RAT)
func establishBackdoor() {
    // Opens persistent backdoor
    // Allows remote control
}

// 4. Cryptominer
func mineWithoutConsent() {
    // Uses system resources for cryptocurrency mining
}
```

## Detection Indicators

### Repository Analysis Checklist

```go
package main

import (
    "fmt"
    "strings"
)

type RepoRisk struct {
    Indicators []string
    RiskScore  int
}

func AnalyzeRepository(desc, topics []string) RepoRisk {
    risk := RepoRisk{Indicators: []string{}}
    
    // Check for crack/keygen keywords
    suspiciousTerms := []string{
        "crack", "keygen", "loader", "pre-activated",
        "bypass", "license key", "full version",
    }
    
    for _, term := range suspiciousTerms {
        if strings.Contains(strings.ToLower(desc), term) {
            risk.Indicators = append(risk.Indicators, 
                fmt.Sprintf("Contains suspicious term: %s", term))
            risk.RiskScore += 20
        }
    }
    
    // Check for contradictory security topics
    securityBypass := []string{"defender-bypass", "exploit-mitigation"}
    for _, topic := range topics {
        for _, bypass := range securityBypass {
            if topic == bypass {
                risk.Indicators = append(risk.Indicators, 
                    "Contains security bypass topic")
                risk.RiskScore += 30
            }
        }
    }
    
    return risk
}
```

## Safe Alternatives

### Legitimate Security Software Acquisition

```go
package main

import "fmt"

// Safe sources for security software
var LegitimateVendors = map[string]string{
    "Bitdefender": "https://www.bitdefender.com",
    "Free Trials": "Official vendor websites only",
    "Open Source": "GitHub repos with verifiable history",
}

func GetLegitimateSource(product string) {
    fmt.Println("NEVER download from:")
    fmt.Println("❌ Crack/keygen repositories")
    fmt.Println("❌ Third-party download sites")
    fmt.Println("❌ Repositories with no commit history")
    
    fmt.Println("\nALWAYS use:")
    fmt.Println("✅ Official vendor websites")
    fmt.Println("✅ Authorized resellers")
    fmt.Println("✅ Free/open-source alternatives")
}
```

## Protecting Yourself

### Verification Steps

```go
package security

import (
    "crypto/sha256"
    "encoding/hex"
    "os"
)

// Verify file integrity before execution
func VerifyFileIntegrity(filePath string, expectedHash string) (bool, error) {
    file, err := os.ReadFile(filePath)
    if err != nil {
        return false, err
    }
    
    hash := sha256.Sum256(file)
    computedHash := hex.EncodeToString(hash[:])
    
    return computedHash == expectedHash, nil
}

// Check digital signature (Windows example)
func CheckDigitalSignature(exePath string) bool {
    // Only run executables signed by trusted publishers
    // Use OS-level verification:
    // Windows: signtool verify /pa file.exe
    // Linux: gpg --verify file.sig file
    return false // Default to unsafe
}
```

### Safe Repository Evaluation

```go
package evaluation

import "time"

type RepoMetrics struct {
    Age            time.Duration
    CommitCount    int
    Contributors   int
    HasDocumentation bool
    License        string
}

func IsRepoTrustworthy(metrics RepoMetrics) bool {
    // Red flags
    if metrics.Age < 30*24*time.Hour {
        return false // Too new
    }
    
    if metrics.CommitCount < 10 {
        return false // No development history
    }
    
    if !metrics.HasDocumentation {
        return false // No README/docs
    }
    
    if metrics.License == "NOASSERTION" || metrics.License == "" {
        return false // No clear license
    }
    
    if metrics.Contributors < 2 {
        return false // Single contributor suspicious
    }
    
    return true
}
```

## Response Actions

### If You've Already Downloaded

```bash
# IMMEDIATE ACTIONS:

# 1. Disconnect from network
# Prevent data exfiltration and C2 communication

# 2. DO NOT RUN any downloaded files

# 3. Scan with multiple antivirus tools
clamscan -r --bell -i /path/to/download
# Or use Windows Defender, Malwarebytes, etc.

# 4. Delete all downloaded files
rm -rf /path/to/suspicious/download

# 5. Change all passwords from a CLEAN device

# 6. Monitor accounts for unauthorized access
# Check: email, banking, cryptocurrency wallets
```

## Educational Purpose

### Understanding Malware Distribution Tactics

```go
package education

// Common social engineering tactics used:
type MalwareTactic struct {
    Method      string
    Description string
    Protection  string
}

var CommonTactics = []MalwareTactic{
    {
        Method:      "SEO Poisoning",
        Description: "Ranks high in search results for 'software crack'",
        Protection:  "Never search for cracks/keygens",
    },
    {
        Method:      "Star Inflation",
        Description: "Fake stars to appear popular/trustworthy",
        Protection:  "Check contributor diversity and commit history",
    },
    {
        Method:      "Technical Terms",
        Description: "Uses legitimate security terminology",
        Protection:  "Verify claims against official documentation",
    },
}
```

## Reporting

### Report Malicious Repositories

```bash
# GitHub abuse reporting
# Navigate to repository
# Click: "..." menu → "Report repository"
# Select: "Malware or harmful content"

# Or via email:
# support@github.com
# Include: Repository URL, description of malicious behavior
```

## Key Takeaways

1. **This repository is MALWARE distribution** - do not download or execute anything
2. **Cracks/keygens are ALWAYS dangerous** - they frequently contain malware
3. **Use official sources** - only download software from verified vendors
4. **Free alternatives exist** - use legitimate open-source security tools instead
5. **Piracy funds cybercrime** - illegal software funds malware operations

## Legitimate Alternatives

```go
package alternatives

// Free, open-source security tools
var FreeSecurityTools = map[string]string{
    "Antivirus":        "ClamAV, Windows Defender",
    "Firewall":         "Built-in OS firewalls",
    "Malware Scanner":  "Malwarebytes Free",
    "Password Manager": "Bitwarden, KeePassXC",
    "VPN":             "ProtonVPN Free, WireGuard",
}
```

---

**Remember**: If something seems too good to be true (free commercial software), it is. Protect yourself and your data by only using legitimate sources.
```
