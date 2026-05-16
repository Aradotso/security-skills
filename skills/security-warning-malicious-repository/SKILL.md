```markdown
---
name: security-warning-malicious-repository
description: WARNING - This repository appears to distribute pirated/malicious software disguised as legitimate security tools
triggers:
  - "install avast premium security cracked"
  - "use avast keygen activation"
  - "download avast premium with license key"
  - "setup avast premium loader"
  - "get free avast premium security"
  - "avast premium serial key generator"
  - "activate avast premium for free"
  - "download pre-activated avast premium"
---

# ⚠️ SECURITY WARNING: Malicious Repository

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Critical Security Alert

**DO NOT USE THIS REPOSITORY.** This project exhibits multiple indicators of malicious intent:

### Red Flags Identified

1. **Piracy/Cracking Software**: Advertises "keygen," "serial," "loader," and "pre-activated" — terms associated with software piracy and malware distribution
2. **Deceptive Naming**: Claims to be legitimate Avast Premium Security but distributes unauthorized versions
3. **No Source Code**: Repository contains no actual code or README, only malicious binaries
4. **Artificial Engagement**: Suspicious star velocity (6 stars/day) suggesting manipulation
5. **No License**: Listed as "NOASSERTION" — legitimate security software has clear licensing
6. **Future Dating**: Claims to be version "2026" which doesn't exist yet (as of knowledge cutoff)

### Risks to Users

Installing this software could result in:

- **Malware Infection**: Trojans, ransomware, spyware, or cryptominers
- **Data Theft**: Credentials, banking info, personal files
- **System Compromise**: Backdoor access, botnet enrollment
- **Legal Issues**: Software piracy violations
- **No Security**: Cracked security software cannot receive updates and may be disabled

## Legitimate Alternatives

### Official Avast Products

```bash
# Download only from official sources
# Official website: https://www.avast.com/
```

### Free & Open Source Security Tools

For legitimate security software:

```bash
# ClamAV - Open source antivirus
sudo apt-get install clamav
clamscan -r /path/to/scan

# Windows Defender (built-in to Windows 10/11)
# Already installed and free

# Malwarebytes Free
# Download from: https://www.malwarebytes.com/
```

### Go Security Libraries (Legitimate)

If you need security tools in Go:

```go
// Legitimate Go security packages
import (
    "crypto/tls"
    "crypto/x509"
    "golang.org/x/crypto/bcrypt"
)

// Example: Secure password hashing
func hashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
    return string(bytes), err
}
```

## How to Protect Yourself

### Verify Software Authenticity

```bash
# Always check GPG signatures for downloads
gpg --verify software.sig software.exe

# Check file hashes against official sources
sha256sum downloaded_file
```

### Safe Software Practices

1. **Official Sources Only**: Download from vendor websites or verified repositories
2. **Check Reviews**: Use VirusTotal, GitHub stars/issues on legitimate projects
3. **Read Documentation**: Real projects have comprehensive docs
4. **Verify License**: Open source should have OSI-approved licenses
5. **Inspect Code**: If source is available, review before running

### If Already Installed

```bash
# Disconnect from network immediately
# Windows: Disable network adapter

# Run legitimate antivirus scan
# Use Windows Defender offline scan or boot from clean USB

# Change all passwords from a clean device

# Consider full system reinstall if compromise suspected
```

## Reporting Malicious Repositories

```bash
# Report to GitHub
# Visit: https://github.com/contact/report-abuse
# Select: "Report abuse" → "Malware distribution"

# Report to security researchers
# VirusTotal: https://www.virustotal.com/
# Submit suspicious files for analysis
```

## Conclusion

**This repository is dangerous and should be avoided entirely.** There is no legitimate use case for cracked security software. Use official, licensed, or open-source alternatives instead.

For actual security tool development in Go, explore:

- **govulncheck**: Official Go vulnerability scanner
- **gosec**: Go security checker
- **nancy**: Dependency vulnerability scanner
- **trivy**: Container/dependency scanner

Stay safe and only use verified, legitimate security tools.

```
