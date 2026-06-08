---
name: dfyx-code-security-review
description: Expert-level code security auditing skill using deep data flow analysis and business logic understanding for 9 languages
triggers:
  - audit this project for security vulnerabilities
  - perform a code security review
  - check this code for security issues
  - scan for security vulnerabilities in the codebase
  - conduct a security audit
  - analyze code security
  - find security flaws in this project
  - run a security code review
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Expert-level code security auditing skill using white-box static analysis methodology. Employs a five-phase standardized audit protocol to systematically discover and validate security vulnerabilities in source code through deep data flow analysis and business logic understanding.

## Overview

**dfyx_code_security_review** is a professional code security audit skill designed for AI coding agents. It uses a comprehensive approach covering:

- **9 Languages**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 Security Dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **Dual-Track Audit Model**: Sink-driven + Control-driven + Config-driven analysis
- **Five-Phase Protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting

## Installation

Clone the repository to your project or a global location:

```bash
# Clone repository
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
cd skill-dfyx_code_security_review

# Install Python dependencies (optional, for standalone scripts)
pip install -r requirements.txt
```

For AI agent integration, ensure the skill directory is accessible to your AI coding agent's skill loading mechanism.

## Core Audit Protocol

### Five-Phase Standardized Audit

```
Phase 1: Reconnaissance & Mapping (10%)
    ↓ Output: Architecture diagram, attack surface inventory
    
Phase 2: Parallel Pattern Matching (30%)
    ↓ Output: High-risk area checklist
    
Phase 3: Deep Taint Tracking & Practical Testing (40%)
    ↓ Output: Confirmed vulnerability list, test validation reports
    
Phase 4: Validation & Attack Chain Construction (15%)
    ↓ Output: Vulnerability validation reports
    
Phase 5: Structured Reporting (5%)
    ↓ Output: Complete audit report
```

### Dual-Track Audit Model

| Track | Dimensions | Method | Target |
|-------|-----------|--------|--------|
| **Sink-driven** | D1 Injection, D4 Deserialization, D5 File, D6 SSRF | Grep dangerous functions → Track data flow → Verify no protection | Existing dangerous code |
| **Control-driven** | D3 Authorization, D9 Business Logic | Enumerate endpoints → Verify security controls → Missing = Vulnerability | Missing security controls |
| **Config-driven** | D2 Authentication, D7 Crypto, D8 Config, D10 Supply Chain | Search configs → Compare security baseline | Misconfigurations |

## Security Dimensions

### D1: Injection Vulnerabilities

**Detection Pattern (SQL Injection in Python/Django)**:

```python
# VULNERABLE: Direct string interpolation
def get_user(username):
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)  # SQL injection risk
    return cursor.fetchone()

# VULNERABLE: String concatenation
def search_products(category):
    query = "SELECT * FROM products WHERE category = '" + category + "'"
    return db.execute(query)

# SECURE: Parameterized queries
def get_user_secure(username):
    query = "SELECT * FROM users WHERE username = %s"
    cursor.execute(query, (username,))
    return cursor.fetchone()

# SECURE: ORM usage
from django.db.models import Q
def search_products_secure(category):
    return Product.objects.filter(category=category)
```

**Detection Pattern (Command Injection in Python)**:

```python
import subprocess
import shlex

# VULNERABLE: Direct user input in shell command
def backup_file(filename):
    subprocess.call(f"tar -czf backup.tar.gz {filename}", shell=True)
    # filename = "file.txt; rm -rf /" leads to command injection

# VULNERABLE: os.system with user input
import os
def process_image(image_path):
    os.system(f"convert {image_path} output.png")

# SECURE: Use array form without shell
def backup_file_secure(filename):
    subprocess.call(["tar", "-czf", "backup.tar.gz", filename])

# SECURE: Validate and sanitize input
def backup_file_validated(filename):
    if not re.match(r'^[a-zA-Z0-9_\-\.]+$', filename):
        raise ValueError("Invalid filename")
    subprocess.call(["tar", "-czf", "backup.tar.gz", filename])
```

**Detection Pattern (SQL Injection in Java/MyBatis)**:

