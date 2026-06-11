---
name: microsoft-security-skills-plugin
description: Install and use curated Microsoft Security skills for AI agents covering Defender, Sentinel, Entra, Purview, Intune, and Security Copilot workflows
triggers:
  - "install Microsoft Security skills for my AI agent"
  - "how do I add security expertise to my coding assistant"
  - "set up Microsoft security skills plugin"
  - "which Microsoft security skill should I use for"
  - "configure security skills for Copilot"
  - "add Defender and Sentinel skills to my agent"
  - "install APM security skills plugin"
  - "how to use security skills with Claude Code"
---

# microsoft-security-skills-plugin

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## What this project does

The Microsoft Security Skills Plugin is a curated collection of 56 security expertise modules that teach AI coding agents how to provide accurate, opinionated Microsoft Security guidance. Instead of generic security advice, agents gain decision trees, workflows, and guardrails grounded in Microsoft Learn documentation across:

- **Threat protection & SecOps**: Defender XDR, Sentinel, Security Copilot, unified SecOps platform
- **Identity & access**: Entra ID, Conditional Access, PIM, ID Protection, Permissions Management
- **Compliance & data protection**: Purview DLP, eDiscovery, audit, data classification, Priva
- **Endpoint & device**: Intune device management, app protection, BitLocker, PAW design
- **Cloud platform security**: Azure Policy, Key Vault, network security, App Service, API security

Each skill packages Microsoft Learn knowledge into actionable guidance that compatible agents (GitHub Copilot, Claude Code, Cursor, Codex CLI, Gemini CLI) can apply during conversations.

## Installation

### Prerequisites

Verify you have Git and Node.js 18+ installed:

```bash
git --version
npx --version
```

### APM (recommended - all hosts at once)

Install across GitHub Copilot, Claude Code, Cursor, OpenCode, Codex, and Gemini with one command:

```bash
apm install vinayaklatthe/microsoft-security-skills
```

### Universal install (manual clone)

Clone the repository and configure your agent to read from the `skills/` directory:

```bash
git clone https://github.com/vinayaklatthe/microsoft-security-skills.git
cd microsoft-security-skills
```

Then point your agent configuration to the `skills/` directory.

### Host-specific install with npx skills

Install globally for a specific agent host:

```bash
# GitHub Copilot (VS Code, Copilot CLI)
npx skills add https://github.com/vinayaklatthe/microsoft-security-skills/tree/main/skills -a github-copilot -g -y

# Claude Code
npx skills add https://github.com/vinayaklatthe/microsoft-security-skills/tree/main/skills -a claude -g -y

# Cursor
npx skills add https://github.com/vinayaklatthe/microsoft-security-skills/tree/main/skills -a cursor -g -y

# Codex CLI
npx skills add https://github.com/vinayaklatthe/microsoft-security-skills/tree/main/skills -a codex -g -y
```

### Gemini CLI

```bash
gemini extensions install https://github.com/vinayaklatthe/microsoft-security-skills
```

## Verifying installation

After installation, test the agent's access to skills with three quick prompts:

### 1. Test security skills

```
What Microsoft Defender controls should I prioritise for a new Microsoft 365 tenant?
```

Expected: Structured, product-specific guidance with Microsoft Learn references.

### 2. Test identity skills

```
How do I design a Conditional Access policy baseline for a mid-size organisation?
```

Expected: Policy framework with named Conditional Access templates and guardrails.

### 3. Test compliance skills

```
What Purview DLP policies should I configure to protect sensitive data in Microsoft 365?
```

Expected: Scoped DLP guidance with workload-specific recommendations.

## How to use skills in conversations

Once installed, the agent automatically scans skill front matter (`name`, `description`, `WHEN:` triggers) during conversations. Simply ask natural questions:

```
Design a Sentinel workspace for my SOC
```

```
Help me build a Conditional Access policy for contractors
```

```
What Defender for Cloud hardening steps should I take first?
```

```
Review my Intune device compliance policy for security gaps
```

The agent loads the most relevant skill (e.g., `sentinel.md`, `conditional-access-mfa.md`, `defender-for-cloud-hardening.md`, `intune-device-mgmt.md`) and follows its decision trees and guardrails.

## Skill selection reference

Use this table to choose the right skill before asking:

