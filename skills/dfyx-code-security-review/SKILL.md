---
name: dfyx-code-security-review
description: Expert-level code security audit skill using deep data flow analysis and business logic understanding for white-box static analysis across 9 languages
triggers:
  - "audit this codebase for security vulnerabilities"
  - "perform a security code review"
  - "find security issues in this project"
  - "analyze code for OWASP vulnerabilities"
  - "check for injection flaws and authentication issues"
  - "run a deep security audit"
  - "scan for SQL injection and RCE vulnerabilities"
  - "review code security following OWASP Top 10"
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

A professional code security audit skill designed for AI coding agents, implementing a five-phase standardized audit protocol with dual-track analysis (Sink-driven, Control-driven, Config-driven) to systematically discover and validate security vulnerabilities in source code.

## Overview

**dfyx_code_security_review** provides expert-level white-box static analysis capabilities covering:

- **9 Languages**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 Security Dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **14 Frameworks**: Spring Boot, Django, Flask, FastAPI, Express, Koa, Gin, Laravel, Rails, ASP.NET Core, Rust Web, NestJS/Fastify, MyBatis, ProcessWire

## Installation

Clone the repository and integrate with your AI coding agent:

```bash
# Clone the project
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git

# For Claude Code
cp -r skill-dfyx_code_security_review ~/.claude/skills/

# Install Python dependencies (optional, for standalone scripts)
cd skill-dfyx_code_security_review
pip install -r requirements.txt
```

## Core Methodology

### Five-Phase Audit Protocol

```
Phase 1: Reconnaissance & Mapping (10%)
    ├─ Technology stack identification
    ├─ Architecture diagram generation
    └─ Attack surface enumeration

Phase 2: Parallel Pattern Matching (30%)
    ├─ Sink-driven scanning (dangerous functions)
    ├─ Control-driven scanning (missing security controls)
    └─ Config-driven scanning (misconfigurations)

Phase 3: Deep Taint Tracking & Practical Testing (40%)
    ├─ Data flow analysis
    ├─ Path-sensitive analysis
    └─ POC generation and verification

Phase 4: Validation & Attack Chain Construction (15%)
    ├─ Vulnerability validation
    ├─ Attack chain analysis
    └─ Exploitability assessment

Phase 5: Structured Reporting (5%)
    ├─ Vulnerability documentation
    ├─ Remediation guidance
    └─ Risk prioritization
```

### Dual-Track Audit Model

| Track | Dimensions | Method | Target |
|-------|-----------|--------|--------|
| **Sink-driven** | D1 Injection, D4 Deserialization, D5 File Ops, D6 SSRF | Grep dangerous functions → Trace data flow → Verify no protection | Existing dangerous code |
| **Control-driven** | D3 Authorization, D9 Business Logic | Enumerate endpoints → Verify security controls exist → Missing = Vulnerability | Missing security controls |
| **Config-driven** | D2 Authentication, D7 Crypto, D8 Config, D10 Supply Chain | Search configs → Compare security baseline | Misconfigurations |

## Security Dimensions

| # | Dimension | Coverage |
|---|-----------|----------|
| D1 | Injection | SQL/Command/LDAP/SSTI/SpEL/JNDI/XPath/XML injection |
| D2 | Authentication | Token/Session/JWT/Filter chain vulnerabilities |
| D3 | Authorization | CRUD consistency, IDOR, horizontal privilege escalation |
| D4 | Deserialization | Java/Python/PHP gadget chains, unsafe deserialization |
| D5 | File Operations | Upload/download/path traversal vulnerabilities |
| D6 | SSRF | URL injection, protocol bypass |
| D7 | Cryptography | Key management, weak algorithms, improper KDF |
| D8 | Configuration | Actuator exposure, CORS, error disclosure |
| D9 | Business Logic | Race conditions, mass assignment, state machine flaws |
| D10 | Supply Chain | Dependency CVEs, outdated packages |

## Scan Modes

| Mode | Use Case | Scope | Time |
|------|----------|-------|------|
| **Quick** | CI/CD, small projects | Critical vulns, secrets, dependency CVEs | 5-10 min |
| **Standard** | Regular audits | OWASP Top 10, auth/authz, crypto | 30-60 min |
| **Deep** | Critical projects, pentest prep | Full coverage, attack chains, business logic | 1-3 hours |

## Usage Patterns

### Basic Audit Request

Trigger an audit using natural language:

```
"Audit this codebase for security vulnerabilities"
"Perform a deep security review of this Spring Boot project"
"Find SQL injection and authentication issues"
```

The agent will respond with:

