---
name: agentic-security-scanner
description: AI-powered security scanner for agentic workflows with plain-English findings, dollar-cost estimates, and auto-fix capabilities
triggers:
  - scan my code for security vulnerabilities
  - check for security issues in this project
  - run agentic security scan
  - find and fix security problems
  - audit this codebase for compliance
  - check dependencies for vulnerabilities
  - scan for hardcoded secrets and SQL injection
  - validate AI-generated code for security
---

# agentic-security

> Skill by [ara.so](https://ara.so) — Security Skills collection

**agentic-security** is a comprehensive security scanner built for AI-driven development workflows. It provides plain-English vulnerability findings with real-world cost estimates, auto-fix capabilities, and compliance framework attestations across 8 languages.

## What It Does

- **12-pillar security scan**: SAST, SCA, secrets, IaC, LLM safety, MCP agent-tool audit, auth/authZ, pipeline integrity, containers, deploy config, supply chain, and trend tracking
- **Plain-English findings**: No CVE jargon—shows stakes, estimated cost, and concrete fixes
- **Auto-fix with preview**: One-command remediation with backup and revert capabilities
- **Compliance reports**: NIST AI 600-1, OWASP ASVS, OWASP LLM Top 10, EU AI Act
- **CISA KEV + EPSS prioritization**: Focus on actively exploited vulnerabilities
- **Function-level reachability**: Know which dependency vulnerabilities actually affect your code
- **Real-time bodyguard**: Intercepts insecure AI-generated code before it hits disk

## Installation

### As Claude Code Plugin

```
/plugin marketplace add https://github.com/Clear-Capabilities/agentic-security
/plugin install agentic-security@clearcapabilities
```

Then restart Claude Code or run `/reload-plugins`.

### As CLI Tool

```bash
npx @clear-capabilities/agentic-security-scanner secure .
```

### Update Plugin

```
/plugin marketplace update clearcapabilities
/plugin install agentic-security@clearcapabilities
```

## Core Commands

### `/agentic-security:secure`

Router command that picks the best next action based on project state.

```
/agentic-security:secure
/agentic-security:secure --tour
/agentic-security:secure --help
/agentic-security:secure --daily
```

### `/agentic-security:scan`

Run security scanner with various modes.

```
# Full scan
/agentic-security:scan

# Scan only changed files
/agentic-security:scan --mode diff

# Watch mode for continuous scanning
/agentic-security:scan --mode watch

# Set baseline for legacy projects
/agentic-security:scan --mode baseline

# Show only new findings since baseline
/agentic-security:scan --since-baseline

# Archaeology mode (deep historical scan)
/agentic-security:scan --mode archaeology
```

### `/agentic-security:triage`

Investigate and validate findings.

```
# Show specific finding
/agentic-security:triage --mode show --id FINDING_ID

# Get detailed explanation
/agentic-security:triage --mode explain --id FINDING_ID

# Validate with PoC attempt
/agentic-security:triage --mode validate --id FINDING_ID

# Red-team simulation
/agentic-security:triage --mode red-team --id FINDING_ID

# Query findings
/agentic-security:triage --mode query "all SQL injection findings in api/"
```

### `/agentic-security:fix`

Remediate vulnerabilities with auto-fix.

```
# Fix specific finding
/agentic-security:fix --id FINDING_ID

# Fix all critical issues
/agentic-security:fix --severity critical

# Fix all issues (with preview)
/agentic-security:fix --all

# Generate PR with fixes
/agentic-security:fix --mode pr --severity critical

# Rotate leaked secret
/agentic-security:fix --mode rotate-secret --id SECRET_FINDING_ID --auto

# Move secrets to vault
/agentic-security:fix --mode vault

# Harden configuration
/agentic-security:fix --mode harden
```

### `/agentic-security:find-and-fix-everything`

One-shot scan and fix for all severities.

```
/agentic-security:find-and-fix-everything
```

## Compliance & Reporting

### Generate Compliance Report

```
# NIST AI 600-1 report
/agentic-security:compliance --report nist

# OWASP ASVS report
/agentic-security:compliance --report asvs

# OWASP LLM Top 10
/agentic-security:compliance --report llm

# EU AI Act
/agentic-security:compliance --report eu-ai-act
```

### Interactive Compliance Walkthrough

```
/agentic-security:compliance --walkthrough nist-ai-600-1
/agentic-security:compliance --walkthrough owasp-asvs-5
/agentic-security:compliance --walkthrough eu-ai-act
```

### Posture & Status

```
# Show security posture
/agentic-security:posture --mode status

# Generate report card
/agentic-security:posture --mode report-card

# Trend analysis
/agentic-security:posture --mode trend

# Threat landscape
/agentic-security:posture --mode threat
```

## Supply Chain Security

```
# Check dependencies
/agentic-security:supply --mode check

# Generate SBOM
/agentic-security:supply --mode sbom

# CVE alerts for dependencies
/agentic-security:supply --mode cve-alerts

# License compliance
/agentic-security:supply --mode license
```

## Setup & Integration

### Install Git Hooks (Bodyguard Mode)

Intercepts insecure AI-generated code in real-time.

```
/agentic-security:setup --mode hooks
```

This blocks:
- SQL injection via concatenation
- Hardcoded API keys and secrets
- `eval()` on user input
- Command injection patterns
- Weak crypto usage

### CI/CD Integration

```
/agentic-security:setup --mode ci
```

### Enable Destructive Operation Guard

```
/agentic-security:setup --mode destructive-guard
```

## Configuration

Create `.agentic-security/config.json` in your project root:

```json
{
  "scan": {
    "exclude": [
      "node_modules/**",
      "dist/**",
      "*.test.js"
    ],
    "severity_threshold": "medium",
    "enable_flow_analysis": true,
    "enable_reachability": true
  },
  "fix": {
    "auto_backup": true,
    "preview_before_apply": true,
    "create_pr": false
  },
  "compliance": {
    "frameworks": ["nist", "asvs", "llm"],
    "custom_controls": ".agentic-security/compliance/custom/controls.json"
  },
  "bodyguard": {
    "enabled": true,
    "block_on_critical": true,
    "allow_override": false
  },
  "notifications": {
    "slack_webhook": "${SLACK_WEBHOOK_URL}",
    "notify_on": ["critical", "high"]
  }
}
```

### Environment Variables

```bash
# Notifications
export SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# Secret rotation (if using auto-rotate)
export STRIPE_API_KEY=sk_live_...
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=...

# GitHub integration (for PR creation)
export GITHUB_TOKEN=ghp_...
```

## Real-World Usage Examples

### Example 1: Initial Project Scan

```javascript
// Run first-time scan on existing project
/agentic-security:scan --mode baseline

// Review critical findings
/agentic-security:triage --mode show --severity critical

// Fix critical SQL injection
/agentic-security:fix --id sql-injection-users-ts-42
```

### Example 2: Fixing Hardcoded Secrets

When scanner finds:

```
[critical] Hardcoded Stripe live key    src/lib/billing.ts:7
  Could enable fraudulent charges against your account.
  Estimated cost if exploited: $50k–$500k
  Fix: rotate via /agentic-security:fix --rotate-secret --auto
```

**Before:**
```typescript
// src/lib/billing.ts
const stripe = new Stripe('sk_live_51HxABC...', {
  apiVersion: '2023-10-16',
});
```

Run fix:
```
/agentic-security:fix --mode rotate-secret --id stripe-key-billing-7 --auto
```

**After:**
```typescript
// src/lib/billing.ts
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2023-10-16',
});
```

The command also:
1. Rotates the key via Stripe API
2. Adds new key to `.env.example` as placeholder
3. Updates CI/CD secrets if configured

### Example 3: SQL Injection Fix

When scanner finds:

```
[critical] SQL Injection    api/users.ts:42
  Could leak PII for ~5,000 users.
  Estimated cost if exploited: $125k–$1.3M
  Fix: use parameterized query
```

**Before:**
```typescript
// api/users.ts
export async function getUser(req: Request, res: Response) {
  const userId = req.query.id;
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  const result = await db.query(query);
  res.json(result.rows[0]);
}
```

Run fix:
```
/agentic-security:fix --id sql-injection-users-ts-42
```

**After:**
```typescript
// api/users.ts
export async function getUser(req: Request, res: Response) {
  const userId = req.query.id;
  const query = 'SELECT * FROM users WHERE id = ?';
  const result = await db.query(query, [userId]);
  res.json(result.rows[0]);
}
```

### Example 4: Webhook Signature Validation

**Before:**
```typescript
// api/stripe-webhook.ts
export async function handleWebhook(req: Request, res: Response) {
  const event = req.body;
  
  if (event.type === 'payment_intent.succeeded') {
    await unlockPaidFeatures(event.data.object.customer);
  }
  
  res.json({ received: true });
}
```

Run fix:
```
/agentic-security:fix --id webhook-signature-missing-12
```

**After:**
```typescript
// api/stripe-webhook.ts
export async function handleWebhook(req: Request, res: Response) {
  const sig = req.headers['stripe-signature'];
  const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET;
  
  let event;
  try {
    event = stripe.webhooks.constructEvent(
      req.rawBody,
      sig,
      endpointSecret
    );
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }
  
  if (event.type === 'payment_intent.succeeded') {
    await unlockPaidFeatures(event.data.object.customer);
  }
  
  res.json({ received: true });
}
```

### Example 5: Pre-Deployment Check in CI

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run agentic-security scan
        run: |
          npx @clear-capabilities/agentic-security-scanner scan --mode diff --since-baseline
          
      - name: Block on critical findings
        run: |
          npx @clear-capabilities/agentic-security-scanner posture --mode status --fail-on critical
```

### Example 6: Daily Security Review

```bash
#!/bin/bash
# daily-security-check.sh

# Run daily scan
npx @clear-capabilities/agentic-security-scanner secure --daily

# Generate report card
npx @clear-capabilities/agentic-security-scanner posture --mode report-card > /tmp/security-report.txt

# Send to Slack
curl -X POST $SLACK_WEBHOOK_URL \
  -H 'Content-Type: application/json' \
  -d "{\"text\": \"$(cat /tmp/security-report.txt)\"}"
```

### Example 7: Dependency Vulnerability Check

```
# Check all dependencies for CVEs
/agentic-security:supply --mode check

# Show only function-reachable vulnerabilities
/agentic-security:supply --mode check --reachability function-reachable

# Generate SBOM for compliance
/agentic-security:supply --mode sbom --format spdx
```

## Common Patterns

### Pattern 1: Legacy Codebase Onboarding

```
# Step 1: Set baseline (don't fail on existing issues)
/agentic-security:scan --mode baseline

# Step 2: Review critical count
/agentic-security:posture --mode status

# Step 3: Fix only new critical issues going forward
/agentic-security:scan --since-baseline --severity critical

# Step 4: Gradually reduce baseline
/agentic-security:fix --severity critical --all
/agentic-security:scan --mode baseline --update
```

### Pattern 2: Pre-Commit Review

```
# Scan only staged changes
/agentic-security:scan --mode diff --staged

# Auto-fix safe issues
/agentic-security:fix --auto-safe

# Block commit if critical found
/agentic-security:posture --mode status --fail-on critical
```

### Pattern 3: Compliance Audit Prep

```
# Generate all compliance reports
/agentic-security:compliance --report nist
/agentic-security:compliance --report asvs
/agentic-security:compliance --report llm

# Create attestation package
/agentic-security:compliance --mode attestation --output ./audit/

# Generate evidence walkthrough
/agentic-security:compliance --walkthrough nist-ai-600-1 --output ./audit/nist-walkthrough.md
```

## Troubleshooting

### High False Positive Rate

```
# Enable flow analysis for better precision
/agentic-security:scan --enable-flow-analysis

# Use validation mode to confirm findings
/agentic-security:triage --mode validate --id FINDING_ID

# Suppress specific false positives
# Add to .agentic-security/suppressions.json:
{
  "suppressions": [
    {
      "id": "sql-injection-safe-query-42",
      "reason": "Uses ORM with parameterization",
      "expires": "2026-12-31"
    }
  ]
}
```

### Scanner Missing Vulnerabilities

```
# Enable archaeology mode for deep scan
/agentic-security:scan --mode archaeology

# Run AI-assisted rescan
/agentic-security:labs --mode model-rescan --severity critical

# Submit custom detection rule
/agentic-security:labs --mode synthesize-rule --pattern "your custom pattern"
```

### Fix Applied Incorrectly

```
# Revert last fix
/agentic-security:fix --revert --id FINDING_ID

# All fixes create backups in .agentic-security/backups/
# Manually restore:
cp .agentic-security/backups/src/lib/billing.ts.backup src/lib/billing.ts
```

### Performance Issues on Large Repos

```
# Exclude build artifacts
# In .agentic-security/config.json:
{
  "scan": {
    "exclude": [
      "node_modules/**",
      "dist/**",
      "build/**",
      ".next/**",
      "*.min.js"
    ]
  }
}

# Scan only changed files
/agentic-security:scan --mode diff

# Disable reachability analysis for faster scans
/agentic-security:scan --disable-reachability
```

### Bodyguard Blocking Valid Code

```
# Disable bodyguard temporarily
/agentic-security:setup --mode bodyguard --disable

# Or allow override in config:
{
  "bodyguard": {
    "enabled": true,
    "allow_override": true
  }
}

# Then override specific block:
git commit --no-verify
```

## Language-Specific Examples

### JavaScript/TypeScript

```typescript
// Insecure: Command injection
import { exec } from 'child_process';
exec(`git clone ${userInput}`); // ❌ Detected

// Secure: Sanitized input
import { exec } from 'child_process';
import { execFile } from 'child_process';
execFile('git', ['clone', userInput]); // ✅ Safe
```

### Python

```python
# Insecure: SQL injection
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")  # ❌ Detected

# Secure: Parameterized query
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))  # ✅ Safe
```

### Java

```java
// Insecure: LDAP injection
String filter = "(uid=" + username + ")";
ctx.search("ou=users", filter, controls);  // ❌ Detected