| Use case | Skill to invoke |
|---|---|
| Multi-product incident (endpoint + identity + email) | `defender-xdr` |
| SIEM setup, KQL detections, log ingestion | `sentinel` |
| Merge Sentinel and Defender XDR in one portal | `unified-secops-platform` |
| AI-assisted incident investigation | `security-copilot` |
| Automate triage with AI agents | `security-copilot-agents` |
| Active breach or ransomware response | `compromise-recovery` |
| Identity and access management (users, SSO, hybrid) | `entra-id` |
| MFA and Conditional Access policies | `conditional-access-mfa` |
| Risky user detection, leaked credentials | `entra-id-protection` |
| Just-in-time admin access (JIT) | `azure-pim` |
| Identity lifecycle, access packages | `entra-id-governance` |
| Multicloud permissions (AWS, GCP, Azure) | `entra-permissions-management` |
| Endpoint EDR, attack surface reduction | `defender-for-endpoint` |
| Identity attacks on Active Directory | `defender-for-identity` |
| Email phishing and BEC protection | `defender-for-office-365` |
| Cloud infrastructure hardening | `defender-for-cloud-hardening` |
| SaaS app posture (M365, Salesforce) | `cloud-app-security-posture` |
| Intune device compliance and config | `intune-device-mgmt` |
| Data loss prevention (Exchange, SharePoint, Teams) | `purview-dlp-policy` |
| Sensitive data discovery and classification | `purview-data-classification` |
| Legal/HR eDiscovery | `purview-ediscovery` |
| AI prompt data monitoring | `purview-dspm-ai` |
| Fix oversharing before Copilot rollout | `purview-copilot-oversharing` |
| Insider data theft detection | `insider-risk-baseline` |
| Zero Trust architecture design | `security-architecture` |
| STRIDE threat modelling | `threat-modelling` |
| Azure network security (hub-spoke, NSG) | `azure-network-security-design` |
| Secrets, keys, certificates management | `azure-key-vault` |
| Azure governance guardrails | `azure-policy` |
| API security (OWASP API Top 10) | `api-security-design` |

## Repository structure

```
microsoft-security-skills/
├── skills/                     # 56 skill definitions
│   ├── defender-xdr/
│   │   └── SKILL.md
│   ├── sentinel/
│   │   └── SKILL.md
│   ├── entra-id/
│   │   └── SKILL.md
│   ├── conditional-access-mfa/
│   │   └── SKILL.md
│   ├── purview-dlp-policy/
│   │   └── SKILL.md
│   ├── intune-device-mgmt/
│   │   └── SKILL.md
│   └── ...
├── plugin.json                 # Plugin metadata
├── validation/                 # Validation harness
└── README.md
```

## Example: Using a skill in code generation

### Scenario: Generate a Conditional Access policy in JavaScript

Ask your agent:

```
Generate a Conditional Access policy JSON for blocking legacy authentication across all cloud apps
```

The agent loads `conditional-access-mfa` skill and produces:

```javascript
// Conditional Access policy: Block legacy authentication
// Reference: https://learn.microsoft.com/entra/identity/conditional-access/

const blockLegacyAuthPolicy = {
  displayName: "Block Legacy Authentication - All Cloud Apps",
  state: "enabledForReportingButNotEnforced", // Start in report-only mode
  conditions: {
    users: {
      includeUsers: ["All"]
    },
    applications: {
      includeApplications: ["All"]
    },
    clientAppTypes: [
      "exchangeActiveSync",
      "other" // Covers POP, IMAP, SMTP AUTH, older Office clients
    ]
  },
  grantControls: {
    operator: "OR",
    builtInControls: ["block"]
  }
};

// Deploy with Microsoft Graph API
const deployPolicy = async (accessToken) => {
  const response = await fetch(
    "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies",
    {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${accessToken}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify(blockLegacyAuthPolicy)
    }
  );
  
  if (!response.ok) {
    throw new Error(`Policy deployment failed: ${response.statusText}`);
  }
  
  return await response.json();
};

// Usage
// 1. Start in report-only mode (state: "enabledForReportingButNotEnforced")
// 2. Monitor sign-in logs for 2-4 weeks
// 3. Remediate users still using legacy protocols
// 4. Change state to "enabled" after validation
```

### Scenario: Query Sentinel with KQL for identity threats

Ask your agent:

```
Write a KQL query to detect potential Entra ID token theft from Sentinel logs
```

The agent loads `sentinel` skill and produces:

```javascript
// Sentinel KQL: Detect potential Entra ID token theft
// Trigger: Multiple sign-ins from different geolocations within 15 minutes
// Reference: https://learn.microsoft.com/azure/sentinel/

const tokenTheftQuery = `
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == "0" // Successful sign-ins only
| extend GeoLocation = strcat(Location.city, ", ", Location.countryOrRegion)
| summarize 
    Locations = make_set(GeoLocation),
    LocationCount = dcount(GeoLocation),
    IPAddresses = make_set(IPAddress)
  by UserPrincipalName, bin(TimeGenerated, 15m)
