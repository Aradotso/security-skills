---
name: dfyx-code-security-review
description: AI-powered multi-language code security auditing with deep data flow analysis, taint tracking, and business logic vulnerability detection
triggers:
  - audit this codebase for security vulnerabilities
  - perform a security code review
  - scan for security issues in this project
  - check for SQL injection and other vulnerabilities
  - run a deep code security audit
  - find security vulnerabilities using dfyx
  - analyze code for OWASP Top 10 issues
  - perform static security analysis with data flow tracking
---

# dfyx_code_security_review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

**dfyx_code_security_review** is an expert-level code security auditing framework designed for AI coding agents. It implements a five-phase white-box static analysis methodology that combines pattern matching, deep data flow analysis, taint tracking, and business logic verification to systematically discover and validate security vulnerabilities in source code.

## What It Does

This skill enables AI agents to perform comprehensive security audits across 9 programming languages (Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust) and 14 frameworks (Spring Boot, Django, Flask, FastAPI, Express, Laravel, Rails, ASP.NET Core, etc.).

**Core Capabilities:**
- **10 Security Dimensions**: Injection (SQL/Cmd/SSTI), Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **Triple-Track Audit Model**: Sink-driven (injection/RCE), Control-driven (authorization/business logic), Config-driven (configuration/crypto)
- **Five-Phase Protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting
- **Real Vulnerability Cases**: 200+ WooYun vulnerability cases (2010-2016) for reference patterns

## Installation

This skill is designed to be used directly by AI coding agents (Claude Code, Cursor, etc.) by loading this SKILL.md file. No separate installation required.

For the optional Python helper scripts:

```bash
# Clone the repository
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
cd skill-dfyx_code_security_review

# Install Python dependencies
pip install -r requirements.txt
```

**requirements.txt contents:**
```
libcst>=1.1.0
tree-sitter>=0.20.0
tree-sitter-python>=0.20.0
tree-sitter-java>=0.20.0
tree-sitter-javascript>=0.20.0
pyyaml>=6.0
jinja2>=3.1.2
```

## Audit Modes

| Mode | Use Case | Scope | Duration |
|------|----------|-------|----------|
| **Quick** | CI/CD, small projects | High-severity, secrets, dependency CVEs | 5-10 min |
| **Standard** | Regular audits | OWASP Top 10, auth/authz, crypto | 30-60 min |
| **Deep** | Critical projects, pentest prep | Full coverage, attack chains, business logic | 1-3 hours |

## Usage Pattern

### Triggering an Audit

User request examples:
```
"Audit this project for security vulnerabilities"
"Perform a deep security review on /path/to/project"
"Check for SQL injection and authorization issues"
```

### AI Agent Response Flow

The AI agent will follow this standardized protocol:

```
[MODE] deep
[RECON] Analyzing codebase...
  - Language: Python
  - Framework: Django 3.2
  - Files: 342 Python files
  - Entry points: 45 views, 12 API endpoints
[PLAN] 
  - 3 parallel agents
  - Coverage: D1-D10 (all dimensions)
  - Estimated: 80 turns, 5-12 vulnerabilities
[SCOPE] Focus areas:
  - Authentication: JWT implementation
  - Authorization: View-level permissions
  - Injection: ORM query construction
  - Business Logic: Payment flow

Confirm to start audit? (yes/no)
```

## Five-Phase Audit Protocol

### Phase 1: Reconnaissance (10%)

**Objective**: Map architecture and attack surface

```python
# Example: Identifying entry points in Django
# The AI agent will analyze urls.py patterns

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('api/users/<int:user_id>/', views.get_user),  # IDOR risk
    path('api/upload/', views.upload_file),            # File upload risk
    path('api/search/', views.search),                 # Injection risk
]

# Agent identifies:
# - 3 API endpoints (attack surface)
# - Parameter types (user_id: int, others: unconstrained)
# - No visible authentication decorators
```

**Output**: Architecture diagram, entry point inventory, technology stack

### Phase 2: Pattern Matching (30%)

**Objective**: Identify high-risk code patterns across all dimensions