```
[MODE] deep
[RECON] 874 files, Spring Boot 1.5 + Shiro 1.6 + JPA + Freemarker
[PLAN] 5 Agents, D1-D10 coverage, estimated 125 turns
[SCOPE] Focus areas: Authentication, SQL Injection, Deserialization
Confirm to start audit? (yes/no)
```

### Code Examples

#### SQL Injection Detection (Java)

**Vulnerable Code:**
```java
// UserController.java
@GetMapping("/user/search")
public List<User> searchUser(@RequestParam String username) {
    String sql = "SELECT * FROM users WHERE username = '" + username + "'";
    return jdbcTemplate.query(sql, new UserRowMapper());
}
```

**Detection Process:**
```
1. [SINK] Identified dangerous sink: jdbcTemplate.query()
2. [SOURCE] Traced user input: @RequestParam username
3. [DATAFLOW] username → sql concatenation → jdbcTemplate.query()
4. [VALIDATION] No PreparedStatement, no input validation
5. [VULNERABILITY] SQL Injection (Critical)
```

**Remediation:**
```java
// Fixed version
@GetMapping("/user/search")
public List<User> searchUser(@RequestParam String username) {
    String sql = "SELECT * FROM users WHERE username = ?";
    return jdbcTemplate.query(sql, new Object[]{username}, new UserRowMapper());
}
```

#### Command Injection Detection (Python)

**Vulnerable Code:**
```python
# utils.py
import subprocess

def ping_host(host):
    cmd = f"ping -c 4 {host}"
    result = subprocess.call(cmd, shell=True)
    return result
```

**Detection Process:**
```
1. [SINK] Identified dangerous sink: subprocess.call(shell=True)
2. [SOURCE] Traced user input: host parameter
3. [DATAFLOW] host → f-string interpolation → subprocess.call()
4. [VALIDATION] No input sanitization, shell=True enables injection
5. [VULNERABILITY] Command Injection (Critical)
```

**Remediation:**
```python
# Fixed version
import subprocess
import shlex

def ping_host(host):
    # Validate host format
    if not re.match(r'^[a-zA-Z0-9.-]+$', host):
        raise ValueError("Invalid host format")
    
    # Use list instead of shell
    result = subprocess.call(['ping', '-c', '4', host])
    return result
```

#### Authorization Bypass Detection (Node.js)

**Vulnerable Code:**
```javascript
// routes/api.js
router.get('/admin/users', (req, res) => {
    // Missing authorization check
    const users = db.getAllUsers();
    res.json(users);
});

router.delete('/admin/user/:id', (req, res) => {
    // Missing authorization check
    db.deleteUser(req.params.id);
    res.json({success: true});
});
```

**Detection Process:**
```
1. [CONTROL] Enumerated admin endpoints: /admin/users, /admin/user/:id
2. [CHECK] Searched for middleware: auth.isAdmin, requireAdmin
3. [MISSING] No authorization middleware found
4. [VALIDATION] Direct access possible without admin role
5. [VULNERABILITY] Missing Authorization (High)
```

**Remediation:**
```javascript
// Fixed version
const requireAdmin = (req, res, next) => {
    if (!req.user || req.user.role !== 'admin') {
        return res.status(403).json({error: 'Forbidden'});
    }
    next();
};

router.get('/admin/users', requireAdmin, (req, res) => {
    const users = db.getAllUsers();
    res.json(users);
});

router.delete('/admin/user/:id', requireAdmin, (req, res) => {
    db.deleteUser(req.params.id);
    res.json({success: true});
});
```

#### Deserialization Vulnerability (Java)

**Vulnerable Code:**
```java
// DataProcessor.java
public Object processData(String base64Data) {
    byte[] data = Base64.getDecoder().decode(base64Data);
    ObjectInputStream ois = new ObjectInputStream(
        new ByteArrayInputStream(data)
    );
    return ois.readObject(); // Unsafe deserialization
}
```

**Detection Process:**
```
1. [SINK] Identified dangerous sink: ObjectInputStream.readObject()
2. [SOURCE] Traced user input: base64Data parameter
3. [DATAFLOW] base64Data → Base64 decode → readObject()
4. [GADGET] Checked classpath for known gadget chains (Commons-Collections)
5. [VULNERABILITY] Unsafe Deserialization → RCE (Critical)
```

