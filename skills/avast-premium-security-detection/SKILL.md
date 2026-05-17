---
name: avast-premium-security-detection
description: Detect and analyze potentially malicious software distribution repositories masquerading as legitimate security software
triggers:
  - identify fake antivirus repositories
  - detect malware distribution on GitHub
  - analyze suspicious security software repos
  - check for piracy and keygen indicators
  - verify legitimate antivirus sources
  - detect software activation scams
  - analyze repository red flags for malware
  - identify social engineering in software repos
---

# Avast Premium Security Detection

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill helps identify and analyze repositories that appear to distribute pirated, cracked, or potentially malicious versions of legitimate security software. The referenced repository exhibits multiple red flags common in malware distribution campaigns.

## Warning Signs Analysis

### Repository Red Flags

The project `viceofficialtower74/Avast-Premium-Security-Windows-Latest` exhibits several critical indicators of a malicious or scam repository:

1. **Keyword Stuffing**: Title includes "Keygen", "Activation", "License Key Pre-Activated", "Loader Serial"
2. **Piracy Claims**: Promises "Full Version" with pre-activation bypassing legitimate licensing
3. **No Source Code**: Claims C++ language but provides no visible code
4. **Suspicious Metrics**: Artificial star growth (6 stars/day) suggesting bot activity
5. **Generic Username**: Random-seeming account name `viceofficialtower74`
6. **Missing README**: No legitimate installation or usage documentation
7. **No License**: NOASSERTION license status for commercial software redistribution

## Detection Patterns

### Identifying Fake Security Software Repos

```cpp
// Conceptual detection patterns (not actual code from repo)
#include <string>
#include <vector>
#include <regex>

struct RepoRiskIndicators {
    bool has_keygen_keywords;
    bool promises_pre_activation;
    bool lacks_source_code;
    bool suspicious_star_growth;
    bool missing_documentation;
    int risk_score;
};

RepoRiskIndicators analyzeRepository(const std::string& description, 
                                     const std::vector<std::string>& topics) {
    RepoRiskIndicators indicators = {};
    
    // Check for piracy-related keywords
    std::vector<std::string> piracy_terms = {
        "keygen", "crack", "pre-activated", "loader", 
        "serial", "license key", "activation"
    };
    
    for (const auto& term : piracy_terms) {
        if (description.find(term) != std::string::npos) {
            indicators.has_keygen_keywords = true;
            indicators.risk_score += 20;
            break;
        }
    }
    
    // Check for activation bypass claims
    std::regex activation_pattern(R"(pre-?activated|full version|cracked)");
    if (std::regex_search(description, activation_pattern)) {
        indicators.promises_pre_activation = true;
        indicators.risk_score += 25;
    }
    
    return indicators;
}
```

### Metadata Analysis

```cpp
#include <json/json.h>
#include <chrono>

bool detectArtificialEngagement(const nlohmann::json& metadata) {
    int stars = metadata["stars"];
    
    // Parse dates
    std::string created = metadata["created_at"];
    std::string updated = metadata["updated_at"];
    
    // Calculate days active
    auto creation_time = parseISO8601(created);
    auto update_time = parseISO8601(updated);
    auto days_active = std::chrono::duration_cast<std::chrono::days>(
        update_time - creation_time
    ).count();
    
    // Suspicious: 68 stars in 11 days (6.2 stars/day) with 0 forks
    double stars_per_day = static_cast<double>(stars) / days_active;
    int forks = metadata["forks"];
    
    if (stars_per_day > 3.0 && forks == 0) {
        return true; // Likely bot-driven engagement
    }
    
    return false;
}
```

## Security Assessment Framework

### Risk Scoring System