```python
# Example: Detecting SQL injection patterns in Python

# VULNERABLE: String concatenation in query
def search_users(request):
    query = request.GET.get('q')
    # D1: SQL Injection - unsanitized input in raw query
    sql = f"SELECT * FROM users WHERE name LIKE '%{query}%'"
    cursor.execute(sql)
    return cursor.fetchall()

# VULNERABLE: ORM filter with raw SQL
def get_products(category):
    # D1: SQL Injection via extra()
    return Product.objects.extra(
        where=[f"category = '{category}'"]
    )

# SAFE: Parameterized query
def search_users_safe(request):
    query = request.GET.get('q')
    cursor.execute(
        "SELECT * FROM users WHERE name LIKE %s",
        [f'%{query}%']
    )
    return cursor.fetchall()
```

**Pattern Rules** (from `resources/rules/sql_injection_rules.md`):

```yaml
# SQL Injection Detection Rules
D1_SQL_INJECTION:
  sinks:
    python:
      - "cursor.execute()"
      - "cursor.executemany()"
      - "Model.objects.raw()"
      - "Model.objects.extra()"
    java:
      - "Statement.execute()"
      - "Statement.executeQuery()"
      - "createNativeQuery()"
    php:
      - "mysql_query()"
      - "mysqli_query()"
      - "PDO->query()"
  
  unsafe_patterns:
    - string_concatenation
    - f-string_interpolation
    - format_string
    - raw_sql_fragments
  
  safe_patterns:
    - parameterized_queries
    - ORM_query_builder
    - prepared_statements
```

### Phase 3: Taint Tracking & Data Flow Analysis (40%)

**Objective**: Trace data flow from source to sink, validate exploitability

```python
# Example: Deep taint analysis for command injection

# VULNERABLE: Multi-hop taint flow
from django.http import JsonResponse
from django.views.decorators.http import require_POST
import subprocess
import os

@require_POST
def backup_database(request):
    # SOURCE: User-controlled input
    db_name = request.POST.get('database')  # ← Taint source
    
    # HOP 1: Propagates to variable
    backup_path = f"/backups/{db_name}.sql"
    
    # HOP 2: Propagates to environment variable
    os.environ['BACKUP_FILE'] = backup_path
    
    # HOP 3: Propagates to command
    cmd = f"mysqldump {db_name} > {backup_path}"
    
    # SINK: Command execution without sanitization
    subprocess.call(cmd, shell=True)  # ← Command Injection (D1)
    
    return JsonResponse({'status': 'success'})

# Agent taint analysis output:
# [TAINT_FLOW]
#   Source: request.POST.get('database') [Line 8]
#   → db_name [Line 8]
#   → backup_path (f-string interpolation) [Line 11]
#   → os.environ['BACKUP_FILE'] [Line 14]
#   → cmd (f-string interpolation) [Line 17]
#   → subprocess.call() with shell=True [Line 20]
# 
# [SANITIZATION_CHECK]
#   ✗ No input validation
#   ✗ No allowlist check
#   ✗ No shell escaping
# 
# [EXPLOITABILITY] HIGH
#   Payload: database="; rm -rf / #"
#   Result: Command injection leading to RCE
```

**Enhanced Taint Analysis** (from `resources/knowledge/taint_analysis_enhanced.md`):

```python
# Example: Inter-procedural taint tracking

class UserService:
    def get_user_input(self, request):
        # SOURCE: Tainted input
        return request.GET.get('user_id')
    
    def build_query(self, user_id):
        # PROPAGATION: Taint flows through method call
        return f"SELECT * FROM users WHERE id = {user_id}"
    
    def execute(self, request):
        user_id = self.get_user_input(request)  # Taint enters
        query = self.build_query(user_id)       # Taint propagates
        cursor.execute(query)                   # SINK: SQL Injection

# Agent inter-procedural analysis:
# [CALL_GRAPH]
#   execute() → get_user_input() → build_query() → cursor.execute()
# 
# [TAINT_PATH]
#   request.GET.get('user_id') 
#   → return value of get_user_input()
#   → user_id parameter in build_query()
#   → f-string in build_query()
#   → return value of build_query()
#   → cursor.execute() argument
# 
# [VULNERABILITY] D1: SQL Injection via inter-procedural taint flow
```

### Phase 4: Validation & Attack Chain Construction (15%)

**Objective**: Verify exploitability, construct proof-of-concept

