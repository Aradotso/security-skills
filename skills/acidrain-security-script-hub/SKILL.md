---
name: acidrain-security-script-hub
description: Web-oriented XSS analysis resources, JavaScript utilities, and PHP examples for authorized security testing and hands-on learning
triggers:
  - how do I use acidrain for xss testing
  - set up acidrain security scripts
  - show me acidrain xss examples
  - test cross-site scripting with acidrain
  - acidrain javascript injection samples
  - configure acidrain php security tests
  - run acidrain web security analysis
  - acidrain authorized penetration testing
---

# AcidRain Security Script Hub Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## What AcidRain Does

AcidRain is a web security toolbox containing XSS injection resources, JavaScript utilities, and PHP examples for authorized security testing, education, and research. It provides browser-side and server-side material organized for experimentation in controlled environments.

**Key capabilities:**
- XSS analysis and cross-site scripting testing resources
- Client-side JavaScript security utilities
- Server-side PHP research examples
- Injection testing samples for input/output validation
- Web security helper scripts for targeted investigation

**⚠️ CRITICAL:** Use only against systems you own or have explicit written authorization to test. Unauthorized use is illegal.

## Installation

Clone the repository and navigate to the working directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

Inspect the directory structure before running any scripts:

```bash
ls -la
tree scripts/  # if available
```

Expected structure:
```
acidrain-security-script-hub/
├── scripts/
│   ├── javascript/
│   ├── php/
│   └── xss/
├── configs/
├── examples/
├── docs/
├── LICENSE
└── README.md
```

## JavaScript Utilities

### Basic XSS Payload Testing

Create a test environment HTML file:

```html
<!DOCTYPE html>
<html>
<head>
    <title>XSS Test Environment</title>
</head>
<body>
    <h1>Authorized Testing Zone</h1>
    <div id="output"></div>
    
    <script>
        // Safe parameter extraction for testing
        function getTestParameter(name) {
            const urlParams = new URLSearchParams(window.location.search);
            return urlParams.get(name);
        }
        
        // Demonstrate unsafe output (for testing only)
        function displayUnsafeOutput(input) {
            document.getElementById('output').innerHTML = input;
        }
        
        // Test XSS vulnerability
        const userInput = getTestParameter('test');
        if (userInput) {
            displayUnsafeOutput(userInput);
        }
    </script>
</body>
</html>
```

### XSS Detection Script

```javascript
// scripts/javascript/xss-detector.js

class XSSDetector {
    constructor() {
        this.payloads = [
            '<script>alert(1)</script>',
            '<img src=x onerror=alert(1)>',
            '<svg onload=alert(1)>',
            'javascript:alert(1)',
            '<iframe src="javascript:alert(1)">',
            '<body onload=alert(1)>',
            '<input autofocus onfocus=alert(1)>',
            '"><script>alert(1)</script>'
        ];
    }
    
    testEndpoint(url, params) {
        console.log(`Testing: ${url}`);
        
        this.payloads.forEach((payload, index) => {
            const testParams = { ...params, test: payload };
            const queryString = new URLSearchParams(testParams).toString();
            const testUrl = `${url}?${queryString}`;
            
            console.log(`[${index + 1}] Payload: ${payload}`);
            console.log(`    URL: ${testUrl}`);
        });
    }
    
    sanitizeInput(input) {
        const element = document.createElement('div');
        element.textContent = input;
        return element.innerHTML;
    }
    
    validateOutput(html) {
        const dangerous = /<script|javascript:|onerror=|onload=/i;
        return !dangerous.test(html);
    }
}

// Usage
const detector = new XSSDetector();
detector.testEndpoint('http://localhost:8000/test.php', { user: 'testuser' });

// Sanitization example
const userInput = '<script>alert("xss")</script>';
const safe = detector.sanitizeInput(userInput);
console.log('Sanitized:', safe);
```

### DOM-Based XSS Analysis

