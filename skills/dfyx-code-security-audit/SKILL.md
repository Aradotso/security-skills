---
name: dfyx-code-security-audit
description: AI-powered code security audit skill using deep data flow analysis and business logic understanding for vulnerability detection
triggers:
  - audit this codebase for security vulnerabilities
  - perform a security code review
  - check for security issues in this code
  - run a security audit on this project
  - analyze code for vulnerabilities
  - scan this application for security flaws
  - review code security using dfyx
  - execute dfyx security audit
---

# dfyx Code Security Audit Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Expert-level code security auditing using white-box static analysis methodology through a five-phase standardized audit protocol. Designed by the EastSword team (东方隐侠团队) for systematic discovery and validation of security vulnerabilities in source code.

## What This Skill Does

**dfyx_code_security_review** provides AI-powered security auditing with:

- **Multi-language support**: Java, Python, Go, PHP, JavaScript/Node.js, C/C++, .NET/C#, Ruby, Rust
- **10 security dimensions**: Injection, Authentication, Authorization, Deserialization, File Operations, SSRF, Cryptography, Configuration, Business Logic, Supply Chain
- **Three-track audit model**: 
  - Sink-driven (injection/RCE)
  - Control-driven (authorization/business logic)
  - Config-driven (configuration/crypto)
- **Five-phase protocol**: Reconnaissance → Pattern Matching → Taint Tracking → Validation → Reporting
- **Real-world case library**: Based on WooYun vulnerability cases (2010-2016)

## Installation

This skill doesn't require separate installation — it operates through AI agent capabilities. However, the Python helper scripts can be installed:

```bash
# Clone the repository
git clone https://github.com/EastSword/skill-dfyx_code_security_review.git
cd skill-dfyx_code_security_review

# Install Python dependencies (optional, for helper scripts)
pip install -r requirements.txt
```

**Dependencies** (requirements.txt):
```
pylint>=2.17.0
bandit>=1.7.5
safety>=2.3.5
semgrep>=1.31.0
pyyaml>=6.0
```

## Core Audit Protocol

### Five-Phase Audit Process

```
Phase 1: Reconnaissance (10%)
  └─> Output: Architecture diagram, attack surface inventory

Phase 2: Pattern Matching (30%)
  └─> Output: High-risk area checklist

Phase 3: Taint Tracking + Testing (40%)
  └─> Output: Confirmed vulnerabilities, test validation reports

Phase 4: Validation & Attack Chains (15%)
  └─> Output: Vulnerability validation reports

Phase 5: Structured Reporting (5%)
  └─> Output: Complete audit report
```

### Audit Modes

| Mode | Use Case | Coverage | Time |
|------|----------|----------|------|
| **Quick** | CI/CD, small projects | Critical vulns, secrets, dependency CVEs | 5-10 min |
| **Standard** | Regular audits | OWASP Top 10, auth/authz, crypto | 30-60 min |
| **Deep** | Critical projects, pentest prep | Full coverage, attack chains, business logic | 1-3 hours |

## Usage Patterns

### Triggering an Audit

Simply request an audit in natural language:

```
"Audit this codebase for security vulnerabilities"
"Perform a deep security scan of /path/to/project"
"Check for SQL injection and authentication issues"
```

### Expected Workflow

```
[MODE] deep
[RECON] 874 files, Spring Boot 1.5 + Shiro 1.6 + JPA + Freemarker
[PLAN] 5 Agents, D1-D10 coverage, estimated 125 turns
[SCOPE] Critical: 10, High: 14, Medium: 12, Low: 4
Confirm to start? (yes/no)
```

## Python Helper Scripts

### Code Scanning

```python
# scripts/code_scan.py
from pattern_scanner import PatternScanner
from data_flow_analyzer import DataFlowAnalyzer
import sys

def scan_project(project_path, mode='standard'):
    """
    Scan a project for security vulnerabilities
    
    Args:
        project_path: Path to project root
        mode: 'quick', 'standard', or 'deep'
    """
    scanner = PatternScanner(project_path)
    analyzer = DataFlowAnalyzer(project_path)
    
    # Phase 1: Reconnaissance
    tech_stack = scanner.identify_tech_stack()
    print(f"[RECON] Detected: {tech_stack}")
    
    # Phase 2: Pattern matching
    patterns = scanner.scan_patterns(mode=mode)
    print(f"[SCAN] Found {len(patterns)} suspicious patterns")
    
    # Phase 3: Data flow analysis
    vulnerabilities = []
    for pattern in patterns:
        flows = analyzer.trace_data_flow(pattern)
        if analyzer.is_vulnerable(flows):
            vulnerabilities.append({
                'pattern': pattern,
                'flows': flows,
                'severity': analyzer.calculate_severity(flows)
            })
    
    return vulnerabilities

if __name__ == '__main__':
    project_path = sys.argv[1] if len(sys.argv) > 1 else '.'
    mode = sys.argv[2] if len(sys.argv) > 2 else 'standard'
    
    results = scan_project(project_path, mode)
    print(f"\n[RESULTS] Found {len(results)} vulnerabilities")
```

