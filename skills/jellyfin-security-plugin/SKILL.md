---
name: jellyfin-security-plugin
description: Add native two-factor authentication, passkeys, SSO, brute-force protection, and audit logging to Jellyfin media servers via server-side plugin.
triggers:
  - add two-factor authentication to jellyfin
  - set up jellyfin totp or passkeys
  - configure jellyfin sso with oidc
  - protect jellyfin with brute force ip banning
  - implement jellyfin device pairing for tv clients
  - enable jellyfin lan bypass for local networks
  - audit jellyfin login attempts and security events
  - integrate jellyfin with authentik or authelia
---

# Jellyfin Security Plugin

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## What It Does

JellyfinSecurity is a comprehensive authentication and hardening plugin for Jellyfin media servers (10.11+). It adds:

- **Multi-factor authentication**: TOTP (Authy/Google Authenticator), passkeys (WebAuthn/FIDO2), email OTP, and recovery codes
- **Single Sign-On**: OIDC provider integration (Authentik, Authelia, Keycloak, Pocket ID, etc.)
- **Brute-force protection**: IP-based rate limiting and banning with configurable thresholds
- **Device management**: TV device pairing via QR code, trusted browser tokens
- **Network controls**: LAN bypass, per-user IP allowlist, impossible-travel detection
- **Audit logging**: Full login/logout/config-change event tracking
- **Step-up authentication**: Re-verify TOTP/passkey/IdP before sensitive admin actions

Server-side enforcement works with **all** Jellyfin clients (web, Android, iOS, Roku, Fire TV, Kodi) and service integrations (Sonarr, Radarr, Tautulli).

---

## Installation

### Via Jellyfin Admin Dashboard (Recommended)

1. Open Jellyfin → **Admin Dashboard** → **Plugins** → **Repositories**
2. Add repository:
   - **Name**: `JellyfinSecurity`
   - **URL**: `https://raw.githubusercontent.com/ZL154/JellyfinSecurity/main/manifest.json`
3. Go to **Catalog** → search "Security" → Install
4. Restart Jellyfin server

### Manual Installation

```bash
# Download latest release
RELEASE_URL="https://github.com/ZL154/JellyfinSecurity/releases/latest/download/JellyfinSecurity.zip"
PLUGIN_DIR="/var/lib/jellyfin/plugins/JellyfinSecurity"

mkdir -p "$PLUGIN_DIR"
curl -L "$RELEASE_URL" -o /tmp/jellyfin-security.zip
unzip /tmp/jellyfin-security.zip -d "$PLUGIN_DIR"
chown -R jellyfin:jellyfin "$PLUGIN_DIR"

# Restart Jellyfin
systemctl restart jellyfin
```

Verify installation:
```bash
# Check plugin loaded
journalctl -u jellyfin | grep "JellyfinSecurity"
# Should see: "JellyfinSecurity v2.5.8 loaded"
```

---

## Configuration

### Admin Dashboard Settings

Navigate to **Dashboard** → **Plugins** → **JellyfinSecurity** → **Settings**.

#### Core Authentication

```yaml
# Enable/disable 2FA methods
TOTPEnabled: true               # TOTP (Authenticator apps)
PasskeysEnabled: true           # WebAuthn/FIDO2 hardware keys
EmailOTPEnabled: false          # Email one-time passwords
RecoveryCodesEnabled: true      # Backup codes (auto-generated with TOTP)

# OIDC/SSO providers
OIDCProviders:
  - Name: "Authentik"
    ClientID: "jellyfin-client"
    ClientSecret: "${OIDC_CLIENT_SECRET}"  # Use environment variable
    Authority: "https://auth.example.com/application/o/jellyfin/"
    Scopes: "openid profile email groups"
    AllowPrivateEndpoints: true  # For LAN-only IdPs
```

#### Brute-Force Protection

```yaml
BruteForceEnabled: true
MaxFailedAttempts: 5            # Attempts before temporary ban
LockoutDuration: 900            # Seconds (15 minutes)
PermanentBanThreshold: 20       # Failed attempts → permanent ban
```

#### LAN Bypass

```yaml
LANBypassEnabled: true
LANBypassCIDRs:                 # RFC1918 + local ranges
  - "192.168.0.0/16"
  - "10.0.0.0/24"               # Be specific to avoid SEC-H3 guard
  - "172.16.0.0/12"
  - "fd00::/8"                  # IPv6 ULA

TrustedProxyCIDRs:              # Your reverse proxy IPs only
  - "10.0.1.5/32"               # Nginx/Traefik container IP
```

