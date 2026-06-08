---
name: dfyx-code-security-review
description: Expert-level code security audit skill using deep data flow analysis, taint tracking, and business logic understanding for 9 languages
triggers:
  - audit this codebase for security vulnerabilities
  - perform a security code review
  - check this code for security issues
  - run a security audit on this project
  - find security vulnerabilities in the code
  - analyze this code for security flaws
  - conduct a code security assessment
  - review code for OWASP Top 10 vulnerabilities
---

# dfyx-code-security-review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

A professional white-box static analysis skill for AI coding agents that performs systematic security audits using a five-phase protocol. Supports 9 languages (Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust) and covers 10 security dimensions including injection, authentication, authorization, deserialization, file operations, SSRF, encryption, configuration, business logic, and supply chain.

## Overview

**dfyx_code_security_review** implements a dual-track audit model:
- **Sink-driven**: Traces dangerous functions backward to find injection/RCE
- **Control-driven**: Enumerates endpoints to find missing security controls
- **Config-driven**: Scans configurations against security baselines

### Core Capabilities

- **10 Security Dimensions (D1-D10)**:
  - D1: Injection (SQL/Cmd/LDAP/SSTI/SpEL/JNDI)
  - D2: Authentication (Token/Session/JWT/Filter chain)
  - D3: Authorization (CRUD consistency, IDOR, horizontal privilege escalation)
  - D4: Deserialization (Java/Python/PHP gadget chains)
  - D5: File Operations (Upload/Download/Path traversal)
  - D6: SSRF (URL injection, protocol restrictions)
  - D7: Cryptography (Key management, encryption modes, KDF)
  - D8: Configuration (Actuator, CORS, error message exposure)
  - D9: Business Logic (Race conditions, mass assignment, state machines, multi-tenancy)
  - D10: Supply Chain (Dependency CVEs, version checks)

- **Three Audit Modes**:
  - **Quick**: CI/CD integration, 5-10 min (high-risk + secrets + CVEs)
  - **Standard**: OWASP Top 10, 30-60 min
  - **Deep**: Full coverage + attack chains, 1-3 hours

## Five-Phase Audit Protocol

```
Phase 1: Reconnaissance (10%)
  → Architecture diagram, attack surface inventory

Phase 2: Pattern Matching (30%)
  → High-risk area identification

Phase 3: Taint Tracking + Validation (40%)
  → Confirmed vulnerabilities with POCs

Phase 4: Attack Chain Construction (15%)
  → Multi-vulnerability exploitation scenarios

Phase 5: Structured Reporting (5%)
  → Complete audit report with remediation
```

## Installation

This skill is designed to be used directly by AI coding agents. No separate installation is required — the agent will reference the knowledge base and scripts as needed.

For manual use of helper scripts:

```bash
# Clone the repository
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
cd skill-dfyx_code_security_review

# Install Python dependencies (optional, for standalone scripts)
pip install -r requirements.txt
```

## Usage Patterns

### Basic Audit Request

When a user requests a security audit, initiate the five-phase protocol:

```
User: "Audit this codebase for security vulnerabilities"

Agent Response:
[MODE] deep
[RECON] 874 files detected, Tech Stack: Spring Boot 1.5 + Shiro 1.6 + JPA + Freemarker
[PLAN] 5 Agents, D1-D10 coverage, estimated 125 turns
[SCOPE] Expected findings: 8-15 vulnerabilities
Confirm to start audit? (yes/no)
```

### Phase 1: Reconnaissance

**Objective**: Map attack surface and identify technology stack

