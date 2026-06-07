---
name: dfyx-code-security-review
description: AI-powered white-box security audit framework with deep data-flow analysis, taint tracking, and attack chain reconstruction for 9 languages
triggers:
  - audit this codebase for security vulnerabilities
  - perform a security code review using dfyx
  - run deep security audit on this project
  - check for SQL injection and other security issues
  - analyze this code for OWASP Top 10 vulnerabilities
  - conduct white-box security testing
  - find security flaws in this application
  - perform taint analysis on user inputs
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

**dfyx_code_security_review** is an expert-level code security audit framework designed for AI assistants (Claude Code, Cursor, etc.). It employs white-box static analysis with a standardized five-phase protocol to systematically discover and validate security vulnerabilities in source code through deep data-flow analysis and business logic understanding.

## Core Capabilities

- **9 Languages**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 Security Dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **Dual-Track Audit Model**: Sink-driven + Control-driven + Config-driven analysis
- **Five-Phase Protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting
- **WooYun Case Library**: Real-world vulnerability patterns from 2010-2016

## Installation

### For AI Code Assistants

Clone this skill into your AI client's skills directory:

```bash
# Claude Code
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git ~/.claude/skills/dfyx-code-security-review

# Cursor
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git ~/Library/Application\ Support/Cursor/Skills/dfyx-code-security-review
```

### Python Dependencies (Optional Scripts)

```bash
cd dfyx-code-security-review
pip install -r requirements.txt
```

**requirements.txt**:
```
tree-sitter>=0.20.0
tree-sitter-python>=0.20.0
tree-sitter-java>=0.20.0
pyyaml>=6.0
jinja2>=3.1.0
```

## Five-Phase Audit Protocol

### Phase 1: Reconnaissance (10%)

**Objective**: Map architecture, identify attack surface, locate entry points

**Methodology**:
1. Identify tech stack (frameworks, language versions, dependencies)
2. Map authentication/authorization mechanisms
3. Enumerate entry points (HTTP routes, CLI commands, API endpoints)
4. Catalog sensitive data flows

**Example Output**:
```
[RECON] Spring Boot 2.7.3 + MyBatis 3.5 + Shiro 1.9
[ENTRY_POINTS] 47 HTTP endpoints, 12 @RequestMapping without auth
[ATTACK_SURFACE] 
  - UserController: /api/user/* (CRUD operations)
  - FileController: /api/file/upload, /api/file/download
  - AdminController: /api/admin/* (requires ROLE_ADMIN)
```

### Phase 2: Pattern Scanning (30%)

**Objective**: Identify high-risk code patterns using parallel agent scanning

**Dual-Track Approach**:

**Track 1: Sink-Driven (Injection/RCE)**
```python
# Pattern: Find dangerous sinks first
dangerous_sinks = {
    'sql_injection': ['executeQuery', 'createNativeQuery', 'createSQLQuery'],
    'command_injection': ['Runtime.exec', 'ProcessBuilder', 'os.system'],
    'template_injection': ['Velocity.evaluate', 'Freemarker.process'],
}

# Then trace backward to sources
def trace_to_source(sink_location, taint_sources):
    # Find data flow from user input to sink
    pass
```

**Track 2: Control-Driven (Authorization)**
```python
# Pattern: Enumerate endpoints, verify security controls
def check_authorization_coverage(endpoints):
    for endpoint in endpoints:
        if not has_auth_annotation(endpoint):
            # Missing security control = vulnerability
            yield VulnerabilityReport(
                type='MISSING_AUTHORIZATION',
                location=endpoint,
                severity='HIGH'
            )
```

**Example Pattern Detection**:

