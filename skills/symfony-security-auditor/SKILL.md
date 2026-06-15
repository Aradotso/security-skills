---
name: symfony-security-auditor
description: AI-powered multi-agent security auditor for Symfony applications using LLMs to detect business logic flaws, broken access control, and complex vulnerabilities
triggers:
  - audit my Symfony application for security vulnerabilities
  - run Symfony security audit with AI
  - check for broken access control in Symfony
  - scan Symfony project for OWASP vulnerabilities
  - detect business logic flaws in my Symfony app
  - generate security report for Symfony using LLMs
  - find missing Voters and authorization issues
  - perform AI-powered penetration testing on Symfony
---

# Symfony Security Auditor Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## What It Does

`symfony-security-auditor` is an AI-powered multi-agent security auditor for Symfony applications that catches vulnerabilities traditional SAST tools miss:

- **Business logic flaws** — authorization bypasses, workflow violations
- **Broken access control** — missing `#[IsGranted]`, missing Voters, IDOR
- **Complex injection chains** — multi-file SQL injection, XSS, deserialization
- **Mass assignment** — unsafe `MapRequestPayload`, form handling
- **Symfony-specific issues** — firewall misconfigurations, unsafe Twig, Messenger handler flaws

Uses a three-stage pipeline:
1. **Ingestion** — scans `.php`, `.twig`, `.yaml`, `.xml` recursively
2. **Mapping** — classifies Controllers, Entities, Voters, Forms, Routes
3. **Audit** — adversarial Attacker agent finds issues, skeptical Reviewer agent eliminates false positives over up to 3 iterations

Outputs to console, JSON, SARIF (GitHub Code Scanning / GitLab Security Dashboard), or HTML.

**Provider-agnostic** via `symfony/ai` — works with Claude, GPT, Gemini, Mistral, Llama, DeepSeek, Ollama.

## Installation

### With Symfony Flex (automatic setup)

```bash
composer require --dev vinceamstoutz/symfony-security-auditor
```

Flex automatically:
- Registers the bundle in `config/bundles.php` for `dev` and `test` environments
- Creates `config/packages/symfony_security_auditor.yaml` with defaults

### Install an AI platform bridge

```bash
# Anthropic Claude (recommended)
composer require symfony/ai-anthropic-platform

# Or OpenAI GPT
composer require symfony/ai-openai-platform

# Or Google Gemini
composer require symfony/ai-google-gemini-platform

# Or Mistral
composer require symfony/ai-mistral-platform

# Or Ollama (local)
composer require symfony/ai-ollama-platform
```

### Configure the AI platform

Create or edit `config/packages/ai.yaml`:

```yaml
# Anthropic Claude
ai:
  platform:
    anthropic:
      api_key: '%env(ANTHROPIC_API_KEY)%'

# Or OpenAI GPT
ai:
  platform:
    openai:
      api_key: '%env(OPENAI_API_KEY)%'

# Or Google Gemini
ai:
  platform:
    google_gemini:
      api_key: '%env(GOOGLE_GEMINI_API_KEY)%'

# Or Ollama (local, no API key)
ai:
  platform:
    ollama:
      base_url: 'http://localhost:11434'
```

Set environment variables in `.env.local`:

```bash
ANTHROPIC_API_KEY=sk-ant-api03-...
# Or
OPENAI_API_KEY=sk-proj-...
# Or
GOOGLE_GEMINI_API_KEY=...
```

### Manual setup (without Flex)

Register bundles in `config/bundles.php`:

```php
return [
    // ...
    Symfony\AI\AiBundle\AiBundle::class => ['all' => true],
    VinceAmstoutz\SymfonySecurityAuditor\SymfonySecurityAuditorBundle::class => ['dev' => true, 'test' => true],
];
```

Create `config/packages/symfony_security_auditor.yaml`:

```yaml
symfony_security_auditor:
    model: 'claude-opus-4-8'
    profile: 'balanced' # fast, balanced, or thorough
```

## Configuration

### Basic configuration

Edit `config/packages/symfony_security_auditor.yaml`:

```yaml
symfony_security_auditor:
    # Model name (provider-specific)
    model: 'claude-opus-4-8' # or 'gpt-4o', 'gemini-2.0-flash-exp', etc.
    
    # One-knob preset: fast, balanced (default), or thorough
    profile: 'balanced'
    
    # Manual tuning (overrides profile)
    max_review_iterations: 2
    enable_attacker_tools: true
    enable_reviewer_tools: false
    enable_poc_generation: false
    enable_concurrent_review: false
    enable_escalation: false
    chunk_token_limit: 8000
    enable_lean_prescan: false
```

