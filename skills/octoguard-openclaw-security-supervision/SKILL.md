---
name: octoguard-openclaw-security-supervision
description: Security governance and audit system for OpenClaw AI agents with real-time policy enforcement, token monitoring, and alert notifications
triggers:
  - how do I secure my OpenClaw deployment
  - set up security policies for AI agent actions
  - monitor and audit OpenClaw tool calls
  - block dangerous operations in OpenClaw
  - configure OctoGuard security rules
  - track token usage and set alerts for AI agents
  - intercept high-risk AI behaviors
  - deploy security supervision for OpenClaw
---

# OctoGuard OpenClaw Security Supervision

> Skill by [ara.so](https://ara.so) — Security Skills collection.

OctoGuard (章鱼卫士) is a security governance and audit system designed specifically for OpenClaw AI agents. It provides real-time detection, policy control, and alert notifications for user dialogue commands, tool calls, and execution results. The system acts as a security gateway that intercepts dangerous operations before they execute, monitors token consumption, and maintains comprehensive audit logs.

## What OctoGuard Does

- **Policy-based Interception**: Configure rules to block sensitive file access, dangerous operations, and high-risk behaviors
- **Token Monitoring**: Real-time tracking of token usage with threshold alerts
- **Audit Logging**: Record all OpenClaw events, allowed actions, and blocked attempts
- **Visual Policy Management**: Web-based dashboard for managing security policies
- **OpenClaw Gateway Control**: Monitor and control OpenClaw gateway status
- **Multi-channel Alerts**: Push notifications via DingTalk, WeChat Work, and Email
- **Security Dashboard**: Comprehensive visualization of OpenClaw security posture

## Architecture

OctoGuard sits between external users and OpenClaw, intercepting all requests through a security policy engine before they reach the AI agent. It operates non-invasively without modifying OpenClaw's core functionality.

```
User Request → OctoGuard Security Gateway → Policy Engine → OpenClaw → AI Agent
                        ↓
              Audit Logs + Alerts
```

## Installation

### Prerequisites

- Node.js (v14+)
- MySQL database
- OpenClaw instance running
- DingTalk/WeChat Work webhook (optional, for alerts)

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/O-ozzz/OctoGuard--Free-OpenClaw-Security-Supervision-System.git
cd OctoGuard--Free-OpenClaw-Security-Supervision-System

# Install backend dependencies
cd backend
npm install

# Configure database connection
# Edit config/database.js
```

**Database Configuration** (`config/database.js`):

```javascript
module.exports = {
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 3306,
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'octoguard',
  connectionLimit: 10,
  waitForConnections: true,
  queueLimit: 0
};
```

**Environment Variables** (`.env`):

```bash
# Database
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=octoguard

# Server
PORT=3000
NODE_ENV=production

# OpenClaw Gateway
OPENCLAW_HOST=localhost
OPENCLAW_PORT=8080
OPENCLAW_API_KEY=your_openclaw_key

# Alert Webhooks
DINGTALK_WEBHOOK=https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN
WECHAT_WEBHOOK=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
EMAIL_TO=security-team@company.com
```

### Initialize Database

```bash
# Run database migrations
npm run migrate

# Start backend server
npm start
# Backend runs on http://localhost:3000
```

### Frontend Setup

```bash
# Install frontend dependencies
cd ../frontend
npm install

# Configure API endpoint
# Edit .env or config file
echo "VITE_API_BASE_URL=http://localhost:3000" > .env

# Build and run frontend
npm run build
npm run preview
# Or for development
npm run dev
```

## Core API

### Security Policy Management

**Create Security Policy**:

```javascript
// POST /api/policies
const axios = require('axios');

async function createSecurityPolicy() {
  const policy = {
    name: "Block Sensitive File Access",
    description: "Prevent access to /etc/passwd and other system files",
    enabled: true,
    priority: 1,
    conditions: {
      type: "tool_call",
      tool_name: "file_read",
      pattern: ".*(/etc/passwd|/etc/shadow|.*\\.pem|.*\\.key).*",
      matchType: "regex"
    },
    action: "block",
    alertChannels: ["dingtalk", "email"]
  };

  const response = await axios.post(
    `${process.env.VITE_API_BASE_URL}/api/policies`,
    policy
  );
  
  return response.data;
}
```

**List Active Policies**:

```javascript
// GET /api/policies
async function listPolicies() {
  const response = await axios.get(
    `${process.env.VITE_API_BASE_URL}/api/policies`,
    { params: { enabled: true } }
  );
  
  return response.data.policies;
}
```

**Toggle Policy Status**:

```javascript
// PATCH /api/policies/:id
async function togglePolicy(policyId, enabled) {
  const response = await axios.patch(
    `${process.env.VITE_API_BASE_URL}/api/policies/${policyId}`,
    { enabled }
  );
  
  return response.data;
}
```

### Request Interception

**Intercept OpenClaw Request**:

```javascript
// Middleware for intercepting OpenClaw requests
const policyEngine = require('./services/policyEngine');
const auditLogger = require('./services/auditLogger');

async function interceptRequest(req, res, next) {
  const { user_input, tool_call, session_id } = req.body;
  
  try {
    // Check against all active policies
    const evaluation = await policyEngine.evaluate({
      userInput: user_input,
      toolCall: tool_call,
      sessionId: session_id,
      timestamp: new Date()
    });
    
    if (evaluation.action === 'block') {
      // Log blocked attempt
      await auditLogger.log({
        type: 'BLOCKED',
        sessionId: session_id,
        reason: evaluation.reason,
        policy: evaluation.policyName,
        details: { user_input, tool_call }
      });
      
      // Send alerts
      await sendAlerts(evaluation);
      
      return res.status(403).json({
        error: 'Request blocked by security policy',
        reason: evaluation.reason,
        policy: evaluation.policyName
      });
    }
    
    // Log allowed request
    await auditLogger.log({
      type: 'ALLOWED',
      sessionId: session_id,
      details: { user_input, tool_call }
    });
    
    next();
  } catch (error) {
    console.error('Policy evaluation error:', error);
    next(error);
  }
}
```

### Token Monitoring

**Track Token Usage**:

```javascript
// POST /api/tokens/track
async function trackTokenUsage(sessionId, tokensUsed, model) {
  const response = await axios.post(
    `${process.env.VITE_API_BASE_URL}/api/tokens/track`,
    {
      session_id: sessionId,
      tokens_used: tokensUsed,
      model: model,
      timestamp: new Date().toISOString()
    }
  );
  
  return response.data;
}
```

**Get Token Statistics**:

```javascript
// GET /api/tokens/stats
async function getTokenStats(startDate, endDate) {
  const response = await axios.get(
    `${process.env.VITE_API_BASE_URL}/api/tokens/stats`,
    {
      params: {
        start_date: startDate,
        end_date: endDate
      }
    }
  );
  
  return {
    totalTokens: response.data.total_tokens,
    sessionCount: response.data.session_count,
    averagePerSession: response.data.average_per_session,
    breakdown: response.data.breakdown
  };
}
```

**Set Token Threshold Alert**:

```javascript
// POST /api/tokens/threshold
async function setTokenThreshold(threshold, period = '1h') {
  const response = await axios.post(
    `${process.env.VITE_API_BASE_URL}/api/tokens/threshold`,
    {
      threshold: threshold,
      period: period,
      alert_channels: ["email", "dingtalk"]
    }
  );
  
  return response.data;
}
```

### Audit Logs

**Query Audit Logs**:

```javascript
// GET /api/audit/logs
async function queryAuditLogs(filters) {
  const response = await axios.get(
    `${process.env.VITE_API_BASE_URL}/api/audit/logs`,
    {
      params: {
        type: filters.type, // 'BLOCKED', 'ALLOWED', 'ERROR'
        session_id: filters.sessionId,
        start_date: filters.startDate,
        end_date: filters.endDate,
        limit: filters.limit || 100,
        offset: filters.offset || 0
      }
    }
  );
  
  return response.data.logs;
}
```

**Export Audit Report**:

```javascript
// GET /api/audit/export
async function exportAuditReport(format = 'csv') {
  const response = await axios.get(
    `${process.env.VITE_API_BASE_URL}/api/audit/export`,
    {
      params: {
        format: format, // 'csv' or 'json'
        start_date: new Date(Date.now() - 30*24*60*60*1000).toISOString(),
        end_date: new Date().toISOString()
      },
      responseType: 'blob'
    }
  );
  
  return response.data;
}
```

## Common Security Patterns

### Pattern 1: File System Protection

```javascript
// Prevent access to sensitive system files
const fileSystemPolicy = {
  name: "File System Protection",
  enabled: true,
  priority: 1,
  conditions: {
    type: "tool_call",
    tool_name: ["file_read", "file_write", "file_delete"],
    patterns: [
      "^/etc/.*",
      "^/root/.*",
      ".*\\.ssh/.*",
      ".*\\.pem$",
      ".*\\.key$",
      ".*password.*",
      ".*secret.*"
    ],
    matchType: "regex_any"
  },
  action: "block",
  message: "Access to system files and credentials is prohibited"
};
```

### Pattern 2: Network Request Filtering

```javascript
// Block requests to internal network ranges
const networkPolicy = {
  name: "Internal Network Protection",
  enabled: true,
  priority: 2,
  conditions: {
    type: "tool_call",
    tool_name: ["http_request", "curl", "wget"],
    patterns: [
      "^https?://10\\..*",
      "^https?://172\\.(1[6-9]|2[0-9]|3[0-1])\\..*",
      "^https?://192\\.168\\..*",
      "^https?://localhost.*",
      "^https?://127\\.0\\.0\\..*"
    ],
    matchType: "regex_any"
  },
  action: "block",
  message: "Requests to internal network addresses are blocked"
};
```

### Pattern 3: Command Execution Prevention

```javascript
// Prevent dangerous shell commands
const commandPolicy = {
  name: "Dangerous Command Prevention",
  enabled: true,
  priority: 1,
  conditions: {
    type: "tool_call",
    tool_name: ["shell_exec", "bash", "terminal"],
    patterns: [
      ".*(rm -rf|dd if=|mkfs|format).*",
      ".*(sudo|su ).*",
      ".*>/dev/sd[a-z].*",
      ".*(wget|curl).*\\|.*sh.*",
      ".*nc -l.*",
      ".*iptables.*"
    ],
    matchType: "regex_any"
  },
  action: "block",
  message: "Dangerous system commands are not allowed"
};
```

### Pattern 4: Data Exfiltration Detection

```javascript
// Detect potential data exfiltration
const exfiltrationPolicy = {
  name: "Data Exfiltration Detection",
  enabled: true,
  priority: 3,
  conditions: {
    type: "combined",
    rules: [
      {
        tool_name: "file_read",
        pattern: ".*(database|backup|export|dump).*"
      },
      {
        tool_name: ["http_request", "ftp_upload"],
        pattern: ".*"
      }
    ],
    logic: "sequential_within_5min"
  },
  action: "block",
  alertChannels: ["email", "dingtalk", "wechat"],
  message: "Potential data exfiltration attempt detected"
};
```

## Alert Configuration

### DingTalk Integration

```javascript
// services/alerts/dingtalk.js
const axios = require('axios');

async function sendDingTalkAlert(event) {
  const webhook = process.env.DINGTALK_WEBHOOK;
  
  const message = {
    msgtype: "markdown",
    markdown: {
      title: `🚨 OctoGuard Security Alert`,
      text: `### Security Event Detected
      
**Type**: ${event.type}
**Policy**: ${event.policyName}
**Session**: ${event.sessionId}
**Time**: ${new Date(event.timestamp).toLocaleString()}

**Reason**: ${event.reason}

**Details**:
\`\`\`
${JSON.stringify(event.details, null, 2)}
\`\`\`

> Please review and take appropriate action.`
    }
  };
  
  await axios.post(webhook, message);
}

module.exports = { sendDingTalkAlert };
```

### Email Alerts

```javascript
// services/alerts/email.js
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransporter({
  host: process.env.EMAIL_HOST,
  port: process.env.EMAIL_PORT,
  secure: false,
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASSWORD
  }
});