```java
// VULNERABLE: String concatenation in MyBatis
@Select("SELECT * FROM users WHERE username = '${username}'")
User findByUsername(@Param("username") String username);

// VULNERABLE: Dynamic SQL with concatenation
public List<Product> searchProducts(String category) {
    String sql = "SELECT * FROM products WHERE category = '" + category + "'";
    return jdbcTemplate.query(sql, new ProductRowMapper());
}

// SECURE: Parameterized queries with #{}
@Select("SELECT * FROM users WHERE username = #{username}")
User findByUsernameSafe(@Param("username") String username);

// SECURE: PreparedStatement
public List<Product> searchProductsSafe(String category) {
    String sql = "SELECT * FROM products WHERE category = ?";
    return jdbcTemplate.query(sql, new Object[]{category}, new ProductRowMapper());
}
```

### D2: Authentication Vulnerabilities

**Detection Pattern (JWT Security in Node.js)**:

```javascript
const jwt = require('jsonwebtoken');

// VULNERABLE: Hardcoded secret in code
const SECRET_KEY = 'my-secret-key';
function generateToken(user) {
    return jwt.sign({ userId: user.id }, SECRET_KEY);
}

// VULNERABLE: No algorithm specification (algorithm confusion attack)
function verifyToken(token) {
    return jwt.verify(token, SECRET_KEY);
}

// VULNERABLE: Weak secret
const WEAK_SECRET = '12345';

// SECURE: Environment variable for secret
function generateTokenSecure(user) {
    return jwt.sign(
        { userId: user.id },
        process.env.JWT_SECRET,
        { 
            algorithm: 'HS256',
            expiresIn: '1h'
        }
    );
}

// SECURE: Explicit algorithm verification
function verifyTokenSecure(token) {
    return jwt.verify(token, process.env.JWT_SECRET, {
        algorithms: ['HS256']
    });
}

// SECURE: Token rotation and refresh
function refreshToken(refreshToken) {
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    return generateTokenSecure(decoded);
}
```

**Detection Pattern (Session Management in Python/Flask)**:

```python
from flask import Flask, session
import secrets

app = Flask(__name__)

# VULNERABLE: Weak secret key
app.secret_key = 'dev'

# VULNERABLE: No session timeout
@app.route('/login', methods=['POST'])
def login():
    # ... authentication ...
    session['user_id'] = user.id
    return redirect('/dashboard')

# SECURE: Strong secret from environment
app.secret_key = os.environ.get('SECRET_KEY') or secrets.token_hex(32)

# SECURE: Session configuration
app.config.update(
    SESSION_COOKIE_SECURE=True,  # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,  # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax',  # CSRF protection
    PERMANENT_SESSION_LIFETIME=timedelta(hours=1)
)

# SECURE: Session validation
@app.before_request
def validate_session():
    if 'user_id' in session:
        if session.get('expires_at', 0) < time.time():
            session.clear()
            return redirect('/login')
```

### D3: Authorization Vulnerabilities

**Detection Pattern (IDOR in RESTful API - Python/Flask)**:

```python
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

# VULNERABLE: No authorization check (IDOR)
@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    user = User.query.get(user_id)
    return jsonify(user.to_dict())

# VULNERABLE: Missing ownership verification
@app.route('/api/documents/<int:doc_id>', methods=['DELETE'])
def delete_document(doc_id):
    document = Document.query.get(doc_id)
    db.session.delete(document)
    db.session.commit()
    return '', 204

# SECURE: Verify user owns the resource
def require_ownership(resource_class, id_param='id'):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            resource_id = kwargs.get(id_param)
            resource = resource_class.query.get_or_404(resource_id)
            current_user_id = get_current_user_id()
            
            if resource.owner_id != current_user_id:
                return jsonify({'error': 'Forbidden'}), 403
            
            kwargs['resource'] = resource
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/api/users/<int:user_id>')
def get_user_secure(user_id):
    current_user_id = get_current_user_id()
    if user_id != current_user_id and not is_admin():
        return jsonify({'error': 'Forbidden'}), 403
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())

@app.route('/api/documents/<int:doc_id>', methods=['DELETE'])
@require_ownership(Document, 'doc_id')
def delete_document_secure(doc_id, resource):
    db.session.delete(resource)
    db.session.commit()
    return '', 204
```

**Detection Pattern (Horizontal Privilege Escalation in Java/Spring)**:

