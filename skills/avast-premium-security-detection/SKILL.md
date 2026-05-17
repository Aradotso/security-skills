---
name: avast-premium-security-detection
description: Identify and analyze potentially malicious software distribution repositories masquerading as legitimate security software
triggers:
  - detect fake antivirus repositories
  - analyze suspicious software distribution
  - identify malware disguised as security tools
  - check for pirated software keygen patterns
  - validate legitimate security software sources
  - investigate cracked software repositories
  - scan for trojan distribution channels
  - verify antivirus software authenticity
---

# Avast Premium Security Detection

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ Critical Security Warning

This repository appears to be a **malicious software distribution channel** disguising itself as legitimate Avast Premium Security software. Key indicators:

- **Unauthorized distribution** of commercial security software
- **Keygen/crack terminology** in description ("Setup Keygen Activation", "Pre-Activated", "Premium Loader Serial")
- **No official affiliation** with Avast Software
- **Suspicious growth pattern** (68 stars, 6 stars/day - likely artificial)
- **No legitimate README** or documentation
- **C++ language tag** misleading (actual payload unknown)

## What This Skill Does

This skill helps security researchers and AI agents identify, analyze, and understand repositories that distribute malware disguised as cracked/pirated commercial software. Such repositories typically:

1. Promise "free" versions of paid software
2. Include keygens, cracks, loaders, or activators
3. Contain malware (trojans, ransomware, spyware, cryptominers)
4. Use SEO-optimized descriptions with emojis and keywords
5. Lack legitimate source code or documentation

## Detection Patterns

### Repository Red Flags

```cpp
// Pattern analysis for malicious software repos
#include <string>
#include <vector>
#include <regex>

struct SuspiciousIndicator {
    std::string pattern;
    std::string severity;  // HIGH, CRITICAL
    std::string description;
};

std::vector<SuspiciousIndicator> detectMaliciousPatterns(
    const std::string& repoDescription,
    const std::string& repoName
) {
    std::vector<SuspiciousIndicator> indicators;
    
    // Check for crack/keygen keywords
    std::vector<std::string> crackKeywords = {
        "keygen", "crack", "pre-activated", "loader", 
        "serial", "license key", "full version", "activat"
    };
    
    for (const auto& keyword : crackKeywords) {
        std::regex pattern(keyword, std::regex::icase);
        if (std::regex_search(repoDescription, pattern)) {
            indicators.push_back({
                keyword,
                "CRITICAL",
                "Crack/keygen terminology detected"
            });
        }
    }
    
    // Check for commercial software being offered "free"
    std::vector<std::string> commercialSoftware = {
        "premium", "pro version", "paid software"
    };
    
    // Check for absence of legitimate code
    if (repoDescription.find("README") == std::string::npos) {
        indicators.push_back({
            "NO_README",
            "HIGH",
            "No legitimate documentation present"
        });
    }
    
    return indicators;
}
```

### Metadata Analysis

```cpp
#include <json/json.h>
#include <iostream>

class RepoSecurityAnalyzer {
public:
    struct AnalysisResult {
        bool isSuspicious;
        std::vector<std::string> warnings;
        int riskScore;  // 0-100
    };
    
    AnalysisResult analyzeRepository(const Json::Value& metadata) {
        AnalysisResult result{false, {}, 0};
        
        // Check license
        std::string license = metadata.get("license", "").asString();
        if (license == "NOASSERTION" || license.empty()) {
            result.warnings.push_back("No valid license specified");
            result.riskScore += 20;
        }
        
        // Analyze star growth rate
        int stars = metadata.get("stars", 0).asInt();
        float starsPerDay = metadata.get("stars_per_day", 0.0).asFloat();
        
        if (starsPerDay > 3.0 && stars < 200) {
            result.warnings.push_back("Suspicious star growth pattern");
            result.riskScore += 30;
        }
        
        // Check for zero forks (common in malware repos)
        int forks = metadata.get("forks", 0).asInt();
        if (forks == 0 && stars > 50) {
            result.warnings.push_back("High stars but zero forks - artificial inflation");
            result.riskScore += 25;
        }
        
        // Check topics for manipulation
        Json::Value topics = metadata.get("topics", Json::arrayValue);
        if (topics.size() > 5) {
            result.warnings.push_back("Excessive SEO-optimized topics");
            result.riskScore += 15;
        }
        
        result.isSuspicious = result.riskScore >= 50;
        return result;
    }
};
```

## Safety Guidelines for Users

### Never Download From This Type of Repository

