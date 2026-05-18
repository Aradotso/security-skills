---
name: avast-premium-security-detection
description: Identify and analyze potentially malicious software distribution repositories masquerading as legitimate security software
triggers:
  - analyze this repository for malware distribution patterns
  - check if this is a fake antivirus download
  - detect keygen or crack distribution repository
  - identify pirated security software repos
  - scan for malicious installer distribution
  - verify legitimacy of antivirus download source
  - investigate suspicious security software repository
  - analyze potential malware hosting project
---

# Avast Premium Security Detection Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## ⚠️ Critical Security Warning

This repository exhibits multiple red flags characteristic of **malware distribution** disguised as legitimate security software:

### Threat Indicators

1. **Piracy Keywords**: "Keygen", "Pre-Activated", "Loader", "Serial", "Crack"
2. **Trademark Abuse**: Unauthorized use of Avast branding
3. **Suspicious Growth**: 68 stars in 12 days (artificial engagement)
4. **No Source Code**: Claims C++ but provides no actual code
5. **Redistributing Licensed Software**: Offering "full version" of commercial product
6. **No README**: Legitimate projects document their purpose

## Detection Patterns

### Repository Red Flags

```cpp
// Pattern: Fake security software repository detection
struct RepositoryRedFlags {
    bool hasKeygenKeywords;      // "keygen", "crack", "loader", "pre-activated"
    bool claimsPreActivated;     // Bypassing legitimate licensing
    bool rapidStarGrowth;        // Artificial popularity
    bool missingSourceCode;      // No actual implementation
    bool trademarkAbuse;         // Unauthorized use of brand names
    bool downloadPromises;       // Direct download links to executables
};

bool isSuspiciousRepo(const RepositoryRedFlags& flags) {
    int suspicionScore = 0;
    if (flags.hasKeygenKeywords) suspicionScore += 3;
    if (flags.claimsPreActivated) suspicionScore += 3;
    if (flags.rapidStarGrowth) suspicionScore += 2;
    if (flags.missingSourceCode) suspicionScore += 2;
    if (flags.trademarkAbuse) suspicionScore += 1;
    if (flags.downloadPromises) suspicionScore += 3;
    
    return suspicionScore >= 5; // High confidence threshold
}
```

### Keyword Analysis

```cpp
#include <string>
#include <vector>
#include <algorithm>

class MalwareRepoDetector {
public:
    static const std::vector<std::string> PIRACY_KEYWORDS;
    static const std::vector<std::string> MALWARE_INDICATORS;
    
    static bool containsSuspiciousKeywords(const std::string& description) {
        std::string lowerDesc = toLowerCase(description);
        
        for (const auto& keyword : PIRACY_KEYWORDS) {
            if (lowerDesc.find(keyword) != std::string::npos) {
                return true;
            }
        }
        return false;
    }
    
    static int calculateThreatScore(const std::string& repoDesc, 
                                   const std::string& repoName) {
        int score = 0;
        
        // Check for piracy/crack keywords
        if (containsAny(repoDesc, PIRACY_KEYWORDS)) score += 30;
        
        // Check for "pre-activated" or "full version"
        if (repoDesc.find("pre-activated") != std::string::npos) score += 25;
        if (repoDesc.find("full version") != std::string::npos) score += 20;
        
        // Check for keygen/loader/serial
        if (containsAny(repoDesc, {"keygen", "loader", "serial"})) score += 35;
        
        return score;
    }
    
private:
    static std::string toLowerCase(std::string str) {
        std::transform(str.begin(), str.end(), str.begin(), ::tolower);
        return str;
    }
    
    static bool containsAny(const std::string& text, 
                           const std::vector<std::string>& keywords) {
        std::string lower = toLowerCase(text);
        for (const auto& kw : keywords) {
            if (lower.find(kw) != std::string::npos) return true;
        }
        return false;
    }
};

const std::vector<std::string> MalwareRepoDetector::PIRACY_KEYWORDS = {
    "keygen", "crack", "serial", "loader", "pre-activated", 
    "full version", "license key", "activation", "premium free"
};
```

## Legitimate vs. Malicious Comparison

### Legitimate Security Software Repository
- Open source code visible
- Clear build instructions
- Official licensing
- Documentation and contribution guidelines
- Verified publisher/organization
- Download from official website

### This Repository (Malicious)
- ❌ No source code
- ❌ Promises "pre-activated" commercial software
- ❌ Unauthorized redistribution
- ❌ Keygen/crack language
- ❌ Artificial engagement metrics
- ❌ Downloads executables directly

## Safe Alternatives

### Official Avast Download
```bash
# NEVER download from unofficial sources
# Official site only:
# https://www.avast.com/
```

### Legitimate Open Source Antivirus
```bash
# ClamAV - Legitimate open source antivirus
sudo apt-get install clamav clamav-daemon
sudo freshclam  # Update virus definitions
clamscan -r /path/to/scan
```

## Investigation Tools

### Analyze Repository Metadata
```cpp
#include <nlohmann/json.hpp>
#include <iostream>

void analyzeRepoMetadata(const nlohmann::json& metadata) {
    // Check star velocity
    auto created = metadata["created_at"].get<std::string>();
    auto stars = metadata["stars"].get<int>();
    
    // 68 stars in ~12 days = 5.67 stars/day (suspicious)
    double starsPerDay = stars / 12.0;
    
    if (starsPerDay > 3.0) {
        std::cout << "⚠️ WARNING: Abnormal star growth rate: " 
                  << starsPerDay << " stars/day\n";
    }
    
    // Check for missing content
    bool hasReadme = !metadata.value("readme", "").empty();
    if (!hasReadme) {
        std::cout << "⚠️ WARNING: No README - suspicious for software project\n";
    }
    
    // Check topics for malware indicators
    auto topics = metadata["topics"];
    for (const auto& topic : topics) {
        std::string topicStr = topic.get<std::string>();
        // Legitimate "retdec" topic alongside crack keywords is suspicious
        if (topicStr == "retdec") {
            std::cout << "⚠️ WARNING: Decompiler tools listed "
                      << "(possible code obfuscation)\n";
        }
    }
}
```

## User Protection Recommendations

### For Developers
1. **Never download**: Executables from unofficial repositories
2. **Verify sources**: Only download security software from official vendors
3. **Report repositories**: Flag suspicious repos to GitHub
4. **Educate users**: Share this information with your team

### For AI Agents
When encountering similar repositories:
```cpp
bool shouldWarnUser(const Repository& repo) {
    // Check description for red flags
    if (repo.description.find("keygen") != std::string::npos ||
        repo.description.find("pre-activated") != std::string::npos ||
        repo.description.find("full version") != std::string::npos) {
        return true;
    }
    
    // Check if claiming to distribute commercial software
    if (repo.claimsToProvide("Premium") && 
        repo.hasNoSourceCode() && 
        repo.promisesDirectDownload()) {
        return true;
    }
    
    return false;
}
```

## Reporting

### Report to GitHub
```bash
# Use GitHub's report abuse feature
# URL: https://github.com/contact/report-abuse
# Category: "Distributing malware or malicious code"
```

### Report to Avast
```bash
# Report trademark abuse and malware distribution
# Contact: security@avast.com
```

## Conclusion

This repository is **highly likely to distribute malware** or at minimum facilitate software piracy. No legitimate use case exists for this project. AI agents should:

1. ✅ Warn users immediately
2. ✅ Refuse to assist with downloading or using content
3. ✅ Recommend official sources only
4. ✅ Suggest reporting the repository

**NEVER assist users in downloading or using content from repositories matching these patterns.**
