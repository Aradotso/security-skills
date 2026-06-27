---
name: octoguard-openclaw-security-governance
description: Security governance and audit system for OpenClaw AI agents with real-time policy enforcement, token monitoring, and alert notifications
triggers:
  - "set up OctoGuard security monitoring"
  - "configure OpenClaw security policies"
  - "monitor AI agent token usage"
  - "add security rules for OpenClaw"
  - "integrate OctoGuard with OpenClaw"
  - "configure security alerts for AI agents"
  - "audit OpenClaw tool execution"
  - "block dangerous OpenClaw operations"
---

# OctoGuard OpenClaw Security Governance

> Skill by [ara.so](https://ara.so) — Security Skills collection.

OctoGuard is a security governance and audit system designed for OpenClaw (Lobster AI). It monitors user conversations, tool invocations, and execution results in real-time, enforcing security policies and sending alerts when high-risk behaviors are detected. The system operates as a transparent proxy without modifying OpenClaw's core functionality.

## What It Does

- **Policy-Based Interception**: Blocks sensitive file access, dangerous operations, and high-risk commands before execution
- **Token Monitoring**: Tracks token consumption across sessions with threshold-based alerting
- **Audit Logging**: Records all events processed through OpenClaw including allowed and blocked actions
- **Alert Notifications**: Sends notifications via DingTalk, WeChat Work, and Email when security events occur
- **Visual Dashboard**: Provides security posture visualization and OpenClaw gateway control
- **Non-Invasive Design**: Works alongside OpenClaw without code modifications

## Architecture

OctoGuard consists of:
- **Backend**: Node.js-based policy engine and API server
- **Frontend**: Vue.js dashboard for policy management and monitoring
- **Database**: Stores policies, audit logs, and token metrics
- **Security Proxy**: Intercepts and evaluates OpenClaw requests

## Installation

### Prerequisites

```bash
# Node.js 14+ required
node --version

# Database (MySQL/PostgreSQL)
# OpenClaw must be installed and accessible
```

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/O-ozzz/OctoGuard--Free-OpenClaw-Security-Supervision-System.git
cd OctoGuard--Free-OpenClaw-Security-Supervision-System/backend

# Install dependencies
npm install

# Configure environment
cp .env.example .env
```

### Environment Configuration

Edit `backend/.env`:

```bash
# Server Configuration
PORT=3000
NODE_ENV=production

# Database Connection
DB_TYPE=mysql
DB_HOST=localhost
DB_PORT=3306
DB_NAME=octoguard
DB_USER=${DB_USERNAME}
DB_PASSWORD=${DB_PASSWORD}

# OpenClaw Integration
OPENCLAW_HOST=localhost
OPENCLAW_PORT=8080
OPENCLAW_API_KEY=${OPENCLAW_API_KEY}

# Alert Notifications
DINGTALK_WEBHOOK=${DINGTALK_WEBHOOK_URL}
WECHAT_WEBHOOK=${WECHAT_WEBHOOK_URL}
EMAIL_SMTP_HOST=smtp.gmail.com
EMAIL_SMTP_PORT=587
EMAIL_USER=${EMAIL_ADDRESS}
EMAIL_PASSWORD=${EMAIL_APP_PASSWORD}
EMAIL_RECIPIENTS=security@company.com

# Token Monitoring
TOKEN_THRESHOLD_WARNING=100000
TOKEN_THRESHOLD_CRITICAL=200000
```

### Database Initialization

```bash
# Run migrations
npm run migrate

# Seed initial policies (optional)
npm run seed
```

### Start Services

```bash
# Start backend
cd backend
npm start

# Start frontend (separate terminal)
cd frontend
npm install
npm run serve
```

### Production Deployment

```bash
# Build frontend
cd frontend
npm run build

# Use PM2 for backend
npm install -g pm2
cd backend
pm2 start server.js --name octoguard
pm2 save
pm2 startup
```

## Configuration

### Proxy Setup

Configure OpenClaw to route through OctoGuard:

```javascript
// openclaw-config.js
module.exports = {
  proxy: {
    enabled: true,
    host: 'localhost',
    port: 3000,
    path: '/api/proxy'
  },
  security: {
    auditEnabled: true,
    blockingMode: true
  }
}
```

### Policy Engine Configuration

Create security policies via API or dashboard:

```javascript
// policy-config.js
const policyExamples = {
  // Block sensitive file access
  fileSecurity: {
    name: "Block Sensitive File Access",
    type: "file_access",
    enabled: true,
    rules: [
      {
        pattern: "/etc/passwd",
        action: "block",
        severity: "critical"
      },
      {
        pattern: ".*\\.env$",
        action: "block",
        severity: "high"
      },
      {
        pattern: "/home/.*/.ssh/.*",
        action: "block",
        severity: "critical"
      }
    ]
  },
  
  // Block dangerous commands
  commandSecurity: {
    name: "Block Dangerous Commands",
    type: "command_execution",
    enabled: true,
    rules: [
      {
        pattern: "rm -rf",
        action: "block",
        severity: "critical"
      },
      {
        pattern: "curl.*\\|.*bash",
        action: "block",
        severity: "high"
      },
      {
        pattern: "chmod 777",
        action: "warn",
        severity: "medium"
      }
    ]
  }
};
```

## API Usage

### Create Security Policy

```javascript
const axios = require('axios');

async function createPolicy(policy) {
  try {
    const response = await axios.post('http://localhost:3000/api/policies', {
      name: policy.name,
      type: policy.type,
      enabled: policy.enabled,
      rules: policy.rules,
      alertOnBlock: true,
      notificationChannels: ['dingtalk', 'email']
    }, {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    });
    
    console.log('Policy created:', response.data.id);
    return response.data;
  } catch (error) {
    console.error('Failed to create policy:', error.response?.data);
    throw error;
  }
}

// Example usage
createPolicy({
  name: "Block Database Dumps",
  type: "command_execution",
  enabled: true,
  rules: [{
    pattern: "mysqldump|pg_dump",
    action: "block",
    severity: "high"
  }]
});
```

### Query Audit Logs

```javascript
async function getAuditLogs(filters = {}) {
  const params = new URLSearchParams({
    page: filters.page || 1,
    limit: filters.limit || 50,
    action: filters.action || '', // 'blocked', 'allowed', 'warned'
    severity: filters.severity || '',
    startDate: filters.startDate || '',
    endDate: filters.endDate || ''
  });
  
  const response = await axios.get(
    `http://localhost:3000/api/audit/logs?${params}`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    }
  );
  
  return response.data;
}

