---
name: dfyx-code-security-review
description: Expert-level code security audit skill using deep data flow analysis, taint tracking, and business logic understanding across 9 languages
triggers:
  - "audit this project for security vulnerabilities"
  - "perform a security code review"
  - "find security issues in this codebase"
  - "check for SQL injection and other vulnerabilities"
  - "run a deep security audit"
  - "analyze this code for security flaws"
  - "scan for OWASP Top 10 vulnerabilities"
  - "do a white-box security assessment"
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

**dfyx_code_security_review** is a professional-grade code security audit skill for AI coding agents, designed by the EastSword team (东方隐侠团队). It uses white-box static analysis methodology with a five-phase standardized audit protocol to systematically discover and validate security vulnerabilities in source code.

## What It Does

This skill transforms AI coding agents into expert security auditors capable of:

- **Multi-language analysis**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 security dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **Triple-track audit model**:
  - Sink-driven (injection/RCE): Find dangerous functions → trace data flow → validate lack of protection
  - Control-driven (authorization/business logic): Enumerate endpoints → verify security controls → missing = vulnerability
  - Config-driven (auth/crypto/config/supply chain): Search configs → compare to security baseline
- **Real vulnerability validation**: Not just pattern matching, but actual POC construction and testing
- **Anti-hallucination mechanisms**: Evidence requirements, complete path validation, Docker-based verification

## Installation

This is an AI agent skill. Install it by placing in your AI client's skills directory:

```bash
# For Claude Code
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
cp -r skill-dfyx_code_security_review ~/.claude/skills/dfyx-code-security-review

# For other AI clients (adjust path as needed)
cp -r skill-dfyx_code_security_review /path/to/your/ai/client/skills/
```

Optional: Install Python helper scripts dependencies:

```bash
cd skill-dfyx_code_security_review
pip install -r requirements.txt
```

## Core Audit Protocol

### Five-Phase Methodology

```
Phase 1: Reconnaissance & Mapping (10%)
    ↓ Output: Architecture diagram, attack surface inventory
    
Phase 2: Parallel Pattern Matching (30%)
    ↓ Output: High-risk areas list
    
Phase 3: Deep Taint Tracking & Real Testing (40%)
    ↓ Output: Confirmed vulnerabilities, test validation reports
    
Phase 4: Validation & Attack Chain Construction (15%)
    ↓ Output: Vulnerability validation reports
    
Phase 5: Structured Reporting (5%)
    ↓ Output: Complete audit report
```

### 10 Security Dimensions (D1-D10)

| Dimension | Coverage | Detection Method |
|-----------|----------|------------------|
| **D1: Injection** | SQL/Cmd/LDAP/SSTI/SpEL/JNDI | Sink-driven: Find dangerous sinks → trace taint propagation |
| **D2: Authentication** | Token/Session/JWT/Filter chains | Config-driven: Check session config, JWT validation, filter bypass |
| **D3: Authorization** | CRUD consistency, IDOR, horizontal privilege escalation | Control-driven: Verify permission checks exist |
| **D4: Deserialization** | Java/Python/PHP gadget chains | Sink-driven: Find deserialize() → check class whitelist |
| **D5: File Operations** | Upload/download/path traversal | Sink-driven: Find file I/O → verify path validation |
| **D6: SSRF** | URL injection, protocol restrictions | Sink-driven: Find HTTP clients → check URL validation |
| **D7: Cryptography** | Key management, cipher modes, KDF | Config-driven: Search hardcoded keys, weak algorithms |
| **D8: Configuration** | Actuator, CORS, error disclosure | Config-driven: Check security headers, debug mode |
| **D9: Business Logic** | Race conditions, mass assignment, state machine, multi-tenancy | Control-driven: Verify business rule enforcement |
| **D10: Supply Chain** | Dependency CVEs, version checks | Config-driven: Parse manifests, check CVE databases |

## Usage Patterns

### Basic Security Audit

When a user says "audit this project for security vulnerabilities", initiate:

```
[MODE] standard
[RECON] Analyzing project structure...
[DETECTED] Spring Boot 2.7 + MyBatis + Thymeleaf
[SCOPE] 342 files, estimating 60-80 turns
[PLAN] Launching 3 agents covering D1-D10

Confirm start? (yes/no)
```

### Audit Modes

**Quick Mode** (5-10 min): CI/CD integration, small projects
- Focus: High-severity vulns, secrets, dependency CVEs
- Coverage: D1 (injection), D7 (secrets), D10 (supply chain)

