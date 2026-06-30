---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for mobile pentesting
triggers:
  - set up android security testing environment
  - configure jamboree android sandbox
  - intercept android app traffic with burp
  - hook android app with objection and frida
  - bypass android certificate pinning
  - configure magisk modules for security testing
  - set up rooted android emulator for pentesting
  - automate android security assessment workflow
---

# JAMBOREE Android Security Sandbox

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk modules** for systemless root and runtime modifications
- **Burp Suite** for HTTP/HTTPS traffic interception
- **Objection** (Frida-based) for runtime instrumentation and hooking
- **Rooted emulator environments** optimized for security testing

This framework eliminates fragmented tool setups by providing a cohesive, pre-configured sandbox for Android application security assessment, reverse engineering, and malware analysis.

## Installation

### Prerequisites

Ensure these tools are installed and in your PATH:

```bash
# Java Development Kit (11+)
java -version

# Android SDK platform tools
adb version

# Python 3.8+ for Objection/Frida
python3 --version
pip3 --version

# Node.js (optional, for some Frida scripts)
node --version
```

### Core Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Install Python dependencies
pip3 install frida-tools objection

# Verify Frida installation
frida --version
objection --version
```

### Environment Configuration

Create a configuration file `config.env`:

```bash
# Android SDK paths
export ANDROID_HOME="/path/to/android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$PATH"

# Burp Suite configuration
export BURP_JAR_PATH="/path/to/burpsuite.jar"
export BURP_PROXY_HOST="127.0.0.1"
export BURP_PROXY_PORT="8080"

# Certificate paths
export BURP_CERT_PATH="./modules/burp/certificates/burp-ca.der"
export SYSTEM_CERT_PATH="/system/etc/security/cacerts"

# Frida/Objection settings
export FRIDA_SERVER_VERSION="16.1.4"
export OBJECTION_SCRIPTS_PATH="./modules/objection/scripts"
```

Load configuration:

```bash
source config.env
```

## Emulator Setup

### Create Rooted Android Virtual Device

```bash
# Create AVD (Android 11, x86_64)
avdmanager create avd \
  --name "JAMBOREE_Security" \
  --package "system-images;android-30;google_apis;x86_64" \
  --device "pixel_4"

# Launch emulator in writable system mode
emulator -avd JAMBOREE_Security \
  -writable-system \
  -no-snapshot-load \
  -http-proxy $BURP_PROXY_HOST:$BURP_PROXY_PORT &

# Wait for device
adb wait-for-device
```

### Install Magisk

```bash
# Download Magisk APK (replace with actual URL)
wget -O magisk.apk https://github.com/topjohnwu/Magisk/releases/download/v26.1/Magisk-v26.1.apk

# Install Magisk
adb install magisk.apk

# Patch boot image (manual step in Magisk app)
# Then flash patched boot image
adb reboot bootloader
fastboot flash boot magisk_patched.img
fastboot reboot
```

### Deploy Frida Server

```bash
# Download Frida server for appropriate architecture
ARCH="x86_64"  # or "arm64" for ARM devices
wget https://github.com/frida/frida/releases/download/${FRIDA_SERVER_VERSION}/frida-server-${FRIDA_SERVER_VERSION}-android-${ARCH}.xz
xz -d frida-server-${FRIDA_SERVER_VERSION}-android-${ARCH}.xz
mv frida-server-${FRIDA_SERVER_VERSION}-android-${ARCH} frida-server

# Push and run Frida server
adb root
adb remount
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"

# Verify Frida is running
frida-ps -U
```

## Burp Suite Integration

### Install CA Certificate as System Certificate

```bash
# Export Burp CA certificate (DER format) from Burp Suite first
# Then convert and install

# Convert DER to PEM
openssl x509 -inform DER -in burp-ca.der -out burp-ca.pem

# Get certificate hash
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)

# Rename and push to system
cp burp-ca.pem ${CERT_HASH}.0
adb root
adb remount
adb push ${CERT_HASH}.0 /system/etc/security/cacerts/
adb shell "chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0"
adb reboot
```

### Configure Proxy Settings

```bash
# Set global proxy via ADB
adb shell settings put global http_proxy $BURP_PROXY_HOST:$BURP_PROXY_PORT

# Or configure WiFi proxy programmatically
adb shell settings put global wifi_proxy_host $BURP_PROXY_HOST
adb shell settings put global wifi_proxy_port $BURP_PROXY_PORT

# Clear proxy when done
adb shell settings delete global http_proxy
```

## Objection Usage

### Basic Runtime Exploration

```bash
# List running processes
frida-ps -Ua

# Attach to application
objection -g com.example.targetapp explore

# Inside Objection REPL
android hooking list activities
android hooking list services
android intent launch_activity com.example.MainActivity
```

### SSL Pinning Bypass

```bash
# Automatic SSL pinning bypass
objection -g com.example.targetapp explore --startup-command "android sslpinning disable"

# Or within REPL
android sslpinning disable
```

### Common Objection Commands

```javascript
// Inside Objection REPL

// Memory search
memory list modules
memory list exports libcrypto.so
memory search "api_key" --string

// Heap search
android heap search instances com.example.User
android heap print_methods com.example.User

// File system
file download /data/data/com.example.app/databases/app.db ./app.db
file upload ./modified.xml /data/data/com.example.app/shared_prefs/config.xml

// SQLite database
sqlite connect /data/data/com.example.app/databases/app.db
sqlite execute "SELECT * FROM users"

// SharedPreferences
android sharedpreferences dump
```

### Custom Frida Scripts

Create `modules/objection/scripts/custom-hook.js`:

```javascript
// Hook specific method
Java.perform(function() {
    var TargetClass = Java.use("com.example.TargetClass");
    
    TargetClass.sensitiveMethod.implementation = function(arg1, arg2) {
        console.log("[*] sensitiveMethod called");
        console.log("[*] arg1: " + arg1);
        console.log("[*] arg2: " + arg2);
        
        // Call original method
        var result = this.sensitiveMethod(arg1, arg2);
        console.log("[*] Original result: " + result);
        
        // Modify return value
        return "MODIFIED_RESULT";
    };
    
    console.log("[*] Hook installed on sensitiveMethod");
});
```

Run custom script:

```bash
frida -U -l modules/objection/scripts/custom-hook.js -f com.example.targetapp
```

### Certificate Pinning Bypass Script

Create `modules/objection/scripts/ssl-bypass.js`:

```javascript
Java.perform(function() {
    console.log("[*] SSL Pinning Bypass Loaded");
    
    // Hook TrustManager
    var X509TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    
    var TrustManager = Java.registerClass({
        name: 'com.custom.TrustManager',
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() {
                return [];
            }
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
    
    console.log("[*] SSL Pinning Bypass Active");
});
```

## Magisk Module Management

### List Installed Modules

```bash
# Via ADB
adb shell su -c "magisk --list"

# Check module status
adb shell "ls -la /data/adb/modules/"
```

### Install Custom Module

Structure for systemless modifications:

```bash
modules/magisk/custom-module/
├── module.prop
├── install.sh
├── uninstall.sh
└── system/
    └── etc/
        └── hosts  # Example: hosts file modification
```

`module.prop`:

```ini
id=jamboree-custom
name=JAMBOREE Custom Module
version=1.0
versionCode=1
author=Security Tester
description=Custom systemless modifications for security testing
```

`install.sh`:

```bash
#!/system/bin/sh
MODPATH=${0%/*}
ui_print "Installing JAMBOREE Custom Module"
ui_print "- Copying files"
cp -af $MODPATH/system/* $MODPATH/system/
set_perm_recursive $MODPATH/system 0 0 0755 0644
ui_print "- Done"
```

Install module:

```bash
# Zip the module
cd modules/magisk/custom-module
zip -r custom-module.zip *

# Install via Magisk Manager or command line
adb push custom-module.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/custom-module.zip"
adb reboot
```

## Orchestration Workflows

### Complete Setup Script

Create `scripts/setup-jamboree.sh`:

```bash
#!/bin/bash
set -e

echo "[*] JAMBOREE Environment Setup"

# Source configuration
source config.env

# Start emulator
echo "[*] Starting emulator..."
emulator -avd JAMBOREE_Security -writable-system -no-snapshot-load &
adb wait-for-device
sleep 10

# Root and remount
echo "[*] Rooting device..."
adb root
adb remount

# Deploy Frida server
echo "[*] Deploying Frida server..."
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
sleep 5

# Install Burp certificate
echo "[*] Installing Burp CA certificate..."
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)
cp burp-ca.pem ${CERT_HASH}.0
adb push ${CERT_HASH}.0 /system/etc/security/cacerts/
adb shell "chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0"

# Configure proxy
echo "[*] Configuring proxy..."
adb shell settings put global http_proxy $BURP_PROXY_HOST:$BURP_PROXY_PORT

# Start Burp Suite
echo "[*] Starting Burp Suite..."
java -jar "$BURP_JAR_PATH" &

echo "[*] Setup complete. Verify with 'frida-ps -U'"
```

Run setup:

```bash
chmod +x scripts/setup-jamboree.sh
./scripts/setup-jamboree.sh
```

### Testing Workflow Script

Create `scripts/test-app.sh`:

```bash
#!/bin/bash

APP_PACKAGE=$1

if [ -z "$APP_PACKAGE" ]; then
    echo "Usage: $0 <package_name>"
    exit 1
fi

echo "[*] Testing application: $APP_PACKAGE"

# Install app if APK provided
if [ -f "$2" ]; then
    echo "[*] Installing APK..."
    adb install -r "$2"
fi

# Clear app data
echo "[*] Clearing app data..."
adb shell pm clear $APP_PACKAGE

# Launch with Objection
echo "[*] Launching with Objection..."
objection -g $APP_PACKAGE explore \
    --startup-command "android sslpinning disable" \
    --startup-command "android root disable" \
    --startup-command "android hooking list activities"
```

Usage:

```bash
./scripts/test-app.sh com.example.targetapp ./target.apk
```

## Common Patterns

### Root Detection Bypass

```javascript
// modules/objection/scripts/root-bypass.js
Java.perform(function() {
    // Common root detection methods
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    RootBeer.isRooted.implementation = function() {
        console.log("[*] Root check bypassed");
        return false;
    };
    
    // File-based checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes("su") || path.includes("magisk")) {
            console.log("[*] Hiding file: " + path);
            return false;
        }
        return this.exists();
    };
});
```

### API Key Extraction

```javascript
// Search memory for API keys
Java.perform(function() {
    Java.choose("com.example.ApiClient", {
        onMatch: function(instance) {
            console.log("[*] Found ApiClient instance");
            try {
                console.log("[*] API Key: " + instance.getApiKey());
            } catch(e) {
                console.log("[!] Error: " + e);
            }
        },
        onComplete: function() {}
    });
});
```

### Traffic Decryption Hook

```javascript
// Hook crypto operations
Java.perform(function() {
    var Cipher = Java.use('javax.crypto.Cipher');
    
    Cipher.doFinal.overload('[B').implementation = function(input) {
        console.log("[*] Cipher.doFinal called");
        console.log("[*] Input: " + bytesToHex(input));
        
        var result = this.doFinal(input);
        console.log("[*] Output: " + bytesToHex(result));
        
        return result;
    };
    
    function bytesToHex(bytes) {
        var hexArray = [];
        for (var i = 0; i < bytes.length; i++) {
            hexArray.push(("0" + (bytes[i] & 0xFF).toString(16)).slice(-2));
        }
        return hexArray.join('');
    }
});
```

## Troubleshooting

### Frida Connection Issues

```bash
# Check if Frida server is running
adb shell "ps | grep frida-server"

# Restart Frida server
adb shell "killall frida-server"
adb shell "/data/local/tmp/frida-server &"

# Check port forwarding
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

# Verify connection
frida-ps -U
```

### Certificate Not Trusted

```bash
# Verify certificate installation
adb shell "ls -l /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)"

# Check certificate format
openssl x509 -in /system/etc/security/cacerts/HASH.0 -text

# Reboot after installation
adb reboot
```

### Objection Fails to Attach

```bash
# Enable debugging in app
adb shell am set-debug-app -w --persistent com.example.targetapp

# Use spawn mode instead of attach
objection -g com.example.targetapp explore --startup-spawn

# Check SELinux mode (should be permissive)
adb shell getenforce
adb shell setenforce 0
```

### Proxy Not Working

```bash
# Verify proxy settings
adb shell settings get global http_proxy

# Test connectivity
adb shell ping -c 3 $BURP_PROXY_HOST

# Check iptables rules (if using VPN mode)
adb shell iptables -t nat -L -v

# Use ProxyDroid or VPN approach for system-wide proxy
```

### Magisk Module Not Loading

```bash
# Check module logs
adb shell su -c "cat /cache/magisk.log"

# Boot to safe mode (disable all modules)
adb reboot
# Hold volume down during boot

# Remove problematic module
adb shell su -c "rm -rf /data/adb/modules/module-name"
adb reboot
```

## Advanced Configurations

### Multi-Device Testing

```bash
# List connected devices
adb devices

# Target specific device
export ANDROID_SERIAL="emulator-5554"
adb -s $ANDROID_SERIAL shell

# Frida with specific device
frida-ps -D emulator-5554
```

### Custom Objection Plugin

Create `modules/objection/plugins/custom.py`:

```python
from objection.utils.plugin import Plugin

class CustomPlugin(Plugin):
    def __init__(self, ns):
        self.ns = ns
    
    def run(self, args):
        api = self.api()
        result = api.android_hooking_watch_class(args[0])
        print(f"[*] Watching class: {args[0]}")
        return result

namespace = 'custom'
plugin = CustomPlugin
```

Load plugin:

```bash
objection -g com.example.app explore --plugin-folder modules/objection/plugins/
# Then use: custom.run com.example.TargetClass
```

This skill provides comprehensive guidance for AI agents to help developers set up and use JAMBOREE for Android security testing, covering installation, configuration, common workflows, and troubleshooting.
