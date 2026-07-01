---
name: jamboree-android-security-sandbox
description: Configure and use JAMBOREE framework for Android security testing with Magisk, Burp Suite, Objection, and rooted emulators
triggers:
  - set up JAMBOREE Android security testing environment
  - configure Burp Suite proxy for Android emulator
  - use Objection to hook into Android app
  - bypass SSL certificate pinning on Android
  - install Magisk modules for Android security testing
  - intercept Android app traffic with Burp Suite
  - configure rooted Android emulator for penetration testing
  - use Frida and Objection for Android runtime analysis
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection

## What JAMBOREE Does

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk** for systemless root and module management
- **Burp Suite** for HTTPS traffic interception
- **Objection** (Frida-based) for runtime instrumentation
- **Rooted Android Emulators** optimized for security testing

The framework eliminates manual configuration by providing pre-wired integrations for penetration testing, malware analysis, and reverse engineering of Android applications.

## Installation

### Prerequisites

Ensure these tools are installed:

```bash
# Verify Java JDK 11+
java -version

# Verify Android SDK and platform tools
adb version

# Install Python 3.8+ for Objection
python3 --version

# Install Frida and Objection
pip3 install frida-tools objection
```

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration script (Phase 1: Validation)
./orchestration/init.sh --validate

# Deploy core components (Phase 2: Deployment)
./orchestration/init.sh --deploy

# Customize configuration (Phase 3: Calibration)
./orchestration/init.sh --configure
```

The deployment script will:
1. Install Magisk modules to the emulator
2. Configure Burp Suite CA certificate as system trusted
3. Set up Objection scripts and plugins
4. Configure proxy settings and route traffic

## Core Commands

### Emulator Management

```bash
# Start optimized Android emulator with Magisk
./scripts/start-emulator.sh --android-version 13 --with-magisk

# List available emulator configurations
./scripts/list-emulators.sh

# Configure emulator proxy settings
adb shell settings put global http_proxy <HOST>:<PORT>

# Remove proxy configuration
adb shell settings put global http_proxy :0
```

### Magisk Module Operations

```bash
# Install custom Magisk module
adb push modules/magisk/custom-module.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/custom-module.zip"

# List installed modules
adb shell su -c "ls /data/adb/modules"

# Enable/disable module
adb shell su -c "touch /data/adb/modules/<module-name>/disable"
adb shell su -c "rm /data/adb/modules/<module-name>/disable"

# Reboot to apply changes
adb reboot
```

### Burp Suite Integration

```bash
# Export Burp CA certificate
curl http://burp/cert -o cacert.der
openssl x509 -inform DER -in cacert.der -out cacert.pem

# Install certificate to Android system store
./scripts/install-burp-cert.sh --cert cacert.pem --emulator-name test_device

# Verify certificate installation
adb shell su -c "ls /system/etc/security/cacerts/"

# Configure iptables redirection (for non-proxy-aware apps)
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 127.0.0.1:8080"
```

### Objection Runtime Instrumentation

```bash
# List running applications
frida-ps -U

# Spawn app with Objection
objection --gadget com.example.targetapp explore

# Attach to running app
objection --gadget com.example.targetapp --attach explore

# Execute pre-configured script
objection --gadget com.example.targetapp run modules/objection/scripts/bypass-ssl.js
```

## Configuration Files

### Network Proxy Configuration

Edit `configurations/network/proxy.conf`:

```conf
# Burp Suite Proxy Settings
PROXY_HOST=127.0.0.1
PROXY_PORT=8080
PROXY_SSL_PORT=8443

# Certificate pinning bypass
ENABLE_SSL_BYPASS=true
SSL_BYPASS_MODE=objection

# Traffic logging
LOG_TRAFFIC=true
LOG_DIR=./logs/traffic
```

### Android Security Settings

Edit `configurations/android/security.conf`:

```conf
# Root hiding configuration
HIDE_MAGISK=true
HIDE_ROOT_APPS=com.example.banking,com.example.payment

# SafetyNet bypass
ENABLE_SAFETYNET_BYPASS=true

# Anti-emulation evasion
SPOOF_BUILD_PROPS=true
SPOOF_DEVICE_MODEL=Pixel 6 Pro
```

### Objection Plugin Configuration

Edit `configurations/objection/plugins.conf`:

```conf
# Auto-load plugins
PLUGINS_DIR=./modules/objection/plugins
AUTO_LOAD=ssl-pinning-bypass,root-detection-bypass,sqlite-explorer

# Hooking preferences
HOOK_MODE=spawn
ENABLE_SCRIPT_RUNTIME=true
```

## Real Code Examples

### 1. Complete SSL Pinning Bypass

```javascript
// modules/objection/scripts/bypass-ssl.js
Java.perform(function() {
    console.log("[*] Starting SSL pinning bypass");
    
    // Hook OkHttp3
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(str, list) {
            console.log('[+] OkHttp3 pinning bypassed for: ' + str);
            return;
        };
    } catch(err) {
        console.log('[-] OkHttp3 not found');
    }
    
    // Hook TrustManagerImpl
    try {
        var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
        TrustManagerImpl.verifyChain.implementation = function(untrustedCert, trustAnchorChain, host, clientAuth, ocspData, tlsSctData) {
            console.log('[+] TrustManager bypassed for: ' + host);
            return untrustedCert;
        };
    } catch(err) {
        console.log('[-] TrustManagerImpl not found');
    }
    
    console.log("[*] SSL pinning bypass complete");
});
```

Usage:

```bash
objection --gadget com.example.app explore --startup-script modules/objection/scripts/bypass-ssl.js
```

### 2. Root Detection Bypass

```javascript
// modules/objection/scripts/bypass-root-detection.js
Java.perform(function() {
    console.log("[*] Bypassing root detection");
    
    // Hook common root detection methods
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    RootBeer.isRooted.implementation = function() {
        console.log('[+] RootBeer.isRooted() bypassed');
        return false;
    };
    
    // Hook su binary checks
    var Runtime = Java.use('java.lang.Runtime');
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.includes('su')) {
            console.log('[+] Blocked su execution check');
            throw new Error('Command not found');
        }
        return this.exec(cmd);
    };
    
    // Hook file existence checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes('su') || path.includes('magisk')) {
            console.log('[+] Hiding root file: ' + path);
            return false;
        }
        return this.exists();
    };
});
```

### 3. Automated Testing Workflow Script

```bash
#!/bin/bash
# scripts/automated-test.sh

APP_PACKAGE="com.example.targetapp"
BURP_HOST="127.0.0.1"
BURP_PORT="8080"

echo "[*] Starting automated security test for $APP_PACKAGE"

# Step 1: Configure proxy
adb shell settings put global http_proxy $BURP_HOST:$BURP_PORT

# Step 2: Install Burp certificate
./scripts/install-burp-cert.sh --auto

# Step 3: Clear app data
adb shell pm clear $APP_PACKAGE

# Step 4: Start Objection with SSL bypass
objection --gadget $APP_PACKAGE explore \
  --startup-script modules/objection/scripts/bypass-ssl.js \
  --startup-script modules/objection/scripts/bypass-root-detection.js \
  --startup-command "android hooking watch class_method android.location.Location.getLatitude --dump-args" &

OBJECTION_PID=$!

# Step 5: Execute monkey testing
adb shell monkey -p $APP_PACKAGE -v 1000

# Step 6: Extract data
adb pull /data/data/$APP_PACKAGE/databases ./output/databases/
adb pull /data/data/$APP_PACKAGE/shared_prefs ./output/shared_prefs/

# Step 7: Cleanup
kill $OBJECTION_PID
adb shell settings put global http_proxy :0

echo "[+] Test complete. Results in ./output/"
```

### 4. Custom Magisk Module for Method Tracing

```bash
# modules/magisk/method-tracer/install.sh
#!/system/bin/sh