```javascript
// scripts/javascript/dom-xss-analyzer.js

function analyzeDOMSinks() {
    const dangerousSinks = {
        'innerHTML': document.querySelectorAll('[id]'),
        'eval': 'eval() calls',
        'setTimeout': 'setTimeout with string',
        'setInterval': 'setInterval with string',
        'document.write': 'document.write() calls',
        'location': 'location manipulation'
    };
    
    console.log('=== DOM XSS Sink Analysis ===');
    
    // Check for innerHTML usage
    document.querySelectorAll('*').forEach(el => {
        const id = el.id;
        if (id) {
            console.log(`Found element with ID: ${id}`);
            console.log(`  - Can be targeted via innerHTML`);
        }
    });
    
    // Monitor location-based sources
    const sources = {
        'location.href': window.location.href,
        'location.search': window.location.search,
        'location.hash': window.location.hash,
        'document.referrer': document.referrer
    };
    
    console.log('\n=== User-Controlled Sources ===');
    Object.entries(sources).forEach(([name, value]) => {
        if (value) {
            console.log(`${name}: ${value}`);
        }
    });
}

// Execute analysis
analyzeDOMSinks();
```

## PHP Security Testing

### Input Validation Testing

```php
<?php
// scripts/php/input-validator.php

class InputValidator {
    
    public static function testInputs() {
        $testCases = [
            '<script>alert(1)</script>',
            '"><img src=x onerror=alert(1)>',
            "'; DROP TABLE users; --",
            '../../../etc/passwd',
            '${7*7}',
            '{{7*7}}'
        ];
        
        echo "=== Input Validation Tests ===\n\n";
        
        foreach ($testCases as $input) {
            echo "Testing: " . $input . "\n";
            echo "Sanitized: " . self::sanitize($input) . "\n";
            echo "Escaped: " . self::escape($input) . "\n\n";
        }
    }
    
    public static function sanitize($input) {
        return htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
    }
    
    public static function escape($input) {
        return addslashes($input);
    }
    
    public static function validateEmail($email) {
        return filter_var($email, FILTER_VALIDATE_EMAIL);
    }
    
    public static function validateURL($url) {
        return filter_var($url, FILTER_VALIDATE_URL);
    }
}

// Run tests
InputValidator::testInputs();
?>
```

### XSS Vulnerable Endpoint (Testing Only)

```php
<?php
// scripts/php/vulnerable-endpoint.php
// WARNING: This is intentionally vulnerable for testing

header('X-Frame-Options: DENY');
header('X-Content-Type-Options: nosniff');

$test_mode = getenv('ACIDRAIN_TEST_MODE');
if ($test_mode !== 'enabled') {
    die('Test mode must be explicitly enabled via ACIDRAIN_TEST_MODE=enabled');
}

// Intentionally vulnerable for testing
function displayUnsafe($input) {
    return "<div>User input: " . $input . "</div>";
}

// Safe version for comparison
function displaySafe($input) {
    return "<div>User input: " . htmlspecialchars($input, ENT_QUOTES, 'UTF-8') . "</div>";
}

if (isset($_GET['unsafe'])) {
    echo displayUnsafe($_GET['unsafe']);
}

if (isset($_GET['safe'])) {
    echo displaySafe($_GET['safe']);
}
?>
```

### Secure Form Handler

```php
<?php
// scripts/php/secure-form-handler.php

class SecureFormHandler {
    
    private $allowedFields = ['name', 'email', 'message'];
    private $errors = [];
    
    public function processForm($data) {
        $sanitized = [];
        
        foreach ($this->allowedFields as $field) {
            if (!isset($data[$field])) {
                $this->errors[] = "Missing field: $field";
                continue;
            }
            
            $value = $data[$field];
            
            // Sanitize
            $value = trim($value);
            $value = stripslashes($value);
            $value = htmlspecialchars($value, ENT_QUOTES, 'UTF-8');
            
            // Validate
            if ($field === 'email' && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
                $this->errors[] = "Invalid email format";
                continue;
            }
            
            $sanitized[$field] = $value;
        }
        
        if (empty($this->errors)) {
            return ['success' => true, 'data' => $sanitized];
        }
        
        return ['success' => false, 'errors' => $this->errors];
    }
    
    public function getErrors() {
        return $this->errors;
    }
}

// Usage
$handler = new SecureFormHandler();
$result = $handler->processForm($_POST);

if ($result['success']) {
    // Process safe data
    $data = $result['data'];
    echo "Form processed successfully\n";
} else {
    echo "Errors:\n";
    foreach ($result['errors'] as $error) {
        echo "- $error\n";
    }
}
?>
```

