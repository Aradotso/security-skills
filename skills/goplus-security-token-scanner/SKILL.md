---
name: goplus-security-token-scanner
description: Use GoPlus Security API to scan tokens, addresses, and NFTs for security threats and malicious contracts
triggers:
  - scan this token address for security issues
  - check if this contract is safe with goplus
  - analyze this wallet address for risks
  - verify nft contract security
  - detect phishing or malicious smart contracts
  - run goplus security scan on this address
  - check token approval risks
  - validate contract security with goplus
---

# GoPlus Security Token Scanner

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

GoPlus Security provides comprehensive blockchain security scanning services for tokens, smart contracts, NFTs, and wallet addresses across multiple chains. This tool helps detect malicious contracts, honeypots, phishing attempts, and other security risks in Web3 applications.

Key capabilities:
- Token security analysis (honeypot detection, trading simulation)
- Smart contract vulnerability scanning
- NFT contract security verification
- Malicious address detection
- dApp security auditing
- Approval security checking

## Installation

### API Access

GoPlus Security operates as a REST API service. No local installation required for basic usage.

```bash
# Set your API key (if using premium features)
export GOPLUS_API_KEY="your_api_key_here"
```

### SDK Integration (JavaScript/TypeScript)

```bash
npm install @goplus/sdk-node
```

### SDK Integration (Python)

```bash
pip install goplus-security
```

## Core API Endpoints

Base URL: `https://api.gopluslabs.io/api/v1`

### Token Security

**Endpoint:** `/token_security/{chain_id}`

Supported chains: eth (1), bsc (56), polygon (137), arbitrum (42161), optimism (10), avalanche (43114), fantom (250), etc.

### Address Security

**Endpoint:** `/address_security/{address}`

### NFT Security

**Endpoint:** `/nft_security/{chain_id}`

### Approval Security

**Endpoint:** `/approval_security/{chain_id}`

## Usage Examples

### JavaScript/TypeScript

```javascript
import axios from 'axios';

const GOPLUS_BASE_URL = 'https://api.gopluslabs.io/api/v1';
const API_KEY = process.env.GOPLUS_API_KEY;

// Scan token security
async function scanToken(chainId, tokenAddress) {
  const url = `${GOPLUS_BASE_URL}/token_security/${chainId}`;
  const params = {
    contract_addresses: tokenAddress
  };
  
  const headers = API_KEY ? { 'Authorization': `Bearer ${API_KEY}` } : {};
  
  try {
    const response = await axios.get(url, { params, headers });
    return response.data;
  } catch (error) {
    console.error('Error scanning token:', error.message);
    throw error;
  }
}

// Check address for malicious activity
async function checkAddress(address) {
  const url = `${GOPLUS_BASE_URL}/address_security/${address}`;
  const headers = API_KEY ? { 'Authorization': `Bearer ${API_KEY}` } : {};
  
  try {
    const response = await axios.get(url, { headers });
    return response.data;
  } catch (error) {
    console.error('Error checking address:', error.message);
    throw error;
  }
}

// Scan NFT contract
async function scanNFT(chainId, contractAddress) {
  const url = `${GOPLUS_BASE_URL}/nft_security/${chainId}`;
  const params = {
    contract_addresses: contractAddress
  };
  
  const headers = API_KEY ? { 'Authorization': `Bearer ${API_KEY}` } : {};
  
  try {
    const response = await axios.get(url, { params, headers });
    return response.data;
  } catch (error) {
    console.error('Error scanning NFT:', error.message);
    throw error;
  }
}

// Usage example
(async () => {
  const tokenResult = await scanToken('1', '0x...');
  console.log('Token Security:', tokenResult);
  
  const addressResult = await checkAddress('0x...');
  console.log('Address Security:', addressResult);
})();
```

### Python

