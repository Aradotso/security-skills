---
name: dfyx-code-security-review
description: AI-powered white-box code security auditing with deep data flow analysis, taint tracking, and business logic vulnerability detection across 9 languages
triggers:
  - "audit this codebase for security vulnerabilities"
  - "perform a code security review"
  - "find security issues in this project"
  - "run a security audit on the code"
  - "check for vulnerabilities using dfyx"
  - "analyze code security with deep data flow analysis"
  - "scan for injection and authentication flaws"
  - "review code for OWASP Top 10 issues"
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

A professional-grade code security auditing framework designed for AI coding agents. Uses white-box static analysis with a five-phase audit protocol to systematically discover and verify vulnerabilities through deep data flow analysis, taint tracking, and business logic understanding.

## What It Does

**dfyx_code_security_review** performs expert-level security audits across:

- **9 Languages**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **14 Frameworks**: Spring Boot, Django, Flask, FastAPI, Express, Koa, Gin, Laravel, Rails, ASP.NET Core, Rust Web, NestJS, Fastify, MyBatis
- **10 Security Dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **3 Analysis Models**: Sink-driven (injection/RCE), Control-driven (authorization/logic), Config-driven (crypto/settings)

## Installation

### For AI Coding Agents

The skill is triggered automatically when you request a security audit. The agent will load all necessary knowledge base files from the skill directory.

### For Manual Use (Optional)

```bash
# Clone the repository
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
cd skill-dfyx_code_security_review

# Install Python dependencies (for optional scripts)
pip install -r requirements.txt
```

## Five-Phase Audit Protocol

```
Phase 1: Reconnaissance & Mapping (10%)
    ↓ Output: Architecture diagram, attack surface inventory
    
Phase 2: Parallel Pattern Matching (30%)
    ↓ Output: High-risk area inventory
    
Phase 3: Deep Taint Tracking & Live Testing (40%)
    ↓ Output: Confirmed vulnerabilities, test validation reports
    
Phase 4: Validation & Attack Chain Construction (15%)
    ↓ Output: Vulnerability validation reports
    
Phase 5: Structured Reporting (5%)
    ↓ Output: Complete audit report
```

## Audit Modes

| Mode | Use Case | Coverage | Time |
|------|----------|----------|------|
| **Quick** | CI/CD, small projects | Critical vulns, secrets, dependency CVEs | 5-10 min |
| **Standard** | Regular audits | OWASP Top 10, auth/authz, crypto | 30-60 min |
| **Deep** | Critical projects, pentest prep | Full coverage, attack chains, business logic | 1-3 hours |

## Usage Patterns

### Basic Audit Request

```
User: "Audit this project for security vulnerabilities"

AI Response:
[MODE] standard
[RECON] Identified: Spring Boot 2.7 + MySQL + Thymeleaf
        874 files, 45K LOC
[PLAN] Launching 3 agents, covering D1-D10
       Estimated 50 turns, 5-10 vulnerabilities
[SCOPE] Focus: Authentication, SQL Injection, Authorization
Confirm to start audit? (yes/no)
```

### Deep Audit with Specific Focus

```
User: "Perform a deep security audit focusing on authentication and authorization"

AI Response:
[MODE] deep
[RECON] Tech stack: Django 3.2 + PostgreSQL + Celery
[FOCUS] D2 (Authentication), D3 (Authorization)
[PLAN] Deep analysis of:
       - JWT token validation
       - Session management
       - RBAC implementation
       - IDOR vulnerabilities
Proceeding with audit...
```

### Quick Scan for CI/CD

```
User: "Quick security scan for CI pipeline"

AI Response:
[MODE] quick
[SCAN] Checking:
       ✓ Hardcoded secrets
       ✓ Critical dependency CVEs
       ✓ SQL injection sinks
       ✓ Command injection patterns
[RESULT] 2 critical issues found
```

## Core Security Dimensions

### D1: Injection Vulnerabilities

Detects SQL, Command, LDAP, SSTI, SpEL, and JNDI injection patterns.

**Example Detection (Java/Spring)**:

```java
// VULNERABLE: SQL Injection
@GetMapping("/users")
public List<User> getUsers(@RequestParam String name) {
    String sql = "SELECT * FROM users WHERE name = '" + name + "'";
    return jdbcTemplate.query(sql, userMapper);
}

// SECURE: Parameterized query
@GetMapping("/users")
public List<User> getUsers(@RequestParam String name) {
    String sql = "SELECT * FROM users WHERE name = ?";
    return jdbcTemplate.query(sql, userMapper, name);
}
```