```cpp
// Example: Safe software verification workflow
#include <openssl/sha.h>
#include <fstream>

class SoftwareValidator {
public:
    // Always verify against official sources
    bool verifyOfficialSource(const std::string& downloadUrl) {
        std::vector<std::string> officialDomains = {
            "avast.com",
            "avg.com"  // Avast's official domains
        };
        
        for (const auto& domain : officialDomains) {
            if (downloadUrl.find(domain) != std::string::npos) {
                return true;
            }
        }
        
        std::cerr << "WARNING: Not from official source!" << std::endl;
        return false;
    }
    
    // Verify digital signature
    bool verifyDigitalSignature(const std::string& filepath) {
        // Use OS-specific signature verification
        // Windows: Authenticode
        // macOS: codesign
        // Linux: gpg signatures
        
        std::cout << "Verifying digital signature for: " << filepath << std::endl;
        
        // Environment variable for trusted certificate
        const char* trustedCert = std::getenv("TRUSTED_CERT_PATH");
        if (!trustedCert) {
            std::cerr << "No trusted certificate configured" << std::endl;
            return false;
        }
        
        // Actual signature verification would go here
        return false; // Default deny
    }
};
```

## Reporting Malicious Repositories

### Automated Reporting

```cpp
#include <curl/curl.h>
#include <string>

class MalwareReporter {
private:
    std::string platformApiEndpoint;
    
public:
    struct ReportData {
        std::string repoUrl;
        std::string repoOwner;
        std::string repoName;
        std::vector<std::string> indicators;
        std::string reporterEmail;
    };
    
    bool reportToGitHub(const ReportData& data) {
        // Report to GitHub's abuse team
        CURL* curl = curl_easy_init();
        if (!curl) return false;
        
        std::string apiToken = std::getenv("GITHUB_TOKEN") 
            ? std::getenv("GITHUB_TOKEN") : "";
        
        std::string reportUrl = "https://github.com/contact/report-abuse";
        
        // Construct abuse report
        std::string reportBody = 
            "Repository: " + data.repoUrl + "\n" +
            "Reason: Distribution of malware disguised as cracked software\n" +
            "Evidence:\n";
            
        for (const auto& indicator : data.indicators) {
            reportBody += "- " + indicator + "\n";
        }
        
        std::cout << "Report generated:\n" << reportBody << std::endl;
        
        curl_easy_cleanup(curl);
        return true;
    }
    
    bool reportToAVVendor(const ReportData& data) {
        // Report to Avast's abuse team
        std::cout << "Contact Avast abuse team: https://www.avast.com/report-malicious-file" << std::endl;
        std::cout << "Unauthorized distribution of: " << data.repoUrl << std::endl;
        return true;
    }
};
```

## Legitimate Alternatives

### How to Get Real Avast Premium Security

```cpp
// Configuration for legitimate sources only
struct LegitimateSource {
    std::string vendor = "Avast Software";
    std::string officialWebsite = "https://www.avast.com";
    std::string downloadPage = "https://www.avast.com/en-us/premium-security";
    bool requiresLicense = true;
    bool hasFreeTrial = true;
    
    void printInstructions() {
        std::cout << "=== LEGITIMATE AVAST PREMIUM SECURITY ===" << std::endl;
        std::cout << "Official website: " << officialWebsite << std::endl;
        std::cout << "Download page: " << downloadPage << std::endl;
        std::cout << "Free trial available: " << (hasFreeTrial ? "Yes" : "No") << std::endl;
        std::cout << "\nWARNING: Never download from third-party sites!" << std::endl;
    }
};
```

## Threat Intelligence Integration

```cpp
#include <set>
#include <memory>

class ThreatIntelligence {
public:
    // Maintain known malicious repository patterns
    std::set<std::string> knownMaliciousPatterns = {
        "keygen",
        "crack",
        "pre-activated",
        "loader",
        "full-version-free"
    };
    
    // Check against VirusTotal (if API key available)
    bool checkVirusTotal(const std::string& repoUrl) {
        const char* vtApiKey = std::getenv("VIRUSTOTAL_API_KEY");
        if (!vtApiKey) {
            std::cerr << "VIRUSTOTAL_API_KEY not set" << std::endl;
            return false;
        }
        
        std::cout << "Checking " << repoUrl << " against VirusTotal..." << std::endl;
        // Implementation would use VirusTotal API
        return false;
    }
};
```

## Conclusion

This "skill" is actually a **security warning system**. The repository in question is highly suspicious and likely contains malware. Users should:

1. **Never download** executables from such repositories
2. **Report** to GitHub and the legitimate software vendor
3. **Obtain software** only from official sources
4. **Use antivirus** from legitimate vendors to scan systems
5. **Educate others** about these distribution methods

For legitimate Avast Premium Security, visit **https://www.avast.com** directly.