```java
// VULNERABLE: No authorization check
@GetMapping("/api/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId) {
    return orderRepository.findById(orderId).orElseThrow();
}

// VULNERABLE: Only checking authentication, not authorization
@GetMapping("/api/profile/{userId}")
@PreAuthorize("isAuthenticated()")
public User getProfile(@PathVariable Long userId) {
    return userRepository.findById(userId).orElseThrow();
}

// SECURE: Verify ownership
@GetMapping("/api/orders/{orderId}")
public Order getOrderSecure(@PathVariable Long orderId, Authentication auth) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    String currentUsername = auth.getName();
    
    if (!order.getUser().getUsername().equals(currentUsername)) {
        throw new AccessDeniedException("Not authorized to view this order");
    }
    
    return order;
}

// SECURE: Using Spring Security SpEL
@GetMapping("/api/orders/{orderId}")
@PreAuthorize("@orderSecurity.canAccess(#orderId, authentication)")
public Order getOrderSpEL(@PathVariable Long orderId) {
    return orderRepository.findById(orderId).orElseThrow();
}

// Security service
@Service
public class OrderSecurity {
    @Autowired
    private OrderRepository orderRepository;
    
    public boolean canAccess(Long orderId, Authentication auth) {
        Order order = orderRepository.findById(orderId).orElse(null);
        if (order == null) return false;
        return order.getUser().getUsername().equals(auth.getName());
    }
}
```

### D4: Deserialization Vulnerabilities

**Detection Pattern (Python Pickle)**:

```python
import pickle
import json
import hmac
import hashlib

# VULNERABLE: Deserializing untrusted data
def load_session(session_data):
    return pickle.loads(session_data)  # RCE risk

# VULNERABLE: No integrity check
def load_user_profile(data):
    return pickle.loads(base64.b64decode(data))

# SECURE: Use JSON instead of pickle
def load_session_secure(session_data):
    return json.loads(session_data)

# SECURE: If pickle is necessary, use HMAC signature
def serialize_with_hmac(obj):
    data = pickle.dumps(obj)
    signature = hmac.new(
        os.environ['SECRET_KEY'].encode(),
        data,
        hashlib.sha256
    ).hexdigest()
    return base64.b64encode(data).decode() + '.' + signature

def deserialize_with_hmac(signed_data):
    try:
        data_b64, signature = signed_data.rsplit('.', 1)
        data = base64.b64decode(data_b64)
        
        expected_sig = hmac.new(
            os.environ['SECRET_KEY'].encode(),
            data,
            hashlib.sha256
        ).hexdigest()
        
        if not hmac.compare_digest(signature, expected_sig):
            raise ValueError("Invalid signature")
        
        return pickle.loads(data)
    except Exception as e:
        raise ValueError(f"Deserialization failed: {e}")
```

**Detection Pattern (Java Deserialization)**:

```java
import java.io.*;

// VULNERABLE: Deserializing untrusted data
public Object loadObject(byte[] data) {
    try (ObjectInputStream ois = new ObjectInputStream(
            new ByteArrayInputStream(data))) {
        return ois.readObject();  // RCE risk with malicious payloads
    }
}

// VULNERABLE: XML deserialization without restrictions
public Object loadFromXML(InputStream xml) {
    XMLDecoder decoder = new XMLDecoder(xml);
    return decoder.readObject();
}

// SECURE: Use JSON instead
import com.fasterxml.jackson.databind.ObjectMapper;

public <T> T loadObjectSecure(String json, Class<T> clazz) {
    ObjectMapper mapper = new ObjectMapper();
    return mapper.readValue(json, clazz);
}

// SECURE: If Java serialization is necessary, use ObjectInputFilter
public Object loadObjectWithFilter(byte[] data) throws IOException, ClassNotFoundException {
    try (ObjectInputStream ois = new ObjectInputStream(
            new ByteArrayInputStream(data))) {
        
        // Whitelist allowed classes
        ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
            "com.example.SafeClass;!*"
        );
        ois.setObjectInputFilter(filter);
        
        return ois.readObject();
    }
}

// SECURE: Implement custom serialization validation
public Object loadObjectValidated(byte[] data) throws Exception {
    ByteArrayInputStream bis = new ByteArrayInputStream(data);
    ValidatingObjectInputStream ois = new ValidatingObjectInputStream(bis);
    
    // Only allow specific classes
    ois.accept(SafeClass.class, AnotherSafeClass.class);
    
    return ois.readObject();
}

class ValidatingObjectInputStream extends ObjectInputStream {
    private Set<Class<?>> allowedClasses = new HashSet<>();
    
    public ValidatingObjectInputStream(InputStream in) throws IOException {
        super(in);
    }
    
    public void accept(Class<?>... classes) {
        allowedClasses.addAll(Arrays.asList(classes));
    }
    
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) 
            throws IOException, ClassNotFoundException {
        Class<?> clazz = super.resolveClass(desc);
        if (!allowedClasses.contains(clazz)) {
            throw new InvalidClassException("Unauthorized class: " + clazz.getName());
        }
        return clazz;
    }
}
```

