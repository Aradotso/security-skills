---
name: azure-security-lab-terraform
description: Deploy and configure the Azure Security Lab - a hands-on Terraform environment with hub-spoke networking, Azure Firewall, NSGs, Key Vault, and security monitoring for learning Azure security controls.
triggers:
  - deploy the Azure security lab
  - set up Azure Firewall hub-spoke network
  - configure Azure security lab with Terraform
  - test Azure network segmentation with firewall
  - deploy Azure Bastion and security controls
  - create Azure security testing environment
  - validate Azure Firewall rules and logging
  - troubleshoot Azure security lab deployment
---

# Azure Security Lab Terraform Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill enables AI agents to deploy, configure, and troubleshoot the Azure Security Lab - a compact Terraform project that creates a hub-spoke network architecture with Azure Firewall Basic, NSGs, route tables, Key Vault, Log Analytics, and Azure Policy for hands-on Azure security learning.

## What This Project Does

The Azure Security Lab deploys a focused security testing environment:

- **Hub-spoke topology**: Hub VNet (10.20.0.0/16) with Azure Firewall, test spoke (10.23.0.0/16) with untrusted client, protected spoke (10.22.0.0/16) with private workload
- **Network security**: Azure Firewall Basic for east-west traffic inspection, NSGs for subnet-level controls, route tables forcing traffic through firewall
- **Compute**: Windows test VM with optional restricted RDP, Windows IIS web VM without public IP
- **Security features**: Key Vault for secrets, Log Analytics for diagnostics, Azure Policy for posture auditing, optional Bastion for private access
- **Cost awareness**: Main cost is Azure Firewall Basic (~0.40 USD/hour), designed for short-term learning deployments

## Installation & Prerequisites

### Requirements

- Terraform >= 1.9.0
- Azure CLI
- Azure subscription with Contributor rights
- Optional: Entra ID permissions for test user creation

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/Jamonygr/Azure-Security-Lab.git
cd Azure-Security-Lab

# Login to Azure
az login
az account set --subscription "<subscription-id>"

# Initialize Terraform
terraform init
```

### Configuration Files

The project uses two main configuration approaches:

1. **terraform.tfvars** - Root-level variables
2. **environments/lab.tfvars** - Environment-specific overrides

```bash
# Create your configuration from example
cp terraform.tfvars.example terraform.tfvars
```

## Key Configuration Variables

### Essential Variables

```hcl
# terraform.tfvars

# Your public IP for RDP access (REQUIRED)
admin_source_ip_cidr = "203.0.113.42/32"

# Core deployment toggles
deploy_firewall                = true
firewall_sku_tier              = "Basic"
force_spoke_egress_to_firewall = false

# Access method
enable_test_client_public_ip = true
deploy_bastion               = false

# Optional features
deploy_keyvault      = true
deploy_log_analytics = true
deploy_policy        = true
deploy_budget        = true

# Advanced features (disabled by default)
deploy_entra_test_users   = false
deploy_conditional_access = false
deploy_sentinel           = false

# Resource naming
environment = "lab"
location    = "westeurope"
prefix      = "azseclab"
```

### Network Architecture

```hcl
# Address space configuration
hub_vnet_address_space      = ["10.20.0.0/16"]
test_vnet_address_space     = ["10.23.0.0/16"]
protected_vnet_address_space = ["10.22.0.0/16"]

# Subnet assignments
firewall_subnet_prefix           = "10.20.1.0/24"
firewall_mgmt_subnet_prefix      = "10.20.3.0/24"
bastion_subnet_prefix            = "10.20.2.0/26"
test_client_subnet_prefix        = "10.23.1.0/24"
protected_web_subnet_prefix      = "10.22.1.0/24"
```

## Core Terraform Commands

### Standard Deployment Workflow

```bash
# Plan deployment (review changes)
terraform plan -var-file="terraform.tfvars"

# Apply with confirmation
terraform apply -var-file="terraform.tfvars"

# Apply without confirmation (CI/CD)
terraform apply -var-file="terraform.tfvars" -auto-approve

# Destroy all resources
terraform destroy -var-file="terraform.tfvars"
```

### Using Environment Files

```bash
# Deploy with environment-specific config
terraform apply -var-file="environments/lab.tfvars"

# Plan specific environment
terraform plan -var-file="environments/prod.tfvars"
```

### Targeted Operations

```bash
# Deploy only firewall module
terraform apply -target=module.firewall

# Destroy only test spoke
terraform destroy -target=module.test_spoke

# Refresh state without changes
terraform refresh
```

## Retrieving Lab Access Information

### Get Outputs

```bash
# Show all outputs
terraform output