```java
// Java SQL Injection Pattern
@GetMapping("/user/search")
public List<User> searchUser(String keyword) {
    // DANGEROUS: Direct string concatenation
    String sql = "SELECT * FROM users WHERE name LIKE '%" + keyword + "%'";
    return jdbcTemplate.query(sql, new UserRowMapper());
}

// Detection Rule (resources/rules/sql_injection_rules.md):
// 1. Find SQL execution methods: query(), execute(), createNativeQuery()
// 2. Check if parameter involves string concatenation
// 3. Verify no PreparedStatement/parameterized query
// 4. Trace keyword source → HTTP parameter → unsanitized
```

### Phase 3: Deep Taint Tracking (40%)

**Objective**: Validate data flows from sources to sinks with actual test verification

**Enhanced Taint Analysis** (see `resources/knowledge/taint_analysis_enhanced.md`):

```python
class TaintAnalyzer:
    def analyze_data_flow(self, source, sink):
        """
        Trace data flow with sanitization awareness
        
        Sources: HTTP params, file reads, database queries
        Sanitizers: escapeHtml(), validate(), sanitize()
        Sinks: SQL exec, command exec, template render
        """
        path = self.find_path(source, sink)
        
        # Check for sanitization in path
        for node in path:
            if self.is_sanitizer(node):
                if self.sanitizer_is_effective(node, sink.type):
                    return None  # Flow is safe
                else:
                    # Ineffective sanitizer = vulnerability
                    return VulnerabilityReport(
                        type='SANITIZATION_BYPASS',
                        path=path,
                        severity='HIGH'
                    )
        
        # No sanitization found = vulnerability
        return VulnerabilityReport(
            type=sink.type,
            path=path,
            severity='CRITICAL'
        )
```

**Real Example - Python Command Injection**:

```python
# Vulnerable code in file_processor.py
from flask import Flask, request
import os

@app.route('/convert')
def convert_file():
    filename = request.args.get('file')
    output = request.args.get('output', 'result.pdf')
    
    # VULNERABLE: User input directly in shell command
    cmd = f"convert {filename} {output}"
    os.system(cmd)  # SINK: Command execution
    
    return {'status': 'done'}

# Taint Flow:
# SOURCE: request.args.get('file') [Line 6]
#   ↓ (no sanitization)
# SINK: os.system(cmd) [Line 10]
#
# POC: /convert?file=test.jpg;whoami;.pdf
# Result: Command injection confirmed
```

**Actual Test Verification**:
```python
# scripts/vulnerability_validator.py
def validate_command_injection(vuln_report):
    """
    Verify vulnerability through actual testing
    """
    # 1. Setup test environment (Docker container)
    container = setup_test_env(vuln_report.project)
    
    # 2. Craft exploit payload
    payload = "test.jpg;cat /etc/passwd;.pdf"
    
    # 3. Execute test
    response = container.request(
        '/convert',
        params={'file': payload}
    )
    
    # 4. Verify exploitation
    if 'root:x:0:0' in response.text:
        return ValidationResult(
            confirmed=True,
            evidence=response.text[:500],
            impact='Remote command execution confirmed'
        )
    
    return ValidationResult(confirmed=False)
```

### Phase 4: Validation & Attack Chain (15%)

**Objective**: Confirm vulnerabilities and construct multi-step attack scenarios

**Attack Chain Example**:

```yaml
# Chain: Information Disclosure → Authentication Bypass → Privilege Escalation
steps:
  - id: 1
    type: INFORMATION_DISCLOSURE
    location: /api/debug/config
    finding: "Actuator endpoint exposed without authentication"
    impact: "Reveals JWT secret key in environment variables"
    
  - id: 2
    type: AUTHENTICATION_BYPASS
    dependency: step_1
    location: JwtTokenProvider.java:45
    finding: "JWT secret leaked from step 1"
    exploit: |
      import jwt
      token = jwt.encode(
          {'user': 'admin', 'role': 'ADMIN'}, 
          secret='leaked_secret_from_step1',
          algorithm='HS256'
      )
    
  - id: 3
    type: PRIVILEGE_ESCALATION
    dependency: step_2
    location: AdminController.java
    finding: "Admin endpoints only check JWT validity, not role claims"
    exploit: "Use forged admin token to access /api/admin/users/delete"
    
total_impact: "Unauthenticated attacker can delete all users"
```

