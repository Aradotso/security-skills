---
name: dfyx-code-security-review
description: Expert-level white-box code security audit skill using deep data flow analysis and business logic understanding for 9 languages
triggers:
  - audit this project for security vulnerabilities
  - perform code security review
  - check code for security issues
  - analyze code security
  - find security vulnerabilities in this code
  - run security audit
  - review code for vulnerabilities
  - scan for security flaws
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Expert-level code security audit skill designed for AI coding agents, implementing a five-phase standardized audit protocol with dual-track analysis models (Sink-driven, Control-driven, Config-driven) for comprehensive vulnerability discovery.

## Overview

**dfyx_code_security_review** performs professional white-box static code analysis across 9 programming languages and 10 security dimensions. It combines pattern matching, taint analysis, and business logic understanding to systematically identify and validate security vulnerabilities.

### Supported Languages & Frameworks

- **Java**: Spring Boot, MyBatis, Struts
- **Python**: Django, Flask, FastAPI
- **Go**: Gin, Echo
- **PHP**: Laravel, Symfony, ProcessWire
- **JavaScript/Node.js**: Express, Koa, NestJS
- **C/C++**: Native applications
- **.NET/C#**: ASP.NET Core
- **Ruby**: Rails
- **Rust**: Actix, Rocket

### 10 Security Dimensions

| Dimension | Coverage |
|-----------|----------|
| D1: Injection | SQL, Command, LDAP, SSTI, SpEL, JNDI |
| D2: Authentication | Token, Session, JWT, Filter chains |
| D3: Authorization | CRUD consistency, IDOR, horizontal privilege escalation |
| D4: Deserialization | Java, Python, PHP gadget chains |
| D5: File Operations | Upload, download, path traversal |
| D6: SSRF | URL injection, protocol restrictions |
| D7: Cryptography | Key management, encryption modes, KDF |
| D8: Configuration | Actuator, CORS, information disclosure |
| D9: Business Logic | Race conditions, mass assignment, state machines |
| D10: Supply Chain | Dependency CVEs, version checks |

## Installation

This skill is triggered automatically by AI coding agents when security audit requests are detected. No manual installation required in most AI environments.

For standalone use or custom integration:

```bash
# Clone or copy to AI agent skills directory
# Claude Code
cp -r dfyx_code_security_review ~/.claude/skills/

# Cursor/other agents
# Copy to appropriate skills directory per agent documentation
```

### Optional Python Scripts

Install dependencies for standalone script usage:

```bash
pip install -r requirements.txt
```

Dependencies include: `tree-sitter`, `semgrep`, `bandit`, `safety`, `pyyaml`, `gitpython`, `requests`

## Audit Modes

| Mode | Use Case | Scope | Duration |
|------|----------|-------|----------|
| **Quick** | CI/CD, small projects | High-risk vulns, secrets, dependency CVEs | 5-10 min |
| **Standard** | Regular audits | OWASP Top 10, auth/authz, crypto | 30-60 min |
| **Deep** | Critical projects, pentest prep | Full coverage, attack chains, business logic | 1-3 hours |

## Five-Phase Audit Protocol

```
Phase 1: Reconnaissance (10%)
    ↓ Output: Architecture diagram, attack surface inventory
Phase 2: Pattern Matching (30%)
    ↓ Output: High-risk area list
Phase 3: Taint Tracking & Validation (40%)
    ↓ Output: Confirmed vulnerabilities, test validation
Phase 4: Verification & Attack Chain (15%)
    ↓ Output: Vulnerability validation report
Phase 5: Structured Reporting (5%)
    ↓ Output: Complete audit report
```

## Usage Examples

### Basic Audit Request

```
User: Please audit /path/to/spring-boot-app for security vulnerabilities

AI Response:
[MODE] standard
[RECON] 347 files, Spring Boot 2.7 + MyBatis + Thymeleaf
[PLAN] 3 Agents, D1-D8 coverage, estimated 60 turns
[SCOPE] Focus: SQL Injection, Auth, IDOR, Crypto
Confirm to start audit? (yes/no)
```

### Deep Audit with Specific Focus