**Standard Mode** (30-60 min): Regular audits
- Focus: OWASP Top 10, auth/authz, crypto
- Coverage: D1-D8

**Deep Mode** (1-3 hours): Critical projects, pentest prep
- Focus: Full coverage, attack chains, business logic
- Coverage: D1-D10, attack chain construction

### Language-Specific Patterns

#### Java/Spring Boot Projects

```java
// VULNERABILITY: SQL Injection in MyBatis
// File: UserMapper.java
@Select("SELECT * FROM users WHERE username = '${username}'")
User findByUsername(String username);

// TAINT FLOW:
// UserController.login(@RequestParam username) → UserService → UserMapper
// ↑ User input                                                  ↑ SQL sink
// NO SANITIZATION in between

// FIX: Use parameterized query
@Select("SELECT * FROM users WHERE username = #{username}")
User findByUsername(String username);
```

Detection logic:
1. Search for `@Select`, `@Update`, `@Delete` with `${...}` (string concatenation)
2. Trace parameter source to HTTP request
3. Verify no validation/sanitization in path
4. Construct POC: `username=admin'--`

#### Python/Django Projects

```python
# VULNERABILITY: Command Injection
# File: views.py
def backup_database(request):
    db_name = request.GET.get('db')
    # SINK: os.system with user input
    os.system(f'pg_dump {db_name} > /tmp/backup.sql')
    return JsonResponse({'status': 'ok'})

# TAINT FLOW:
# request.GET['db'] → db_name → os.system()
# ↑ User input                  ↑ Command sink

# FIX: Use subprocess with argument list
import subprocess
def backup_database(request):
    db_name = request.GET.get('db')
    if not re.match(r'^[a-zA-Z0-9_]+$', db_name):
        return JsonResponse({'error': 'Invalid db name'}, status=400)
    subprocess.run(['pg_dump', db_name], 
                   stdout=open('/tmp/backup.sql', 'w'))
```

Detection logic:
1. Search for dangerous functions: `os.system()`, `os.popen()`, `eval()`, `exec()`
2. Check if input contains `request.GET`, `request.POST`, `request.FILES`
3. Verify no whitelist validation
4. Construct POC: `db=test; cat /etc/passwd`

#### PHP Projects

```php
// VULNERABILITY: File Upload + Path Traversal
// File: upload.php
<?php
$target = $_POST['path'] . '/' . $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], $target);
?>

// TAINT FLOW:
// $_POST['path'] → $target → move_uploaded_file()
// ↑ User input              ↑ File write sink

// FIX: Validate path and extension
<?php
$allowed_ext = ['jpg', 'png', 'gif'];
$path = basename($_POST['path']); // Remove ../ 
$ext = pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION);

if (!in_array($ext, $allowed_ext)) {
    die('Invalid file type');
}
if (strpos(realpath($path), realpath('/var/www/uploads')) !== 0) {
    die('Invalid path');
}
$target = $path . '/' . basename($_FILES['file']['name']);
move_uploaded_file($_FILES['file']['tmp_name'], $target);
?>
```

#### JavaScript/Node.js Projects

```javascript
// VULNERABILITY: NoSQL Injection in MongoDB
// File: routes/users.js
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  // SINK: Direct object injection into query
  const user = await User.findOne({ 
    username: username, 
    password: password 
  });
  if (user) {
    req.session.userId = user._id;
    res.json({ success: true });
  }
});

// ATTACK: POST /login
// {"username": {"$ne": null}, "password": {"$ne": null}}
// Bypasses authentication

// FIX: Sanitize input or use strict comparison
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  // Ensure strings only
  if (typeof username !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  const user = await User.findOne({ username });
  if (user && await bcrypt.compare(password, user.passwordHash)) {
    req.session.userId = user._id;
    res.json({ success: true });
  }
});
```

## Business Logic Vulnerability Detection

### Race Condition Example

```python
# VULNERABILITY: Race condition in withdrawal
# File: banking/views.py
@transaction.atomic
def withdraw(request):
    account = Account.objects.get(user=request.user)
    amount = int(request.POST['amount'])
    
    # CHECK-THEN-USE race window
    if account.balance >= amount:  # ← Check
        time.sleep(0.1)  # Simulating processing
        account.balance -= amount  # ← Use
        account.save()
        return JsonResponse({'status': 'ok'})

# ATTACK: Send 2 concurrent requests with amount=balance
# Request 1: Check (balance=100, amount=100) → PASS
# Request 2: Check (balance=100, amount=100) → PASS
# Request 1: balance = 0
# Request 2: balance = -100 (overdraft!)

# FIX: Use database-level atomic operations
@transaction.atomic
def withdraw(request):
    amount = int(request.POST['amount'])
    
    # Atomic update with WHERE clause
    updated = Account.objects.filter(
        user=request.user,
        balance__gte=amount  # Check in WHERE
    ).update(balance=F('balance') - amount)  # Update atomically
    
    if updated:
        return JsonResponse({'status': 'ok'})
    return JsonResponse({'error': 'Insufficient funds'}, status=400)
```

