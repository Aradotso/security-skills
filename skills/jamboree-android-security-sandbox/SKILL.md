---
name: jamboree-android-security-sandbox
description: Set up and use JAMBOREE framework for Android security testing with Magisk, Burp Suite, Objection, and rooted emulators
triggers:
  - "set up android security testing environment"
  - "configure jamboree for android pentesting"
  - "intercept android app traffic with burp suite"
  - "use objection to hook android runtime"
  - "bypass ssl pinning on android"
  - "configure magisk modules for security testing"
  - "set up rooted android emulator for testing"
  - "debug android app security with frida"
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk** for systemless root and module management
- **Burp Suite** for traffic interception and analysis
- **Objection** (Frida-based) for runtime instrumentation
- **Rooted AVD** configurations optimized for security testing

This framework eliminates the fragmentation of Android security testing by providing a pre-configured, self-healing environment for penetration testing, reverse engineering, and malware analysis.

## Installation

### Prerequisites

Ensure you have the required dependencies:

```bash
# Verify Java JDK (11+)
java -version

# Verify Android SDK
adb version

# Install Python (for Objection/Frida)
python3 --version
pip3 --version

# Install Frida tools
pip3 install frida-tools objection
```

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration setup script
./orchestration/setup.sh

# Validate environment
./orchestration/validators/check_environment.sh
```

### Configuration Files

Create your environment configuration:

```bash
# configurations/network/proxy.conf
BURP_HOST=127.0.0.1
BURP_PORT=8080
BURP_CERT_PATH=/path/to/burp-cert.der

# configurations/android/device.conf
ANDROID_SERIAL=emulator-5554
ANDROID_VERSION=11
ROOT_METHOD=magisk

# configurations/security/objection.conf
FRIDA_SERVER_VERSION=16.1.4
OBJECTION_STARTUP_SCRIPT=/path/to/startup.js
```

## Core Workflows

### 1. Starting the Testing Environment

```bash
# Start Android emulator with pre-configured AVD
emulator -avd JAMBOREE_Android_11 -writable-system -no-snapshot

# Wait for boot
adb wait-for-device

# Deploy Magisk modules
./orchestration/deployers/deploy_magisk.sh

# Start Frida server on device
adb push frida-server-16.1.4-android-x86_64 /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"

# Verify Frida connection
frida-ps -U
```

### 2. Burp Suite Proxy Configuration

```bash
# Install Burp CA certificate
adb push burp-cert.der /sdcard/
adb shell "su -c 'mv /sdcard/burp-cert.der /system/etc/security/cacerts/9a5ba575.0'"
adb shell "su -c 'chmod 644 /system/etc/security/cacerts/9a5ba575.0'"

# Configure global proxy
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# Or use proxy script
./modules/burp/configure_proxy.sh --host 127.0.0.1 --port 8080
```

### 3. Objection Runtime Hooking

```bash
# List running applications
frida-ps -Ua

# Attach to running app
objection -g com.example.targetapp explore

# Spawn app with Objection
objection -g com.example.targetapp explore --startup-command "android hooking watch class_method com.example.MainActivity.checkRoot"
```

### 4. Certificate Pinning Bypass

Create a startup script for automatic SSL pinning bypass:

```javascript
// modules/objection/scripts/ssl_bypass.js

