---
name: ip-security-analyzer-cloudflare-worker
description: Deploy and customize a Cloudflare Worker for IP intelligence, VPN/proxy detection, WebRTC leak testing, and security scoring with IPinfo and AbuseIPDB integration
triggers:
  - analyze IP addresses with cloudflare worker
  - detect VPN or proxy using cloudflare
  - implement WebRTC leak testing
  - create IP security dashboard
  - check IP reputation with AbuseIPDB
  - build network forensics tool on cloudflare
  - setup IP intelligence worker
  - detect datacenter and hosting IPs
---

# IP Security Analyzer Cloudflare Worker

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

IP Security Analyzer is a single-file Cloudflare Worker that provides forensic IP intelligence by combining Cloudflare metadata, IPinfo Lite ASN enrichment, AbuseIPDB reputation data, request header consistency checks, and browser-side WebRTC leak detection. It separates network classification from risk scoring — infrastructure IPs (datacenter, CDN, cloud) are identified but not penalized unless abuse indicators exist.

**Key capabilities:**
- Public IP detection and network identity analysis
- ASN and ISP classification (hosting, mobile, residential, VPN/proxy)
- Abuse reputation scoring via AbuseIPDB
- Request header consistency checks (automation detection)
- Browser WebRTC candidate leak testing
- Transparent risk scoring model (starts at 100, penalized only by evidence)
- RESTful JSON API and modern HTML dashboard

## Installation

### Option 1: Cloudflare Dashboard

1. Open Cloudflare Dashboard → **Workers & Pages**
2. Click **Create Worker**
3. Name it (e.g., `ip-security-analyzer`)
4. Click **Edit Code**
5. Paste the Worker code from the repository
6. Click **Save and Deploy**

### Option 2: Wrangler CLI

```bash
# Initialize project
npx wrangler init ip-security-analyzer
cd ip-security-analyzer

# Create worker file
cat > src/index.js << 'EOF'
// Paste the full Worker code here
EOF

# Deploy
npx wrangler deploy
```

### Setting Up External APIs

The Worker functions without external keys but enrichment is recommended:

```bash
# Add secrets via Wrangler
npx wrangler secret put IPINFO_TOKEN
# Enter your IPinfo Lite token

npx wrangler secret put ABUSEIPDB_KEY
# Enter your AbuseIPDB API key
```

**Get API keys:**
- IPinfo Lite: https://ipinfo.io/signup (free tier available)
- AbuseIPDB: https://www.abuseipdb.com/register (free tier available)

**Alternatively, add via Cloudflare Dashboard:**
```
Worker → Settings → Variables and Secrets → Add Variable
```

## Core API Endpoints

### GET `/json` or `/api`

Returns full server-side IP analysis as JSON.

```javascript
// Example fetch from client
const response = await fetch('https://your-worker.workers.dev/json');
const data = await response.json();

console.log(data.ip.address);          // "203.0.113.42"
console.log(data.network.asn);         // 15169
console.log(data.network.isp);         // "Google LLC"
console.log(data.risk.score);          // 100
console.log(data.risk.verdict);        // "Low Risk"
console.log(data.risk.tags);           // ["Hosting/Datacenter"]
```

**Response structure:**
```json
{
  "status": "success",
  "ip": {
    "address": "203.0.113.42",
    "version": "IPv4"
  },
  "network": {
    "asn": 15169,
    "isp": "Google LLC",
    "localClassification": {
      "type": "Hosting / Datacenter",
      "flags": {
        "hostingName": true,
        "vpnProxyName": false,
        "abuseDatacenterUsage": true
      }
    }
  },
  "location": {
    "country": "US",
    "region": "California",
    "city": "Mountain View"
  },
  "externalIntel": {
    "ipinfoLite": {
      "enabled": true,
      "ok": true,
      "asn": "AS15169",
      "org": "Google LLC"
    },
    "abuseipdb": {
      "enabled": true,
      "ok": true,
      "abuseConfidenceScore": 0,
      "totalReports": 0,
      "isTor": false,
      "usageType": "Data Center/Web Hosting/Transit"
    }
  },
  "cloudflare": {
    "colo": "SJC",
    "tlsVersion": "TLSv1.3",
    "httpProtocol": "HTTP/2",
    "ray": "abc123..."
  },
  "risk": {
    "score": 100,
    "verdict": "Low Risk",
    "tags": ["Hosting/Datacenter"],
    "findings": []
  }
}
```

