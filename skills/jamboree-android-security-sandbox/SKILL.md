---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and rooted emulators for penetration testing.
triggers:
  - set up android security testing environment
  - configure burp suite for android testing
  - install magisk modules for penetration testing
  - use objection to hook android app
  - bypass ssl pinning on android
  - intercept android app traffic with burp
  - configure rooted android emulator for security testing
  - set up jamboree android testing framework
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk modules** for systemless root and environment modification
- **Burp Suite proxy** for HTTPS traffic interception
- **Objection/Frida** for runtime instrumentation and hooking
- **Rooted Android emulators** optimized for security testing

This framework eliminates the fragmentation of manual tool configuration and provides a cohesive, repeatable environment for Android application security assessment.

## Installation

### Prerequisites

Ensure these tools are installed:

```bash
# Java Development Kit 11+
java -version

# Android SDK with platform tools
echo $ANDROID_HOME
adb version

# Python 3.8+ for Objection
python3 --version
pip3 --version

# Burp Suite (Community or Professional)
# Download from https://portswigger.net/burp
```

### Core Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration setup script
chmod +x orchestration/setup.sh
./orchestration/setup.sh

# The script will:
# 1. Validate dependencies
# 2. Deploy Magisk modules
# 3. Configure Burp Suite integration
# 4. Install Objection and Frida
# 5. Set up Android emulator with root
```

### Install Objection and Frida

```bash
# Install Objection (Frida-based runtime exploration tool)
pip3 install objection

# Install Frida tools
pip3 install frida-tools

# Verify installation
objection version
frida --version
```

## Key Components and Configuration

### 1. Magisk Module Management

JAMBOREE includes pre-configured Magisk modules in `modules/magisk/`. These provide systemless root and environment modifications.

```bash
# Deploy Magisk modules to connected device
adb push modules/magisk/systemless/*.zip /sdcard/Download/
adb shell su -c "magisk --install-module /sdcard/Download/module.zip"

# Verify module installation
adb shell su -c "magisk --list"

# Enable systemless hosts for DNS interception
adb shell su -c "echo '127.0.0.1 api.example.com' >> /system/etc/hosts"
```

**Common Magisk Modules in JAMBOREE:**
- BusyBox for advanced shell commands
- MagiskHide configuration for root detection bypass
- Certificate pinning bypass modules

### 2. Burp Suite Proxy Configuration

Configure Android device/emulator to route traffic through Burp Suite.

```bash
# Set proxy on device (replace with your Burp Suite IP:port)
export BURP_HOST="192.168.1.100"
export BURP_PORT="8080"

adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# For system-wide proxy including HTTPS
adb shell settings put global global_http_proxy_host ${BURP_HOST}
adb shell settings put global global_http_proxy_port ${BURP_PORT}

# Clear proxy settings when done
adb shell settings put global http_proxy :0
```

**Install Burp CA Certificate:**

```bash
# Export Burp Suite CA certificate (DER format)
# In Burp: Proxy > Options > Import/Export CA certificate

# Convert to Android system format
openssl x509 -inform DER -in burp_ca.der -out burp_ca.pem
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp_ca.pem | head -1)
cp burp_ca.pem ${CERT_HASH}.0

# Push to system certificate store (requires root)
adb root
adb remount
adb push ${CERT_HASH}.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

**JAMBOREE Automated Certificate Installation:**

```bash
# Use the included certificate deployment script
./orchestration/deployers/deploy_burp_cert.sh --burp-export burp_ca.der

# This handles:
# - Certificate conversion
# - Hash generation
# - System store installation
# - Trust verification
```

### 3. Objection Runtime Instrumentation

Objection provides runtime manipulation of Android apps without repackaging.

```bash
# List running applications
frida-ps -U

# Attach to running app with Objection
objection -g com.example.app explore

# Or spawn app with Objection
objection -g com.example.app explore --startup-command "android hooking watch class_method com.example.MainActivity.onCreate"
```

**Common Objection Commands:**

```bash
# Inside Objection REPL

# SSL pinning bypass
android sslpinning disable

# List activities
android hooking list activities

# Hook method and trace calls
android hooking watch class_method com.example.api.ApiClient.sendRequest --dump-args --dump-return

# List loaded classes
android hooking list classes

# Search for specific class
android hooking search classes database

# Dump SharedPreferences
android hooking list shared_preferences
android hooking get shared_preferences com.example.app.prefs

# Explore SQLite databases
android sqlite connect /data/data/com.example.app/databases/app.db
.tables
SELECT * FROM users;

# Memory dump
memory dump all /sdcard/dump.bin

# File system operations
file ls /data/data/com.example.app/
file cat /data/data/com.example.app/shared_prefs/config.xml
```

**JAMBOREE Pre-Built Objection Scripts:**

```bash
# Run pre-configured scripts from modules/objection/scripts/
objection -g com.example.app explore --startup-script modules/objection/scripts/bypass_root_detection.js

# Common scripts included:
# - bypass_root_detection.js
# - ssl_pinning_bypass.js
# - anti_emulator_bypass.js
# - trace_crypto_operations.js
```

### 4. Emulator Configuration

JAMBOREE includes optimized AVD configurations in `configurations/android/`.

```bash
# Create rooted emulator using provided config
avdmanager create avd -n jamboree_testing -k "system-images;android-30;google_apis;x86_64" -c 4096M

# Start emulator with network settings
emulator -avd jamboree_testing -writable-system -http-proxy ${BURP_HOST}:${BURP_PORT}

# Install Magisk on emulator
adb push modules/magisk/magisk_installer.apk /sdcard/
adb shell pm install /sdcard/magisk_installer.apk

# Root emulator with Magisk
adb shell "/data/local/tmp/magisk --install"
adb reboot
```

**Emulator Anti-Detection Configuration:**

```javascript
// Apply build.prop modifications to evade emulator detection
// Located in: configurations/android/build.prop.template

adb root
adb remount
adb pull /system/build.prop
// Edit build.prop with realistic device values
adb push build.prop /system/build.prop
adb reboot
```

## Complete Workflow Example

### Scenario: Intercepting and Bypassing SSL Pinning

```bash
# 1. Start Burp Suite (ensure proxy listener on 0.0.0.0:8080)

# 2. Configure device proxy
export BURP_HOST="192.168.1.100"
export BURP_PORT="8080"
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# 3. Install Burp CA certificate
./orchestration/deployers/deploy_burp_cert.sh --burp-export ~/Downloads/burp_ca.der

# 4. Start target app with Objection
objection -g com.target.app explore

# 5. In Objection REPL, bypass SSL pinning
android sslpinning disable

# 6. Hook API method to view/modify requests
android hooking watch class_method com.target.app.network.ApiClient.post --dump-args --dump-return

# 7. Trigger app functionality and observe in Burp Suite

# 8. Export intercepted traffic
# In Burp: Proxy > HTTP History > Save items
```

### Scenario: Runtime Method Hooking with Custom Script

```javascript
// File: custom_hooks/login_bypass.js
Java.perform(function() {
    var AuthManager = Java.use("com.target.app.auth.AuthManager");
    
    // Hook isLoggedIn method
    AuthManager.isLoggedIn.implementation = function() {
        console.log("[*] isLoggedIn() called - returning true");
        return true;
    };
    
    // Hook validateCredentials method
    AuthManager.validateCredentials.implementation = function(username, password) {
        console.log("[*] validateCredentials() called");
        console.log("    Username: " + username);
        console.log("    Password: " + password);
        // Call original method
        var result = this.validateCredentials(username, password);
        console.log("    Original result: " + result);
        return true; // Always return success
    };
    
    console.log("[+] Auth hooks installed successfully");
});
```

**Load custom hook:**

```bash
objection -g com.target.app explore --startup-script custom_hooks/login_bypass.js
```

### Scenario: Database Extraction and Analysis

```bash
# Connect to app with Objection
objection -g com.target.app explore

# List databases
android sqlite list

# Connect to specific database
android sqlite connect /data/data/com.target.app/databases/userdata.db

# Explore schema
.schema

# Extract sensitive data
SELECT * FROM users WHERE role='admin';

# Export entire database
android sqlite export /data/data/com.target.app/databases/userdata.db /sdcard/exported_db.sqlite

# Pull to local machine
exit
adb pull /sdcard/exported_db.sqlite ./
```

## Advanced Configuration

### Network Traffic Routing

```bash
# Route specific domains through Burp Suite
# Edit: configurations/network/proxy_rules.conf

# Apply iptables rules for transparent proxying
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDIRECT --to-ports 8080"

# Verify rules
adb shell su -c "iptables -t nat -L -n -v"
```

### Persistent Frida Server

```bash
# Download Frida server for Android
wget https://github.com/frida/frida/releases/download/16.0.0/frida-server-16.0.0-android-x86_64.xz
unxz frida-server-16.0.0-android-x86_64.xz

# Push and run persistently
adb push frida-server-16.0.0-android-x86_64 /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "su -c '/data/local/tmp/frida-server &'"

# Verify Frida is listening
frida-ps -U
```

### Custom Magisk Module Creation

```bash
# JAMBOREE module template in: modules/magisk/template/

# Module structure:
# custom_module/
# ├── module.prop
# ├── install.sh
# ├── system/
# │   └── etc/
# │       └── hosts
# └── service.sh

# Example module.prop:
cat > module.prop << EOF
id=jamboree_custom
name=JAMBOREE Custom Module
version=1.0
versionCode=1
author=YourName
description=Custom security testing module
EOF

# Package module
cd custom_module
zip -r ../custom_module.zip *

# Install via Magisk Manager or adb
adb push custom_module.zip /sdcard/Download/
adb shell su -c "magisk --install-module /sdcard/Download/custom_module.zip"
```

## Troubleshooting

### Burp Suite Not Intercepting HTTPS Traffic

```bash
# Verify proxy settings
adb shell settings get global http_proxy

# Check certificate installation
adb shell su -c "ls -la /system/etc/security/cacerts/ | grep burp"

# Test connection
adb shell curl -x ${BURP_HOST}:${BURP_PORT} https://www.google.com -v

# Force app to use proxy (for apps ignoring system proxy)
adb shell su -c "iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner $(adb shell dumpsys package com.target.app | grep userId | cut -d= -f2) -j REDIRECT --to-port 8080"
```

### Objection Cannot Attach to Process

```bash
# Ensure Frida server is running
adb shell "su -c 'ps | grep frida-server'"

# If not running, start it
adb shell "su -c '/data/local/tmp/frida-server &'"

# Check app is debuggable
adb shell dumpsys package com.target.app | grep flags

# If not debuggable, make it debuggable (requires repackaging)
# Or use gadget injection method
objection patchapk --source target.apk

# Install patched APK
adb install target_patched.apk
```

### Magisk Module Not Loading

```bash
# Check Magisk logs
adb shell su -c "magisk --log"

# Enter safe mode to disable problematic modules
# Hold Volume Down during boot

# Remove conflicting module
adb shell su -c "rm -rf /data/adb/modules/problematic_module"
adb reboot

# Verify module compatibility
adb shell su -c "magisk --version"
```

### SSL Pinning Bypass Not Working

```bash
# Try alternative bypass methods

# Method 1: Universal SSL pinning bypass script
objection -g com.target.app explore --startup-script modules/objection/scripts/universal_ssl_bypass.js

# Method 2: Frida-based Objection command
objection -g com.target.app explore
android sslpinning disable --quiet

# Method 3: Use specific framework bypass
# For OkHttp3
android hooking watch class_method okhttp3.CertificatePinner.check --dump-args

# For TrustManager
android hooking watch class_method javax.net.ssl.X509TrustManager.checkServerTrusted --dump-args
```

### Emulator Detection Bypass

```bash
# Apply build.prop modifications
adb root
adb remount

# Modify build.prop with realistic device values
adb shell "setprop ro.product.model 'SM-G998B'"
adb shell "setprop ro.product.manufacturer 'samsung'"
adb shell "setprop ro.build.fingerprint 'samsung/SM-G998B/beyond2q:12/SP1A.210812.016/G998BXXU5DVLC:user/release-keys'"

# Hide Magisk from app
adb shell su -c "magiskhide enable"
adb shell su -c "magiskhide add com.target.app"

# Use JAMBOREE anti-detection module
adb push modules/magisk/anti_detection.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/anti_detection.zip"
adb reboot
```

## Environment Variables

```bash
# Set these in your shell profile for consistent configuration

export ANDROID_HOME="/path/to/android-sdk"
export BURP_HOST="192.168.1.100"
export BURP_PORT="8080"
export JAMBOREE_HOME="/path/to/Android-Mobile-Security-Sandbox-Testing"
export FRIDA_SERVER_PATH="/data/local/tmp/frida-server"

# Add to PATH
export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator
```

## Integration with CI/CD

```yaml
# Example GitHub Actions workflow for automated security testing
# File: .github/workflows/android_security_test.yml

name: Android Security Testing
on: [push]

jobs:
  security_scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup JAMBOREE
        run: |
          git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
          cd Android-Mobile-Security-Sandbox-Testing
          ./orchestration/setup.sh
      
      - name: Start Emulator
        run: |
          emulator -avd jamboree_testing -no-window -no-audio &
          adb wait-for-device
      
      - name: Run Security Tests
        run: |
          objection -g com.target.app explore --startup-script test_scripts/automated_security_check.js
```

## Key Files and Directories

```
JAMBOREE/
├── orchestration/
│   ├── setup.sh                    # Main setup script
│   ├── validators/                 # Dependency checkers
│   └── deployers/
│       ├── deploy_burp_cert.sh     # Burp certificate installer
│       └── deploy_magisk.sh        # Magisk module deployer
├── modules/
│   ├── magisk/
│   │   ├── systemless/             # Pre-built Magisk modules
│   │   └── template/               # Module creation template
│   ├── objection/
│   │   └── scripts/
│   │       ├── bypass_root_detection.js
│   │       ├── ssl_pinning_bypass.js
│   │       └── universal_ssl_bypass.js
│   └── burp/
│       ├── certificates/           # CA certificates
│       └── extensions/             # Burp Suite plugins
└── configurations/
    ├── android/
    │   ├── build.prop.template     # Device fingerprint config
    │   └── avd_configs/            # Emulator configurations
    └── network/
        └── proxy_rules.conf        # Traffic routing rules
```

This skill provides comprehensive guidance for using JAMBOREE to conduct Android application security assessments with integrated tooling for interception, instrumentation, and runtime manipulation.
