---
name: octoguard-openclaw-security-supervision
description: Security governance and audit system for OpenClaw AI agents with policy-based interception, token monitoring, and alert notifications
triggers:
  - how do I secure my OpenClaw AI agent
  - set up security policies for OpenClaw
  - monitor and audit AI agent behavior
  - configure OctoGuard security rules
  - intercept dangerous AI agent operations
  - track token usage in OpenClaw
  - set up alerts for risky AI actions
  - audit OpenClaw tool execution
---

# OctoGuard OpenClaw Security Supervision

> Skill by [ara.so](https://ara.so) — Security Skills collection.

OctoGuard (章鱼卫士) is a security governance and audit system designed for OpenClaw AI agents. It provides real-time detection, policy-based control, and alert notifications for user dialogue commands, tool calls, and execution results. The system acts as a security gateway that monitors and controls all AI agent behaviors without modifying the original OpenClaw deployment.

## What OctoGuard Does

- **Policy-Based Interception**: Block access to sensitive files, dangerous operations, and high-risk behaviors based on configurable security rules
- **Token Monitoring**: Track total token usage, session details, and threshold-based alerting
- **Audit Logging**: Record all events processed through OpenClaw, including allowed and blocked actions
- **Visual Policy Management**: Web-based dashboard for viewing, enabling/disabling, and managing security policies
- **OpenClaw Control**: Monitor OpenClaw gateway status and emergency shutdown capability
- **Alert Notifications**: Push notifications via DingTalk, WeChat Work, and Email when interception events occur
- **Security Dashboard**: Comprehensive visualization of OpenClaw security posture

## Architecture

OctoGuard operates as a security proxy layer between external requests and the OpenClaw gateway:

```
User Request → OctoGuard Security Engine → Policy Evaluation → OpenClaw Gateway → AI Agent
                                        ↓
                                   Audit Log & Alerts
```

## Installation

### Prerequisites

- Node.js 14+ and npm
- MySQL 5.7+ or compatible database
- OpenClaw instance running
- Network access to OpenClaw gateway

### Database Setup

1. Create MySQL database and import schema:

```sql
CREATE DATABASE octoguard CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE octoguard;
SOURCE backend/schema.sql;
```

2. Configure database connection in `backend/config/db.js`:

```javascript
module.exports = {
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 3306,
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'octoguard',
  connectionLimit: 10
};
```

### Backend Setup

```bash
cd backend
npm install
npm start
```

The backend API will start on `http://localhost:3000` by default.

### Frontend Setup

```bash
cd frontend
npm install
npm run build
npm run serve
```

Access the dashboard at `http://localhost:8080`.

### OpenClaw Integration

Configure OctoGuard to proxy requests to your OpenClaw instance in `backend/config/openclaw.js`:

```javascript
module.exports = {
  gatewayUrl: process.env.OPENCLAW_GATEWAY_URL || 'http://localhost:8000',
  timeout: parseInt(process.env.OPENCLAW_TIMEOUT) || 30000,
  retries: parseInt(process.env.OPENCLAW_RETRIES) || 3
};
```

## Core API Usage

### Security Policy Engine

Create and manage security policies through the API:

```javascript
const axios = require('axios');

// Create a new security policy
async function createPolicy(policyData) {
  const response = await axios.post('http://localhost:3000/api/policies', {
    name: policyData.name,
    description: policyData.description,
    ruleType: policyData.ruleType, // 'keyword', 'regex', 'path', 'command'
    pattern: policyData.pattern,
    action: policyData.action, // 'block', 'alert', 'log'
    enabled: true,
    severity: policyData.severity // 'low', 'medium', 'high', 'critical'
  });
  return response.data;
}

// Example: Block sensitive file access
await createPolicy({
  name: 'Block Sensitive Files',
  description: 'Prevent access to /etc/passwd and shadow files',
  ruleType: 'regex',
  pattern: '(/etc/(passwd|shadow)|.*\\.key|.*\\.pem)',
  action: 'block',
  severity: 'critical'
});

// Example: Alert on database operations
await createPolicy({
  name: 'Database Operation Alert',
  description: 'Alert when database commands are executed',
  ruleType: 'keyword',
  pattern: 'DROP TABLE,DELETE FROM,TRUNCATE',
  action: 'alert',
  severity: 'high'
});
```

### Intercepting Requests

The security middleware processes all requests before forwarding to OpenClaw:

```javascript
const express = require('express');
const policyEngine = require('./services/policyEngine');
const auditLogger = require('./services/auditLogger');

const router = express.Router();

router.post('/execute', async (req, res) => {
  const { userId, sessionId, command, toolName, parameters } = req.body;

  // Evaluate request against security policies
  const evaluation = await policyEngine.evaluate({
    command,
    toolName,
    parameters,
    userId,
    sessionId
  });

  // Log the evaluation
  await auditLogger.log({
    userId,
    sessionId,
    command,
    toolName,
    parameters,
    action: evaluation.action,
    matchedPolicies: evaluation.matchedPolicies,
    timestamp: new Date()
  });

  // Block if policy dictates
  if (evaluation.action === 'block') {
    await sendAlert(evaluation);
    return res.status(403).json({
      blocked: true,
      reason: evaluation.reason,
      policies: evaluation.matchedPolicies
    });
  }

  // Forward to OpenClaw
  try {
    const openclawResponse = await forwardToOpenClaw(req.body);
    
    // Log successful execution
    await auditLogger.logExecution({
      userId,
      sessionId,
      success: true,
      response: openclawResponse.data
    });

    return res.json(openclawResponse.data);
  } catch (error) {
    await auditLogger.logExecution({
      userId,
      sessionId,
      success: false,
      error: error.message
    });
    throw error;
  }
});

module.exports = router;
```

### Policy Engine Implementation

```javascript
// backend/services/policyEngine.js
class PolicyEngine {
  constructor() {
    this.policies = [];
    this.loadPolicies();
  }

  async loadPolicies() {
    // Load active policies from database
    const db = require('../config/db');
    const [rows] = await db.query(
      'SELECT * FROM policies WHERE enabled = 1 ORDER BY severity DESC'
    );
    this.policies = rows;
  }

  async evaluate(request) {
    const matchedPolicies = [];
    let highestAction = 'allow';
    let reason = '';

    const requestString = JSON.stringify(request).toLowerCase();

    for (const policy of this.policies) {
      let matched = false;

      switch (policy.ruleType) {
        case 'keyword':
          const keywords = policy.pattern.split(',');
          matched = keywords.some(kw => 
            requestString.includes(kw.trim().toLowerCase())
          );
          break;

        case 'regex':
          const regex = new RegExp(policy.pattern, 'i');
          matched = regex.test(requestString);
          break;

        case 'path':
          matched = request.parameters?.path && 
                   request.parameters.path.includes(policy.pattern);
          break;

        case 'command':
          matched = request.command?.includes(policy.pattern);
          break;
      }

      if (matched) {
        matchedPolicies.push({
          id: policy.id,
          name: policy.name,
          severity: policy.severity,
          action: policy.action
        });

        if (policy.action === 'block') {
          highestAction = 'block';
          reason = policy.description;
          break;
        } else if (policy.action === 'alert' && highestAction !== 'block') {
          highestAction = 'alert';
          reason = policy.description;
        }
      }
    }

    return {
      action: highestAction,
      reason,
      matchedPolicies
    };
  }
}

module.exports = new PolicyEngine();
```

### Token Monitoring

Track and monitor token usage across sessions:

```javascript
// backend/services/tokenMonitor.js
const db = require('../config/db');

class TokenMonitor {
  async recordUsage(sessionId, userId, tokensUsed, modelName) {
    await db.query(
      `INSERT INTO token_usage 
       (session_id, user_id, tokens_used, model_name, timestamp)
       VALUES (?, ?, ?, ?, NOW())`,
      [sessionId, userId, tokensUsed, modelName]
    );

    // Check threshold
    const threshold = await this.getThreshold(userId);
    const totalUsage = await this.getTotalUsage(userId);

    if (totalUsage >= threshold) {
      await this.sendThresholdAlert(userId, totalUsage, threshold);
    }
  }

  async getTotalUsage(userId, period = 'day') {
    const periodMap = {
      day: 'DATE(timestamp) = CURDATE()',
      week: 'YEARWEEK(timestamp) = YEARWEEK(NOW())',
      month: 'MONTH(timestamp) = MONTH(NOW()) AND YEAR(timestamp) = YEAR(NOW())'
    };

    const [rows] = await db.query(
      `SELECT SUM(tokens_used) as total
       FROM token_usage
       WHERE user_id = ? AND ${periodMap[period]}`,
      [userId]
    );

    return rows[0]?.total || 0;
  }

  async getThreshold(userId) {
    const [rows] = await db.query(
      'SELECT token_threshold FROM users WHERE id = ?',
      [userId]
    );
    return rows[0]?.token_threshold || 1000000;
  }

  async sendThresholdAlert(userId, usage, threshold) {
    const alertService = require('./alertService');
    await alertService.send({
      type: 'token_threshold',
      userId,
      message: `Token usage (${usage}) exceeded threshold (${threshold})`,
      severity: 'warning'
    });
  }
}

module.exports = new TokenMonitor();
```

### Alert Configuration

Configure multiple alert channels:

```javascript
// backend/services/alertService.js
const axios = require('axios');

class AlertService {
  async send(alert) {
    const channels = await this.getEnabledChannels();

    const promises = channels.map(channel => {
      switch (channel.type) {
        case 'dingtalk':
          return this.sendDingTalk(channel.webhook, alert);
        case 'wechat_work':
          return this.sendWechatWork(channel.webhook, alert);
        case 'email':
          return this.sendEmail(channel.config, alert);
      }
    });

    await Promise.allSettled(promises);
  }

  async sendDingTalk(webhook, alert) {
    await axios.post(webhook, {
      msgtype: 'text',
      text: {
        content: `🚨 OctoGuard Alert\n\n${alert.message}\n\nSeverity: ${alert.severity}\nTime: ${new Date().toISOString()}`
      }
    });
  }

  async sendWechatWork(webhook, alert) {
    await axios.post(webhook, {
      msgtype: 'text',
      text: {
        content: `🚨 OctoGuard Alert\n\n${alert.message}\n\nSeverity: ${alert.severity}\nTime: ${new Date().toISOString()}`
      }
    });
  }

  async sendEmail(config, alert) {
    const nodemailer = require('nodemailer');
    
    const transporter = nodemailer.createTransporter({
      host: config.smtp_host,
      port: config.smtp_port,
      secure: config.smtp_secure,
      auth: {
        user: config.smtp_user,
        pass: process.env.SMTP_PASSWORD
      }
    });

    await transporter.sendMail({
      from: config.from_email,
      to: config.to_email,
      subject: `OctoGuard Alert - ${alert.severity}`,
      text: `${alert.message}\n\nTime: ${new Date().toISOString()}`
    });
  }

  async getEnabledChannels() {
    const db = require('../config/db');
    const [rows] = await db.query(
      'SELECT * FROM alert_channels WHERE enabled = 1'
    );
    return rows;
  }
}

module.exports = new AlertService();
```

## Common Usage Patterns

### Pattern 1: File Access Protection

```javascript
// Prevent access to sensitive system files
await createPolicy({
  name: 'System File Protection',
  description: 'Block access to critical system files',
  ruleType: 'regex',
  pattern: '^(/etc|/sys|/proc|/boot|/root)',
  action: 'block',
  severity: 'critical'
});

// Prevent credential file access
await createPolicy({
  name: 'Credential File Protection',
  description: 'Block access to credential files',
  ruleType: 'regex',
  pattern: '\\.(key|pem|p12|pfx|crt|cer|env|credentials)$',
  action: 'block',
  severity: 'critical'
});
```

### Pattern 2: Command Execution Control

```javascript
// Block dangerous shell commands
await createPolicy({
  name: 'Dangerous Command Block',
  description: 'Prevent execution of destructive commands',
  ruleType: 'keyword',
  pattern: 'rm -rf,mkfs,dd if=,format,fdisk,parted',
  action: 'block',
  severity: 'critical'
});

// Monitor network operations
await createPolicy({
  name: 'Network Activity Monitor',
  description: 'Alert on network-related commands',
  ruleType: 'keyword',
  pattern: 'curl,wget,nc,netcat,ssh,scp,ftp',
  action: 'alert',
  severity: 'medium'
});
```

### Pattern 3: Data Exfiltration Prevention

```javascript
// Monitor large data transfers
await createPolicy({
  name: 'Data Transfer Monitor',
  description: 'Alert on potential data exfiltration',
  ruleType: 'keyword',
  pattern: 'base64,gzip,tar,zip,compress',
  action: 'alert',
  severity: 'high'
});

// Block external connections
await createPolicy({
  name: 'External Connection Block',
  description: 'Prevent connections to external IPs',
  ruleType: 'regex',
  pattern: '(?:\\d{1,3}\\.){3}\\d{1,3}|http[s]?://',
  action: 'block',
  severity: 'high'
});
```

### Pattern 4: Session Management

```javascript
const TokenMonitor = require('./services/tokenMonitor');

// Track token usage per session
async function handleAIRequest(userId, sessionId, request) {
  const response = await executeOpenClawRequest(request);
  
  // Record token usage
  await TokenMonitor.recordUsage(
    sessionId,
    userId,
    response.tokensUsed,
    response.model
  );

  return response;
}

// Get usage statistics
async function getUserStats(userId) {
  const dayUsage = await TokenMonitor.getTotalUsage(userId, 'day');
  const weekUsage = await TokenMonitor.getTotalUsage(userId, 'week');
  const monthUsage = await TokenMonitor.getTotalUsage(userId, 'month');

  return { dayUsage, weekUsage, monthUsage };
}
```

## Configuration

### Environment Variables

```bash
# Database
DB_HOST=localhost
DB_PORT=3306
DB_USER=octoguard
DB_PASSWORD=your_secure_password
DB_NAME=octoguard

# OpenClaw Gateway
OPENCLAW_GATEWAY_URL=http://localhost:8000
OPENCLAW_TIMEOUT=30000
OPENCLAW_RETRIES=3

# Server
PORT=3000
NODE_ENV=production

# Alert Services
SMTP_PASSWORD=your_smtp_password
DINGTALK_WEBHOOK=your_webhook_url
WECHAT_WORK_WEBHOOK=your_webhook_url

# Token Monitoring
DEFAULT_TOKEN_THRESHOLD=1000000
TOKEN_ALERT_ENABLED=true
```

### Policy Configuration File

```json
{
  "defaultPolicies": [
    {
      "name": "System Protection",
      "ruleType": "regex",
      "pattern": "(/etc/passwd|/etc/shadow)",
      "action": "block",
      "severity": "critical"
    },
    {
      "name": "Database Safety",
      "ruleType": "keyword",
      "pattern": "DROP,TRUNCATE,DELETE",
      "action": "alert",
      "severity": "high"
    }
  ],
  "auditRetentionDays": 90,
  "alertBatchSize": 10,
  "policyRefreshInterval": 60000
}
```

## Troubleshooting

### OctoGuard Not Intercepting Requests

**Issue**: Policies are configured but requests are not being intercepted.

**Solutions**:
- Verify policies are enabled: `SELECT * FROM policies WHERE enabled = 1`
- Check policy engine is loaded: Restart backend to reload policies
- Verify request routing through OctoGuard proxy
- Check pattern matching in logs

### High False Positive Rate

**Issue**: Too many legitimate requests being blocked.

**Solutions**:
- Review and refine regex patterns for specificity
- Use `alert` action instead of `block` for testing
- Add whitelist exceptions to policies
- Lower severity thresholds

### Alerts Not Sending

**Issue**: Configured alerts are not being delivered.

**Solutions**:
- Verify webhook URLs are correct and accessible
- Check SMTP credentials and network access
- Review alert channel enabled status
- Check application logs for delivery errors
- Test webhooks independently with curl

### Token Monitoring Inaccurate

**Issue**: Token counts don't match expected usage.

**Solutions**:
- Ensure all OpenClaw requests flow through OctoGuard
- Verify token extraction from OpenClaw responses
- Check database connection and query execution
- Review token calculation logic for specific models

### Performance Degradation

**Issue**: OctoGuard adds significant latency.

**Solutions**:
- Enable policy caching: Refresh policies periodically not per-request
- Optimize database queries with indexes
- Use connection pooling for database
- Consider async logging for audit events
- Profile and optimize regex patterns

### Database Connection Issues

**Issue**: Cannot connect to MySQL database.

**Solutions**:
```javascript
// Test database connection
const mysql = require('mysql2/promise');

async function testConnection() {
  try {
    const connection = await mysql.createConnection({
      host: process.env.DB_HOST,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME
    });
    console.log('Database connected successfully');
    await connection.end();
  } catch (error) {
    console.error('Database connection failed:', error.message);
  }
}

testConnection();
```

## Best Practices

1. **Start with Alert Mode**: Use `alert` action initially to tune policies before enabling `block`
2. **Regular Policy Review**: Audit and update policies based on logged events
3. **Layered Defense**: Combine multiple policy types (keyword, regex, path) for comprehensive coverage
4. **Monitor Performance**: Track policy evaluation latency and optimize slow patterns
5. **Secure Credentials**: Always use environment variables for sensitive configuration
6. **Backup Policies**: Export and version control policy configurations
7. **Test Webhooks**: Verify alert delivery before relying on notifications in production
8. **Archive Logs**: Implement log rotation and archival for compliance requirements