### POST `/report`

Receives browser WebRTC candidates and returns combined risk analysis.

```javascript
// Client-side WebRTC leak test
async function testWebRTC() {
  const pc = new RTCPeerConnection({ iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] });
  const candidates = [];

  pc.onicecandidate = (event) => {
    if (event.candidate) {
      candidates.push({
        candidate: event.candidate.candidate,
        type: event.candidate.type,
        protocol: event.candidate.protocol
      });
    } else {
      // All candidates gathered
      sendCandidates(candidates);
    }
  };

  pc.createDataChannel('test');
  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);
}

async function sendCandidates(candidates) {
  const response = await fetch('/report', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ candidates })
  });
  
  const result = await response.json();
  console.log('Risk score:', result.risk.score);
  console.log('WebRTC findings:', result.webrtc);
}
```

**Response includes:**
```json
{
  "webrtc": {
    "publicIPs": ["198.51.100.42"],
    "privateIPs": ["192.168.1.100"],
    "mdnsIPs": ["abcd1234-5678-90ab-cdef-1234567890ab.local"],
    "mismatch": true,
    "httpIP": "203.0.113.42"
  },
  "risk": {
    "score": 75,
    "findings": [
      {
        "category": "WebRTC Leak",
        "severity": "High",
        "message": "WebRTC exposed different public IP: 198.51.100.42 vs HTTP IP 203.0.113.42",
        "points": -15
      }
    ]
  }
}
```

### GET `/health`

Basic status endpoint.

```bash
curl https://your-worker.workers.dev/health
```

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2026-07-29T13:48:27Z"
}
```

## Configuration

### Manual API Key Fallback

If you cannot use Cloudflare secrets, edit the Worker code:

```javascript
// At the top of the Worker file
const MANUAL_IPINFO_TOKEN = ""; // Add your IPinfo token here
const MANUAL_ABUSEIPDB_KEY = ""; // Add your AbuseIPDB key here

// The Worker checks env.IPINFO_TOKEN first, then falls back to MANUAL_IPINFO_TOKEN
```

**⚠️ Important:** Never commit API keys to public repositories. Use environment variables in production.

### Custom Domain Setup

Attach a custom domain via Cloudflare Dashboard:

```
Workers & Pages → Your Worker → Settings → Triggers → Add Custom Domain
```

Example: `myip.example.com`

### Customizing Risk Scoring

The risk model is defined in the Worker code. Default penalties:

```javascript
// Example from the Worker scoring logic
const scoringModel = {
  tor: -50,                           // Tor exit node
  abuseConfidence: -1,                // Per point of AbuseIPDB confidence
  abuseReports: {
    low: -5,      // 1-10 reports
    medium: -15,  // 11-50 reports
    high: -30     // 50+ reports
  },
  vpnProxyName: -20,                  // VPN/proxy keyword in ISP name
  automationUA: -15,                  // curl, wget, python, selenium
  noUserAgent: -10,
  missingHeaders: -5,
  oldTLS: -10,                        // Non-TLS 1.3
  botScoreLow: -20,                   // Cloudflare Bot Management < 30
  webrtcMismatch: -15,                // Different public IP via WebRTC
  webrtcPrivate: -5                   // Private IP exposure
};
```

To customize, locate the risk calculation functions in the Worker code and adjust penalty values.

## Network Classification

The Worker classifies networks using heuristic ASN/ISP name matching:

```javascript
// Classification keywords (conceptual, from Worker code)
const classificationPatterns = {
  hosting: ['cloud', 'hosting', 'datacenter', 'server', 'vps', 'colocation', 'transit'],
  vpnProxy: ['vpn', 'proxy', 'tunnel', 'privacy', 'tor', 'anonymizer'],
  mobile: ['mobile', 'wireless', 'cellular', 'lte', '5g', '4g'],
  education: ['university', 'college', 'edu', 'academic'],
  corporate: ['corporate', 'enterprise', 'business']
};
```

**Classification types:**
- `Hosting / Datacenter`
- `Mobile / Cellular`
- `Corporate / Business`
- `Education / Campus`
- `VPN / Proxy / Privacy`
- `Likely Residential`
- `Unknown`

**Important:** Hosting/datacenter classification does NOT reduce risk score by itself.

## Common Patterns

### Embedding IP Analysis in Your Application

```javascript
// Server-side fetch example
export default {
  async fetch(request, env) {
    // Get client IP
    const clientIP = request.headers.get('CF-Connecting-IP');
    
    // Call the IP Security Worker
    const analysis = await fetch(`https://your-worker.workers.dev/json?ip=${clientIP}`);
    const data = await analysis.json();
    
    // Make decisions based on risk
    if (data.risk.score < 50) {
      return new Response('Access denied - high risk IP', { status: 403 });
    }
    
    if (data.network.localClassification.flags.vpnProxyName) {
      console.log('VPN detected:', data.network.isp);
    }
    
    return new Response('Access granted');
  }
};
```

### Custom WebRTC Leak Detector

```javascript
class WebRTCLeakDetector {
  constructor(reportEndpoint) {
    this.endpoint = reportEndpoint;
    this.candidates = [];
  }