### D5: File Operation Vulnerabilities

**Detection Pattern (Path Traversal in Python)**:

```python
import os
from pathlib import Path

# VULNERABLE: Direct path concatenation
@app.route('/download/<filename>')
def download_file(filename):
    file_path = f'/var/www/uploads/{filename}'
    return send_file(file_path)
    # filename = "../../etc/passwd" leads to path traversal

# VULNERABLE: Insufficient validation
def read_user_file(filename):
    if '..' not in filename:  # Weak check, can be bypassed
        return open(f'/data/{filename}').read()

# SECURE: Resolve and validate path
UPLOAD_DIR = Path('/var/www/uploads').resolve()

@app.route('/download/<filename>')
def download_file_secure(filename):
    # Resolve the full path
    file_path = (UPLOAD_DIR / filename).resolve()
    
    # Ensure path is within allowed directory
    if not str(file_path).startswith(str(UPLOAD_DIR)):
        abort(400, "Invalid file path")
    
    if not file_path.exists():
        abort(404)
    
    return send_file(file_path)

# SECURE: Use allowed filename whitelist
import re

def read_user_file_secure(filename):
    # Only allow alphanumeric, dash, underscore, dot
    if not re.match(r'^[a-zA-Z0-9_\-\.]+$', filename):
        raise ValueError("Invalid filename")
    
    file_path = (UPLOAD_DIR / filename).resolve()
    
    if not str(file_path).startswith(str(UPLOAD_DIR)):
        raise ValueError("Path traversal detected")
    
    return file_path.read_text()
```

**Detection Pattern (File Upload Vulnerabilities in PHP)**:

```php
<?php
// VULNERABLE: No file type validation
if (isset($_FILES['upload'])) {
    $target = "uploads/" . $_FILES['upload']['name'];
    move_uploaded_file($_FILES['upload']['tmp_name'], $target);
    // Can upload shell.php and execute arbitrary code
}

// VULNERABLE: Client-side MIME type check only
if ($_FILES['upload']['type'] == 'image/jpeg') {
    move_uploaded_file($_FILES['upload']['tmp_name'], 
                      "uploads/" . $_FILES['upload']['name']);
    // MIME type can be spoofed
}

// SECURE: Multiple validation layers
function secureFileUpload($file) {
    $allowed_extensions = ['jpg', 'jpeg', 'png', 'gif'];
    $max_size = 5 * 1024 * 1024; // 5MB
    $upload_dir = '/var/www/uploads/';
    
    // 1. Check file size
    if ($file['size'] > $max_size) {
        throw new Exception("File too large");
    }
    
    // 2. Validate extension
    $ext = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    if (!in_array($ext, $allowed_extensions)) {
        throw new Exception("Invalid file type");
    }
    
    // 3. Verify actual file type (magic bytes)
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mime = finfo_file($finfo, $file['tmp_name']);
    finfo_close($finfo);
    
    $allowed_mimes = ['image/jpeg', 'image/png', 'image/gif'];
    if (!in_array($mime, $allowed_mimes)) {
        throw new Exception("Invalid MIME type");
    }
    
    // 4. Generate random filename
    $new_filename = bin2hex(random_bytes(16)) . '.' . $ext;
    $target_path = $upload_dir . $new_filename;
    
    // 5. Move file
    if (!move_uploaded_file($file['tmp_name'], $target_path)) {
        throw new Exception("Upload failed");
    }
    
    // 6. Set restrictive permissions
    chmod($target_path, 0644);
    
    return $new_filename;
}

// SECURE: Additional re-encoding for images
function secureImageUpload($file) {
    $tmp_path = $file['tmp_name'];
    
    // Re-encode image to strip metadata and potential exploits
    $image = imagecreatefromstring(file_get_contents($tmp_path));
    if ($image === false) {
        throw new Exception("Invalid image");
    }
    
    $new_filename = bin2hex(random_bytes(16)) . '.png';
    $target_path = '/var/www/uploads/' . $new_filename;
    
    imagepng($image, $target_path);
    imagedestroy($image);
    
    chmod($target_path, 0644);
    
    return $new_filename;
}
?>
```