**Detection Method**:
1. Grep for dangerous sinks: `executeQuery`, `createQuery`, `exec`, `Runtime.exec`
2. Trace data flow from source (user input) to sink
3. Verify absence of sanitization/parameterization
4. Generate POC payload

### D2: Authentication Flaws

Validates token generation, session management, and authentication filter chains.

**Example Detection (Python/Flask)**:

```python
# VULNERABLE: Weak JWT secret
app.config['JWT_SECRET_KEY'] = 'secret123'  # Hardcoded, weak

# VULNERABLE: No token expiration
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(request.json)
    token = jwt.encode({'user_id': user.id}, app.config['JWT_SECRET_KEY'])
    return {'token': token}

# SECURE: Strong secret from env, expiration set
app.config['JWT_SECRET_KEY'] = os.environ['JWT_SECRET_KEY']
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=1)

@app.route('/login', methods=['POST'])
def login():
    user = authenticate(request.json)
    token = create_access_token(identity=user.id)
    return {'token': token}
```

### D3: Authorization Issues

Checks CRUD permission consistency, IDOR, and horizontal privilege escalation.

**Example Detection (Node.js/Express)**:

```javascript
// VULNERABLE: IDOR - no ownership check
app.delete('/api/documents/:id', authenticate, async (req, res) => {
  await Document.findByIdAndDelete(req.params.id);
  res.json({ success: true });
});

// SECURE: Verify ownership before deletion
app.delete('/api/documents/:id', authenticate, async (req, res) => {
  const doc = await Document.findById(req.params.id);
  
  if (!doc) {
    return res.status(404).json({ error: 'Not found' });
  }
  
  if (doc.userId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  await doc.remove();
  res.json({ success: true });
});
```

**Detection Method** (Control-driven):
1. Enumerate all CRUD endpoints
2. Check for authorization middleware/decorators
3. Verify ownership validation in controller
4. Test with different user contexts (if possible)

### D4: Deserialization Attacks

Identifies unsafe deserialization in Java, Python, PHP with gadget chain analysis.

**Example Detection (Java)**:

```java
// VULNERABLE: Unsafe deserialization
@PostMapping("/import")
public void importData(@RequestBody byte[] data) {
    ObjectInputStream ois = new ObjectInputStream(
        new ByteArrayInputStream(data)
    );
    Object obj = ois.readObject();
    processData(obj);
}

// SECURE: Use safe formats like JSON
@PostMapping("/import")
public void importData(@RequestBody String jsonData) {
    ObjectMapper mapper = new ObjectMapper();
    DataObject obj = mapper.readValue(jsonData, DataObject.class);
    processData(obj);
}
```

### D5: File Operation Vulnerabilities

Covers path traversal, unrestricted file upload, and arbitrary file read/write.

**Example Detection (PHP)**:

```php
// VULNERABLE: Path traversal
$file = $_GET['file'];
$content = file_get_contents("/var/www/docs/" . $file);
echo $content;

// SECURE: Whitelist validation
$file = basename($_GET['file']); // Remove path components
$allowed_files = ['doc1.txt', 'doc2.txt', 'doc3.txt'];

if (!in_array($file, $allowed_files)) {
    die('Invalid file');
}

$content = file_get_contents("/var/www/docs/" . $file);
echo $content;
```

### D6: SSRF (Server-Side Request Forgery)

Detects URL injection and insufficient protocol restrictions.

**Example Detection (Python)**:

```python
# VULNERABLE: SSRF
@app.route('/fetch')
def fetch_url():
    url = request.args.get('url')
    response = requests.get(url)
    return response.content

# SECURE: Protocol and domain whitelist
import urllib.parse

ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com']

@app.route('/fetch')
def fetch_url():
    url = request.args.get('url')
    parsed = urllib.parse.urlparse(url)
    
    # Check protocol
    if parsed.scheme not in ['http', 'https']:
        return 'Invalid protocol', 400
    
    # Check domain whitelist
    if parsed.netloc not in ALLOWED_DOMAINS:
        return 'Domain not allowed', 400
    
    # Prevent private IP access
    import ipaddress
    try:
        ip = ipaddress.ip_address(parsed.hostname)
        if ip.is_private:
            return 'Private IP not allowed', 400
    except ValueError:
        pass
    
    response = requests.get(url, timeout=5)
    return response.content
```