## Configuration

### Environment Setup

Create a `.env` file in the project root:

```bash
# .env
ACIDRAIN_TEST_MODE=enabled
ACIDRAIN_TARGET_URL=http://localhost:8000
ACIDRAIN_LOG_LEVEL=debug
ACIDRAIN_REPORT_DIR=./reports
```

Load environment variables in PHP:

```php
<?php
// Load configuration
$dotenv = parse_ini_file('.env');
foreach ($dotenv as $key => $value) {
    putenv("$key=$value");
}

$testMode = getenv('ACIDRAIN_TEST_MODE');
$targetUrl = getenv('ACIDRAIN_TARGET_URL');
?>
```

### Test Server Setup

Start a PHP development server for testing:

```bash
# Navigate to scripts directory
cd scripts/php

# Start server
php -S localhost:8000

# Test endpoint
curl "http://localhost:8000/vulnerable-endpoint.php?safe=<script>test</script>"
```

## Common Testing Patterns

### XSS Reflection Testing

```javascript
// Test for reflected XSS
function testReflectedXSS(baseUrl, params) {
    const payloads = [
        '<script>alert(document.domain)</script>',
        '<img src=x onerror=alert(1)>',
        '"><svg/onload=alert(1)>',
        'javascript:alert(1)'
    ];
    
    payloads.forEach(payload => {
        const testParams = { ...params, input: payload };
        const url = `${baseUrl}?${new URLSearchParams(testParams)}`;
        
        console.log(`Testing URL: ${url}`);
        
        // In practice, use fetch or automated browser
        fetch(url)
            .then(response => response.text())
            .then(html => {
                if (html.includes(payload)) {
                    console.warn('⚠️ Potential XSS: Payload reflected');
                } else {
                    console.log('✓ Payload sanitized or filtered');
                }
            });
    });
}

// Usage
testReflectedXSS('http://localhost:8000/test.php', { user: 'test' });
```

### Stored XSS Testing

```php
<?php
// Test stored XSS vulnerability
function testStoredXSS($pdo, $payload) {
    // Insert test payload
    $stmt = $pdo->prepare("INSERT INTO comments (content) VALUES (?)");
    $stmt->execute([$payload]);
    
    // Retrieve and display (vulnerable)
    $stmt = $pdo->query("SELECT content FROM comments ORDER BY id DESC LIMIT 1");
    $comment = $stmt->fetch(PDO::FETCH_ASSOC);
    
    echo "Vulnerable output: " . $comment['content'] . "\n";
    echo "Safe output: " . htmlspecialchars($comment['content'], ENT_QUOTES, 'UTF-8') . "\n";
}

// Database setup for testing
$pdo = new PDO(
    'sqlite::memory:',
    null,
    null,
    [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
);

$pdo->exec("CREATE TABLE comments (id INTEGER PRIMARY KEY, content TEXT)");

// Test
testStoredXSS($pdo, '<script>alert("stored")</script>');
?>
```

### Content Security Policy Testing

```javascript
// Check CSP headers
async function checkCSP(url) {
    try {
        const response = await fetch(url);
        const csp = response.headers.get('Content-Security-Policy');
        
        if (csp) {
            console.log('CSP Header found:', csp);
            
            // Parse directives
            const directives = csp.split(';').map(d => d.trim());
            directives.forEach(directive => {
                console.log(`  - ${directive}`);
            });
            
            // Check for unsafe directives
            if (csp.includes("'unsafe-inline'")) {
                console.warn('⚠️ CSP allows unsafe-inline');
            }
            if (csp.includes("'unsafe-eval'")) {
                console.warn('⚠️ CSP allows unsafe-eval');
            }
        } else {
            console.warn('⚠️ No CSP header found');
        }
    } catch (error) {
        console.error('Error checking CSP:', error);
    }
}

checkCSP('http://localhost:8000');
```