Java.perform(function() {
    console.log("[*] SSL Pinning Bypass Active");
    
    // Hook TrustManager
    var TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    
    var TrustManagerImpl = Java.registerClass({
        name: 'com.custom.TrustManagerImpl',
        implements: [TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    // Hook SSLContext.init()
    SSLContext.init.overload(
        '[Ljavax.net.ssl.KeyManager;',
        '[Ljavax.net.ssl.TrustManager;',
        'java.security.SecureRandom'
    ).implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[+] SSLContext.init() hooked");
        this.init(keyManager, [TrustManagerImpl.$new()], secureRandom);
    };
});
```

Use the script:

```bash
# Load script on app spawn
objection -g com.example.app explore --startup-script modules/objection/scripts/ssl_bypass.js

# Or within Objection REPL
import ssl_bypass.js
```

## Common Objection Commands

```bash
# Inside Objection REPL (objection explore)

# List loaded classes
android hooking list classes

# Search for specific class
android hooking search classes MainActivity

# List class methods
android hooking list class_methods com.example.MainActivity

# Watch method calls
android hooking watch class_method com.example.MainActivity.checkRoot --dump-args --dump-return

# Get method implementation
android hooking get current_activity

# Dump SQLite databases
android sqlite dump com.example.app.db

# List shared preferences
android sslpinning disable

# Bypass root detection
android root disable

# Memory dump
memory dump all <output-file>
```

## Magisk Module Management

```bash
# List installed modules
adb shell "su -c 'ls /data/adb/modules'"

# Install custom module
adb push custom_module.zip /sdcard/
adb shell "su -c 'magisk --install-module /sdcard/custom_module.zip'"

# Enable/disable module
adb shell "su -c 'touch /data/adb/modules/module_name/disable'"
adb shell "su -c 'rm /data/adb/modules/module_name/disable'"

# Reboot to apply changes
adb reboot
```

### Creating Custom Magisk Module

```bash
# modules/magisk/custom/module.prop
id=jamboree_custom
name=JAMBOREE Custom Security Module
version=1.0
versionCode=1
author=YourName
description=Custom security testing module
```

```bash
# modules/magisk/custom/post-fs-data.sh
#!/system/bin/sh
MODDIR=${0%/*}

# Disable SafetyNet checks
resetprop ro.debuggable 0
resetprop ro.secure 1
```

## Runtime Method Hooking Examples

### Hook Cryptographic Operations

```javascript
// modules/objection/scripts/crypto_hook.js

Java.perform(function() {
    var Cipher = Java.use('javax.crypto.Cipher');
    
    Cipher.doFinal.overload('[B').implementation = function(input) {
        console.log("[*] Cipher.doFinal() called");
        console.log("[*] Input: " + bytesToHex(input));
        
        var result = this.doFinal(input);
        console.log("[*] Output: " + bytesToHex(result));
        
        return result;
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

### Hook Network Requests

```javascript
// modules/objection/scripts/network_hook.js

Java.perform(function() {
    var URL = Java.use('java.net.URL');
    var HttpURLConnection = Java.use('java.net.HttpURLConnection');
    
    URL.$init.overload('java.lang.String').implementation = function(url) {
        console.log("[*] URL requested: " + url);
        return this.$init(url);
    };
    
    HttpURLConnection.getInputStream.implementation = function() {
        console.log("[*] HTTP Response received");
        var stream = this.getInputStream();
        console.log("[*] Response Code: " + this.getResponseCode());
        return stream;
    };
});
```

### Bypass Root Detection

```javascript
// modules/objection/scripts/root_bypass.js

Java.perform(function() {
    console.log("[*] Root Detection Bypass Active");
    
    // Hook common root detection methods
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    RootBeer.isRooted.implementation = function() {
        console.log("[+] RootBeer.isRooted() bypassed");
        return false;
    };
    
    // Hook file existence checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes("su") || path.includes("magisk")) {
            console.log("[+] File.exists() bypassed for: " + path);
            return false;
        }
        return this.exists();
    };
});
```

## Traffic Interception Workflow

```bash
# 1. Configure Burp Suite listener on 8080
# 2. Start Burp and configure CA cert export

# 3. Push certificate to device
adb push burp-cacert.der /sdcard/

# 4. Install as system certificate
adb shell "su -c 'mount -o rw,remount /system'"
adb shell "su -c 'cp /sdcard/burp-cacert.der /system/etc/security/cacerts/9a5ba575.0'"
adb shell "su -c 'chmod 644 /system/etc/security/cacerts/9a5ba575.0'"
adb shell "su -c 'chown root:root /system/etc/security/cacerts/9a5ba575.0'"

# 5. Set global proxy
adb shell settings put global http_proxy 192.168.1.100:8080

# 6. Verify traffic in Burp Suite HTTP history

# 7. For apps with SSL pinning, use Objection
objection -g com.example.app explore -s "android sslpinning disable"
```

## Automation Scripts

### Complete Setup Script

```bash
#!/bin/bash
# orchestration/automated_setup.sh

set -e

EMULATOR_NAME="JAMBOREE_Android_11"
PACKAGE_NAME="$1"

echo "[*] Starting JAMBOREE environment..."

# Start emulator
emulator -avd $EMULATOR_NAME -no-snapshot-load &
EMULATOR_PID=$!

# Wait for device
adb wait-for-device
sleep 10

# Deploy Frida
adb push frida-server /data/local/tmp/
adb shell "su -c 'chmod 755 /data/local/tmp/frida-server'"
adb shell "su -c '/data/local/tmp/frida-server &'" &

# Configure proxy
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# Install target app if provided
if [ -n "$PACKAGE_NAME" ]; then
    adb install -r "$PACKAGE_NAME"
fi

echo "[+] Environment ready. Starting Objection..."
objection -g $(adb shell pm list packages | grep -v "^package:" | head -n1) explore
```

### Cleanup Script

```bash
#!/bin/bash
# orchestration/cleanup.sh

echo "[*] Cleaning up JAMBOREE environment..."

# Kill Frida server
adb shell "su -c 'killall frida-server'"

# Remove proxy settings
adb shell settings put global http_proxy :0

# Stop emulator
adb emu kill

echo "[+] Cleanup complete"
```

## Troubleshooting

### Frida Connection Issues

```bash
# Check Frida server is running
adb shell "ps -A | grep frida"

# Restart Frida server
adb shell "su -c 'killall frida-server'"
adb shell "su -c '/data/local/tmp/frida-server &'"

# Check port forwarding
adb forward tcp:27042 tcp:27042

# Test connection
frida-ps -U
```

### Certificate Trust Issues

```bash
# Verify certificate is installed
adb shell "su -c 'ls -la /system/etc/security/cacerts/ | grep 9a5ba575'"

# Check certificate hash
openssl x509 -inform DER -in burp-cert.der -subject_hash_old -noout

# Reinstall for Android 11+
adb shell "su -c 'mount -o rw,remount /'"
adb shell "su -c 'cp /sdcard/9a5ba575.0 /system/etc/security/cacerts/'"
adb reboot
```

### Magisk Module Not Loading

```bash
# Enter safe mode
adb reboot
# Hold Volume Down during boot

# Check module status
adb shell "su -c 'magisk --list'"

# Check logs
adb shell "su -c 'logcat | grep Magisk'"

# Reinstall module
adb shell "su -c 'rm -rf /data/adb/modules/module_name'"
adb push module.zip /sdcard/
adb shell "su -c 'magisk --install-module /sdcard/module.zip'"
```

### Objection Crashes

```bash
# Update Frida/Objection
pip3 install --upgrade frida-tools objection

# Check app is debuggable
adb shell "dumpsys package com.example.app | grep debuggable"

# Force spawn mode instead of attach
objection -g com.example.app explore

# Use specific Frida version
objection -g com.example.app --frida-version 16.1.4 explore
```

## Environment Variables

Always reference sensitive values via environment variables:

```bash
# ~/.bashrc or project .env
export BURP_HOST="127.0.0.1"
export BURP_PORT="8080"
export BURP_CERT_PATH="/path/to/cert.der"
export ANDROID_HOME="/path/to/sdk"
export FRIDA_SERVER_PATH="/data/local/tmp/frida-server"
export JAMBOREE_CONFIG_PATH="./configurations"
```

## Best Practices

1. **Always test in isolated environments** - Use dedicated AVDs, never production devices
2. **Version control your scripts** - Keep custom hooks and configurations in version control
3. **Document your findings** - Use structured logging from the framework
4. **Rotate test environments** - Rebuild AVDs periodically to avoid fingerprinting
5. **Keep tools updated** - Regularly update Frida, Objection, and Magisk modules
6. **Validate before automation** - Test manual workflows before scripting them

## Advanced Use Cases

### Automated API Fuzzing

```bash
# Use Burp Suite with Objection hooks
objection -g com.example.app explore -s "android hooking watch class_method com.api.Client.sendRequest --dump-args"

# In Burp Suite, use Intruder on captured requests
```

### Memory Dumping Sensitive Data

```bash
# Dump heap memory
objection -g com.example.app explore -c "memory dump all /tmp/heap_dump.bin"

# Search for patterns
strings /tmp/heap_dump.bin | grep -i "password\|token\|api"
```

### Persistent Hooking Across Reboots

```bash
# Create Magisk module with Frida gadget injection
# modules/magisk/persistent_hook/service.sh
#!/system/bin/sh

while [ "$(getprop sys.boot_completed)" != "1" ]; do
    sleep 1
done

/data/local/tmp/frida-server &
```

This skill provides comprehensive coverage of JAMBOREE framework usage for AI coding agents assisting with Android security testing workflows.
