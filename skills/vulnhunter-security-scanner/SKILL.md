---
name: vulnhunter-security-scanner
description: Agentic AI security tool that applies attacker-first analysis to source code, using falsification-based reasoning to minimize false positives
triggers:
  - scan this code for security vulnerabilities
  - run vulnhunter to find exploitable bugs
  - perform attacker-first security analysis
  - hunt for security vulnerabilities in this codebase
  - analyze this code with vulnhunter
  - check for exploitable security issues
  - run security scanning with falsification engine
  - find real vulnerabilities not just patterns
---

# VulnHunter Security Scanner

> Skill by [ara.so](https://ara.so) — Security Skills collection.

VulnHunter is an agentic AI security tool that applies proactive, attacker-first analysis directly to source code. Unlike traditional SAST scanners that flag patterns and generate false positives, VulnHunter reasons like an adversary: it identifies which defects are actually exploitable, maps attack paths, and proposes evidence-backed fixes.

## Core Concepts

**Attacker-First Forward Analysis**: Starts at attacker-accessible entry points (APIs, file uploads, network messages) and reasons forward to evaluate whether an attacker can truly break through, rather than working backward from dangerous sinks.

**Falsification Engine**: After finding a potential vulnerability, VulnHunter runs a structured workflow to *disprove* its own argument, searching for flawed assumptions, logic gaps, or security controls that would block the attack.

**Three-Skill Loop**:
- `/vulnhunt` - Hunt for vulnerabilities
- `/vulnhunter-fix` - Fix verified issues with test-driven remediation
- `/vulnhunt-fix-verify` - Independently verify fixes

## Prerequisites

- **Claude Code CLI** authenticated with access to **Claude Opus** (required)
- Python 3.12+ (for agent runtime and harness tooling)
- Git and GitHub CLI (`gh`) authenticated (for fix workflow)
- **Anthropic Cyber Verification Program enrollment** recommended to avoid cyber safeguard blocks

## Installation

```bash
# Clone the repository
git clone https://github.com/capitalone/vulnhunter.git
cd vulnhunter

# Install skills to ~/.claude/skills/
./install.sh

# Optional: Uninstall
# ./uninstall.sh
```

**Note**: `install.sh` copies files (not symlinks) because symlinks can break subagent functionality. Re-run after pulling updates.

## Core Usage

### 1. Running the Scanner (`/vulnhunt`)

```bash
# Launch Claude Code with the vulnhunt skill
claude --model opus \
  --add-dir ~/.claude/skills/vulnhunt \
  --add-dir ~/.claude/skills/vulnhunt/phases

# Inside Claude Code session:
/vulnhunt
```

The scanner will:
1. **Recon Phase**: Map entry points and dangerous sinks
2. **Parallel Hunt**: Trace paths from entries to sinks
3. **Adversarial Disprove**: Apply falsification to eliminate false positives
4. **Capability Filter**: Verify actual exploitability
5. **Output**: Emit only verified issues with exploit paths and proposed fixes

### 2. Running the Fixer (`/vulnhunter-fix`)

First install Python dependencies:

```bash
cd vulnhunter-fix
pip install -e ".[dev]"
```

Launch with the fix skill:

```bash
claude --model opus --add-dir ~/.claude/skills/vulnhunter-fix

# Inside Claude Code session:
/vulnhunter-fix
```

The fixer implements **RED-GREEN-REFACTOR** workflow:
1. Write exploit demonstration
2. Create failing security test (RED)
3. Implement code fix (GREEN)
4. Verify exploit blocked without regressions
5. Cut reviewable PR

### 3. Running the Verifier (`/vulnhunt-fix-verify`)

The verifier runs read-only validation on fixes. Pre-create the output directory:

```bash
mkdir -p /tmp/verify-output

claude --model opus \
  --add-dir ~/.claude/skills/vulnhunt-fix-verify \
  --add-dir ~/.claude/skills/vulnhunt-fix-verify/phases

# Inside Claude Code session:
/vulnhunt-fix-verify \
  repo=/absolute/path/to/repo \
  report=/absolute/path/to/vulnhunt_report.md \
  fixed=VULN-001,VULN-003 \
  out=/tmp/verify-output \
  comments=/absolute/path/to/pr_comments.json \
  additional_repos=/path/to/lib1,/path/to/lib2
```

Parameters:
- `repo` - Absolute path to repository root
- `report` - Absolute path to VulnHunt findings report
- `fixed` - Comma-separated list of vulnerability IDs to verify
- `out` - Absolute path to output directory (must exist)
- `comments` - (Optional) Path to PR review comments JSON
- `additional_repos` - (Optional) Comma-separated paths to dependency repos

## Headless Runtime Agent

For CI/CD or non-interactive pipelines:

```bash
cd vulnhunter-agent
pip install -e ".[dev]"

# Configure via config.yaml
cp config.example.yaml config.yaml
# Edit config.yaml with your settings
```

Example `config.yaml`:

```yaml
anthropic_api_key: ${ANTHROPIC_API_KEY}
model: claude-opus-4-20250514
github_token: ${GITHUB_TOKEN}

targets:
  - url: https://github.com/org/repo
    branch: main
    language: python
    create_issues: true

output_dir: ./scan-results
max_concurrent: 3
```

Run headless scan:

```bash
python -m vulnhunter_agent.main --config config.yaml
```

The agent will:
- Clone target repositories
- Execute `/vulnhunt` skill
- Parse findings
- Create GitHub issues for verified vulnerabilities

## Batch Scanning with Harness

For workstation-scale batch operations:

```bash
cd harness
pip install -e ".[dev]"

# Configure target list
echo "https://github.com/org/repo1" >> local_harness/batch/REPO_LIST.txt
echo "https://github.com/org/repo2" >> local_harness/batch/REPO_LIST.txt
# Lines starting with # are ignored

# Run batch scan
python -m local_harness.batch.run scan

# Resume interrupted scan
python -m local_harness.batch.run scan --resume

# Check progress
python -m local_harness.batch.run status

# Collect all findings
python -m local_harness.batch.run collect
```

Results are stored in `harness/local_harness/batch/workdir/`.

## Benchmarking Mode

Evaluate scanner accuracy against known vulnerabilities:

```bash
cd harness
pip install -e ".[dev]"

# Run full benchmark
python -m local_harness.benchmark.run

# Benchmark specific repository
python -m local_harness.benchmark.run --repos "OWASP/NodeGoat"

# Regenerate metrics only
python -m local_harness.benchmark.run --tally-only
```

### Creating Ground Truth

Define known vulnerabilities in `harness/local_harness/benchmark/ground_truth/<repo>.json`:

```json
{
  "repo_name": "my-vulnerable-app",
  "repo_url": "https://github.com/org/my-vulnerable-app",
  "vulnerabilities": [
    {
      "id": "SQL-001",
      "type": "SQL Injection",
      "file": "src/db/user.py",
      "line": 42,
      "severity": "critical",
      "description": "Unsanitized user input in SQL query",
      "exploit_vector": "username parameter in /api/login"
    }
  ]
}
```

Configure benchmark engines in `harness/local_harness/config.py`:

```python
BENCHMARK_CONFIG = {
    "scanner_model": "claude-opus-4-20250514",
    "judge_model": "claude-opus-4-20250514",
    "max_workers": 4
}
```

## Configuration

### VulnHunt Scanner Configuration

The scanner is prompt-driven. Customize behavior by editing `vulnhunt/SKILL.md` and phase files in `vulnhunt/phases/`:

- `vulnhunt/phases/recon.md` - Entry point discovery
- `vulnhunt/phases/hunt.md` - Path tracing logic
- `vulnhunt/phases/disprove.md` - Falsification rules
- `vulnhunt/phases/capability.md` - Exploitability verification

### VulnHunter-Fix Configuration

Edit `vulnhunter-fix/vulnhunter_fix/config.py`:

```python
DEFAULT_CONFIG = {
    "test_framework": "pytest",  # or "unittest", "jest", etc.
    "create_pr": True,
    "pr_base_branch": "main",
    "require_tests": True,
    "max_fix_attempts": 3
}
```

Or override at runtime by creating `.vulnhunter-fix.yaml` in project root:

```yaml
test_framework: pytest
create_pr: true
pr_base_branch: develop
require_tests: true
max_fix_attempts: 5
```

## Common Patterns

### Pattern 1: Quick Security Audit

```bash
# One-shot vulnerability scan
claude --model opus \
  --add-dir ~/.claude/skills/vulnhunt \
  --add-dir ~/.claude/skills/vulnhunt/phases \
  << 'EOF'
/vulnhunt
EOF
```

### Pattern 2: Full Remediation Workflow

```python
#!/usr/bin/env python3
"""Complete hunt-fix-verify workflow"""
import subprocess
import json
from pathlib import Path

def run_workflow(repo_path: Path):
    # Step 1: Scan
    print("[1/3] Scanning for vulnerabilities...")
    subprocess.run([
        "claude", "--model", "opus",
        "--add-dir", "~/.claude/skills/vulnhunt",
        "--add-dir", "~/.claude/skills/vulnhunt/phases",
        "-c", "/vulnhunt"
    ], cwd=repo_path)
    
    # Step 2: Fix
    print("[2/3] Applying fixes...")
    subprocess.run([
        "claude", "--model", "opus",
        "--add-dir", "~/.claude/skills/vulnhunter-fix",
        "-c", "/vulnhunter-fix"
    ], cwd=repo_path)
    
    # Step 3: Verify
    print("[3/3] Verifying fixes...")
    out_dir = Path("/tmp/verify-out")
    out_dir.mkdir(exist_ok=True)
    
    subprocess.run([
        "claude", "--model", "opus",
        "--add-dir", "~/.claude/skills/vulnhunt-fix-verify",
        "--add-dir", "~/.claude/skills/vulnhunt-fix-verify/phases",
        "-c", f"/vulnhunt-fix-verify repo={repo_path} report={repo_path}/vulnhunt_report.md out={out_dir}"
    ])

if __name__ == "__main__":
    run_workflow(Path.cwd())
```

### Pattern 3: CI/CD Integration

```yaml
# .github/workflows/vulnhunt.yml
name: VulnHunter Security Scan

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2 AM
  workflow_dispatch:

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      
      - name: Install VulnHunter Agent
        run: |
          git clone https://github.com/capitalone/vulnhunter.git
          cd vulnhunter/vulnhunter-agent
          pip install -e ".[dev]"
      
      - name: Run Security Scan
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          cd vulnhunter/vulnhunter-agent
          cat > config.yaml << EOF
          anthropic_api_key: ${ANTHROPIC_API_KEY}
          github_token: ${GITHUB_TOKEN}
          model: claude-opus-4-20250514
          targets:
            - url: ${{ github.server_url }}/${{ github.repository }}
              branch: ${{ github.ref_name }}
              create_issues: true
          output_dir: ./results
          EOF
          python -m vulnhunter_agent.main --config config.yaml
      
      - name: Upload Results
        uses: actions/upload-artifact@v4
        with:
          name: vulnhunt-results
          path: vulnhunter/vulnhunter-agent/results/
```

### Pattern 4: Custom Falsification Rules

Extend the falsification engine by adding constraints to `vulnhunt/phases/disprove.md`:

```markdown
## Custom Falsification Rules

### Framework-Specific Protections

**Django ORM**: If the code uses Django ORM methods like `.filter()`, 
`.get()`, `.exclude()` with keyword arguments (not raw SQL), the ORM 
provides automatic parameterization. Flag this as NOT exploitable.

**Express.js Helmet**: If `helmet()` middleware is configured and the 
vulnerability is XSS-related, verify CSP headers block inline script 
execution.

### Language-Specific Checks

**Python Type Hints**: If function signatures use strict type hints 
with runtime validation (e.g., Pydantic), trace whether malicious input 
can bypass type coercion.

**Rust Ownership**: For memory safety issues, verify whether Rust's 
borrow checker already prevents the exploit at compile time.
```

## Testing

Each component has isolated test suites:

```bash
# Test harness
cd harness
pip install -e ".[dev]"
python -m pytest tests/ --cov=local_harness

# Test fixer
cd vulnhunter-fix
pip install -e ".[dev]"
python -m pytest -q

# Test agent
cd vulnhunter-agent
pip install -e ".[dev]"
python -m pytest -q
```

## Troubleshooting

### Issue: Cyber Safeguard Blocks

**Symptom**: Requests blocked with cyber abuse warnings

**Solution**: Enroll in [Anthropic Cyber Verification Program](https://portal.anthropic.com/programs/cvp)

```bash
# Verify enrollment status
curl https://api.anthropic.com/v1/cyber/status \
  -H "x-api-key: $ANTHROPIC_API_KEY"
```

### Issue: High False Positive Rate

**Symptom**: Many reported vulnerabilities are not exploitable

**Solution**: Strengthen falsification phase:

1. Review `vulnhunt/phases/disprove.md`
2. Add domain-specific sanitization patterns
3. Increase falsification iterations in `SKILL.md`:

```markdown
## Falsification Configuration

Run **3 independent falsification passes** (up from default 1):
- Pass 1: Check input validation
- Pass 2: Verify sanitization/encoding
- Pass 3: Confirm no bypass via edge cases
```

### Issue: Scanner Misses Known Vulnerabilities

**Symptom**: Benchmark shows low recall

**Solution**: Expand reconnaissance phase:

1. Edit `vulnhunt/phases/recon.md` to include more entry point types
2. Add framework-specific entry points:

```markdown
## Additional Entry Points

**FastAPI**: `@app.post()`, `@app.get()` decorated async functions
**Spring Boot**: `@RestController`, `@RequestMapping` methods
**GraphQL**: `resolve_*` functions in schema definitions
```

### Issue: Verifier Rejects Valid Fixes

**Symptom**: `/vulnhunt-fix-verify` marks fixes as unsuccessful

**Solution**: Check verification logs for specific failure reason:

```bash
cat /path/to/verify-output/VULN-001-verification.log
```

Common causes:
- Test not exercising exact exploit path
- Fix incomplete (e.g., sanitizes one parameter but not all)
- Regression introduced (unrelated functionality broken)

### Issue: Headless Agent Crashes

**Symptom**: `vulnhunter-agent` exits with errors

**Solution**: Enable debug logging:

```yaml
# config.yaml
log_level: DEBUG
log_file: ./agent-debug.log
```

Check common issues:
- Invalid GitHub token: `gh auth status`
- API rate limits: `curl -H "x-api-key: $ANTHROPIC_API_KEY" https://api.anthropic.com/v1/usage`
- Disk space: `df -h`

### Issue: Skill Not Found

**Symptom**: `/vulnhunt` command not recognized

**Solution**: Verify installation:

```bash
ls -la ~/.claude/skills/vulnhunt
ls -la ~/.claude/skills/vulnhunt/phases

# Reinstall if missing
cd /path/to/vulnhunter
./install.sh
```

## Environment Variables

```bash
# Required for Anthropic API access
export ANTHROPIC_API_KEY="your-api-key"

# Required for GitHub integration
export GITHUB_TOKEN="your-github-token"

# Optional: Custom skill directory
export CLAUDE_SKILLS_DIR="$HOME/custom-skills"

# Optional: Control batch concurrency
export VULNHUNT_MAX_WORKERS=4

# Optional: Verification timeout (seconds)
export VULNHUNT_VERIFY_TIMEOUT=1800
```

## Additional Resources

- **Official Docs**: [VulnHunter GitHub](https://github.com/capitalone/vulnhunter)
- **Claude Code Docs**: [docs.claude.com/claude-code](https://docs.claude.com/en/docs/claude-code)
- **Cyber Verification**: [Anthropic CVP Portal](https://portal.anthropic.com/programs/cvp)
- **Contributing**: See `CONTRIBUTING.md` in repository
- **Security**: Report issues per `SECURITY.md` guidelines
