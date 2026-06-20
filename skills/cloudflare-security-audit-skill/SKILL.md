---
name: cloudflare-security-audit-skill
description: Multi-phase AI security audit orchestration that finds exploitable vulnerabilities through parallel agents, adversarial validation, and machine-readable findings
triggers:
  - run a security audit on this codebase
  - find security vulnerabilities in this project
  - perform a security review with the Cloudflare audit skill
  - do a multi-phase security assessment
  - audit this code for exploitable vulnerabilities
  - run the security-audit skill on this repo
  - pen-test this codebase with structured output
  - analyze this project for security flaws
---

# cloudflare-security-audit-skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

A coding-agent skill that orchestrates multi-phase security audits through parallel agents. It runs six phases: reconnaissance, hunting, adversarial validation, reporting, structured output generation, and independent verification. Designed to find exploitable vulnerabilities with real impact, not theoretical issues.

This is the open-source skill that seeded Cloudflare's vulnerability discovery harness. Multiple runs against the same codebase are additive—each explores different attack surfaces while reading prior findings to avoid duplicates.

## What it does

The skill coordinates multiple parallel agents through a structured pipeline:

1. **Recon** — Maps architecture, trust boundaries, input surfaces → `architecture.md`
2. **Hunt** — 7 parallel agents attack from different angles (injection, access control, business logic, crypto, feature abuse, chained attacks, wildcard)
3. **Validate** — Separate agents adversarially review each finding to eliminate false positives
4. **Report** — Generates `REPORT.md` and `FINDINGS-DETAIL.md`
5. **Structured output** — Produces `findings.json` validated against `report-schema.json`
6. **Independent verification** — Fresh agents verify every claim against actual source code

## Installation

Install using the Skills CLI:

```bash
npx skills add https://github.com/cloudflare/security-audit-skill \
  --skill security-audit
```

For user-level installation (available across all projects):

```bash
npx skills add https://github.com/cloudflare/security-audit-skill \
  --skill security-audit \
  --global
```

Check installation:

```bash
npx skills list
```

## Usage

### Basic audit

Navigate to the codebase you want to audit and invoke:

```
security audit this codebase
```

```
find security vulnerabilities in ./src
```

```
do a security review
```

The skill activates on trigger phrases and prompts for an output directory if not specified.

### Specify output directory

```
security audit this project, output to ~/audits/myapp
```

```
run security-audit skill on ./api, save to /tmp/audit-results
```

Default output location: `~/security-audit-skill/<repo-name>/run-<N>`

### Target specific directories

```
audit the authentication module in ./src/auth
```

```
find vulnerabilities in ./payment-processing
```

### Multi-run strategy

Run multiple audits for better coverage (each explores different paths):

```
security audit this codebase (run 1 of 3)
```

The skill reads prior `findings.json` files to skip known issues and target unexplored areas.

## Core principles

The skill enforces strict security research standards:

### Only report exploitable findings

Every finding requires:
- Concrete attack scenario
- Actual input/payload
- Observable impact
- Not "could theoretically" or "might allow"

**Good**: "CSRF in `/api/delete-account` — no token validation. POST from attacker.com deletes victim's account."

**Bad**: "Endpoint lacks CSRF protection and could be vulnerable."

### Adversarial validation

- The validator agent is never the finder agent
- Validators attempt to disprove findings
- Findings survive only if exploitation path is reproducible

### Severity = Likelihood × Impact

Not "deviates from best practice" but actual risk:

- **CRITICAL**: Full system compromise, data exfiltration, RCE
- **HIGH**: Authentication bypass, privilege escalation, PII leak
- **MEDIUM**: Partial data exposure, limited privilege escalation
- **LOW**: Information disclosure with minimal impact
- **INFO**: Defense-in-depth gaps, hardening opportunities

### Defense-in-depth gaps ≠ vulnerabilities

If Layer A blocks the attack, missing Layer B is a hardening note, not a vulnerability.

Example: If input validation prevents SQL injection, missing prepared statements is INFO at best.

## Output files

After a complete audit run, the output directory contains:

| File | Purpose |
|------|---------|
| `architecture.md` | Recon phase: trust boundaries, data flows, input surfaces |
| `REPORT.md` | Executive summary with confirmed findings |
| `FINDINGS-DETAIL.md` | Detailed traces for MEDIUM+ findings |
| `findings.json` | Machine-readable findings conforming to `report-schema.json` |
| `validation-logs/` | Adversarial review transcripts |

## Schema validation

The skill includes zero-dependency validation:

```javascript
// validate-findings.cjs usage
const { validateFindings } = require('./validate-findings.cjs');
const findings = require('./findings.json');

const result = validateFindings(findings);
if (!result.valid) {
  console.error('Validation errors:', result.errors);
  process.exit(1);
}
```

Run standalone validation:

```bash
node validate-findings.cjs path/to/findings.json
```

## findings.json structure

```json
{
  "audit_metadata": {
    "audit_date": "2026-06-20T10:30:00Z",
    "target_repository": "acme/payments-api",
    "skill_version": "1.0.0",
    "run_number": 1
  },
  "confirmed_findings": [
    {
      "id": "VULN-001",
      "title": "SQL Injection in transaction search",
      "severity": "CRITICAL",
      "category": "Injection",
      "description": "User-supplied 'query' parameter concatenated directly into SQL",
      "location": {
        "file": "src/transactions.js",
        "line_start": 45,
        "line_end": 48,
        "function": "searchTransactions"
      },
      "exploit_scenario": {
        "attack_vector": "GET /api/transactions?query=' OR '1'='1",
        "preconditions": ["Authenticated user"],
        "impact": "Full database read access, including other users' transactions"
      },
      "evidence": {
        "vulnerable_code": "const sql = `SELECT * FROM transactions WHERE description LIKE '%${req.query.query}%'`;",
        "proof_of_concept": "curl -H 'Authorization: Bearer $TOKEN' 'https://api.example.com/api/transactions?query=%27%20OR%20%271%27=%271'"
      },
      "remediation": "Use parameterized queries: `SELECT * FROM transactions WHERE description LIKE ?` with bound parameter",
      "cwe_ids": ["CWE-89"],
      "verification_status": "independently_verified",
      "verification_notes": "Confirmed vulnerable code at src/transactions.js:45-48. No input sanitization present."
    }
  ],
  "rejected_findings": [
    {
      "id": "REJECT-001",
      "title": "Missing rate limiting on login endpoint",
      "reason": "WAF-level rate limiting confirmed in infrastructure layer (cloudflare-config.yaml). Defense-in-depth gap, not exploitable vulnerability.",
      "rejected_by": "validator-agent-3",
      "original_severity": "MEDIUM"
    }
  ],
  "coverage_notes": "Focused on API layer (./src/api). Database layer (./src/db) deferred to next run.",
  "prior_runs_reviewed": ["run-1/findings.json"]
}
```

## Attack classes

The hunting phase spawns parallel agents for:

### Core attack classes

1. **Injection** — SQL, NoSQL, command, template, XSS, XXE
2. **Access Control** — Broken auth, IDOR, privilege escalation, path traversal
3. **Business Logic** — Race conditions, state manipulation, workflow bypass
4. **Cryptography** — Weak crypto, key management, random number generation
5. **Feature Abuse** — Intended features used maliciously
6. **Chained Attacks** — Multi-step exploitation chains

### Wildcard agent

Explores unconventional attack surfaces specific to the codebase's domain.

### Obvious-things sweep

Quick pass for common mistakes (hardcoded secrets, debug endpoints, default credentials).

## Configuration

The skill reads configuration from:

1. Command-line flags (via Skills CLI)
2. Skill metadata in the repository (`.skills/security-audit.json`)
3. Environment variables

### Environment variables

```bash
# Output directory override
export SECURITY_AUDIT_OUTPUT_DIR=~/custom-audits

# Parallel agent count (default: 7 for hunting phase)
export SECURITY_AUDIT_PARALLEL_AGENTS=10

# Skip phases (comma-separated: recon,hunt,validate,report,structured,verify)
export SECURITY_AUDIT_SKIP_PHASES=recon

# Minimum severity to report (CRITICAL, HIGH, MEDIUM, LOW, INFO)
export SECURITY_AUDIT_MIN_SEVERITY=MEDIUM

# Read prior runs from custom location
export SECURITY_AUDIT_PRIOR_RUNS_DIR=~/audits/myapp
```

### Per-project configuration

Create `.skills/security-audit.json` in your repository:

```json
{
  "exclude_paths": [
    "test/**",
    "node_modules/**",
    "vendor/**"
  ],
  "focus_areas": [
    "src/auth",
    "src/payment"
  ],
  "custom_attack_classes": [
    {
      "name": "GraphQL-specific",
      "description": "Query complexity attacks, introspection abuse, batching vulnerabilities"
    }
  ],
  "severity_overrides": {
    "info_to_low": ["Missing security headers on non-sensitive endpoints"]
  }
}
```

## Common patterns

### Pattern: Incremental coverage

Run multiple audits with different focus areas:

```bash
# Run 1: Authentication
security audit ./src/auth, output to ~/audits/myapp/run-1

# Run 2: API layer (reads run-1 findings)
security audit ./src/api, output to ~/audits/myapp/run-2

# Run 3: Business logic (reads run-1 and run-2)
security audit ./src/services, output to ~/audits/myapp/run-3
```

### Pattern: CI/CD integration

```yaml
# .github/workflows/security-audit.yml
name: Security Audit
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly
  workflow_dispatch:

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      
      - name: Install skill
        run: npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
      
      - name: Run audit
        run: |
          # Your coding agent invocation here
          # Example with Claude Code (adjust for your agent)
          echo "security audit this codebase, output to ./audit-results" | your-agent-cli
      
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: security-audit
          path: audit-results/
      
      - name: Check for critical findings
        run: |
          node -e "
            const findings = require('./audit-results/findings.json');
            const critical = findings.confirmed_findings.filter(f => f.severity === 'CRITICAL');
            if (critical.length > 0) {
              console.error(\`Found \${critical.length} CRITICAL findings\`);
              process.exit(1);
            }
          "
```

### Pattern: Custom attack class

Extend hunting with domain-specific attacks:

```javascript
// .skills/custom-attacks/api-gateway-abuse.md

# API Gateway Abuse Attack Class

## Objective
Find vulnerabilities in API gateway routing, transformation, and authentication delegation.

## Attack vectors

1. **Path manipulation**
   - Backend routing confusion via path traversal
   - Host header injection to backend services
   - HTTP method override bypass

2. **Transformation vulnerabilities**
   - JSON-to-XML conversion XXE
   - Query parameter injection via transformation
   - Header smuggling through gateway rewrites

3. **Auth delegation issues**
   - JWT validation in gateway but not backend
   - API key leakage in error messages
   - Rate limit bypass via different endpoints

## Success criteria
- Concrete request/response showing bypass
- Backend behavior different from gateway expectations
- Actual impact (data access, privilege escalation)
```

Then reference in `.skills/security-audit.json`:

```json
{
  "custom_attack_classes": [
    {
      "name": "API Gateway Abuse",
      "file": ".skills/custom-attacks/api-gateway-abuse.md"
    }
  ]
}
```

## Troubleshooting

### Issue: Too many false positives

**Symptom**: Findings include theoretical vulnerabilities without exploitation paths.

**Solution**: The validation phase should catch these. If not:

1. Check validation logs in `validation-logs/` for validator agent reasoning
2. Strengthen validation prompt: "Prove this finding is NOT exploitable"
3. Set `SECURITY_AUDIT_MIN_SEVERITY=MEDIUM` to filter noise

### Issue: Missing obvious vulnerabilities

**Symptom**: Known vulnerabilities not in findings.

**Solution**:

1. Check `coverage_notes` in `findings.json` — may be deferred to another run
2. Review `architecture.md` — recon may have missed the affected component
3. Run additional audits with explicit focus: `audit ./src/vulnerable-module`
4. Check if prior runs marked it as rejected (see `rejected_findings`)

### Issue: Validation phase rejects everything

**Symptom**: All findings end up in `rejected_findings`.

**Solution**:

1. Review `rejection_reason` — often "no concrete exploitation path"
2. Hunters must provide actual payloads, not descriptions
3. Check if defense-in-depth layers are blocking attacks (this is correct rejection)
4. Ensure findings include `proof_of_concept` with real inputs

### Issue: Schema validation fails

**Symptom**: `validate-findings.cjs` reports errors.

**Solution**:

```bash
# Run validator with verbose output
node validate-findings.cjs findings.json --verbose

# Common errors:
# - Missing required fields (id, title, severity, location, exploit_scenario)
# - Invalid severity (must be CRITICAL, HIGH, MEDIUM, LOW, INFO)
# - Malformed location (missing file or line_start)
# - Empty evidence.vulnerable_code
```

Fix by ensuring structured output phase strictly follows `report-schema.json`.

### Issue: Agents go off-task

**Symptom**: Agents suggest refactoring, performance improvements, or non-security issues.

**Solution**: The skill enforces focus via system prompts. If this happens:

1. Check agent logs for prompt drift
2. Re-run with explicit constraint: `security audit focused only on exploitable vulnerabilities`
3. Review `ATTACK-CLASSES.md` definitions — agents should stay within attack class scope

### Issue: Duplicate findings across runs

**Symptom**: Same vulnerability reported in multiple runs.

**Solution**:

1. Ensure `prior_runs_reviewed` in `findings.json` lists previous runs
2. Check `SECURITY_AUDIT_PRIOR_RUNS_DIR` points to correct location
3. Verify finding IDs are stable (based on file + location, not random)
4. Manual deduplication via `id` field in `findings.json`

### Issue: Can't reproduce PoC

**Symptom**: `proof_of_concept` in findings doesn't work when tested.

**Solution**:

1. This is a validation failure — escalate to independent verification phase
2. Check `verification_status` field — should be `independently_verified`
3. If verification passed but PoC fails, file issue with verification logs
4. Ensure PoC includes all preconditions (auth tokens, session state, etc.)

## Advanced usage

### Diff-based auditing

Audit only changed code between commits:

```bash
# Get changed files
git diff --name-only HEAD~5 HEAD > changed-files.txt

# Configure skill to focus on those files
export SECURITY_AUDIT_FOCUS_FILES=$(cat changed-files.txt | tr '\n' ',')

# Run audit
security audit changed code since 5 commits ago
```

### Integration with existing security tools

Combine with SAST/DAST tools:

```javascript
// scripts/merge-findings.js
const skillFindings = require('./audit-results/findings.json');
const sastResults = require('./sast-output.json');

const merged = {
  ...skillFindings,
  confirmed_findings: [
    ...skillFindings.confirmed_findings,
    ...sastResults.issues
      .filter(issue => issue.confidence === 'HIGH')
      .map(issue => ({
        id: `SAST-${issue.id}`,
        title: issue.title,
        severity: mapSeverity(issue.severity),
        category: issue.category,
        location: {
          file: issue.file,
          line_start: issue.line,
          line_end: issue.line
        },
        // ... map other fields
        verification_status: 'tool_reported'
      }))
  ]
};

console.log(JSON.stringify(merged, null, 2));
```

### Custom reporting

Generate reports in different formats:

```javascript
// scripts/generate-html-report.js
const findings = require('./findings.json');
const fs = require('fs');

const html = `
<!DOCTYPE html>
<html>
<head><title>Security Audit Report</title></head>
<body>
  <h1>Security Audit: ${findings.audit_metadata.target_repository}</h1>
  <p>Date: ${findings.audit_metadata.audit_date}</p>
  
  <h2>Critical Findings (${findings.confirmed_findings.filter(f => f.severity === 'CRITICAL').length})</h2>
  ${findings.confirmed_findings
    .filter(f => f.severity === 'CRITICAL')
    .map(f => `
      <div class="finding">
        <h3>${f.id}: ${f.title}</h3>
        <p><strong>Location:</strong> ${f.location.file}:${f.location.line_start}</p>
        <p><strong>Impact:</strong> ${f.exploit_scenario.impact}</p>
        <pre>${f.evidence.vulnerable_code}</pre>
      </div>
    `).join('\n')}
</body>
</html>
`;

fs.writeFileSync('audit-report.html', html);
```

## Requirements

- Coding agent with tool use and parallel sub-agent support (Claude Code, Cursor, Codex, etc.)
- Node.js (for schema validation)
- Git (for multi-run deduplication)

## Related skills

- `sast-integration-skill` — Integrates SAST tool output with adversarial validation
- `threat-modeling-skill` — Architecture-level threat modeling before code audit
- `exploit-development-skill` — Turns findings into working exploits for penetration testing

## License

MIT — see [LICENSE](https://github.com/cloudflare/security-audit-skill/blob/main/LICENSE)