```
User: Run deep security audit on this Django project, focus on authentication and authorization

AI Response:
[MODE] deep
[RECON] 524 files, Django 3.2 + DRF + Celery + PostgreSQL
[PLAN] 5 Agents, D2-D3 priority, D1,D4-D10 secondary
[SCOPE] 
- Phase 1: Map auth middleware chain
- Phase 2: Scan @login_required, permissions
- Phase 3: Trace user input → DB queries
- Phase 4: Test IDOR, horizontal escalation
Estimated 90 turns. Proceed? (yes/no)
```

## Dual-Track Analysis Models

### 1. Sink-Driven (D1, D4, D5, D6)

```python
# Example: SQL Injection Detection
# Pattern: Search for dangerous sinks
dangerous_sinks = [
    'execute(', 'executemany(', 'raw(',  # Python
    'createQuery(', 'createNativeQuery(',  # Java
    'DB::raw(', 'DB::select('  # PHP Laravel
]

# Taint tracking flow
user_input = request.GET['id']  # Source (tainted)
query = f"SELECT * FROM users WHERE id={user_input}"  # No sanitization
cursor.execute(query)  # Sink - VULNERABILITY CONFIRMED

# Verification checklist:
# ✓ Source identified: HTTP parameter
# ✓ No sanitization middleware
# ✓ Direct string concatenation
# ✓ Sink reached: SQL execute
```

### 2. Control-Driven (D3, D9)

```java
// Example: Authorization Check
// Pattern: Enumerate endpoints → verify security controls

@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    // GOOD: Authorization present
    @GetMapping("/{id}")
    @PreAuthorize("hasPermission(#id, 'Order', 'READ')")
    public Order getOrder(@PathVariable Long id) {
        return orderService.findById(id);
    }
    
    // VULNERABLE: Missing authorization check
    @DeleteMapping("/{id}")  // ❌ No @PreAuthorize
    public void deleteOrder(@PathVariable Long id) {
        orderService.delete(id);  // IDOR vulnerability
    }
}

// Verification:
// ✓ CRUD operations: GET, POST, PUT, DELETE
// ✗ DELETE missing @PreAuthorize → IDOR confirmed
// Attack: User A can delete User B's order
```

### 3. Config-Driven (D2, D7, D8, D10)

```yaml
# Example: Spring Boot Configuration Audit
# File: application.yml

spring:
  security:
    # VULNERABLE: JWT secret in config
    jwt:
      secret: "hardcoded-secret-key-123"  # ❌ Should use ${JWT_SECRET}
  
  # VULNERABLE: Actuator exposed
  boot:
    admin:
      context-path: /admin
  management:
    endpoints:
      web:
        exposure:
          include: "*"  # ❌ Exposes /actuator/env, /actuator/heapdump

# Verification against baseline:
# ✗ Hardcoded secrets (D7)
# ✗ Actuator fully exposed without auth (D8)
# ✗ No rate limiting config (D9)
```

## Real Code Examples

### SQL Injection Detection (Java/MyBatis)

```java
// VULNERABLE CODE
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE username = '${username}'")
    User findByUsername(@Param("username") String username);
    // ❌ ${} causes SQL injection
}

// TAINT ANALYSIS
// Source: @RequestParam String username
// Flow: Controller → Service → Mapper
// Sink: @Select with ${}
// Vulnerability: SQL Injection (Critical)

// FIX
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE username = #{username}")
    User findByUsername(@Param("username") String username);
    // ✓ #{} uses PreparedStatement
}
```

### Command Injection (Python/Flask)

```python
# VULNERABLE CODE
from flask import Flask, request
import subprocess

@app.route('/ping')
def ping():
    host = request.args.get('host')
    # ❌ Direct shell injection
    result = subprocess.run(f'ping -c 4 {host}', shell=True, capture_output=True)
    return result.stdout

# TAINT FLOW
# Source: request.args.get('host')
# Sanitization: None
# Sink: subprocess.run(..., shell=True)
# PoC: /ping?host=example.com;whoami

# FIX
import shlex
@app.route('/ping')
def ping():
    host = request.args.get('host')
    # Whitelist validation
    if not re.match(r'^[a-zA-Z0-9.-]+$', host):
        return 'Invalid host', 400
    # Avoid shell=True
    result = subprocess.run(['ping', '-c', '4', host], capture_output=True)
    return result.stdout
```

### IDOR (Insecure Direct Object Reference)

