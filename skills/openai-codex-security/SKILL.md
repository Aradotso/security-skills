---
name: openai-codex-security
description: OpenAI Codex Security CLI and SDK for AI-powered vulnerability scanning, validation, and automated security fixes in codebases
triggers:
  - scan my code for security vulnerabilities
  - run a codex security scan
  - find security issues with openai codex
  - use codex security to check for vulnerabilities
  - analyze my codebase for security flaws
  - run openai security scanner on this project
  - detect and fix security vulnerabilities with codex
  - perform an ai-powered security audit
---

# OpenAI Codex Security

> Skill by [ara.so](https://ara.so) — Security Skills collection

OpenAI Codex Security is an AI-powered CLI and TypeScript SDK that finds, validates, and fixes security vulnerabilities in your codebase. It leverages OpenAI's models to perform intelligent code scanning, vulnerability detection, and automated security remediation.

## Installation

Requires Node.js 22.13.0+ (22.x), 24.x, or 26.x; Python 3.10+; and access to Codex Security.

```bash
npm install @openai/codex-security
```

For global CLI access:

```bash
npm install -g @openai/codex-security
```

## Authentication

### Interactive Login (Local Development)

```bash
npx @openai/codex-security login
```

This stores credentials in Codex's credential home or system keyring for persistent access.

### API Key (CI/CD)

Set environment variables instead of interactive login:

```bash
export OPENAI_API_KEY=your_key_here
# or
export CODEX_API_KEY=your_key_here
```

Environment API keys take precedence in non-interactive environments and are never persisted.

### Credential Selection

If both ChatGPT sign-in and API key are available:

```bash
# Use ChatGPT credentials
npx @openai/codex-security scan . --auth chatgpt

# Use API key
npx @openai/codex-security scan . --auth api-key
```

To make ChatGPT sign-in the default:

```bash
unset OPENAI_API_KEY CODEX_API_KEY
```

## CLI Commands

### Basic Scan

Scan current directory:

```bash
npx @openai/codex-security scan .
```

Scan specific directory:

```bash
npx @openai/codex-security scan /path/to/project
```

### Advanced Scanning Options

Use specific model with higher effort:

```bash
npx @openai/codex-security scan . --model gpt-5.6-terra --effort high
```

Available effort levels:
- `low` - Quick scan, fewer checks
- `medium` - Balanced (default)
- `high` - Thorough scan, maximum coverage

### Scan Output

Scan history is stored in the Codex Security workbench state directory. If the directory is not writable, configure an alternative:

```bash
export CODEX_SECURITY_STATE_DIR=/path/to/writable/dir
npx @openai/codex-security scan .
```

## TypeScript SDK

### Basic Usage

```typescript
import { CodexSecurity } from "@openai/codex-security";

const security = new CodexSecurity();
const result = await security.run(".");

console.log(`Scan complete. Report: ${result.reportPath}`);
console.log(`Vulnerabilities found: ${result.vulnerabilities.length}`);

await security.close();
```

### Scan with Options

```typescript
import { CodexSecurity } from "@openai/codex-security";

const security = new CodexSecurity({
  model: "gpt-5.6-terra",
  effort: "high",
  auth: "api-key"
});

const result = await security.run("/path/to/project");

// Process vulnerabilities
for (const vuln of result.vulnerabilities) {
  console.log(`${vuln.severity}: ${vuln.title}`);
  console.log(`  File: ${vuln.file}:${vuln.line}`);
  console.log(`  Description: ${vuln.description}`);
}

await security.close();
```

### Scan Specific Files

```typescript
import { CodexSecurity } from "@openai/codex-security";

const security = new CodexSecurity();

const result = await security.run(".", {
  files: [
    "src/auth.ts",
    "src/api/users.ts",
    "lib/db.ts"
  ]
});

console.log(`Scanned ${result.filesScanned} files`);
await security.close();
```

### Using Environment Variables for Authentication

```typescript
import { CodexSecurity } from "@openai/codex-security";

// Ensure OPENAI_API_KEY or CODEX_API_KEY is set
if (!process.env.OPENAI_API_KEY && !process.env.CODEX_API_KEY) {
  throw new Error("API key required");
}

const security = new CodexSecurity({
  auth: "api-key"
});

const result = await security.run(".");
await security.close();
```

### Handling Scan Results

```typescript
import { CodexSecurity } from "@openai/codex-security";
import fs from "fs/promises";

const security = new CodexSecurity();
const result = await security.run(".");

// Filter by severity
const critical = result.vulnerabilities.filter(v => v.severity === "critical");
const high = result.vulnerabilities.filter(v => v.severity === "high");

console.log(`Critical: ${critical.length}, High: ${high.length}`);

// Export to JSON
await fs.writeFile(
  "security-report.json",
  JSON.stringify(result, null, 2)
);

// Check if scan passed
if (result.vulnerabilities.length === 0) {
  console.log("✅ No vulnerabilities found");
} else {
  console.error(`❌ Found ${result.vulnerabilities.length} vulnerabilities`);
  process.exit(1);
}

await security.close();
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'
      
      - name: Run Codex Security Scan
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: |
          npx @openai/codex-security scan . --model gpt-5.6-terra
```

### GitLab CI

```yaml
security_scan:
  image: node:22
  before_script:
    - apt-get update && apt-get install -y python3
  script:
    - npx @openai/codex-security scan . --effort high
  variables:
    OPENAI_API_KEY: $OPENAI_API_KEY
  only:
    - main
    - merge_requests
```

### CircleCI

```yaml
version: 2.1

jobs:
  security_scan:
    docker:
      - image: cimg/node:22.13
    steps:
      - checkout
      - run:
          name: Install Python
          command: sudo apt-get update && sudo apt-get install -y python3
      - run:
          name: Run Security Scan
          command: npx @openai/codex-security scan .
          environment:
            OPENAI_API_KEY: ${OPENAI_API_KEY}

workflows:
  version: 2
  scan:
    jobs:
      - security_scan
```

## Common Patterns

### Pre-commit Hook

Create `.husky/pre-commit`:

```bash
#!/bin/sh
npx @openai/codex-security scan . --effort low --auth chatgpt
```

### Integration with Testing

```typescript
import { CodexSecurity } from "@openai/codex-security";
import { describe, it, expect, afterAll } from "@jest/globals";

describe("Security Checks", () => {
  const security = new CodexSecurity();

  afterAll(async () => {
    await security.close();
  });

  it("should have no critical vulnerabilities", async () => {
    const result = await security.run(".");
    const critical = result.vulnerabilities.filter(
      v => v.severity === "critical"
    );
    
    expect(critical).toHaveLength(0);
  }, 300000); // 5 min timeout

  it("should have no high severity SQL injection issues", async () => {
    const result = await security.run(".");
    const sqlInjection = result.vulnerabilities.filter(
      v => v.type.includes("sql-injection") && v.severity === "high"
    );
    
    expect(sqlInjection).toHaveLength(0);
  }, 300000);
});
```

### Automated Fix Workflow

```typescript
import { CodexSecurity } from "@openai/codex-security";
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);
const security = new CodexSecurity({ model: "gpt-5.6-terra" });

async function scanAndFix() {
  console.log("Running security scan...");
  const result = await security.run(".");
  
  if (result.fixableVulnerabilities > 0) {
    console.log(`Found ${result.fixableVulnerabilities} auto-fixable issues`);
    
    // Apply fixes (hypothetical API)
    // await security.fix(result);
    
    // Verify fixes with another scan
    const verifyResult = await security.run(".");
    console.log(`Remaining issues: ${verifyResult.vulnerabilities.length}`);
  }
  
  await security.close();
}

scanAndFix().catch(console.error);
```

## Environment Variables

- `OPENAI_API_KEY` - OpenAI API key for authentication
- `CODEX_API_KEY` - Alternative API key variable
- `CODEX_SECURITY_STATE_DIR` - Custom directory for scan history (default: system-specific state directory)

## Troubleshooting

### Node.js Version Issues

Ensure you're using a compatible Node.js version:

```bash
node --version  # Should be 22.13.0+, 24.x, or 26.x
```

Use `nvm` to switch versions:

```bash
nvm install 22
nvm use 22
```

### Python Not Found

Install Python 3.10 or later:

```bash
# Ubuntu/Debian
sudo apt-get install python3.10

# macOS
brew install python@3.10

# Verify
python3 --version
```

### Credential Issues

If credentials aren't persisting:

```bash
# Check current credentials
npx @openai/codex-security login --status

# Re-login
npx @openai/codex-security logout
npx @openai/codex-security login
```

For CI environments, always use environment variables instead of login.

### State Directory Not Writable

Set a custom state directory:

```bash
export CODEX_SECURITY_STATE_DIR="${HOME}/.codex-security-state"
mkdir -p "${CODEX_SECURITY_STATE_DIR}"
npx @openai/codex-security scan .
```

### Scan Timeout

For large codebases, increase effort level or scan specific directories:

```bash
# Lower effort for faster scans
npx @openai/codex-security scan . --effort low

# Scan specific subdirectories
npx @openai/codex-security scan ./src
```

### API Rate Limits

If you encounter rate limits, consider:

1. Reducing scan frequency
2. Using lower effort levels
3. Scanning only changed files in CI
4. Requesting higher rate limits from OpenAI

### Access Denied

Ensure your account has access to Codex Security. For best results, verify your account for [Trusted Access](https://chatgpt.com/cyber).