```python
# Pattern: Identify framework and version
import re
import os

def detect_tech_stack(project_path):
    """Detect technology stack from project files"""
    findings = {
        'language': None,
        'framework': None,
        'version': None,
        'dependencies': []
    }
    
    # Check for pom.xml (Java/Maven)
    pom_path = os.path.join(project_path, 'pom.xml')
    if os.path.exists(pom_path):
        with open(pom_path, 'r') as f:
            content = f.read()
            # Extract Spring Boot version
            spring_match = re.search(r'<spring-boot.version>([\d.]+)</spring-boot.version>', content)
            if spring_match:
                findings['framework'] = 'Spring Boot'
                findings['version'] = spring_match.group(1)
    
    # Check for requirements.txt (Python)
    req_path = os.path.join(project_path, 'requirements.txt')
    if os.path.exists(req_path):
        findings['language'] = 'Python'
        with open(req_path, 'r') as f:
            for line in f:
                if 'django' in line.lower():
                    findings['framework'] = 'Django'
                    version = re.search(r'[=<>]+(\d+\.\d+)', line)
                    if version:
                        findings['version'] = version.group(1)
                elif 'flask' in line.lower():
                    findings['framework'] = 'Flask'
    
    # Check for package.json (Node.js)
    pkg_path = os.path.join(project_path, 'package.json')
    if os.path.exists(pkg_path):
        import json
        findings['language'] = 'JavaScript'
        with open(pkg_path, 'r') as f:
            data = json.load(f)
            if 'express' in data.get('dependencies', {}):
                findings['framework'] = 'Express'
                findings['version'] = data['dependencies']['express']
    
    return findings

# Generate attack surface map
def map_attack_surface(project_path):
    """Enumerate entry points and data flows"""
    entry_points = []
    
    # Find HTTP endpoints
    for root, dirs, files in os.walk(project_path):
        for file in files:
            if file.endswith(('.java', '.py', '.js', '.php')):
                filepath = os.path.join(root, file)
                with open(filepath, 'r', encoding='utf-8', errors='ignore') as f:
                    content = f.read()
                    
                    # Spring Boot endpoints
                    if '@RequestMapping' in content or '@GetMapping' in content or '@PostMapping' in content:
                        endpoints = re.findall(r'@(?:Request|Get|Post|Put|Delete)Mapping\(["\']([^"\']+)["\']', content)
                        for ep in endpoints:
                            entry_points.append({
                                'file': filepath,
                                'endpoint': ep,
                                'type': 'HTTP',
                                'framework': 'Spring Boot'
                            })
                    
                    # Django views
                    if 'def ' in content and 'request' in content and file.endswith('.py'):
                        methods = re.findall(r'def\s+(\w+)\s*\([^)]*request[^)]*\)', content)
                        for method in methods:
                            entry_points.append({
                                'file': filepath,
                                'endpoint': method,
                                'type': 'HTTP',
                                'framework': 'Django'
                            })
    
    return entry_points
```

### Phase 2: Pattern Matching

**Objective**: Identify high-risk code patterns using sink-driven analysis

#### D1: SQL Injection Detection

```python
# Pattern: Find SQL injection vulnerabilities
import re

def scan_sql_injection(file_path, language='java'):
    """Detect SQL injection patterns"""
    vulnerabilities = []
    
    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        lines = f.readlines()
    
    if language == 'java':
        # Dangerous patterns
        dangerous_patterns = [
            (r'createQuery\s*\(\s*["\'].*?\+.*?["\']', 'JPA native query with concatenation'),
            (r'Statement\s+\w+\s*=.*?createStatement\(\)', 'JDBC Statement usage'),
            (r'\.execute\w*\(\s*["\'].*?\+', 'SQL execution with concatenation'),
            (r'jdbcTemplate\.query\w*\([^?]*\+', 'JdbcTemplate without parameterization')
        ]
        
        for i, line in enumerate(lines):
            for pattern, description in dangerous_patterns:
                if re.search(pattern, line):
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'SQL Injection',
                        'severity': 'CRITICAL',
                        'description': description,
                        'cwe': 'CWE-89'
                    })
    
    elif language == 'python':
        # Django ORM raw queries
        if 'raw(' in ''.join(lines) or 'execute(' in ''.join(lines):
            for i, line in enumerate(lines):
                if '.raw(' in line and '%' in line:
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'SQL Injection',
                        'severity': 'CRITICAL',
                        'description': 'Django raw() with string formatting',
                        'cwe': 'CWE-89'
                    })
                elif 'cursor.execute(' in line and ('%' in line or '+' in line or 'format(' in line):
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'SQL Injection',
                        'severity': 'CRITICAL',
                        'description': 'cursor.execute with string interpolation',
                        'cwe': 'CWE-89'
                    })
    
    elif language == 'php':
        for i, line in enumerate(lines):
            if 'mysql_query' in line or 'mysqli_query' in line:
                if '$_GET' in line or '$_POST' in line or '$_REQUEST' in line:
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'SQL Injection',
                        'severity': 'CRITICAL',
                        'description': 'Direct user input in SQL query',
                        'cwe': 'CWE-89'
                    })
    
    return vulnerabilities

# Example usage
# results = scan_sql_injection('UserController.java', 'java')
```