async function sendEmailAlert(event) {
  const mailOptions = {
    from: process.env.EMAIL_USER,
    to: process.env.EMAIL_TO,
    subject: `[OctoGuard] Security Alert - ${event.type}`,
    html: `
      <h2>🚨 OctoGuard Security Alert</h2>
      <p><strong>Type:</strong> ${event.type}</p>
      <p><strong>Policy:</strong> ${event.policyName}</p>
      <p><strong>Session:</strong> ${event.sessionId}</p>
      <p><strong>Time:</strong> ${new Date(event.timestamp).toLocaleString()}</p>
      <p><strong>Reason:</strong> ${event.reason}</p>
      <h3>Details:</h3>
      <pre>${JSON.stringify(event.details, null, 2)}</pre>
    `
  };
  
  await transporter.sendMail(mailOptions);
}

module.exports = { sendEmailAlert };
```

## Policy Engine Implementation

```javascript
// services/policyEngine.js
const db = require('../config/database');

class PolicyEngine {
  async evaluate(request) {
    // Load active policies sorted by priority
    const policies = await this.loadActivePolicies();
    
    for (const policy of policies) {
      const matches = await this.checkPolicy(policy, request);
      
      if (matches) {
        return {
          action: policy.action,
          reason: policy.message || `Blocked by policy: ${policy.name}`,
          policyName: policy.name,
          policyId: policy.id
        };
      }
    }
    
    return { action: 'allow' };
  }
  