### Phase 5: Structured Reporting (5%)

**Report Sections**:

1. **Executive Summary**: Total vulnerabilities by severity
2. **Critical Findings**: Detailed analysis with code snippets
3. **Data Flow Diagrams**: Visual representation of taint paths
4. **Reproduction Steps**: Step-by-step POC
5. **Remediation Guidance**: Code-level fixes with before/after

**Example Report Excerpt**:

```markdown
## Critical Finding #1: SQL Injection in User Search

**Severity**: CRITICAL (CVSS 9.8)
**Location**: `UserController.java:34`
**CWE**: CWE-89 (SQL Injection)

### Vulnerable Code
```java
String sql = "SELECT * FROM users WHERE name LIKE '%" + keyword + "%'";
return jdbcTemplate.query(sql, new UserRowMapper());
```

### Data Flow
```
HTTP Request (/api/user/search?keyword=admin)
  ↓
UserController.searchUser(String keyword) [Line 33]
  ↓ (no sanitization)
String concatenation [Line 34]
  ↓
jdbcTemplate.query(sql) [Line 35] ← SINK
```

### Exploitation
```bash
# POC: Boolean-based blind SQL injection
curl "http://target/api/user/search?keyword=admin' AND 1=1--"  # Returns results
curl "http://target/api/user/search?keyword=admin' AND 1=2--"  # Returns empty

# Impact: Full database extraction possible
```

### Remediation
```java
// Use parameterized query
String sql = "SELECT * FROM users WHERE name LIKE ?";
return jdbcTemplate.query(sql, new UserRowMapper(), "%" + keyword + "%");
```
```

## 10 Security Dimensions Coverage

### D1: Injection Vulnerabilities

**Covered Types**: SQL, NoSQL, LDAP, OS Command, Template (SSTI), Expression (SpEL/OGNL/JNDI)

**Detection Strategy**:
```python
# Pattern: Find sinks → Trace to sources → Verify no sanitization
injection_patterns = {
    'sql': {
        'sinks': ['execute', 'executeQuery', 'query', 'createNativeQuery'],
        'sources': ['request.getParameter', '@RequestParam', '@PathVariable'],
        'sanitizers': ['PreparedStatement', 'NamedParameterJdbcTemplate']
    },
    'command': {
        'sinks': ['Runtime.exec', 'ProcessBuilder', 'os.system', 'subprocess.call'],
        'sources': ['user_input', 'request_data'],
        'sanitizers': ['shlex.quote', 'quote_plus', 'whitelist_validation']
    }
}
```

**Real Case - Spring SSTI**:
```java
// Vulnerable: User input in template name
@GetMapping("/render")
public String renderTemplate(@RequestParam String template) {
    return template;  // Spring resolves this as view name
}

// Exploit: /render?template=freemarker:${7*7}
// Detection: Check if @RequestParam returns to view resolver without whitelist
```

### D2: Authentication Flaws

**Covered Issues**: Weak session management, JWT vulnerabilities, filter chain bypass

**Detection Pattern**:
```python
def check_jwt_security(codebase):
    jwt_config = find_jwt_configuration(codebase)
    
    issues = []
    
    # Check 1: Weak secret
    if jwt_config.secret.length < 32:
        issues.append("JWT secret too short (< 256 bits)")
    
    # Check 2: Algorithm confusion
    if 'none' in jwt_config.allowed_algorithms:
        issues.append("Algorithm 'none' allowed")
    
    # Check 3: No expiration
    if not jwt_config.validates_expiration:
        issues.append("Token expiration not validated")
    
    return issues
```

### D3: Authorization Bypass

**Covered Issues**: Missing access control, IDOR, horizontal/vertical privilege escalation