**⚠️ Trusted Proxy Pitfall**: Do NOT set broad ranges like `10.0.0.0/8` in `TrustedProxyCIDRs` — the SEC-H3 guard will refuse LAN bypass for direct clients if their IP falls within a trusted-proxy range but no `X-Forwarded-For` header is present. Use `/32` (single IP) or tight `/24` subnets for your actual reverse proxy.

#### Device Pairing (TV Clients)

```yaml
TVPairingEnabled: true
PairingCodeExpiration: 300      # Seconds (5 minutes)
```

#### Step-Up Authentication

```yaml
StepUpLevel: AllConfigChanges   # Re-verify TOTP/passkey before:
                                # - Plugin settings changes
                                # - User permission changes
                                # - IP allowlist modifications
```

---

## Usage Patterns

### End-User Enrollment (TOTP)

After installing the plugin, users enroll via Jellyfin web UI:

1. **User** → **Profile** → **Two-Factor Authentication**
2. Click **Enable TOTP**
3. Scan QR code with Authy/Google Authenticator
4. Enter 6-digit code to confirm
5. Save 8 recovery codes (required for account recovery)

**No user code changes needed** — the plugin intercepts `/Users/AuthenticateByName` at the middleware level.

### TV Device Pairing (Roku, Fire TV, etc.)

For clients without keyboard input:

```javascript
// TV app requests pairing code
POST /JellyfinSecurity/PairDevice
{
  "DeviceName": "Living Room Roku",
  "DeviceId": "roku-device-12345"
}

// Response contains code + QR URL
{
  "pairingCode": "AB12-CD34",
  "qrCodeUrl": "/JellyfinSecurity/PairDeviceQR?code=AB12-CD34",
  "expiresAt": "2026-06-12T12:05:00Z"
}

// User approves via Admin Dashboard → Pending Pairs
// TV polls for approval:
GET /JellyfinSecurity/CheckPairingStatus?code=AB12-CD34

// After approval, use returned token in X-Device-Token header
```

### Trusted Browser Token (Web Client)

After successful 2FA login, the plugin sets a signed cookie:

```http
Set-Cookie: JellyfinSecurity-TrustedDevice=<hmac-signed-token>; 
            HttpOnly; Secure; SameSite=Strict; Max-Age=7776000
```

Subsequent logins from the same browser skip 2FA for 90 days (configurable). The token is bound to `User-Agent` + IP subnet (configurable prefix length).

### OIDC Sign-In Flow

```csharp
// Plugin auto-registers endpoints at startup
// User clicks "Sign in with Authentik" button on login page

// 1. Initiate OIDC flow
GET /JellyfinSecurity/OIDC/Authorize?providerId=authentik

// 2. User redirects to IdP, signs in, returns to callback
GET /JellyfinSecurity/OIDC/Callback?code=...&state=...

// 3. Plugin exchanges code for tokens, creates/links Jellyfin user
// 4. Sets Jellyfin auth token cookie, redirects to /web/index.html
```

**Userinfo claim mapping** (auto-merged):
- `preferred_username` or `email` → Jellyfin username
- `email` → Jellyfin email
- `name` → Jellyfin display name
- `groups` → Jellyfin user policies (if `SyncGroupsEnabled: true`)

### Per-User IP Allowlist

```yaml
# Admin Dashboard → JellyfinSecurity → Users → Edit User
AllowedIPs:
  - "203.0.113.0/24"    # Office network
  - "198.51.100.42/32"  # Home static IP
```

User cannot authenticate from any IP outside this list. Leave empty to disable IP restrictions for that user.

### Programmatic API Access (Sonarr, Radarr, etc.)

**Option 1: API Key Bypass** (recommended for service integrations)

```yaml
# Admin → JellyfinSecurity → Settings
APIKeyBypassEnabled: true
```

Services using `X-Emby-Token` header are exempt from 2FA. Generate API key in Jellyfin **Dashboard** → **API Keys**.

```bash
curl -H "X-Emby-Token: ${JELLYFIN_API_KEY}" \
     https://jellyfin.example.com/Users/Me
```

**Option 2: Device Token**

Pair device once, then include token in every request:

```bash
# Pair device (one-time)
curl -X POST https://jellyfin.example.com/JellyfinSecurity/PairDevice \
  -H "Content-Type: application/json" \
  -d '{"DeviceName":"Sonarr","DeviceId":"sonarr-instance-1"}'

# Approve pairing via Admin UI, retrieve token from response

# Use token in subsequent requests
curl -H "X-Device-Token: ${DEVICE_TOKEN}" \
     -H "X-Emby-Token: ${JELLYFIN_API_KEY}" \
     https://jellyfin.example.com/Library/Movies
```

---

## Code Examples

### Custom Middleware Integration (C#)

If you're building a **separate Jellyfin plugin** that needs to hook into JellyfinSecurity's verified-session state:

```csharp
using JellyfinSecurity.Services;
using Microsoft.AspNetCore.Http;

public class CustomAuthMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ISessionVerifier _sessionVerifier;

    public CustomAuthMiddleware(
        RequestDelegate next,
        ISessionVerifier sessionVerifier)
    {
        _next = next;
        _sessionVerifier = sessionVerifier;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var authToken = context.Request.Headers["X-Emby-Token"].FirstOrDefault();
        
        if (string.IsNullOrEmpty(authToken))
        {
            context.Response.StatusCode = 401;
            return;
        }

        // Check if this token passed 2FA verification
        if (!_sessionVerifier.IsVerified(authToken))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("2FA verification required");
            return;
        }

        await _next(context);
    }
}
```

### Audit Log Query (C#)

```csharp
using JellyfinSecurity.Data;
using JellyfinSecurity.Models;

public class AuditLogService
{
    private readonly IAuditLogStore _auditLog;

    public async Task<List<AuditEvent>> GetFailedLoginsAsync(
        DateTime since,
        int limit = 100)
    {
        return await _auditLog.QueryAsync(
            eventType: AuditEventType.LoginFailed,
            startDate: since,
            limit: limit
        );
    }

    public async Task<List<AuditEvent>> GetUserActionsAsync(Guid userId)
    {
        return await _auditLog.QueryByUserAsync(userId);
    }
}
```

### Email OTP Service Configuration (C#)

```csharp
using JellyfinSecurity.Configuration;

var smtpConfig = new EmailOTPConfiguration
{
    Enabled = true,
    SMTPHost = "smtp.gmail.com",
    SMTPPort = 587,
    UseTLS = true,
    Username = "noreply@example.com",
    Password = Environment.GetEnvironmentVariable("SMTP_PASSWORD"),
    FromAddress = "noreply@example.com",
    FromName = "Jellyfin Security",
    CodeExpiration = 300  // 5 minutes
};
```

### HIBP Password Check (C#)

The plugin includes k-anonymity HIBP integration for password breach detection:

```csharp
using JellyfinSecurity.Services;

public class PasswordValidator
{
    private readonly IHIBPService _hibp;

    public async Task<bool> IsPasswordCompromisedAsync(string password)
    {
        // Sends only first 5 chars of SHA-1 hash to HIBP
        // (k-anonymity model — password never leaves server in plaintext)
        return await _hibp.IsPasswordPwnedAsync(password);
    }
}
```

---

## Common Workflows

### Scenario: User Locked Out (Forgot TOTP Device)

**Admin recovery via Dashboard**:

1. **Dashboard** → **JellyfinSecurity** → **Users**
2. Select locked-out user → **Reset 2FA**
3. User can log in with password only (2FA disabled)
4. User re-enrolls TOTP from profile page

**User self-recovery** (if recovery codes saved):

1. Login page → **Use Recovery Code**
2. Enter one of the 8 saved codes (single-use)
3. After login, user can disable TOTP or generate new QR

### Scenario: Impossible Travel Alert

```yaml
# Admin → Settings
ImpossibleTravelEnabled: true
ImpossibleTravelThreshold: 500  # km/hour (flags physically impossible logins)
```

When detected:
1. Login blocked automatically
2. Admin email notification sent (if `AdminEmailAlerts: true`)
3. Audit log entry: `AuditEventType.ImpossibleTravel`
4. Admin must manually unban IP via **Dashboard** → **Banned IPs**

### Scenario: SSO-Only Deployment (Hide Built-In Login)

```yaml
# Admin → Settings
ShowBuiltIn2FAButton: false
ShowBuiltInPasskeyButton: false

OIDCProviders:
  - Name: "Corporate SSO"
    ClientID: "${OIDC_CLIENT_ID}"
    ClientSecret: "${OIDC_CLIENT_SECRET}"
    Authority: "https://sso.corp.example.com"
```

Login page shows only **"Sign in with Corporate SSO"** button.

---

## Troubleshooting

