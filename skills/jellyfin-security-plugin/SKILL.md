---
name: jellyfin-security-plugin
description: Add comprehensive MFA (TOTP, passkeys, email OTP), OIDC/SSO, brute-force protection, impossible-travel detection, device pairing, and audit logging to Jellyfin servers
triggers:
  - "add two-factor authentication to jellyfin"
  - "set up MFA for jellyfin server"
  - "configure TOTP or passkeys in jellyfin"
  - "enable OIDC SSO for jellyfin"
  - "implement brute-force protection for jellyfin"
  - "add device pairing to jellyfin"
  - "configure jellyfin security hardening"
  - "set up impossible travel detection"
---

# Jellyfin Security Plugin

> Skill by [ara.so](https://ara.so) — Security Skills collection.

JellyfinSecurity is a comprehensive authentication and hardening plugin for Jellyfin servers. It adds native two-factor authentication (TOTP, passkeys, email OTP), OIDC/SSO sign-in, brute-force IP banning, impossible-travel detection, per-user IP allowlists, device pairing, trusted-browser cookies, step-up authentication, and full audit logging — all server-side, working with every Jellyfin client (web, mobile, TV, API integrations like Sonarr/Radarr).

## Installation

### Add the Plugin Repository

1. In Jellyfin Dashboard → **Plugins** → **Repositories**, add:
   ```
   https://raw.githubusercontent.com/ZL154/JellyfinSecurity/main/manifest.json
   ```

2. Go to **Catalog**, find **Jellyfin Security**, click **Install**

3. Restart Jellyfin server

4. Navigate to **Dashboard** → **Plugins** → **Jellyfin Security** to configure

### Manual Installation (Development)

```bash
# Clone and build
git clone https://github.com/ZL154/JellyfinSecurity.git
cd JellyfinSecurity
dotnet build -c Release

# Copy to Jellyfin plugins directory
# Linux/Docker:
cp bin/Release/net8.0/JellyfinSecurity.dll /var/lib/jellyfin/plugins/JellyfinSecurity/
# Windows:
# copy bin\Release\net8.0\JellyfinSecurity.dll %AppData%\Jellyfin\Server\plugins\JellyfinSecurity\

# Restart Jellyfin
```

## Core Concepts

### Authentication Methods

The plugin supports five authentication methods:

1. **TOTP (Time-based One-Time Password)** — standard authenticator apps (Authy, Google Authenticator, etc.)
2. **Passkeys (WebAuthn/FIDO2)** — hardware keys (YubiKey) or platform authenticators (Face ID, Windows Hello)
3. **Email OTP** — 8-digit codes sent via SMTP
4. **Recovery Codes** — 10 single-use backup codes (PBKDF2-hashed)
5. **OIDC/SSO** — external identity providers (Authentik, Authelia, Keycloak, etc.)

### Enforcement Model

- **Server-side middleware** intercepts all authentication requests before Jellyfin's core auth
- Works with **all clients** (web UI, mobile apps, Jellyfin Media Player, Kodi, API clients)
- No client modifications required

### Bypass Rules (in precedence order)

1. **LAN Bypass** — clients from configured LAN CIDRs skip MFA (optional, requires proper `X-Forwarded-For` handling)
2. **Trusted Device Tokens** — encrypted browser cookies valid for 30 days (configurable)
3. **TV Device Pairing** — admin-approved device GUIDs
4. **API Key Bypass** — Sonarr/Radarr/etc. using Jellyfin API keys
5. **Exempt User CIDRs** — per-user IP allowlists

## Configuration

### Basic Setup

```csharp
// Example: Programmatic configuration (rarely needed; use admin UI)
using JellyfinSecurity.Configuration;

var config = new PluginConfiguration
{
    EnableTwoFactor = true,
    RequireTwoFactor = true,  // Force MFA for all users
    TrustedDeviceDurationDays = 30,
    
    // LAN bypass (optional)
    LanBypassEnabled = true,
    LanCidrs = new[] { "192.168.1.0/24", "10.0.0.0/8" },
    
    // Brute-force protection
    MaxLoginAttempts = 5,
    LockoutDurationMinutes = 15,
    
    // Email OTP (requires SMTP config)
    SmtpHost = "smtp.gmail.com",
    SmtpPort = 587,
    SmtpUsername = "alerts@example.com",
    SmtpPassword = Environment.GetEnvironmentVariable("SMTP_PASSWORD"),
    SmtpFromAddress = "jellyfin@example.com"
};
```

### OIDC/SSO Setup

The plugin supports multiple OIDC providers simultaneously:

```csharp
// Configuration via admin UI → SSO Providers tab
// Example provider JSON (Authentik):
{
  "Name": "Authentik",
  "ClientId": "jellyfin-client",
  "ClientSecret": "${OIDC_CLIENT_SECRET}",  // Use env var
  "Authority": "https://auth.example.com/application/o/jellyfin/",
  "Scopes": ["openid", "profile", "email"],
  "UsePkce": true,
  "ClaimMappings": {
    "preferred_username": "username",
    "email": "email",
    "name": "displayName"
  }
}
```

**Key OIDC Features:**
- Multiple providers (e.g., Authentik for admins, Google for users)
- LAN-only IdP support (disable SSRF guard for private networks)
- Step-up re-authentication for sensitive actions
- Auto-linking or manual admin approval

### Step-Up Authentication

Require re-authentication for sensitive admin actions:

```csharp
// StepUpLevel options (admin UI dropdown):
// - Off: No step-up required
// - AllConfigChanges: Plugin settings + user management
// - DeleteActions: Above + user deletion
// - PluginConfigOnly: Only plugin settings

// Example: Enforce TOTP for all config changes
config.StepUpLevel = StepUpLevel.AllConfigChanges;
config.SelfServiceStepUpMode = SelfServiceStepUpMode.Forced; // Users must verify even for profile changes
```

### Impossible Travel Detection

Automatically flag/block logins from suspicious geographic locations:

```csharp
config.ImpossibleTravelEnabled = true;
config.ImpossibleTravelMode = ImpossibleTravelMode.Block;  // or LogOnly
config.ImpossibleTravelThresholdKmPerHour = 800.0;  // ~500 mph
config.MaxMindLicenseKey = Environment.GetEnvironmentVariable("MAXMIND_LICENSE_KEY");
```

The plugin downloads MaxMind GeoLite2 databases automatically and checks if login locations are physically impossible based on time deltas.

### Per-User IP Allowlist

```csharp
// Example: Allow user 'alice' from specific IPs only
// In admin UI → Users tab → Edit User → Exempt CIDRs:
// 203.0.113.0/24
// 2001:db8::/32

// Programmatic (via plugin's UserDataManager):
var userData = await userDataManager.GetUserDataAsync(userId);
userData.ExemptCidrs = new[] { "203.0.113.0/24" };
await userDataManager.SaveUserDataAsync(userData);
```

## Code Examples

### Enrolling TOTP (C# Plugin Context)

```csharp
using JellyfinSecurity.Totp;
using JellyfinSecurity.Data;

public async Task<TotpEnrollmentResult> EnrollUserTotp(Guid userId, string username)
{
    var totpManager = new TotpManager();
    var userDataManager = GetUserDataManager(); // Plugin service
    
    // Generate secret
    var secret = totpManager.GenerateSecret();
    var qrCodeUri = totpManager.GetQrCodeUri(username, secret, "Jellyfin");
    
    // Store pending enrollment (not active until verified)
    var userData = await userDataManager.GetUserDataAsync(userId);
    userData.TotpSecretPending = secret;
    await userDataManager.SaveUserDataAsync(userData);
    
    return new TotpEnrollmentResult
    {
        Secret = secret,
        QrCodeUri = qrCodeUri,
        ManualEntryKey = FormatSecretForDisplay(secret)
    };
}

public async Task<bool> VerifyAndActivateTotp(Guid userId, string code)
{
    var totpManager = new TotpManager();
    var userDataManager = GetUserDataManager();
    
    var userData = await userDataManager.GetUserDataAsync(userId);
    if (string.IsNullOrEmpty(userData.TotpSecretPending))
        return false;
    
    // Verify code against pending secret
    if (!totpManager.ValidateCode(userData.TotpSecretPending, code))
        return false;
    
    // Activate TOTP
    userData.TotpSecret = userData.TotpSecretPending;
    userData.TotpSecretPending = null;
    userData.TotpEnabled = true;
    await userDataManager.SaveUserDataAsync(userData);
    
    return true;
}
```

### Passkey Registration (WebAuthn)

```csharp
using Fido2NetLib;
using Fido2NetLib.Objects;

public async Task<CredentialCreateOptions> StartPasskeyRegistration(Guid userId, string username)
{
    var fido2 = GetFido2Instance(); // Plugin's Fido2 singleton
    var userDataManager = GetUserDataManager();
    
    var userData = await userDataManager.GetUserDataAsync(userId);
    var existingKeys = userData.WebAuthnCredentials ?? new List<StoredCredential>();
    
    var fidoUser = new Fido2User
    {
        Id = userId.ToByteArray(),
        Name = username,
        DisplayName = username
    };
    
    var options = fido2.RequestNewCredential(
        fidoUser,
        existingKeys.Select(c => new PublicKeyCredentialDescriptor(c.DescriptorId)).ToList(),
        AuthenticatorSelection.Default,
        AttestationConveyancePreference.None
    );
    
    // Store challenge for verification
    await StoreChallengeAsync(userId, options.Challenge);
    
    return options;
}

public async Task<bool> CompletePasskeyRegistration(Guid userId, AuthenticatorAttestationRawResponse attestation)
{
    var fido2 = GetFido2Instance();
    var userDataManager = GetUserDataManager();
    
    var challenge = await RetrieveChallengeAsync(userId);
    var options = /* reconstruct CredentialCreateOptions from challenge */;
    
    var result = await fido2.MakeNewCredentialAsync(
        attestation,
        options,
        async (args, cancellationToken) => true // Credential ID uniqueness check
    );
    
    if (result.Status != "ok")
        return false;
    
    // Store credential
    var userData = await userDataManager.GetUserDataAsync(userId);
    userData.WebAuthnCredentials ??= new List<StoredCredential>();
    userData.WebAuthnCredentials.Add(new StoredCredential
    {
        DescriptorId = result.Result.CredentialId,
        PublicKey = result.Result.PublicKey,
        SignCount = result.Result.Counter,
        CredType = result.Result.CredType,
        AaGuid = result.Result.Aaguid,
        UserHandle = result.Result.User.Id,
        CreatedAt = DateTime.UtcNow
    });
    userData.PasskeysEnabled = true;
    await userDataManager.SaveUserDataAsync(userData);
    
    return true;
}
```

### Email OTP

```csharp
using JellyfinSecurity.Email;

public async Task<bool> SendEmailOtp(Guid userId, string email)
{
    var otpManager = GetOtpManager();
    var emailSender = GetEmailSender(); // Plugin's SMTP service
    
    // Generate 8-digit code
    var code = otpManager.GenerateEmailOtp();
    var expiresAt = DateTime.UtcNow.AddMinutes(10);
    
    // Store code (hashed)
    await otpManager.StoreEmailOtpAsync(userId, code, expiresAt);
    
    // Send email
    var subject = "Your Jellyfin Login Code";
    var body = $"Your verification code is: {code}\n\nValid for 10 minutes.";
    
    return await emailSender.SendEmailAsync(email, subject, body);
}

public async Task<bool> VerifyEmailOtp(Guid userId, string code)
{
    var otpManager = GetOtpManager();
    return await otpManager.ValidateEmailOtpAsync(userId, code);
}
```

### Checking Authentication Requirements

```csharp
using JellyfinSecurity.Middleware;

public async Task<AuthRequirement> GetAuthRequirementsForUser(Guid userId, string remoteIp)
{
    var config = GetPluginConfiguration();
    var userDataManager = GetUserDataManager();
    var deviceManager = GetDeviceManager();
    
    // Check bypass rules
    if (config.LanBypassEnabled && IsLanIp(remoteIp, config.LanCidrs))
        return AuthRequirement.None;
    
    var userData = await userDataManager.GetUserDataAsync(userId);
    
    // Check per-user IP allowlist
    if (userData.ExemptCidrs?.Any() == true && MatchesCidr(remoteIp, userData.ExemptCidrs))
        return AuthRequirement.None;
    
    // Check trusted device token (from cookie)
    var deviceToken = GetDeviceTokenFromRequest();
    if (!string.IsNullOrEmpty(deviceToken) && await deviceManager.IsDeviceTrustedAsync(userId, deviceToken))
        return AuthRequirement.None;
    
    // Check TV pairing
    var deviceId = GetDeviceIdFromRequest();
    if (!string.IsNullOrEmpty(deviceId) && await deviceManager.IsDevicePairedAsync(userId, deviceId))
        return AuthRequirement.None;
    
    // Determine required method
    if (userData.TotpEnabled)
        return AuthRequirement.Totp;
    if (userData.PasskeysEnabled)
        return AuthRequirement.Passkey;
    if (userData.EmailOtpEnabled)
        return AuthRequirement.EmailOtp;
    
    return config.RequireTwoFactor ? AuthRequirement.MustEnroll : AuthRequirement.None;
}
```

### Brute-Force Protection

```csharp
using JellyfinSecurity.BruteForce;

public async Task<bool> CheckAndRecordLoginAttempt(string username, string remoteIp, bool success)
{
    var bruteForceManager = GetBruteForceManager();
    var config = GetPluginConfiguration();
    
    if (success)
    {
        await bruteForceManager.ClearAttemptsAsync(username, remoteIp);
        return true;
    }
    
    // Record failure
    var attempts = await bruteForceManager.RecordFailedAttemptAsync(username, remoteIp);
    
    if (attempts >= config.MaxLoginAttempts)
    {
        // Lock out
        await bruteForceManager.LockoutAsync(
            username,
            remoteIp,
            TimeSpan.FromMinutes(config.LockoutDurationMinutes)
        );
        
        // Log audit event
        await LogAuditEventAsync(new AuditEvent
        {
            EventType = "BruteForceLockout",
            Username = username,
            IpAddress = remoteIp,
            Timestamp = DateTime.UtcNow,
            Details = $"Locked out after {attempts} failed attempts"
        });
        
        return false;
    }
    
    return true; // Not locked out yet
}

public async Task<bool> IsIpLockedOut(string remoteIp)
{
    var bruteForceManager = GetBruteForceManager();
    return await bruteForceManager.IsLockedOutAsync(remoteIp);
}
```

### Device Pairing (TV Clients)

```csharp
using JellyfinSecurity.DevicePairing;

public async Task<PairingRequest> RequestDevicePairing(Guid userId, string deviceId, string deviceName)
{
    var pairingManager = GetDevicePairingManager();
    
    var request = new PairingRequest
    {
        Id = Guid.NewGuid(),
        UserId = userId,
        DeviceId = deviceId,
        DeviceName = deviceName,
        RequestedAt = DateTime.UtcNow,
        Status = PairingStatus.Pending
    };
    
    await pairingManager.SavePairingRequestAsync(request);
    
    // Notify admins (email/webhook/etc.)
    await NotifyAdminsOfPairingRequestAsync(request);
    
    return request;
}

public async Task<bool> ApprovePairingRequest(Guid requestId, Guid adminUserId)
{
    var pairingManager = GetDevicePairingManager();
    
    var request = await pairingManager.GetPairingRequestAsync(requestId);
    if (request == null || request.Status != PairingStatus.Pending)
        return false;
    
    request.Status = PairingStatus.Approved;
    request.ApprovedBy = adminUserId;
    request.ApprovedAt = DateTime.UtcNow;
    
    await pairingManager.SavePairingRequestAsync(request);
    
    // Add to paired devices
    await pairingManager.AddPairedDeviceAsync(request.UserId, request.DeviceId);
    
    // Log audit
    await LogAuditEventAsync(new AuditEvent
    {
        EventType = "DevicePaired",
        UserId = request.UserId,
        AdminUserId = adminUserId,
        Details = $"Device {request.DeviceName} ({request.DeviceId}) approved"
    });
    
    return true;
}
```

## Common Patterns

### Middleware Integration

The plugin uses ASP.NET Core middleware to intercept authentication:

```csharp
// Simplified middleware flow (internal to plugin)
public class TwoFactorAuthMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        // Skip non-auth endpoints
        if (!IsAuthenticationEndpoint(context.Request.Path))
        {
            await next(context);
            return;
        }
        
        // Check bypass rules
        var remoteIp = GetRealIpAddress(context);
        if (ShouldBypass(context, remoteIp))
        {
            await next(context);
            return;
        }
        
        // Require 2FA
        var userId = GetUserIdFromRequest(context);
        if (!await Is2FASatisfiedAsync(userId, context))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "TwoFactorRequired",
                Message = "Two-factor authentication is required"
            });
            return;
        }
        
        await next(context);
    }
}
```

### Trusted Device Cookies

```csharp
// Setting trusted device cookie (internal to plugin)
public void SetTrustedDeviceCookie(HttpContext context, Guid userId)
{
    var token = GenerateSecureToken(); // 32-byte random + HMAC
    var encrypted = EncryptToken(token, userId); // AES-GCM with AAD
    
    context.Response.Cookies.Append("JellyfinTrustedDevice", encrypted, new CookieOptions
    {
        HttpOnly = true,
        Secure = true,
        SameSite = SameSiteMode.Lax,
        Expires = DateTimeOffset.UtcNow.AddDays(30),
        Path = "/"
    });
    
    // Store token hash server-side
    await StoreDeviceTokenAsync(userId, HashToken(token));
}
```

### Audit Logging

```csharp
using JellyfinSecurity.Audit;

public async Task LogSecurityEvent(AuditEventType eventType, Guid? userId, string details)
{
    var auditLogger = GetAuditLogger();
    
    await auditLogger.LogAsync(new AuditEvent
    {
        EventType = eventType.ToString(),
        UserId = userId,
        Timestamp = DateTime.UtcNow,
        IpAddress = GetCurrentRequestIp(),
        UserAgent = GetCurrentUserAgent(),
        Details = details,
        Success = true
    });
}

// Example: Log TOTP verification
await LogSecurityEvent(
    AuditEventType.TotpVerified,
    userId,
    "User verified TOTP code successfully"
);

// Query audit log
var events = await auditLogger.GetEventsAsync(
    userId: userId,
    startDate: DateTime.UtcNow.AddDays(-7),
    eventTypes: new[] { "TotpVerified", "PasskeyUsed", "BruteForceLockout" }
);
```

## Troubleshooting

### LAN Bypass Not Working

**Problem:** Users on local network still prompted for 2FA

**Solutions:**
1. Check `X-Forwarded-For` header is being passed by reverse proxy:
   ```nginx
   # Nginx
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   
   # Caddy
   reverse_proxy jellyfin:8096 {
       header_up X-Forwarded-For {remote_host}
   }
   ```

2. Add proxy IP to **Trusted Proxy CIDRs** (Admin UI → Network tab)
   - ⚠️ **Do not use broad ranges** (e.g., `10.0.0.0/8`) — use specific proxy IPs like `172.17.0.1/32`
   - The SEC-H3 guard refuses LAN bypass if XFF header is missing/untrusted

3. Verify LAN CIDRs match your network:
   ```csharp
   // Check logs for CIDR match failures
   // Admin UI → Plugins → Jellyfin Security → LAN CIDRs
   // Example: 192.168.1.0/24, 10.0.50.0/24
   ```

### OIDC Redirect URI Mismatch

**Problem:** `redirect_uri_mismatch` error after OIDC login

**Solutions:**
1. Check redirect URI in provider config matches:
   ```
   https://jellyfin.example.com/sso/callback/{providerId}
   ```

2. For reverse proxies, ensure `X-Forwarded-Proto` and `X-Forwarded-Host` are set:
   ```nginx
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_set_header X-Forwarded-Host $host;
   ```

3. For LAN-only IdPs, enable **Private/LAN IdP** toggle in provider settings

### TOTP Codes Not Accepting

**Problem:** Valid TOTP codes rejected

**Solutions:**
1. Check server time sync:
   ```bash
   # Linux
   timedatectl status
   ntpq -p
   
   # Sync if needed
   sudo ntpdate -s time.nist.gov
   ```

2. Verify authenticator app time is correct (most apps sync automatically)

3. Check time-skew tolerance in logs (plugin allows ±1 time step = 30 seconds)

### Trusted Device Cookies Not Persisting

**Problem:** Users re-prompted for 2FA on every login

**Solutions:**
1. Verify `Secure` cookie flag matches your deployment:
   - HTTPS required if `Secure=true` (default)
   - For HTTP testing, temporarily disable in plugin config

2. Check browser cookie settings (third-party cookie blocking can interfere)

3. Verify cookie domain/path in browser DevTools → Application → Cookies

### Email OTP Not Sending

**Problem:** No emails received for OTP codes

**Solutions:**
1. Test SMTP settings:
   ```csharp
   // Admin UI → Email tab → Send Test Email
   ```

2. Check common SMTP ports:
   - **587** (STARTTLS) — most common
   - **465** (implicit TLS) — requires `SmtpUseSsl=true`
   - **25** (plain) — often blocked by ISPs

3. For Gmail/Google Workspace:
   - Use App Password (not account password)
   - Enable "Less secure app access" or use OAuth2

4. Check spam folder and firewall rules

### Impossible Travel False Positives

**Problem:** Legitimate logins blocked as impossible travel

**Solutions:**
1. Adjust threshold (Admin UI → Impossible Travel tab):
   ```csharp
   config.ImpossibleTravelThresholdKmPerHour = 1000.0; // More permissive
   ```

2. Use `LogOnly` mode initially to tune threshold:
   ```csharp
   config.ImpossibleTravelMode = ImpossibleTravelMode.LogOnly;
   ```

3. Add VPN/proxy IPs to user's Exempt CIDRs

4. Check MaxMind database updates (plugin auto-updates weekly)

### Step-Up Modal Not Appearing

**Problem:** Admin UI changes save without prompting for re-auth

**Solutions:**
1. Verify `StepUpLevel` is set (Admin UI → Security tab):
   ```csharp
   config.StepUpLevel = StepUpLevel.AllConfigChanges;
   ```

2. Ensure user has enrolled at least one 2FA method (TOTP/passkey/recovery)

3. Check browser console for JavaScript errors blocking modal

4. For OIDC-only users, verify `SsoLink` is stored (step-up uses IdP re-auth)

### User Lockout Recovery

**Problem:** Admin locked out after misconfiguration

**Solutions:**
1. **Disable plugin temporarily:**
   ```bash
   # Move plugin DLL out of plugins folder
   mv /var/lib/jellyfin/plugins/JellyfinSecurity /tmp/
   systemctl restart jellyfin
   ```

2. **Use recovery codes** (if enrolled):
   - Each user gets 10 single-use codes during TOTP/passkey enrollment
   - Codes are PBKDF2-hashed server-side

3. **Manual database edit** (last resort):
   ```bash
   # Edit user data file
   nano /var/lib/jellyfin/plugins/JellyfinSecurity_[version]/userdata/{userId}.json
   
   # Set TotpEnabled, PasskeysEnabled, EmailOtpEnabled to false
   # Or delete entire file to reset user's plugin data
   ```

4. **LAN bypass** (if enabled):
   - Access Jellyfin from LAN IP to skip 2FA

### Device Pairing Not Available

**Problem:** TV client pairing option missing

**Solutions:**
1. Verify TV pairing enabled (Admin UI → Device Pairing tab)

2. Ensure client sends `X-Emby-Authorization` header with `DeviceId`

3. Check pending pairing requests in admin UI → Pairing Requests tab

4. Some older clients don't expose device ID — use API key bypass instead

## API Reference

### REST Endpoints (Plugin-Added)

```http
### Enroll TOTP
POST /api/security/totp/enroll
Authorization: MediaBrowser Token="{JELLYFIN_TOKEN}"
Content-Type: application/json

{
  "userId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}

### Verify TOTP
POST /api/security/totp/verify
Content-Type: application/json

{
  "userId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "code": "123456"
}

### Start Passkey Registration
POST /api/security/passkey/register/start
Authorization: MediaBrowser Token="{JELLYFIN_TOKEN}"

### Complete Passkey Registration
POST /api/security/passkey/register/complete
Authorization: MediaBrowser Token="{JELLYFIN_TOKEN}"
Content-Type: application/json

{
  "attestationResponse": { /* WebAuthn credential */ }
}

### Request Device Pairing
POST /api/security/pairing/request
Content-Type: application/json

{
  "userId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "deviceId": "tv-living-room-abc123",
  "deviceName": "Samsung TV (Living Room)"
}

### Get Audit Log
GET /api/security/audit?userId={userId}&startDate={iso8601}&eventType={type}
Authorization: MediaBrowser Token="{JELLYFIN_TOKEN}"
```

## Security Considerations

- **TOTP secrets** are stored encrypted at rest using AES-256-GCM
- **Passkey credentials** use FIDO2 attestation; private keys never leave device
- **Recovery codes** are PBKDF2-hashed (100k iterations) before storage
- **Trusted device tokens** are HMAC-signed and encrypted with per-user keys
- **Email OTPs** are SHA-256 hashed server-side
- **OIDC state/nonce** parameters prevent CSRF and replay attacks
- **LAN bypass requires X-Forwarded-For validation** to prevent spoofing
- **Step-up challenges expire after 5 minutes** and are single-use

### Threat Model

**Defends against:**
- Credential stuffing / brute-force (rate limiting + lockout)
- Session hijacking (impossible travel detection)
- Unauthorized API access (requires MFA even with valid password)
- Phishing (passkeys are origin-bound)
- Stolen passwords (2FA required)

**Does NOT defend against:**
- Compromised Jellyfin server (plugin runs in-process)
- Malicious Jellyfin clients (no client-side enforcement)
- Supply chain attacks on dependencies (use CodeQL + Dependabot)
- Physical device theft with unlocked authenticator app

## Environment Variables

```bash
# SMTP password (for email OTP)
SMTP_PASSWORD=your-smtp-password

# OIDC client secrets (if using SSO)
OIDC_CLIENT_SECRET=your-oidc-client-secret

# MaxMind license key (for impossible travel detection)
MAXMIND_LICENSE_KEY=your-maxmind-key

# Example Docker Compose
version: '3.8'
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    environment:
      - SMTP_PASSWORD=${SMTP_PASSWORD}
      - OIDC_CLIENT_SECRET=${OIDC_CLIENT_SECRET}
      - MAXMIND_LICENSE_KEY=${MAXMIND_LICENSE_KEY}
    volumes:
      - /var/lib/jellyfin:/config
```

## Development

### Running Tests

```bash
dotnet test --configuration Release --logger "console;verb
