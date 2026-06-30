---
name: jamboree-android-security-testing
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and rooted emulators for mobile penetration testing
triggers:
  - set up android security testing environment
  - configure burp suite with android emulator
  - install magisk modules for security testing
  - intercept android app traffic with burp
  - use objection to bypass ssl pinning
  - root android emulator for penetration testing
  - set up jamboree android testing framework
  - configure frida and objection for android
---

# JAMBOREE Android Security Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates Magisk module management, Burp Suite proxy configuration, Objection runtime exploration, and rooted emulator environments into a single orchestrated workflow for mobile penetration testing.

## What JAMBOREE Does

JAMBOREE provides:
- **Automated Magisk module deployment** for systemless root and BusyBox
- **Burp Suite proxy integration** with automatic CA certificate installation
- **Objection/Frida runtime hooks** for SSL pinning bypass and method tracing
- **Pre-configured Android emulator environments** optimized for security testing
- **Self-healing configuration** that detects and repairs broken proxy chains

## Installation

### Prerequisites

Ensure the following tools are installed:

```bash
# Verify Java JDK 11+
java -version

# Verify Android SDK platform tools
adb --version

# Install Python and pip for Objection/Frida
python3 --version
pip3 --version

# Install Frida and Objection
pip3 install frida-tools objection
```

### Clone and Initialize

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run environment validation
./orchestration/validators/check-environment.sh

# Deploy core components
./orchestration/deployers/install-all.sh
```

### Environment Variables

Configure these in your shell profile or `.env` file:

```bash
# Android SDK path
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$ANDROID_HOME/platform-tools:$PATH"

# Burp Suite configuration
export BURP_PROXY_HOST="127.0.0.1"
export BURP_PROXY_PORT="8080"
export BURP_CERT_PATH="$HOME/.jamboree/burp-cert.der"

# Frida/Objection settings
export FRIDA_SERVER_VERSION="16.1.4"
export OBJECTION_STARTUP_SCRIPT="$HOME/.jamboree/objection-init.js"
```

## Key Components

### 1. Emulator Setup

Create and configure a rooted Android emulator:

```bash
# Create AVD with Magisk support
avdmanager create avd -n jamboree-test \
  -k "system-images;android-30;google_apis_playstore;x86_64" \
  -d "pixel_5"

# Start emulator with writable system partition
emulator -avd jamboree-test -writable-system -no-snapshot-load &

# Wait for boot
adb wait-for-device

# Install Magisk (automated via orchestration script)
./modules/magisk/systemless/install-magisk.sh
```

### 2. Burp Suite Certificate Installation

Install Burp CA certificate as system certificate:

```bash
# Export Burp certificate (DER format from Burp Suite)
# Place at $BURP_CERT_PATH

# Convert and install as system certificate
./modules/burp/certificates/install-burp-cert.sh

# Verify installation
adb shell "ls /system/etc/security/cacerts/ | grep burp"
```

Manual certificate installation script:

```bash
#!/bin/bash
# install-burp-cert.sh

CERT_PATH="${BURP_CERT_PATH}"
CERT_HASH=$(openssl x509 -inform DER -in "$CERT_PATH" -subject_hash_old -noout)

adb root
adb remount
adb push "$CERT_PATH" "/system/etc/security/cacerts/${CERT_HASH}.0"
adb shell "chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0"
adb reboot
```

### 3. Proxy Configuration

Configure device to route traffic through Burp Suite:

```bash
# Set global proxy
adb shell settings put global http_proxy "${BURP_PROXY_HOST}:${BURP_PROXY_PORT}"

# Verify proxy settings
adb shell settings get global http_proxy

# Clear proxy when done
adb shell settings put global http_proxy :0
```

### 4. Objection Runtime Hooks

Launch Objection against target application:

```bash
# List running applications
frida-ps -Ua

# Attach to running app
objection -g com.example.targetapp explore

# Or spawn app with Objection
objection -g com.example.targetapp explore --startup-script "$OBJECTION_STARTUP_SCRIPT"
```

Common Objection commands:

```javascript
// Inside Objection REPL

// Bypass SSL pinning
android sslpinning disable

// List activities
android hooking list activities

// Watch class methods
android hooking watch class_method com.example.MainActivity.getData --dump-args --dump-return

// Dump SharedPreferences
android shared_preferences dump

// List SQLite databases
android sqlite execute "SELECT name FROM sqlite_master WHERE type='table';"

// File system operations
file ls /data/data/com.example.targetapp/