// Secure: Escaped input
String filter = "(uid=" + escapeForLDAP(username) + ")";
ctx.search("ou=users", filter, controls);  // ✅ Safe
```

## Advanced Features

### Custom Compliance Controls

```json
// .agentic-security/compliance/custom/controls.json
{
  "framework": "internal-security-policy",
  "version": "2.1",
  "controls": [
    {
      "id": "ISP-001",
      "title": "All API endpoints require authentication",
      "check": "auth-required-on-routes",
      "severity": "critical"
    }
  ]
}
```

Then run:
```
/agentic-security:compliance --report custom --framework internal-security-policy
```

### Cross-Repository Analysis

```
# Scan multiple repos for shared vulnerabilities
/agentic-security:labs --mode cross-repo --repos org/repo1,org/repo2,org/repo3
```

### Risk Dollar Estimation

```
# Show findings with cost-benefit analysis
/agentic-security:labs --mode risk-dollars --severity high
```

## Integration with Other Tools

### SARIF Export (for GitHub Code Scanning)

```bash
npx @clear-capabilities/agentic-security-scanner scan --output sarif > results.sarif

# Upload to GitHub
gh api /repos/OWNER/REPO/code-scanning/sarifs \
  -F sarif=@results.sarif \
  -F commit_sha=$GITHUB_SHA \
  -F ref=$GITHUB_REF
```

### DefectDojo Integration

```bash
# Generate SARIF with codeFlows
npx @clear-capabilities/agentic-security-scanner scan \
  --output sarif \
  --enable-code-flows > agentic-security.sarif

# Import to DefectDojo API
curl -X POST https://defectdojo.example.com/api/v2/import-scan/ \
  -H "Authorization: Token ${DEFECTDOJO_TOKEN}" \
  -F "scan_type=SARIF" \
  -F "file=@agentic-security.sarif" \
  -F "engagement=1"
```

---

**License**: PolyForm Internal Use (use to secure your own code, not for resale)

**Support**: Open GitHub issue at [Clear-Capabilities/agentic-security](https://github.com/Clear-Capabilities/agentic-security)
