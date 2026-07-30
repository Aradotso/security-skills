---
name: api-security-checklist
description: Comprehensive API security checklist and best practices for designing, testing, and securing REST, GraphQL, and OAuth APIs
triggers:
  - "show me API security best practices"
  - "how do I secure my REST API"
  - "what security checks should I implement for my API"
  - "help me review API security"
  - "OAuth security recommendations"
  - "API authentication and authorization checklist"
  - "protect my API from attacks"
  - "API security audit checklist"
---

# API Security Checklist Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

This skill provides comprehensive guidance on API security best practices based on the widely-adopted API Security Checklist. Use this to design, audit, and secure REST, GraphQL, and OAuth APIs against common vulnerabilities and attack vectors.

## Overview

The API Security Checklist covers critical security countermeasures across:
- **Authentication** - Secure user identity verification
- **Authorization** - Access control and OAuth flows
- **Input Validation** - Preventing injection attacks
- **Output Security** - Secure response handling
- **Processing** - Backend security measures
- **Monitoring** - Detection and alerting
- **CI/CD** - Secure development lifecycle

## Installation

This is a knowledge resource, not a software package. To use:

1. **Bookmark for reference**: Keep the checklist accessible during API development
2. **Integrate into code reviews**: Use as a PR checklist template
3. **Add to CI/CD**: Implement automated checks based on these guidelines
4. **Security audits**: Use as an audit framework

## Authentication Security

### ❌ Avoid Basic Auth

```javascript
// BAD - Basic Auth is insecure
app.get('/api/users', (req, res) => {
  const auth = req.headers.authorization;
  const [user, pass] = Buffer.from(auth.split(' ')[1], 'base64').toString().split(':');
  // Don't do this!
});
```

### ✅ Use Standard Authentication

```javascript
// GOOD - JWT with proper validation
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
  const token = req.headers['authorization']?.split(' ')[1];
  
  if (!token) return res.sendStatus(401);
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
}

app.get('/api/users', authenticateToken, (req, res) => {
  res.json({ user: req.user });
});
```

### Rate Limiting and Max Retry

```javascript
const rateLimit = require('express-rate-limit');

// Limit login attempts
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

app.post('/api/login', loginLimiter, async (req, res) => {
  // Login logic
});
```

### Password Storage

```javascript
const bcrypt = require('bcrypt');

// GOOD - Hash passwords with bcrypt
async function hashPassword(password) {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
}

async function verifyPassword(password, hash) {
  return await bcrypt.compare(password, hash);
}

// Usage
app.post('/api/register', async (req, res) => {
  const { email, password } = req.body;
  const hashedPassword = await hashPassword(password);
  // Store hashedPassword in database
});
```

## Access Control

### HTTPS and Security Headers

```javascript
const helmet = require('helmet');
const express = require('express');

const app = express();

// Use Helmet for security headers
app.use(helmet({
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'none'"]
    }
  },
  frameguard: { action: 'deny' },
  noSniff: true
}));

// Force HTTPS in production
if (process.env.NODE_ENV === 'production') {
  app.use((req, res, next) => {
    if (!req.secure) {
      return res.redirect('https://' + req.headers.host + req.url);
    }
    next();
  });
}
```

### IP Whitelisting for Private APIs

```javascript
const ipWhitelist = process.env.ALLOWED_IPS?.split(',') || [];

function checkIPWhitelist(req, res, next) {
  const clientIP = req.ip || req.connection.remoteAddress;
  
  if (!ipWhitelist.includes(clientIP)) {
    return res.status(403).json({ error: 'IP not authorized' });
  }
  
  next();
}

app.use('/api/admin', checkIPWhitelist);
```

### DDoS Protection

```javascript
const rateLimit = require('express-rate-limit');

// General API rate limiting
const apiLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
  message: 'Too many requests from this IP'
});

app.use('/api/', apiLimiter);
```

## OAuth Security

### Validate Redirect URI

