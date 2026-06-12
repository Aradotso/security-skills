---
name: solana-security-standard
description: Use the SOL-0XX Solana security standard to scan, detect, and prevent 37 classes of Solana program vulnerabilities ($514M+ in real exploits) across AI tools, editors, CLI, and CI
triggers:
  - scan my Solana program for security issues
  - check this Anchor code against SOL-0XX rules
  - run solana security standard on my programs
  - integrate solana security checks into my project
  - what SOL-0XX rules apply to this code
  - set up solana security scanning in CI
  - validate this solana program against the security standard
  - detect solana vulnerabilities in my codebase
---

# solana-security-standard

> Skill by [ara.so](https://ara.so) — Security Skills collection.

The **Solana Security Standard** provides **37 SOL-0XX rules** distilled from $514M+ of real Solana exploits. It catches caller-controlled clock values, cross-market state asymmetry, missing Anchor constraints, wrapper/engine drift, and 33 other bug classes — firing as you code in AI tools, editors, CLI, and CI.

**SOL-001** alone covers **2 confirmed bounty wins** (caller-controlled `now_slot` in percolator). The standard works across **Claude Code, Cursor, Windsurf, Copilot, Cline, Aider**, the **VS Code extension**, the **CLI**, **Semgrep**, **GitHub Actions**, and any **MCP client**.

## Installation

### Claude Code (quickest)

```bash
mkdir -p .claude && \
  curl -sL https://raw.githubusercontent.com/Copenhagen0x/solana-security-standard/main/plugin-guidance.md \
       -o .claude/claude-security-guidance.md && \
  curl -sL https://raw.githubusercontent.com/Copenhagen0x/solana-security-standard/main/security-patterns.yaml \
       -o .claude/security-patterns.yaml
```

Then install the security-guidance plugin:

```text
/plugin install security-guidance@claude-plugins-official
/reload-plugins
```

Or install the full MCP-powered plugin:

```text
/plugin marketplace add Copenhagen0x/solana-security-standard
/plugin install solana-security-standard@solana-security-standard
```

### CLI (any machine)

```bash
npx @jelleo/solana-security-standard scan ./programs
```

No install needed — runs via `npx`. Zero dependencies.

### CI — GitHub Action

```yaml
# .github/workflows/solana-security.yml
name: Solana Security Standard
on: [pull_request]
permissions:
  contents: read
  security-events: write
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Copenhagen0x/solana-security-standard@v1
        with:
          paths: ./programs
          fail-on-findings: true
          upload-sarif: true
```

### VS Code / Cursor / Windsurf

Install from [Open VSX](https://open-vsx.org/extension/jelleo/solana-security-standard):

- **Cursor / Windsurf / VSCodium**: Search "Solana Security Standard" in Extensions
- **VS Code**: Sideload `.vsix` from `extensions/vscode/`

Shows inline warnings as you type in Rust and TypeScript/JS files.

### MCP (any MCP client)

Add to your MCP settings (`.cline/mcp.json`, `.cursor/mcp.json`, etc.):

```json
{
  "mcpServers": {
    "solana-security-standard": {
      "command": "npx",
      "args": ["-y", "@jelleo/solana-security-mcp"]
    }
  }
}
```

Exposes `scan_solana_code` tool + `list_solana_security_rules` resource.

### AI Agents (Codex, Copilot, Cursor, Windsurf, Cline, Aider)

Copy the rules file for your tool:

| Tool | Copy into repo |
|------|----------------|
| Codex / AGENTS.md | `integrations/codex/AGENTS.md` |
| GitHub Copilot | `integrations/copilot/.github/copilot-instructions.md` |
| Cursor | `integrations/cursor/.cursor/` |
| Windsurf | `integrations/windsurf/.windsurf/` |
| Cline | `integrations/cline/.clinerules` |
| Aider | `integrations/aider/` |

```bash
# Example for Cursor:
curl -sL https://raw.githubusercontent.com/Copenhagen0x/solana-security-standard/main/integrations/cursor/.cursor/rules -o .cursor/rules
```

### Semgrep

```bash
semgrep --config https://raw.githubusercontent.com/Copenhagen0x/solana-security-standard/main/semgrep/solana-security-standard.yaml ./programs
```

## CLI Commands

### Scan programs

```bash
# Scan a directory
npx @jelleo/solana-security-standard scan ./programs

# Scan specific file
npx @jelleo/solana-security-standard scan ./programs/src/lib.rs

# JSON output (for CI)
npx @jelleo/solana-security-standard scan ./programs --format json

# SARIF output (GitHub Code Scanning)
npx @jelleo/solana-security-standard scan ./programs --format sarif > results.sarif

# Filter by tier (drop low-severity hygiene findings)
npx @jelleo/solana-security-standard scan ./programs --min-tier high

# Baseline mode (gate only on NEW findings)
npx @jelleo/solana-security-standard scan ./programs --baseline .sss-baseline.json
```

### Generate baseline

```bash
# Create baseline of current findings
npx @jelleo/solana-security-standard scan ./programs --format json > .sss-baseline.json

# Future scans compare against it
npx @jelleo/solana-security-standard scan ./programs --baseline .sss-baseline.json
```

CLI exits **non-zero** on findings (gates CI).

## Key Rules (37 total)

### Critical On-Chain Rules

| Rule | Bug Class | Example |
|------|-----------|---------|
| **SOL-001** | Unauthenticated `now_slot` / clock spoofing | 2 confirmed bounty wins |
| **SOL-002** | Cross-market state asymmetry | Counter inflation |
| **SOL-003** | Wrapper re-implements engine logic | Drift from canonical |
| **SOL-004** | Missing signer check | Unauthorized access |
| **SOL-005** | Missing owner check | Wrong program control |
| **SOL-006** | Account data type confusion | Misinterpreted structs |
| **SOL-007** | Uninitialized account | Reading zeroed data |
| **SOL-008** | Arithmetic overflow/underflow | Balance manipulation |
| **SOL-009** | PDA derivation mismatch | Wrong seeds → wrong account |
| **SOL-010** | Missing rent exemption | Account closure drain |

### Integrator / Client-Side Rules

| Rule | Bug Class |
|------|-----------|
| **SOL-029** | Client-side arithmetic overflow (TypeScript/JS) |
| **SOL-030** | Missing transaction simulation |
| **SOL-031** | Hardcoded program IDs in client |

Full list: https://github.com/Copenhagen0x/solana-security-standard/blob/main/claude-security-guidance.md

## Real Code Examples

### SOL-001: Unauthenticated `now_slot` (2 bounty wins)

**Vulnerable** (percolator-prog ACTIVATE):

```rust
pub fn activate(ctx: Context<Activate>, now_slot: u64) -> Result<()> {
    let state = &mut ctx.accounts.state;
    let clock = Clock::get()?;
    
    // Caller controls now_slot — can bypass time checks
    if now_slot < state.cooldown_end {
        return Err(ErrorCode::CooldownActive.into());
    }
    
    state.active = true;
    Ok(())
}
```

**Fixed**:

```rust
pub fn activate(ctx: Context<Activate>) -> Result<()> {
    let state = &mut ctx.accounts.state;
    let clock = Clock::get()?;
    
    // Use authenticated clock.slot
    if clock.slot < state.cooldown_end {
        return Err(ErrorCode::CooldownActive.into());
    }
    
    state.active = true;
    Ok(())
}
```

### SOL-004: Missing signer check

**Vulnerable**:

```rust
#[derive(Accounts)]
pub struct Withdraw<'info> {
    #[account(mut)]
    pub authority: AccountInfo<'info>, // Not checked as signer!
    #[account(mut)]
    pub vault: Account<'info, Vault>,
}

pub fn withdraw(ctx: Context<Withdraw>, amount: u64) -> Result<()> {
    // Anyone can pass any authority account
    transfer_from_vault(&ctx.accounts.vault, amount)?;
    Ok(())
}
```

**Fixed**:

```rust
#[derive(Accounts)]
pub struct Withdraw<'info> {
    #[account(mut)]
    pub authority: Signer<'info>, // Enforces signature
    #[account(
        mut,
        has_one = authority // Links vault to this authority
    )]
    pub vault: Account<'info, Vault>,
}
```

### SOL-009: PDA derivation mismatch

**Vulnerable**:

```rust
// Instruction handler uses wrong seeds
let (vault_pda, _bump) = Pubkey::find_program_address(
    &[b"vault", user.key().as_ref()], // Missing market_id seed!
    program_id
);

if vault_pda != *vault.key {
    return Err(ErrorCode::InvalidVault.into());
}
```

**Fixed**:

```rust
// Match the seeds used in account derivation
let (vault_pda, _bump) = Pubkey::find_program_address(
    &[b"vault", market_id.as_ref(), user.key().as_ref()],
    program_id
);

if vault_pda != *vault.key {
    return Err(ErrorCode::InvalidVault.into());
}
```

### SOL-029: Client-side arithmetic overflow (TypeScript)

**Vulnerable**:

```typescript
// JavaScript numbers lose precision beyond 2^53
const depositAmount = userBalance + newDeposit;
const tx = await program.methods
  .deposit(new BN(depositAmount)) // Precision loss!
  .accounts({ vault, user })
  .rpc();
```

**Fixed**:

```typescript
import { BN } from "@coral-xyz/anchor";

// Use BN for all token math
const depositAmount = new BN(userBalance).add(new BN(newDeposit));
const tx = await program.methods
  .deposit(depositAmount)
  .accounts({ vault, user })
  .rpc();
```

## Configuration

### GitHub Action Options

```yaml
- uses: Copenhagen0x/solana-security-standard@v1
  with:
    paths: ./programs              # Scan target (default: .)
    fail-on-findings: true         # Exit 1 on findings (default: true)
    upload-sarif: true             # GitHub Code Scanning (default: true)
    min-tier: high                 # Filter: critical, high, medium, low
    baseline: .sss-baseline.json   # Gate only on NEW findings
```

### VS Code Extension Settings

```json
{
  "solanaSecurityStandard.minTier": "medium",
  "solanaSecurityStandard.excludePatterns": [
    "**/tests/**",
    "**/target/**"
  ]
}
```

### Semgrep Integration

```bash
# Custom config location
semgrep --config ./my-sss-rules.yaml ./programs

# Combine with other rulesets
semgrep --config auto --config ./semgrep/solana-security-standard.yaml ./programs
```

## Common Patterns

### Pre-commit Hook

```bash
# .git/hooks/pre-commit
#!/bin/bash
npx @jelleo/solana-security-standard scan programs/ --min-tier high
if [ $? -ne 0 ]; then
  echo "❌ Solana security scan failed — fix findings or commit with --no-verify"
  exit 1
fi
```

```bash
chmod +x .git/hooks/pre-commit
```

### Baseline Workflow (gate on regressions only)

```bash
# 1. Generate baseline from current state
npx @jelleo/solana-security-standard scan ./programs --format json > .sss-baseline.json

# 2. Commit it
git add .sss-baseline.json
git commit -m "chore: add security baseline"

# 3. CI uses --baseline mode
npx @jelleo/solana-security-standard scan ./programs --baseline .sss-baseline.json
```

Only **new** findings fail the build.

### Multi-repo Monorepo Scan

```bash
# Scan multiple program directories
npx @jelleo/solana-security-standard scan \
  ./packages/protocol/programs \
  ./packages/staking/programs \
  ./packages/governance/programs
```

### Filter by Rule

CLI doesn't support per-rule filtering, but you can grep JSON output:

```bash
npx @jelleo/solana-security-standard scan ./programs --format json \
  | jq '.findings[] | select(.rule == "SOL-001")'
```

## MCP Tools

When installed as MCP server:

### `scan_solana_code`

```typescript
// Agent calls this automatically when you say:
// "scan my Solana program for security issues"

{
  "code": "pub fn transfer(ctx: Context<Transfer>, amount: u64) -> Result<()> { ... }",
  "language": "rust"
}

// Returns findings with rule, severity, message, line numbers
```

### `list_solana_security_rules`

```typescript
// Agent calls when you ask:
// "what SOL-0XX rules apply to signer checks?"

// Returns all 37 rules with descriptions, tiers, examples
```

## Troubleshooting

### "Module not found" in CLI

```bash
# Clear npx cache and retry
rm -rf ~/.npm/_npx
npx @jelleo/solana-security-standard scan ./programs
```

### VS Code extension not activating

1. Check output panel: View → Output → "Solana Security Standard"
2. Verify file is `.rs`, `.ts`, or `.js`
3. Reload window: Cmd/Ctrl+Shift+P → "Reload Window"

### False positives

```rust
// SOL-004 flags this as missing signer check:
pub authority: AccountInfo<'info>,

// If it's intentional (e.g., read-only query), add comment:
pub authority: AccountInfo<'info>, // READ-ONLY: no sig required

// Or use Anchor's Signer<'info> to suppress:
pub authority: UncheckedAccount<'info>, // Explicitly unchecked
```

### GitHub Action fails with SARIF error

```yaml
# Ensure security-events: write permission:
permissions:
  contents: read
  security-events: write
```

### Baseline not filtering findings

Baseline only gates **new** findings. If you want to suppress **known** findings:

```bash
# Regenerate baseline after fixing some issues
npx @jelleo/solana-security-standard scan ./programs --format json > .sss-baseline.json
```

## Real Exploit Mappings

Every rule links to real disclosed exploits:

- **SOL-001**: Wormhole ($325M), percolator bounties
- **SOL-008**: Mango Markets ($114M)
- **SOL-010**: Cashio ($52M)
- **SOL-017**: Crema Finance ($8.8M)

See [`hacks/README.md`](https://github.com/Copenhagen0x/solana-security-standard/blob/main/hacks/README.md) for full database.

## Badge for Your README

Show adoption:

```markdown
[![Solana Security Standard](https://img.shields.io/badge/Solana%20Security%20Standard-SOL--0XX-a855f7?labelColor=6d28d9)](https://github.com/Copenhagen0x/solana-security-standard)
```

## Resources

- Full rule catalog: https://github.com/Copenhagen0x/solana-security-standard/blob/main/claude-security-guidance.md
- Hacks database ($514M+): https://github.com/Copenhagen0x/solana-security-standard/blob/main/hacks/README.md
- Per-rule explainers: https://github.com/Copenhagen0x/solana-security-standard/tree/main/content
- Benchmark (rule accuracy): https://github.com/Copenhagen0x/solana-security-standard/blob/main/BENCHMARK.md
