---
name: octoguard-openclaw-security-governance
description: Security governance and audit system for OpenClaw AI agents with real-time policy enforcement, token monitoring, and alert notifications
triggers:
  - how do I secure my OpenClaw AI agent
  - set up security policies for OpenClaw
  - monitor AI agent behavior and token usage
  - implement OctoGuard security system
  - configure security rules for AI employees
  - audit OpenClaw tool calls and conversations
  - block dangerous AI operations with OctoGuard
  - integrate security supervision for OpenClaw
---

# OctoGuard OpenClaw Security Governance

> Skill by [ara.so](https://ara.so) — Security Skills collection.

OctoGuard (章鱼卫士) is a comprehensive security governance and audit system designed for OpenClaw AI agents. It provides real-time detection, policy control, and alert notifications for user conversations, tool calls, and execution results. The system acts as a security layer that monitors and controls AI agent behavior without modifying the underlying OpenClaw implementation.

## What OctoGuard Does

- **Real-time Policy Enforcement**: Intercepts and blocks high-risk behaviors (sensitive file access, dangerous operations) before execution
- **Token Monitoring**: Tracks total usage, session details, and threshold-based alerting for token consumption
- **Audit Logging**: Records all events processed through OpenClaw including allowed and blocked actions
- **Visual Policy Management**: Web UI for viewing, enabling/disabling, and managing security policies
- **OpenClaw Control**: Monitor gateway status and control OpenClaw runtime
- **Alert Notifications**: Push notifications via DingTalk, Enterprise WeChat, and Email when security events occur
- **Security Dashboard**: Visualization of security posture and complete AI agent behavior tracking

## Architecture

OctoGuard operates as a proxy/gateway layer:
```
External User → OpenClaw Gateway → OctoGuard Policy Engine → OpenClaw → AI Model
                                         ↓
                                    Audit Log + Alerts
```

The system is non-invasive - it wraps OpenClaw without modifying its core functionality.

## Installation

### Prerequisites

- Node.js 14+ and npm/yarn
- MySQL 5.7+ or compatible database
- Running OpenClaw instance
- (Optional) DingTalk/WeChat webhook for alerts

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/O-ozzz/OctoGuard--Free-OpenClaw-Security-Supervision-System.git
cd OctoGuard--Free-OpenClaw-Security-Supervision-System/backend

# Install dependencies
npm install

# Configure database connection
# Edit config/database.js
cat > config/database.js << EOF
module.exports = {
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 3306,
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'octoguard'
};
EOF

# Initialize database
mysql -u root -p < schema/init.sql

# Configure OpenClaw gateway path
# Edit config/openclaw.js
cat > config/openclaw.js << EOF
module.exports = {
  gatewayUrl: process.env.OPENCLAW_GATEWAY || 'http://localhost:3000',
  interceptPath: process.env.INTERCEPT_PATH || '/api/intercept'
};
EOF

# Start backend server
npm start
# Default port: 8080
```

### Frontend Setup

```bash
# Navigate to frontend directory
cd ../frontend

# Install dependencies
npm install

# Configure API endpoint
# Edit .env
cat > .env << EOF
VITE_API_BASE_URL=http://localhost:8080
EOF

# Build and serve
npm run build
npm run preview
# Or for development
npm run dev
```

## Database Schema

```sql
-- Security policies table
CREATE TABLE security_policies (
  id INT AUTO_INCREMENT PRIMARY KEY,
  policy_name VARCHAR(255) NOT NULL,
  policy_type VARCHAR(50) NOT NULL,
  pattern VARCHAR(500) NOT NULL,
  action VARCHAR(50) DEFAULT 'block',
  enabled BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Audit logs table
CREATE TABLE audit_logs (
  id INT AUTO_INCREMENT PRIMARY KEY,
  session_id VARCHAR(255),
  user_input TEXT,
  tool_name VARCHAR(255),
  tool_params JSON,
  result TEXT,
  action_taken VARCHAR(50),
  policy_matched VARCHAR(255),
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  risk_level VARCHAR(50)
);

-- Token usage tracking
CREATE TABLE token_usage (
  id INT AUTO_INCREMENT PRIMARY KEY,
  session_id VARCHAR(255),
  prompt_tokens INT,
  completion_tokens INT,
  total_tokens INT,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Core API Usage

### Policy Management API

```javascript
// backend/routes/policies.js
const express = require('express');
const router = express.Router();
const db = require('../config/database');

// Create a new security policy
router.post('/policies', async (req, res) => {
  const { policy_name, policy_type, pattern, action, enabled } = req.body;
  
  try {
    const query = `
      INSERT INTO security_policies 
      (policy_name, policy_type, pattern, action, enabled) 
      VALUES (?, ?, ?, ?, ?)
    `;
    
    const result = await db.execute(query, [
      policy_name, 
      policy_type, 
      pattern, 
      action || 'block', 
      enabled !== false
    ]);
    
    res.json({ 
      success: true, 
      policy_id: result.insertId 
    });
  } catch (error) {
    res.status(500).json({ 
      success: false, 
      error: error.message 
    });
  }
});

// Get all policies
router.get('/policies', async (req, res) => {
  try {
    const [policies] = await db.query(
      'SELECT * FROM security_policies ORDER BY created_at DESC'
    );
    res.json({ success: true, data: policies });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Toggle policy status
router.patch('/policies/:id/toggle', async (req, res) => {
  const { id } = req.params;
  
  try {
    await db.execute(
      'UPDATE security_policies SET enabled = NOT enabled WHERE id = ?',
      [id]
    );
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Delete policy
router.delete('/policies/:id', async (req, res) => {
  const { id } = req.params;
  
  try {
    await db.execute('DELETE FROM security_policies WHERE id = ?', [id]);
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

module.exports = router;
```

### Policy Engine Implementation

```javascript
// backend/engine/policyEngine.js
const db = require('../config/database');
const { sendAlert } = require('./alertService');

class PolicyEngine {
  async checkPolicy(userInput, toolName, toolParams) {
    // Fetch all enabled policies
    const [policies] = await db.query(
      'SELECT * FROM security_policies WHERE enabled = true'
    );
    
    for (const policy of policies) {
      const violated = this.evaluatePolicy(
        policy, 
        userInput, 
        toolName, 
        toolParams
      );
      
      if (violated) {
        return {
          allowed: false,
          policy_matched: policy.policy_name,
          action: policy.action,
          risk_level: this.calculateRiskLevel(policy.policy_type)
        };
      }
    }
    
    return { allowed: true, policy_matched: null };
  }
  
  evaluatePolicy(policy, userInput, toolName, toolParams) {
    switch (policy.policy_type) {
      case 'keyword':
        return this.checkKeyword(policy.pattern, userInput);
      
      case 'file_path':
        return this.checkFilePath(policy.pattern, toolParams);
      
      case 'tool_restriction':
        return this.checkToolRestriction(policy.pattern, toolName);
      
      case 'regex':
        return this.checkRegex(policy.pattern, userInput);
      
      default:
        return false;
    }
  }
  
  checkKeyword(pattern, text) {
    const keywords = pattern.split(',').map(k => k.trim().toLowerCase());
    const lowerText = text.toLowerCase();
    return keywords.some(keyword => lowerText.includes(keyword));
  }
  
  checkFilePath(pattern, params) {
    if (!params || !params.path) return false;
    
    const restrictedPaths = pattern.split(',').map(p => p.trim());
    return restrictedPaths.some(path => 
      params.path.includes(path) || params.path.startsWith(path)
    );
  }
  
  checkToolRestriction(pattern, toolName) {
    const restrictedTools = pattern.split(',').map(t => t.trim().toLowerCase());
    return restrictedTools.includes(toolName.toLowerCase());
  }
  
  checkRegex(pattern, text) {
    try {
      const regex = new RegExp(pattern, 'i');
      return regex.test(text);
    } catch (error) {
      console.error('Invalid regex pattern:', error);
      return false;
    }
  }
  
  calculateRiskLevel(policyType) {
    const riskMap = {
      'file_path': 'high',
      'tool_restriction': 'high',
      'keyword': 'medium',
      'regex': 'medium'
    };
    return riskMap[policyType] || 'low';
  }
}

module.exports = new PolicyEngine();
```

### Interception Middleware

```javascript
// backend/middleware/interceptor.js
const policyEngine = require('../engine/policyEngine');
const auditLogger = require('../services/auditLogger');
const tokenTracker = require('../services/tokenTracker');

async function interceptRequest(req, res, next) {
  const { user_input, tool_name, tool_params, session_id } = req.body;
  
  try {
    // Check against security policies
    const policyCheck = await policyEngine.checkPolicy(
      user_input,
      tool_name,
      tool_params
    );
    
    if (!policyCheck.allowed) {
      // Log blocked action
      await auditLogger.log({
        session_id,
        user_input,
        tool_name,
        tool_params,
        action_taken: 'blocked',
        policy_matched: policyCheck.policy_matched,
        risk_level: policyCheck.risk_level
      });
      
      // Send alert
      await sendAlert({
        type: 'security_violation',
        policy: policyCheck.policy_matched,
        user_input,
        tool_name,
        risk_level: policyCheck.risk_level
      });
      
      return res.status(403).json({
        success: false,
        message: '操作已被安全策略拦截',
        policy: policyCheck.policy_matched
      });
    }
    
    // Track token usage (before forwarding)
    await tokenTracker.estimateUsage(session_id, user_input);
    
    // Log allowed action
    await auditLogger.log({
      session_id,
      user_input,
      tool_name,
      tool_params,
      action_taken: 'allowed',
      policy_matched: null
    });
    
    // Forward to OpenClaw
    next();
  } catch (error) {
    console.error('Interception error:', error);
    res.status(500).json({ 
      success: false, 
      error: 'Internal security check failed' 
    });
  }
}

module.exports = { interceptRequest };
```

### Alert Service

```javascript
// backend/services/alertService.js
const axios = require('axios');
const nodemailer = require('nodemailer');

class AlertService {
  async sendAlert(alertData) {
    const { type, policy, user_input, tool_name, risk_level } = alertData;
    
    // Send to DingTalk
    if (process.env.DINGTALK_WEBHOOK) {
      await this.sendDingTalk(alertData);
    }
    
    // Send to Enterprise WeChat
    if (process.env.WECHAT_WEBHOOK) {
      await this.sendWeChat(alertData);
    }
    
    // Send email
    if (process.env.EMAIL_ENABLED === 'true') {
      await this.sendEmail(alertData);
    }
  }
  
  async sendDingTalk(alertData) {
    const message = {
      msgtype: 'markdown',
      markdown: {
        title: '🚨 OctoGuard 安全告警',
        text: `### 安全策略拦截通知\n\n` +
              `**风险等级**: ${alertData.risk_level}\n\n` +
              `**触发策略**: ${alertData.policy}\n\n` +
              `**用户输入**: ${alertData.user_input}\n\n` +
              `**工具名称**: ${alertData.tool_name}\n\n` +
              `**时间**: ${new Date().toLocaleString('zh-CN')}`
      }
    };
    
    try {
      await axios.post(process.env.DINGTALK_WEBHOOK, message);
    } catch (error) {
      console.error('DingTalk alert failed:', error);
    }
  }
  
  async sendWeChat(alertData) {
    const message = {
      msgtype: 'markdown',
      markdown: {
        content: `## OctoGuard 安全告警\n` +
                `风险等级: <font color="warning">${alertData.risk_level}</font>\n` +
                `策略: ${alertData.policy}\n` +
                `工具: ${alertData.tool_name}`
      }
    };
    
    try {
      await axios.post(process.env.WECHAT_WEBHOOK, message);
    } catch (error) {
      console.error('WeChat alert failed:', error);
    }
  }
  
  async sendEmail(alertData) {
    const transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: process.env.SMTP_PORT,
      secure: true,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASSWORD
      }
    });
    
    const mailOptions = {
      from: process.env.SMTP_USER,
      to: process.env.ALERT_EMAIL,
      subject: '🚨 OctoGuard Security Alert',
      html: `
        <h2>Security Policy Violation Detected</h2>
        <p><strong>Risk Level:</strong> ${alertData.risk_level}</p>
        <p><strong>Policy:</strong> ${alertData.policy}</p>
        <p><strong>Tool:</strong> ${alertData.tool_name}</p>
        <p><strong>User Input:</strong> ${alertData.user_input}</p>
        <p><strong>Time:</strong> ${new Date().toLocaleString()}</p>
      `
    };
    
    try {
      await transporter.sendMail(mailOptions);
    } catch (error) {
      console.error('Email alert failed:', error);
    }
  }
}