Detection logic:
1. Search for patterns: read → delay → write
2. Check if resource is shared (database record, file)
3. Verify no row-level locking or atomic operations
4. Suggest concurrent request attack

### Horizontal Privilege Escalation (IDOR)

```java
// VULNERABILITY: Missing user ownership check
// File: DocumentController.java
@GetMapping("/api/documents/{id}")
public Document getDocument(@PathVariable Long id) {
    // MISSING: Verify document belongs to current user
    return documentRepository.findById(id).orElseThrow();
}

// ATTACK: User A can access User B's documents
// GET /api/documents/123 (where 123 belongs to User B)

// FIX: Add ownership verification
@GetMapping("/api/documents/{id}")
public Document getDocument(@PathVariable Long id, 
                           @AuthenticationPrincipal User currentUser) {
    Document doc = documentRepository.findById(id).orElseThrow();
    
    if (!doc.getOwnerId().equals(currentUser.getId())) {
        throw new AccessDeniedException("Not your document");
    }
    
    return doc;
}
```

Detection logic:
1. Enumerate all CRUD endpoints
2. Check if user/tenant ownership verification exists
3. For endpoints with IDs, verify authorization logic
4. Missing check = IDOR vulnerability

## Configuration Vulnerability Detection

### Secrets in Code

```python
# VULNERABILITY: Hardcoded credentials
# File: config.py
DATABASE_URL = "postgresql://admin:P@ssw0rd123@db.internal:5432/prod"
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
JWT_SECRET = "my-secret-key-12345"

# FIX: Use environment variables
import os

DATABASE_URL = os.getenv('DATABASE_URL')
AWS_ACCESS_KEY = os.getenv('AWS_ACCESS_KEY')
JWT_SECRET = os.getenv('JWT_SECRET')

if not all([DATABASE_URL, AWS_ACCESS_KEY, JWT_SECRET]):
    raise RuntimeError("Missing required environment variables")
```

Detection regex patterns:
```python
patterns = [
    r'password\s*=\s*["\'][^"\']+["\']',
    r'api[_-]?key\s*=\s*["\'][^"\']+["\']',
    r'secret\s*=\s*["\'][^"\']+["\']',
    r'AKIA[0-9A-Z]{16}',  # AWS access key
    r'postgresql://[^:]+:[^@]+@',  # DB URL with password
]
```

### Weak Cryptography

```java
// VULNERABILITY: Weak cipher mode
// File: EncryptionUtil.java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
// ECB mode is vulnerable to pattern analysis

// FIX: Use authenticated encryption
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
GCMParameterSpec gcmSpec = new GCMParameterSpec(128, iv);
cipher.init(Cipher.ENCRYPT_MODE, secretKey, gcmSpec);
```

Detection logic:
1. Search for `Cipher.getInstance()`, `hashlib.md5()`, `DES`, `RC4`
2. Check cipher modes: ECB (bad), CBC without MAC (bad), GCM (good)
3. Check hash functions: MD5/SHA1 (bad for passwords), bcrypt/argon2 (good)
4. Verify IV generation: static IV (bad), random IV (good)

## Helper Scripts Usage

While primarily AI-driven, the skill includes optional Python scripts:

### Pattern Scanner

```bash
# Scan for SQL injection patterns
python scripts/pattern_scanner.py /path/to/project --pattern sql_injection

# Scan for all vulnerabilities
python scripts/pattern_scanner.py /path/to/project --all
```

```python
# In Python code
from scripts.pattern_scanner import PatternScanner

scanner = PatternScanner()
results = scanner.scan_directory('/path/to/project', 
                                  patterns=['sql_injection', 'xss'])

for result in results:
    print(f"{result['file']}:{result['line']} - {result['type']}")
```

### Data Flow Analyzer

```bash
# Trace taint flow from source to sink
python scripts/data_flow_analyzer.py /path/to/project \
    --source "request.GET" \
    --sink "os.system"
```