```cpp
#include <iostream>
#include <map>

enum class RiskLevel {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
};

class SecurityRepoAnalyzer {
public:
    RiskLevel assessRepository(const std::string& repo_url) {
        int total_score = 0;
        
        // Criteria scoring
        std::map<std::string, int> criteria = {
            {"keygen_keywords", 25},
            {"no_source_code", 20},
            {"fake_stars", 15},
            {"missing_readme", 10},
            {"promises_bypass", 30}
        };
        
        // Analyze each criterion (pseudo-implementation)
        if (containsKeygenKeywords()) total_score += criteria["keygen_keywords"];
        if (hasNoSourceCode()) total_score += criteria["no_source_code"];
        if (artificialEngagement()) total_score += criteria["fake_stars"];
        if (missingDocumentation()) total_score += criteria["missing_readme"];
        if (promisesLicenseBypass()) total_score += criteria["promises_bypass"];
        
        // Determine risk level
        if (total_score >= 70) return RiskLevel::CRITICAL;
        if (total_score >= 50) return RiskLevel::HIGH;
        if (total_score >= 30) return RiskLevel::MEDIUM;
        return RiskLevel::LOW;
    }
    
private:
    bool containsKeygenKeywords() { return true; }
    bool hasNoSourceCode() { return true; }
    bool artificialEngagement() { return true; }
    bool missingDocumentation() { return true; }
    bool promisesLicenseBypass() { return true; }
};
```

## Legitimate Avast Installation

### Proper Download Sources

```cpp
// Safe download verification pattern
#include <string>
#include <openssl/sha.h>

struct LegitimateSource {
    std::string vendor = "Avast Software s.r.o.";
    std::string official_url = "https://www.avast.com/";
    std::string download_domain = "download.avast.com";
    bool require_https = true;
    bool verify_signature = true;
};

bool verifyAvastInstaller(const std::string& file_path) {
    // Verify digital signature (Windows-specific)
    // Check: Publisher = "Avast Software s.r.o."
    // Check: Certificate chain valid
    // Check: SHA-256 hash matches official release
    
    return verifyCodeSignature(file_path, "Avast Software s.r.o.");
}
```

## Red Flags Checklist

When evaluating security software repositories:

- ❌ **Never download** from repositories promising "cracks" or "keygens"
- ❌ **Avoid** repos with no visible source code claiming to be open source
- ❌ **Distrust** artificial engagement (high stars, zero forks/issues)
- ❌ **Reject** pre-activated or license-bypassing installers
- ✅ **Use** official vendor websites for commercial software
- ✅ **Verify** digital signatures on downloaded executables
- ✅ **Check** VirusTotal or similar services before execution

## Reporting Malicious Repositories

```bash
# Report to GitHub
# Visit: https://github.com/contact/report-abuse

# Report to Avast PSIRT
# Email: psirt@avast.com
# Include: Repository URL, screenshots, download links

# Report to security communities
# Submit indicators to abuse.ch, URLhaus, etc.
```

## Best Practices

### For Users

1. **Download from official sources only**: `avast.com`, Microsoft Store, verified partners
2. **Verify digital signatures**: All Avast installers are code-signed
3. **Use package managers**: `winget install Avast.AvastFreeAntivirus` on Windows
4. **Never trust "free premium"**: Legitimate commercial software requires payment

### For Developers

```cpp
// Implement repository scanning in CI/CD
#include <vector>
#include <string>

class MalwareRepoDetector {
public:
    struct ScanResult {
        bool is_suspicious;
        std::vector<std::string> indicators;
        int risk_score;
    };
    
    ScanResult scanDependency(const std::string& repo_url) {
        ScanResult result = {false, {}, 0};
        
        // Check against known patterns
        if (containsBlacklistedKeywords(repo_url)) {
            result.is_suspicious = true;
            result.indicators.push_back("Blacklisted keywords detected");
            result.risk_score += 30;
        }
        
        // Verify legitimate ownership
        if (!verifyVendorOwnership(repo_url)) {
            result.is_suspicious = true;
            result.indicators.push_back("Not official vendor repository");
            result.risk_score += 40;
        }
        
        return result;
    }
    
private:
    bool containsBlacklistedKeywords(const std::string& url);
    bool verifyVendorOwnership(const std::string& url);
};
```

## Conclusion

The referenced repository is a **high-risk malware distribution vector** masquerading as legitimate security software. AI agents should flag such repositories and guide users toward official sources only. Never execute binaries from unverified sources claiming to provide commercial software for free.