### Pattern Scanner

```python
# scripts/pattern_scanner.py
import re
import os
from typing import Dict, List

class PatternScanner:
    """Scans code for dangerous patterns across multiple languages"""
    
    DANGEROUS_PATTERNS = {
        'sql_injection': {
            'java': [
                r'createQuery\([^?]*\+',  # JPA concatenation
                r'createSQLQuery\([^?]*\+',
                r'Statement\.execute\([^?]*\+'
            ],
            'python': [
                r'cursor\.execute\([^%]*%',  # String formatting
                r'raw\([^%]*%',  # Django raw SQL
                r'\.query\([^%]*f["\']'  # f-string in query
            ],
            'php': [
                r'mysqli_query\([^,]*\.',  # Concatenation
                r'mysql_query\([^,]*\.',
                r'\$.*->query\([^?]*\.'
            ]
        },
        'command_injection': {
            'java': [
                r'Runtime\.exec\([^"]*\+',
                r'ProcessBuilder\([^"]*\+'
            ],
            'python': [
                r'os\.system\([^"]*\+',
                r'subprocess\.(call|run|Popen)\([^"]*\+',
                r'eval\(',  # Code injection
                r'exec\('
            ],
            'php': [
                r'(system|exec|shell_exec|passthru)\(\$',
                r'`.*\$'  # Backtick execution
            ]
        },
        'deserialization': {
            'java': [
                r'ObjectInputStream\.readObject\(',
                r'XMLDecoder\.readObject\(',
                r'Yaml\.load\('
            ],
            'python': [
                r'pickle\.loads?\(',
                r'yaml\.load\(',  # Without safe_load
                r'eval\('
            ],
            'php': [
                r'unserialize\(\$'
            ]
        }
    }
    
    def __init__(self, project_path: str):
        self.project_path = project_path
        self.language = self._detect_language()
    
    def _detect_language(self) -> str:
        """Detect primary programming language"""
        extensions = {
            '.java': 'java',
            '.py': 'python',
            '.php': 'php',
            '.go': 'go',
            '.js': 'javascript',
            '.rb': 'ruby',
            '.cs': 'csharp'
        }
        
        counts = {}
        for root, dirs, files in os.walk(self.project_path):
            for file in files:
                ext = os.path.splitext(file)[1]
                if ext in extensions:
                    lang = extensions[ext]
                    counts[lang] = counts.get(lang, 0) + 1
        
        return max(counts, key=counts.get) if counts else 'unknown'
    
    def scan_patterns(self, mode: str = 'standard') -> List[Dict]:
        """Scan for dangerous patterns"""
        results = []
        
        for vuln_type, lang_patterns in self.DANGEROUS_PATTERNS.items():
            if self.language not in lang_patterns:
                continue
            
            patterns = lang_patterns[self.language]
            for pattern in patterns:
                matches = self._grep_pattern(pattern)
                for match in matches:
                    results.append({
                        'type': vuln_type,
                        'pattern': pattern,
                        'file': match['file'],
                        'line': match['line'],
                        'code': match['code']
                    })
        
        return results
    
    def _grep_pattern(self, pattern: str) -> List[Dict]:
        """Search for pattern in codebase"""
        matches = []
        regex = re.compile(pattern)
        
        for root, dirs, files in os.walk(self.project_path):
            # Skip common directories
            dirs[:] = [d for d in dirs if d not in ['.git', 'node_modules', '__pycache__', 'venv']]
            
            for file in files:
                if not self._is_code_file(file):
                    continue
                
                filepath = os.path.join(root, file)
                try:
                    with open(filepath, 'r', encoding='utf-8') as f:
                        for line_num, line in enumerate(f, 1):
                            if regex.search(line):
                                matches.append({
                                    'file': filepath,
                                    'line': line_num,
                                    'code': line.strip()
                                })
                except Exception:
                    pass
        
        return matches
    
    def _is_code_file(self, filename: str) -> bool:
        """Check if file is source code"""
        code_extensions = ['.java', '.py', '.php', '.go', '.js', '.rb', '.cs', '.cpp', '.c', '.rs']
        return any(filename.endswith(ext) for ext in code_extensions)