```python
from scripts.data_flow_analyzer import DataFlowAnalyzer

analyzer = DataFlowAnalyzer()
flows = analyzer.trace_taint('/path/to/project', 
                              source_pattern='request.GET',
                              sink_pattern='os.system')

for flow in flows:
    print(f"Taint flow: {' → '.join(flow['path'])}")
```

### Secret Finder

```bash
# Find hardcoded secrets
python scripts/secret_finder.py /path/to/project --output secrets.json
```

Detects:
- AWS/Azure/GCP credentials
- Database connection strings
- API keys
- Private keys
- JWT secrets

### Dependency Analyzer

```bash
# Check for vulnerable dependencies
python scripts/dependency_analyzer.py /path/to/project

# Output:
# [CRITICAL] jackson-databind 2.9.8 - CVE-2019-12384
# [HIGH] spring-web 5.0.7 - CVE-2020-5421
```

### Vulnerability Validator

```bash
# Validate SQL injection with Docker
python scripts/vulnerability_validator.py \
    --type sql_injection \
    --file UserMapper.java \
    --line 42 \
    --docker-compose docker-compose.yml
```

Creates isolated test environment, sends POC payload, validates actual exploitability.

## Report Generation

### Standard Report Format

```markdown
# Security Audit Report

## Executive Summary
- **Project**: MyApp v2.1
- **Language**: Java + Spring Boot 2.7
- **Audit Date**: 2026-06-06
- **Severity**: 3 Critical, 7 High, 12 Medium, 5 Low

## Critical Vulnerabilities

### [C1] SQL Injection in User Authentication
- **File**: `src/main/java/com/example/UserMapper.java:42`
- **Severity**: Critical (CVSS 9.8)
- **CWE**: CWE-89

**Vulnerable Code**:
```java
@Select("SELECT * FROM users WHERE username = '${username}'")
```

**Taint Flow**:
```
UserController.login(@RequestParam username)
  ↓ [No sanitization]
UserService.authenticate(username)
  ↓ [No sanitization]
UserMapper.findByUsername(username)
  ↓ [String concatenation]
SQL Query (SINK)
```

**POC**:
```bash
curl -X POST http://localhost:8080/login \
  -d "username=admin'--&password=any"
# Successfully bypasses authentication
```

**Fix**:
```java
@Select("SELECT * FROM users WHERE username = #{username}")
```

**Impact**: Complete authentication bypass, data exfiltration
**Priority**: P0 - Fix immediately
```

### Generate Report

```bash
# Generate from scan results
python scripts/report_generator.py \
    --input results.json \
    --output report.md \
    --template templates/report-templates/web-app-report-template.md
```

## Anti-Hallucination Mechanisms

The skill enforces strict verification to prevent false positives:

### Evidence Requirements

Every vulnerability must have:
1. **Exact file path and line number**
2. **Complete taint flow** from source to sink
3. **Actual code snippet** (not paraphrased)
4. **POC that demonstrates exploitability**

### Path Validation

```
✅ VALID PATH:
request.POST['user_id'] → user_id → User.objects.get(id=user_id)
[Source: HTTP input] → [Variable] → [Sink: DB query]

❌ INVALID PATH (missing steps):
request.POST → User.objects.get()
[What variable? What happened in between?]
```

### Real Testing Requirement

For Deep mode, vulnerabilities must be validated:

```bash
# Automated validation with Docker
python scripts/vulnerability_validator.py \
    --vuln sql_injection \
    --payload "admin'--" \
    --expect "authentication bypass"
```

If POC fails → Not a real vulnerability → Discard

## Framework-Specific Knowledge

### Spring Boot Security

```java
// Check 1: Verify Spring Security is enabled
// File: pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-security-starter</artifactId>
</dependency>

// Check 2: Verify filter chain configuration
// File: SecurityConfig.java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/admin/**").hasRole("ADMIN")  // Good
            .antMatchers("/api/**").permitAll();        // Potential issue
    }
}

// Check 3: Method-level security
@PreAuthorize("hasRole('ADMIN')")  // Good
public void deleteUser(Long id) { }

// Missing @PreAuthorize = Potential authz issue
```

### Django Security