```javascript
// VULNERABLE CODE (Node.js/Express)
app.get('/api/documents/:id', async (req, res) => {
    // ❌ No ownership check
    const doc = await Document.findById(req.params.id);
    res.json(doc);
});

// ATTACK VECTOR
// 1. User A creates document (id=123)
// 2. User B requests /api/documents/123
// 3. User B receives User A's document

// FIX: Add authorization
app.get('/api/documents/:id', authenticateUser, async (req, res) => {
    const doc = await Document.findById(req.params.id);
    
    // ✓ Ownership check
    if (doc.userId !== req.user.id) {
        return res.status(403).json({ error: 'Forbidden' });
    }
    
    res.json(doc);
});
```

### Deserialization (Java)

```java
// VULNERABLE CODE
public class UserController {
    @PostMapping("/import")
    public void importData(@RequestParam("file") MultipartFile file) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(file.getInputStream());
        Object obj = ois.readObject();  // ❌ Untrusted deserialization
        processData(obj);
    }
}

// EXPLOITATION
// 1. Attacker crafts malicious serialized object (CommonsCollections gadget)
// 2. Uploads via /import endpoint
// 3. readObject() triggers RCE

// FIX: Use safe formats
@PostMapping("/import")
public void importData(@RequestParam("file") MultipartFile file) throws Exception {
    // ✓ Use JSON instead of Java serialization
    ObjectMapper mapper = new ObjectMapper();
    UserData data = mapper.readValue(file.getInputStream(), UserData.class);
    
    // Additional validation
    validator.validate(data);
    processData(data);
}
```

### Path Traversal (Go)

```go
// VULNERABLE CODE
func downloadFile(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    // ❌ No path validation
    data, err := ioutil.ReadFile("/var/uploads/" + filename)
    if err != nil {
        http.Error(w, "File not found", 404)
        return
    }
    w.Write(data)
}

// ATTACK: /download?file=../../etc/passwd

// FIX: Validate and sanitize path
import "path/filepath"

func downloadFile(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    
    // ✓ Clean path
    cleanPath := filepath.Clean(filename)
    
    // ✓ Prevent directory traversal
    if strings.Contains(cleanPath, "..") {
        http.Error(w, "Invalid filename", 400)
        return
    }
    
    // ✓ Join safely
    fullPath := filepath.Join("/var/uploads", cleanPath)
    
    // ✓ Verify path is within uploads directory
    if !strings.HasPrefix(fullPath, "/var/uploads/") {
        http.Error(w, "Access denied", 403)
        return
    }
    
    data, err := ioutil.ReadFile(fullPath)
    // ... rest of code
}
```

## Anti-Hallucination Mechanisms

The skill implements strict validation to prevent false positives:

```markdown
### Evidence Requirements
- ✓ Source code snippet (exact line numbers)
- ✓ Complete data flow path
- ✓ Proof of missing sanitization
- ✓ Actual test validation (PoC when possible)

### Path Completeness
Source → [Intermediate Steps] → Sink
- All intermediate functions traced
- Framework middleware considered
- Sanitization/validation points checked

### Severity Validation Matrix
| Severity | Requirements |
|----------|--------------|
| Critical | Unauthenticated RCE/SQLi with PoC |
| High | Authenticated RCE/SQLi, or unauth data leak |
| Medium | Requires specific conditions |
| Low | Theoretical or low impact |
```

## Configuration

### Environment Variables

```bash
# For standalone script usage
export AUDIT_MODE=deep  # quick|standard|deep
export AUDIT_OUTPUT_DIR=/path/to/reports
export AUDIT_INCLUDE_POC=true
export AUDIT_VERIFY_DEPS=true
```

### Custom Rules

Create custom detection rules in `resources/rules/`:

```yaml
# custom_injection_rule.yaml
rules:
  - id: custom-sql-injection
    languages: [python]
    severity: CRITICAL
    pattern: |
      cursor.execute($QUERY)
    pattern-not: |
      cursor.execute($QUERY, ...)
    message: "SQL Injection via string formatting"
    metadata:
      dimension: D1
      cwe: CWE-89
```

## Standalone Script Usage

### Code Scanner

```bash
# Full audit
python scripts/code_scan.py /path/to/project

# Quick scan
python scripts/code_scan.py /path/to/project --mode quick

# Specific dimensions
python scripts/code_scan.py /path/to/project --dimensions D1,D3,D4

# Output formats
python scripts/code_scan.py /path/to/project --format json --output results.json
```