```python
import os
import requests

GOPLUS_BASE_URL = 'https://api.gopluslabs.io/api/v1'
API_KEY = os.getenv('GOPLUS_API_KEY')

def scan_token(chain_id, token_address):
    """Scan token for security issues"""
    url = f'{GOPLUS_BASE_URL}/token_security/{chain_id}'
    params = {'contract_addresses': token_address}
    headers = {'Authorization': f'Bearer {API_KEY}'} if API_KEY else {}
    
    response = requests.get(url, params=params, headers=headers)
    response.raise_for_status()
    return response.json()

def check_address(address):
    """Check if address is malicious"""
    url = f'{GOPLUS_BASE_URL}/address_security/{address}'
    headers = {'Authorization': f'Bearer {API_KEY}'} if API_KEY else {}
    
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()

def scan_nft(chain_id, contract_address):
    """Scan NFT contract for security issues"""
    url = f'{GOPLUS_BASE_URL}/nft_security/{chain_id}'
    params = {'contract_addresses': contract_address}
    headers = {'Authorization': f'Bearer {API_KEY}'} if API_KEY else {}
    
    response = requests.get(url, params=params, headers=headers)
    response.raise_for_status()
    return response.json()

def check_approvals(chain_id, addresses):
    """Check token approvals for security risks"""
    url = f'{GOPLUS_BASE_URL}/approval_security/{chain_id}'
    params = {'addresses': addresses}
    headers = {'Authorization': f'Bearer {API_KEY}'} if API_KEY else {}
    
    response = requests.get(url, params=params, headers=headers)
    response.raise_for_status()
    return response.json()

# Usage
if __name__ == '__main__':
    result = scan_token('1', '0x...')
    print('Token Security:', result)
```

### cURL

```bash
# Scan token on Ethereum
curl "https://api.gopluslabs.io/api/v1/token_security/1?contract_addresses=0x..."

# Check address security
curl "https://api.gopluslabs.io/api/v1/address_security/0x..."

# Scan NFT on BSC
curl "https://api.gopluslabs.io/api/v1/nft_security/56?contract_addresses=0x..."
```

## Response Interpretation

### Token Security Response

```javascript
{
  "code": 1,
  "message": "OK",
  "result": {
    "0x...": {
      "is_open_source": "1",
      "is_proxy": "0",
      "is_honeypot": "0",
      "honeypot_with_same_creator": "0",
      "is_blacklisted": "0",
      "is_whitelisted": "0",
      "is_mintable": "0",
      "can_take_back_ownership": "0",
      "owner_change_balance": "0",
      "hidden_owner": "0",
      "selfdestruct": "0",
      "external_call": "0",
      "buy_tax": "0.05",
      "sell_tax": "0.05",
      "trading_cooldown": "0",
      "transfer_pausable": "0",
      "holder_count": "1234",
      "total_supply": "1000000000",
      "creator_address": "0x...",
      "creator_balance": "100000",
      "creator_percent": "0.01"
    }
  }
}
```

### Key Risk Indicators

- `is_honeypot: "1"` - Cannot sell after buying
- `is_blacklisted: "1"` - Known malicious contract
- `can_take_back_ownership: "1"` - Owner can reclaim control
- `hidden_owner: "1"` - Ownership obfuscated
- `selfdestruct: "1"` - Contract can be destroyed
- High `buy_tax` or `sell_tax` (>10%) - Potential rug pull
- `transfer_pausable: "1"` - Trading can be paused

## Common Patterns

### Security Dashboard

```javascript
async function getSecurityScore(chainId, tokenAddress) {
  const result = await scanToken(chainId, tokenAddress);
  const data = result.result[tokenAddress.toLowerCase()];
  
  let score = 100;
  let risks = [];
  
  if (data.is_honeypot === '1') {
    score -= 50;
    risks.push('CRITICAL: Honeypot detected');
  }
  
  if (data.is_blacklisted === '1') {
    score -= 40;
    risks.push('CRITICAL: Blacklisted contract');
  }
  
  if (data.hidden_owner === '1') {
    score -= 20;
    risks.push('HIGH: Hidden owner');
  }
  
  if (data.can_take_back_ownership === '1') {
    score -= 15;
    risks.push('HIGH: Can take back ownership');
  }
  
  if (parseFloat(data.buy_tax) > 0.1 || parseFloat(data.sell_tax) > 0.1) {
    score -= 15;
    risks.push('MEDIUM: High tax rates');
  }
  
  if (data.is_mintable === '1') {
    score -= 10;
    risks.push('MEDIUM: Mintable token');
  }
  
  return {
    score: Math.max(score, 0),
    risks,
    data
  };
}
```