```python
from urllib.parse import urlparse

ALLOWED_REDIRECT_URIS = [
    'https://app.example.com/callback',
    'https://app.example.com/oauth/callback'
]

def validate_redirect_uri(redirect_uri):
    """Always validate redirect_uri server-side"""
    if redirect_uri not in ALLOWED_REDIRECT_URIS:
        raise ValueError('Invalid redirect_uri')
    return True

# In OAuth authorization endpoint
@app.route('/oauth/authorize')
def authorize():
    redirect_uri = request.args.get('redirect_uri')
    
    try:
        validate_redirect_uri(redirect_uri)
    except ValueError:
        return {'error': 'invalid_redirect_uri'}, 400
    
    # Continue with authorization
```

### Use State Parameter for CSRF Protection

```javascript
const crypto = require('crypto');

// Generate state parameter
function generateState() {
  return crypto.randomBytes(32).toString('hex');
}

// OAuth authorization request
app.get('/oauth/login', (req, res) => {
  const state = generateState();
  
  // Store state in session
  req.session.oauthState = state;
  
  const authUrl = `https://provider.com/oauth/authorize?` +
    `client_id=${process.env.OAUTH_CLIENT_ID}` +
    `&redirect_uri=${encodeURIComponent(process.env.OAUTH_REDIRECT_URI)}` +
    `&response_type=code` +
    `&state=${state}` +
    `&scope=read`;
  
  res.redirect(authUrl);
});

// OAuth callback - validate state
app.get('/oauth/callback', (req, res) => {
  const { code, state } = req.query;
  
  // Validate state parameter
  if (state !== req.session.oauthState) {
    return res.status(403).json({ error: 'Invalid state parameter' });
  }
  
  // Exchange code for token
  // Never use response_type=token (implicit flow)
});
```

### Scope Validation

```javascript
const VALID_SCOPES = ['read', 'write', 'admin'];
const DEFAULT_SCOPE = 'read';

function validateScopes(requestedScopes) {
  if (!requestedScopes) return [DEFAULT_SCOPE];
  
  const scopes = requestedScopes.split(' ');
  const validScopes = scopes.filter(scope => VALID_SCOPES.includes(scope));
  
  return validScopes.length > 0 ? validScopes : [DEFAULT_SCOPE];
}

app.post('/oauth/token', (req, res) => {
  const requestedScopes = req.body.scope;
  const allowedScopes = validateScopes(requestedScopes);
  
  // Generate token with validated scopes only
  const token = jwt.sign(
    { scopes: allowedScopes },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );
  
  res.json({ access_token: token, scope: allowedScopes.join(' ') });
});
```

## Input Validation

### HTTP Method Validation

```javascript
const ALLOWED_METHODS = {
  '/api/users': ['GET', 'POST'],
  '/api/users/:id': ['GET', 'PUT', 'PATCH', 'DELETE']
};

function validateMethod(req, res, next) {
  const allowedForRoute = ALLOWED_METHODS[req.route.path];
  
  if (!allowedForRoute || !allowedForRoute.includes(req.method)) {
    res.set('Allow', allowedForRoute.join(', '));
    return res.status(405).json({ error: 'Method Not Allowed' });
  }
  
  next();
}

app.use(validateMethod);
```

### Content-Type Validation

```javascript
const SUPPORTED_CONTENT_TYPES = [
  'application/json',
  'application/xml'
];

function validateContentType(req, res, next) {
  // Validate Accept header
  const accept = req.headers.accept;
  const acceptsSupported = SUPPORTED_CONTENT_TYPES.some(type => 
    accept?.includes(type)
  );
  
  if (!acceptsSupported && accept !== '*/*') {
    return res.status(406).json({ error: 'Not Acceptable' });
  }
  
  // Validate Content-Type for POST/PUT/PATCH
  if (['POST', 'PUT', 'PATCH'].includes(req.method)) {
    const contentType = req.headers['content-type']?.split(';')[0];
    
    if (!SUPPORTED_CONTENT_TYPES.includes(contentType)) {
      return res.status(415).json({ error: 'Unsupported Media Type' });
    }
  }
  
  next();
}