| where LocationCount > 1 // Multiple locations in same 15-min window
| project 
    TimeGenerated, 
    UserPrincipalName, 
    LocationCount, 
    Locations, 
    IPAddresses
| order by TimeGenerated desc
`;

// Export as Sentinel analytic rule (JavaScript SDK example)
const { SecretClient } = require("@azure/keyvault-secrets");
const { DefaultAzureCredential } = require("@azure/identity");

const deployAnalyticRule = async () => {
  const credential = new DefaultAzureCredential();
  const subscriptionId = process.env.AZURE_SUBSCRIPTION_ID;
  const resourceGroup = process.env.AZURE_RESOURCE_GROUP;
  const workspaceName = process.env.SENTINEL_WORKSPACE_NAME;
  
  const rule = {
    displayName: "Potential Entra ID Token Theft - Impossible Travel",
    description: "Detects successful sign-ins from multiple geolocations within 15 minutes, indicating potential token theft or session hijacking",
    severity: "High",
    enabled: true,
    query: tokenTheftQuery,
    queryFrequency: "PT1H", // Run every hour
    queryPeriod: "P1D", // Look back 1 day
    triggerOperator: "GreaterThan",
    triggerThreshold: 0,
    suppressionDuration: "PT1H",
    suppressionEnabled: false,
    tactics: ["CredentialAccess", "InitialAccess"],
    techniques: ["T1528", "T1539"] // MITRE ATT&CK
  };
  
  const url = `https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroup}/providers/Microsoft.OperationalInsights/workspaces/${workspaceName}/providers/Microsoft.SecurityInsights/alertRules/token-theft-rule-001?api-version=2023-02-01`;
  
  const accessToken = await credential.getToken("https://management.azure.com/.default");
  
  const response = await fetch(url, {
    method: "PUT",
    headers: {
      "Authorization": `Bearer ${accessToken.token}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      kind: "Scheduled",
      properties: rule
    })
  });
  
  if (!response.ok) {
    throw new Error(`Rule deployment failed: ${response.statusText}`);
  }
  
  return await response.json();
};

// Deploy the rule
deployAnalyticRule().catch(console.error);
```

## Configuration

### Multi-host compatibility

The plugin works across:

- **GitHub Copilot** (VS Code, Copilot CLI)
- **Claude Code**
- **Cursor**
- **Codex CLI**
- **Gemini CLI**
- **OpenCode**

No per-host configuration required after installation.

### Custom skill directories

If you clone the repo, point your agent to the `skills/` directory:

```bash
# Example: VS Code settings.json for GitHub Copilot
{
  "github.copilot.advanced": {
    "customSkillsPath": "/path/to/microsoft-security-skills/skills"
  }
}
```

### Environment variables for examples

When running code examples that interact with Microsoft APIs:

```bash
export AZURE_TENANT_ID="your-tenant-id"
export AZURE_CLIENT_ID="your-client-id"
export AZURE_CLIENT_SECRET="your-client-secret"
export AZURE_SUBSCRIPTION_ID="your-subscription-id"
export AZURE_RESOURCE_GROUP="your-resource-group"
export SENTINEL_WORKSPACE_NAME="your-sentinel-workspace"
```

Use Azure Key Vault for production secrets:

```javascript
const { SecretClient } = require("@azure/keyvault-secrets");
const { DefaultAzureCredential } = require("@azure/identity");

const credential = new DefaultAzureCredential();
const vaultUrl = `https://${process.env.KEY_VAULT_NAME}.vault.azure.net`;
const client = new SecretClient(vaultUrl, credential);

const secret = await client.getSecret("client-secret");
```

## Common patterns

### Pattern 1: Skill-guided policy creation

```javascript
// 1. Ask agent: "Design a Purview DLP policy for financial data"
// 2. Agent loads purview-dlp-policy skill
// 3. Agent returns policy JSON with guardrails

const dlpPolicy = {
  name: "Block External Sharing - Financial Data",
  locations: [
    { location: "ExchangeOnline" },
    { location: "SharePointOnline" },
    { location: "OneDriveForBusiness" },
    { location: "TeamsChat" }
  ],
  rules: [
    {
      name: "Detect Financial Information",
      conditions: {
        contentContainsSensitiveInformation: [
          { name: "Credit Card Number", minCount: 1 },
          { name: "U.S. Bank Account Number", minCount: 1 },
          { name: "International Banking Account Number (IBAN)", minCount: 1 }
        ]
      },
      actions: {
        blockAccess: true,
        notifyUser: true,
        incidentReport: true
      }
    }
  ]
};

// Deploy with Purview API or PowerShell
```

### Pattern 2: Security assessment automation

```javascript
// Ask: "What Defender for Cloud recommendations should I prioritize?"
// Agent loads defender-for-cloud-hardening skill

