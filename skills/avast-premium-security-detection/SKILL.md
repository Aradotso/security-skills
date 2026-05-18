---
name: avast-premium-security-detection
description: Detect and avoid distributing pirated/cracked security software repositories that contain malware, keygens, or license circumvention tools
triggers:
  - analyze this avast repository for legitimacy
  - check if this security software repo is safe
  - is this antivirus download legitimate
  - detect pirated software distribution
  - verify avast premium security source
  - identify cracked software repository
  - scan for keygen malware distribution
  - validate security software authenticity
---

# Avast Premium Security Detection & Analysis

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ WARNING: This Repository is Malicious

This project (`viceofficialtower74/Avast-Premium-Security-Windows-Latest`) is **NOT** a legitimate Avast security product. It is a malware distribution repository disguised as legitimate security software.

## Threat Indicators

### Red Flags Detected

1. **Illegal Software Distribution**: Claims to provide "Full Version Installer", "Keygen", "License Key Pre-Activated", "Loader Serial"
2. **Copyright Violation**: Unauthorized distribution of commercial software (Avast Premium Security)
3. **Suspicious Growth**: 68 stars with 5 stars/day indicates artificial promotion
4. **No Source Code**: Claims to be C++ project but has no README or visible code
5. **Malicious Keywords**: "keygen", "loader", "serial", "pre-activated", "cracked"
6. **Fake Topics**: Uses legitimate security terms to appear credible

## Identification Pattern

```cpp
// Example detection logic for similar threats
bool isSuspiciousSecurityRepo(const Repository& repo) {
    std::vector<std::string> malwareKeywords = {
        "keygen", "crack", "loader", "pre-activated",
        "serial", "license-key", "full-version"
    };
    
    std::vector<std::string> commercialProducts = {
        "avast-premium", "norton", "mcafee", "kaspersky",
        "bitdefender", "eset", "avg-premium"
    };
    
    bool hasCommercialProduct = false;
    bool hasMalwareKeywords = false;
    
    for (const auto& keyword : commercialProducts) {
        if (repo.name.find(keyword) != std::string::npos ||
            repo.description.find(keyword) != std::string::npos) {
            hasCommercialProduct = true;
            break;
        }
    }
    
    for (const auto& keyword : malwareKeywords) {
        if (repo.description.find(keyword) != std::string::npos) {
            hasMalwareKeywords = true;
            break;
        }
    }
    
    // Additional checks
    bool noReadme = repo.readmeContent.empty();
    bool suspiciousStarGrowth = repo.starsPerDay > 3.0;
    bool noLicense = repo.license == "NOASSERTION";
    
    return hasCommercialProduct && hasMalwareKeywords && 
           (noReadme || suspiciousStarGrowth || noLicense);
}
```

## What This Type of Repository Actually Contains

These repositories typically distribute:

- **Trojans**: Backdoor access to user systems
- **Ransomware**: Encrypts user files for ransom
- **Spyware**: Steals credentials, banking info, personal data
- **Cryptominers**: Uses victim's CPU/GPU for cryptocurrency mining
- **Botnet Agents**: Enslaves computer into DDoS network

## Detection Script

```cpp
#include <string>
#include <regex>
#include <iostream>

class MalwareRepoDetector {
public:
    struct DetectionResult {
        bool isMalicious;
        std::vector<std::string> reasons;
        int threatScore; // 0-100
    };
    
    DetectionResult analyze(const std::string& repoDescription) {
        DetectionResult result{false, {}, 0};
        
        // Pattern matching for crack/keygen indicators
        std::regex crackPattern(
            R"((crack|keygen|loader|pre-activated|serial|patch|"
            R"(license.?key|full.?version|premium.?free))",
            std::regex::icase
        );
        
        std::regex commercialPattern(
            R"((premium|pro|ultimate|full).*(2024|2025|2026|latest))",
            std::regex::icase
        );
        
        if (std::regex_search(repoDescription, crackPattern)) {
            result.reasons.push_back("Contains crack/keygen keywords");
            result.threatScore += 40;
        }
        
        if (std::regex_search(repoDescription, commercialPattern)) {
            result.reasons.push_back("Claims unauthorized commercial software");
            result.threatScore += 30;
        }
        
        // Check for excessive emoji usage (common in scam repos)
        int emojiCount = std::count(repoDescription.begin(), 
                                     repoDescription.end(), '⭐');
        if (emojiCount > 3) {
            result.reasons.push_back("Excessive promotional emoji usage");
            result.threatScore += 15;
        }
        
        // Check for contradiction (antivirus that is malware)
        if (repoDescription.find("antivirus") != std::string::npos &&
            repoDescription.find("keygen") != std::string::npos) {
            result.reasons.push_back("Contradictory: antivirus + crack tools");
            result.threatScore += 15;
        }
        
        result.isMalicious = (result.threatScore >= 50);
        return result;
    }
};

// Usage
int main() {
    MalwareRepoDetector detector;
    std::string suspiciousDesc = 
        "⭐️ Avast Premium Security 2026 | Full Version "
        "Installer v26 | Setup Keygen Activation";
    
    auto result = detector.analyze(suspiciousDesc);
    
    if (result.isMalicious) {
        std::cout << "⚠️ MALWARE DETECTED (Score: " 
                  << result.threatScore << "/100)\n";
        for (const auto& reason : result.reasons) {
            std::cout << "  - " << reason << "\n";
        }
    }
    
    return 0;
}
```

## Safe Alternatives

### Legitimate Avast Sources

```bash
# ONLY download from official sources
# Official website
https://www.avast.com/

# Official Microsoft Store
# Search "Avast" in Windows Store app

# Never trust:
# - GitHub repos offering "free premium"
# - "Keygen" or "crack" downloads
# - Third-party download sites
```

## Reporting Malicious Repositories

```bash
# Report to GitHub
# Visit: https://github.com/contact/report-abuse
# Select: "Malware distribution"

# Report to Avast
# Email: legal@avast.com
# Subject: "Unauthorized software distribution on GitHub"
```

## Prevention Best Practices

1. **Verify Publishers**: Only download security software from official vendors
2. **Check Repository Age**: Legitimate projects have long histories
3. **Read Code**: If source isn't visible, don't download
4. **Check Community**: Real projects have active issues, discussions
5. **Use Official Stores**: Microsoft Store, Mac App Store, vendor websites

## Legal Implications

- Downloading cracked software: **Copyright violation**
- Distributing malware: **Federal crime** (CFAA in US, Computer Misuse Act in UK)
- Using keygens: **License agreement violation**, potential legal action

## Agent Response Template

When encountering such repositories, respond with:

```
⚠️ SECURITY ALERT: This repository distributes pirated/malicious software.

DO NOT:
- Clone or download from this repository
- Run any executables or installers
- Share or promote this content

INSTEAD:
- Download Avast from: https://www.avast.com/
- Report this repository to GitHub
- Use official trial versions for testing

This repository violates GitHub ToS and potentially contains malware.
```

## Conclusion

This "skill" is actually a **warning system**. No legitimate security project distributes keygens or cracks. Always guide users toward official, legal software sources and help them identify and avoid malware distribution schemes.