### Split-model configuration (cost optimization)

Use a powerful model for attack, cheap model for review:

```yaml
symfony_security_auditor:
    attacker_model: 'claude-opus-4-8'
    reviewer_model: 'claude-haiku-4'
```

Cuts cost ~20× while maintaining high detection rate.

### Rate limiting

```yaml
symfony_security_auditor:
    rate_limit:
        requests_per_minute: 50
        tokens_per_minute: 80000
```

### Profiles explained

```yaml
# Fast: minimal iterations, no tools, no PoC (~$0.50/project)
profile: 'fast'

# Balanced: 2 iterations, attacker tools, selective PoC (~$2-5/project)
profile: 'balanced'

# Thorough: 3 iterations, all tools, PoC for all high-severity (~$10-20/project)
profile: 'thorough'
```

## Key Commands

### Run security audit

```bash
# Audit current directory
bin/console audit:run

# Audit specific project
bin/console audit:run /path/to/project

# Dry run (estimate cost, no actual audit)
bin/console audit:run --dry-run

# Output formats
bin/console audit:run --format console  # human-readable (default)
bin/console audit:run --format json --output report.json
bin/console audit:run --format sarif --output report.sarif
bin/console audit:run --format html --output report.html

# Diff mode (audit only changed files since branch)
bin/console audit:run --since=main

# Baseline suppression
bin/console audit:run --generate-baseline  # creates baseline.json
bin/console audit:run --baseline baseline.json  # suppress known findings

# Verbose output
bin/console audit:run -v   # show progress
bin/console audit:run -vv  # show API calls
bin/console audit:run -vvv # debug mode
```

### Exit codes

- `0` — no vulnerabilities found
- `1` — vulnerabilities found
- `2` — audit failed (error)

## Real-World Usage Examples

### Example 1: Basic security audit

```bash
# Install
composer require --dev vinceamstoutz/symfony-security-auditor
composer require symfony/ai-anthropic-platform

# Configure (add to .env.local)
echo "ANTHROPIC_API_KEY=sk-ant-api03-..." >> .env.local

# Run
bin/console audit:run

# Output:
# ┌─────────────────────────────────────────────────────────
# │ HIGH: Missing Authorization Check in Delete Action
# ├─────────────────────────────────────────────────────────
# │ File: src/Controller/PostController.php:42
# │ Category: Broken Access Control
# │
# │ The delete() method allows any authenticated user to
# │ delete any post without checking ownership.
# │
# │ Recommendation:
# │ Add #[IsGranted('POST_DELETE', 'post')] or create a Voter
# └─────────────────────────────────────────────────────────
```

### Example 2: CI integration with SARIF output

```bash
# Generate SARIF for GitHub Code Scanning
bin/console audit:run \
    --format sarif \
    --output symfony-security.sarif \
    --since=main

# Upload to GitHub
gh api repos/{owner}/{repo}/code-scanning/sarifs \
    -F sarif=@symfony-security.sarif \
    -F commit_sha=$(git rev-parse HEAD) \
    -F ref=refs/heads/main
```

### Example 3: Split-model configuration for cost savings

```yaml
# config/packages/symfony_security_auditor.yaml
symfony_security_auditor:
    attacker_model: 'claude-opus-4-8'  # powerful, expensive
    reviewer_model: 'claude-haiku-4'    # fast, cheap
    profile: 'balanced'
```

```bash
bin/console audit:run --dry-run
# Estimated cost: $2.34 (vs $45.67 with single Opus model)

bin/console audit:run
```

### Example 4: Baseline workflow for existing projects

```bash
# First run: generate baseline of known issues
bin/console audit:run --generate-baseline

# This creates baseline.json with all findings
# Review and accept known issues, then commit baseline.json

# Subsequent runs: only report new issues
bin/console audit:run --baseline baseline.json

# Exit code 0 if only baseline findings
# Exit code 1 if new findings detected
```

### Example 5: HTML report for stakeholders

```bash
bin/console audit:run \
    --format html \
    --output security-report-$(date +%Y%m%d).html

# Generates self-contained HTML report with:
# - Executive summary
# - Findings grouped by severity
# - File/line locations
# - Recommendations
# - PoCs (if enabled)
```

## Common Patterns

### Pattern 1: GitHub Actions workflow

Create `.github/workflows/security-audit.yml`:

```yaml
name: Security Audit
on:
  schedule:
    - cron: '0 2 * * *'  # nightly at 2 AM
  workflow_dispatch:

jobs:
  audit:
    runs-on: ubuntu-latest
    permissions:
      security-events: write  # for SARIF upload
      contents: read
    steps:
      - uses: actions/checkout@v4
      
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      
      - run: composer install --no-dev
      
      - uses: vinceamstoutz/symfony-security-auditor@1.10.0
        with:
          format: sarif
          output: symfony-security.sarif
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: symfony-security.sarif
          category: symfony-security-auditor
```

### Pattern 2: GitLab CI pipeline

Create `.gitlab-ci.yml`:

```yaml
security_audit:
  stage: test
  image: php:8.3
  script:
    - composer install --dev
    - bin/console audit:run --format sarif --output symfony-security.sarif
  artifacts:
    reports:
      sast: symfony-security.sarif
  variables:
    ANTHROPIC_API_KEY: $ANTHROPIC_API_KEY
  only:
    - schedules
```

### Pattern 3: Pull request diff audit

```yaml
# .github/workflows/pr-audit.yml
name: PR Security Audit
on: pull_request

jobs:
  audit-diff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # full history for --since
      
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      
      - run: composer install --dev
      
      - name: Audit changed files
        run: |
          bin/console audit:run \
            --since=origin/${{ github.base_ref }} \
            --format json \
            --output audit.json
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      
      - name: Comment on PR
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const audit = JSON.parse(fs.readFileSync('audit.json'));
            const comment = `## 🔒 Security Audit Found Issues\n\n${audit.summary}`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

### Pattern 4: Programmatic usage in PHP

```php
<?php

namespace App\Service;

use VinceAmstoutz\SymfonySecurityAuditor\Application\Service\AuditOrchestrator;
use VinceAmstoutz\SymfonySecurityAuditor\Domain\ValueObject\AuditOptions;

class SecurityAuditService
{
    public function __construct(
        private AuditOrchestrator $orchestrator
    ) {}

    public function auditProject(string $projectPath): array
    {
        $options = new AuditOptions(
            projectPath: $projectPath,
            format: 'json',
            dryRun: false,
            since: null,
            baseline: null
        );

        $result = $this->orchestrator->execute($options);

        return [
            'vulnerabilities' => $result->getVulnerabilities(),
            'summary' => $result->getSummary(),
            'cost' => $result->getCost(),
        ];
    }
}
```

### Pattern 5: Custom vulnerability filters

```php
<?php

namespace App\Security;

use VinceAmstoutz\SymfonySecurityAuditor\Domain\Model\Vulnerability;

class VulnerabilityFilter
{
    public function filterCritical(array $vulnerabilities): array
    {
        return array_filter(
            $vulnerabilities,
            fn(Vulnerability $v) => $v->getSeverity() === 'CRITICAL'
        );
    }

    public function filterByCategory(array $vulnerabilities, string $category): array
    {
        return array_filter(
            $vulnerabilities,
            fn(Vulnerability $v) => $v->getCategory() === $category
        );
    }

    public function excludeAccepted(array $vulnerabilities, array $acceptedIds): array
    {
        return array_filter(
            $vulnerabilities,
            fn(Vulnerability $v) => !in_array($v->getId(), $acceptedIds)
        );
    }
}
```

## Troubleshooting

### Issue: "Model not found" error

**Cause**: Incorrect model name for the provider.

**Solution**: Check provider-specific model names:

```yaml
# Anthropic Claude
model: 'claude-opus-4-8'      # or 'claude-sonnet-4', 'claude-haiku-4'

# OpenAI
model: 'gpt-4o'                # or 'gpt-4-turbo', 'gpt-3.5-turbo'

# Google Gemini
model: 'gemini-2.0-flash-exp'  # or 'gemini-1.5-pro'

# Mistral
model: 'mistral-large-latest'  # or 'mistral-small-latest'

# Ollama (local)
model: 'llama3.1'              # or 'codellama', 'deepseek-coder'
```

### Issue: High API costs

**Solution 1**: Use split-model configuration:

```yaml
symfony_security_auditor:
    attacker_model: 'claude-opus-4-8'
    reviewer_model: 'claude-haiku-4'  # ~20× cheaper
```

**Solution 2**: Enable lean prescan to drop low-risk files:

```yaml
symfony_security_auditor:
    enable_lean_prescan: true
```

**Solution 3**: Use `fast` profile for initial scans:

```yaml
symfony_security_auditor:
    profile: 'fast'
```

**Solution 4**: Audit only changed files:

```bash
bin/console audit:run --since=main
```

### Issue: Rate limit errors (429)

**Solution**: Configure rate limits:

```yaml
symfony_security_auditor:
    rate_limit:
        requests_per_minute: 50    # adjust per provider limits
        tokens_per_minute: 80000
```

### Issue: Too many false positives

**Solution 1**: Use baseline suppression:

```bash
bin/console audit:run --generate-baseline
# Review baseline.json, remove false positives
bin/console audit:run --baseline baseline.json
```

**Solution 2**: Increase review iterations:

```yaml
symfony_security_auditor:
    max_review_iterations: 3  # more iterations = fewer false positives
```

**Solution 3**: Use a stronger reviewer model:

```yaml
symfony_security_auditor:
    reviewer_model: 'claude-opus-4-8'  # instead of haiku
```

### Issue: Missing vulnerabilities

**Solution 1**: Enable all detection features:

```yaml
symfony_security_auditor:
    profile: 'thorough'
    enable_attacker_tools: true
    enable_reviewer_tools: true
```

**Solution 2**: Increase chunk token limit for complex files:

```yaml
symfony_security_auditor:
    chunk_token_limit: 16000  # default is 8000
```

**Solution 3**: Disable lean prescan:

```yaml
symfony_security_auditor:
    enable_lean_prescan: false
```

### Issue: Audit runs slowly

**Solution 1**: Enable concurrent review:

```yaml
symfony_security_auditor:
    enable_concurrent_review: true
```

**Solution 2**: Use faster models:

```yaml
symfony_security_auditor:
    model: 'claude-haiku-4'  # or 'gpt-4o-mini', 'gemini-2.0-flash-exp'
```

**Solution 3**: Reduce review iterations:

```yaml
symfony_security_auditor:
    max_review_iterations: 1
```

### Issue: SARIF upload fails on GitHub

**Cause**: Missing `security-events: write` permission.

**Solution**: Add permission to workflow:

```yaml
permissions:
  security-events: write
  contents: read
```

### Issue: Ollama connection refused

**Cause**: Ollama server not running.

**Solution**: Start Ollama server:

```bash
ollama serve

# In another terminal, pull model
ollama pull llama3.1

# Then run audit
bin/console audit:run
```

### Issue: "No findings" on project with known vulnerabilities

**Cause**: Model not sophisticated enough, or context window too small.

**Solution**: Use a more powerful model:

```yaml
symfony_security_auditor:
    model: 'claude-opus-4-8'  # best detection rate
    profile: 'thorough'
```

### Issue: Cost estimation is inaccurate

**Cause**: Provider doesn't support prompt caching, or cache not warm.

**Solution**: Run `--dry-run` twice:

```bash
# First run: no cache
bin/console audit:run --dry-run
# Estimated: $12.50

# Second run: cache warm (Anthropic only)
bin/console audit:run --dry-run
# Estimated: $1.80 (90% input tokens cached)
```

## Advanced Features

### Enable PoC generation for high-severity findings

```yaml
symfony_security_auditor:
    enable_poc_generation: true
    poc_severity_threshold: 'HIGH'  # HIGH or CRITICAL
```

Output includes concrete reproduction steps:

```json
{
  "poc": {
    "type": "curl",
    "command": "curl -X DELETE https://app.example.com/api/posts/123 -H 'Authorization: Bearer USER_TOKEN'",
    "expected": "Post deleted without ownership check"
  }
}
```

### Custom advisory feeds

Implement `AdvisoryProviderInterface`:

```php
<?php

namespace App\Security;

use VinceAmstoutz\SymfonySecurityAuditor\Domain\Service\AdvisoryProviderInterface;

class CustomAdvisoryProvider implements AdvisoryProviderInterface
{
    public function lookup(string $package, string $version): ?array
    {
        // Query your internal vulnerability database
        return [
            'id' => 'INTERNAL-2024-001',
            'title' => 'Custom vulnerability',
            'severity' => 'HIGH',
        ];
    }
}
```

Register in `services.yaml`:

```yaml
services:
    App\Security\CustomAdvisoryProvider:
        decorates: VinceAmstoutz\SymfonySecurityAuditor\Domain\Service\AdvisoryProviderInterface
```

### Content-hash caching

Automatically enabled — identical code chunks skip LLM:

```bash
# First run
bin/console audit:run
# 50 chunks analyzed, cost: $5.00

# No code changes
bin/console audit:run
# 50 chunks cached, cost: $0.00
```

Cache stored in `var/cache/dev/symfony_security_auditor/`.

---

**Tip**: For best results, combine this auditor with PHPStan (type safety), Psalm Security (taint analysis), and Dependabot (dependency CVEs). This auditor excels at business logic flaws that static analysis cannot detect.