module.exports = new AlertService();
```

### Token Monitoring

```javascript
// backend/services/tokenTracker.js
const db = require('../config/database');
const { sendAlert } = require('./alertService');

class TokenTracker {
  async trackUsage(session_id, prompt_tokens, completion_tokens) {
    const total_tokens = prompt_tokens + completion_tokens;
    
    await db.execute(
      `INSERT INTO token_usage 
       (session_id, prompt_tokens, completion_tokens, total_tokens) 
       VALUES (?, ?, ?, ?)`,
      [session_id, prompt_tokens, completion_tokens, total_tokens]
    );
    
    // Check threshold
    await this.checkThreshold(session_id);
  }
  
  async checkThreshold(session_id) {
    const threshold = parseInt(process.env.TOKEN_THRESHOLD || '100000');
    
    const [result] = await db.query(
      'SELECT SUM(total_tokens) as total FROM token_usage WHERE session_id = ?',
      [session_id]
    );
    
    if (result[0].total >= threshold) {
      await sendAlert({
        type: 'token_threshold',
        session_id,
        total_tokens: result[0].total,
        threshold
      });
    }
  }
  
  async getSessionUsage(session_id) {
    const [rows] = await db.query(
      'SELECT * FROM token_usage WHERE session_id = ? ORDER BY timestamp DESC',
      [session_id]
    );
    return rows;
  }
  