#### D1: Command Injection Detection

```python
def scan_command_injection(file_path, language='java'):
    """Detect command injection vulnerabilities"""
    vulnerabilities = []
    
    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        lines = f.readlines()
    
    if language == 'java':
        dangerous_patterns = [
            (r'Runtime\.getRuntime\(\)\.exec\s*\(', 'Runtime.exec() usage'),
            (r'ProcessBuilder\s*\(.*?\+', 'ProcessBuilder with concatenation'),
            (r'new\s+ProcessBuilder\s*\([^)]*\+[^)]*\)', 'ProcessBuilder with dynamic args')
        ]
        
        for i, line in enumerate(lines):
            for pattern, description in dangerous_patterns:
                if re.search(pattern, line):
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'Command Injection',
                        'severity': 'CRITICAL',
                        'description': description,
                        'cwe': 'CWE-78'
                    })
    
    elif language == 'python':
        for i, line in enumerate(lines):
            if any(func in line for func in ['os.system(', 'os.popen(', 'subprocess.call(', 'subprocess.run(', 'subprocess.Popen(']):
                if 'shell=True' in line or ('f"' in line or "f'" in line):
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'Command Injection',
                        'severity': 'CRITICAL',
                        'description': 'Shell command with shell=True or f-string',
                        'cwe': 'CWE-78'
                    })
    
    elif language == 'php':
        dangerous_funcs = ['exec', 'shell_exec', 'system', 'passthru', 'popen', 'proc_open']
        for i, line in enumerate(lines):
            for func in dangerous_funcs:
                if func + '(' in line and ('$_' in line or '$' in line):
                    vulnerabilities.append({
                        'file': file_path,
                        'line': i + 1,
                        'code': line.strip(),
                        'type': 'Command Injection',
                        'severity': 'CRITICAL',
                        'description': f'{func}() with user input',
                        'cwe': 'CWE-78'
                    })
    
    return vulnerabilities
```

#### D4: Deserialization Vulnerabilities

```python
def scan_deserialization(file_path, language='java'):
    """Detect insecure deserialization"""
    vulnerabilities = []
    
    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        lines = f.readlines()
    
    if language == 'java':
        for i, line in enumerate(lines):
            # ObjectInputStream usage
            if 'ObjectInputStream' in line and 'readObject()' in line:
                vulnerabilities.append({
                    'file': file_path,
                    'line': i + 1,
                    'code': line.strip(),
                    'type': 'Insecure Deserialization',
                    'severity': 'CRITICAL',
                    'description': 'ObjectInputStream.readObject() without validation',
                    'cwe': 'CWE-502'
                })
            
            # XMLDecoder usage
            if 'XMLDecoder' in line and ('readObject' in line or 'new XMLDecoder' in line):
                vulnerabilities.append({
                    'file': file_path,
                    'line': i + 1,
                    'code': line.strip(),
                    'type': 'Insecure Deserialization',
                    'severity': 'CRITICAL',
                    'description': 'XMLDecoder usage (gadget chain risk)',
                    'cwe': 'CWE-502'
                })
    
    elif language == 'python':
        for i, line in enumerate(lines):
            if 'pickle.loads(' in line or 'pickle.load(' in line:
                vulnerabilities.append({
                    'file': file_path,
                    'line': i + 1,
                    'code': line.strip(),
                    'type': 'Insecure Deserialization',
                    'severity': 'CRITICAL',
                    'description': 'pickle deserialization (RCE risk)',
                    'cwe': 'CWE-502'
                })
            
            if 'yaml.load(' in line and 'Loader=' not in line:
                vulnerabilities.append({
                    'file': file_path,
                    'line': i + 1,
                    'code': line.strip(),
                    'type': 'Insecure Deserialization',
                    'severity': 'HIGH',
                    'description': 'yaml.load() without safe loader',
                    'cwe': 'CWE-502'
                })
    
    elif language == 'php':
        for i, line in enumerate(lines):
            if 'unserialize(' in line and ('$_' in line or '$' in line):
                vulnerabilities.append({
                    'file': file_path,
                    'line': i + 1,
                    'code': line.strip(),
                    'type': 'Insecure Deserialization',
                    'severity': 'CRITICAL',
                    'description': 'unserialize() with user input',
                    'cwe': 'CWE-502'
                })
    
    return vulnerabilities
```