// Memory search
memory search "password" --string
```

### 5. Custom Frida Scripts

Create persistent hooks with Frida:

```javascript
// objection-init.js - Startup script for automated hooking

Java.perform(function() {
    console.log("[*] JAMBOREE: Loading custom hooks");
    
    // Bypass root detection
    var RootBeer = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer.isRooted.implementation = function() {
        console.log("[*] Root detection bypassed");
        return false;
    };
    
    // Hook cryptographic operations
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function(input) {
        console.log("[*] Cipher.doFinal called");
        console.log("[*] Input: " + bytesToHex(input));
        var result = this.doFinal(input);
        console.log("[*] Output: " + bytesToHex(result));
        return result;
    };
    
    // Intercept API responses
    var OkHttpClient = Java.use("okhttp3.OkHttpClient");
    var Interceptor = Java.use("okhttp3.Interceptor");
    OkHttpClient.build.implementation = function() {
        var client = this.build();
        console.log("[*] OkHttp client intercepted");
        // Add custom logging interceptor
        return client;
    };
    
    function bytesToHex(bytes) {
        var hex = "";
        for (var i = 0; i < bytes.length; i++) {
            hex += ("0" + (bytes[i] & 0xFF).toString(16)).slice(-2);
        }
        return hex;
    }
});
```

Run Frida script directly:

```bash
# Attach to process with custom script
frida -U -f com.example.targetapp -l hooks.js --no-pause
```

## Configuration Files

### Magisk Module Configuration

```bash
# modules/magisk/systemless/module.prop
id=jamboree-systemless
name=JAMBOREE Systemless Root
version=1.0
versionCode=1
author=JAMBOREE
description=Systemless root with BusyBox for security testing
```

### Burp Suite Extension Configuration

```json
{
  "proxy": {
    "host": "${BURP_PROXY_HOST}",
    "port": "${BURP_PROXY_PORT}",
    "ssl_pass_through": [
      "*.google.com",
      "*.gstatic.com"
    ]
  },
  "certificate": {
    "auto_install": true,
    "path": "${BURP_CERT_PATH}"
  },
  "logging": {
    "level": "info",
    "file": "/var/log/jamboree-burp.log"
  }
}
```

### Orchestration Configuration

```yaml
# configurations/security/jamboree-config.yaml
environment:
  android_version: "30"
  emulator_name: "jamboree-test"
  root_method: "magisk"

proxy:
  enabled: true
  host: "${BURP_PROXY_HOST}"
  port: "${BURP_PROXY_PORT}"
  certificate_pinning_bypass: true

objection:
  auto_start: true
  startup_script: "${OBJECTION_STARTUP_SCRIPT}"
  hooks:
    - ssl_pinning_bypass
    - root_detection_bypass
    - certificate_validation_bypass

logging:
  verbose: true
  output_dir: "./logs"
```

## Common Testing Workflows

### Workflow 1: Intercept HTTPS Traffic

```bash
# 1. Start emulator and Burp Suite
./orchestration/deployers/start-environment.sh

# 2. Install target APK
adb install target-app.apk

# 3. Configure proxy
adb shell settings put global http_proxy "127.0.0.1:8080"

# 4. Launch Objection with SSL pinning bypass
objection -g com.example.targetapp explore

# Inside Objection:
# > android sslpinning disable