### D7: Cryptography Issues

Reviews encryption algorithms, key management, and KDF implementations.

**Example Detection (Go)**:

```go
// VULNERABLE: Weak encryption (ECB mode, hardcoded key)
func encrypt(data []byte) []byte {
    key := []byte("1234567890123456") // Hardcoded
    block, _ := aes.NewCipher(key)
    
    encrypted := make([]byte, len(data))
    // ECB mode - encrypts each block identically
    for i := 0; i < len(data); i += aes.BlockSize {
        block.Encrypt(encrypted[i:], data[i:])
    }
    return encrypted
}

// SECURE: AES-GCM with IV and env key
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "io"
    "os"
)

func encrypt(plaintext []byte) ([]byte, error) {
    key := []byte(os.Getenv("ENCRYPTION_KEY")) // From env
    
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    io.ReadFull(rand.Reader, nonce)
    
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}
```

### D8: Configuration Vulnerabilities

Checks for exposed actuators, CORS misconfig, verbose error messages.

**Example Detection (Spring Boot)**:

```yaml
# VULNERABLE: application.yml
management:
  endpoints:
    web:
      exposure:
        include: "*"  # Exposes all actuator endpoints
  endpoint:
    health:
      show-details: always

spring:
  datasource:
    url: jdbc:mysql://localhost/db?user=root&password=admin123

# SECURE: application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info  # Only safe endpoints
  endpoint:
    health:
      show-details: when-authorized

spring:
  datasource:
    url: jdbc:mysql://localhost/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
```

### D9: Business Logic Flaws

Identifies race conditions, mass assignment, state machine bypasses, multi-tenant isolation issues.

**Example Detection (Ruby/Rails)**:

```ruby
# VULNERABLE: Race condition in balance transfer
def transfer
  from_account = Account.find(params[:from_id])
  to_account = Account.find(params[:to_id])
  amount = params[:amount].to_f
  
  # Race condition: check and update not atomic
  if from_account.balance >= amount
    from_account.update(balance: from_account.balance - amount)
    to_account.update(balance: to_account.balance + amount)
  end
end

# SECURE: Database-level atomic transaction with locking
def transfer
  ActiveRecord::Base.transaction do
    from_account = Account.lock.find(params[:from_id])
    to_account = Account.lock.find(params[:to_id])
    amount = params[:amount].to_f
    
    raise InsufficientFunds if from_account.balance < amount
    
    from_account.update!(balance: from_account.balance - amount)
    to_account.update!(balance: to_account.balance + amount)
  end
rescue InsufficientFunds
  render json: { error: 'Insufficient funds' }, status: 400
end
```

### D10: Supply Chain Security

Scans dependencies for known CVEs and version vulnerabilities.

**Example Detection**:

```javascript
// package.json - VULNERABLE
{
  "dependencies": {
    "express": "4.16.0",  // CVE-2022-24999
    "lodash": "4.17.15",  // CVE-2020-8203
    "axios": "0.18.0"     // CVE-2021-3749
  }
}

// package.json - SECURE (updated versions)
{
  "dependencies": {
    "express": "^4.18.2",
    "lodash": "^4.17.21",
    "axios": "^1.6.0"
  }
}
```

## Taint Analysis Flow

The framework uses multi-stage taint tracking:

```
Source (User Input) → Propagation → Sink (Dangerous Function)
     ↓                    ↓              ↓
   @RequestParam      String concat    jdbcTemplate.query()
   request.args       list.append      os.system()
   $_GET              array_merge      eval()
```

**Example Taint Flow (Python)**:

```python
# Traced data flow
def search_users(request):
    # SOURCE: User input
    query = request.GET.get('q')
    
    # PROPAGATION: Through variable assignments
    search_term = query
    sql_query = f"SELECT * FROM users WHERE name LIKE '%{search_term}%'"
    
    # SINK: Dangerous function
    cursor.execute(sql_query)  # SQL Injection!
```

**Detection Output**:

```
[VULN] SQL Injection
  Source: request.GET.get('q') @ line 2
  Flow: query → search_term → sql_query
  Sink: cursor.execute(sql_query) @ line 7
  Sanitization: NONE
  Severity: CRITICAL
  POC: /?q=admin'%20OR%20'1'='1
```

## Using Optional Python Scripts

### Pattern Scanner

```bash
python scripts/pattern_scanner.py /path/to/project --language python --patterns sql_injection,command_injection
```

**Output**:
```json
{
  "findings": [
    {
      "type": "SQL_INJECTION",
      "file": "app/views.py",
      "line": 42,
      "code": "cursor.execute(f\"SELECT * FROM users WHERE id={user_id}\")",
      "severity": "CRITICAL"
    }
  ]
}
```

### Data Flow Analyzer

```bash
python scripts/data_flow_analyzer.py /path/to/project --source request.args --sink execute
```

### Secret Finder

```bash
python scripts/secret_finder.py /path/to/project --output secrets.json
```

**Detects**:
- Hardcoded API keys
- Database credentials
- JWT secrets
- Private keys
- AWS/GCP credentials

### Dependency Analyzer

```bash
python scripts/dependency_analyzer.py /path/to/project --check-cves
```

**Output**:
```
[CRITICAL] lodash@4.17.15 - CVE-2020-8203 (Prototype Pollution)
[HIGH] axios@0.18.0 - CVE-2021-3749 (SSRF)
[MEDIUM] express@4.16.0 - CVE-2022-24999 (DoS)
```

### Full Scan

```bash
python scripts/code_scan.py /path/to/project --mode deep --output report.json
```

**Generates**:
- Vulnerability inventory
- Data flow graphs
- Attack chain analysis
- Remediation recommendations

## Configuration

### Environment Variables

```bash
# API keys for external scanners (optional)
export SNYK_TOKEN="your-snyk-token"
export GITHUB_TOKEN="your-github-token"

# Database credentials for live testing (optional)
export DB_HOST="localhost"
export DB_USER="testuser"
export DB_PASSWORD="testpass"
export DB_NAME="testdb"

# Docker for environment simulation (optional)
export DOCKER_HOST="unix:///var/run/docker.sock"
```

### Audit Configuration (audit_config.yaml)

```yaml
mode: deep  # quick, standard, deep

scope:
  languages: [java, python, javascript]
  frameworks: [spring, django, express]
  dimensions: [D1, D2, D3, D4, D5, D6, D7, D8, D9, D10]

exclusions:
  paths:
    - node_modules/
    - .git/
    - tests/
  files:
    - "*.test.js"
    - "*.spec.py"

validation:
  enable_docker: true
  enable_poc_generation: true
  enable_live_testing: false  # Requires test environment

reporting:
  format: markdown  # markdown, json, html
  include_code_snippets: true
  include_dataflow_diagrams: true
  include_attack_chains: true
```

## Real-World Audit Example

### Project: E-commerce Application (Spring Boot)

```
User: "Audit this e-commerce application for security issues"

AI Agent Execution:

[PHASE 1: RECONNAISSANCE] (2 min)
  ✓ Detected: Spring Boot 2.5.6 + MyBatis + Thymeleaf
  ✓ Architecture: MVC, 3-tier (Controller → Service → DAO)
  ✓ Entry points: 47 REST endpoints, 12 web pages
  ✓ Attack surface: User management, order processing, payment

[PHASE 2: PATTERN MATCHING] (8 min)
  Agent-1 (D1-D3): Scanning injection and auth/authz
    → Found: 3 SQL injection candidates
    → Found: JWT secret in application.properties
    → Found: Missing permission checks in OrderController
  
  Agent-2 (D4-D6): Scanning deserialization and file ops
    → Found: Unsafe ObjectInputStream usage
    → Found: Path traversal in file download
  
  Agent-3 (D7-D10): Scanning crypto and config
    → Found: MD5 password hashing
    → Found: Exposed Actuator endpoints

[PHASE 3: TAINT TRACKING] (15 min)
  [CRITICAL] SQL Injection in OrderController.searchOrders()
    Source: @RequestParam String keyword (line 156)
    Flow: keyword → sql = "SELECT * FROM orders WHERE name LIKE '%" + keyword + "%'"
    Sink: sqlSession.selectList(sql) (line 158)
    Sanitization: NONE
    
    POC Test:
      GET /api/orders/search?keyword=test'%20OR%20'1'='1
      Response: Returns all orders (bypasses WHERE clause)
    
    Evidence:
      ✓ User input reaches SQL sink
      ✓ No parameterized query
      ✓ Live test confirms exploitation
  
  [CRITICAL] IDOR in OrderController.deleteOrder()
    Missing Check: User ownership validation
    Endpoint: DELETE /api/orders/{id}
    
    POC Test:
      User A (ID: 123) can delete User B's order (ID: 456)
      DELETE /api/orders/456
      Response: 200 OK (should be 403 Forbidden)
    
    Evidence:
      ✓ No authorization filter
      ✓ No ownership check in service layer
      ✓ Live test confirms horizontal privilege escalation

[PHASE 4: ATTACK CHAIN] (5 min)
  Attack Chain: SQL Injection → Data Extraction → Account Takeover
    1. SQL injection extracts user credentials
       /?keyword=' UNION SELECT username,password FROM users--
    2. Crack MD5 hashes (weak algorithm)
    3. Login as admin
    4. Use IDOR to delete all orders

[PHASE 5: REPORTING]
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  SECURITY AUDIT REPORT
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Project: E-commerce Application
  Date: 2026-06-06
  Scope: Full codebase (874 files, 45K LOC)
  
  SUMMARY
    CRITICAL: 4 issues
    HIGH: 6 issues
    MEDIUM: 8 issues
    LOW: 3 issues
  
  CRITICAL FINDINGS
  
  [C-1] SQL Injection in Order Search
    File: OrderController.java:156-158
    Severity: CRITICAL
    CWE: CWE-89
    
    Vulnerable Code:
      String sql = "SELECT * FROM orders WHERE name LIKE '%" + keyword + "%'";
      return sqlSession.selectList(sql);
    
    Fix:
      String sql = "SELECT * FROM orders WHERE name LIKE CONCAT('%', #{keyword}, '%')";
      return sqlSession.selectList("searchOrders", keyword);
    
    Or use parameterized MyBatis XML:
      <select id="searchOrders" resultType="Order">
        SELECT * FROM orders WHERE name LIKE CONCAT('%', #{keyword}, '%')
      </select>
  
  [C-2] IDOR - Unauthorized Order Deletion
    File: OrderController.java:201-205
    Severity: CRITICAL
    CWE: CWE-639
    
    Vulnerable Code:
      @DeleteMapping("/orders/{id}")
      public void deleteOrder(@PathVariable Long id) {
          orderService.deleteOrder(id);
      }
    
    Fix:
      @DeleteMapping("/orders/{id}")
      public void deleteOrder(@PathVariable Long id, 
                              @AuthenticationPrincipal User user) {
          Order order = orderService.getOrder(id);
          if (order == null) {
              throw new NotFoundException();
          }
          if (!order.getUserId().equals(user.getId())) {
              throw new ForbiddenException();
          }
          orderService.deleteOrder(id);
      }
  
  [...more findings...]
  
  REMEDIATION PRIORITY
    1. Fix SQL Injection (C-1) - Immediate
    2. Fix IDOR (C-2) - Immediate
    3. Upgrade password hashing to bcrypt (C-3) - 1 week
    4. Disable Actuator endpoints (H-1) - 1 week
    5. Implement CSRF protection (H-2) - 2 weeks
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## WooYun Case Library Integration

The framework includes 100+ real-world vulnerability cases from WooYun (2010-2016).

**Example Usage**:

```
User: "Show me examples of authentication bypass in PHP applications"