app.use(validateContentType);
```

### Input Sanitization

```javascript
const validator = require('validator');

// Prevent XSS, SQL Injection, etc.
function sanitizeInput(data) {
  if (typeof data === 'string') {
    return validator.escape(data);
  }
  
  if (Array.isArray(data)) {
    return data.map(sanitizeInput);
  }
  
  if (typeof data === 'object' && data !== null) {
    const sanitized = {};
    for (const [key, value] of Object.entries(data)) {
      sanitized[key] = sanitizeInput(value);
    }
    return sanitized;
  }
  
  return data;
}

app.post('/api/users', (req, res) => {
  const sanitizedBody = sanitizeInput(req.body);
  // Use sanitizedBody instead of req.body
});
```

### Prevent XXE (XML External Entity)

```javascript
const libxmljs = require('libxmljs');

function parseXMLSafely(xmlString) {
  try {
    // Disable external entity parsing
    const doc = libxmljs.parseXml(xmlString, {
      noent: false,  // Don't substitute entities
      nonet: true,   // Don't access network
      dtdload: false // Don't load external DTDs
    });
    return doc;
  } catch (error) {
    throw new Error('Invalid XML');
  }
}

app.post('/api/data', (req, res) => {
  if (req.headers['content-type'] === 'application/xml') {
    try {
      const doc = parseXMLSafely(req.body);
      // Process document
    } catch (error) {
      return res.status(400).json({ error: 'Invalid XML' });
    }
  }
});
```

## Processing Security

### Avoid Auto-Increment IDs (Use UUIDs)

```javascript
const { v4: uuidv4 } = require('uuid');

// GOOD - Use UUIDs instead of auto-increment IDs
app.post('/api/users', async (req, res) => {
  const user = {
    id: uuidv4(), // e.g., '9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d'
    ...req.body
  };
  
  await db.users.create(user);
  res.json(user);
});
```

### Use /me for User Resources

```javascript
// BAD - Exposes user IDs
app.get('/api/users/:userId/orders', (req, res) => {
  // User could change userId to access others' data
});

// GOOD - Use /me for authenticated user resources
app.get('/api/me/orders', authenticateToken, async (req, res) => {
  const orders = await db.orders.find({ userId: req.user.id });
  res.json(orders);
});
```

### Background Processing with Workers

```javascript
const Queue = require('bull');
const uploadQueue = new Queue('file-uploads', process.env.REDIS_URL);

// Don't block HTTP response with heavy processing
app.post('/api/upload', async (req, res) => {
  const { file } = req.body;
  
  // Add to queue immediately
  const job = await uploadQueue.add({
    fileId: file.id,
    userId: req.user.id
  });
  
  // Return fast response
  res.status(202).json({
    message: 'Upload processing',
    jobId: job.id
  });
});

// Process in background worker
uploadQueue.process(async (job) => {
  const { fileId, userId } = job.data;
  // Heavy processing here
  await processLargeFile(fileId);
});
```

### Disable Debug Mode in Production

```javascript
// Check environment
if (process.env.NODE_ENV === 'production') {
  // Disable verbose error messages
  app.use((err, req, res, next) => {
    console.error(err.stack); // Log server-side only
    
    res.status(500).json({
      error: 'Internal Server Error' // Generic message
    });
  });
} else {
  // Development - show detailed errors
  app.use((err, req, res, next) => {
    res.status(500).json({
      error: err.message,
      stack: err.stack
    });
  });
}
```

## Output Security

### Security Headers

```javascript
app.use((req, res, next) => {
  // Prevent MIME sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'deny');
  
  // Content Security Policy
  res.setHeader('Content-Security-Policy', "default-src 'none'");
  
  // Remove fingerprinting headers
  res.removeHeader('X-Powered-By');
  
  next();
});
```

### Force Content-Type

```javascript
app.get('/api/users', (req, res) => {
  const users = [{ id: 1, name: 'John' }];
  
  // Always set explicit content-type
  res.setHeader('Content-Type', 'application/json');
  res.json(users);
});
```

### Generic Error Messages

```python
import logging