### LAN Bypass Not Working

**Symptom**: Local clients (192.168.x.x) still prompted for 2FA despite `LANBypassEnabled: true`.

**Diagnosis**:
```bash
# Check Jellyfin logs for SEC-H3 guard message
journalctl -u jellyfin | grep "SEC-H3"

# Example output:
# "SEC-H3: Client IP 192.168.1.100 is within trusted proxy range 192.168.0.0/16 
#  but no X-Forwarded-For header present — refusing LAN bypass"
```

**Fix**:
1. If behind reverse proxy, ensure `X-Forwarded-For` header is set:
   ```nginx
   # Nginx
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   ```
   ```yaml
   # Traefik (dynamic config)
   http:
     middlewares:
       jellyfin-headers:
         headers:
           customRequestHeaders:
             X-Forwarded-For: ""  # Traefik auto-populates
   ```

2. If **direct** LAN access (no proxy), tighten `TrustedProxyCIDRs`:
   ```yaml
   # BEFORE (too broad):
   TrustedProxyCIDRs:
     - "192.168.0.0/16"  # Includes LAN clients!
   
   # AFTER (proxy IP only):
   TrustedProxyCIDRs:
     - "192.168.1.5/32"  # Nginx container IP
   ```

### Step-Up Modal Not Appearing

**Symptom**: Admin clicks **Save Settings** but nothing happens (v2.5.7 and earlier).

**Fix**: Upgrade to v2.5.8+. The admin UI now uses step-up-aware fetch wrapper:

```javascript
// Fixed in v2.5.8 — PluginConfigPage.js
async function saveConfiguration() {
    await stepUpAwareFetch('/JellyfinSecurity/Configuration', {
        method: 'POST',
        body: JSON.stringify(config)
    });
}
```

### OIDC Redirect URI Mismatch

**Symptom**: IdP returns `invalid_redirect_uri` error.

**Diagnosis**:
```bash
# Check Jellyfin base URL
curl -s http://localhost:8096/System/Configuration | jq -r '.BaseUrl'
```

**Fix**:
1. Ensure `BaseUrl` matches public-facing URL in Jellyfin **Dashboard** → **Networking**:
   ```yaml
   BaseUrl: "https://jellyfin.example.com"  # Must include https:// if behind SSL proxy
   ```

2. Register exact redirect URI in IdP:
   ```
   https://jellyfin.example.com/JellyfinSecurity/OIDC/Callback
   ```

3. If using Docker + reverse proxy, verify `X-Forwarded-Proto` header:
   ```nginx
   proxy_set_header X-Forwarded-Proto $scheme;
   ```

### Device Token Expired After Server Restart

**Symptom**: All TV clients require re-pairing after `docker restart jellyfin` (v2.5.6 and earlier).

**Fix**: Upgrade to v2.5.7+. Verified tokens are now persisted to `verified-tokens.json`:

```bash
# Check persistence file exists
ls -lh /config/data/jellyfinsecurity/verified-tokens.json

# Should see entries like:
# {"TokenHash":"sha256:abcd1234...", "VerifiedAt":"2026-06-12T10:00:00Z"}
```

### TOTP Code Rejected (Time Sync Issue)

**Symptom**: Valid TOTP code from Authy/Google Authenticator shows "Invalid code."

**Diagnosis**:
```bash
# Check server time
timedatectl

# Check time skew tolerance (default ±1 step = 60 seconds)
journalctl -u jellyfin | grep "TOTP time skew"
```

**Fix**:
1. Enable NTP on Jellyfin server:
   ```bash
   timedatectl set-ntp true
   ```

2. Increase skew window (admin settings):
   ```yaml
   TOTPTimeSkew: 2  # Allow ±2 steps (120 seconds)
   ```

### High CPU Usage from Audit Log

**Symptom**: `jellyfin` process consuming high CPU after enabling audit logging.

**Fix**:
1. Enable log rotation:
   ```yaml
   AuditLogEnabled: true
   AuditLogRotation: true
   AuditLogMaxSize: 104857600  # 100 MB
   AuditLogMaxAge: 30          # Days
   ```

2. Exclude high-frequency events:
   ```yaml
   AuditLogExcludedEvents:
     - "HeartbeatReceived"
     - "SessionActivity"
   ```

---

## Security Considerations

### What This Plugin Defends Against