const assessSecureScore = async (credential, subscriptionId) => {
  const url = `https://management.azure.com/subscriptions/${subscriptionId}/providers/Microsoft.Security/secureScores/ascScore?api-version=2020-01-01`;
  
  const token = await credential.getToken("https://management.azure.com/.default");
  const response = await fetch(url, {
    headers: { "Authorization": `Bearer ${token.token}` }
  });
  
  const data = await response.json();
  
  // Prioritize by score impact
  const recommendations = data.properties.recommendations
    .filter(r => r.weight > 0)
    .sort((a, b) => b.weight - a.weight)
    .slice(0, 10);
  
  return recommendations.map(r => ({
    name: r.displayName,
    severity: r.severity,
    scoreImpact: r.weight,
    remediationSteps: r.remediationDescription
  }));
};
```

### Pattern 3: Incident investigation workflow

```javascript
// Ask: "Help me investigate a Defender XDR incident"
// Agent loads defender-xdr skill

const investigateIncident = async (incidentId, accessToken) => {
  // 1. Get incident details
  const incident = await fetch(
    `https://graph.microsoft.com/v1.0/security/incidents/${incidentId}`,
    { headers: { "Authorization": `Bearer ${accessToken}` } }
  ).then(r => r.json());
  
  // 2. Get alerts
  const alerts = incident.alerts.map(a => ({
    title: a.title,
    severity: a.severity,
    category: a.category,
    affectedAssets: a.assets
  }));
  
  // 3. Pivot to related entities
  const users = [...new Set(alerts.flatMap(a => 
    a.affectedAssets.filter(asset => asset.type === 'User')
      .map(asset => asset.identifier)
  ))];
  
  const devices = [...new Set(alerts.flatMap(a => 
    a.affectedAssets.filter(asset => asset.type === 'Device')
      .map(asset => asset.identifier)
  ))];
  
  return {
    incident: {
      id: incident.id,
      severity: incident.severity,
      status: incident.status
    },
    alerts,
    affectedUsers: users,
    affectedDevices: devices,
    nextSteps: [
      "Review alert timelines in Defender portal",
      "Check for lateral movement indicators",
      "Validate user sign-in logs in Entra ID",
      "Isolate affected devices if needed"
    ]
  };
};
```

## Troubleshooting

### Skills not loading in agent

**Symptom**: Agent gives generic advice instead of Microsoft-specific guidance.

**Fix**:
1. Verify installation: `ls skills/` should show skill directories
2. Restart your agent/IDE
3. Test with explicit skill invocation: "Use the defender-xdr skill to help me..."

### APM install fails

**Symptom**: `apm install` returns error.

**Fix**:
```bash
# Install APM first
npm install -g @microsoft/apm

# Verify APM is in PATH
apm --version

# Retry install
apm install vinayaklatthe/microsoft-security-skills
```

### Skill outputs outdated recommendations

**Symptom**: Agent references deprecated Microsoft Learn pages.

**Fix**:
```bash
# Update the plugin
cd microsoft-security-skills
git pull origin main

# Or reinstall with APM
apm update vinayaklatthe/microsoft-security-skills
```

### Code examples fail with authentication errors

**Symptom**: `Unauthorized` or `Forbidden` when running generated code.

**Fix**:
1. Verify Azure credentials:
   ```bash
   az login
   az account show
   ```

2. Verify app registration permissions in Azure portal:
   - Microsoft Graph API: `Policy.Read.All`, `SecurityEvents.Read.All`, etc.
   - Grant admin consent for tenant-wide permissions

3. Use Managed Identity for Azure-hosted code:
   ```javascript
   const { DefaultAzureCredential } = require("@azure/identity");
   const credential = new DefaultAzureCredential();
   ```

### Which skill to use for overlapping scenarios

**Symptom**: Uncertain which skill to invoke (e.g., SOC spans SIEM + XDR).

**Strategy**:
1. Start with the most specific skill (e.g., `sentinel` for KQL queries)
2. Follow cross-references in skill body (e.g., "See also: `defender-xdr`")
3. Ask agent: "Which security skill should I use for [scenario]?"

## Validation

The plugin includes a zero-dependency validation harness in `validation/` that measures the lift each skill adds over an unaided model. Validated across:

- GPT-4 Turbo (OpenAI)
- Claude 3.5 Sonnet (Anthropic)

Run validation:

```bash
cd validation
node validate.js
```

## References

- **Microsoft Learn Security**: https://learn.microsoft.com/security/
- **APM (Agent Plugin Manager)**: https://github.com/microsoft/apm
- **Plugin repository**: https://github.com/vinayaklatthe/microsoft-security-skills
- **License**: MIT