### Batch Scanning

```javascript
async function scanMultipleTokens(chainId, addresses) {
  const url = `${GOPLUS_BASE_URL}/token_security/${chainId}`;
  const params = {
    contract_addresses: addresses.join(',')
  };
  
  const headers = API_KEY ? { 'Authorization': `Bearer ${API_KEY}` } : {};
  const response = await axios.get(url, { params, headers });
  
  return response.data;
}

// Scan up to 50 tokens at once
const results = await scanMultipleTokens('1', [
  '0xtoken1...',
  '0xtoken2...',
  '0xtoken3...'
]);
```

### Wallet Protection

```python
def check_wallet_safety(chain_id, wallet_address):
    """Check wallet and its approvals for security issues"""
    
    # Check address reputation
    address_result = check_address(wallet_address)
    
    # Check token approvals
    approval_result = check_approvals(chain_id, wallet_address)
    
    dangerous_approvals = []
    if approval_result['code'] == 1:
        for approval in approval_result.get('result', []):
            if approval.get('is_malicious') == '1':
                dangerous_approvals.append(approval)
    
    return {
        'is_malicious': address_result.get('result', {}).get('is_malicious'),
        'dangerous_approvals': dangerous_approvals,
        'recommendation': 'Revoke approvals' if dangerous_approvals else 'Safe'
    }
```

## Configuration

### Chain IDs

```javascript
const CHAIN_IDS = {
  ethereum: '1',
  bsc: '56',
  polygon: '137',
  avalanche: '43114',
  fantom: '250',
  arbitrum: '42161',
  optimism: '10',
  cronos: '25',
  moonbeam: '1284'
};
```

### Rate Limiting

Free tier: 100 requests/day per IP
Premium tier: Higher limits with API key

```javascript
// Implement rate limiting
const rateLimit = require('axios-rate-limit');
const http = rateLimit(axios.create(), {
  maxRequests: 2,
  perMilliseconds: 1000
});
```

## Troubleshooting

### Invalid Chain ID
Ensure you're using the correct numeric chain ID, not the chain name.

### Empty Results
Token may be too new or not yet indexed. Wait a few minutes and retry.

### 429 Too Many Requests
You've exceeded rate limits. Implement exponential backoff or upgrade to premium.

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.response?.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
      } else {
        throw error;
      }
    }
  }
}
```

### Contract Address Validation

```javascript
function isValidAddress(address) {
  return /^0x[a-fA-F0-9]{40}$/.test(address);
}
```

## Security Best Practices

1. **Never trust a single indicator** - Check multiple security flags
2. **Cache results** - Token security doesn't change frequently
3. **Monitor approval security** - Regularly check wallet approvals
4. **Combine with other tools** - Use alongside other security scanners
5. **Verify source** - Always check contract source code on block explorers

## Integration Example

```javascript
class TokenSecurityChecker {
  constructor(apiKey = null) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.gopluslabs.io/api/v1';
  }
  
  async checkTokenSafety(chainId, tokenAddress) {
    const result = await this.scanToken(chainId, tokenAddress);
    const data = result.result[tokenAddress.toLowerCase()];
    
    const criticalIssues = [
      data.is_honeypot === '1',
      data.is_blacklisted === '1',
      data.hidden_owner === '1'
    ].filter(Boolean).length;
    
    return {
      isSafe: criticalIssues === 0,
      criticalIssues,
      data
    };
  }
  
  async scanToken(chainId, tokenAddress) {
    const url = `${this.baseUrl}/token_security/${chainId}`;
    const headers = this.apiKey ? { 'Authorization': `Bearer ${this.apiKey}` } : {};
    const response = await axios.get(url, {
      params: { contract_addresses: tokenAddress },
      headers
    });
    return response.data;
  }
}

// Usage
const checker = new TokenSecurityChecker(process.env.GOPLUS_API_KEY);
const safety = await checker.checkTokenSafety('1', '0x...');
console.log(safety.isSafe ? 'Token appears safe' : 'WARNING: Security issues detected');
```