  async loadActivePolicies() {
    const [policies] = await db.query(
      'SELECT * FROM policies WHERE enabled = true ORDER BY priority ASC'
    );
    return policies;
  }
  
  async checkPolicy(policy, request) {
    const conditions = JSON.parse(policy.conditions);
    
    // Check tool call conditions
    if (conditions.type === 'tool_call' && request.toolCall) {
      if (conditions.tool_name) {
        const toolNames = Array.isArray(conditions.tool_name) 
          ? conditions.tool_name 
          : [conditions.tool_name];
          
        if (!toolNames.includes(request.toolCall.name)) {
          return false;
        }
      }
      
      // Check pattern matching
      if (conditions.pattern) {
        const patterns = Array.isArray(conditions.patterns)
          ? conditions.patterns
          : [conditions.pattern];
          
        const content = JSON.stringify(request.toolCall.arguments);
        
        for (const pattern of patterns) {
          const regex = new RegExp(pattern, 'i');
          if (regex.test(content)) {
            return true;
          }
        }
      }
    }
    
    // Check user input conditions
    if (conditions.type === 'user_input' && request.userInput) {
      if (conditions.pattern) {
        const regex = new RegExp(conditions.pattern, 'i');
        if (regex.test(request.userInput)) {
          return true;
        }
      }
    }
    
    return false;
  }
}

