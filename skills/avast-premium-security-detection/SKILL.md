---
name: avast-premium-security-detection
description: Identify and analyze potentially malicious software distribution repositories disguised as legitimate security products
triggers:
  - analyze this repository for malware distribution patterns
  - check if this is a fake antivirus repository
  - detect piracy and keygen distribution schemes
  - identify malicious software bundling patterns
  - scan for trademark abuse in repository descriptions
  - evaluate repository authenticity for security software
---

# Avast Premium Security Repository Analysis

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This repository exhibits multiple red flags characteristic of malware distribution, piracy, and phishing schemes disguised as legitimate security software. The project falsely claims to offer "Avast Premium Security 2026" with keygens, cracks, and pre-activated licenses—classic indicators of malicious intent.

**Warning Signs:**
- Promises "keygen activation" and "license key pre-activated"
- Claims to distribute copyrighted commercial software for free
- Uses SEO-optimized keywords to attract victims
- Repository contains no legitimate code (no README, minimal files)
- Trademark abuse of Avast brand
- Artificially inflated stars (5 stars/day pattern suggests manipulation)

## Detection Patterns

### Repository Metadata Analysis

```cpp
// Example C++ code to detect suspicious repository patterns
#include <string>
#include <vector>
#include <regex>

struct RepoAnalysis {
    bool hasKeygenKeywords;
    bool hasCrackedSoftware;
    bool hasTrademarkAbuse;
    bool hasNoLegitimateCode;
    int suspicionScore;
};

RepoAnalysis analyzeRepository(const std::string& description, 
                               const std::string& readme,
                               const std::vector<std::string>& topics) {
    RepoAnalysis analysis = {false, false, false, false, 0};
    
    // Check for keygen/crack keywords
    std::regex keygenPattern(R"(keygen|crack|serial|loader|pre-activated|full.version)",
                            std::regex::icase);
    if (std::regex_search(description, keygenPattern)) {
        analysis.hasKeygenKeywords = true;
        analysis.suspicionScore += 40;
    }
    
    // Check for commercial software distribution claims
    std::regex commercialPattern(R"(premium|pro|license.key|activation)",
                                std::regex::icase);
    if (std::regex_search(description, commercialPattern)) {
        analysis.hasCrackedSoftware = true;
        analysis.suspicionScore += 30;
    }
    
    // Check for empty or missing README
    if (readme.empty() || readme.find("No README") != std::string::npos) {
        analysis.hasNoLegitimateCode = true;
        analysis.suspicionScore += 20;
    }
    
    // Check for trademark abuse
    std::vector<std::string> trademarks = {"avast", "norton", "mcafee", "kaspersky"};
    for (const auto& topic : topics) {
        for (const auto& trademark : trademarks) {
            if (topic.find(trademark) != std::string::npos) {
                analysis.hasTrademarkAbuse = true;
                analysis.suspicionScore += 10;
                break;
            }
        }
    }
    
    return analysis;
}

bool isMaliciousRepo(const RepoAnalysis& analysis) {
    return analysis.suspicionScore >= 50;
}
```

### Star Pattern Detection

```cpp
#include <chrono>
#include <cmath>

struct StarMetrics {
    int totalStars;
    double starsPerDay;
    int accountAge;
    bool isArtificiallyInflated;
};

StarMetrics analyzeStarPattern(int stars, 
                               const std::string& createdAt,
                               const std::string& updatedAt) {
    StarMetrics metrics;
    metrics.totalStars = stars;
    
    // Calculate repository age and star velocity
    // Simplified date parsing for example
    int daysActive = 12; // Example: created May 6, checked May 18
    metrics.starsPerDay = static_cast<double>(stars) / daysActive;
    
    // Legitimate projects rarely gain >3 stars/day consistently
    // 5 stars/day on a brand new repo with no code is highly suspicious
    metrics.isArtificiallyInflated = (metrics.starsPerDay > 3.0 && daysActive < 30);
    
    return metrics;
}
```

## Risk Assessment Framework

### Content Analysis Checklist