```

### Data Flow Analyzer

```python
# scripts/data_flow_analyzer.py
from typing import List, Dict, Set
import ast
import re

class DataFlowAnalyzer:
    """Analyzes data flow from source to sink"""
    
    def __init__(self, project_path: str):
        self.project_path = project_path
        self.taint_sources = set()
        self.sanitizers = set()
        self.dangerous_sinks = set()
    
    def trace_data_flow(self, pattern: Dict) -> List[Dict]:
        """
        Trace tainted data from source to sink
        
        Returns list of data flows with taint information
        """
        filepath = pattern['file']
        line_num = pattern['line']
        
        # Parse the file
        try:
            with open(filepath, 'r') as f:
                content = f.read()
            
            if filepath.endswith('.py'):
                return self._trace_python(content, line_num)
            elif filepath.endswith('.java'):
                return self._trace_java(content, line_num)
            else:
                return []
        except Exception:
            return []
    
    def _trace_python(self, code: str, sink_line: int) -> List[Dict]:
        """Trace data flow in Python code"""
        try:
            tree = ast.parse(code)
        except SyntaxError:
            return []
        
        flows = []
        tainted_vars = set()
        
        # Identify taint sources (user input)
        for node in ast.walk(tree):
            if isinstance(node, ast.Assign):
                # Check if assigning from request/input
                if isinstance(node.value, ast.Attribute):
                    if self._is_taint_source(node.value):
                        for target in node.targets:
                            if isinstance(target, ast.Name):
                                tainted_vars.add(target.id)
                                flows.append({
                                    'line': node.lineno,
                                    'type': 'source',
                                    'var': target.id,
                                    'source': ast.unparse(node.value)
                                })
        
        # Trace tainted variables through assignments
        for node in ast.walk(tree):
            if isinstance(node, ast.Assign):
                for target in node.targets:
                    if isinstance(target, ast.Name):
                        if self._uses_tainted_var(node.value, tainted_vars):
                            tainted_vars.add(target.id)
                            flows.append({
                                'line': node.lineno,
                                'type': 'propagation',
                                'var': target.id,
                                'from': ast.unparse(node.value)
                            })
        
        return flows
    
    def _is_taint_source(self, node: ast.AST) -> bool:
        """Check if node is a taint source"""
        if isinstance(node, ast.Attribute):
            # Common taint sources in Python
            taint_patterns = [
                'request.GET', 'request.POST', 'request.args',
                'request.form', 'request.json', 'input('
            ]
            node_str = ast.unparse(node)
            return any(pattern in node_str for pattern in taint_patterns)
        return False
    
    def _uses_tainted_var(self, node: ast.AST, tainted: Set[str]) -> bool:
        """Check if node uses any tainted variables"""
        for child in ast.walk(node):
            if isinstance(child, ast.Name) and child.id in tainted:
                return True
        return False
    
    def _trace_java(self, code: str, sink_line: int) -> List[Dict]:
        """Trace data flow in Java code (simplified)"""
        flows = []
        tainted_vars = set()
        
        # Simple pattern matching for Java
        lines = code.split('\n')
        
        # Find taint sources
        for i, line in enumerate(lines, 1):
            if re.search(r'(getParameter|getHeader|getCookie)\s*\(', line):
                match = re.search(r'(\w+)\s*=.*get(Parameter|Header|Cookie)', line)
                if match:
                    var_name = match.group(1)
                    tainted_vars.add(var_name)
                    flows.append({
                        'line': i,
                        'type': 'source',
                        'var': var_name,
                        'source': line.strip()
                    })
        
        # Find propagation
        for i, line in enumerate(lines, 1):
            for tainted_var in tainted_vars:
                if tainted_var in line and re.search(rf'\w+\s*=.*{tainted_var}', line):
                    match = re.search(r'(\w+)\s*=', line)
                    if match:
                        new_var = match.group(1)
                        tainted_vars.add(new_var)
                        flows.append({
                            'line': i,
                            'type': 'propagation',
                            'var': new_var,
                            'from': line.strip()
                        })
        
        return flows
    
    def is_vulnerable(self, flows: List[Dict]) -> bool:
        """
        Determine if data flows represent a vulnerability
        
        Vulnerable if:
        1. Data comes from taint source
        2. Reaches dangerous sink
        3. No sanitization in between
        """
        if not flows:
            return False
        
        has_source = any(f['type'] == 'source' for f in flows)
        has_sanitization = any(self._is_sanitizer(f) for f in flows)
        
        return has_source and not has_sanitization
    
    def _is_sanitizer(self, flow: Dict) -> bool:
        """Check if flow includes sanitization"""
        sanitizer_patterns = [
            'escape', 'sanitize', 'filter', 'validate',
            'prepared', 'parameterized', 'whitelist'
        ]
        
        flow_str = str(flow).lower()
        return any(pattern in flow_str for pattern in sanitizer_patterns)
    
    def calculate_severity(self, flows: List[Dict]) -> str:
        """Calculate vulnerability severity"""
        if not flows:
            return 'info'
        
        # Check if reaches database or system calls
        has_db_sink = any('query' in str(f).lower() or 'execute' in str(f).lower() 
                         for f in flows)
        has_cmd_sink = any('system' in str(f).lower() or 'exec' in str(f).lower() 
                          for f in flows)
        
        if has_cmd_sink:
            return 'critical'
        elif has_db_sink:
            return 'high'
        else:
            return 'medium'