### Phase 3: Taint Tracking

**Objective**: Trace data flow from source to sink to confirm exploitability

```python
def taint_analysis(file_path, sink_line, language='java'):
    """
    Backward taint analysis from dangerous sink
    Returns: data flow path, sanitization status, exploitability
    """
    
    with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
        lines = f.readlines()
    
    # Extract method containing the sink
    method_start = None
    method_end = None
    brace_count = 0
    
    for i in range(sink_line - 1, -1, -1):
        if re.search(r'(public|private|protected).*?\(', lines[i]):
            method_start = i
            break
    
    if method_start is not None:
        for i in range(method_start, len(lines)):
            if '{' in lines[i]:
                brace_count += lines[i].count('{')
            if '}' in lines[i]:
                brace_count -= lines[i].count('}')
            if brace_count == 0 and i > method_start:
                method_end = i
                break
    
    # Extract variables involved in the sink
    sink_code = lines[sink_line - 1]
    variables = re.findall(r'\b([a-zA-Z_]\w*)\b', sink_code)
    
    # Trace backward to find sources
    taint_flow = []
    sources = []
    sanitizers = []
    
    for var in variables:
        for i in range(sink_line - 1, method_start - 1, -1):
            line = lines[i]
            
            # Check if variable is assigned from user input
            if language == 'java':
                if f'{var} =' in line or f'{var}=' in line:
                    if 'getParameter(' in line or 'getHeader(' in line or 'getPathVariable(' in line:
                        sources.append({
                            'line': i + 1,
                            'variable': var,
                            'source': 'HTTP Request',
                            'code': line.strip()
                        })
                    
                    # Check for sanitization
                    if 'escape' in line.lower() or 'sanitize' in line.lower() or 'preparedStatement' in line:
                        sanitizers.append({
                            'line': i + 1,
                            'method': 'Sanitization detected',
                            'code': line.strip()
                        })
            
            elif language == 'python':
                if f'{var} =' in line or f'{var}=' in line:
                    if 'request.' in line and any(x in line for x in ['GET', 'POST', 'args', 'form', 'json']):
                        sources.append({
                            'line': i + 1,
                            'variable': var,
                            'source': 'HTTP Request',
                            'code': line.strip()
                        })
                    
                    if 'escape(' in line or 'clean(' in line or 'validate(' in line:
                        sanitizers.append({
                            'line': i + 1,
                            'method': 'Sanitization detected',
                            'code': line.strip()
                        })
    
    # Determine exploitability
    exploitable = len(sources) > 0 and len(sanitizers) == 0
    
    return {
        'exploitable': exploitable,
        'sources': sources,
        'sanitizers': sanitizers,
        'confidence': 'HIGH' if exploitable else 'MEDIUM',
        'data_flow': taint_flow
    }

# Example: Trace SQL injection
# analysis = taint_analysis('UserService.java', 42, 'java')
# if analysis['exploitable']:
#     print(f"[CRITICAL] Confirmed SQL injection at line 42")
#     print(f"Data flow: {analysis['sources']} -> sink (no sanitization)")
```

### Phase 3: Authentication & Authorization Analysis (Control-Driven)