- ✅ Credential stuffing (brute-force IP banning)
- ✅ Phishing (TOTP/passkeys immune to credential reuse)
- ✅ Unauthorized LAN access (IP allowlist, impossible travel)
- ✅ Compromised passwords (HIBP integration)
- ✅ Session hijacking (token binding to User-Agent + IP)
- ✅ Privilege escalation (step-up auth for admin actions)

### What This Plugin Does NOT Defend Against

- ❌ Server-side vulnerabilities in Jellyfin core (keep Jellyfin updated)
- ❌ Client-side XSS (use Content-Security-Policy headers in reverse proxy)
- ❌ TLS/certificate issues (configure reverse proxy with valid certs)
- ❌ Physical access to server (encrypt `/config` volume)
- ❌ Supply-chain attacks on plugin dependencies (verify release SHA-256)

### Recommended Deployment Hardening

```nginx
# Nginx reverse proxy config
server {
    listen 443 ssl http2;
    server_name jellyfin.example.com;

    ssl_certificate /etc/letsencrypt/live/jellyfin.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jellyfin.example.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;

    # Pass real client IP
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    location / {
        proxy_pass http://jellyfin:8096;
    }
}
```

```yaml
# Docker Compose with read-only filesystem
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
    volumes:
      - /path/to/config:/config  # Must be writable for plugin data
      - /path/to/media:/media:ro
    environment:
      - JELLYFIN_PublishedServerUrl=https://jellyfin.example.com
```

---

## Testing

The plugin includes 254 xUnit tests covering security-critical paths:

```bash
# Run full test suite
git clone https://github.com/ZL154/JellyfinSecurity.git
cd JellyfinSecurity
dotnet test --logger "console;verbosity=detailed"

# Run specific test categories
dotnet test --filter "Category=Crypto"       # TOTP, passkey, cookie HMAC
dotnet test --filter "Category=Middleware"   # Step-up, LAN bypass, brute-force
dotnet test --filter "Category=OIDC"         # SSO flows, token exchange
```

Key test coverage:
- TOTP replay protection (time-step validation)
- Recovery code PBKDF2 hashing (100k iterations)
- Trusted browser token HMAC verification
- CIDR parser edge cases (IPv6, /0, /128)
- X-Forwarded-For trust-walk (multi-proxy chains)
- AES-GCM v2 authenticated encryption
- HIBP k-anonymity hashing (SHA-1 prefix)
- Step-up challenge consumption (single-use tokens)

---

## API Reference

### Public Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/JellyfinSecurity/OIDC/Authorize` | GET | Initiate OIDC flow |
| `/JellyfinSecurity/OIDC/Callback` | GET | OIDC redirect callback |
| `/JellyfinSecurity/PairDevice` | POST | Request TV pairing code |
| `/JellyfinSecurity/CheckPairingStatus` | GET | Poll for pairing approval |
| `/JellyfinSecurity/VerifyTOTP` | POST | Submit TOTP code |
| `/JellyfinSecurity/VerifyPasskey` | POST | Complete WebAuthn ceremony |
| `/JellyfinSecurity/SendEmailOTP` | POST | Request email OTP |

### Admin-Only Endpoints (Require Step-Up if Enabled)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/JellyfinSecurity/Configuration` | GET/POST | Plugin settings |
| `/JellyfinSecurity/Users` | GET | List users with 2FA status |
| `/JellyfinSecurity/Users/{id}` | GET/PUT | User-specific config |
| `/JellyfinSecurity/Users/{id}/ResetTOTP` | POST | Disable user's 2FA |
| `/JellyfinSecurity/PendingPairs` | GET | List awaiting approval |
| `/JellyfinSecurity/ApprovePair` | POST | Approve TV pairing |
| `/JellyfinSecurity/DenyPair` | POST | Reject TV pairing |
| `/JellyfinSecurity/AuditLog` | GET | Query security events |

---

## Further Resources

- **GitHub Repository**: https://github.com/ZL154/JellyfinSecurity
- **Security Policy**: [SECURITY.md](https://github.com/ZL154/JellyfinSecurity/blob/main/SECURITY.md)
- **OpenSSF Scorecard**: https://securityscorecards.dev/viewer/?uri=github.com/ZL154/JellyfinSecurity
- **Jellyfin Plugin Portal**: https://jellyfin.org/plugins/
- **TOTP RFC 6238**: https://datatracker.ietf.org/doc/html/rfc6238
- **WebAuthn Spec**: https://www.w3.org/TR/webauthn-2/

For security issues, email the maintainer directly (see SECURITY.md) or file a private advisory via GitHub Security.