// Get blocked events from last 24 hours
const blocked = await getAuditLogs({
  action: 'blocked',
  startDate: new Date(Date.now() - 86400000).toISOString()
});
```

### Monitor Token Usage

```javascript
async function getTokenMetrics() {
  const response = await axios.get(
    'http://localhost:3000/api/tokens/metrics',
    {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    }
  );
  
  const { total, sessions, threshold, alerts } = response.data;
  
  console.log(`Total tokens: ${total}`);
  console.log(`Active sessions: ${sessions.length}`);
  console.log(`Threshold: ${threshold.warning}/${threshold.critical}`);
  
  if (total > threshold.critical) {
    console.warn('CRITICAL: Token usage exceeds threshold!');
  }
  
  return response.data;
}

// Set up periodic monitoring
setInterval(async () => {
  const metrics = await getTokenMetrics();
  // Custom logic here
}, 300000); // Every 5 minutes
```

### Control OpenClaw Gateway

```javascript
async function controlGateway(action) {
  const response = await axios.post(
    'http://localhost:3000/api/gateway/control',
    { action }, // 'stop', 'start', 'restart', 'status'
    {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    }
  );
  
  return response.data;
}

// Check gateway status
const status = await controlGateway('status');
console.log('OpenClaw status:', status.running ? 'Running' : 'Stopped');