  async test() {
    const config = {
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        { urls: 'stun:stun1.l.google.com:19302' }
      ]
    };
    
    const pc = new RTCPeerConnection(config);
    
    return new Promise((resolve, reject) => {
      pc.onicecandidate = (event) => {
        if (event.candidate) {
          this.candidates.push({
            candidate: event.candidate.candidate,
            type: event.candidate.type,
            protocol: event.candidate.protocol,
            address: event.candidate.address
          });
        } else {
          // Gathering complete
          this.report().then(resolve).catch(reject);
        }
      };

      pc.onicegatheringstatechange = () => {
        if (pc.iceGatheringState === 'complete') {
          setTimeout(() => this.report().then(resolve), 1000);
        }
      };

      pc.createDataChannel('leak-test');
      pc.createOffer()
        .then(offer => pc.setLocalDescription(offer))
        .catch(reject);
    });
  }

  async report() {
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ candidates: this.candidates })
    });
    
    return await response.json();
  }

  extractIPs() {
    const ips = new Set();
    this.candidates.forEach(c => {
      const match = c.candidate.match(/(\d+\.\d+\.\d+\.\d+)/);
      if (match) ips.add(match[1]);
    });
    return Array.from(ips);
  }
}

// Usage
const detector = new WebRTCLeakDetector('/report');
const result = await detector.test();
console.log('Public IPs:', result.webrtc.publicIPs);
console.log('Private IPs:', result.webrtc.privateIPs);
console.log('Mismatch detected:', result.webrtc.mismatch);
```

### Checking Specific IP Address

```javascript
// Query analysis for a specific IP
async function analyzeIP(targetIP) {
  const response = await fetch(`https://your-worker.workers.dev/json?ip=${targetIP}`);
  const data = await response.json();
  
  return {
    isVPN: data.network.localClassification.flags.vpnProxyName,
    isTor: data.externalIntel.abuseipdb.isTor,
    isHosting: data.network.localClassification.type.includes('Hosting'),
    riskScore: data.risk.score,
    abuseScore: data.externalIntel.abuseipdb.abuseConfidenceScore,
    country: data.location.country,
    asn: data.network.asn,
    isp: data.network.isp
  };
}