AI Response:
[WOOYUN CASE #WY-2014-67234]
  Type: Authentication Bypass
  Framework: Discuz X3.1
  
  Vulnerability:
    Cookie forgery due to predictable salt generation
  
  Exploit:
    1. Obtain user's username and uid
    2. Generate cookie: md5(md5(uid).salt)
    3. Salt is predictable: md5(timestamp)
    4. Forge admin cookie and bypass login
  
  Code Location: /source/function/function_core.php:184
  
  Fix: Use cryptographically secure random salt
```

## Anti-Hallucination Mechanisms

To ensure accuracy, the framework enforces:

1. **Evidence Requirement**: Every finding must include:
   - Source code location (file:line)
   - Complete data flow path
   - POC or test validation

2. **Path Completeness**: Reject findings with gaps like:
   ```
   Source: request.GET.get('q')
   [GAP: Missing propagation steps]
   Sink: cursor.execute(query)
   ```

3. **Live Testing** (when enabled):
   ```python
   # Docker container spawned for safe testing
   docker run --rm -v $(pwd):/app test-env python test_exploit.py
   ```

4. **Multi-Agent Verification**: Critical findings reviewed by 2+ agents

## Troubleshooting

### Issue: Too many false positives

**Solution**: Enable stricter validation
```yaml
validation:
  enable_live_testing: true
  require_evidence: strict
  min_confidence: 0.8
```

### Issue: Missing framework-specific vulnerabilities

**Solution**: Add custom rules
```python
# resources/rules/custom_rules.py
CUSTOM_PATTERNS = {
    'spring_spel_injection': {
        'pattern': r'@Value\(".*#\{.*\}".*\)',
        'severity': 'CRITICAL',
        'description': 'SpEL injection in @Value annotation'
    }
}
```

### Issue: Slow audit on large codebases

**Solution**: Use quick mode or limit scope
```yaml
mode: quick
scope:
  dimensions: [D1, D2, D3]  # Focus on injection and auth
  max_files: 500
```

### Issue: Cannot access private repositories

**Solution**: Configure Git credentials
```bash
git config --global credential.helper store
export GITHUB_TOKEN="ghp_your_token_here"
```

## Advanced Features

### Custom Sink Definitions

```python
# Add framework-specific sinks
CUSTOM_SINKS = {
    'java': {
        'mybatis_injection': [
            'sqlSession.selectList',
            'sqlSession.selectOne',
            '${}' # MyBatis inline parameter
        ]
    }
}
```

### Attack Chain Templates

```yaml
attack_chains:
  - name: "SQL Injection to RCE"
    steps:
      - vuln: SQL_INJECTION
        action: Extract database credentials
      - vuln: WEAK_CRYPTO
        action: Crack password hashes
      - vuln: COMMAND_INJECTION
        action: Execute system commands
```

### Docker Environment Simulation

```bash
# Automated vulnerability validation in isolated container
python scripts/docker_verification.py \
  --vuln-type sql_injection \
  --payload "' OR '1'='1" \
  --project /path/to/project
```

**Output**:
```
[DOCKER] Building test environment...
[DOCKER] Container ID: a3f9b2c1
[TEST] Sending payload: ' OR '1'='1
[RESULT] Vulnerability confirmed - SQL error in response
[CLEANUP] Container removed
```

## Report Templates

### Quick Report (CI/CD)

```markdown
## Security Scan Results

**Scan Date**: 2026-06-06  
**Mode**: Quick  
**Duration**: 7 minutes

### Summary
- **CRITICAL**: 2
- **HIGH**: 3
- **MEDIUM**: 1

### Action Required
1. Fix SQL injection in `UserController.java:142`
2. Rotate exposed AWS key in `config.py:8`
3. Update `lodash` to 4.17.21 (CVE-2020-8203)
```

### Full Report (Pentest)

Includes:
- Executive summary
- Detailed findings with POCs
- Data flow diagrams (Mermaid)
- Attack chain visualization
- Remediation roadmap
- Compliance mapping (OWASP, CWE, PCI-DSS)

## Integration Examples

### GitHub Actions

```yaml
name: Security Audit
on: [push]
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run dfyx audit
        run: |
          pip install -r requirements.txt
          python scripts/code_scan.py . --mode quick --output scan.json
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: security-scan
          path: scan.json
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit
python scripts/pattern_scanner.py . --severity critical,high
if [ $? -ne 0 ]; then
  echo "Security issues detected. Commit aborted."
  exit 1
fi
```

## Learning Resources

- **OWASP Testing Guide**: Attack surface analysis methodology
- **WooYun Case Studies**: Real-world vulnerability patterns
- **CWE Database**: Weakness classification and mitigation
- **Framework Security Guides**: Spring Security, Django Security, etc.

## Best Practices

1. **Run audits early**: Integrate into development workflow
2. **Prioritize fixes**: CRITICAL → HIGH → MEDIUM
3. **Verify fixes**: Re-run audit after remediation
4. **Track metrics**: Vulnerability trends over time
5. **Educate team**: Share findings and secure coding patterns

---

**Note**: This skill is for security research and education. Always obtain authorization before testing systems you don't own.