**Example - IDOR Detection**:
```python
# Pattern: Find CRUD operations without owner check
def detect_idor(endpoints):
    for endpoint in endpoints:
        if endpoint.method in ['DELETE', 'PUT', 'PATCH']:
            # Check if owner verification exists
            if not has_owner_check(endpoint):
                yield IDORVulnerability(
                    endpoint=endpoint.path,
                    parameter=endpoint.id_param,
                    exploit=f"User can modify resource belonging to others"
                )

# Example vulnerable code:
@DeleteMapping("/api/post/{id}")
public void deletePost(@PathVariable Long id) {
    postRepository.deleteById(id);  // No ownership check!
}
```

### D4: Deserialization

**Covered Languages**: Java (ObjectInputStream), Python (pickle), PHP (unserialize)

**Java Gadget Chain Detection**:
```python
dangerous_classes = [
    'org.apache.commons.collections.Transformer',
    'org.springframework.beans.factory.ObjectFactory',
    'com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl'
]

def check_deserialization(code):
    if uses_objectinputstream(code):
        if not has_whitelist_filter(code):
            # Check for gadget chain libraries in dependencies
            if has_commons_collections(dependencies):
                return "CRITICAL: Unsafe deserialization with gadget chain available"
```

### D5: File Operations

**Covered Issues**: Path traversal, unrestricted upload, arbitrary file read/write

**Real Example - Path Traversal**:
```python
# Vulnerable Flask code
@app.route('/download')
def download():
    filename = request.args.get('file')
    return send_file(f'/uploads/{filename}')  # VULNERABLE

# Detection:
# 1. Find file operation functions: send_file, open(), File()
# 2. Check if path includes user input
# 3. Verify no path normalization/whitelist
# 4. POC: /download?file=../../../../etc/passwd
```

### D6: SSRF

**Detection Strategy**:
```python
# Pattern: URL from user input → HTTP request
def detect_ssrf(code):
    http_functions = ['requests.get', 'urllib.request', 'HttpClient.execute']
    
    for func_call in find_function_calls(code, http_functions):
        url_param = func_call.arguments[0]
        
        if is_user_controlled(url_param):
            if not has_url_whitelist(func_call):
                yield SSRFVulnerability(
                    location=func_call.location,
                    sink=func_call.function,
                    source=get_taint_source(url_param)
                )
```

### D7: Cryptography

**Covered Issues**: Hard-coded keys, weak algorithms, insecure random

**Detection Rules**:
```python
crypto_issues = {
    'weak_algorithm': ['MD5', 'SHA1', 'DES', 'RC4'],
    'weak_mode': ['ECB'],
    'weak_random': ['Random()', 'Math.random()'],
    'hardcoded_key': r'(key|password|secret)\s*=\s*["\'][^"\']{8,}["\']'
}

# Example vulnerable code:
cipher = AES.new(b'hardcoded_key123', AES.MODE_ECB)  # 2 issues!
```

### D8: Security Misconfiguration

**Covered Areas**: Debug mode, CORS, security headers, actuator exposure

**Spring Boot Actuator Check**:
```yaml
# Check application.yml for dangerous settings
management:
  endpoints:
    web:
      exposure:
        include: "*"  # VULNERABLE: All endpoints exposed
  security:
    enabled: false  # CRITICAL: No authentication required
```

### D9: Business Logic

**Covered Flaws**: Race conditions, mass assignment, state machine bypass, multi-tenant isolation

**Race Condition Example**:
```python
# Vulnerable: TOCTOU in balance check
def withdraw(user_id, amount):
    balance = get_balance(user_id)  # Time of Check
    if balance >= amount:
        time.sleep(0.1)  # Vulnerability window
        update_balance(user_id, balance - amount)  # Time of Use

# Detection: Find check-then-act patterns without locking
# Exploit: Send parallel requests to overdraw account
```

### D10: Supply Chain

**Covered Checks**: Known CVEs, outdated dependencies, typosquatting