```python
# Example: Authorization bypass chain

# FILE: views.py
from django.contrib.auth.decorators import login_required

@login_required
def update_user_profile(request, user_id):
    # D3: IDOR - No ownership check
    user = User.objects.get(id=user_id)
    user.email = request.POST.get('email')
    user.save()
    return JsonResponse({'status': 'updated'})

@login_required
def delete_user(request, user_id):
    # D3: Horizontal privilege escalation
    if not request.user.is_staff:
        return JsonResponse({'error': 'forbidden'}, status=403)
    
    # D3: Missing ownership validation for staff users
    User.objects.filter(id=user_id).delete()
    return JsonResponse({'status': 'deleted'})

# Agent validation output:
# [VULNERABILITY_CHAIN]
#   Step 1: Create low-privilege account (user_a)
#   Step 2: Identify target user ID (user_b, id=100)
#   Step 3: Exploit IDOR in update_user_profile:
#           POST /api/users/100/update
#           Body: email=attacker@evil.com
#           → Updates user_b's email without ownership check
#   Step 4: Trigger password reset for user_b
#   Step 5: Gain access to user_b account
# 
# [POC]
#   curl -X POST http://target/api/users/100/update \
#     -H "Cookie: sessionid=<user_a_session>" \
#     -d "email=attacker@evil.com"
# 
# [IMPACT] High - Account takeover of arbitrary users
```

**Business Logic Vulnerability Example**:

```python
# Example: Race condition in payment processing

from django.db import transaction
from decimal import Decimal

class PaymentView(APIView):
    def post(self, request):
        user = request.user
        amount = Decimal(request.data['amount'])
        
        # D9: Race condition - No atomic balance check
        if user.balance >= amount:
            # VULNERABLE: Time window between check and update
            time.sleep(0.1)  # Simulate processing delay
            user.balance -= amount
            user.save()
            
            Payment.objects.create(user=user, amount=amount)
            return Response({'status': 'success'})
        
        return Response({'error': 'insufficient funds'}, status=400)

# FIX: Use database-level atomic operations
class PaymentViewFixed(APIView):
    def post(self, request):
        user = request.user
        amount = Decimal(request.data['amount'])
        
        with transaction.atomic():
            # Lock the row during transaction
            user = User.objects.select_for_update().get(id=user.id)
            
            if user.balance >= amount:
                user.balance -= amount
                user.save()
                Payment.objects.create(user=user, amount=amount)
                return Response({'status': 'success'})
            
            return Response({'error': 'insufficient funds'}, status=400)

# Agent identifies:
# [D9_BUSINESS_LOGIC] Race Condition
#   Exploit: Send multiple concurrent payment requests
#   Result: Spend same balance multiple times
#   Fix: select_for_update() with transaction.atomic()
```

### Phase 5: Structured Reporting (5%)

**Objective**: Generate actionable security report

```markdown
# Security Audit Report

## Executive Summary
- **Total Vulnerabilities**: 12 (3 Critical, 5 High, 3 Medium, 1 Low)
- **Project**: Django E-commerce Platform
- **Audit Mode**: Deep
- **Coverage**: D1-D10 (100%)

## Critical Vulnerabilities

### [CRITICAL-1] SQL Injection in Product Search
**Severity**: Critical (CVSS 9.8)
**Dimension**: D1 - Injection
**Location**: `app/views.py:45-48`

**Vulnerable Code**:
```python
def search_products(request):
    query = request.GET.get('q')
    sql = f"SELECT * FROM products WHERE name LIKE '%{query}%'"
    cursor.execute(sql)
```

**Data Flow**:
```
request.GET.get('q') → query → f-string → cursor.execute()
```

**Proof of Concept**:
```bash
curl "http://target/search?q=test'%20UNION%20SELECT%20username,password%20FROM%20users--"
```

**Impact**: Database compromise, credential theft, data exfiltration

**Fix**:
```python
def search_products(request):
    query = request.GET.get('q')
    cursor.execute(
        "SELECT * FROM products WHERE name LIKE %s",
        [f'%{query}%']
    )
```

**Verification**:
```python
# Test that parameterization prevents injection
query = "test' OR '1'='1"
cursor.execute("SELECT * FROM products WHERE name LIKE %s", [f'%{query}%'])
# Query becomes: WHERE name LIKE '%test\' OR \'1\'=\'1%'
# Special chars are escaped, preventing injection
```

---

### [CRITICAL-2] Authentication Bypass via JWT Algorithm Confusion
**Severity**: Critical (CVSS 9.1)
**Dimension**: D2 - Authentication
**Location**: `app/auth.py:12-20`

**Vulnerable Code**:
```python
import jwt