```python
def analyze_auth_authorization(project_path, framework='spring'):
    """
    Control-driven analysis: Find missing security controls
    """
    findings = []
    
    if framework == 'spring':
        # Find security configuration
        security_config = None
        for root, dirs, files in os.walk(project_path):
            for file in files:
                if 'SecurityConfig' in file or 'WebSecurityConfig' in file:
                    security_config = os.path.join(root, file)
                    break
        
        if not security_config:
            findings.append({
                'type': 'Missing Security Configuration',
                'severity': 'HIGH',
                'description': 'No Spring Security configuration found',
                'recommendation': 'Implement WebSecurityConfigurerAdapter'
            })
        else:
            with open(security_config, 'r') as f:
                config = f.read()
                
                # Check for permitAll() abuse
                permit_all = re.findall(r'\.antMatchers\(["\']([^"\']+)["\']\)\.permitAll\(\)', config)
                for path in permit_all:
                    if not path.endswith(('.css', '.js', '.png', '.jpg')):
                        findings.append({
                            'type': 'Overly Permissive Access Control',
                            'severity': 'MEDIUM',
                            'path': path,
                            'description': f'Endpoint {path} allows unauthenticated access',
                            'recommendation': 'Review if authentication is required'
                        })
        
        # Find controllers and check for authorization
        for root, dirs, files in os.walk(project_path):
            for file in files:
                if file.endswith('Controller.java'):
                    filepath = os.path.join(root, file)
                    with open(filepath, 'r') as f:
                        content = f.read()
                        
                        # Find endpoints
                        endpoints = re.findall(r'@(Get|Post|Put|Delete)Mapping\(["\']([^"\']+)["\']', content)
                        
                        for method, path in endpoints:
                            # Check for authorization annotations
                            endpoint_context = content[max(0, content.find(path) - 500):content.find(path) + 100]
                            
                            has_preauthorize = '@PreAuthorize' in endpoint_context
                            has_secured = '@Secured' in endpoint_context
                            
                            if method in ['Post', 'Put', 'Delete'] and not (has_preauthorize or has_secured):
                                findings.append({
                                    'type': 'Missing Authorization Check',
                                    'severity': 'HIGH',
                                    'file': filepath,
                                    'endpoint': f'{method} {path}',
                                    'description': 'Mutating endpoint without @PreAuthorize or @Secured',
                                    'recommendation': 'Add @PreAuthorize annotation to verify permissions',
                                    'cwe': 'CWE-862'
                                })
    
    elif framework == 'django':
        # Check settings.py for middleware
        settings_path = os.path.join(project_path, 'settings.py')
        if os.path.exists(settings_path):
            with open(settings_path, 'r') as f:
                settings = f.read()
                
                required_middleware = [
                    'AuthenticationMiddleware',
                    'CsrfViewMiddleware',
                    'SecurityMiddleware'
                ]
                
                for mw in required_middleware:
                    if mw not in settings:
                        findings.append({
                            'type': 'Missing Security Middleware',
                            'severity': 'HIGH',
                            'middleware': mw,
                            'description': f'{mw} not found in MIDDLEWARE',
                            'recommendation': f'Add django.middleware.{mw.lower()}'
                        })
        
        # Check views for @login_required
        for root, dirs, files in os.walk(project_path):
            for file in files:
                if file.endswith('views.py'):
                    filepath = os.path.join(root, file)
                    with open(filepath, 'r') as f:
                        lines = f.readlines()
                        
                        for i, line in enumerate(lines):
                            if re.match(r'def\s+\w+\s*\(request', line):
                                # Check if @login_required is present above
                                decorator_found = False
                                for j in range(max(0, i - 3), i):
                                    if '@login_required' in lines[j] or '@permission_required' in lines[j]:
                                        decorator_found = True
                                        break
                                
                                if not decorator_found:
                                    findings.append({
                                        'type': 'Missing Authentication Decorator',
                                        'severity': 'MEDIUM',
                                        'file': filepath,
                                        'line': i + 1,
                                        'description': 'View function without @login_required',
                                        'recommendation': 'Add @login_required decorator if authentication needed'
                                    })
    
    return findings
```