**Dependency Analysis**:
```python
# scripts/dependency_analyzer.py
def check_dependencies(project_path):
    deps = parse_dependencies(project_path)  # requirements.txt, pom.xml, etc.
    
    for dep in deps:
        # Check against vulnerability databases
        cves = query_nvd(dep.name, dep.version)
        
        if cves:
            yield DependencyVulnerability(
                package=dep.name,
                version=dep.version,
                cves=cves,
                fix_version=get_patched_version(dep.name)
            )

# Example output:
# CRITICAL: Apache Log4j 2.14.1 → CVE-2021-44228 (Log4Shell)
# Fix: Upgrade to 2.17.1
```

## Scan Modes

### Quick Mode (5-10 min)
```python
# Focus: High-severity, low-false-positive checks
quick_checks = [
    'hardcoded_secrets',
    'sql_injection_obvious',
    'command_injection_obvious',
    'known_cves',
    'debug_mode_enabled'
]
```

### Standard Mode (30-60 min)
```python
# Coverage: OWASP Top 10 + authentication/authorization
standard_checks = quick_checks + [
    'auth_bypass',
    'idor',
    'xss',
    'ssrf',
    'path_traversal',
    'insecure_deserialization'
]
```

### Deep Mode (1-3 hours)
```python
# Full coverage: All 10 dimensions + attack chains + actual testing
deep_checks = standard_checks + [
    'business_logic',
    'race_conditions',
    'attack_chain_analysis',
    'multi_step_exploits',
    'docker_validation'  # Spin up test environment
]
```

## Usage Examples

### Example 1: Audit Spring Boot Project

```python
# User prompt: "Audit this Spring Boot project for security issues"

# AI Agent Response:
"""
[MODE] standard
[RECON] Scanning project structure...
  - Framework: Spring Boot 2.7.3
  - Database: PostgreSQL with JPA
  - Auth: Spring Security + JWT
  - Files: 234 Java files, 12 controllers

[PHASE 1] Architecture Analysis
  ✓ Mapped 47 HTTP endpoints
  ✓ Identified 3 authentication filters
  ⚠ Found 8 endpoints without @PreAuthorize

[PHASE 2] Pattern Scanning (3 agents)
  Agent-D1 (Injection): Scanning SQL/Command injection...
    ⚠ Found 3 potential SQL injection points
  Agent-D3 (AuthZ): Checking authorization coverage...
    ⚠ Found 8 missing access controls
  Agent-D7 (Crypto): Analyzing cryptographic usage...
    ⚠ Found hardcoded JWT secret

[PHASE 3] Taint Analysis
  Analyzing UserController.searchUser()...
    SOURCE: @RequestParam String keyword [Line 45]
    FLOW: String concatenation [Line 47]
    SINK: jdbcTemplate.query(sql) [Line 48]
    ✗ No sanitization found
    ✓ CONFIRMED: SQL Injection (CRITICAL)

[PHASE 4] Actual Test Verification
  Setting up test environment (Docker)...
  Testing SQL injection POC...
    Request: /api/user/search?keyword=admin' UNION SELECT password FROM admin_users--
    Response: {"users":[{"password":"admin_hash_exposed"}]}
    ✓ EXPLOITATION CONFIRMED

[REPORT] Security Audit Complete
  Total Issues: 12
    - CRITICAL: 2 (SQL Injection, Hard-coded Secret)
    - HIGH: 5 (Missing Authorization, IDOR)
    - MEDIUM: 4 (Weak Crypto, Information Disclosure)
    - LOW: 1 (Verbose Error Messages)

Generating detailed report...
"""
```

### Example 2: Quick Scan for Secrets

