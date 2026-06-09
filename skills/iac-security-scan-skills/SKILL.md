---
name: iac-security-scan-skills
description: AI-powered Infrastructure-as-Code security scanner with attack chain analysis and multi-domain vulnerability detection
triggers:
  - scan infrastructure as code for security issues
  - run IaC security analysis
  - check terraform for vulnerabilities
  - analyze cloudformation security
  - detect infrastructure security risks
  - scan terraform for compliance issues
  - run infrastructure security scan
  - check IaC for attack paths
---

# IaC Security Scanner Skills

> Skill by [ara.so](https://ara.so) — Security Skills collection.

AI-powered security assessment for Infrastructure-as-Code that provides deep, context-aware security analysis of Terraform and CloudFormation projects. Unlike traditional linters, this scanner identifies cross-domain attack chains, privilege escalation paths, and compliance gaps through multi-stage analysis orchestrated entirely by AI agents.

## What It Does

The scanner runs a 3-step pipeline:

1. **Analysis** — Maps repository structure, resources, trust boundaries, and exposure points
2. **Domain Scanning** — Parallel security analysis across IAM, network, storage, secrets, logging, and serverless domains
3. **Attack Chain Correlation** — Identifies composite attack paths across domains and calculates minimum-cut fixes

**Key Features:**
- Cross-domain attack chain detection (internet → workload → IAM → data)
- Minimum-cut analysis (smallest set of fixes that breaks the most attack paths)
- Confidence scores (0.0–1.0) based on IaC evidence vs heuristics
- Multiple scan modes (full, fast, scoped)
- Parallel domain analysis with intelligent file routing
- HTML, CSV, and terminal output formats

## Installation

Copy the scanner skills into your IaC project root:

```bash
# Clone the repository
git clone https://github.com/senaykt/iac-security-scan-skills.git

# Copy skills to your project
cd your-terraform-project
cp -r ../iac-security-scan-skills/{AGENTS.md,CLAUDE.md,.agents,.claude,.cursor} ./

# Add scan output to .gitignore
echo "iac-scan/" >> .gitignore
```

**IDE Compatibility:**
- `.agents/skills/` — Opencode, Codex CLI, Windsurf
- `.claude/skills/` — Claude Code
- `.cursor/skills/` — Cursor

All directories contain identical skills; the orchestration files (`AGENTS.md` or `CLAUDE.md`) route to the correct location.

## Scan Modes

### Full Scan

Runs complete pipeline: analysis → all domain skills → attack chains → report.

```
Run IaC security scan
```

Or target a specific directory:

```
Run IaC security scan on the terraform/ directory
```

**Output:**
- `iac-scan/analysis.md` — Repository intelligence (≤120 lines)
- `iac-scan/{domain}-results.md` — Findings per domain (≤150 lines each)
- `iac-scan/attack-chain-results.md` — Cross-domain attack paths
- `iac-scan/report.html` — Full HTML report
- `iac-scan/findings.csv` — Spreadsheet export
- `iac-scan/architecture-diagram.html` — Visual architecture

### Fast Scan

Single-pass analysis for quick PR checks. High/Critical findings only, no attack chain correlation.

```
Run fast IaC scan
```

**Output:**
- `iac-scan/fast-results.md` — Compact findings
- `iac-scan/fast-report.html` — Lite HTML report

**Use case:** PR reviews, pre-commit hooks, quick developer feedback.

### Scoped Scan

Run only specific security domains:

```
Scan only IAM and network
Scan only storage, secrets, logging
```

**Valid domains:** `IAM`, `network`, `storage`, `secrets`, `logging`, `serverless`

Attack chain analysis runs only if 2+ domains are selected.

### Individual Analyzers

Run a single domain skill:

```
Run IAM analyzer
Run network analyzer
Run storage analyzer
Run secrets analyzer
Run logging analyzer
Run serverless analyzer
```

Step 1 (analysis) runs automatically if `iac-scan/analysis.md` doesn't exist.

### Post-Scan Commands

Re-run specific stages without full rescan:

```
Run attack chain analysis
Generate security report
```

These read existing `iac-scan/*-results.md` files and regenerate derived outputs.

## Security Domains

### IAM (iac-iam)

**Detects:**
- Privilege escalation paths (iam:PassRole abuse, wildcard permissions)
- Dangerous trust relationships (overly permissive assume role policies)
- Cross-account access risks
- Lateral movement chains
- Service role misconfigurations

**Example Finding:**
```
ID: IAM-003
Severity: HIGH
Confidence: 0.95
Title: Lambda execution role allows privilege escalation via iam:PassRole
Description: Role 'data-processor-lambda-role' has iam:PassRole on resource "*" 
combined with lambda:CreateFunction, enabling attackers to escalate to any role.
Attack Path: Compromise Lambda → PassRole to admin role → Full account access
```

### Network (iac-network)

**Detects:**
- Public exposure of internal services
- VPC segmentation gaps
- Security group misconfigurations (0.0.0.0/0 ingress)
- Unrestricted egress (data exfiltration risk)
- Missing VPC endpoints (forced internet routing)

**Example Finding:**
```
ID: NET-007
Severity: CRITICAL
Confidence: 1.0
Title: RDS database directly exposed to internet
Description: Security group 'db-sg' allows 0.0.0.0/0 ingress on port 5432, 
and RDS instance is in public subnet with publicly_accessible = true.
Blast Radius: Customer PII in 'users' and 'transactions' tables
```

### Storage (iac-storage)

**Detects:**
- Unencrypted data stores (S3, RDS, DynamoDB)
- Public S3 buckets
- Missing backup/disaster recovery
- Ransomware readiness gaps (versioning, object lock)
- Dangerous data access patterns

**Example Finding:**
```
ID: STR-012
Severity: HIGH
Confidence: 0.90
Title: S3 bucket lacks versioning and object lock for ransomware protection
Description: Bucket 'customer-backups' stores critical data but has no 
versioning or object lock, enabling ransomware to permanently delete backups.
Recommendation: Enable versioning + object lock (compliance mode, 90d retention)
```

### Secrets (iac-secrets)

**Detects:**
- Hardcoded credentials in IaC files
- Leaked API keys in environment variables
- Terraform state file credential exposure
- CI/CD secret management issues
- Plaintext passwords

**Example Finding:**
```
ID: SEC-002
Severity: CRITICAL
Confidence: 1.0
Title: AWS access key hardcoded in Lambda environment variable
Description: terraform/lambda.tf line 47 contains 'AWS_ACCESS_KEY_ID' with 
literal value 'AKIA...' instead of secret manager reference.
File: terraform/lambda.tf:47
Code: environment = { AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE" }
```

### Logging & Monitoring (iac-logging-monitoring)

**Detects:**
- CloudTrail gaps (missing regions, disabled logging)
- GuardDuty not enabled
- Detection blind spots for exposed/privileged resources
- Forensic readiness issues
- Alerting pipeline gaps

**Example Finding:**
```
ID: LOG-005
Severity: MEDIUM
Confidence: 0.85
Title: No CloudWatch alarms for privilege escalation APIs
Description: IAM roles allow iam:PutUserPolicy and iam:CreateAccessKey but 
no CloudWatch alarm monitors these high-risk API calls.
Recommendation: Create EventBridge rule → SNS alarm for privilege escalation APIs
```

### Serverless (iac-serverless)

**Detects:**
- Unauthenticated Lambda function URLs
- API Gateway authorization misconfigurations
- Event injection vulnerabilities
- Denial-of-wallet risks (unbounded concurrency)
- Overprivileged execution roles

**Example Finding:**
```
ID: SLS-008
Severity: HIGH
Confidence: 0.92
Title: Lambda function URL has no authentication and IAM role allows S3 write
Description: Function 'image-processor' has public URL (auth_type = "NONE") 
and execution role allows s3:PutObject on all buckets.
Attack: Unauthenticated attacker → unlimited invocations → cost spike + data poisoning
```

## Attack Chain Analysis

The `iac-attack-chain` skill reads all domain findings and constructs multi-step attack paths.

**Example Attack Chain:**
```
CHAIN-003: Internet to Admin Access (4 steps, exploitability: 0.88)

Step 1 [NET-007]: Public RDS database (0.0.0.0/0 ingress)
  ↓
Step 2 [SEC-002]: Hardcoded AWS credentials in Lambda environment
  ↓
Step 3 [IAM-003]: Lambda role allows iam:PassRole + lambda:CreateFunction
  ↓
Step 4 [IAM-011]: PassRole to 'AdminAccessRole' → full account compromise

Minimum Cut: Fix NET-007 OR SEC-002 (either breaks the chain)
```

**Minimum-Cut Analysis:**

The scanner identifies the smallest set of fixes that breaks the most attack paths:

```
Priority Fix List (breaks 8 of 12 attack chains):
1. NET-007 (Remove 0.0.0.0/0 from RDS security group) → breaks 5 chains
2. IAM-003 (Scope down iam:PassRole to specific roles) → breaks 3 chains
3. STR-012 (Enable S3 versioning + object lock) → breaks 2 chains
```

## Output Files

All results are written to `iac-scan/`:

```
iac-scan/
├── analysis.md                   # Repository intelligence (Step 1)
├── iam-results.md                # IAM findings
├── network-results.md            # Network findings
├── storage-results.md            # Storage findings
├── secrets-results.md            # Secrets findings
├── logging-monitoring-results.md # Logging findings
├── serverless-results.md         # Serverless findings
├── attack-chain-results.md       # Cross-domain attack chains (Step 2.5)
├── findings.csv                  # CSV export (Step 3)
├── report.html                   # HTML report (Step 3)
└── architecture-diagram.html     # Architecture visualization (Step 3)
```

**Fast scan outputs:**
```
iac-scan/
├── fast-results.md
└── fast-report.html
```

## Supported Project Layouts

The scanner handles all common IaC structures:

| Layout | Example |
|--------|---------|
| Flat files | `main.tf`, `vpc.tf`, `iam.tf` at root |
| Per-environment | `envs/prod/`, `envs/staging/`, `envs/dev/` |
| Per-account | `accounts/prod-app/`, `accounts/security/` |
| Terragrunt | Layered `account.hcl` / `region.hcl` / `env.hcl` |
| Per-service repo | App code + `infra/` or `terraform/` subdirectory |
| CDK app | `cdk.json`, `lib/`, `bin/`, synthesized CloudFormation |
| Serverless Framework | `serverless.yml`, `template.yaml` |
| CloudFormation StackSets | Multi-region / multi-account stacks |
| Atlantis-managed | `atlantis.yaml` with project list |
| Monorepo | Multiple stacks, modules, environments |

## Pipeline Optimizations

**File Routing:**
Step 1 creates a routing table mapping each IaC file to relevant domain skills. Step 2 skills only read files they need (e.g., `iam-analyzer` skips networking configs).

**Compact Artifacts:**
- `analysis.md`: ≤120 lines (no full JSON graph dumps)
- Each `*-results.md`: ≤150 lines
- Individual findings: ≤8 lines each

**Skip Irrelevant Skills:**
`analysis.md` includes "Recommended Skills" section that excludes domains with no resources (e.g., no Lambda → skip serverless).

**Secrets Scope:**
On full scans, `iac-secrets` may check all files. On fast/scoped scans, uses `analysis.md` file list (IaC/CI/CD/env files only).

**Incremental Re-runs:**
If `iac-scan/analysis.md` or domain results already exist, orchestration can skip completed steps and only regenerate HTML/CSV.

## Common Patterns

### PR Review Workflow

```bash
# Developer workflow
git checkout -b feature/add-s3-bucket
# ... make IaC changes ...

# Quick check before commit
# In AI IDE: "Run fast IaC scan"

# Review iac-scan/fast-report.html
# Fix High/Critical findings

git add .
git commit -m "Add S3 bucket with encryption"
```

### Pre-Release Full Scan

```bash
# Before production deployment
# In AI IDE: "Run IaC security scan"

# Review all outputs:
# - iac-scan/report.html (full findings)
# - iac-scan/attack-chain-results.md (composite risks)
# - iac-scan/findings.csv (import to ticketing system)

# Prioritize fixes using minimum-cut analysis
```

### Focused Investigation

```bash
# Security team investigates IAM configuration
# In AI IDE: "Run IAM analyzer"

# Review iac-scan/iam-results.md
# Check privilege escalation paths

# Expand investigation to attack chains
# In AI IDE: "Run attack chain analysis"
```

### Quarterly Compliance Review

```bash
# In AI IDE: "Run IaC security scan"

# Export findings
cp iac-scan/findings.csv /path/to/compliance-reports/2026-Q2-iac-scan.csv

# Review compliance mappings in iac-scan/report.html
# - CIS AWS Foundations Benchmark
# - NIST 800-53
# - PCI DSS
# - SOC 2
```

## Configuration

No configuration files needed. The scanner is controlled entirely through natural language commands.

**Environment-Specific Scans:**

```
Run IaC security scan on the envs/prod/ directory
Scan only IAM and network in accounts/security/
```

**Adjust Scope via Commands:**

```
# Minimum scan (fastest)
Run fast IaC scan

# Medium scan (2-3 domains)
Scan only IAM, storage, secrets

# Full scan (all domains + attack chains)
Run IaC security scan
```

## Troubleshooting

### "No IaC files found"

**Cause:** Scanner didn't detect Terraform/CloudFormation files.

**Solution:**
```
# Explicitly specify directory
Run IaC security scan on the infrastructure/ directory

# Or check file extensions
# Supported: .tf, .hcl, .yaml, .yml, .json (for CloudFormation)
```

### "Analysis step incomplete"

**Cause:** `iac-scan/analysis.md` is missing or truncated.

**Solution:**
```
# Delete partial results and re-run
rm -rf iac-scan/
# In AI IDE: "Run IaC security scan"
```

### "Attack chain analysis skipped"

**Cause:** Only 1 domain was scanned (attack chains require 2+ domains).

**Solution:**
```
# Run additional domain
Run network analyzer

# Then correlation
Run attack chain analysis
```

### "Report shows 'Domain not scanned'"

**Expected behavior** — scoped scans skip irrelevant domains.

**Solution:** If you need all domains:
```
Run IaC security scan
```

### "Secrets scan missed a credential"

**Cause:** Fast scan uses limited file list; scoped scan may skip secrets domain.

**Solution:**
```
# Run full secrets analysis
Run secrets analyzer

# Or full scan
Run IaC security scan
```

### "Too many findings to review"

**Solution:**
```
# Start with fast scan (High/Critical only)
Run fast IaC scan

# Focus on attack chains
Run attack chain analysis

# Use minimum-cut analysis in report.html
# Look for "Priority Fix List" section
```

## Integration Examples

### GitHub Actions

```yaml
# .github/workflows/iac-scan.yml
name: IaC Security Scan
on:
  pull_request:
    paths:
      - 'terraform/**'
      - 'cloudformation/**'

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run fast IaC scan
        # Assumes AI agent CLI available in environment
        run: |
          # AI agent processes AGENTS.md and runs fast scan
          # Output: iac-scan/fast-report.html
          
      - name: Upload scan results
        uses: actions/upload-artifact@v4
        with:
          name: iac-scan-results
          path: iac-scan/
          
      - name: Check for Critical findings
        run: |
          if grep -q "Severity: CRITICAL" iac-scan/fast-results.md; then
            echo "❌ Critical security findings detected"
            exit 1
          fi
```

### GitLab CI

```yaml
# .gitlab-ci.yml
iac-security-scan:
  stage: test
  script:
    - echo "Run IaC security scan" | ai-agent-cli
  artifacts:
    paths:
      - iac-scan/
    reports:
      junit: iac-scan/findings.csv
  only:
    changes:
      - terraform/**/*
      - infrastructure/**/*
```

### Pre-Commit Hook

```bash
# .git/hooks/pre-commit
#!/bin/bash

# Check if IaC files changed
if git diff --cached --name-only | grep -qE '\.(tf|hcl|ya?ml)$'; then
  echo "🔍 Running fast IaC security scan..."
  
  # AI agent runs fast scan
  echo "Run fast IaC scan" | ai-agent-cli
  
  # Check for Critical findings
  if grep -q "CRITICAL" iac-scan/fast-results.md; then
    echo "❌ Critical security issues found. Review iac-scan/fast-report.html"
    exit 1
  fi
  
  echo "✅ No critical issues detected"
fi
```

## Advanced Usage

### Custom Severity Filtering

Filter CSV export for specific severities:

```bash
# Extract only Critical/High findings
awk -F',' '$3 ~ /CRITICAL|HIGH/ {print}' iac-scan/findings.csv > high-priority.csv
```

### Diff Between Scans

Compare findings across deployments:

```bash
# Before changes
cp iac-scan/findings.csv baseline-findings.csv

# After changes
# In AI IDE: "Run IaC security scan"

# Compare
diff baseline-findings.csv iac-scan/findings.csv
```

### Compliance Report Generation

Extract compliance-mapped findings:

```bash
# Filter for PCI DSS findings
grep "PCI DSS" iac-scan/report.html > pci-compliance-report.html

# Or use CSV for custom reports
awk -F',' '$NF ~ /PCI-DSS/ {print $2,$3,$4}' iac-scan/findings.csv
```

## Skill Files Reference

| Skill | Lines | Purpose |
|-------|-------|---------|
| `iac-analysis` | ~715 | Repository intelligence, file routing, resource inventory |
| `iac-iam` | ~630 | IAM policy analysis, privilege escalation |
| `iac-network` | ~640 | Network exposure, segmentation |
| `iac-storage` | ~750 | Data stores, encryption, backup/DR |
| `iac-secrets` | ~285 | Credential detection |
| `iac-logging-monitoring` | ~620 | Detection coverage, forensic readiness |
| `iac-serverless` | ~640 | Serverless attack surface |
| `iac-attack-chain` | ~790 | Cross-domain attack path correlation |
| `iac-report` | ~180 | Deduplication, prioritization, output formatting |
| `iac-fast-scan` | ~175 | All-in-one lite scan (High/Critical only) |

Total: ~5,400 lines for full pipeline, ~175 lines for fast mode.

## License

MIT License — see repository for details.
