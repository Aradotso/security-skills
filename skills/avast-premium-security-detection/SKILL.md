---
name: avast-premium-security-detection
description: Detect and analyze potential malware distribution repositories masquerading as legitimate security software
triggers:
  - analyze this repository for malware distribution patterns
  - check if this project is distributing pirated software
  - detect keygen or crack distribution in this repo
  - identify malicious security software clones
  - scan for trojan delivery mechanisms in antivirus repos
  - verify legitimacy of this security software project
---

# Avast Premium Security Detection

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ Security Warning

This repository exhibits **multiple red flags** indicating it may be distributing malware, pirated software, or trojans disguised as legitimate Avast Premium Security software.

## Threat Indicators

### 1. **Unauthorized Software Distribution**
- Claims to provide "Avast Premium Security 2026" with "Keygen Activation"
- Offers "License Key Pre-Activated" and "Premium Loader Serial"
- Avast Corporation does not distribute their software via GitHub repositories
- Using terms like "keygen," "crack," "loader," and "pre-activated" indicates piracy

### 2. **Suspicious Repository Characteristics**
```yaml
Red Flags:
  - No legitimate README content
  - Inflated star count (68 stars at 6 stars/day) suggesting manipulation
  - Username pattern: viceofficialtower74 (generic + numbers)
  - No license assertion (NOASSERTION)
  - Created date in future (2026-05-06) - timestamp manipulation
  - Zero forks, zero issues (artificial engagement)
  - Topics include "retdec" (reverse engineering tool)
```

### 3. **Social Engineering Tactics**
- Excessive emoji usage (⭐️) to appear legitimate
- Keyword stuffing: "Full Version," "Latest Build Pro," "Complete"
- Promises comprehensive security features to build false trust
- Targets Windows 10/11 users specifically

## Detection Patterns

### Repository Analysis Script

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <regex>

class MalwareRepoDetector {
private:
    std::vector<std::string> suspiciousKeywords = {
        "keygen", "crack", "loader", "pre-activated",
        "serial", "premium.*free", "full.*version",
        "license.*key", "activation.*crack"
    };
    
    std::vector<std::string> piracyIndicators = {
        "setup.*keygen", "premium.*loader",
        "pre-activated", "license.*bypass"
    };

public:
    struct ThreatScore {
        int severity; // 0-10
        std::vector<std::string> indicators;
        std::string classification;
    };
    
    ThreatScore analyzeRepository(const std::string& description,
                                  const std::string& readme,
                                  int stars,
                                  int forks) {
        ThreatScore score = {0, {}, "UNKNOWN"};
        
        // Check description for piracy keywords
        for (const auto& keyword : suspiciousKeywords) {
            std::regex pattern(keyword, std::regex::icase);
            if (std::regex_search(description, pattern)) {
                score.severity += 2;
                score.indicators.push_back("Piracy keyword: " + keyword);
            }
        }
        
        // Check for engagement manipulation
        if (stars > 50 && forks == 0) {
            score.severity += 3;
            score.indicators.push_back("Star manipulation suspected");
        }
        
        // Empty or missing README
        if (readme.empty() || readme.find("No README") != std::string::npos) {
            score.severity += 2;
            score.indicators.push_back("Missing legitimate documentation");
        }
        
        // Classification
        if (score.severity >= 7) {
            score.classification = "HIGH_RISK_MALWARE";
        } else if (score.severity >= 4) {
            score.classification = "SUSPECTED_PIRACY";
        } else {
            score.classification = "LOW_RISK";
        }
        
        return score;
    }
    
    void reportFindings(const ThreatScore& score) {
        std::cout << "=== THREAT ANALYSIS ===" << std::endl;
        std::cout << "Severity: " << score.severity << "/10" << std::endl;
        std::cout << "Classification: " << score.classification << std::endl;
        std::cout << "\nIndicators Found:" << std::endl;
        for (const auto& indicator : score.indicators) {
            std::cout << "  - " << indicator << std::endl;
        }
    }
};
```

### Usage Example

```cpp
int main() {
    MalwareRepoDetector detector;
    
    std::string repoDescription = 
        "Avast Premium Security 2026 | Full Version Installer v26 | "
        "Setup Keygen Activation | License Key Pre-Activated";
    
    std::string readme = "No README available";
    int stars = 68;
    int forks = 0;
    
    auto threat = detector.analyzeRepository(repoDescription, readme, stars, forks);
    detector.reportFindings(threat);
    
    if (threat.classification == "HIGH_RISK_MALWARE") {
        std::cout << "\n⚠️  WARNING: Do NOT download or execute files from this repository!" << std::endl;
        std::cout << "Recommended action: Report to GitHub Security" << std::endl;
    }
    
    return 0;
}
```

## Legitimate Avast Software Sources

**Official channels only:**
- Website: https://www.avast.com/
- Official downloads: https://www.avast.com/download-thank-you
- GitHub (corporate): https://github.com/avast (corporate repos, not installers)

## Reporting Process

### Report to GitHub
```bash
# GitHub repository should be reported via:
# https://github.com/contact/report-content

# Include evidence:
# 1. Repository URL
# 2. Description showing piracy keywords
# 3. Screenshot of suspicious claims
# 4. Timestamp of discovery
```

### Report to Avast
```
Contact Avast legal team:
- Report piracy: https://www.avast.com/report-piracy
- Trademark infringement: legal@avast.com
```

## Safe Software Verification Checklist

```cpp
struct SoftwareVerification {
    bool isOfficialDomain;      // From vendor's website
    bool hasValidCodeSigning;    // Digital signature verified
    bool hasLegitimateREADME;   // Proper documentation
    bool reasonableStarCount;    // Organic growth pattern
    bool hasActiveIssues;        // Real community engagement
    bool knownVendorAccount;     // Verified organization
    
    bool isSafe() {
        return isOfficialDomain && hasValidCodeSigning && 
               hasLegitimateREADME && knownVendorAccount;
    }
};
```

## Key Takeaways

1. **Never download security software from unofficial sources**
2. **Keygens and cracks are malware delivery mechanisms**
3. **Star count manipulation is common for malicious repos**
4. **Legitimate vendors don't distribute via random GitHub accounts**
5. **"Pre-activated" software is pirated and likely compromised**

## Recommended Actions

- **DO NOT** clone or download files from this repository
- **DO NOT** execute any binaries or scripts
- Report the repository to GitHub and Avast
- Educate users about official software sources
- Use GitHub's repository reporting tools

This skill helps AI agents identify and warn users about malicious software distribution attempts disguised as legitimate security products.