  async getTotalUsage() {
    const [result] = await db.query(
      'SELECT SUM(total_tokens) as total FROM token_usage'
    );
    return result[0].total || 0;
  }
}

module.exports = new TokenTracker();
```

## Configuration Examples

### Common Security Policies

```javascript
// Block sensitive file access
{
  policy_name: "Block System Files",
  policy_type: "file_path",
  pattern: "/etc,/root,/var/log,C:\\Windows",
  action: "block",
  enabled: true
}

// Block dangerous keywords
{
  policy_name: "Sensitive Data Keywords",
  policy_type: "keyword",
  pattern: "password,secret,api_key,token,credit_card",
  action: "block",
  enabled: true
}

// Restrict dangerous tools
{
  policy_name: "Dangerous Tool Restriction",
  policy_type: "tool_restriction",
  pattern: "shell_exec,system_command,file_delete",
  action: "block",
  enabled: true
}

// Regex pattern matching
{
  policy_name: "SQL Injection Detection",
  policy_type: "regex",
  pattern: "(DROP|DELETE|UPDATE)\\s+(TABLE|FROM|SET)",
  action: "block",
  enabled: true
}
```

### Environment Variables

```bash
# Database
DB_HOST=localhost
DB_PORT=3306
DB_USER=octoguard
DB_PASSWORD=your_secure_password
DB_NAME=octoguard

# OpenClaw
OPENCLAW_GATEWAY=http://localhost:3000
INTERCEPT_PATH=/api/intercept

# Token Monitoring
TOKEN_THRESHOLD=100000

# Alert Webhooks
DINGTALK_WEBHOOK=https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN
WECHAT_WEBHOOK=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY

# Email Alerts
EMAIL_ENABLED=true
SMTP_HOST=smtp.gmail.com
SMTP_PORT=465
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your_app_password
ALERT_EMAIL=security-team@company.com
```

## Troubleshooting

### Policy Not Triggering

**Issue**: Security policy exists but doesn't block requests

**Solutions**:
```javascript
// 1. Verify policy is enabled
const [policy] = await db.query(
  'SELECT enabled FROM security_policies WHERE id = ?', 
  [policyId]
);
console.log('Policy enabled:', policy[0].enabled);

// 2. Check pattern syntax
// For file_path policies, use absolute paths
// For keyword policies, use comma-separated lowercase terms

// 3. Test pattern matching manually
const policyEngine = require('./engine/policyEngine');
const result = policyEngine.checkKeyword('password,secret', 'my password is 123');
console.log('Should be true:', result);
```

### Alert Not Sending

**Issue**: Security events logged but no alerts received

**Solutions**:
```javascript
// 1. Test webhook directly
const axios = require('axios');
await axios.post(process.env.DINGTALK_WEBHOOK, {
  msgtype: 'text',
  text: { content: 'Test alert from OctoGuard' }
});

// 2. Check environment variables
console.log('DingTalk webhook:', process.env.DINGTALK_WEBHOOK);
console.log('Email enabled:', process.env.EMAIL_ENABLED);

// 3. Verify alertService is called
const { sendAlert } = require('./services/alertService');
await sendAlert({
  type: 'test',
  policy: 'Test Policy',
  user_input: 'Test input',
  tool_name: 'test_tool',
  risk_level: 'low'
});
```

### Database Connection Issues

```javascript
// Test database connection
const db = require('./config/database');

async function testConnection() {
  try {
    const [rows] = await db.query('SELECT 1');
    console.log('Database connected successfully');
  } catch (error) {
    console.error('Database connection failed:', error.message);
    // Check: credentials, network, MySQL service status
  }
}

testConnection();
```

### High Token Usage Not Alerting

```javascript
// Manually check token threshold
const tokenTracker = require('./services/tokenTracker');

async function checkTokens(sessionId) {
  const usage = await tokenTracker.getSessionUsage(sessionId);
  const total = usage.reduce((sum, row) => sum + row.total_tokens, 0);
  const threshold = parseInt(process.env.TOKEN_THRESHOLD);
  
  console.log(`Total: ${total}, Threshold: ${threshold}`);
  
  if (total >= threshold) {
    console.log('Should trigger alert');
  }
}
```

## Best Practices

1. **Layered Policies**: Start with broad keyword policies, then add specific file_path and tool restrictions
2. **Test Mode**: Deploy with `action: "log"` first to avoid blocking legitimate operations
3. **Alert Fatigue**: Set appropriate thresholds to avoid excessive notifications
4. **Regular Audits**: Review audit logs weekly to identify emerging patterns
5. **Backup Policies**: Export policy configurations regularly for disaster recovery

```javascript
// Export all policies
router.get('/policies/export', async (req, res) => {
  const [policies] = await db.query('SELECT * FROM security_policies');
  res.json({ policies, exported_at: new Date().toISOString() });
});
```