logger = logging.getLogger(__name__)

@app.errorhandler(Exception)
def handle_error(error):
    # Log detailed error server-side
    logger.error(f"Error occurred: {str(error)}", exc_info=True)
    
    # Return generic message to client
    return {
        'error': 'An error occurred processing your request'
    }, 500

# Don't return sensitive details
@app.route('/api/users/<user_id>')
def get_user(user_id):
    try:
        user = db.query(f"SELECT * FROM users WHERE id = ?", [user_id])
        return jsonify(user)
    except DatabaseError as e:
        # BAD - exposes database structure
        # return {'error': f'Database error: {str(e)}'}, 500
        
        # GOOD - generic message
        logger.error(f"Database error for user {user_id}: {str(e)}")
        return {'error': 'Unable to retrieve user'}, 500
```

### Proper Status Codes

```javascript
app.post('/api/users', async (req, res) => {
  try {
    const user = await createUser(req.body);
    res.status(201).json(user); // 201 Created
  } catch (error) {
    if (error.type === 'VALIDATION_ERROR') {
      res.status(400).json({ error: error.message }); // 400 Bad Request
    } else if (error.type === 'DUPLICATE') {
      res.status(409).json({ error: 'User already exists' }); // 409 Conflict
    } else {
      res.status(500).json({ error: 'Internal Server Error' });
    }
  }
});

app.delete('/api/users/:id', authenticateToken, async (req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' }); // 403 Forbidden
  }
  
  const deleted = await deleteUser(req.params.id);
  
  if (!deleted) {
    return res.status(404).json({ error: 'User not found' }); // 404 Not Found
  }
  
  res.status(204).send(); // 204 No Content
});
```

## GraphQL-Specific Security

### Disable Introspection in Production

```javascript
const { ApolloServer } = require('apollo-server');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: process.env.NODE_ENV !== 'production',
  playground: process.env.NODE_ENV !== 'production'
});
```

### Query Depth Limiting

```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)] // Max depth of 5
});
```

### Query Cost Analysis

```javascript
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    createComplexityLimitRule(1000, {
      onCost: (cost) => {
        console.log('Query cost:', cost);
      }
    })
  ]
});
```

## Monitoring and Logging

### Centralized Logging

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Log all requests
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    path: req.path,
    ip: req.ip,
    userAgent: req.headers['user-agent']
  });
  next();
});

// Don't log sensitive data
app.post('/api/login', (req, res) => {
  logger.info({
    event: 'login_attempt',
    email: req.body.email
    // DON'T log password
  });
});
```

### Alert on Suspicious Activity

```javascript
const alertThreshold = 10;
const suspiciousIPs = new Map();

app.use((req, res, next) => {
  const ip = req.ip;
  const count = suspiciousIPs.get(ip) || 0;
  
  if (res.statusCode === 401 || res.statusCode === 403) {
    suspiciousIPs.set(ip, count + 1);
    
    if (count + 1 >= alertThreshold) {
      // Send alert
      sendAlert({
        type: 'suspicious_activity',
        ip: ip,
        failedAttempts: count + 1
      });
    }
  }
  
  next();
});

function sendAlert(alert) {
  // Send to Slack, email, SMS, etc.
  console.log('ALERT:', alert);
}
```

## CI/CD Security

### Dependency Scanning

```yaml
# .github/workflows/security.yml
name: Security Checks

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run dependency audit
        run: npm audit --audit-level=moderate
      
      - name: Check for known vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### Static Code Analysis

```yaml
      - name: Run static analysis
        run: |
          npm install -g eslint eslint-plugin-security
          eslint . --ext .js --plugin security
```

### Secret Scanning

```yaml
      - name: Scan for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
```

## Common Patterns and Best Practices

### API Key Management

```javascript
const crypto = require('crypto');

// Generate API key
function generateAPIKey() {
  return crypto.randomBytes(32).toString('hex');
}