// Emergency shutdown
if (criticalThreatDetected) {
  await controlGateway('stop');
  console.log('Gateway shut down for security');
}
```

## Common Patterns

### Real-Time Request Monitoring

```javascript
const WebSocket = require('ws');

function monitorRequests() {
  const ws = new WebSocket('ws://localhost:3000/api/monitor/stream', {
    headers: {
      'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
    }
  });
  
  ws.on('message', (data) => {
    const event = JSON.parse(data);
    
    switch (event.type) {
      case 'request':
        console.log(`[${event.timestamp}] Request: ${event.command}`);
        break;
      case 'blocked':
        console.error(`[BLOCKED] ${event.command} - Reason: ${event.reason}`);
        // Send to SIEM
        break;
      case 'token_threshold':
        console.warn(`Token usage: ${event.usage}/${event.threshold}`);
        break;
    }
  });
  
  ws.on('error', (error) => {
    console.error('WebSocket error:', error);
  });
}

monitorRequests();
```

### Dynamic Policy Updates

```javascript
async function updatePolicyRules(policyId, newRules) {
  const response = await axios.patch(
    `http://localhost:3000/api/policies/${policyId}`,
    { rules: newRules },
    {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    }
  );
  
  console.log(`Policy ${policyId} updated`);
  return response.data;
}

// Add new rule to existing policy
const currentPolicy = await axios.get(
  `http://localhost:3000/api/policies/${policyId}`
);

currentPolicy.data.rules.push({
  pattern: "nc -l",
  action: "block",
  severity: "high"
});

await updatePolicyRules(policyId, currentPolicy.data.rules);
```

### Custom Alert Handler

```javascript
async function setupCustomAlerts() {
  const response = await axios.post(
    'http://localhost:3000/api/alerts/webhook',
    {
      url: process.env.CUSTOM_WEBHOOK_URL,
      events: ['blocked', 'critical'],
      format: 'json',
      headers: {
        'X-Custom-Auth': process.env.WEBHOOK_SECRET
      }
    },
    {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    }
  );
  
  return response.data;
}

// Your webhook endpoint receives:
// {
//   "event": "blocked",
//   "timestamp": "2026-06-04T18:10:31Z",
//   "severity": "critical",
//   "command": "rm -rf /",
//   "policy": "Block Dangerous Commands",
//   "user": "external_user_123"
// }
```

### Bulk Policy Management

```javascript
async function bulkPolicyOperation(operation, policyIds) {
  const response = await axios.post(
    'http://localhost:3000/api/policies/bulk',
    {
      operation, // 'enable', 'disable', 'delete'
      policyIds
    },
    {
      headers: {
        'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
      }
    }
  );
  
  return response.data;
}

// Disable all policies temporarily for maintenance
const allPolicies = await axios.get('http://localhost:3000/api/policies');
const policyIds = allPolicies.data.map(p => p.id);
await bulkPolicyOperation('disable', policyIds);

// Re-enable after maintenance
await bulkPolicyOperation('enable', policyIds);
```

## Dashboard Access

Access the web interface:

```
http://localhost:8080
```

Default credentials are set during initial setup. Change immediately:

```bash
curl -X POST http://localhost:3000/api/auth/change-password \
  -H "Authorization: Bearer ${OCTOGUARD_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "currentPassword": "'${CURRENT_PASSWORD}'",
    "newPassword": "'${NEW_PASSWORD}'"
  }'
```

## Troubleshooting

### Policies Not Blocking

**Issue**: Requests pass through despite active policies

**Solutions**:

```javascript
// 1. Verify policy is enabled
const policy = await axios.get(`http://localhost:3000/api/policies/${policyId}`);
console.log('Enabled:', policy.data.enabled);

