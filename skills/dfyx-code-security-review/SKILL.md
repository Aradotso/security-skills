---
name: dfyx-code-security-review
description: AI-powered expert-level code security auditing with deep data flow analysis, multi-language support, and systematic vulnerability detection
triggers:
  - audit this project for security vulnerabilities
  - perform a code security review
  - check for security issues in the codebase
  - run a security audit on this code
  - analyze this project for vulnerabilities
  - find security flaws in this application
  - conduct a white-box security assessment
  - review code for OWASP Top 10 issues
---

# dfyx Code Security Review

> Skill by [ara.so](https://ara.so) — Security Skills collection.

A comprehensive AI-driven code security auditing framework supporting 9 programming languages with a five-phase standardized audit protocol. Uses sink-driven, control-driven, and config-driven analysis models to systematically discover injection flaws, authentication bypasses, authorization issues, and business logic vulnerabilities.

## Overview

**dfyx_code_security_review** provides expert-level white-box static analysis through:

- **9 Languages**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 Security Dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **3 Audit Models**: Sink-driven (injection/RCE), Control-driven (authorization/logic), Config-driven (settings/crypto)
- **5-Phase Protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting
- **Real-World Cases**: Based on WooYun vulnerability database (2010-2016)

## Installation

### For AI Coding Agents

This skill is automatically loaded when agents detect security audit requests. No manual installation required.

### For Manual Use

```bash
# Clone or copy to AI client skills directory
# Claude Code
cp -r dfyx_code_security_review ~/.claude/skills/

# Install Python dependencies for scripts
pip install -r requirements.txt
```

**requirements.txt**:
```
bandit>=1.7.5
semgrep>=1.45.0
safety>=2.3.5
pyyaml>=6.0
requests>=2.31.0
tree-sitter>=0.20.0
tree-sitter-python>=0.20.0
tree-sitter-java>=0.20.0
tree-sitter-javascript>=0.20.0
```

## Five-Phase Audit Protocol

### Phase 1: Reconnaissance (10%)

Understand architecture, map attack surface, identify entry points.

```python
# scripts/code_scan.py - Phase 1 execution
import os
from pathlib import Path
from typing import Dict, List

def reconnaissance_phase(project_path: str) -> Dict:
    """
    Phase 1: Map architecture and attack surface
    Returns: {
        'tech_stack': dict,
        'entry_points': list,
        'attack_surface': list,
        'dependencies': list
    }
    """
    results = {
        'tech_stack': identify_tech_stack(project_path),
        'entry_points': find_entry_points(project_path),
        'attack_surface': map_attack_surface(project_path),
        'dependencies': analyze_dependencies(project_path)
    }
    return results

def identify_tech_stack(path: str) -> Dict:
    """Detect languages, frameworks, and versions"""
    stack = {'languages': [], 'frameworks': [], 'databases': []}
    
    # Language detection
    if Path(path, 'pom.xml').exists() or Path(path, 'build.gradle').exists():
        stack['languages'].append('Java')
        stack['frameworks'].extend(detect_java_frameworks(path))
    
    if Path(path, 'requirements.txt').exists() or Path(path, 'setup.py').exists():
        stack['languages'].append('Python')
        stack['frameworks'].extend(detect_python_frameworks(path))
    
    if Path(path, 'package.json').exists():
        stack['languages'].append('JavaScript')
        stack['frameworks'].extend(detect_js_frameworks(path))
    
    if Path(path, 'go.mod').exists():
        stack['languages'].append('Go')
        stack['frameworks'].extend(detect_go_frameworks(path))
    
    return stack

def find_entry_points(path: str) -> List[Dict]:
    """Identify API endpoints, routes, controllers"""
    entry_points = []
    
    # Spring Boot endpoints
    for file_path in Path(path).rglob('*Controller.java'):
        with open(file_path) as f:
            content = f.read()
            if '@RestController' in content or '@Controller' in content:
                entry_points.extend(extract_spring_endpoints(content, file_path))
    
    # Flask/Django routes
    for file_path in Path(path).rglob('*.py'):
        with open(file_path) as f:
            content = f.read()
            if '@app.route' in content or '@api.route' in content:
                entry_points.extend(extract_flask_routes(content, file_path))
    
    # Express routes
    for file_path in Path(path).rglob('*.js'):
        with open(file_path) as f:
            content = f.read()
            if 'app.get(' in content or 'router.post(' in content:
                entry_points.extend(extract_express_routes(content, file_path))
    
    return entry_points
```

### Phase 2: Pattern Matching (30%)

Parallel scanning for dangerous functions across 10 security dimensions.

```python
# scripts/pattern_scanner.py
import re
from typing import List, Dict

# D1: Injection patterns
INJECTION_PATTERNS = {
    'sql_injection': {
        'java': [
            r'Statement\.execute\(',
            r'createStatement\(\)\.execute',
            r'createQuery\([^?]*\+',  # String concatenation
        ],
        'python': [
            r'cursor\.execute\([^?]*%',
            r'\.raw\([^?]*\+',
            r'\.extra\(.*where=.*\+',
        ],
        'go': [
            r'db\.Exec\([^?]*\+',
            r'db\.Query\([^?]*fmt\.Sprintf',
        ]
    },
    'command_injection': {
        'java': [
            r'Runtime\.getRuntime\(\)\.exec\(',
            r'ProcessBuilder\(',
        ],
        'python': [
            r'os\.system\(',
            r'subprocess\.call\([^[]',
            r'eval\(',
            r'exec\(',
        ],
        'javascript': [
            r'child_process\.exec\(',
            r'eval\(',
        ]
    },
    'template_injection': {
        'java': [
            r'FreeMarkerConfigurationFactory',
            r'\.setTemplateLoader\(',
            r'SpelExpressionParser',
        ],
        'python': [
            r'render_template_string\(',
            r'jinja2\.Template\(',
        ]
    }
}

# D3: Authorization patterns
AUTHZ_PATTERNS = {
    'missing_checks': {
        'java': [
            r'@GetMapping.*@PathVariable.*id',  # CRUD without @PreAuthorize
            r'@DeleteMapping(?!.*@PreAuthorize)',
        ],
        'python': [
            r'@app\.route.*<int:id>',  # No permission decorator
            r'def delete_.*\(.*id.*\):(?!.*check_permission)',
        ]
    }
}

def scan_for_patterns(file_path: str, language: str) -> List[Dict]:
    """
    Scan file for dangerous patterns
    Returns list of findings with context
    """
    findings = []
    
    with open(file_path) as f:
        lines = f.readlines()
    
    # D1: Injection scanning
    for vuln_type, patterns in INJECTION_PATTERNS.items():
        if language in patterns:
            for pattern in patterns[language]:
                for line_num, line in enumerate(lines, 1):
                    if re.search(pattern, line):
                        findings.append({
                            'type': vuln_type,
                            'dimension': 'D1_Injection',
                            'file': file_path,
                            'line': line_num,
                            'code': line.strip(),
                            'pattern': pattern,
                            'severity': 'CRITICAL'
                        })
    
    # D3: Authorization scanning
    for check_type, patterns in AUTHZ_PATTERNS.items():
        if language in patterns:
            for pattern in patterns[language]:
                for line_num, line in enumerate(lines, 1):
                    if re.search(pattern, line):
                        # Check surrounding lines for security controls
                        context = ''.join(lines[max(0, line_num-5):line_num+5])
                        if not has_security_control(context, language):
                            findings.append({
                                'type': 'missing_authorization',
                                'dimension': 'D3_Authorization',
                                'file': file_path,
                                'line': line_num,
                                'code': line.strip(),
                                'severity': 'HIGH'
                            })
    
    return findings

def has_security_control(context: str, language: str) -> bool:
    """Check if security control exists in context"""
    if language == 'java':
        return any(s in context for s in ['@PreAuthorize', '@Secured', '@RolesAllowed'])
    elif language == 'python':
        return any(s in context for s in ['@login_required', '@permission_required', 'check_permission'])
    return False
```

### Phase 3: Taint Tracking (40%)

Deep data flow analysis from source to sink.

```python
# scripts/data_flow_analyzer.py
from typing import List, Dict, Set
from dataclasses import dataclass

@dataclass
class TaintSource:
    """User-controlled input source"""
    variable: str
    type: str  # 'http_param', 'path_variable', 'header', 'cookie'
    location: str
    line: int

@dataclass
class TaintSink:
    """Dangerous function call"""
    function: str
    arguments: List[str]
    location: str
    line: int

class DataFlowAnalyzer:
    def __init__(self, language: str):
        self.language = language
        self.taint_sources = []
        self.taint_sinks = []
        self.sanitizers = set()
    
    def trace_taint_flow(self, source: TaintSource, sink: TaintSink, code: str) -> Dict:
        """
        Trace data flow from source to sink
        Returns: {
            'vulnerable': bool,
            'flow_path': list,
            'sanitizers_found': list,
            'bypassed': bool
        }
        """
        # Build control flow graph
        cfg = self.build_cfg(code)
        
        # Find all paths from source to sink
        paths = self.find_paths(cfg, source, sink)
        
        for path in paths:
            sanitizers = self.identify_sanitizers_in_path(path)
            
            # Check if sanitizers are bypassable
            if not sanitizers or self.can_bypass_sanitizers(sanitizers, source.type):
                return {
                    'vulnerable': True,
                    'flow_path': path,
                    'sanitizers_found': sanitizers,
                    'bypassed': len(sanitizers) > 0,
                    'exploit_vector': self.generate_exploit_vector(source, sink, sanitizers)
                }
        
        return {'vulnerable': False, 'flow_path': [], 'sanitizers_found': [], 'bypassed': False}
    
    def identify_sanitizers_in_path(self, path: List) -> List[str]:
        """Find sanitization functions in data flow path"""
        sanitizers = []
        
        if self.language == 'java':
            patterns = [
                'PreparedStatement.setString',
                'StringEscapeUtils.escapeSql',
                'ESAPI.encoder().encodeForSQL',
                'Pattern.quote'
            ]
        elif self.language == 'python':
            patterns = [
                'cursor.execute(',  # With parameterization
                'escape_string(',
                'quote(',
                're.escape('
            ]
        
        for node in path:
            for pattern in patterns:
                if pattern in node.get('code', ''):
                    sanitizers.append(pattern)
        
        return sanitizers
    
    def can_bypass_sanitizers(self, sanitizers: List[str], input_type: str) -> bool:
        """Check if sanitizers can be bypassed"""
        # Weak sanitizers
        weak = ['addslashes', 'htmlspecialchars', 'strip_tags']
        
        # Context-specific bypasses
        if input_type == 'json' and 'json.loads' in sanitizers:
            return False  # JSON parsing is safe
        
        if any(w in str(sanitizers) for w in weak):
            return True
        
        # Check for incomplete sanitization
        if len(sanitizers) == 1 and 'escape' in sanitizers[0]:
            return True  # Single escape often insufficient
        
        return False

# Example: SQL injection taint analysis
def analyze_sql_injection(file_path: str) -> List[Dict]:
    """
    Complete SQL injection analysis with taint tracking
    """
    analyzer = DataFlowAnalyzer('java')
    vulnerabilities = []
    
    with open(file_path) as f:
        code = f.read()
    
    # Find sources (user input)
    sources = [
        TaintSource('username', 'http_param', file_path, 45),
        TaintSource('userId', 'path_variable', file_path, 47)
    ]
    
    # Find sinks (SQL execution)
    sinks = [
        TaintSink('Statement.execute', ['query'], file_path, 52)
    ]
    
    # Trace each source-sink pair
    for source in sources:
        for sink in sinks:
            result = analyzer.trace_taint_flow(source, sink, code)
            
            if result['vulnerable']:
                vulnerabilities.append({
                    'type': 'SQL Injection',
                    'severity': 'CRITICAL',
                    'source': f"{source.variable} ({source.type})",
                    'sink': f"{sink.function} at line {sink.line}",
                    'flow_path': result['flow_path'],
                    'sanitizers': result['sanitizers_found'],
                    'bypassed': result['bypassed'],
                    'exploit': result.get('exploit_vector', ''),
                    'file': file_path
                })
    
    return vulnerabilities
```

### Phase 4: Validation & Attack Chain (15%)

Validate findings and construct exploit chains.

```python
# scripts/vulnerability_validator.py
import subprocess
import json
from typing import Dict, List

class VulnerabilityValidator:
    """Validate vulnerabilities with actual testing"""
    
    def validate_sql_injection(self, vulnerability: Dict) -> Dict:
        """
        Validate SQL injection with payloads
        """
        result = {
            'confirmed': False,
            'payloads_tested': [],
            'evidence': []
        }
        
        # Common SQLi payloads
        payloads = [
            "' OR '1'='1",
            "' UNION SELECT NULL--",
            "'; WAITFOR DELAY '00:00:05'--",
            "' AND SLEEP(5)--"
        ]
        
        for payload in payloads:
            # Simulate request (in real audit, use actual HTTP client)
            test_result = self.test_payload(
                vulnerability['endpoint'],
                vulnerability['parameter'],
                payload
            )
            
            result['payloads_tested'].append(payload)
            
            if test_result['vulnerable']:
                result['confirmed'] = True
                result['evidence'].append({
                    'payload': payload,
                    'response': test_result['response'],
                    'proof': test_result['proof']
                })
                break
        
        return result
    
    def build_attack_chain(self, vulnerabilities: List[Dict]) -> List[Dict]:
        """
        Construct multi-step attack chains
        Example: IDOR + SQL Injection = Full database access
        """
        chains = []
        
        # Find complementary vulnerabilities
        authz_issues = [v for v in vulnerabilities if v['dimension'] == 'D3_Authorization']
        injection_issues = [v for v in vulnerabilities if v['dimension'] == 'D1_Injection']
        
        for authz in authz_issues:
            for injection in injection_issues:
                if self.can_chain(authz, injection):
                    chains.append({
                        'title': 'Authorization Bypass → SQL Injection',
                        'severity': 'CRITICAL',
                        'steps': [
                            {
                                'step': 1,
                                'vuln': authz,
                                'action': 'Bypass authorization to access admin endpoint'
                            },
                            {
                                'step': 2,
                                'vuln': injection,
                                'action': 'Inject SQL payload to extract database'
                            }
                        ],
                        'impact': 'Complete database compromise',
                        'poc': self.generate_chain_poc(authz, injection)
                    })
        
        return chains
    
    def can_chain(self, vuln1: Dict, vuln2: Dict) -> bool:
        """Check if two vulnerabilities can be chained"""
        # Same application/module
        if vuln1.get('module') != vuln2.get('module'):
            return False
        
        # Logical connection
        if 'endpoint' in vuln1 and 'endpoint' in vuln2:
            return vuln1['endpoint'].split('/')[1] == vuln2['endpoint'].split('/')[1]
        
        return False
```

### Phase 5: Reporting (5%)

Generate structured security audit reports.

```python
# scripts/report_generator.py
from datetime import datetime
from typing import Dict, List

class ReportGenerator:
    def generate_report(self, vulnerabilities: List[Dict], metadata: Dict) -> str:
        """
        Generate complete security audit report
        """
        report = f"""# Security Audit Report
**Project**: {metadata['project_name']}
**Date**: {datetime.now().strftime('%Y-%m-%d')}
**Auditor**: AI Code Security Review
**Scope**: {', '.join(metadata['tech_stack']['languages'])}

## Executive Summary

Total Findings: {len(vulnerabilities)}
- Critical: {len([v for v in vulnerabilities if v['severity'] == 'CRITICAL'])}
- High: {len([v for v in vulnerabilities if v['severity'] == 'HIGH'])}
- Medium: {len([v for v in vulnerabilities if v['severity'] == 'MEDIUM'])}
- Low: {len([v for v in vulnerabilities if v['severity'] == 'LOW'])}

## Critical Findings

"""
        
        for vuln in sorted(vulnerabilities, key=lambda x: x['severity'], reverse=True):
            if vuln['severity'] == 'CRITICAL':
                report += self.format_vulnerability(vuln)
        
        report += "\n## Detailed Findings\n\n"
        
        for vuln in vulnerabilities:
            report += self.format_vulnerability(vuln)
        
        report += "\n## Recommendations\n\n"
        report += self.generate_recommendations(vulnerabilities)
        
        return report
    
    def format_vulnerability(self, vuln: Dict) -> str:
        """Format single vulnerability for report"""
        return f"""
### {vuln['type']} - {vuln['severity']}

**Location**: `{vuln['file']}:{vuln.get('line', 'N/A')}`

**Description**: 
{vuln.get('description', 'Vulnerability detected by pattern matching and data flow analysis.')}

**Vulnerable Code**:
```{vuln.get('language', 'text')}
{vuln.get('code', 'N/A')}
```

**Data Flow**:
{self.format_data_flow(vuln.get('flow_path', []))}

**Proof of Concept**:
```
{vuln.get('exploit', 'N/A')}
```

**Impact**: {vuln.get('impact', 'See OWASP guidelines for this vulnerability type.')}

**Remediation**:
```{vuln.get('language', 'text')}
{vuln.get('fix', 'Use parameterized queries, input validation, and proper encoding.')}
```

**References**:
- OWASP: {vuln.get('owasp_ref', 'N/A')}
- CWE: {vuln.get('cwe', 'N/A')}

---
"""
    
    def format_data_flow(self, flow_path: List) -> str:
        """Format data flow visualization"""
        if not flow_path:
            return "No data flow traced."
        
        flow = "```\n"
        for i, node in enumerate(flow_path):
            flow += f"{i+1}. {node.get('description', node)}\n"
        flow += "```\n"
        return flow
```

## 10 Security Dimensions

### D1: Injection Vulnerabilities

**Detection Strategy**: Sink-driven analysis

```python
# Example: Command injection detection
def detect_command_injection(file_path: str) -> List:
    dangerous_functions = {
        'python': ['os.system', 'subprocess.call', 'eval', 'exec'],
        'java': ['Runtime.getRuntime().exec', 'ProcessBuilder'],
        'javascript': ['child_process.exec', 'eval'],
        'php': ['exec', 'system', 'passthru', 'shell_exec']
    }
    
    findings = []
    # Scan for patterns and trace taint
    # ... (see pattern_scanner.py)
    return findings
```

### D2: Authentication Issues

**Detection Strategy**: Config-driven + control-driven

```python
# Example: Weak JWT detection
def audit_jwt_security(project_path: str) -> List:
    issues = []
    
    # Check for hardcoded secrets
    for file in find_config_files(project_path):
        content = read_file(file)
        if 'jwt.secret' in content or 'JWT_SECRET' in content:
            if not uses_env_var(content):
                issues.append({
                    'type': 'Hardcoded JWT Secret',
                    'severity': 'CRITICAL',
                    'file': file,
                    'recommendation': 'Use environment variable: JWT_SECRET=<value>'
                })
    
    # Check for weak algorithms
    for file in find_source_files(project_path):
        if 'algorithm: "none"' in read_file(file) or '.verify(signature=False)' in read_file(file):
            issues.append({
                'type': 'JWT Algorithm None',
                'severity': 'CRITICAL',
                'file': file
            })
    
    return issues
```

### D3: Authorization Flaws

**Detection Strategy**: Control-driven (enumerate endpoints, check for missing controls)

```python
# Example: IDOR detection
def detect_idor(project_path: str, language: str) -> List:
    """
    Insecure Direct Object Reference detection
    """
    issues = []
    
    if language == 'java':
        # Find CRUD endpoints
        for controller in find_controllers(project_path):
            endpoints = extract_endpoints(controller)
            
            for endpoint in endpoints:
                if has_path_variable(endpoint, 'id'):
                    # Check for authorization
                    if not has_preauthorize(endpoint):
                        issues.append({
                            'type': 'Missing Authorization Check (IDOR)',
                            'severity': 'HIGH',
                            'endpoint': endpoint['path'],
                            'file': controller,
                            'recommendation': 'Add @PreAuthorize("hasPermission(#id, \'Entity\', \'read\')")'
                        })
    
    return issues
```

### D4: Deserialization

**Detection Strategy**: Sink-driven (gadget chain analysis)

```python
# Example: Java deserialization
def detect_unsafe_deserialization(file_path: str) -> List:
    patterns = [
        'ObjectInputStream.readObject()',
        'XMLDecoder.readObject()',
        'XStream.fromXML()',
        'ObjectMapper.readValue(',  # Without type validation
        'JSON.parseObject(',  # Fastjson
    ]
    
    findings = []
    content = read_file(file_path)
    
    for pattern in patterns:
        if pattern in content:
            # Check for input source
            source = trace_input_source(content, pattern)
            if source and source['type'] in ['http_body', 'http_param']:
                findings.append({
                    'type': 'Unsafe Deserialization',
                    'severity': 'CRITICAL',
                    'pattern': pattern,
                    'source': source,
                    'gadgets': check_for_gadgets(file_path)
                })
    
    return findings
```

### D9: Business Logic Vulnerabilities

**Detection Strategy**: Control-driven (verify state machines, transaction logic)

```python
# Example: Race condition detection
def detect_race_conditions(project_path: str) -> List:
    """
    Find TOCTOU (Time-of-Check-Time-of-Use) vulnerabilities
    """
    issues = []
    
    for file in find_source_files(project_path):
        code = read_file(file)
        
        # Pattern: check balance -> deduct balance (no lock)
        if 'getBalance()' in code and 'setBalance(' in code:
            # Check for synchronization
            if not has_lock(code) and not has_transaction(code):
                issues.append({
                    'type': 'Race Condition - Balance Check',
                    'severity': 'HIGH',
                    'file': file,
                    'description': 'Balance check and deduction not atomic',
                    'poc': 'Send multiple concurrent withdrawal requests',
                    'fix': 'Use @Transactional with isolation level or explicit locking'
                })
    
    return issues
```

## Audit Modes

### Quick Mode (5-10 minutes)

```python
# Quick audit - high-priority checks only
def quick_audit(project_path: str) -> Dict:
    return {
        'secrets': detect_secrets(project_path),
        'critical_injection': scan_critical_sinks(project_path),
        'dependency_cve': check_dependencies(project_path)
    }
```

### Standard Mode (30-60 minutes)

```python
# Standard audit - OWASP Top 10 coverage
def standard_audit(project_path: str) -> Dict:
    return {
        **quick_audit(project_path),
        'authentication': audit_authentication(project_path),
        'authorization': audit_authorization(project_path),
        'cryptography': audit_cryptography(project_path),
        'file_operations': audit_file_ops(project_path)
    }
```

### Deep Mode (1-3 hours)

```python
# Deep audit - full coverage with attack chains
def deep_audit(project_path: str) -> Dict:
    results = standard_audit(project_path)
    
    # Add deep analysis
    results['business_logic'] = audit_business_logic(project_path)
    results['deserialization'] = audit_deserialization(project_path)
    results['ssrf'] = audit_ssrf(project_path)
    results['attack_chains'] = build_attack_chains(results)
    
    return results
```

## Language-Specific Patterns

### Java (Spring Boot)

```java
// Vulnerable: SQL injection
@GetMapping("/user/{id}")
public User getUser(@PathVariable String id) {
    String query = "SELECT * FROM users WHERE id = " + id; // VULNERABLE
    return jdbcTemplate.query(query);
}

// Secure: Parameterized query
@GetMapping("/user/{id}")
public User getUser(@PathVariable String id) {
    String query = "SELECT * FROM users WHERE id = ?";
    return jdbcTemplate.queryForObject(query, new Object[]{id}, userRowMapper);
}

// Vulnerable: Missing authorization
@DeleteMapping("/user/{id}")
public void deleteUser(@PathVariable Long id) { // VULNERABLE - no @PreAuthorize
    userService.delete(id);
}

// Secure: With authorization
@DeleteMapping("/user/{id}")
@PreAuthorize("hasRole('ADMIN') or @userSecurity.isOwner(#id)")
public void deleteUser(@PathVariable Long id) {
    userService.delete(id);
}
```

### Python (Django/Flask)

```python
# Vulnerable: SQL injection
def get_user(request):
    user_id = request.GET.get('id')
    query = f"SELECT * FROM users WHERE id = {user_id}"  # VULNERABLE
    cursor.execute(query)

# Secure: Parameterized query
def get_user(request):
    user_id = request.GET.get('id')
    cursor.execute("SELECT * FROM users WHERE id = %s", [user_id])

# Vulnerable: Command injection
import os
def ping_host(request):
    host = request.POST.get('host')
    os.system(f"ping -c 4 {host}")  # VULNERABLE

# Secure: Input validation + safe API
import subprocess
def ping_host(request):
    host = request.POST.get('host')
    if not re.match(r'^[\w\.-]+$', host):
        return "Invalid host"
    subprocess.run(['ping', '-c', '4', host], check=True)
```

### JavaScript (Node.js/Express)

```javascript
// Vulnerable: NoSQL injection
app.get('/user', (req, res) => {
    const username = req.query.username;
    db.collection('users').findOne({username: username}); // VULNERABLE if object passed
});

// Secure: Type validation
app.get('/user', (req, res) => {
    const username = req.query.username;
    if (typeof username !== 'string') {
        return res.status(400).send('Invalid input');
    }
    db.collection('users').findOne({username: username});
});

// Vulnerable: Prototype pollution
const merge = (target, source) => {
    for (let key in source) {
        target[key] = source[key]; // VULNERABLE
    }
};

// Secure: Avoid __proto__
const merge = (target, source) => {
    for (let key in source) {
        if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
            continue;
        }
        target[key] = source[key];
    }
};
```

### Go

```go
// Vulnerable: SQL injection
func GetUser(w http.ResponseWriter, r *http.Request) {
    id := r.URL.Query().Get("id")
    query := "SELECT * FROM users WHERE id = " + id // VULNERABLE
    db.Query(query)
}

// Secure: Parameterized query
func GetUser(w http.ResponseWriter, r *http.Request) {
    id := r.URL.Query().Get("id")
    query := "SELECT * FROM users WHERE id = $