# Get specific output
terraform output test_client
terraform output protected_web

# Get sensitive password (raw format)
terraform output -raw windows_admin_password

# Get firewall IPs (when deployed)
terraform output firewall_private_ip
terraform output firewall_public_ip
```

### Output Structure

```hcl
# Example outputs
test_client = {
  private_ip = "10.23.1.4"
  public_ip  = "20.73.45.67"  # when enable_test_client_public_ip = true
  rdp_command = "mstsc /v:20.73.45.67"
}

protected_web = {
  private_ip = "10.22.1.4"
  test_url   = "http://10.22.1.4"
}

windows_admin_username = "azureadmin"
windows_admin_password = <sensitive>
```

## Common Deployment Patterns

### Pattern 1: Default Security Lab (Firewall + Public RDP)

```hcl
# terraform.tfvars
admin_source_ip_cidr         = "YOUR_IP/32"
deploy_firewall              = true
firewall_sku_tier            = "Basic"
enable_test_client_public_ip = true
deploy_bastion               = false
deploy_keyvault              = true
deploy_log_analytics         = true
deploy_policy                = true
```

```bash
terraform apply -var-file="terraform.tfvars"

# Get RDP access
terraform output test_client

# RDP to test client from your IP
mstsc /v:<test_client_public_ip>
```

### Pattern 2: Private Access with Bastion

```hcl
# terraform.tfvars
admin_source_ip_cidr         = "0.0.0.0/0"  # Not used with Bastion
deploy_firewall              = true
enable_test_client_public_ip = false
deploy_bastion               = true
```

```bash
terraform apply -var-file="terraform.tfvars"

# Access via Azure Portal Bastion
# Navigate to VM → Connect → Bastion
```

### Pattern 3: Minimal Lab Without Firewall

```hcl
# terraform.tfvars
admin_source_ip_cidr         = "YOUR_IP/32"
deploy_firewall              = false
enable_test_client_public_ip = true
deploy_bastion               = false
```

```bash
# Lower cost, no firewall inspection
terraform apply -var-file="terraform.tfvars"

# Test direct spoke-to-spoke connectivity
# NSGs still enforce security
```

### Pattern 4: Identity Lab with Entra Users

```hcl
# terraform.tfvars
deploy_entra_test_users = true
entra_domain_name       = "contoso.onmicrosoft.com"

# Requires these permissions:
# - User.ReadWrite.All
# - Group.ReadWrite.All
```

```bash
# Grant permissions in Azure Portal first
az ad app permission grant --id <app-id> --api 00000003-0000-0000-c000-000000000000

terraform apply -var-file="terraform.tfvars"

# Created users:
# - sec-admin@contoso.onmicrosoft.com
# - sec-analyst@contoso.onmicrosoft.com
# - lab-user@contoso.onmicrosoft.com
# - break-glass@contoso.onmicrosoft.com
```

### Pattern 5: SOC Lab with Sentinel

```hcl
# terraform.tfvars
deploy_sentinel      = true
deploy_log_analytics = true
deploy_firewall      = true
```

```bash
terraform apply -var-file="terraform.tfvars"

# Query firewall logs in Log Analytics
# Navigate to workspace → Logs
```

## Testing Lab Scenarios

### Scenario 1: Verify Firewall East-West Routing

```bash
# 1. RDP to test client
mstsc /v:<test_client_public_ip>

# 2. Inside test client PowerShell:
$protectedIP = "10.22.1.4"
$firewallIP = "10.20.1.4"

# Test HTTP (should succeed - firewall allows web)
curl http://$protectedIP

# Test RDP (should fail - firewall denies admin protocols)
Test-NetConnection -ComputerName $protectedIP -Port 3389

# Check routing
Get-NetRoute | Where-Object {$_.DestinationPrefix -eq "10.22.0.0/16"}
# Should show NextHop = firewall IP when force_spoke_egress_to_firewall = true
```

### Scenario 2: Key Vault Secret Retrieval

```bash
# Get Key Vault name from output
KV_NAME=$(terraform output -raw keyvault_name)

# Retrieve Windows admin password
az keyvault secret show --vault-name $KV_NAME --name windows-admin-password --query value -o tsv

# List all lab secrets
az keyvault secret list --vault-name $KV_NAME --query "[].name" -o table
```

### Scenario 3: Firewall Log Analysis

```kusto
// In Log Analytics workspace

// Show all firewall deny events
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK"
| where Category in ("AzureFirewallNetworkRule", "AZFWNetworkRule")
| where msg_s has "Deny"
| project TimeGenerated, msg_s, Protocol, SourceIP, DestinationIP, DestinationPort
| order by TimeGenerated desc
| take 50