### Pattern Scanner

```python
# scripts/pattern_scanner.py
from pattern_scanner import PatternScanner

scanner = PatternScanner('/path/to/project')

# Scan for SQL injection
sql_vulns = scanner.scan_pattern('sql_injection', language='java')

# Results
for vuln in sql_vulns:
    print(f"{vuln['file']}:{vuln['line']} - {vuln['description']}")
    print(f"Code: {vuln['code_snippet']}")
    print(f"Severity: {vuln['severity']}\n")
```

### Dependency Analyzer

```bash
# Analyze dependencies for CVEs
python scripts/dependency_analyzer.py /path/to/project

# Output
# [HIGH] Spring Framework 5.2.8 - CVE-2021-22118 (CVSS 7.5)
# [CRITICAL] Log4j 2.14.1 - CVE-2021-44228 (CVSS 10.0)
# [MEDIUM] Jackson 2.9.8 - CVE-2020-24750 (CVSS 6.5)
```

### Secret Finder

```bash
# Find hardcoded secrets
python scripts/secret_finder.py /path/to/project

# Output
# [SECRET] application.yml:12 - JWT secret key
# [SECRET] config.py:45 - AWS access key
# [SECRET] .env.example:3 - Database password (example file)
```

## Common Patterns

### Phase 1: Reconnaissance

```markdown
## Reconnaissance Checklist

1. Technology Stack Identification
   - Parse pom.xml, package.json, requirements.txt, go.mod
   - Identify framework versions
   - List third-party dependencies

2. Entry Point Enumeration
   - Web: Controllers, routes, views
   - API: GraphQL schemas, REST endpoints
   - CLI: Argument parsers

3. Authentication Flow Mapping
   - Login endpoints
   - Token generation/validation
   - Session management
   - Password reset flows

4. Data Flow Architecture
   - Database models
   - ORM queries
   - Cache layers
   - Message queues
```

### Phase 2: Pattern Matching

```python
# Multi-language sink detection
DANGEROUS_SINKS = {
    'java': [
        'Runtime.getRuntime().exec(',
        'ProcessBuilder(',
        'ScriptEngine.eval(',
        'statement.execute(',
        'createNativeQuery('
    ],
    'python': [
        'eval(', 'exec(', 'compile(',
        'subprocess.run(', 'os.system(',
        'cursor.execute(', 'pickle.loads('
    ],
    'javascript': [
        'eval(', 'Function(',
        'child_process.exec(',
        'vm.runInNewContext(',
        'query.raw('
    ],
    'php': [
        'eval(', 'assert(',
        'exec(', 'system(', 'shell_exec(',
        'unserialize(',
        'mysql_query(', 'mysqli_query('
    ]
}
```

### Phase 3: Taint Analysis

```markdown
## Taint Tracking Template

### 1. Identify Sources (User-Controlled Input)
- HTTP: req.params, req.query, req.body, req.headers
- CLI: sys.argv, process.argv
- File: file uploads, config files
- DB: user-generated content
- Environment: process.env (if user can modify)

### 2. Trace Propagation
- Variable assignments
- Function calls
- Return values
- Array/Object modifications

### 3. Check Sanitization
- Input validation (whitelist, regex)
- Encoding/escaping
- Parameterized queries
- Framework built-in protections

### 4. Identify Sinks
- SQL execution
- Command execution
- Template rendering
- File operations
- Deserialization
```

### Phase 4: Vulnerability Validation

```python
# Validation checklist generator
def validate_vulnerability(vuln_candidate):
    checks = {
        'source_confirmed': False,
        'sink_confirmed': False,
        'no_sanitization': False,
        'exploitable': False,
        'poc_created': False
    }
    
    # 1. Confirm source is user-controlled
    if is_user_input(vuln_candidate.source):
        checks['source_confirmed'] = True
    
    # 2. Confirm sink is dangerous
    if is_dangerous_sink(vuln_candidate.sink):
        checks['sink_confirmed'] = True
    
    # 3. Verify no sanitization in path
    if not has_sanitization(vuln_candidate.data_flow):
        checks['no_sanitization'] = True
    
    # 4. Assess exploitability
    if checks['source_confirmed'] and checks['sink_confirmed'] and checks['no_sanitization']:
        checks['exploitable'] = True
    
    # 5. Create PoC (if safe)
    if checks['exploitable'] and vuln_candidate.safe_to_test:
        checks['poc_created'] = create_poc(vuln_candidate)
    
    return checks
```