// 2. Check rule regex patterns
const testPattern = (pattern, testString) => {
  const regex = new RegExp(pattern);
  return regex.test(testString);
};

console.log(testPattern('rm -rf', 'rm -rf /tmp')); // Should be true

// 3. Verify blocking mode is active
const config = await axios.get('http://localhost:3000/api/config');
console.log('Blocking mode:', config.data.blockingMode);
```

### High Token Consumption

**Issue**: Token usage exceeding thresholds unexpectedly

**Solutions**:

```javascript
// Identify high-usage sessions
async function findHighUsageSessions() {
  const sessions = await axios.get('http://localhost:3000/api/tokens/sessions');
  
  const sorted = sessions.data.sort((a, b) => b.tokenUsage - a.tokenUsage);
  
  console.log('Top 5 token consumers:');
  sorted.slice(0, 5).forEach(session => {
    console.log(`Session ${session.id}: ${session.tokenUsage} tokens`);
    console.log(`User: ${session.userId}, Commands: ${session.commandCount}`);
  });
  
  return sorted;
}

// Set per-session limits
await axios.post('http://localhost:3000/api/tokens/limits', {
  sessionLimit: 50000,
  userDailyLimit: 200000,
  enforceHardLimits: true
});
```

### Alert Notifications Not Sending

**Issue**: Security events not triggering notifications

**Solutions**:

```javascript
// Test notification channels
async function testNotifications() {
  const channels = ['dingtalk', 'wechat', 'email'];
  
  for (const channel of channels) {
    try {
      const response = await axios.post(
        'http://localhost:3000/api/alerts/test',
        { channel },
        {
          headers: {
            'Authorization': `Bearer ${process.env.OCTOGUARD_API_KEY}`
          }
        }
      );
      console.log(`${channel}: ${response.data.status}`);
    } catch (error) {
      console.error(`${channel} failed:`, error.response?.data);
    }
  }
}

// Verify webhook URLs
const config = await axios.get('http://localhost:3000/api/config/alerts');
console.log('DingTalk webhook:', config.data.dingtalk?.webhook ? 'Configured' : 'Missing');
```

### Database Connection Issues

**Issue**: Cannot connect to database

**Solutions**:

```bash
# Test database connection
node -e "
const mysql = require('mysql2');
const conn = mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});
conn.connect(err => {
  if (err) console.error('Connection failed:', err);
  else console.log('Connected successfully');
  conn.end();
});
"
```

### OpenClaw Gateway Not Responding

**Issue**: Gateway status shows offline

**Solutions**:

```javascript
// Diagnostic check
async function diagnoseGateway() {
  try {
    // Check if OpenClaw is accessible
    const directCheck = await axios.get(
      `http://${process.env.OPENCLAW_HOST}:${process.env.OPENCLAW_PORT}/health`
    );
    console.log('Direct OpenClaw access: OK');
    
    // Check OctoGuard proxy
    const proxyCheck = await axios.get('http://localhost:3000/api/gateway/health');
    console.log('Proxy health:', proxyCheck.data);
    
  } catch (error) {
    console.error('Gateway diagnostic failed:', error.message);
    console.log('Verify OPENCLAW_HOST and OPENCLAW_PORT in .env');
  }
}

diagnoseGateway();
```

## Security Best Practices

1. **Change default credentials** immediately after deployment
2. **Use environment variables** for all sensitive configuration
3. **Enable HTTPS** in production deployments
4. **Rotate API keys** regularly
5. **Monitor audit logs** daily for suspicious patterns
6. **Set appropriate token thresholds** based on usage patterns
7. **Test policies** in non-blocking mode before enforcing
8. **Back up policies and audit logs** regularly

## Support

For issues and feature requests:
- GitHub Issues: https://github.com/O-ozzz/OctoGuard--Free-OpenClaw-Security-Supervision-System/issues
- Email: 1710628464@qq.com
- X/Twitter: @LYtow9610