**Remediation:**
```java
// Fixed version using safe alternatives
public Map<String, Object> processData(String jsonData) {
    ObjectMapper mapper = new ObjectMapper();
    // Use JSON instead of Java serialization
    return mapper.readValue(jsonData, 
        new TypeReference<Map<String, Object>>(){});
}

// OR use whitelist if serialization is required
public Object processData(String base64Data) {
    byte[] data = Base64.getDecoder().decode(base64Data);
    ValidatingObjectInputStream vois = new ValidatingObjectInputStream(
        new ByteArrayInputStream(data)
    );
    // Whitelist allowed classes
    vois.accept(SafeClass.class, AnotherSafeClass.class);
    return vois.readObject();
}
```

#### Path Traversal Detection (PHP)

**Vulnerable Code:**
```php
// download.php
<?php
$file = $_GET['file'];
$path = "/var/www/uploads/" . $file;
if (file_exists($path)) {
    header('Content-Type: application/octet-stream');
    readfile($path);
}
?>
```

**Detection Process:**
```
1. [SINK] Identified file operation: readfile()
2. [SOURCE] Traced user input: $_GET['file']
3. [DATAFLOW] $_GET['file'] → concatenation → readfile()
4. [VALIDATION] No path sanitization, allows "../" sequences
5. [ATTACK] Payload: ?file=../../../etc/passwd
6. [VULNERABILITY] Path Traversal (High)
```

**Remediation:**
```php
// Fixed version
<?php
function sanitizeFilename($filename) {
    // Remove directory traversal sequences
    $filename = str_replace(['../', '..\\'], '', $filename);
    // Only allow alphanumeric, dash, underscore, dot
    $filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
    return basename($filename);
}

$file = sanitizeFilename($_GET['file']);
$path = "/var/www/uploads/" . $file;

// Verify file is within allowed directory
$realpath = realpath($path);
if ($realpath && strpos($realpath, '/var/www/uploads/') === 0 && file_exists($realpath)) {
    header('Content-Type: application/octet-stream');
    readfile($realpath);
} else {
    http_response_code(404);
    echo "File not found";
}
?>
```

## Standalone Script Usage

For command-line execution:

```bash
# Full scan
python scripts/code_scan.py /path/to/project --mode deep --output results.json

# Quick scan (CI/CD)
python scripts/code_scan.py /path/to/project --mode quick

# Focus on specific dimensions
python scripts/code_scan.py /path/to/project --dimensions D1,D2,D3

# Secret detection only
python scripts/secret_finder.py /path/to/project

# Dependency vulnerability check
python scripts/dependency_analyzer.py /path/to/project

# Generate report
python scripts/report_generator.py --input results.json --output report.md
```

### Python API Usage

```python
from scripts.code_scan import SecurityScanner
from scripts.data_flow_analyzer import DataFlowAnalyzer

# Initialize scanner
scanner = SecurityScanner(project_path='/path/to/project', mode='deep')

# Run scan
results = scanner.scan()

# Analyze specific vulnerability
dfa = DataFlowAnalyzer(project_path='/path/to/project')
flow = dfa.trace_taint(
    source='request.getParameter("id")',
    sink='jdbcTemplate.query(sql)',
    file='UserController.java'
)

# Generate report
scanner.generate_report(results, output_file='security_report.md')
```

## Configuration

### Environment Variables

```bash
# Set audit mode
export DFYX_AUDIT_MODE=deep  # quick|standard|deep

# Enable Docker verification
export DFYX_DOCKER_VERIFY=true

# Set report output format
export DFYX_REPORT_FORMAT=markdown  # markdown|json|html

# Configure parallel agents
export DFYX_AGENTS=5

# Set vulnerability severity threshold
export DFYX_MIN_SEVERITY=medium  # low|medium|high|critical
```

### Custom Rules Configuration

Create `custom_rules.yaml`:

```yaml
custom_rules:
  - id: CUSTOM_001
    name: "Internal API Key Exposure"
    pattern: 'INTERNAL_API_KEY\s*=\s*["\'][^"\']+["\']'
    severity: critical
    language: [python, javascript]
    
  - id: CUSTOM_002
    name: "Unsafe Eval Usage"
    pattern: '\beval\s*\('
    severity: high
    language: [javascript, python]
    
  - id: CUSTOM_003
    name: "Missing CSRF Protection"
    pattern: '@PostMapping.*\n(?!.*@CsrfToken)'
    severity: medium
    language: [java]
```

Load custom rules:

```bash
python scripts/code_scan.py /path/to/project --rules custom_rules.yaml
```

## Report Structure

### Generated Report Sections

1. **Executive Summary**
   - Total vulnerabilities by severity
   - Critical findings summary
   - Risk score

2. **Vulnerability Details**
   - Code location with line numbers
   - Data flow diagram
   - Exploitation POC
   - Remediation code examples