def verify_token(token):
    # D2: Accepts 'none' algorithm
    decoded = jwt.decode(
        token,
        settings.SECRET_KEY,
        algorithms=['HS256', 'RS256', 'none']  # ← Vulnerability
    )
    return decoded
```

**Exploit**:
```python
# Attacker creates unsigned token with alg=none
import jwt
import base64

header = base64.b64encode(b'{"alg":"none","typ":"JWT"}').decode()
payload = base64.b64encode(b'{"user_id":1,"is_admin":true}').decode()
fake_token = f"{header}.{payload}."

# Server accepts it because 'none' is in algorithms list
```

**Fix**:
```python
def verify_token(token):
    decoded = jwt.decode(
        token,
        settings.SECRET_KEY,
        algorithms=['HS256']  # Only allow expected algorithm
    )
    return decoded
```
```

## Configuration

### Audit Scope Configuration

Create `.dfyx_audit.yaml` in project root:

```yaml
# .dfyx_audit.yaml
audit:
  mode: deep  # quick | standard | deep
  
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
  
  exclude_paths:
    - "*/tests/*"
    - "*/migrations/*"
    - "*/static/*"
    - "node_modules/*"
  
  include_paths:
    - "app/*"
    - "api/*"
    - "services/*"
  
  severity_threshold: medium  # low | medium | high | critical
  
  frameworks:
    python:
      - django
      - flask
    java:
      - spring-boot
      - mybatis
  
  custom_sinks:
    # Add project-specific dangerous functions
    python:
      - "custom_module.execute_raw_query"
      - "legacy.unsafe_operation"
  
  custom_sanitizers:
    # Add project-specific sanitization functions
    python:
      - "utils.sanitize_input"
      - "validators.clean_sql"
```

### Environment Variables

```bash
# For optional Python scripts
export DFYX_AUDIT_MODE=deep
export DFYX_OUTPUT_FORMAT=markdown  # markdown | json | html
export DFYX_REPORT_PATH=./security_report.md
export DFYX_LOG_LEVEL=INFO  # DEBUG | INFO | WARNING | ERROR
```

## Optional Python Scripts

While the AI agent uses the skill directly, Python scripts are provided for standalone use:

### Code Scanning

```bash
# Full audit with default settings
python scripts/code_scan.py /path/to/project

# Quick scan for CI/CD
python scripts/code_scan.py /path/to/project --mode quick

# Deep scan with custom config
python scripts/code_scan.py /path/to/project \
  --mode deep \
  --config .dfyx_audit.yaml \
  --output results.json
```

### Pattern Scanner

```python
# scripts/pattern_scanner.py
from pattern_scanner import PatternScanner

scanner = PatternScanner(
    language='python',
    rules_path='resources/rules/'
)

# Scan for SQL injection patterns
results = scanner.scan_directory(
    '/path/to/project',
    dimensions=['D1'],  # Focus on injection
    exclude_patterns=['*/tests/*']
)

for vuln in results:
    print(f"{vuln.severity}: {vuln.type} at {vuln.file}:{vuln.line}")
    print(f"Code: {vuln.code_snippet}")
    print(f"Fix: {vuln.recommendation}\n")
```

### Data Flow Analyzer

```python
# scripts/data_flow_analyzer.py
from data_flow_analyzer import TaintAnalyzer

analyzer = TaintAnalyzer(language='python')

# Analyze taint flow from source to sink
flow = analyzer.trace_taint(
    source_file='views.py',
    source_line=45,
    source_var='user_input',
    max_depth=10  # Maximum call depth
)

print(f"Taint flow path:")
for hop in flow.path:
    print(f"  {hop.file}:{hop.line} - {hop.function}() - {hop.variable}")

if flow.reaches_sink:
    print(f"\nReaches dangerous sink: {flow.sink.function}()")
    print(f"Exploitability: {flow.exploitability_score}/10")
```

### Secret Detection

```bash
# Find hardcoded secrets
python scripts/secret_finder.py /path/to/project \
  --output secrets.json \
  --exclude "*/tests/*"
```