## Troubleshooting

### Issue: Too Many False Positives

```markdown
Solution: Enable strict validation mode

In AI conversation:
"Please re-audit with strict validation: require complete data flow paths and proof of missing sanitization"

Or in config:
export AUDIT_STRICT_MODE=true
python scripts/code_scan.py /path/to/project --strict
```

### Issue: Audit Takes Too Long

```markdown
Solution: Use Quick mode or limit dimensions

"Run quick audit focusing on D1 (Injection) and D3 (Authorization)"

Or:
python scripts/code_scan.py /path/to/project --mode quick --dimensions D1,D3
```

### Issue: Missing Framework-Specific Protections

```markdown
Solution: Explicitly declare framework versions

"This is Spring Boot 2.7 with Spring Security 5.6 enabled. Re-evaluate auth vulnerabilities."

The skill will adjust analysis based on framework security features:
- Spring Security → checks for @PreAuthorize, SecurityFilterChain
- Django → checks for CSRF middleware, permission_classes
- Express → checks for helmet, express-validator
```

### Issue: Dependency Scan Fails

```bash
# Ensure package manifest files are present
ls pom.xml package.json requirements.txt go.mod Cargo.toml

# Update vulnerability database
pip install --upgrade safety

# Manual dependency check
python scripts/dependency_analyzer.py /path/to/project --update-db
```

### Issue: Need Custom Validation Rules

```python
# Add custom rule in resources/rules/custom_rules.yaml
rules:
  - id: custom-deserialize-check
    pattern: |
      pickle.loads($DATA)
    languages: [python]
    severity: HIGH
    message: "Unsafe pickle deserialization"
    fix: "Use json.loads() or validate input source"

# Reference in audit
"Apply custom rules from resources/rules/custom_rules.yaml during Phase 2"
```

## Report Structure

Generated reports follow this structure:

```markdown
# Security Audit Report

## Executive Summary
- Total Vulnerabilities: 15 (Critical: 3, High: 5, Medium: 5, Low: 2)
- Risk Score: 8.2/10
- OWASP Top 10 Coverage: 9/10 categories

## Critical Findings

### [C1] SQL Injection in User Search
**Severity**: Critical (CVSS 9.8)
**Location**: `src/controllers/UserController.java:45`
**CWE**: CWE-89

#### Vulnerability Details
```java
// Vulnerable code
String query = "SELECT * FROM users WHERE username='" + input + "'";
stmt.executeQuery(query);
```

**Data Flow**:
```
HTTP Request → @RequestParam username → userService.search(username) 
  → userRepository.findByUsername(username) → JDBC executeQuery()
```

**Proof of Concept**:
```bash
curl "http://localhost:8080/users/search?username=admin'--"
# Returns all users without authentication
```

#### Recommended Fix
```java
// Use PreparedStatement
String query = "SELECT * FROM users WHERE username=?";
PreparedStatement pstmt = conn.prepareStatement(query);
pstmt.setString(1, input);
ResultSet rs = pstmt.executeQuery();
```

**Alternative Fix**: Use JPA with parameterized queries
```java
@Query("SELECT u FROM User u WHERE u.username = :username")
User findByUsername(@Param("username") String username);
```

## Appendix
- Attack Chain Diagrams
- Dependency CVE List
- Secure Configuration Baseline
```

## Advanced Features

### Attack Chain Analysis

The skill can identify multi-stage attacks:

```markdown
### Attack Chain: Admin Account Takeover

1. [D8] Information Disclosure
   - /actuator/env exposes JWT secret

2. [D7] Weak Cryptography
   - JWT uses HS256 with leaked secret

3. [D2] Authentication Bypass
   - Forge admin JWT token

4. [D3] Authorization Flaw
   - Admin endpoints don't verify token claims

5. [D1] SQL Injection
   - Admin SQL query builder lacks sanitization

**Impact**: Complete system compromise
**Likelihood**: High (all steps confirmed exploitable)
**CVSS**: 10.0 (Network, Low Complexity, No Privileges Required)
```

### Multi-Tenant Security