// Show allowed web traffic
AzureDiagnostics
| where Category == "AzureFirewallNetworkRule"
| where msg_s has "Allow"
| where DestinationPort in ("80", "443")
| summarize Count = count() by SourceIP, DestinationIP, DestinationPort
```

### Scenario 4: NSG Flow Verification

```bash
# Check NSG rules from Azure CLI
PROTECTED_NSG_NAME="<prefix>-protected-web-nsg"
RG_NAME="<prefix>-protected-rg"

az network nsg rule list \
  --resource-group $RG_NAME \
  --nsg-name $PROTECTED_NSG_NAME \
  --query "[].{Name:name, Priority:priority, Access:access, Protocol:protocol, Direction:direction}" \
  --output table
```

## Module Structure

The project uses reusable modules:

```hcl
# Example module usage from main.tf

module "hub_network" {
  source = "./modules/hub-network"
  
  resource_group_name = azurerm_resource_group.hub.name
  location            = var.location
  vnet_address_space  = var.hub_vnet_address_space
  
  deploy_bastion          = var.deploy_bastion
  bastion_subnet_prefix   = var.bastion_subnet_prefix
  
  tags = local.common_tags
}

module "firewall" {
  source = "./modules/firewall"
  count  = var.deploy_firewall ? 1 : 0
  
  resource_group_name = azurerm_resource_group.hub.name
  location            = var.location
  
  firewall_subnet_id      = module.hub_network.firewall_subnet_id
  firewall_mgmt_subnet_id = module.hub_network.firewall_mgmt_subnet_id
  
  sku_tier = var.firewall_sku_tier
  
  test_spoke_cidr      = var.test_vnet_address_space[0]
  protected_spoke_cidr = var.protected_vnet_address_space[0]
  
  tags = local.common_tags
}

module "test_spoke" {
  source = "./modules/test-spoke"
  
  resource_group_name = azurerm_resource_group.test.name
  location            = var.location
  vnet_address_space  = var.test_vnet_address_space
  
  admin_username       = var.windows_admin_username
  admin_password       = random_password.windows_admin.result
  admin_source_ip_cidr = var.admin_source_ip_cidr
  
  enable_public_ip = var.enable_test_client_public_ip
  
  firewall_private_ip = var.deploy_firewall ? module.firewall[0].private_ip : null
  force_firewall_routing = var.force_spoke_egress_to_firewall
  
  tags = local.common_tags
}
```

## Troubleshooting Guide

### Issue: Terraform Init Fails

```bash
# Clear backend state
rm -rf .terraform/
rm .terraform.lock.hcl

# Re-initialize
terraform init -upgrade
```

### Issue: Asymmetric Routing (RDP Fails After Apply)

**Symptom**: Cannot RDP to test client after enabling firewall routing

**Cause**: `force_spoke_egress_to_firewall = true` with public RDP causes asymmetric routing

**Solution**:

```hcl
# Option 1: Use public RDP without forced routing
force_spoke_egress_to_firewall = false
enable_test_client_public_ip = true

# Option 2: Use Bastion for private access
force_spoke_egress_to_firewall = true
enable_test_client_public_ip = false
deploy_bastion = true
```

### Issue: Cannot Reach Protected Web from Test Client

```bash
# 1. Check route table
az network route-table route list \
  --resource-group <prefix>-test-rg \
  --route-table-name <prefix>-test-rt \
  --output table

# 2. Verify firewall rules
az network firewall policy rule-collection-group list \
  --policy-name <prefix>-fw-policy \
  --resource-group <prefix>-hub-rg

# 3. Check NSG on protected subnet
az network nsg rule list \
  --resource-group <prefix>-protected-rg \
  --nsg-name <prefix>-protected-web-nsg \
  --include-default \
  --output table

# 4. Test from test client:
Test-NetConnection -ComputerName 10.22.1.4 -Port 80
```

### Issue: Key Vault Access Denied

```bash
# Check current user identity
az account show --query user.name -o tsv

# Grant yourself access
KV_NAME=$(terraform output -raw keyvault_name)
USER_ID=$(az ad signed-in-user show --query id -o tsv)

az keyvault set-policy \
  --name $KV_NAME \
  --object-id $USER_ID \
  --secret-permissions get list
```

### Issue: Entra User Creation Fails

**Error**: "Insufficient privileges to complete the operation"

**Solution**:

```bash
# Check Graph API permissions
az ad app permission list --id <app-id>

# Required permissions:
# - User.ReadWrite.All
# - Group.ReadWrite.All