module.exports = new PolicyEngine();
```

## OpenClaw Gateway Control

```javascript
// controllers/gatewayController.js
const axios = require('axios');
const { spawn } = require('child_process');

class GatewayController {
  async getStatus() {
    try {
      const response = await axios.get(
        `http://${process.env.OPENCLAW_HOST}:${process.env.OPENCLAW_PORT}/health`,
        { timeout: 5000 }
      );
      
      return {
        status: 'running',
        health: response.data,
        uptime: response.data.uptime
      };
    } catch (error) {
      return {
        status: 'stopped',
        error: error.message
      };
    }
  }
  
  async shutdown() {
    try {
      await axios.post(
        `http://${process.env.OPENCLAW_HOST}:${process.env.OPENCLAW_PORT}/admin/shutdown`,
        {},
        {
          headers: {
            'X-API-Key': process.env.OPENCLAW_API_KEY
          }
        }
      );
      
      return { success: true, message: 'OpenClaw gateway shutdown initiated' };
    } catch (error) {
      throw new Error(`Failed to shutdown gateway: ${error.message}`);
    }
  }
}

module.exports = new GatewayController();
```

## Troubleshooting

### Issue: Policies Not Blocking Requests

**Check policy priority and conditions**:

```javascript
// Debug policy evaluation
async function debugPolicy(policyId, testRequest) {
  const policy = await db.query('SELECT * FROM policies WHERE id = ?', [policyId]);
  const conditions = JSON.parse(policy[0].conditions);
  
  console.log('Policy conditions:', conditions);
  console.log('Test request:', testRequest);
  
  const engine = require('./services/policyEngine');
  const result = await engine.checkPolicy(policy[0], testRequest);
  
  console.log('Match result:', result);
  return result;
}
```

### Issue: Alerts Not Sending

**Verify webhook configuration**:

```bash
# Test DingTalk webhook
curl -X POST "${DINGTALK_WEBHOOK}" \
  -H 'Content-Type: application/json' \
  -d '{"msgtype":"text","text":{"content":"Test from OctoGuard"}}'

# Check email configuration
node -e "
const nodemailer = require('nodemailer');
const transporter = nodemailer.createTransporter({
  host: '${EMAIL_HOST}',
  auth: { user: '${EMAIL_USER}', pass: '${EMAIL_PASSWORD}' }
});
transporter.verify((err, success) => {
  console.log(err || 'Email config OK');
});
"
```

### Issue: High Token Usage Not Alerting

**Check threshold configuration**:

```javascript
// Verify token threshold settings
async function checkTokenThreshold() {
  const [settings] = await db.query(
    'SELECT * FROM token_thresholds WHERE enabled = true'
  );
  
  console.log('Active thresholds:', settings);
  
  // Manually trigger check
  const totalTokens = await db.query(
    'SELECT SUM(tokens_used) as total FROM token_usage WHERE timestamp > ?',
    [new Date(Date.now() - 3600000)] // Last hour
  );
  
  console.log('Current usage:', totalTokens[0].total);
  
  if (totalTokens[0].total > settings[0].threshold) {
    console.log('Threshold exceeded - alert should trigger');
  }
}
```

### Issue: Database Connection Errors

**Test database connectivity**:

```javascript
// Test database connection
const mysql = require('mysql2/promise');

async function testDatabaseConnection() {
  try {
    const connection = await mysql.createConnection({
      host: process.env.DB_HOST,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME
    });
    
    const [rows] = await connection.execute('SELECT 1 as test');
    console.log('Database connection successful:', rows);
    
    await connection.end();
    return true;
  } catch (error) {
    console.error('Database connection failed:', error.message);
    return false;
  }
}
```

### Issue: Frontend Cannot Reach Backend

**Verify CORS and API endpoint**:

```javascript
// backend/server.js - Add CORS middleware
const cors = require('cors');

app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:5173',
  credentials: true
}));

// Frontend - Check API base URL
console.log('API Base URL:', import.meta.env.VITE_API_BASE_URL);
```

## Performance Optimization

**Cache active policies in memory**:

```javascript
// services/policyCache.js
class PolicyCache {
  constructor() {
    this.cache = null;
    this.lastUpdate = null;
    this.ttl = 60000; // 1 minute
  }
  
  async getPolicies() {
    const now = Date.now();
    
    if (!this.cache || !this.lastUpdate || (now - this.lastUpdate) > this.ttl) {
      const [policies] = await db.query(
        'SELECT * FROM policies WHERE enabled = true ORDER BY priority ASC'
      );
      
      this.cache = policies;
      this.lastUpdate = now;
    }
    
    return this.cache;
  }
  
  invalidate() {
    this.cache = null;
    this.lastUpdate = null;
  }
}

module.exports = new PolicyCache();
```

OctoGuard provides comprehensive security supervision for OpenClaw deployments, ensuring AI agents operate within defined security boundaries while maintaining full audit trails and real-time alerting capabilities.