const result = await analyzeIP('1.1.1.1');
console.log(result);
// {
//   isVPN: false,
//   isTor: false,
//   isHosting: true,
//   riskScore: 100,
//   abuseScore: 0,
//   country: 'AU',
//   asn: 13335,
//   isp: 'Cloudflare, Inc.'
// }
```

## Troubleshooting

### External API Failures

If IPinfo or AbuseIPDB fails, the Worker continues with Cloudflare metadata only:

```json
{
  "externalIntel": {
    "ipinfoLite": {
      "enabled": true,
      "ok": false,
      "error": "API request failed"
    },
    "abuseipdb": {
      "enabled": false,
      "reason": "No API key configured"
    }
  }
}
```

**Fix:**
- Verify API keys are correctly set in Cloudflare secrets
- Check API key validity and quota limits
- Review Worker logs: `npx wrangler tail`

### WebRTC Test Returns Empty Candidates

**Causes:**
- Browser privacy settings block WebRTC
- VPN/browser extension blocks ICE gathering
- STUN server unreachable

**Solution:**
```javascript
// Add timeout and error handling
const testWithTimeout = (timeout = 5000) => {
  return Promise.race([
    detector.test(),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('WebRTC timeout')), timeout)
    )
  ]).catch(error => {
    console.log('WebRTC blocked or failed:', error.message);
    return { blocked: true };
  });
};
```

### Score Always 100 Despite VPN

The Worker does not penalize infrastructure IPs by default. Score decreases only with evidence:

**Check for:**
- VPN/proxy keywords in ISP name (`data.network.localClassification.flags.vpnProxyName`)
- Abuse reports (`data.externalIntel.abuseipdb.totalReports`)
- WebRTC mismatch (`data.webrtc.mismatch`)
- Automation headers (`data.risk.findings`)

**Example high-risk VPN:**
```json
{
  "network": {
    "isp": "NordVPN",
    "localClassification": {
      "type": "VPN / Proxy / Privacy",
      "flags": { "vpnProxyName": true }
    }
  },
  "risk": {
    "score": 80,
    "findings": [
      { "category": "Network Classification", "message": "VPN/Proxy keyword detected", "points": -20 }
    ]
  }
}
```

### Custom Domain Shows Worker URL

After adding custom domain:
1. Wait 1-2 minutes for DNS propagation
2. Verify DNS record in Cloudflare DNS settings
3. Check Worker triggers: `Settings → Triggers → Routes`

### CORS Issues When Calling from Browser

Add CORS headers in Worker response:

```javascript
// In the Worker code, modify response headers
const headers = {
  'Content-Type': 'application/json',
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type'
};

// Handle preflight
if (request.method === 'OPTIONS') {
  return new Response(null, { headers });
}
```

## Advanced Usage

### Integrating with Cloudflare Bot Management

If you have Cloudflare Bot Management enabled:

```javascript
// Worker automatically reads bot score from request.cf
const botScore = request.cf?.botManagement?.score || null;

// Low bot score reduces risk score
if (botScore !== null && botScore < 30) {
  findings.push({
    category: 'Bot Detection',
    severity: 'High',
    message: `Low bot score detected: ${botScore}`,
    points: -20
  });
}
```

### Rate Limiting Analysis Endpoint

```javascript
// Add KV namespace for rate limiting
export default {
  async fetch(request, env) {
    const ip = request.headers.get('CF-Connecting-IP');
    const key = `ratelimit:${ip}`;
    
    // Check rate limit (5 requests per minute)
    const count = await env.RATE_LIMIT.get(key);
    if (count && parseInt(count) > 5) {
      return new Response('Rate limit exceeded', { status: 429 });
    }
    
    // Increment
    const newCount = count ? parseInt(count) + 1 : 1;
    await env.RATE_LIMIT.put(key, newCount.toString(), { expirationTtl: 60 });
    
    // Continue with analysis...
  }
};
```

### Logging and Monitoring

```javascript
// Add structured logging
const logAnalysis = (ip, result) => {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    ip: ip,
    asn: result.network.asn,
    isp: result.network.isp,
    riskScore: result.risk.score,
    verdict: result.risk.verdict,
    findings: result.risk.findings.length
  }));
};

// View logs
// npx wrangler tail --format pretty
```

This skill provides comprehensive guidance for deploying, configuring, and integrating the IP Security Analyzer Cloudflare Worker for network forensics, VPN/proxy detection, and IP intelligence analysis.