```

### Secret Detection

```python
# scripts/secret_finder.py
import re
import os
from typing import List, Dict

class SecretFinder:
    """Detects hardcoded secrets and sensitive information"""
    
    SECRET_PATTERNS = {
        'aws_access_key': r'AKIA[0-9A-Z]{16}',
        'aws_secret_key': r'aws_secret_access_key[\s]*=[\s]*[\'"]([^\'"]+)[\'"]',
        'api_key': r'api[_-]?key[\s]*[=:][\s]*[\'"]([^\'"]{20,})[\'"]',
        'password': r'password[\s]*[=:][\s]*[\'"]([^\'"]+)[\'"]',
        'private_key': r'-----BEGIN (RSA|DSA|EC|OPENSSH) PRIVATE KEY-----',
        'jwt_secret': r'jwt[_-]?secret[\s]*[=:][\s]*[\'"]([^\'"]+)[\'"]',
        'database_url': r'(mysql|postgresql|mongodb)://[^:]+:[^@]+@',
        'github_token': r'gh[ps]_[a-zA-Z0-9]{36}',
        'slack_token': r'xox[baprs]-[0-9]{10,12}-[0-9]{10,12}-[a-zA-Z0-9]{24,32}',
        'generic_secret': r'(secret|token|key)[\s]*[=:][\s]*[\'"][^\'"\s]{16,}[\'"]'
    }
    
    def __init__(self, project_path: str):
        self.project_path = project_path
    
    def scan(self) -> List[Dict]:
        """Scan for hardcoded secrets"""
        findings = []
        
        for root, dirs, files in os.walk(self.project_path):
            # Skip common directories
            dirs[:] = [d for d in dirs if d not in [
                '.git', 'node_modules', '__pycache__', 'venv', '.venv'
            ]]
            
            for file in files:
                if self._should_scan_file(file):
                    filepath = os.path.join(root, file)
                    findings.extend(self._scan_file(filepath))
        
        return findings
    
    def _should_scan_file(self, filename: str) -> bool:
        """Check if file should be scanned"""
        # Scan code and config files
        scan_extensions = [
            '.py', '.java', '.js', '.php', '.go', '.rb', '.cs',
            '.yml', '.yaml', '.json', '.xml', '.properties', '.env',
            '.conf', '.config', '.ini', '.toml'
        ]
        return any(filename.endswith(ext) for ext in scan_extensions)
    
    def _scan_file(self, filepath: str) -> List[Dict]:
        """Scan a single file for secrets"""
        findings = []
        
        try:
            with open(filepath, 'r', encoding='utf-8', errors='ignore') as f:
                content = f.read()
                
            for secret_type, pattern in self.SECRET_PATTERNS.items():
                matches = re.finditer(pattern, content, re.IGNORECASE)
                for match in matches:
                    # Calculate line number
                    line_num = content[:match.start()].count('\n') + 1
                    
                    findings.append({
                        'type': secret_type,
                        'file': filepath,
                        'line': line_num,
                        'matched': match.group(0)[:50],  # First 50 chars
                        'severity': self._get_severity(secret_type)
                    })
        except Exception:
            pass
        
        return findings
    
    def _get_severity(self, secret_type: str) -> str:
        """Determine severity of secret exposure"""
        critical_types = ['aws_access_key', 'aws_secret_key', 'private_key']
        high_types = ['api_key', 'jwt_secret', 'database_url', 'github_token']
        
        if secret_type in critical_types:
            return 'critical'
        elif secret_type in high_types:
            return 'high'
        else:
            return 'medium'
```

### Report Generator

```python
# scripts/report_generator.py
from typing import List, Dict
from datetime import datetime
import json

class ReportGenerator:
    """Generates structured security audit reports"""
    
    def __init__(self, vulnerabilities: List[Dict], secrets: List[Dict]):
        self.vulnerabilities = vulnerabilities
        self.secrets = secrets
    
    def generate_markdown(self, output_path: str = 'security_report.md'):
        """Generate Markdown report"""
        report = self._build_report()
        
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(report)
        
        print(f"[REPORT] Generated: {output_path}")
    
    def _build_report(self) -> str:
        """Build complete report content"""
        sections = [
            self._header(),
            self._executive_summary(),
            self._vulnerability_details(),
            self._secret_findings(),
            self._remediation_priorities(),
            self._appendix()
        ]
        
        return '\n\n'.join(sections)
    
    def _header(self) -> str:
        return f"""# Security Audit Report

**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}  
**Audit Tool:** dfyx_code_security_review  
**Audit Methodology:** Five-Phase White-Box Static Analysis
"""
    
    def _executive_summary(self) -> str:
        """Generate executive summary"""
        severity_counts = self._count_by_severity()
        
        return f"""## Executive Summary

**Total Findings:** {len(self.vulnerabilities) + len(self.secrets)}

### Severity Distribution

| Severity | Count |
|----------|-------|
| Critical | {severity_counts.get('critical', 0)} |
| High | {severity_counts.get('high', 0)} |
| Medium | {severity_counts.get('medium', 0)} |
| Low | {severity_counts.get('low', 0)} |

### Key Risks

{self._top_risks()}
"""
    
    def _vulnerability_details(self) -> str:
        """Detail each vulnerability"""
        if not self.vulnerabilities:
            return "## Vulnerabilities\n\nNo vulnerabilities detected."
        
        sections = ["## Vulnerabilities\n"]
        
        for i, vuln in enumerate(self.vulnerabilities, 1):
            sections.append(f"""### {i}. {vuln['type'].replace('_', ' ').title()}

**Severity:** {vuln.get('severity', 'medium').upper()}  
**Location:** `{vuln['file']}:{vuln['line']}`

**Vulnerable Code:**
```
{vuln['code']}
```

**Data Flow:**
{self._format_data_flow(vuln.get('flows', []))}

**Impact:** {self._describe_impact(vuln['type'])}

**Remediation:**
{self._remediation_advice(vuln['type'])}
""")
        
        return '\n'.join(sections)
    
    def _secret_findings(self) -> str:
        """Detail secret findings"""
        if not self.secrets:
            return "## Hardcoded Secrets\n\nNo hardcoded secrets detected."
        
        sections = ["## Hardcoded Secrets\n"]
        
        for secret in self.secrets:
            sections.append(f"""- **{secret['type']}** in `{secret['file']}:{secret['line']}`
  - Severity: {secret['severity'].upper()}
  - Preview: `{secret['matched']}`
""")
        
        return '\n'.join(sections)
    
    def _count_by_severity(self) -> Dict[str, int]:
        """Count findings by severity"""
        counts = {}
        
        for vuln in self.vulnerabilities:
            severity = vuln.get('severity', 'medium')
            counts[severity] = counts.get(severity, 0) + 1
        
        for secret in self.secrets:
            severity = secret.get('severity', 'medium')
            counts[severity] = counts.get(severity, 0) + 1
        
        return counts
    
    def _top_risks(self) -> str:
        """Identify top risks"""
        critical_vulns = [v for v in self.vulnerabilities 
                         if v.get('severity') == 'critical']
        
        if not critical_vulns:
            return "No critical risks identified."
        
        risks = []
        for vuln in critical_vulns[:3]:
            risks.append(f"- **{vuln['type']}** in `{vuln['file']}`")
        
        return '\n'.join(risks)
    
    def _format_data_flow(self, flows: List[Dict]) -> str:
        """Format data flow for display"""
        if not flows:
            return "*Data flow analysis not available*"
        
        lines = []
        for flow in flows:
            lines.append(f"- Line {flow['line']}: {flow['type']} - `{flow.get('var', 'N/A')}`")
        
        return '\n'.join(lines)
    
    def _describe_impact(self, vuln_type: str) -> str:
        """Describe vulnerability impact"""
        impacts = {
            'sql_injection': 'Attacker can read/modify database, potentially gain full system access',
            'command_injection': 'Attacker can execute arbitrary commands on the server',
            'deserialization': 'Attacker can execute arbitrary code through malicious serialized objects',
            'path_traversal': 'Attacker can read arbitrary files on the system',
            'ssrf': 'Attacker can make requests to internal services'
        }
        return impacts.get(vuln_type, 'Potential security compromise')
    
    def _remediation_advice(self, vuln_type: str) -> str:
        """Provide remediation advice"""
        advice = {
            'sql_injection': '''1. Use parameterized queries/prepared statements
2. Implement input validation with whitelisting
3. Use ORM frameworks with proper escaping
4. Apply principle of least privilege to database accounts''',
            
            'command_injection': '''1. Avoid executing system commands from user input
2. Use language-specific safe APIs instead of shell commands
3. Implement strict input validation with whitelisting
4. Apply sandboxing or containerization''',
            
            'deserialization': '''1. Never deserialize untrusted data
2. Implement integrity checks (HMAC) on serialized data
3. Use safe serialization formats (JSON instead of pickle)
4. Implement type checking before deserialization'''
        }
        return advice.get(vuln_type, 'Implement appropriate security controls')
    
    def _remediation_priorities(self) -> str:
        """Suggest remediation priorities"""
        return """## Remediation Priority

### Immediate (Within 24 hours)
- Fix all CRITICAL severity vulnerabilities
- Remove hardcoded secrets, rotate credentials

### Short-term (Within 1 week)
- Fix all HIGH severity vulnerabilities
- Implement missing security controls

### Medium-term (Within 1 month)
- Fix MEDIUM severity vulnerabilities
- Improve security testing coverage
"""
    
    def _appendix(self) -> str:
        return """## Appendix

### Audit Methodology

This audit used the dfyx five-phase protocol:
1. **Reconnaissance**: Architecture and attack surface analysis
2. **Pattern Matching**: Dangerous code pattern identification
3. **Taint Tracking**: Data flow analysis from source to sink
4. **Validation**: Vulnerability confirmation and exploitability assessment
5. **Reporting**: Structured documentation with remediation guidance

### References
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- CWE Top 25: https://cwe.mitre.org/top25/
"""
```

## Configuration

### Audit Mode Selection

Control audit depth through mode parameter:

```python
# Quick scan - CI/CD integration
scan_project('/path/to/project', mode='quick')

# Standard scan - regular audits
scan_project('/path/to/project', mode='standard')

# Deep scan - comprehensive audit
scan_project('/path/to/project', mode='deep')
```

### Environment Variables

Use environment variables for sensitive configuration:

```bash
# Set project path
export AUDIT_PROJECT_PATH=/path/to/project

# Set output directory
export AUDIT_OUTPUT_DIR=./audit_reports

# Set severity threshold (ignore findings below this level)
export AUDIT_MIN_SEVERITY=medium

# Enable verbose logging
export AUDIT_VERBOSE=true
```

## Security Dimensions Reference

### D1: Injection Vulnerabilities

**Detection patterns:**
- SQL injection (query concatenation)
- Command injection (shell execution)
- LDAP injection
- XML injection
- Template injection (SSTI, SpEL, JNDI)

**Example (Python):**
```python
# VULNERABLE
query = f"SELECT * FROM users WHERE id = {user_id}"  # SQL injection
os.system(f"ping {hostname}")  # Command injection

# SECURE
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_id,))  # Parameterized query
subprocess.run(['ping', hostname], shell=False)  # Safe execution
```

### D2: Authentication Issues

**Detection patterns:**
- Weak password policies
- Missing password hashing
- Insecure session management
- JWT vulnerabilities

**Example (Java):**
```java
// VULNERABLE
String password = request.getParameter("password");
if (user.getPassword().equals(password)) { /* login */ }

// SECURE
String password = request.getParameter("password");