```python
# scripts/secret_finder.py
import re

def find_secrets(file_path):
    """
    Quick scan for hardcoded secrets
    """
    patterns = {
        'aws_key': r'AKIA[0-9A-Z]{16}',
        'private_key': r'-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----',
        'jwt_secret': r'(jwt_secret|jwtSecret)\s*=\s*["\'][^"\']{16,}["\']',
        'api_key': r'(api_key|apiKey)\s*=\s*["\'][a-zA-Z0-9]{32,}["\']',
    }
    
    with open(file_path) as f:
        content = f.read()
        
        for secret_type, pattern in patterns.items():
            matches = re.finditer(pattern, content, re.IGNORECASE)
            for match in matches:
                yield {
                    'type': secret_type,
                    'file': file_path,
                    'line': content[:match.start()].count('\n') + 1,
                    'value': match.group()[:20] + '...'  # Truncate for safety
                }

# Usage:
# python scripts/secret_finder.py /path/to/project
```

### Example 3: Custom Validation Rule

```python
# scripts/custom_validators/graphql_injection.py
"""
Custom validator for GraphQL injection
"""

def validate_graphql_query(code_snippet):
    """
    Check if user input is directly interpolated into GraphQL query
    """
    pattern = r'(query|mutation)\s*=\s*[f"\'].*?\{[^}]*\+.*?\}.*?[f"\']'
    
    if re.search(pattern, code_snippet):
        # Found string interpolation in GraphQL query
        return {
            'vulnerability': 'GRAPHQL_INJECTION',
            'severity': 'HIGH',
            'description': 'User input directly interpolated into GraphQL query',
            'remediation': 'Use parameterized queries with variables',
            'example_fix': '''
# Before (Vulnerable):
query = f"{{ user(id: {user_id}) {{ name email }} }}"

# After (Safe):
query = "query($id: ID!) { user(id: $id) { name email } }"
variables = {"id": user_id}
'''
        }
    
    return None
```

## Configuration

### Audit Configuration File

Create `.dfyx-audit.yml` in project root:

```yaml
# Audit scope configuration
scope:
  include:
    - "src/**/*.java"
    - "src/**/*.py"
  exclude:
    - "src/test/**"
    - "**/*Test.java"
    - "node_modules/**"

# Scan mode: quick | standard | deep
mode: standard

# Security dimensions to check (D1-D10)
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

# Severity threshold for reporting
min_severity: MEDIUM

# Enable actual test validation (requires Docker)
enable_validation: true

# Parallel agent count
agents: 3

# Custom rules directory
custom_rules: ./.dfyx-rules/

# Report format: markdown | json | html
report_format: markdown
report_output: ./security-audit-report.md
```

### Environment Variables

```bash
# Docker configuration for test validation
export DFYX_DOCKER_ENABLED=true
export DFYX_DOCKER_NETWORK=audit-network

# Database for dependency CVE lookup
export NVD_API_KEY=${NVD_API_KEY}  # Get from https://nvd.nist.gov/developers/request-an-api-key

# Logging
export DFYX_LOG_LEVEL=INFO
export DFYX_LOG_FILE=./audit.log
```

## Anti-Hallucination Mechanisms

The framework includes strict validation to prevent false positives:

### 1. Evidence Requirements

```python
class VulnerabilityReport:
    def __init__(self, type, location, severity):
        self.type = type
        self.location = location
        self.severity = severity
        self.evidence = {}  # REQUIRED
    
    def validate(self):
        """
        Report is invalid unless it includes:
        - Exact file path and line number
        - Code snippet showing vulnerability
        - Complete data flow from source to sink
        - Actual test validation result (for CRITICAL/HIGH)
        """
        required_evidence = [
            'file_path',
            'line_number',
            'code_snippet',
            'data_flow',
        ]
        
        if self.severity in ['CRITICAL', 'HIGH']:
            required_evidence.append('validation_result')
        
        for field in required_evidence:
            if field not in self.evidence:
                raise InvalidReportError(f"Missing evidence: {field}")
```

### 2. Path Completeness Check