3. **Attack Chain Analysis**
   - Multi-stage attack scenarios
   - Combined vulnerability exploitation

4. **Dependency Analysis**
   - Known CVEs in dependencies
   - Outdated package versions
   - License compliance

5. **Secrets Detection**
   - API keys, passwords in code
   - Hard-coded credentials
   - Environment leakage

## Common Patterns

### Pattern 1: SQL Injection in ORM

```python
# Django vulnerable pattern
def search_users(request):
    query = request.GET.get('q')
    # Raw SQL with format string - VULNERABLE
    users = User.objects.raw(
        f"SELECT * FROM users WHERE name LIKE '%{query}%'"
    )
    return render(request, 'users.html', {'users': users})

# Fixed version
def search_users(request):
    query = request.GET.get('q')
    # Use parameterized query
    users = User.objects.raw(
        "SELECT * FROM users WHERE name LIKE %s",
        [f'%{query}%']
    )
    return render(request, 'users.html', {'users': users})
```

### Pattern 2: JWT Authentication Bypass

```javascript
// Vulnerable JWT validation
function verifyToken(token) {
    try {
        // Using 'none' algorithm vulnerable to bypass
        const decoded = jwt.decode(token, {complete: true});
        if (decoded.payload.admin) {
            return decoded.payload;
        }
    } catch(e) {
        return null;
    }
}

// Fixed version
function verifyToken(token) {
    try {
        // Verify signature with secret
        const decoded = jwt.verify(token, process.env.JWT_SECRET, {
            algorithms: ['HS256'] // Whitelist allowed algorithms
        });
        if (decoded.admin === true) {
            return decoded;
        }
    } catch(e) {
        return null;
    }
}
```

### Pattern 3: IDOR in REST API

```java
// Vulnerable controller
@GetMapping("/api/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId) {
    // Missing authorization check - any user can access any order
    return orderService.findById(orderId);
}

// Fixed version
@GetMapping("/api/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId, 
                      @AuthenticationPrincipal User currentUser) {
    Order order = orderService.findById(orderId);
    
    // Verify order belongs to current user
    if (!order.getUserId().equals(currentUser.getId()) && 
        !currentUser.isAdmin()) {
        throw new AccessDeniedException("Cannot access this order");
    }
    
    return order;
}
```

### Pattern 4: XXE in XML Parsing

```java
// Vulnerable XML parser
public Document parseXML(String xmlString) {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    DocumentBuilder builder = factory.newDocumentBuilder();
    return builder.parse(new InputSource(new StringReader(xmlString)));
}

// Fixed version
public Document parseXML(String xmlString) {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    
    // Disable external entities
    factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
    factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
    factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
    factory.setExpandEntityReferences(false);
    
    DocumentBuilder builder = factory.newDocumentBuilder();
    return builder.parse(new InputSource(new StringReader(xmlString)));
}
```

### Pattern 5: Race Condition in Balance Transfer

```python
# Vulnerable transfer function
def transfer_money(from_account, to_account, amount):
    # Race condition: multiple concurrent transfers can overdraw
    balance = get_balance(from_account)
    if balance >= amount:
        time.sleep(0.1)  # Simulate processing delay
        decrease_balance(from_account, amount)
        increase_balance(to_account, amount)
        return True
    return False

# Fixed version with transaction lock
from django.db import transaction

@transaction.atomic
def transfer_money(from_account, to_account, amount):
    # Use select_for_update to lock row
    account = Account.objects.select_for_update().get(id=from_account)
    
    if account.balance >= amount:
        account.balance -= amount
        account.save()
        
        recipient = Account.objects.select_for_update().get(id=to_account)
        recipient.balance += amount
        recipient.save()
        return True
    return False
```

## Troubleshooting

### Issue: High False Positive Rate

**Solution:**
```bash
# Enable stricter validation
export DFYX_VALIDATION_MODE=strict

# Require POC generation for all findings
export DFYX_REQUIRE_POC=true

# Run with actual testing verification
python scripts/code_scan.py /path/to/project --verify-with-tests
```

### Issue: Missing Framework-Specific Vulnerabilities

**Solution:**
```bash
# Enable framework-aware analysis
python scripts/code_scan.py /path/to/project --framework spring-boot

# Add framework-specific rules
python scripts/code_scan.py /path/to/project --rules resources/rules/spring_security_rules.yaml
```

### Issue: Slow Scan Performance

**Solution:**
```bash
# Use quick mode for CI/CD
python scripts/code_scan.py /path/to/project --mode quick

# Increase parallel agents
export DFYX_AGENTS=10

# Exclude test files
python scripts/code_scan.py /path/to/project --exclude "**/test/**,**/tests/**"
```

### Issue: Cannot Trace Complex Data Flow

**Solution:**
```bash
# Enable enhanced taint analysis
python scripts/code_scan.py /path/to/project --enhanced-taint

# Increase analysis depth
export DFYX_TAINT_DEPTH=10

# Manual data flow analysis
python scripts/data_flow_analyzer.py \
    --source "HttpServletRequest.getParameter" \
    --sink "Runtime.exec" \
    --file "VulnerableController.java"
```

## Anti-Hallucination Mechanisms

The skill implements strict evidence requirements:

1. **Code Evidence**: Must provide exact file path and line numbers
2. **Data Flow Path**: Must trace complete source → sink path
3. **POC Generation**: Must demonstrate actual exploitability
4. **Docker Verification**: Optional containerized validation

```bash
# Enable all anti-hallucination checks
export DFYX_ANTI_HALLUCINATION=true
export DFYX_REQUIRE_EVIDENCE=true
export DFYX_DOCKER_VERIFY=true
```

## Integration with CI/CD

### GitHub Actions

```yaml
name: Security Audit
on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dfyx
        run: |
          git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
          cd skill-dfyx_code_security_review
          pip install -r requirements.txt
      
      - name: Run Security Scan
        run: |
          python skill-dfyx_code_security_review/scripts/code_scan.py . \
            --mode quick \
            --output results.json \
            --min-severity high
      
      - name: Upload Results
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: results.json
```

### GitLab CI

```yaml
security-audit:
  stage: test
  image: python:3.10
  script:
    - git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
    - cd skill-dfyx_code_security_review && pip install -r requirements.txt
    - cd .. && python skill-dfyx_code_security_review/scripts/code_scan.py . --mode quick
  artifacts:
    paths:
      - security_report.md
    expire_in: 1 week
```

## Advanced Features

### Attack Chain Analysis

The skill automatically identifies vulnerability combinations:

```
Attack Chain #1: Admin Account Takeover
├─ [Step 1] IDOR in /api/user/{id} (No authorization check)
├─ [Step 2] User enumeration via timing attack
├─ [Step 3] Password reset token prediction (weak randomness)
└─ [Impact] Full admin account compromise

POC:
1. Enumerate admin user ID: GET /api/user/1 (returns admin profile)
2. Trigger password reset: POST /api/reset-password {userId: 1}
3. Predict token using timestamp + user ID
4. Reset admin password: POST /api/confirm-reset {token: predicted}
```

### Business Logic Vulnerability Detection

```python
# Example: Discount code reuse vulnerability
@app.route('/apply-discount', methods=['POST'])
def apply_discount():
    code = request.json['code']
    # Missing: check if code already used by this user
    discount = DiscountCode.query.filter_by(code=code).first()
    if discount:
        session['discount'] = discount.percentage
        return {'success': True}
```

**Detection:**
```
[BUSINESS_LOGIC] Missing state validation
├─ Endpoint: /apply-discount
├─ Issue: No check for duplicate discount code usage
├─ Impact: User can apply same discount code multiple times
└─ Fix: Add user-code usage tracking table
```

## WooYun Case Database

The skill includes 1000+ real-world vulnerability cases from WooYun (2010-2016):

```bash
# Search WooYun cases for similar vulnerabilities
python scripts/code_scan.py /path/to/project --with-wooyun-cases

# Output will reference similar cases
# "Similar to WooYun-2016-12345: Dedecms SQL Injection in article.php"
```

## Best Practices

1. **Pre-Audit Checklist**
   - Ensure complete source code access
   - Identify all entry points (APIs, forms, file uploads)
   - Document authentication mechanisms
   - List third-party dependencies

2. **During Audit**
   - Confirm each finding with POC
   - Test in isolated environment
   - Document data flow with diagrams
   - Prioritize by exploitability + impact

3. **Post-Audit**
   - Validate all remediation code
   - Re-scan after fixes
   - Add to regression test suite
   - Update threat model

## License

MIT License - Free for security research and educational purposes only.

## Resources

- **Documentation**: `resources/knowledge/` (13 knowledge base documents)
- **Rule Sets**: `resources/rules/` (vulnerability detection patterns)
- **WooYun Cases**: `resources/wooyun/wooyun_cases_by_vulnerability_type.md`
- **Templates**: `templates/report-templates/` (standard report formats)
- **Examples**: `examples/` (audit examples and POCs)

For issues or contributions, visit: https://github.com/EastSword/skill-dfyx_code_security_review