### Phase 3: Business Logic Vulnerabilities (Control-Driven)

```python
def detect_business_logic_flaws(project_path, language='java'):
    """
    Find business logic vulnerabilities:
    - Race conditions
    - Mass assignment
    - Price manipulation
    - IDOR patterns
    """
    findings = []
    
    # Race condition patterns
    race_patterns = {
        'java': [
            (r'if\s*\([^)]*balance[^)]*\)\s*{[^}]*balance\s*[-=]', 'TOCTOU in balance check'),
            (r'if\s*\([^)]*stock[^)]*\)\s*{[^}]*stock\s*[-=]', 'TOCTOU in stock check'),
        ],
        'python': [
            (r'if\s+.*?\.balance\s*[<>=].*?:\s*\n\s+.*?\.balance\s*[-+]=', 'TOCTOU in balance update'),
            (r'if\s+.*?\.quantity\s*[<>=].*?:\s*\n\s+.*?\.quantity\s*[-+]=', 'TOCTOU in quantity check'),
        ]
    }
    
    # Mass assignment patterns
    mass_assignment_patterns = {
        'java': [
            (r'\.setAll\(', 'Bulk property setter'),
            (r'BeanUtils\.copyProperties\(', 'BeanUtils copy without whitelist'),
        ],
        'python': [
            (r'for\s+key\s+in\s+request\.\w+:', 'Loop over request data'),
            (r'\.update\(request\.(?:POST|json)\)', 'Model.update() with request data'),
        ]
    }
    
    for root, dirs, files in os.walk(project_path):
        for file in files:
            if (language == 'java' and file.endswith('.java')) or \
               (language == 'python' and file.endswith('.py')):
                
                filepath = os.path.join(root, file)
                with open(filepath, 'r', encoding='utf-8', errors='ignore') as f:
                    content = f.read()
                    lines = content.split('\n')
                
                # Check race conditions
                for pattern, description in race_patterns.get(language, []):
                    matches = re.finditer(pattern, content)
                    for match in matches:
                        line_num = content[:match.start()].count('\n') + 1
                        findings.append({
                            'type': 'Race Condition',
                            'severity': 'HIGH',
                            'file': filepath,
                            'line': line_num,
                            'description': description,
                            'recommendation': 'Use database transactions or locks',
                            'cwe': 'CWE-362'
                        })
                
                # Check mass assignment
                for pattern, description in mass_assignment_patterns.get(language, []):
                    matches = re.finditer(pattern, content)
                    for match in matches:
                        line_num = content[:match.start()].count('\n') + 1
                        findings.append({
                            'type': 'Mass Assignment',
                            'severity': 'MEDIUM',
                            'file': filepath,
                            'line': line_num,
                            'description': description,
                            'recommendation': 'Use explicit whitelist of allowed fields',
                            'cwe': 'CWE-915'
                        })
                
                # Check IDOR patterns (missing authorization)
                if language == 'java':
                    # Look for direct ID usage without ownership check
                    idor_patterns = [
                        r'findById\(.*?getId\(\)',
                        r'getById\(.*?getParameter\(',
                        r'delete\(.*?getPathVariable\('
                    ]
                    
                    for pattern in idor_patterns:
                        matches = re.finditer(pattern, content)
                        for match in matches:
                            line_num = content[:match.start()].count('\n') + 1
                            # Check if there's authorization within 10 lines
                            context_start = max(0, line_num - 10)
                            context_end = min(len(lines), line_num + 10)
                            context = '\n'.join(lines[context_start:context_end])
                            
                            if not re.search(r'(checkOwnership|verifyAccess|hasPermission|principal\.getId)', context):
                                findings.append({
                                    'type': 'IDOR (Insecure Direct Object Reference)',
                                    'severity': 'HIGH',
                                    'file': filepath,
                                    'line': line_num,
                                    'description': 'Direct ID usage without ownership verification',
                                    'recommendation': 'Verify user owns the resource before access',
                                    'cwe': 'CWE-639'
                                })
    
    return findings
```

### Phase 4: Vulnerability Validation with POC

```python
def generate_