# Grant admin consent in Azure Portal:
# Azure AD → App registrations → Your app → API permissions → Grant admin consent
```

### Issue: Budget Alerts Not Received

```hcl
# Verify budget configuration
deploy_budget = true
budget_amount = 500

# Check contact role in subscription
az account show --query user.assignedRoles
# Must have "Owner" role for budget alerts
```

### Issue: Firewall Logs Not Appearing

```bash
# Verify diagnostic settings
az monitor diagnostic-settings list \
  --resource <firewall-resource-id> \
  --query "[].{Name:name, Workspace:workspaceId}" \
  --output table

# Check Log Analytics workspace
az monitor log-analytics workspace show \
  --resource-group <prefix>-security-rg \
  --workspace-name <prefix>-law

# Allow 5-15 minutes for initial log ingestion
```

## Cost Control Strategies

### Estimate Before Deploy

```bash
# Use Azure Pricing Calculator
# Main costs:
# - Azure Firewall Basic: ~0.40 USD/hour (~292 USD/month)
# - Azure Bastion: ~0.19 USD/hour (~140 USD/month)
# - VMs: 2x B2s ~0.04 USD/hour each (~60 USD/month)
# - Storage, networking: ~10-20 USD/month

# Total: ~400-500 USD/month with firewall and Bastion
```

### Deploy Time-Limited Lab

```bash
# Deploy for testing
terraform apply -var-file="terraform.tfvars"

# Run validation tests
./scripts/validate-lab.sh

# Destroy immediately after
terraform destroy -var-file="terraform.tfvars" -auto-approve
```

### Minimal Cost Configuration

```hcl
# Lowest cost setup
deploy_firewall              = false
deploy_bastion               = false
enable_test_client_public_ip = true
vm_size                      = "Standard_B1s"  # Smallest VM

# Cost: ~50-80 USD/month
```

## GitHub Actions Integration

The project includes CI validation:

```yaml
# .github/workflows/terraform-validate.yml
name: Terraform Validate

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.9.0
      
      - name: Terraform Init
        run: terraform init -backend=false
      
      - name: Terraform Format Check
        run: terraform fmt -check -recursive
      
      - name: Terraform Validate
        run: terraform validate
```

## Security Best Practices

### 1. Restrict Admin Access

```hcl
# Always use /32 for admin source
admin_source_ip_cidr = "203.0.113.42/32"  # Your IP only

# Never use:
# admin_source_ip_cidr = "0.0.0.0/0"  # Insecure!
```

### 2. Use Environment Variables for Sensitive Data

```bash
# Export Azure credentials
export ARM_SUBSCRIPTION_ID="<subscription-id>"
export ARM_CLIENT_ID="<client-id>"
export ARM_CLIENT_SECRET="<client-secret>"
export ARM_TENANT_ID="<tenant-id>"

# Terraform will use these automatically
terraform apply
```

### 3. Enable All Security Features

```hcl
deploy_keyvault      = true  # Store secrets securely
deploy_log_analytics = true  # Enable logging
deploy_policy        = true  # Audit posture
deploy_budget        = true  # Cost alerts
```

### 4. Review Firewall Rules Before Deploy

```hcl
# Check modules/firewall/main.tf for rule logic
# Understand what traffic is allowed/denied
# Customize for your lab scenarios
```

## Wiki Reference

The project includes extensive wiki documentation:

- **Get Started**: `wiki/get-started/deploy.md`, `wiki/get-started/final-validation.md`
- **Architecture**: `wiki/architecture/overview.md`, `wiki/architecture/security-controls.md`
- **Scenarios**: `wiki/scenarios/firewall-east-west.md`, `wiki/scenarios/key-vault-secrets.md`
- **Reference**: `wiki/reference/variables.md`, `wiki/reference/security-test-matrix.md`
- **Runbooks**: `wiki/runbooks/apply-test-destroy.md`, `wiki/runbooks/final-check.md`

## Quick Reference Commands

```bash
# Complete deployment workflow
terraform init
terraform plan -var-file="terraform.tfvars"
terraform apply -var-file="terraform.tfvars"
terraform output
terraform destroy -var-file="terraform.tfvars"

# Get lab access
terraform output test_client
terraform output -raw windows_admin_password

# Query firewall logs
az monitor log-analytics query \
  --workspace <workspace-id> \
  --analytics-query "AzureDiagnostics | where Category == 'AzureFirewallNetworkRule' | take 10"

# Check Key Vault secrets
az keyvault secret list --vault-name <kv-name>

# Validate NSG rules
az network nsg rule list --resource-group <rg> --nsg-name <nsg>
```

This skill provides comprehensive coverage for deploying, configuring, and troubleshooting the Azure Security Lab with Terraform.
