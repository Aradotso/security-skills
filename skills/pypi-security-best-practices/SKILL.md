---
name: pypi-security-best-practices
description: Guide for implementing security best practices when using Python packages from PyPI with uv and pip
triggers:
  - how do I secure my Python package installations
  - show me PyPI security best practices
  - how to prevent supply chain attacks in Python
  - configure uv with security hardening
  - implement dependency cooldown for PyPI packages
  - verify package hashes with pip and uv
  - secure my Python development environment
  - protect against malicious PyPI packages
---

# PyPI Security Best Practices

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive guidance on securing Python package installations from PyPI, covering supply chain attack mitigation, dependency verification, and secure development practices for both `uv` and `pip` package managers.

## Overview

PyPI security best practices help protect against supply chain attacks like the LiteLLM/Telnyx incident (119k+ malicious downloads in under 3 hours) and other compromised package scenarios. This guide covers secure package installation, dependency management, and development environment hardening.

**Key Security Principles:**
- Prefer binary-only installations to avoid arbitrary code execution
- Implement dependency cooldowns to avoid newly-published malicious packages
- Pin dependencies with cryptographic hash verification
- Use deterministic installations and prevent dependency confusion
- Scan for vulnerabilities and verify package health

## Installation

### uv (Recommended)

```bash
# Install uv (macOS/Linux)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install uv (Windows)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Verify installation
uv --version
```

### pip

```bash
# pip is included with Python 3.4+
python -m pip --version

# Upgrade to latest pip
python -m pip install --upgrade pip
```

### Security Tools

```bash
# Install pip-audit for vulnerability scanning
python -m pip install pip-audit

# Install uv-secure for lockfile scanning
uv tool install uv-secure
```

## Core Security Practices

### 1. Binary-Only Installations

Source distributions can execute arbitrary code via `setup.py`. Enforce binary-only installs:

**With uv:**
```bash
# Command line
uv pip install --only-binary :all: requests

# In pyproject.toml
[tool.uv.pip]
only-binary = [":all:"]

# In uv.toml
[pip]
only-binary = [":all:"]
```

**With pip:**
```bash
# Command line
pip install --only-binary :all: requests

# Environment variable
export PIP_ONLY_BINARY=:all:
pip install requests

# In pip.conf (Linux/macOS: ~/.config/pip/pip.conf)
[install]
only-binary = :all:
```

### 2. Dependency Cooldowns

Avoid newly-published malicious packages by excluding recent releases:

**With uv:**
```toml
# pyproject.toml
[tool.uv]
exclude-newer = "7 days"  # Recommended for general use

# Or more aggressive for production
exclude-newer = "30 days"
```

```bash
# Command line usage
uv lock --exclude-newer "7 days"
uv sync --exclude-newer "7 days"

# Environment variable
export UV_EXCLUDE_NEWER="7 days"
uv sync
```

**Per-package overrides:**
```toml
# pyproject.toml - exempt security patches
[tool.uv]
exclude-newer = "7 days"
exclude-newer-package = { requests = "1 day" }
```

**With pip (v26.1+):**
```ini
# ~/.config/pip/pip.conf
[install]
uploaded-prior-to = P7D
```

```bash
# Command line (absolute date)
pip install --uploaded-prior-to=2026-06-01 requests

# Bypass cooldown for urgent patches
pip install --uploaded-prior-to=P0D requests==2.32.3
```

**Dependabot cooldown:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    cooldown: 7  # Wait 7 days after release
```

**Renovate cooldown:**
```json
{
  "packageRules": [
    {
      "matchDatasources": ["pypi"],
      "minimumReleaseAge": "7 days"
    }
  ]
}
```

### 3. Hash Verification

Always verify package integrity with cryptographic hashes:

**With uv (automatic in lockfile):**
```bash
# Generate lockfile with hashes
uv lock

# Install with hash verification (automatic)
uv sync

# For requirements.txt workflow
uv pip compile --generate-hashes requirements.in -o requirements.txt
uv pip install -r requirements.txt
```

**Example lockfile entry:**
```toml
[[package]]
name = "requests"
version = "2.32.3"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "certifi" },
    { name = "charset-normalizer" },
]
wheels = [
    { url = "https://files.pythonhosted.org/packages/.../requests-2.32.3-py3-none-any.whl", hash = "sha256:70761cfe03c773ceb22aa2f671b4757976145175cdfca038c02654d061d6dcc6" },
]
```

**With pip:**
```bash
# Generate hashed requirements
pip-compile --generate-hashes requirements.in

# Install with hash verification
pip install --require-hashes -r requirements.txt
```

**Example requirements.txt with hashes:**
```txt
requests==2.32.3 \
    --hash=sha256:70761cfe03c773ceb22aa2f671b4757976145175cdfca038c02654d061d6dcc6 \
    --hash=sha256:55365417734eb18255590a9ff9eb97e9e1da868d4ccd6402399eaf68af20a760
certifi==2024.2.2 \
    --hash=sha256:0569859f95fc761b18b45ef421b1290a0f65f147e92a1e5eb3e635f9a5e4e66f
```

### 4. Deterministic Installations

Use lockfiles for reproducible builds:

**With uv:**
```bash
# Create lockfile
uv lock

# Install exact versions from lockfile
uv sync

# Install without updating lockfile
uv sync --frozen
```

**With pip:**
```bash
# Generate pinned requirements
pip freeze > requirements.txt

# Or use pip-tools
pip-compile requirements.in -o requirements.txt

# Install exact versions
pip install -r requirements.txt
```

### 5. Prevent Dependency Confusion

Configure package sources to prevent private/public namespace collisions:

**With uv:**
```toml
# pyproject.toml
[[tool.uv.index]]
name = "company-internal"
url = "https://pypi.company.com/simple"
explicit = true  # Only use for explicitly specified packages

[[tool.uv.index]]
name = "pypi"
url = "https://pypi.org/simple"
default = true
```

**With pip:**
```ini
# pip.conf
[global]
index-url = https://pypi.org/simple
extra-index-url = 
    https://pypi.company.com/simple

[install]
# Require that private packages come from internal index
trusted-host = pypi.company.com
```

### 6. Vulnerability Scanning

Regularly scan dependencies for known vulnerabilities:

**With pip-audit:**
```bash
# Scan installed packages
pip-audit

# Scan requirements file
pip-audit -r requirements.txt

# Output as JSON
pip-audit --format json -o audit.json

# Fix vulnerabilities automatically
pip-audit --fix

# Ignore specific vulnerabilities
pip-audit --ignore-vuln PYSEC-2024-1234
```

**With uv-secure:**
```bash
# Scan uv lockfile
uv-secure scan

# Fail CI on vulnerabilities
uv-secure scan --exit-code

# Generate SARIF for GitHub
uv-secure scan --format sarif -o results.sarif
```

**In CI/CD (GitHub Actions):**
```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install uv
        uses: astral-sh/setup-uv@v3
      
      - name: Scan for vulnerabilities
        run: |
          uv tool install uv-secure
          uv-secure scan --exit-code
```

### 7. Harden Package Installs with Security Tools

**Socket.dev for real-time protection:**
```bash
# Install Socket CLI
npm install -g @socketsecurity/cli

# Scan Python dependencies
socket python scan requirements.txt

# Monitor CI/CD
socket ci
```

**Phylum for supply chain analysis:**
```bash
# Install Phylum
curl -sSL https://sh.phylum.io/ | sh

# Analyze dependencies
phylum analyze requirements.txt

# Block malicious packages in CI
phylum check requirements.txt --fail-on-critical
```

## Secure Local Development

### 8. No Plaintext Secrets in .env Files

Use secret management instead of plaintext `.env` files:

**With 1Password:**
```bash
# Store secret
op item create --category=password \
  --title "API_KEY" \
  --vault "Development" \
  password="${API_KEY_VALUE}"

# Load secrets into environment
eval $(op inject -i .env.template -o .env)

# Run with secrets
op run -- python app.py
```

**.env.template (commit this):**
```bash
API_KEY=op://Development/API_KEY/password
DATABASE_URL=op://Development/DATABASE_URL/password
```

**With doppler:**
```bash
# Install doppler
brew install dopplerhq/cli/doppler  # macOS
# or curl -Ls https://cli.doppler.com/install.sh | sh

# Login and setup
doppler login
doppler setup

# Run with secrets
doppler run -- python app.py
```

### 9. Work in Dev Containers

Isolate development environments with containers:

**devcontainer.json:**
```json
{
  "name": "Python Development",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "features": {
    "ghcr.io/devcontainers/features/uv:1": {}
  },
  "postCreateCommand": "uv sync",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "charliermarsh.ruff"
      ]
    }
  },
  "remoteEnv": {
    "UV_EXCLUDE_NEWER": "7 days"
  }
}
```

**Docker Compose for local development:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/workspace
      - uv-cache:/root/.cache/uv
    environment:
      - UV_EXCLUDE_NEWER=7 days
      - UV_NO_SYNC=1
    command: uv run python app.py

volumes:
  uv-cache:
```

## Maintainer Security Practices

### 10. Enable 2FA for PyPI Accounts

```bash
# PyPI requires 2FA for all accounts
# Visit https://pypi.org/manage/account/two-factor/
# Use TOTP app or security key (recommended)
```

### 11. Publish with Trusted Publishing (OIDC)

**GitHub Actions workflow:**
```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

permissions:
  id-token: write  # Required for trusted publishing

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: astral-sh/setup-uv@v3
      
      - name: Build package
        run: uv build
      
      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          # No API token needed - uses OIDC
          skip-existing: true
```

**Configure on PyPI:**
1. Go to https://pypi.org/manage/account/publishing/
2. Add GitHub repository
3. Specify workflow name and environment

### 12. Publish with Package Attestations

Generate provenance attestations:

```yaml
name: Publish with Attestations

on:
  release:
    types: [published]

permissions:
  id-token: write
  contents: read
  attestations: write

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: astral-sh/setup-uv@v3
      
      - name: Build
        run: uv build
      
      - name: Generate attestations
        uses: actions/attest-build-provenance@v1
        with:
          subject-path: dist/*
      
      - name: Publish
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          attestations: true
```

**Verify attestations:**
```bash
# Download and verify package attestations
pip download --no-deps requests==2.32.3
gh attestation verify requests-2.32.3-py3-none-any.whl \
  --owner psf
```

### 13. Secure CI/CD Release Pipeline

**Branch protection and signed commits:**
```yaml
# .github/workflows/release.yml
name: Secure Release

on:
  push:
    tags:
      - 'v*'

jobs:
  security-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Verify signed commits
      - name: Verify signatures
        run: |
          git verify-commit HEAD || exit 1
      
      # Scan dependencies
      - name: Vulnerability scan
        run: |
          uv tool install uv-secure
          uv-secure scan --exit-code
      
      # SBOM generation
      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          format: cyclonedx-json
          output-file: sbom.json
      
      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json
```

### 14. Reduce Package Dependency Tree

Minimize dependencies to reduce attack surface:

```bash
# Analyze dependency tree
uv tree

# Check for unnecessary dependencies
uv pip list --format json | jq '.[] | select(.required_by == [])'

# Use extras for optional dependencies
# pyproject.toml
[project.optional-dependencies]
dev = ["pytest", "ruff"]
docs = ["sphinx", "mkdocs"]
```

## Package Health Practices

### 15. Generate and Track SBOMs

**Generate SBOM with uv:**
```bash
# Export to CycloneDX format
uv export --format requirements-txt | \
  cyclonedx-py requirements -r -i - -o sbom.json

# With syft
syft packages dir:. -o cyclonedx-json > sbom.json
```

**Track SBOMs in CI:**
```yaml
- name: Generate SBOM
  run: |
    uv export --format requirements-txt > requirements.txt
    syft packages file:requirements.txt -o cyclonedx-json=sbom.json

- name: Upload to Dependency-Track
  env:
    API_KEY: ${{ secrets.DEPENDENCY_TRACK_KEY }}
  run: |
    curl -X POST https://dtrack.company.com/api/v1/bom \
      -H "X-Api-Key: $API_KEY" \
      -F "project=my-project" \
      -F "bom=@sbom.json"
```

### 16. Consult Vulnerability Databases

**Check package health signals:**
```python
# Using pypistats
import pypistats

# Download statistics
stats = pypistats.recent("requests")
print(f"Recent downloads: {stats}")

# Using PyPI JSON API
import requests

response = requests.get("https://pypi.org/pypi/requests/json")
data = response.json()

# Check release cadence
releases = data["releases"]
print(f"Total releases: {len(releases)}")

# Check project URLs
urls = data["info"]["project_urls"]
repo = urls.get("Source")
print(f"Repository: {repo}")
```

**Query OSV database:**
```bash
# Using osv-scanner
osv-scanner --lockfile uv.lock

# Query specific package
curl -X POST https://api.osv.dev/v1/query \
  -H "Content-Type: application/json" \
  -d '{
    "package": {"name": "requests", "ecosystem": "PyPI"},
    "version": "2.31.0"
  }'
```

### 17. Verify Published Package Contents

**Inspect package before installing:**
```bash
# Download without installing
pip download --no-deps requests==2.32.3

# Unzip and inspect
unzip -l requests-2.32.3-py3-none-any.whl

# Check for suspicious files
unzip -p requests-2.32.3-py3-none-any.whl | grep -E '\.exe$|\.dll$|setup\.py'
```

**Use quarantine tools:**
```python
# Inspect in isolated environment
import zipfile
import tempfile
import os

def inspect_wheel(wheel_path):
    with tempfile.TemporaryDirectory() as tmpdir:
        with zipfile.ZipFile(wheel_path, 'r') as zip_ref:
            zip_ref.extractall(tmpdir)
        
        # List all files
        for root, dirs, files in os.walk(tmpdir):
            for file in files:
                path = os.path.join(root, file)
                print(f"File: {path}")
                
                # Check for executable permissions
                if os.access(path, os.X_OK):
                    print(f"  WARNING: Executable file")

inspect_wheel("requests-2.32.3-py3-none-any.whl")
```

## Configuration Examples

### Complete pyproject.toml with Security Hardening

```toml
[project]
name = "my-secure-app"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "ruff>=0.3.0",
    "pip-audit>=2.7.0",
]

[tool.uv]
# Dependency cooldown
exclude-newer = "7 days"

# Binary-only installations
[tool.uv.pip]
only-binary = [":all:"]

# Dependency sources
[[tool.uv.index]]
name = "pypi"
url = "https://pypi.org/simple"
default = true

[tool.ruff]
select = ["E", "F", "S"]  # Include security checks
ignore = ["S101"]  # Allow assert in tests

[tool.pytest.ini_options]
testpaths = ["tests"]
```

### Complete uv.toml for Global Configuration

```toml
# ~/.config/uv/uv.toml (macOS/Linux)
# %APPDATA%\uv\uv.toml (Windows)

# Dependency cooldown
exclude-newer = "7 days"

# Binary-only installations
[pip]
only-binary = [":all:"]

# Cache configuration
[cache]
dir = "~/.cache/uv"

# Install configuration
[install]
reinstall = false
```

## Troubleshooting

### Cooldown Blocks Required Package

**Problem:** `exclude-newer` filters out a necessary package version

**Solution:**
```bash
# Temporarily disable cooldown
uv sync --exclude-newer P0D

# Or add per-package override
[tool.uv]
exclude-newer = "7 days"
exclude-newer-package = { my-package = "1 day" }
```

### Binary Not Available for Platform

**Problem:** `--only-binary :all:` fails because no wheel exists

**Solution:**
```bash
# Allow source build for specific package
uv pip install --only-binary :all: --no-binary problematic-package requests

# Or in config
[tool.uv.pip]
only-binary = [":all:"]
no-binary = ["problematic-package"]
```

### Hash Verification Fails

**Problem:** Hash mismatch during installation

**Solution:**
```bash
# Regenerate lockfile
uv lock --upgrade-package package-name

# Or regenerate requirements
uv pip compile --generate-hashes --upgrade requirements.in

# Verify package wasn't tampered with
pip download --no-deps package-name==version
sha256sum package-name-*.whl
```

### Dependency Confusion Attack

**Problem:** Wrong package version from wrong index

**Solution:**
```toml
# Use explicit indexes
[[tool.uv.index]]
name = "internal"
url = "https://pypi.internal.com/simple"
explicit = true  # Only use when explicitly specified

# Then specify in dependencies
dependencies = [
    "internal-package @ https://pypi.internal.com/simple/internal-package",
]
```

### CI/CD Pipeline Fails After Security Hardening

**Problem:** Pipeline breaks with security settings

**Solution:**
```yaml
# Gradual rollout - start with warnings
- name: Security scan
  run: uv-secure scan || true

# Then enforce
- name: Security scan (enforced)
  run: uv-secure scan --exit-code
  
# Allow cooldown bypass for security patches
- name: Install with conditional cooldown
  run: |
    if [ "${{ github.event_name }}" == "dependabot" ]; then
      uv sync --exclude-newer P0D
    else
      uv sync --exclude-newer 7d
    fi
```

## References

- [uv Documentation](https://docs.astral.sh/uv/)
- [pip Documentation](https://pip.pypa.io/)
- [PyPI Security Advisories](https://pypi.org/security/)
- [OSV Vulnerability Database](https://osv.dev/)
- [Supply Chain Security Best Practices (SLSA)](https://slsa.dev/)