### D6: SSRF Vulnerabilities

**Detection Pattern (SSRF in Python)**:

```python
import requests
from urllib.parse import urlparse
import ipaddress

# VULNERABLE: No URL validation
@app.route('/fetch')
def fetch_url():
    url = request.args.get('url')
    response = requests.get(url)
    # url = "http://169.254.169.254/latest/meta-data/" (AWS metadata)
    # url = "http://localhost:8080/admin" (internal services)
    return response.content

# VULNERABLE: Weak validation
def fetch_image(url):
    if url.startswith('http://') or url.startswith('https://'):
        return requests.get(url).content
    # Can be bypassed with redirect chains

# SECURE: Whitelist allowed domains
ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com']

def is_safe_url(url):
    try:
        parsed = urlparse(url)
        
        # Only allow http/https
        if parsed.scheme not in ['http', 'https']:
            return False
        
        # Check domain whitelist
        if parsed.hostname not in ALLOWED_DOMAINS:
            return False
        
        # Resolve IP and check it's not private
        import socket
        ip = socket.gethostbyname(parsed.hostname)
        ip_obj = ipaddress.ip_address(ip)
        
        if ip_obj.is_private or ip_obj.is_loopback or ip_obj.is_link_local:
            return False
        
        return True
    except Exception:
        return False

@app.route('/fetch')
def fetch_url_secure():
    url = request.args.get('url')
    
    if not is_safe_url(url):
        abort(400, "Invalid URL")
    
    # Disable redirects to prevent bypass
    response = requests.get(url, allow_redirects=False, timeout=5)
    
    return response.content

# SECURE: Use allowlist with additional checks
def fetch_external_api(endpoint):
    base_url = 'https://api.example.com'
    
    # Validate endpoint format
    if not re.match(r'^/[a-z0-9/\-]+$', endpoint):
        raise ValueError("Invalid endpoint")
    
    url = f"{base_url}{endpoint}"
    
    # Additional safeguards
    response = requests.get(
        url,
        allow_redirects=False,
        timeout=5,
        verify=True  # Verify SSL certificates
    )
    
    return response.json()
```

**Detection Pattern (SSRF in Java/Spring)**:

```java
import org.springframework.web.client.RestTemplate;
import java.net.URI;
import java.net.InetAddress;

// VULNERABLE: No URL validation
@GetMapping("/proxy")
public String proxyRequest(@RequestParam String url) {
    RestTemplate restTemplate = new RestTemplate();
    return restTemplate.getForObject(url, String.class);
}

// SECURE: Validate and restrict URLs
@GetMapping("/proxy")
public String proxyRequestSecure(@RequestParam String url) throws Exception {
    // Parse URL
    URI uri = new URI(url);
    
    // Only allow http/https
    if (!uri.getScheme().matches("^https?$")) {
        throw new IllegalArgumentException("Invalid protocol");
    }
    
    // Check domain whitelist
    List<String> allowedDomains = Arrays.asList("api.example.com", "cdn.example.com");
    if (!allowedDomains.contains(uri.getHost())) {
        throw new IllegalArgumentException("Domain not allowed");
    }
    
    // Resolve and check IP
    InetAddress address = InetAddress.getByName(uri.getHost());
    if (address.isLoopbackAddress() || 
        address.isLinkLocalAddress() || 
        address.isSiteLocalAddress()) {
        throw new IllegalArgumentException("Private IP not allowed");
    }
    
    // Configure RestTemplate with restrictions
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.setRequestFactory(new SimpleClientHttpRequestFactory() {{
        setConnectTimeout(5000);
        setReadTimeout(5000);
    }});
    
    return restTemplate.getForObject(uri, String.class);
}

// SECURE: Custom HTTP client with safeguards
private boolean isPrivateIP(String hostname) throws Exception {
    InetAddress address = InetAddress.getByName(hostname);
    return address.isLoopbackAddress() || 
           address.isLinkLocalAddress() || 
           address.isSiteLocalAddress() ||
           address.getHostAddress().startsWith("169.254.");
}
```

## Audit Modes

### Quick Mode (5-10 minutes)

Focus on high-severity issues:
- Critical injection vulnerabilities (SQL, Command, SSTI)
- Hardcoded secrets and credentials
- Known vulnerable dependencies (CVEs)
- Authentication bypass patterns

