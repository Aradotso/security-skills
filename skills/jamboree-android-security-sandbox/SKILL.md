---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and rooted emulators for penetration testing and reverse engineering.
triggers:
  - set up android security testing environment
  - configure burp suite with android emulator
  - install magisk modules for penetration testing
  - bypass ssl pinning on android
  - use objection for runtime hooking
  - create rooted android testing sandbox
  - intercept android app traffic with burp
  - setup frida and objection on android
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk** for systemless root and module management
- **Burp Suite** for HTTPS traffic interception
- **Objection** (Frida-based) for runtime instrumentation
- **Rooted Android Emulators** optimized for security testing

This framework eliminates configuration fragmentation by providing pre-configured environments for mobile app penetration testing, malware analysis, and reverse engineering.

## Installation

### Prerequisites

```bash
# Verify Java Development Kit (11+)
java -version

# Verify Android SDK installation
echo $ANDROID_SDK_ROOT

# Verify ADB is accessible
adb version

# Install Python for Objection
python3 --version
pip3 --version
```

### Core Dependencies

```bash
# Install Frida tools
pip3 install frida-tools objection

# Install ADB if not present
# Ubuntu/Debian
sudo apt-get install android-tools-adb android-tools-fastboot

# macOS
brew install android-platform-tools

# Verify installations
frida --version
objection --version
```

### Burp Suite Setup

```bash
# Ensure Burp Suite Professional or Community is installed
# Download CA certificate from Burp: http://burp/cert

# Convert certificate for Android (if needed)
openssl x509 -inform DER -in cacert.der -out cacert.pem

# Get certificate hash for Android system trust
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1)
mv cacert.pem ${CERT_HASH}.0
```

## Configuration

### Environment Variables

```bash
# Set in ~/.bashrc or ~/.zshrc
export ANDROID_SDK_ROOT="$HOME/Android/Sdk"
export PATH="$PATH:$ANDROID_SDK_ROOT/platform-tools"
export PATH="$PATH:$ANDROID_SDK_ROOT/emulator"

# Burp Suite proxy configuration
export BURP_PROXY_HOST="127.0.0.1"
export BURP_PROXY_PORT="8080"

# Objection/Frida server configuration
export FRIDA_SERVER_PORT="27042"
```

### Emulator Setup

```bash
# Create AVD (Android Virtual Device) for testing
# Use Google APIs (not Play Store) for root compatibility
avdmanager create avd \
  -n "Security_Test_Android11" \
  -k "system-images;android-30;google_apis;x86_64" \
  -d "pixel_3a" \
  --force

# Start emulator with writable system
emulator -avd Security_Test_Android11 \
  -writable-system \
  -no-snapshot-load \
  -dns-server 8.8.8.8 \
  &

# Wait for boot
adb wait-for-device
```

### Magisk Installation on Emulator

```bash
# Download Magisk APK (example for v26.1)
wget https://github.com/topjohnwu/Magisk/releases/download/v26.1/Magisk-v26.1.apk

# Install Magisk Manager
adb install Magisk-v26.1.apk

# Root system using Magisk (requires boot.img patching)
# Extract boot.img from system image, patch via Magisk Manager, then:
adb root
adb remount
adb push patched_boot.img /sdcard/
# Follow Magisk Manager instructions to apply
```

### Burp Certificate Installation

```bash
# Push Burp CA certificate to system trust store
adb root
adb remount
adb push ${CERT_HASH}.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot

# Configure proxy on emulator
adb shell settings put global http_proxy ${BURP_PROXY_HOST}:${BURP_PROXY_PORT}

# Or use manual proxy in Android Settings > Network & Internet > Wi-Fi > Advanced
```

### Frida Server Deployment

```bash
# Download Frida server for Android architecture
FRIDA_VERSION=$(frida --version)
wget https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-x86_64.xz
unxz frida-server-${FRIDA_VERSION}-android-x86_64.xz

# Push to device and set permissions
adb push frida-server-${FRIDA_VERSION}-android-x86_64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server

# Start Frida server
adb shell "/data/local/tmp/frida-server &"
```

## Key Commands