## Reporting and Logging

### Generate Test Report

```php
<?php
// scripts/php/report-generator.php

class TestReportGenerator {
    private $results = [];
    private $reportDir;
    
    public function __construct() {
        $this->reportDir = getenv('ACIDRAIN_REPORT_DIR') ?: './reports';
        if (!is_dir($this->reportDir)) {
            mkdir($this->reportDir, 0755, true);
        }
    }
    
    public function addResult($testName, $status, $details) {
        $this->results[] = [
            'timestamp' => date('Y-m-d H:i:s'),
            'test' => $testName,
            'status' => $status,
            'details' => $details
        ];
    }
    
    public function generateReport() {
        $filename = $this->reportDir . '/report_' . date('Ymd_His') . '.json';
        
        $report = [
            'generated' => date('c'),
            'total_tests' => count($this->results),
            'results' => $this->results
        ];
        
        file_put_contents($filename, json_encode($report, JSON_PRETTY_PRINT));
        
        echo "Report generated: $filename\n";
        return $filename;
    }
}

// Usage
$reporter = new TestReportGenerator();
$reporter->addResult('XSS Test 1', 'PASS', 'Input properly sanitized');
$reporter->addResult('XSS Test 2', 'FAIL', 'Payload reflected in response');
$reporter->generateReport();
?>
```

## Troubleshooting

### Scripts Not Executing

**Issue:** PHP scripts return 403 or permission errors

**Solution:**
```bash
# Fix file permissions
chmod +x scripts/php/*.php
chmod 755 scripts/

# Check PHP is installed
php --version

# Verify script syntax
php -l scripts/php/your-script.php
```

### CORS Issues with JavaScript

**Issue:** Browser blocks fetch requests during testing

**Solution:**
```javascript
// Add CORS headers in PHP test endpoints
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Methods: GET, POST, OPTIONS');
header('Access-Control-Allow-Headers: Content-Type');

// Or use a proxy for testing
const proxyUrl = 'http://localhost:8080/';
fetch(proxyUrl + targetUrl);
```

### Test Mode Not Enabled

**Issue:** Scripts refuse to run vulnerable code

**Solution:**
```bash
# Enable test mode
export ACIDRAIN_TEST_MODE=enabled

# Or in .env
echo "ACIDRAIN_TEST_MODE=enabled" >> .env

# Verify
php -r "echo getenv('ACIDRAIN_TEST_MODE');"
```

### False Positives in Detection

**Issue:** XSS detector reports issues on properly sanitized code

**Solution:**
```javascript
// Improve detection logic
function validateSanitization(input, output) {
    const encoded = input
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#x27;');
    
    return output === encoded;
}
```

### Database Connection Errors

**Issue:** Stored XSS tests fail due to database connection

**Solution:**
```php
<?php
// Use SQLite for isolated testing
$pdo = new PDO('sqlite::memory:');

// Or configure MySQL/PostgreSQL
$host = getenv('DB_HOST') ?: 'localhost';
$db = getenv('DB_NAME') ?: 'acidrain_test';
$user = getenv('DB_USER') ?: 'root';
$pass = getenv('DB_PASS') ?: '';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
} catch (PDOException $e) {
    die("Database error: " . $e->getMessage());
}
?>
```

## Safety Reminders

1. **Authorization Required:** Only test systems you own or have written permission to test
2. **Isolated Environment:** Use local VMs, containers, or dedicated test infrastructure
3. **No Production Data:** Never test against production systems or real user data
4. **Document Findings:** Keep detailed records of authorized testing activities
5. **Responsible Disclosure:** Report discovered vulnerabilities through proper channels

## Best Practices

- Always sanitize user input with `htmlspecialchars()` or equivalent
- Implement Content Security Policy headers
- Use prepared statements for database queries
- Validate and whitelist input rather than blacklisting patterns
- Log all testing activities for audit purposes
- Test in isolated environments separate from development/production
- Keep AcidRain updated by pulling latest changes regularly