```python
# Example: Quick scan for hardcoded secrets
import re

SECRET_PATTERNS = [
    (r'password\s*=\s*["\']([^"\']+)["\']', 'Hardcoded password'),
    (r'api[_-]?key\s*=\s*["\']([^"\']+)["\']', 'Hardcoded API key'),
    (r'secret[_-]?key\s*=\s*["\']([^"\']+)["\']', 'Hardcoded secret'),
    (r'AKIA[0-9A-Z]{16}', 'AWS Access Key'),
    (r'mongodb(\+srv)?://[^/]+/[^"\']+', 'MongoDB connection string'),
]

def quick_secret_scan(file_path):
    findings = []
    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        content = f.read()
        for pattern, description in SECRET_PATTERNS:
            matches = re.finditer(pattern, content, re.IGNORECASE)
            for match in matches:
                findings.append({
                    'file': file_path,
                    'line': content[:match.start()].count('\n') + 1,
                    'type': description,
                    'match': match.group(0)
                })
    return findings
```

### Standard Mode (30-60 minutes)

Comprehensive coverage of OWASP Top 10:
- All injection types
- Authentication and session management
- Authorization checks (IDOR, privilege escalation)
- Sensitive data exposure
- Security misconfigurations

### Deep Mode (1-3 hours)

Full coverage with attack chain analysis:
- Business logic vulnerabilities
- Race conditions
- Complex data flow analysis
- Multi-stage attack chains
- Configuration drift analysis

## Using Auxiliary Scripts

### Pattern Scanner

```bash
# Scan for SQL injection patterns in Python files
python scripts/pattern_scanner.py \
  --path /path/to/project \
  --language python \
  --pattern sql-injection

# Scan for command injection in all files
python scripts/pattern_scanner.py \
  --path /path/to/project \
  --pattern command-injection \
  --output results.json
```

### Data Flow Analyzer

```python
# scripts/data_flow_analyzer.py
from typing import List, Dict, Set

class DataFlowAnalyzer:
    def __init__(self, code_base_path: str):
        self.code_base_path = code_base_path
        self.taint_sources = set()
        self.sanitizers = set()
        self.sinks = set()
        
    def analyze_taint_flow(self, source_var: str, file_path: str) -> List[Dict]:
        """
        Trace data flow from source to potential sinks
        Returns list of vulnerable paths
        """
        vulnerable_paths = []
        
        # Example: Trace HTTP parameter to SQL query
        # Implementation would use AST parsing
        
        return vulnerable_paths
    
    def register_source(self, pattern: str):
        """Register taint source (e.g., request.GET, request.POST)"""
        self.taint_sources.add(pattern)
    
    def register_sanitizer(self, function: str):
        """Register sanitization function"""
        self.sanitizers.add(function)
    
    def register_sink(self, function: str):
        """Register dangerous sink function"""
        self.sinks.add(function)

# Usage
analyzer = DataFlowAnalyzer('/path/to/project')

# Configure for Django
analyzer.register_source('request.GET')
analyzer.register_source('request.POST')
analyzer.register_sanitizer('escape')
analyzer.register_sink('execute')
analyzer.register_sink('raw')

vulnerabilities = analyzer.analyze_taint_flow('user_input', 'views.py')
```

### Secret Finder

```bash
# Scan for exposed secrets
python scripts/secret_finder.py \
  --path /path/to/project \
  --exclude node_modules,venv,.git \
  --output secrets_report.json

# Check against custom patterns
python scripts/secret_finder.py \
  --path /path/to/project \
  --patterns custom_patterns.yaml
```

### Dependency Analyzer

```bash
# Analyze Python dependencies
python scripts/dependency_analyzer.py \
  --requirements requirements.txt \
  --check-cve \
  --output dependency_report.json

# Analyze npm dependencies
python scripts/dependency_analyzer.py \
  --package-json package.json \
  --check-cve \
  --severity high,critical
```

## Configuration

### Audit Configuration (`.dfyx-audit.yaml`)

```yaml
# Project-specific audit configuration
audit:
  mode: deep  # quick | standard | deep
  
  # Languages to scan
  languages:
    - python
    - javascript
    - java
  
  # Directories to exclude
  exclude:
    - node_modules
    - venv
    - .git
    - build
    - dist
    - test
    - __pycache__
  
  # Security dimensions to focus on
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
  