// Store hashed API key
async function createAPIKey(userId) {
  const apiKey = generateAPIKey();
  const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
  
  await db.apiKeys.create({
    userId,
    keyHash: hashedKey,
    createdAt: new Date()
  });
  
  // Return plain key only once
  return apiKey;
}

// Validate API key
async function validateAPIKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
  const keyRecord = await db.apiKeys.findOne({ keyHash: hashedKey });
  
  if (!keyRecord) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  req.user = { id: keyRecord.userId };
  next();
}
```

### CORS Configuration

```javascript
const cors = require('cors');

const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    
    if (!origin || allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  maxAge: 86400 // 24 hours
};

app.use(cors(corsOptions));
```

### Request Signing

```javascript
const crypto = require('crypto');

// Sign sensitive requests
function signRequest(payload, secret) {
  const signature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return signature;
}

// Verify request signature
function verifySignature(req, res, next) {
  const signature = req.headers['x-signature'];
  const timestamp = req.headers['x-timestamp'];
  
  // Prevent replay attacks (5 minute window)
  if (Date.now() - parseInt(timestamp) > 300000) {
    return res.status(401).json({ error: 'Request expired' });
  }
  
  const payload = { ...req.body, timestamp };
  const expectedSignature = signRequest(payload, process.env.SIGNING_SECRET);
  
  if (signature !== expectedSignature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  next();
}
```

## Troubleshooting

### Issue: Rate limiting blocking legitimate users

**Solution**: Implement sliding window with user identification

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const limiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
  }),
  windowMs: 15 * 60 * 1000,
  max: async (req) => {
    // Authenticated users get higher limits
    if (req.user) return 1000;
    return 100;
  },
  keyGenerator: (req) => {
    // Use user ID for authenticated, IP for anonymous
    return req.user ? req.user.id : req.ip;
  }
});
```

### Issue: CORS errors in production

**Solution**: Properly configure CORS with environment-specific origins

```javascript
const allowedOrigins = {
  development: ['http://localhost:3000'],
  production: ['https://app.example.com', 'https://www.example.com']
};

const origins = allowedOrigins[process.env.NODE_ENV] || [];
```

### Issue: Token expiration causing user logouts

**Solution**: Implement refresh token pattern

```javascript
function generateTokens(userId) {
  const accessToken = jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: '15m' } // Short-lived
  );
  
  const refreshToken = jwt.sign(
    { userId },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' } // Long-lived
  );
  
  return { accessToken, refreshToken };
}

app.post('/api/refresh', (req, res) => {
  const { refreshToken } = req.body;
  
  jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    
    const { accessToken, refreshToken: newRefreshToken } = generateTokens(user.userId);
    
    res.json({ accessToken, refreshToken: newRefreshToken });
  });
});
```

## Security Audit Checklist

Use this checklist when reviewing APIs:

**Authentication**
- [ ] No Basic Auth in use
- [ ] Passwords properly hashed (bcrypt, Argon2)
- [ ] Rate limiting on login endpoints
- [ ] MFA available for sensitive operations

**Authorization**
- [ ] OAuth redirect_uri validation
- [ ] CSRF protection with state parameter
- [ ] Proper scope validation
- [ ] Access control checked on every endpoint

**Input**
- [ ] HTTP method validation
- [ ] Content-Type validation
- [ ] Input sanitization
- [ ] XXE protection for XML
- [ ] No sensitive data in URLs

**Output**
- [ ] Security headers set
- [ ] Generic error messages
- [ ] Proper status codes
- [ ] No sensitive data in responses

**Infrastructure**
- [ ] HTTPS enforced
- [ ] HSTS enabled
- [ ] Rate limiting implemented
- [ ] Debug mode disabled in production

**Monitoring**
- [ ] Centralized logging
- [ ] No sensitive data logged
- [ ] Alerts configured
- [ ] Security events tracked

## Resources

- Original checklist: https://github.com/shieldfy/API-Security-Checklist
- OWASP API Security Top 10: https://owasp.org/www-project-api-security/
- OAuth 2.0 Security Best Practices: https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics
