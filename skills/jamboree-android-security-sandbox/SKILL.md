---
name: jamboree-android-security-sandbox
description: Unified Android security testing framework integrating Magisk, Burp Suite, Objection, and root emulation for penetration testing and reverse engineering
triggers:
  - "set up Android security testing environment"
  - "configure JAMBOREE for mobile app pentesting"
  - "bypass certificate pinning with Objection"
  - "intercept Android app traffic with Burp Suite"
  - "set up rooted Android emulator for testing"
  - "analyze Android app with Frida hooks"
  - "configure Magisk modules for security testing"
  - "troubleshoot Android proxy certificate issues"
---

# JAMBOREE Android Security Sandbox

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a comprehensive Android security testing framework that unifies:

- **Magisk** for systemless root and module management
- **Burp Suite** for traffic interception and manipulation
- **Objection** (Frida) for runtime instrumentation and hooking
- **Rooted Android emulators** optimized for security testing

This creates a repeatable, automated environment for penetration testing, reverse engineering, and malware analysis of Android applications.

## Installation

### Prerequisites

```bash
# Verify Java Development Kit
java -version  # Should be JDK 11+

# Verify Android SDK and platform tools
adb version
emulator -version

# Verify Python for Objection
python3 --version  # Should be 3.8+
pip3 --version
```

### Core Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration script
./orchestration/deploy.sh

# Or manually configure components
./orchestration/validators/check-dependencies.sh
./orchestration/deployers/setup-magisk.sh
./orchestration/deployers/setup-objection.sh
./orchestration/deployers/setup-burp.sh
```

### Install Objection

```bash
# Install Objection via pip
pip3 install objection

# Install Frida tools
pip3 install frida-tools

# Verify installation
objection version
frida --version
```

## Environment Configuration

### 1. Emulator Setup

```bash
# Create Android Virtual Device with the provided configuration
avdmanager create avd -n jamboree_test \
  -k "system-images;android-30;google_apis_playstore;x86_64" \
  -d pixel_4

# Copy JAMBOREE-optimized AVD config
cp configurations/android/config.ini ~/.android/avd/jamboree_test.avd/

# Launch emulator
emulator -avd jamboree_test -writable-system -no-snapshot-load
```

### 2. Magisk Module Deployment

```bash
# Push Magisk modules to device
adb root
adb remount
adb push modules/magisk/systemless/ /data/adb/modules/

# Install BusyBox module
adb push modules/magisk/busybox.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/busybox.zip"

# Reboot to activate modules
adb reboot
```

### 3. Burp Suite Certificate Installation

```bash
# Export Burp CA certificate (from Burp Suite > Proxy > Options)
# Save as burp-ca-cert.der

# Convert to PEM format
openssl x509 -inform DER -in burp-ca-cert.der -out burp-ca-cert.pem

# Get certificate hash
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca-cert.pem | head -1)

# Install as system certificate
adb root
adb remount
adb push burp-ca-cert.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### 4. Proxy Configuration

```bash
# Set global proxy (replace with your Burp Suite IP:PORT)
adb shell settings put global http_proxy ${BURP_PROXY_HOST}:${BURP_PROXY_PORT}

# Or configure per-WiFi network
adb shell settings put global http_proxy $(ip route | grep default | awk '{print $3}'):8080

# Verify proxy settings
adb shell settings get global http_proxy

# Clear proxy
adb shell settings put global http_proxy :0
```

## Core Commands and Usage

### Objection Runtime Instrumentation

```bash
# List running applications
frida-ps -Ua

# Spawn app with Objection
objection -g com.example.targetapp explore

# Attach to running app
objection -g com.example.targetapp attach
```

### Common Objection Commands (Interactive Shell)

```bash
# List activities
android hooking list activities

# Search for classes
android hooking search classes target

# Search for methods
android hooking search methods decrypt

# Hook a method
android hooking watch class_method com.example.Class.method --dump-args --dump-return

# Bypass SSL pinning
android sslpinning disable

# List loaded classes
android hooking list classes

# Dump memory
memory dump all target.dmp

# Export SQLite databases
sqlite connect /data/data/com.example.app/databases/app.db
sqlite execute "SELECT * FROM users;"

# Read SharedPreferences
android sharedpreferences dump com.example.app

# Monitor file system access
android hooking watch class_method java.io.File.$init --dump-args
```

### Frida Scripting Examples

#### Basic Hook Script

```javascript
// hook-example.js - Hook a specific method
Java.perform(function() {
    var targetClass = Java.use("com.example.app.SecurityManager");
    
    targetClass.isRooted.implementation = function() {
        console.log("[*] isRooted() called");
        console.log("[*] Original return:", this.isRooted());
        return false; // Bypass root detection
    };
    
    targetClass.checkSignature.implementation = function(signature) {
        console.log("[*] checkSignature called with:", signature);
        return true; // Bypass signature verification
    };
});
```

#### SSL Pinning Bypass

```javascript
// ssl-bypass.js - Universal SSL pinning bypass
Java.perform(function() {
    // Hook OkHttp3 CertificatePinner
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
        console.log("[*] Certificate pinning bypassed for:", arguments[0]);
    };
    
    // Hook TrustManager
    var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    
    var TrustManager = Java.registerClass({
        name: 'com.sensepost.test.TrustManager',
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    var TrustManagers = [TrustManager.$new()];
    var SSLContext_init = SSLContext.init.overload(
        '[Ljavax.net.ssl.KeyManager;', 
        '[Ljavax.net.ssl.TrustManager;', 
        'java.security.SecureRandom'
    );
    
    SSLContext_init.implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[*] SSLContext.init() called, replacing TrustManager");
        SSLContext_init.call(this, keyManager, TrustManagers, secureRandom);
    };
});
```

#### Method Tracing and Logging

```javascript
// trace-api-calls.js - Trace specific API calls
Java.perform(function() {
    var ApiClient = Java.use("com.example.app.network.ApiClient");
    
    ApiClient.makeRequest.implementation = function(url, method, headers, body) {
        console.log("\n[*] === API Request ===");
        console.log("[*] URL:", url);
        console.log("[*] Method:", method);
        console.log("[*] Headers:", JSON.stringify(headers));
        console.log("[*] Body:", body);
        
        var result = this.makeRequest(url, method, headers, body);
        
        console.log("[*] Response:", result);
        console.log("[*] === End Request ===\n");
        
        return result;
    };
});
```

### Load Custom Frida Scripts

```bash
# Load script via Objection
objection -g com.example.app explore -s hook-example.js

# Load via Frida directly
frida -U -f com.example.app -l ssl-bypass.js --no-pause

# Spawn and load multiple scripts
frida -U -f com.example.app \
  -l ssl-bypass.js \
  -l trace-api-calls.js \
  --no-pause
```

## Configuration Files

### Burp Suite Integration Settings

```ini
# configurations/burp/proxy-settings.ini
[Proxy]
ListenAddress=0.0.0.0
ListenPort=8080
EnableSSL=true
CertificateFile=/path/to/burp-ca-cert.pem

[Upstream]
UseUpstreamProxy=false
UpstreamHost=
UpstreamPort=

[SSL]
SupportInvisibleProxying=true
GenerateCertsWithSpecificHostname=true
```

### Android Network Security Config

```xml
<!-- Place in res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </base-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

## Common Workflows

### Complete App Analysis Workflow

```bash
#!/bin/bash
# workflow-full-analysis.sh

APP_PACKAGE="com.example.targetapp"
BURP_HOST="${BURP_PROXY_HOST}"
BURP_PORT="${BURP_PROXY_PORT}"

# 1. Configure proxy
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# 2. Install and launch app
adb install -r target-app.apk
adb shell monkey -p ${APP_PACKAGE} -c android.intent.category.LAUNCHER 1

# 3. Start Objection with SSL bypass
objection -g ${APP_PACKAGE} explore <<EOF
android sslpinning disable
android hooking list activities
android hooking watch class_method ${APP_PACKAGE}.MainActivity.onCreate --dump-args
EOF

# 4. Export data after testing
objection -g ${APP_PACKAGE} explore <<EOF
android sharedpreferences dump ${APP_PACKAGE}
sqlite connect /data/data/${APP_PACKAGE}/databases/main.db
sqlite execute "SELECT * FROM users;"
file download /data/data/${APP_PACKAGE}/files/config.json ./exported/
EOF

# 5. Cleanup
adb shell settings put global http_proxy :0
```

### Automated Root Detection Bypass

```javascript
// auto-bypass-root.js - Comprehensive root detection bypass
Java.perform(function() {
    console.log("[*] Loading root detection bypass");
    
    // Common root detection methods
    var rootPackages = [
        "com.noshufou.android.su",
        "com.thirdparty.superuser",
        "eu.chainfire.supersu",
        "com.koushikdutta.superuser",
        "com.topjohnwu.magisk"
    ];
    
    // Hook PackageManager
    var PackageManager = Java.use("android.app.ApplicationPackageManager");
    PackageManager.getPackageInfo.overload('java.lang.String', 'int').implementation = function(pkg, flags) {
        if (rootPackages.indexOf(pkg) >= 0) {
            console.log("[*] Hiding package:", pkg);
            throw Java.use("android.content.pm.PackageManager$NameNotFoundException").$new();
        }
        return this.getPackageInfo(pkg, flags);
    };
    
    // Hook file existence checks
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        var suspicious = ["/system/app/Superuser.apk", "/sbin/su", "/system/bin/su", "/system/xbin/su"];
        
        if (suspicious.indexOf(path) >= 0) {
            console.log("[*] Hiding file:", path);
            return false;
        }
        return this.exists();
    };
    
    // Hook Runtime.exec for su commands
    var Runtime = Java.use("java.lang.Runtime");
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.indexOf("su") >= 0) {
            console.log("[*] Blocked su command:", cmd);
            throw Java.use("java.io.IOException").$new("Operation not permitted");
        }
        return this.exec(cmd);
    };
});
```

### Dynamic Analysis Script

```bash
#!/bin/bash
# dynamic-analysis.sh - Automated dynamic analysis

APP_PACKAGE="$1"
OUTPUT_DIR="analysis-output"

mkdir -p ${OUTPUT_DIR}

# Start logcat capture
adb logcat -c
adb logcat > ${OUTPUT_DIR}/logcat.txt &
LOGCAT_PID=$!

# Start network capture
adb shell "tcpdump -i any -s 0 -w /sdcard/capture.pcap" &
TCPDUMP_PID=$!

# Launch app with Objection hooks
cat > /tmp/objection-batch.txt <<EOF
android hooking list activities
android hooking list services
android hooking search methods crypto
android intent launch_activity ${APP_PACKAGE}
android hooking watch class_method javax.crypto.Cipher.doFinal --dump-args --dump-return
android heap search instances android.content.SharedPreferences
EOF

objection -g ${APP_PACKAGE} explore -c /tmp/objection-batch.txt

# Interactive testing period
echo "Perform manual testing. Press Enter when done..."
read

# Cleanup and export
kill ${LOGCAT_PID}
adb shell "kill ${TCPDUMP_PID}"
adb pull /sdcard/capture.pcap ${OUTPUT_DIR}/
adb shell rm /sdcard/capture.pcap

echo "Analysis complete. Results in ${OUTPUT_DIR}/"
```

## Troubleshooting

### Certificate Trust Issues

```bash
# Verify certificate installation
adb shell "ls -la /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in burp-ca-cert.pem | head -1)"

# Check Android version certificate location (Android 14+)
adb shell "ls -la /apex/com.android.conscrypt/cacerts/"

# Manually verify trust
adb shell "openssl s_client -connect example.com:443 -CApath /system/etc/security/cacerts/"

# Force reinstall system certificate
adb root
adb disable-verity
adb reboot
adb root
adb remount
adb push burp-ca-cert.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### Objection/Frida Connection Failures

```bash
# Verify Frida server version matches client
frida --version
adb shell "/data/local/tmp/frida-server --version"

# Download matching Frida server
wget https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-x86_64.xz
unxz frida-server-*.xz
chmod +x frida-server-*
adb push frida-server-* /data/local/tmp/frida-server

# Start Frida server
adb shell "chmod +x /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"

# Verify connection
frida-ps -Ua

# If app is not debuggable, patch AndroidManifest.xml
apktool d target-app.apk
# Edit AndroidManifest.xml: android:debuggable="true"
apktool b target-app -o target-app-debug.apk
# Sign and install
```

### Magisk Module Conflicts

```bash
# List active modules
adb shell "ls /data/adb/modules/"

# Disable specific module
adb shell "touch /data/adb/modules/[module-name]/disable"
adb reboot

# Remove module
adb shell "rm -rf /data/adb/modules/[module-name]"
adb reboot

# Boot into safe mode (Magisk disabled)
adb shell "setprop persist.sys.safemode 1"
adb reboot
```

### Proxy Not Intercepting Traffic

```bash
# Verify proxy settings
adb shell settings get global http_proxy
adb shell settings get global https_proxy

# Test proxy connectivity
adb shell "curl -x http://${BURP_PROXY_HOST}:${BURP_PROXY_PORT} http://example.com"

# Check app-specific network security config
adb pull /data/app/com.example.app-*/base.apk
unzip base.apk res/xml/network_security_config.xml

# Force VPN mode if global proxy fails
# Use Burp Suite Mobile Assistant or ProxyDroid

# Verify no firewall/SELinux blocks
adb shell "iptables -L -n"
adb shell "getenforce"
adb shell "setenforce 0"  # Temporarily disable SELinux
```

## Environment Variables

Required environment variables for automation scripts:

```bash
# Burp Suite proxy configuration
export BURP_PROXY_HOST="192.168.1.100"
export BURP_PROXY_PORT="8080"

# Android SDK paths
export ANDROID_HOME="/path/to/android-sdk"
export PATH="${PATH}:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator"

# Frida/Objection configuration
export FRIDA_VERSION="16.1.4"

# Target application
export TARGET_PACKAGE="com.example.app"
```

## Integration with CI/CD

```yaml
# .github/workflows/android-security-test.yml
name: Android Security Testing
on: [push]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup JAMBOREE
        run: |
          ./orchestration/deploy.sh
          
      - name: Start emulator
        run: |
          emulator -avd jamboree_test -no-window -no-audio &
          adb wait-for-device
          
      - name: Install and test app
        run: |
          adb install -r target-app.apk
          ./workflows/automated-pentest.sh ${{ secrets.TARGET_PACKAGE }}
          
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: security-test-results
          path: analysis-output/
```

## Best Practices

1. **Always work in isolated environments** — Use dedicated emulators or test devices
2. **Keep Frida and Objection updated** — Version mismatches cause connection failures
3. **Use environment variables** — Never hardcode proxy addresses or credentials
4. **Save Objection scripts** — Create reusable `.js` files for common bypass patterns
5. **Document your hooks** — Comment complex Frida scripts for team collaboration
6. **Automate repetitive tasks** — Create bash scripts for common workflow sequences
7. **Monitor resource usage** — Emulators can consume significant CPU/memory
8. **Verify certificate trust** — Always check system certificate installation before testing

This skill enables AI agents to guide developers through comprehensive Android security testing using the JAMBOREE framework's unified toolchain.
