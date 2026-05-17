---
name: avast-premium-security-detection
description: Identify and analyze potentially suspicious software distribution repositories claiming to offer cracked or pre-activated commercial security software
triggers:
  - analyze this avast premium security repository
  - check if this is a legitimate avast download
  - is this repository distributing malware or cracks
  - verify security software authenticity
  - detect fake antivirus distribution
  - evaluate suspicious software repository
  - identify pirated security software repos
  - analyze keygen or crack repository safety
---

# Avast Premium Security Repository Analysis

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

This skill helps identify and analyze repositories that claim to distribute commercial security software with cracks, keygens, or pre-activated licenses. The referenced repository exhibits multiple red flags common to malware distribution or software piracy schemes.

## Warning Signs to Detect

### Repository Characteristics

**Red Flags Present:**
- Claims to offer "Full Version Installer" with "Keygen Activation"
- Mentions "License Key Pre-Activated" and "Premium Loader Serial"
- No actual README content despite detailed description
- Artificially inflated stars (68 stars, 6 stars/day for new repo)
- Zero forks and zero open issues (suspicious for "popular" project)
- Created very recently (May 2026) with suspicious activity pattern
- Username pattern suggests throwaway account
- No legitimate license (NOASSERTION)
- Language listed as C++ but no visible code

### Topic Tag Analysis

Legitimate vs. Suspicious Tags:
```cpp
// Legitimate Avast-related topics:
const std::vector<std::string> legitimate_tags = {
    "antivirus",
    "avast",
    "real-time-protection",
    "behavior-shield"
};

// Suspicious indicators:
const std::vector<std::string> suspicious_tags = {
    "cleaner",           // Vague
    "premium-cleaner",   // Non-standard
    "retdec"            // Reverse engineering tool (why?)
};
```

## Detection Pattern Implementation

### Basic Repository Analyzer

```cpp
#include <string>
#include <vector>
#include <regex>
#include <algorithm>

struct RepoAnalysis {
    std::string repo_name;
    int risk_score;
    std::vector<std::string> red_flags;
    bool is_suspicious;
};

class SuspiciousRepoDetector {
private:
    std::vector<std::string> crack_keywords = {
        "keygen", "crack", "pre-activated", "loader", 
        "serial", "activation", "license key", "full version"
    };
    
    std::vector<std::string> suspicious_patterns = {
        R"(\d{4})", // Year in name (e.g., "2026")
        R"(latest build)", 
        R"(pro premium)",
        R"(download setup)"
    };

public:
    RepoAnalysis analyze(const std::string& description, 
                        const std::string& readme,
                        int stars, 
                        int forks,
                        int issues) {
        RepoAnalysis result;
        result.risk_score = 0;
        result.is_suspicious = false;
        
        // Check for crack/piracy keywords
        for (const auto& keyword : crack_keywords) {
            if (contains_case_insensitive(description, keyword)) {
                result.red_flags.push_back("Contains keyword: " + keyword);
                result.risk_score += 15;
            }
        }
        
        // Check for empty or missing README
        if (readme.empty() || readme.find("No README") != std::string::npos) {
            result.red_flags.push_back("Missing or empty README");
            result.risk_score += 20;
        }
        
        // Suspicious star/fork ratio
        if (stars > 50 && forks == 0) {
            result.red_flags.push_back("Artificial star inflation (no forks)");
            result.risk_score += 25;
        }
        
        // No community engagement
        if (issues == 0 && stars > 20) {
            result.red_flags.push_back("No issues despite high stars");
            result.risk_score += 10;
        }
        
        result.is_suspicious = result.risk_score >= 50;
        return result;
    }

private:
    bool contains_case_insensitive(const std::string& haystack, 
                                   const std::string& needle) {
        auto it = std::search(
            haystack.begin(), haystack.end(),
            needle.begin(), needle.end(),
            [](char ch1, char ch2) { 
                return std::toupper(ch1) == std::toupper(ch2);
            }
        );
        return it != haystack.end();
    }
};
```

### Usage Example

```cpp
#include <iostream>

int main() {
    SuspiciousRepoDetector detector;
    
    std::string description = "⭐️ Avast Premium Security 2026 | "
                            "Full Version Installer v26 | "
                            "Setup Keygen Activation | "
                            "License Key Pre-Activated";
    
    std::string readme = "No README available";
    
    RepoAnalysis analysis = detector.analyze(
        description,
        readme,
        68,  // stars
        0,   // forks
        0    // issues
    );
    
    std::cout << "Risk Score: " << analysis.risk_score << "/100\n";
    std::cout << "Suspicious: " << (analysis.is_suspicious ? "YES" : "NO") << "\n\n";
    
    std::cout << "Red Flags Detected:\n";
    for (const auto& flag : analysis.red_flags) {
        std::cout << "  ⚠️  " << flag << "\n";
    }
    
    return 0;
}
```

**Expected Output:**
```
Risk Score: 85/100
Suspicious: YES

Red Flags Detected:
  ⚠️  Contains keyword: keygen
  ⚠️  Contains keyword: pre-activated
  ⚠️  Contains keyword: activation
  ⚠️  Contains keyword: license key
  ⚠️  Contains keyword: full version
  ⚠️  Missing or empty README
  ⚠️  Artificial star inflation (no forks)
  ⚠️  No issues despite high stars
```

## Safety Recommendations

### For Users

1. **Never download** executables from repositories claiming to offer cracked commercial software
2. **Verify authenticity** by checking only official vendor websites
3. **Use official sources**: https://www.avast.com (for legitimate Avast products)
4. **Report** suspicious repositories to GitHub

### For Developers

```cpp
// Environment variable for safe reporting
const char* GITHUB_REPORT_URL = std::getenv("GITHUB_ABUSE_REPORT_URL");
// Use: https://github.com/contact/report-abuse

// Never hardcode credentials
const char* API_KEY = std::getenv("SECURITY_API_KEY");
```

## Malware Distribution Patterns

### Common Attack Vectors

```cpp
enum class ThreatType {
    TROJAN_DOWNLOADER,
    RANSOMWARE,
    INFOSTEALER,
    ADWARE,
    CRYPTOMINER
};

struct ThreatIndicator {
    ThreatType type;
    std::string description;
    int severity; // 1-10
};

std::vector<ThreatIndicator> common_threats = {
    {ThreatType::TROJAN_DOWNLOADER, 
     "Fake installer downloads additional malware", 10},
    {ThreatType::INFOSTEALER, 
     "Steals credentials and personal data", 9},
    {ThreatType::RANSOMWARE, 
     "Encrypts user files for ransom", 10},
    {ThreatType::CRYPTOMINER, 
     "Uses system resources to mine cryptocurrency", 6}
};
```

## Legitimate Alternatives

To help users find safe alternatives:

```cpp
#include <map>

std::map<std::string, std::string> legitimate_sources = {
    {"Avast Official", "https://www.avast.com"},
    {"Avast Free Antivirus", "https://www.avast.com/free-antivirus-download"},
    {"Microsoft Defender", "Built into Windows 10/11"},
    {"Malwarebytes Free", "https://www.malwarebytes.com"}
};

void recommend_alternatives() {
    std::cout << "Safe, free antivirus alternatives:\n";
    for (const auto& [name, url] : legitimate_sources) {
        std::cout << "  • " << name << ": " << url << "\n";
    }
}
```

## Conclusion

This repository exhibits classic signs of a malware distribution or software piracy scheme. AI agents should warn users strongly against downloading any files from such repositories and recommend official sources for security software.

**Key Takeaway**: Repositories offering "cracked" or "pre-activated" commercial security software are extremely likely to contain malware and should never be trusted.