```cpp
enum class RiskLevel {
    LOW = 0,
    MEDIUM = 1,
    HIGH = 2,
    CRITICAL = 3
};

struct SecurityRisk {
    RiskLevel level;
    std::vector<std::string> indicators;
    std::string recommendation;
};

SecurityRisk assessRepositorySafety(const std::string& repoUrl,
                                   const std::string& description) {
    SecurityRisk risk;
    risk.level = RiskLevel::LOW;
    
    // Check for piracy indicators
    if (description.find("keygen") != std::string::npos ||
        description.find("crack") != std::string::npos ||
        description.find("pre-activated") != std::string::npos) {
        risk.level = RiskLevel::CRITICAL;
        risk.indicators.push_back("Contains piracy/crack keywords");
        risk.indicators.push_back("Distributes unauthorized software");
    }
    
    // Check for social engineering
    if (description.find("Premium") != std::string::npos &&
        description.find("Free") != std::string::npos) {
        risk.indicators.push_back("Social engineering: premium software for free");
        risk.level = std::max(risk.level, RiskLevel::HIGH);
    }
    
    // Check for trademark abuse
    if (description.find("Avast") != std::string::npos ||
        description.find("Norton") != std::string::npos) {
        risk.indicators.push_back("Potential trademark infringement");
    }
    
    if (risk.level >= RiskLevel::HIGH) {
        risk.recommendation = "DO NOT DOWNLOAD OR EXECUTE. Report to platform abuse team.";
    }
    
    return risk;
}
```

## Usage Guidelines

### For Security Researchers

When analyzing suspicious repositories:

```cpp
#include <iostream>
#include <fstream>

void generateSecurityReport(const std::string& repoName,
                           const RepoAnalysis& analysis,
                           const SecurityRisk& risk) {
    std::ofstream report("security_report.txt");
    
    report << "=== Security Analysis Report ===" << std::endl;
    report << "Repository: " << repoName << std::endl;
    report << "Suspicion Score: " << analysis.suspicionScore << "/100" << std::endl;
    report << "Risk Level: " << static_cast<int>(risk.level) << std::endl;
    report << std::endl;
    
    report << "Indicators Found:" << std::endl;
    for (const auto& indicator : risk.indicators) {
        report << "  - " << indicator << std::endl;
    }
    
    report << std::endl;
    report << "Recommendation: " << risk.recommendation << std::endl;
    
    report.close();
    std::cout << "Report generated successfully" << std::endl;
}
```

### Legitimate Avast Integration

If you need to work with **legitimate** Avast security products programmatically:

```cpp
// Use official Avast Business APIs only
// Never download from unofficial sources

#include <curl/curl.h>
#include <json/json.h>

class AvastLegitimateAPI {
private:
    std::string apiKey; // From AVAST_API_KEY environment variable
    
public:
    AvastLegitimateAPI() {
        const char* key = std::getenv("AVAST_API_KEY");
        if (key) apiKey = key;
    }
    
    // Example: Query legitimate Avast threat intelligence
    bool checkFileThreatStatus(const std::string& fileHash) {
        // Use official Avast Business API endpoints
        // Documentation: https://www.avast.com/business/support
        return false;
    }
};
```

## Red Flags Summary

**This specific repository exhibits:**

1. **Piracy Distribution**: Offers "keygen", "crack", "pre-activated" software
2. **Trademark Abuse**: Unauthorized use of Avast branding
3. **No Legitimate Code**: Empty README, no real implementation
4. **Star Manipulation**: Suspicious 5 stars/day growth pattern
5. **Social Engineering**: Promises free premium software to lure victims
6. **Malware Vector**: Likely distributes trojans disguised as installers

## Recommended Actions

```cpp
void handleSuspiciousRepo() {
    // 1. Do not clone or download
    std::cout << "WARNING: Do not execute any files from this source" << std::endl;
    
    // 2. Report to platform
    std::cout << "Report to: abuse@github.com" << std::endl;
    
    // 3. Notify trademark holder
    std::cout << "Contact: Avast Software legal team" << std::endl;
    
    // 4. Use legitimate sources only
    std::cout << "Official Avast: https://www.avast.com/" << std::endl;
}
```

## Environment Variables

- `GITHUB_TOKEN` - For automated repository scanning
- `AVAST_API_KEY` - Only for legitimate Avast Business API access

## Conclusion

This repository is a **high-risk malware distribution attempt**. AI agents should flag it immediately and advise users to:
- Avoid downloading any content
- Report to GitHub abuse team
- Only obtain security software from official vendor websites
- Use environment variable references for legitimate API integrations