```python
# Tenant isolation check pattern
def check_tenant_isolation(model_class, endpoints):
    issues = []
    
    for endpoint in endpoints:
        # Check 1: Query filters include tenant_id
        if not has_tenant_filter(endpoint.query):
            issues.append({
                'endpoint': endpoint.path,
                'issue': 'Missing tenant_id filter',
                'impact': 'Cross-tenant data leak'
            })
        
        # Check 2: Row-level security enabled
        if not has_rls_policy(model_class):
            issues.append({
                'model': model_class.__name__,
                'issue': 'No RLS policy',
                'impact': 'Database-level isolation missing'
            })
    
    return issues

# Example finding:
# [HIGH] /api/documents endpoint missing tenant_id filter
# User from Tenant A can access Tenant B's documents by ID enumeration
```

### Business Logic Vulnerabilities

```javascript
// Race condition detection
// VULNERABLE: Concurrent withdrawal
app.post('/withdraw', async (req, res) => {
    const account = await Account.findById(req.user.accountId);
    
    if (account.balance >= req.body.amount) {
        // ❌ Race condition: balance can be checked twice simultaneously
        account.balance -= req.body.amount;
        await account.save();
        res.json({ success: true });
    }
});

// ATTACK:
// 1. Balance: $100
// 2. Send two concurrent requests: withdraw $100
// 3. Both pass balance check
// 4. Final balance: -$100

// FIX: Use database-level atomic operations
app.post('/withdraw', async (req, res) => {
    const result = await Account.updateOne(
        { 
            _id: req.user.accountId,
            balance: { $gte: req.body.amount }  // ✓ Atomic check
        },
        { 
            $inc: { balance: -req.body.amount }  // ✓ Atomic update
        }
    );
    
    if (result.nModified === 0) {
        return res.status(400).json({ error: 'Insufficient funds' });
    }
    
    res.json({ success: true });
});
```

## Integration with CI/CD

```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on:
  pull_request:
    branches: [ main ]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install -r dfyx_code_security_review/requirements.txt
      
      - name: Run security audit
        run: |
          python dfyx_code_security_review/scripts/code_scan.py . \
            --mode quick \
            --format json \
            --output audit-results.json
      
      - name: Check for critical vulnerabilities
        run: |
          CRITICAL_COUNT=$(jq '[.vulnerabilities[] | select(.severity=="CRITICAL")] | length' audit-results.json)
          if [ "$CRITICAL_COUNT" -gt 0 ]; then
            echo "Found $CRITICAL_COUNT critical vulnerabilities"
            exit 1
          fi
      
      - name: Upload audit report
        uses: actions/upload-artifact@v3
        with:
          name: security-audit-report
          path: audit-results.json
```

## Knowledge Base References

The skill utilizes extensive knowledge bases from the project:

- **WooYun Cases**: 10+ real-world vulnerability examples (2010-2016)
- **OWASP Guidelines**: ASVS, Top 10, Cheat Sheets
- **CWE Mappings**: 50+ Common Weakness Enumeration entries
- **Framework Security**: Spring Security, Django Security, Express Security best practices

Access via: `resources/wooyun/wooyun_cases_by_vulnerability_type.md`

## Best Practices

1. **Start with Reconnaissance**: Always run Phase 1 completely before pattern matching
2. **Validate Data Flows**: Require complete source-to-sink paths with evidence
3. **Test When Safe**: Create PoCs for critical findings in isolated environments
4. **Prioritize by Impact**: Fix Critical/High severity issues first
5. **Document Assumptions**: Note framework versions, security middleware, custom protections
6. **Re-audit After Fixes**: Verify remediation with focused re-scan

## Performance Optimization

```markdown
### For Large Codebases (>100k LOC)

1. Use incremental scanning:
   "Audit only files changed in last commit"

2. Parallelize by dimension:
   "Run 3 agents: Agent 1 (D1-D3), Agent 2 (D4-D6), Agent 3 (D7-D10)"

3. Focus on high-risk areas:
   "Priority: Authentication, payment processing, admin panels"

4. Cache analysis results:
   export AUDIT_CACHE_DIR=/tmp/audit-cache
```

## License

This skill documentation is provided under MIT License. The original project license applies to all source code and resources.

---

**Security Notice**: This skill is for authorized security testing only. Always obtain proper authorization before auditing production systems. Use PoC exploits only in isolated test environments.