MODDIR=${0%/*}

# Create tracing directory
mkdir -p /data/local/tmp/traces

# Install Frida server
cp $MODDIR/frida-server-16.0.19-android-arm64 /data/local/tmp/frida-server
chmod 755 /data/local/tmp/frida-server

# Start Frida on boot
echo "/data/local/tmp/frida-server &" > /data/adb/service.d/frida.sh
chmod 755 /data/adb/service.d/frida.sh

ui_print "[+] Method tracing module installed"
```

## Common Patterns

### Pattern 1: Intercepting HTTPS Traffic

```bash
# Start Burp Suite listening on 8080
# Configure and install certificate
./scripts/install-burp-cert.sh

# Set system-wide proxy
adb shell settings put global http_proxy 127.0.0.1:8080

# For apps that ignore proxy settings, use iptables
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 8080"
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDIRECT --to-port 8080"

# Launch app with Objection SSL bypass
objection --gadget com.example.app explore --startup-script modules/objection/scripts/bypass-ssl.js
```

### Pattern 2: Runtime Method Hooking

```bash
# Inside Objection REPL
objection> android hooking list classes
objection> android hooking search classes payment
objection> android hooking watch class_method com.example.PaymentManager.processPayment --dump-args --dump-return --dump-backtrace

# Save hooked method to script
objection> android hooking generate simple com.example.PaymentManager
```

### Pattern 3: Database Extraction and Analysis

```bash
# List databases
objection> android sqlite connect /data/data/com.example.app/databases/userdata.db

# Query database
objection> android sqlite execute schema
objection> android sqlite execute query SELECT * FROM users

# Export database
adb pull /data/data/com.example.app/databases/userdata.db ./analysis/

# Decrypt encrypted databases (if key found in memory)
objection> memory dump all from_base 0x12345678 length 32
```

### Pattern 4: SharedPreferences Manipulation

```bash
# Inside Objection
objection> android prefs show com.example.app

# Modify preference value
objection> env
objection> android prefs set_string com.example.app isPremium "true"

# Watch for preference changes
objection> android hooking watch class_method android.content.SharedPreferences\$Editor.putString --dump-args
```

## Troubleshooting

### Issue: Certificate Not Trusted

**Symptom**: Apps still fail SSL handshake despite certificate installation

**Solution**:

```bash
# Remount system as writable
adb shell su -c "mount -o rw,remount /system"

# Verify certificate hash
openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1

# Install with correct naming
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1)
adb push cacert.pem /sdcard/
adb shell su -c "cp /sdcard/cacert.pem /system/etc/security/cacerts/${CERT_HASH}.0"
adb shell su -c "chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0"
adb reboot
```

### Issue: Objection Cannot Find Process

**Symptom**: `Failed to spawn: unable to find process with name 'com.example.app'`

**Solution**:

```bash
# Ensure Frida server is running
adb shell su -c "ps | grep frida"

# If not running, start manually
adb shell su -c "/data/local/tmp/frida-server &"

# Use PID instead of package name
PROC_PID=$(adb shell pidof com.example.app)
frida -U -p $PROC_PID -l modules/objection/scripts/bypass-ssl.js
```

### Issue: Magisk Module Not Loading

**Symptom**: Module installed but not active after reboot

**Solution**:

```bash
# Check module structure
adb shell su -c "ls -la /data/adb/modules/your-module/"

# Verify module.prop exists
adb shell su -c "cat /data/adb/modules/your-module/module.prop"

# Check for disable flag
adb shell su -c "ls /data/adb/modules/your-module/disable"
adb shell su -c "rm /data/adb/modules/your-module/disable"

# Check Magisk logs
adb shell su -c "logcat | grep Magisk"
```

### Issue: Emulator Detected by App

**Symptom**: App refuses to run, shows "emulator detected" message

**Solution**:

```bash
# Apply build.prop spoofing
adb shell su -c "mount -o rw,remount /system"
adb shell su -c "sed -i 's/ro.build.tags=test-keys/ro.build.tags=release-keys/' /system/build.prop"
adb shell su -c "sed -i 's/ro.kernel.qemu=1/ro.kernel.qemu=0/' /system/build.prop"

# Use pre-configured anti-detection module
adb push modules/magisk/anti-detection.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/anti-detection.zip"
adb reboot
```

### Issue: Frida Detection by App

**Symptom**: App crashes or behaves differently when Frida is attached

**Solution**:

```javascript
// Use stealthy Frida injection
// modules/objection/scripts/stealth-mode.js
Java.perform(function() {
    // Rename Frida-related strings
    var System = Java.use('java.lang.System');
    System.getProperty.implementation = function(prop) {
        if (prop === 'frida' || prop === 'xposed') {
            return null;
        }
        return this.getProperty(prop);
    };
    
    // Hide Frida thread names
    var Thread = Java.use('java.lang.Thread');
    Thread.getName.implementation = function() {
        var name = this.getName();
        if (name.includes('frida') || name.includes('gmain')) {
            return 'DefaultThread';
        }
        return name;
    };
});
```

## Environment Variables

The framework respects these environment variables:

```bash
# Burp Suite configuration
export BURP_HOST=127.0.0.1
export BURP_PORT=8080
export BURP_CERT_PATH=/path/to/cacert.pem

# Android SDK paths
export ANDROID_HOME=/path/to/android-sdk
export ANDROID_AVD_HOME=/path/to/.android/avd

# Frida configuration
export FRIDA_SERVER_PATH=/data/local/tmp/frida-server

# Logging
export JAMBOREE_LOG_LEVEL=DEBUG
export JAMBOREE_LOG_DIR=./logs
```

Use these in scripts:

```bash
adb shell settings put global http_proxy ${BURP_HOST:-127.0.0.1}:${BURP_PORT:-8080}
```