```python
# Check 1: Verify middleware
# File: settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',  # Must have
    'django.contrib.auth.middleware.AuthenticationMiddleware',
]

# Check 2: Security settings
SECURE_SSL_REDIRECT = True  # Good
SESSION_COOKIE_SECURE = True  # Good
CSRF_COOKIE_HTTPONLY = True  # Good
SECRET_KEY = os.getenv('SECRET_KEY')  # Good, not hardcoded

# Check 3: View-level protection
from django.contrib.auth.decorators import login_required, permission_required

@login_required  # Good
@permission_required('app.delete_user')  # Good
def delete_user(request, user_id):
    pass
```

## Common Attack Chains

### Chain 1: Auth Bypass → Privilege Escalation → Data Exfil

```
1. SQL Injection in login bypasses authentication
   → Gain user session

2. IDOR in /api/users/{id} allows viewing any user
   → Enumerate admin user ID

3. Mass assignment in /api/users/{id}/update allows role change
   → {"role": "admin"} → Become admin

4. Missing access control on /api/export allows data export
   → Download entire database
```

### Chain 2: File Upload → RCE

```
1. No file type validation in upload endpoint
   → Upload shell.jsp

2. Predictable upload directory (/uploads/)
   → Access http://target/uploads/shell.jsp

3. JSP execution enabled in Tomcat config
   → Remote code execution
```

## Troubleshooting

### Issue: Too many false positives

**Solution**: 
- Use Deep mode for better context understanding
- Enable real testing validation: `--validate-with-docker`
- Check if framework has built-in protection (e.g., Django ORM auto-escapes SQL)

### Issue: Missing vulnerabilities in framework code

**Solution**:
- Ensure all framework-specific rules are loaded
- Check `resources/knowledge/` for framework coverage
- Add custom rules in `resources/rules/custom_rules.md`

### Issue: Slow scanning on large projects

**Solution**:
```bash
# Use Quick mode for initial scan
--mode quick

# Focus on high-risk directories
--include "src/main/java/com/example/controllers/**"
--exclude "src/test/**"

# Parallelize with multiple agents
--agents 5
```

### Issue: Cannot validate POC in Docker

**Solution**:
```yaml
# Ensure docker-compose.yml is present
version: '3'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
  db:
    image: postgres:13
```

```bash
# Run validation
python scripts/vulnerability_validator.py \
    --docker-compose docker-compose.yml \
    --wait-for-healthy 30
```

## Advanced Usage

### Custom Rule Development

Create custom detection rules in `resources/rules/custom_rules.md`:

```yaml
rule:
  id: CUSTOM-001
  name: Unsafe Reflection in Java
  severity: HIGH
  pattern:
    - Class.forName($userInput)
    - $class.newInstance()
  taint_sources:
    - request.getParameter()
    - request.getHeader()
  validation:
    - Verify no whitelist of allowed classes
  fix: |
    Use a whitelist:
    Set<String> allowedClasses = Set.of("com.example.SafeClass");
    if (!allowedClasses.contains(className)) {
        throw new SecurityException("Class not allowed");
    }
```

### Integration with CI/CD

```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run Security Audit
        run: |
          python scripts/code_scan.py . \
            --mode quick \
            --output results.json \
            --fail-on critical,high
      
      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: security-audit
          path: results.json
```

### WooYun Case Database

The skill includes 200+ real-world vulnerabilities from WooYun (2010-2016):

```
resources/wooyun/wooyun_cases_by_vulnerability_type.md
├── SQL Injection (50 cases)
├── XSS (30 cases)
├── File Upload (25 cases)
├── SSRF (20 cases)
└── ... (more categories)
```

Reference format:
```markdown
### WooYun-2015-123456: Dedecms SQL Injection

**Vulnerability**: SQL injection in article search
**File**: `plus/search.php`
**Code**:
```php
$keyword = $_GET['keyword'];
$sql = "SELECT * FROM articles WHERE title LIKE '%{$keyword}%'";
```

**Exploit**: `keyword=test%' UNION SELECT user,password FROM admin--`
**Impact**: Database compromise
```

Use for learning attack patterns and validation techniques.

---

## Reference Documentation

Key files in `resources/knowledge/`:

- `architecture_analysis.md`: Phase 1 methodology
- `pattern_scanning.md`: Phase 2 methodology  
- `data_flow_analysis.md`: Phase 3 methodology
- `taint_analysis_enhanced.md`: Advanced taint tracking
- `vulnerability_validation.md`: Phase 4 methodology
- `attack_chain_analysis.md`: Multi-vulnerability chaining
- `anti_hallucination.md`: Accuracy enforcement
- `docker_verification.md`: Automated POC validation

This skill is maintained by the EastSword team for security research and education purposes only. Use responsibly and only on systems you have permission to test.