```python
# scripts/secret_finder.py usage in code
from secret_finder import SecretDetector

detector = SecretDetector()
secrets = detector.scan_directory('/path/to/project')

for secret in secrets:
    print(f"[{secret.type}] {secret.file}:{secret.line}")
    print(f"  Pattern: {secret.pattern}")
    print(f"  Entropy: {secret.entropy}")
    if secret.is_high_confidence:
        print(f"  ⚠️  High confidence real secret")
```

### Dependency Analysis

```bash
# Check for vulnerable dependencies
python scripts/dependency_analyzer.py /path/to/project

# Output CVEs and vulnerable versions
# CVE-2023-12345: Django < 3.2.18 - SQL Injection
# CVE-2023-67890: requests < 2.31.0 - SSRF
```

### Report Generation

```bash
# Generate Markdown report
python scripts/report_generator.py \
  --input results.json \
  --output report.md \
  --template templates/report-templates/web-app-report-template.md

# Generate HTML report
python scripts/report_generator.py \
  --input results.json \
  --output report.html \
  --format html
```

## Language-Specific Examples

### Java (Spring Boot)

```java
// VULNERABLE: SQL Injection via MyBatis
@Mapper
public interface UserMapper {
    // D1: SQL Injection - ${} inserts value directly without escaping
    @Select("SELECT * FROM users WHERE username = '${username}'")
    User findByUsername(@Param("username") String username);
}

// FIX: Use #{} for parameterization
@Mapper
public interface UserMapperFixed {
    @Select("SELECT * FROM users WHERE username = #{username}")
    User findByUsername(@Param("username") String username);
}

// VULNERABLE: Deserialization
@RestController
public class DataController {
    @PostMapping("/import")
    public ResponseEntity<?> importData(@RequestBody String data) {
        // D4: Insecure deserialization
        ObjectInputStream ois = new ObjectInputStream(
            new ByteArrayInputStream(Base64.getDecoder().decode(data))
        );
        Object obj = ois.readObject();  // RCE via gadget chains
        return ResponseEntity.ok(obj);
    }
}

// FIX: Use JSON instead of Java serialization
@PostMapping("/import")
public ResponseEntity<?> importData(@RequestBody DataDTO data) {
    // Safe: JSON deserialization with whitelisted types
    return ResponseEntity.ok(processData(data));
}
```

### PHP (Laravel)

```php
<?php
// VULNERABLE: Mass Assignment
class UserController extends Controller {
    public function update(Request $request, $id) {
        $user = User::find($id);
        // D9: Mass Assignment - user can set is_admin=1
        $user->update($request->all());
        return response()->json($user);
    }
}

// FIX: Whitelist allowed fields
class UserControllerFixed extends Controller {
    public function update(Request $request, $id) {
        $user = User::find($id);
        $user->update($request->only(['name', 'email']));
        return response()->json($user);
    }
}

// VULNERABLE: File Upload
public function upload(Request $request) {
    // D5: No file type validation
    $file = $request->file('upload');
    $path = $file->store('uploads');
    return response()->json(['path' => $path]);
}

// FIX: Validate file type and sanitize filename
public function uploadFixed(Request $request) {
    $request->validate([
        'upload' => 'required|file|mimes:jpg,png,pdf|max:2048'
    ]);
    
    $file = $request->file('upload');
    $filename = Str::random(40) . '.' . $file->extension();
    $path = $file->storeAs('uploads', $filename);
    
    return response()->json(['path' => $path]);
}
?>
```

### Go (Gin Framework)

```go
// VULNERABLE: Command Injection
func BackupHandler(c *gin.Context) {
    filename := c.Query("file")
    
    // D1: Command injection via unsanitized input
    cmd := exec.Command("sh", "-c", "tar -czf /backups/"+filename)
    output, _ := cmd.CombinedOutput()
    
    c.String(200, string(output))
}

// FIX: Avoid shell, use argument array
func BackupHandlerFixed(c *gin.Context) {
    filename := c.Query("file")
    
    // Validate filename
    if !isValidFilename(filename) {
        c.JSON(400, gin.H{"error": "invalid filename"})
        return
    }
    
    // Safe: No shell interpretation
    cmd := exec.Command("tar", "-czf", "/backups/"+filename)
    output, err := cmd.CombinedOutput()
    
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }
    
    c.String(200, string(output))
}

func isValidFilename(name string) bool {
    // Only allow alphanumeric, dash, underscore
    matched, _ := regexp.MatchString(`^[a-zA-Z0-9_-]+\.tar\.gz$`, name)
    return matched
}
```