```python
def validate_taint_path(path):
    """
    Ensure data flow path is complete and realistic
    """
    # Must have clear source
    if not path.source or not is_user_controlled(path.source):
        return False
    
    # Each step must be reachable
    for i in range(len(path.steps) - 1):
        if not is_data_flow_possible(path.steps[i], path.steps[i+1]):
            return False
    
    # Must reach actual sink
    if not is_dangerous_sink(path.sink):
        return False
    
    return True
```

### 3. Actual Test Validation

For CRITICAL and HIGH severity findings, require actual exploitation:

```python
def require_actual_test(vulnerability):
    """
    Don't report CRITICAL/HIGH without actual test
    """
    if vulnerability.severity in ['CRITICAL', 'HIGH']:
        # Spin up Docker test environment
        test_env = create_test_environment(vulnerability.project)
        
        # Attempt exploitation
        result = test_env.exploit(vulnerability.poc)
        
        if not result.success:
            # Downgrade severity or reject finding
            vulnerability.severity = 'POTENTIAL_' + vulnerability.severity
            vulnerability.note = "Could not verify through actual testing"
        else:
            vulnerability.evidence['test_result'] = result.output
```

## Common Patterns

### Pattern 1: Multi-Step Attack Chain

```python
# Combine multiple low-severity issues into critical chain
def build_attack_chain(vulnerabilities):
    chains = []
    
    # Example: Info Disclosure + Auth Bypass
    info_leak = find_vuln_by_type(vulnerabilities, 'INFORMATION_DISCLOSURE')
    auth_bypass = find_vuln_by_type(vulnerabilities, 'AUTHENTICATION_BYPASS')
    
    if info_leak and auth_bypass:
        # Check if info leak enables auth bypass
        if info_leak.reveals('secret_key') and auth_bypass.requires('secret_key'):
            chains.append(AttackChain(
                steps=[info_leak, auth_bypass],
                impact='Complete authentication bypass',
                severity='CRITICAL'  # Escalated from individual findings
            ))
    
    return chains
```

### Pattern 2: Framework-Specific Checks

```python
# Spring Boot specific patterns
def audit_spring_boot(project):
    # Check 1: Actuator exposure
    if has_actuator_dependency(project):
        config = find_file(project, 'application.yml')
        if 'management.endpoints.web.exposure.include: "*"' in config:
            yield Vulnerability('ACTUATOR_EXPOSED', severity='HIGH')
    
    # Check 2: SpEL injection in @Value
    for annotation in find_annotations(project, '@Value'):
        if contains_user_input(annotation.value):
            yield Vulnerability('SPEL_INJECTION', severity='CRITICAL')
    
    # Check 3: JPA query injection
    for method in find_repository_methods(project):
        if method.has_annotation('@Query') and method.uses_string_concat():
            yield Vulnerability('JPA_INJECTION', severity='HIGH')
```

### Pattern 3: Language-Specific Sinks

```python
# Python-specific dangerous functions
python_sinks = {
    'command_injection': [
        'os.system',
        'os.popen',
        'subprocess.call',
        'subprocess.Popen',
        'eval',
        'exec',
    ],
    'sql_injection': [
        'cursor.execute',
        'connection.execute',
        'db.session.execute',
    ],
    'template_injection': [
        'Template().render',
        'jinja2.Template',
        'render_template_string',
    ],
    'deserialization': [
        'pickle.loads',
        'yaml.load',  # Without safe_load
        'marshal.loads',
    ]
}
```

## Troubleshooting

### Issue 1: High False Positive Rate

**Symptom**: Many reported vulnerabilities are not actually exploitable

**Solution**:
```yaml
# Enable stricter validation in .dfyx-audit.yml
validation:
  require_actual_test: true  # Require Docker validation
  require_complete_path: true  # Reject incomplete data flows
  min_confidence: 0.8  # Only report high-confidence findings
```

### Issue 2: Missing Vulnerabilities

**Symptom**: Known vulnerabilities not detected

**Debug Steps**:
```python
# 1. Check if file is in scope
python scripts/code_scan.py --debug --file suspicious_file.py

#