# 5. Use the app and observe traffic in Burp Suite
```

### Workflow 2: Bypass Certificate Pinning

```bash
# Create custom Frida script
cat > bypass-pinning.js << 'EOF'
Java.perform(function() {
    // TrustManager bypass
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    
    var TrustManagerImpl = Java.registerClass({
        name: "com.jamboree.TrustManagerImpl",
        implements: [TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    var TrustManagers = [TrustManagerImpl.$new()];
    var SSLContextInstance = SSLContext.getInstance("TLS");
    SSLContextInstance.init(null, TrustManagers, null);
    
    console.log("[*] Certificate pinning bypassed");
});
EOF

# Run with Frida
frida -U -f com.example.targetapp -l bypass-pinning.js --no-pause
```

### Workflow 3: Extract Cryptographic Keys

```javascript
// key-extraction.js
Java.perform(function() {
    var SecretKeySpec = Java.use("javax.crypto.spec.SecretKeySpec");
    
    SecretKeySpec.$init.overload("[B", "java.lang.String").implementation = function(keyBytes, algorithm) {
        console.log("[*] SecretKeySpec created");
        console.log("[*] Algorithm: " + algorithm);
        console.log("[*] Key (hex): " + bytesToHex(keyBytes));
        
        // Save to file for later analysis
        var File = Java.use("java.io.File");
        var FileOutputStream = Java.use("java.io.FileOutputStream");
        var path = "/sdcard/Download/extracted_key.bin";
        var file = File.$new(path);
        var fos = FileOutputStream.$new(file);
        fos.write(keyBytes);
        fos.close();
        console.log("[*] Key saved to: " + path);
        
        return this.$init(keyBytes, algorithm);
    };
    
    function bytesToHex(bytes) {
        var hex = "";
        for (var i = 0; i < bytes.length; i++) {
            hex += ("0" + (bytes[i] & 0xFF).toString(16)).slice(-2);
        }
        return hex;
    }
});
```

### Workflow 4: Database Analysis

```bash
# List all SQLite databases
objection -g com.example.targetapp explore

# Inside Objection:
# > android sqlite list

# Connect to specific database
# > android sqlite connect /data/data/com.example.targetapp/databases/app.db

# Execute queries
# > android sqlite execute "SELECT * FROM users"

# Export database for offline analysis
adb pull /data/data/com.example.targetapp/databases/app.db ./
sqlitebrowser app.db
```

## Troubleshooting

### Certificate Not Trusted

If HTTPS interception fails:

```bash
# Verify certificate installation
adb shell "ls -la /system/etc/security/cacerts/ | grep $(openssl x509 -inform DER -in $BURP_CERT_PATH -subject_hash_old -noout)"

# Check system time sync
adb shell "date"

# Reinstall certificate with correct permissions
adb root
adb remount
./modules/burp/certificates/install-burp-cert.sh
adb reboot
```

### Frida Server Connection Issues

```bash
# Check if frida-server is running
adb shell "ps -A | grep frida"

# Restart frida-server
adb shell "su -c 'killall frida-server'"
adb push frida-server-${FRIDA_SERVER_VERSION}-android-x86_64 /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "su -c '/data/local/tmp/frida-server &'"

# Verify connection
frida-ps -U
```

### Magisk Module Not Loading

```bash
# Enter Safe Mode
adb reboot

# Remove conflicting modules
adb shell "su -c 'rm -rf /data/adb/modules/conflicting-module'"

# Clear Magisk cache
adb shell "su -c 'rm -rf /data/adb/magisk.db'"

# Reinstall module
./modules/magisk/systemless/install-magisk.sh
```

### Proxy Not Working

```bash
# Verify proxy settings
adb shell settings get global http_proxy

# Test connectivity through proxy
adb shell "curl -x 127.0.0.1:8080 http://example.com"

# Check iptables rules
adb shell "su -c 'iptables -t nat -L'"

# Use VPN-based interception if global proxy fails
# Install and configure ProxyDroid or similar app
```

### App Detects Emulator

Modify AVD to mimic physical device:

```bash
# Edit emulator config
echo "hw.device.manufacturer=Samsung" >> ~/.android/avd/jamboree-test.avd/config.ini
echo "hw.device.name=Galaxy S21" >> ~/.android/avd/jamboree-test.avd/config.ini

# Use Magisk Hide
adb shell "su -c 'magiskhide enable'"
adb shell "su -c 'magiskhide add com.example.targetapp'"
```

## Integration with CI/CD

Automate testing in GitLab CI:

```yaml
# .gitlab-ci.yml
android_security_test:
  image: androidsdk/android-30
  services:
    - name: budtmo/docker-android-x86-11.0
      alias: android
  script:
    - ./orchestration/validators/check-environment.sh
    - ./orchestration/deployers/install-all.sh
    - adb connect android:5555
    - adb install target-app.apk
    - objection -g com.example.targetapp explore --startup-script test-hooks.js
  artifacts:
    paths:
      - logs/
      - reports/
```

## Advanced Usage

### Custom Module Development

Create a new Magisk module:

```bash
# modules/magisk/custom-hooks/
mkdir -p custom-hooks/{system,META-INF/com/google/android}

cat > custom-hooks/module.prop << EOF
id=custom-hooks
name=Custom Security Hooks
version=1.0
versionCode=1
author=Security Team
description=Custom hooks for targeted app analysis
EOF

cat > custom-hooks/install.sh << 'EOF'
#!/system/bin/sh
MODPATH=${0%/*}
ui_print "Installing custom hooks..."
# Installation logic here
EOF

chmod +x custom-hooks/install.sh
zip -r custom-hooks.zip custom-hooks/
adb push custom-hooks.zip /sdcard/
# Install via Magisk Manager
```

This skill provides comprehensive guidance for AI coding agents to help developers set up and use the JAMBOREE Android security testing framework effectively.