### JavaScript/Node.js (Express)

```javascript
// VULNERABLE: NoSQL Injection
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  // D1: NoSQL Injection - object injection in MongoDB
  const user = await User.findOne({
    username: username,
    password: password  // Attacker sends: {"$ne": null}
  });
  
  if (user) {
    res.json({ token: generateToken(user) });
  }
});

// FIX: Ensure string types and use password hashing
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  
  // Ensure string types
  if (typeof username !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  const user = await User.findOne({ username });
  
  if (user && await bcrypt.compare(password, user.passwordHash)) {
    res.json({ token: generateToken(user) });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// VULNERABLE: SSRF
app.get('/fetch', async (req, res) => {
  const url = req.query.url;
  
  // D6: SSRF - can access internal services
  const response = await axios.get(url);
  res.send(response.data);
});

// FIX: Whitelist allowed domains
const ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com'];

app.get('/fetch', async (req, res) => {
  const url = req.query.url;
  
  try {
    const parsedUrl = new URL(url);
    
    // Check protocol
    if (!['http:', 'https:'].includes(parsedUrl.protocol)) {
      return res.status(400).json({ error: 'Invalid protocol' });
    }
    
    // Check domain whitelist
    if (!ALLOWED_DOMAINS.includes(parsedUrl.hostname)) {
      return res.status(400).json({ error: 'Domain not allowed' });
    }
    
    const response = await axios.get(url, {
      timeout: 5000,
      maxRedirects: 0
    });
    
    res.send(response.data);
  } catch (error) {
    res.status(400).json({ error: 'Invalid URL' });
  }
});
```

## Common Patterns & Anti-Patterns

### Authentication Patterns

```python
# ANTI-PATTERN: Weak session management
def login(request):
    user = authenticate(username=request.POST['username'],
                       password=request.POST['password'])
    if user:
        # D2: Predictable session ID
        request.session['user_id'] = user.id
        request.session['session_id'] = str(user.id) + str(int(time.time()))
        return redirect('dashboard')

# PATTERN: Secure session management
from django.contrib.auth import login as django_login

def login_secure(request):
    user = authenticate(username=request.POST['username'],
                       password=request.POST['password'])
    if user:
        django_login(request, user)  # Uses cryptographically secure session
        return redirect('dashboard')
```

### Authorization Patterns

```python
# ANTI-PATTERN: Missing authorization checks
@login_required
def delete_document(request, doc_id):
    # D3: Any authenticated user can delete any document
    Document.objects.filter(id=doc_id).delete()
    return JsonResponse({'status': 'deleted'})

# PATTERN: Proper authorization
@login_required
def delete_document_secure(request, doc_id):
    try:
        doc = Document.objects.get(id=doc_id)
        
        # Check ownership
        if doc.owner != request.user:
            return JsonResponse({'error': 'forbidden'}, status=403)
        
        doc.delete()
        return JsonResponse({'status': 'deleted'})
    except Document.DoesNotExist:
        return JsonResponse({'error': 'not found'}, status=404)
```

### Cryptography Patterns

```python
# ANTI-PATTERN: Weak encryption
from Crypto.Cipher import AES
import base64

def encrypt_data(data):
    # D7: ECB mode, static IV, weak key derivation
    key = hashlib.md5(b'my_secret_key').digest()
    cipher = AES.new(key, AES.MODE_ECB)
    encrypted = cipher.encrypt(pad(data))
    return base64.b64encode(encrypted)

# PATTERN: Strong encryption
from cryptography.fernet import Fernet
import os

def encrypt_data_secure(data):
    # Use Fernet (AES-128-CBC + HMAC)
    key = os.environ.get('ENCRYPTION_KEY').encode()  # From env, not hardcoded
    f = Fernet(key)
    encrypted = f.encrypt(data.encode())
    return encrypted

# Generate key once:
# key = Fernet.generate_key()
# Store securely in environment: export ENCRYPTION_KEY=<key>
```

## Troubleshooting

### False Positives

**Issue**: AI reports vulnerabilities in test code or mock data

**Solution**: Configure exclusion paths in `.dfyx_audit.yaml`:
```yaml
exclude_paths:
  - "*/tests/*"
  - "*/test_*.py"
  - "*/mocks/*"
  - "*_test.go"
```

### Missing Vulnerabilities