### Emulator Management

```bash
# List available AVDs
emulator -list-avds

# Start specific AVD with security testing flags
emulator -avd Security_Test_Android11 \
  -writable-system \
  -http-proxy ${BURP_PROXY_HOST}:${BURP_PROXY_PORT} \
  -dns-server 8.8.8.8,8.8.4.4 \
  -no-snapshot-load \
  &

# Check emulator status
adb devices -l

# Root shell access
adb root && adb shell
```

### Objection Runtime Instrumentation

```bash
# List running applications
frida-ps -Ua

# Explore application with Objection
objection -g com.example.targetapp explore

# Common Objection commands (run within explore session):
# - List activities
android hooking list activities

# - List services
android hooking list services

# - Bypass SSL pinning
android sslpinning disable

# - Watch method calls
android hooking watch class_method com.example.TargetClass.sensitiveMethod --dump-args --dump-return

# - Memory search
memory search "password" --string

# - Dump loaded classes
android hooking list classes

# - Get environment info
env
```

### SSL Pinning Bypass

```bash
# Using Objection (easiest method)
objection -g com.example.targetapp explore
android sslpinning disable

# Using Frida script directly
frida -U -f com.example.targetapp -l ssl-pinning-bypass.js --no-pause
```

**ssl-pinning-bypass.js** (Frida script):

```javascript
Java.perform(function() {
    console.log("[*] Bypassing SSL Pinning...");
    
    // Bypass TrustManager
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    
    var TrustManagerImpl = Java.registerClass({
        name: "com.custom.TrustManagerImpl",
        implements: [TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    // Hook SSLContext.init()
    SSLContext.init.overload(
        "[Ljavax.net.ssl.KeyManager;",
        "[Ljavax.net.ssl.TrustManager;",
        "java.security.SecureRandom"
    ).implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[*] SSLContext.init() called, replacing TrustManager");
        var customTM = TrustManagerImpl.$new();
        this.init(keyManager, [customTM], secureRandom);
    };
    
    console.log("[+] SSL Pinning bypass complete");
});
```

### Traffic Interception Workflow

```bash
# 1. Start Burp Suite and configure listener on 8080

# 2. Set proxy on Android
adb shell settings put global http_proxy ${BURP_PROXY_HOST}:${BURP_PROXY_PORT}

# 3. Start target application
adb shell am start -n com.example.targetapp/.MainActivity

# 4. Launch Objection with SSL bypass
objection -g com.example.targetapp explore --startup-command "android sslpinning disable"

# 5. Observe traffic in Burp Suite HTTP History

# 6. Clear proxy when done
adb shell settings put global http_proxy :0
```

## Common Patterns

### Automated Setup Script

```bash
#!/bin/bash
# setup-jamboree.sh - Automated environment setup

set -e

echo "[*] Starting JAMBOREE setup..."

# Check prerequisites
command -v adb >/dev/null 2>&1 || { echo "ADB required"; exit 1; }
command -v frida >/dev/null 2>&1 || { echo "Frida required"; exit 1; }

# Start emulator
AVD_NAME="${1:-Security_Test_Android11}"
echo "[*] Starting emulator: $AVD_NAME"
emulator -avd "$AVD_NAME" -writable-system -no-snapshot-load &
EMULATOR_PID=$!

# Wait for device
echo "[*] Waiting for device..."
adb wait-for-device
sleep 10

# Deploy Frida server
echo "[*] Deploying Frida server..."
FRIDA_VERSION=$(frida --version)
FRIDA_SERVER="frida-server-${FRIDA_VERSION}-android-x86_64"
if [ ! -f "$FRIDA_SERVER" ]; then
    wget "https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/${FRIDA_SERVER}.xz"
    unxz "${FRIDA_SERVER}.xz"
fi

adb root
adb push "$FRIDA_SERVER" /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server &"

# Configure proxy
echo "[*] Configuring Burp proxy..."
adb shell settings put global http_proxy ${BURP_PROXY_HOST:-127.0.0.1}:${BURP_PROXY_PORT:-8080}

echo "[+] JAMBOREE setup complete!"
echo "[*] Emulator PID: $EMULATOR_PID"
echo "[*] Ready for testing"
```

### Dynamic Method Hooking

```javascript
// hook-crypto.js - Monitor cryptographic operations
Java.perform(function() {
    console.log("[*] Hooking cryptographic methods...");
    
    // Hook AES encryption
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function(input) {
        console.log("[Cipher] doFinal called");
        console.log("  Algorithm: " + this.getAlgorithm());
        console.log("  Input (hex): " + bytesToHex(input));
        
        var result = this.doFinal(input);
        console.log("  Output (hex): " + bytesToHex(result));
        return result;
    };
    
    // Hook key generation
    var KeyGenerator = Java.use("javax.crypto.KeyGenerator");
    KeyGenerator.generateKey.implementation = function() {
        console.log("[KeyGenerator] Generating key: " + this.getAlgorithm());
        return this.generateKey();
    };
    
    function bytesToHex(bytes) {
        var hex = [];
        for (var i = 0; i < bytes.length; i++) {
            hex.push(("0" + (bytes[i] & 0xFF).toString(16)).slice(-2));
        }
        return hex.join("");
    }
    
    console.log("[+] Crypto hooks installed");
});
```

Run with:
```bash
frida -U -f com.example.targetapp -l hook-crypto.js --no-pause
```

### Root Detection Bypass

```javascript
// bypass-root-detection.js
Java.perform(function() {
    console.log("[*] Bypassing root detection...");
    
    // Common root detection patterns
    var patterns = [
        { pkg: "com.noshufou.android.su", desc: "SuperSU package" },
        { pkg: "eu.chainfire.supersu", desc: "SuperSU EU" },
        { pkg: "com.topjohnwu.magisk", desc: "Magisk Manager" }
    ];
    
    // Hook PackageManager
    var PackageManager = Java.use("android.app.ApplicationPackageManager");
    PackageManager.getPackageInfo.overload("java.lang.String", "int").implementation = function(pkg, flags) {
        for (var i = 0; i < patterns.length; i++) {
            if (pkg === patterns[i].pkg) {
                console.log("[*] Blocked package check: " + patterns[i].desc);
                throw Java.use("android.content.pm.PackageManager$NameNotFoundException").$new();
            }
        }
        return this.getPackageInfo(pkg, flags);
    };
    
    // Hook Runtime.exec for su binary checks
    var Runtime = Java.use("java.lang.Runtime");
    Runtime.exec.overload("java.lang.String").implementation = function(cmd) {
        if (cmd.includes("su") || cmd.includes("which su")) {
            console.log("[*] Blocked su command: " + cmd);
            throw Java.use("java.io.IOException").$new("Permission denied");
        }
        return this.exec(cmd);
    };
    
    // Hook File.exists for common root files
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        var rootPaths = ["/system/app/Superuser.apk", "/system/xbin/su", "/system/bin/su"];
        if (rootPaths.indexOf(path) >= 0) {
            console.log("[*] Blocked file check: " + path);
            return false;
        }
        return this.exists();
    };
    
    console.log("[+] Root detection bypass active");
});
```

### Application Data Extraction

```bash
# Using Objection to extract app data
objection -g com.example.targetapp explore

# Inside Objection session:
# List SQLite databases
android sqlite list

# Connect to database
android sqlite connect /data/data/com.example.targetapp/databases/app.db

# Query data
android sqlite execute "SELECT * FROM users"

# Export SharedPreferences
android plist cat /data/data/com.example.targetapp/shared_prefs/prefs.xml

# Download files
file download /data/data/com.example.targetapp/files/sensitive.txt ./sensitive.txt
```

## Troubleshooting

### Frida Connection Issues

```bash
# Check if Frida server is running
adb shell ps | grep frida-server

# Restart Frida server
adb shell pkill frida-server
adb shell "/data/local/tmp/frida-server &"

# Verify connectivity
frida-ps -U

# Check port forwarding
adb forward --list

# Manual port forward if needed
adb forward tcp:27042 tcp:27042
```

### Certificate Trust Problems

```bash
# Verify certificate installation
adb shell ls -la /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1)

# Check certificate hash matches filename
EXPECTED_HASH=$(openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1)
echo "Expected filename: ${EXPECTED_HASH}.0"

# For Android 7+, install as user certificate if system install fails
adb push cacert.pem /sdcard/Download/
# Then manually install via Settings > Security > Install from storage

# Reboot after system certificate changes
adb reboot
```

### Objection Won't Attach

```bash
# Check if app is debuggable
adb shell dumpsys package com.example.targetapp | grep -A1 "flags="

# Force debuggable mode (requires root)
adb shell
su
pm grant com.example.targetapp android.permission.SET_DEBUG_APP
am set-debug-app -w --persistent com.example.targetapp

# Try spawn mode instead of attach
objection -g com.example.targetapp explore --startup-command "android hooking disable"

# Check Frida version compatibility
frida --version
objection --version
# Frida and objection versions should be compatible
```

### Proxy Not Intercepting Traffic

```bash
# Verify proxy settings
adb shell settings get global http_proxy

# Check if app respects system proxy
# Some apps ignore system proxy - use VPN mode instead

# Install ProxyDroid or similar for system-wide proxy
adb install ProxyDroid.apk

# Alternative: Use iptables redirect (requires root)
adb shell
su
iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination ${BURP_PROXY_HOST}:${BURP_PROXY_PORT}
iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination ${BURP_PROXY_HOST}:${BURP_PROXY_PORT}
```

### Magisk Module Conflicts

```bash
# Enter Safe Mode (Magisk modules disabled)
# Hold volume down during boot animation

# Remove conflicting modules via Magisk Manager or command line
adb shell
su
cd /data/adb/modules
ls -la
# Remove problematic module
rm -rf module_name/

# Reboot to normal mode
reboot
```

### Emulator Performance Issues

```bash
# Increase emulator RAM and CPU cores
emulator -avd Security_Test_Android11 \
  -memory 4096 \
  -cores 4 \
  -gpu swiftshader_indirect \
  -writable-system \
  &

# Use x86_64 images for better performance on x86 hosts
# Avoid ARM translation unless testing ARM-specific behavior

# Disable unnecessary services
adb shell pm disable-user --user 0 com.google.android.gms
```

## Advanced Techniques

### Automated SSL Pinning Detection

```javascript
// detect-pinning.js
Java.perform(function() {
    console.log("[*] Detecting SSL pinning implementation...");
    
    var detections = [];
    
    // Check for OkHttp CertificatePinner
    try {
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        detections.push("OkHttp CertificatePinner detected");
    } catch(e) {}
    
    // Check for TrustKit
    try {
        var TrustKit = Java.use("com.datatheorem.android.trustkit.TrustKit");
        detections.push("TrustKit framework detected");
    } catch(e) {}
    
    // Check for custom TrustManager implementations
    var classes = Java.enumerateLoadedClassesSync();
    classes.forEach(function(className) {
        if (className.includes("TrustManager") && !className.startsWith("javax")) {
            detections.push("Custom TrustManager: " + className);
        }
    });
    
    console.log("[+] Pinning detection results:");
    detections.forEach(function(d) {
        console.log("  - " + d);
    });
});
```

### Network Request Logger

```bash
# Using Objection's built-in request monitoring
objection -g com.example.targetapp explore --startup-command "android hooking watch class okhttp3.OkHttpClient"

# Or custom Frida script for detailed logging
frida -U -f com.example.targetapp -l log-requests.js --no-pause
```

## Best Practices

1. **Always use writable-system flag** when starting emulators for security testing
2. **Backup AVD snapshots** before making system modifications
3. **Use separate AVDs** for different Android versions and configurations
4. **Keep Frida/Objection updated** to avoid compatibility issues
5. **Document certificate hashes** for reproducible setups
6. **Use environment variables** for proxy/server configuration
7. **Test SSL pinning bypasses** on multiple Android versions
8. **Monitor Frida server logs** during active hooking sessions

This skill provides comprehensive coverage of Android security testing workflows using the JAMBOREE framework, enabling AI agents to assist developers with mobile application penetration testing, reverse engineering, and security analysis tasks.
