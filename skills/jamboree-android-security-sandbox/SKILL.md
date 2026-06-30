---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for mobile penetration testing
triggers:
  - set up android security testing environment
  - configure jamboree sandbox for mobile pentesting
  - intercept android app traffic with burp suite
  - hook android app with frida and objection
  - bypass ssl pinning on android
  - set up rooted android emulator for testing
  - configure magisk modules for security research
  - debug android app with runtime instrumentation
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk** for systemless root and module management
- **Burp Suite** for HTTPS traffic interception
- **Objection** for runtime manipulation via Frida
- **Rooted Android Emulators** optimized for security testing

This framework eliminates the fragmentation of manual tool configuration and provides a repeatable, cohesive environment for Android application security assessment.

## Installation

### Prerequisites

Ensure these tools are installed and in your PATH:

```bash
# Check Java (JDK 11+)
java -version

# Check Android SDK tools
adb version
emulator -version

# Check Python 3.8+ (for Objection)
python3 --version

# Check Node.js (for Frida)
node --version
npm --version
```

### Core Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration script (Phase 1: Validation)
./orchestration/deploy.sh --validate

# Deploy the full stack (Phase 2: Core Deployment)
./orchestration/deploy.sh --deploy

# Calibrate for your environment (Phase 3)
./orchestration/deploy.sh --calibrate
```

### Install Individual Components

```bash
# Install Frida tools
pip3 install frida-tools

# Install Objection
pip3 install objection

# Download Burp Suite CA certificate
./modules/burp/fetch-cert.sh

# Install Magisk modules
./modules/magisk/install-modules.sh
```

## Environment Configuration

### Setting Up the Android Emulator

```bash
# Create a rooted AVD with Magisk
./orchestration/create-avd.sh \
  --name "security-test-avd" \
  --android-version 13 \
  --arch x86_64 \
  --with-magisk

# Start the emulator
emulator -avd security-test-avd -writable-system -no-snapshot
```

### Configure Burp Suite Proxy

```bash
# Install Burp CA certificate as system certificate
./modules/burp/install-cert.sh --device emulator-5554

# Set up proxy routing (WiFi settings)
adb shell settings put global http_proxy 127.0.0.1:8080

# Verify proxy configuration
adb shell settings get global http_proxy
```

**Manual Burp Suite Configuration:**

1. Start Burp Suite and navigate to Proxy → Options
2. Add a proxy listener on `127.0.0.1:8080`
3. Export CA certificate (DER format) to `modules/burp/certificates/`
4. Run the installation script above

### Configure Objection & Frida

```bash
# Verify Frida server is running on device/emulator
adb shell "su -c '/data/local/tmp/frida-server &'"

# List available Frida processes
frida-ps -U

# Test Objection connection
objection -g <package-name> explore
```

## Key Workflows

### 1. Intercepting HTTPS Traffic

```bash
# Start the target application with proxy configured
adb shell am start -n com.example.app/.MainActivity

# In Burp Suite, navigate to Proxy → Intercept
# Enable "Intercept is on"

# Monitor traffic in HTTP history tab
# Modify and forward requests as needed
```

**Bypass Certificate Pinning:**

```bash
# Use pre-configured Objection script
objection -g com.example.app explore

# Inside Objection console
android sslpinning disable

# Verify bypass
android hooking list classes
```

### 2. Runtime Instrumentation with Objection

```javascript
// Save as modules/objection/scripts/dump-methods.js
Java.perform(function() {
    var targetClass = Java.use('com.example.app.SecurityManager');
    
    // Hook authentication method
    targetClass.authenticate.implementation = function(username, password) {
        console.log('[*] Authentication called');
        console.log('[*] Username: ' + username);
        console.log('[*] Password: ' + password);
        
        // Call original method
        var result = this.authenticate(username, password);
        console.log('[*] Result: ' + result);
        
        return result;
    };
});
```

**Load and execute:**

```bash
# Spawn app with Frida script
frida -U -l modules/objection/scripts/dump-methods.js -f com.example.app

# Or use Objection
objection -g com.example.app explore --startup-script dump-methods.js
```

### 3. Exploring Application Data

```bash
# Launch Objection
objection -g com.example.app explore

# Inside Objection console:

# List activities
android hooking list activities

# Dump shared preferences
android shmtx dump

# List loaded classes
android hooking list classes

# Search for specific classes
android hooking search classes crypto

# Export SQLite databases
sqlite connect /data/data/com.example.app/databases/app.db
.tables
SELECT * FROM users;
```

### 4. Bypassing Root Detection

```javascript
// Save as modules/objection/scripts/bypass-root.js
Java.perform(function() {
    // Hook common root detection methods
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    
    RootBeer.isRooted.implementation = function() {
        console.log('[*] Root check bypassed');
        return false;
    };
    
    RootBeer.isRootedWithoutBusyBoxCheck.implementation = function() {
        console.log('[*] Root check (no busybox) bypassed');
        return false;
    };
    
    // Hook file existence checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        
        // List of suspicious root-related paths
        var rootPaths = [
            '/system/app/Superuser.apk',
            '/system/xbin/su',
            '/system/bin/su',
            '/data/local/xbin/su',
            '/data/local/bin/su'
        ];
        
        if (rootPaths.indexOf(path) >= 0) {
            console.log('[*] Hiding root file: ' + path);
            return false;
        }
        
        return this.exists();
    };
});
```

**Execute:**

```bash
frida -U -l modules/objection/scripts/bypass-root.js -f com.example.app
```

### 5. Extracting and Analyzing APKs

```bash
# Pull APK from device
adb shell pm path com.example.app
adb pull /data/app/com.example.app-[random]/base.apk ./app.apk

# Decompile with apktool
apktool d app.apk -o ./decompiled

# Analyze with jadx
jadx app.apk -d ./jadx-output

# Search for sensitive strings
grep -r "api_key" ./jadx-output/
grep -r "password" ./jadx-output/
grep -r "http://" ./jadx-output/
```

## Common Frida Hooks

### Hook Constructor

```javascript
// modules/objection/scripts/hook-constructor.js
Java.perform(function() {
    var TargetClass = Java.use('com.example.app.User');
    
    TargetClass.$init.overload('java.lang.String', 'int').implementation = function(name, age) {
        console.log('[*] User constructor called');
        console.log('[*] Name: ' + name + ', Age: ' + age);
        return this.$init(name, age);
    };
});
```

### Hook Static Method

```javascript
// modules/objection/scripts/hook-static.js
Java.perform(function() {
    var Crypto = Java.use('com.example.app.CryptoUtil');
    
    Crypto.encrypt.implementation = function(plaintext, key) {
        console.log('[*] Encrypting: ' + plaintext);
        console.log('[*] Key: ' + key);
        
        var result = this.encrypt(plaintext, key);
        console.log('[*] Ciphertext: ' + result);
        
        return result;
    };
});
```

### Trace All Methods in a Class

```javascript
// modules/objection/scripts/trace-class.js
Java.perform(function() {
    var targetClass = Java.use('com.example.app.NetworkManager');
    var methods = targetClass.class.getDeclaredMethods();
    
    methods.forEach(function(method) {
        var methodName = method.getName();
        
        try {
            targetClass[methodName].implementation = function() {
                console.log('[*] Called: ' + methodName);
                console.log('[*] Arguments: ' + JSON.stringify(arguments));
                
                var result = this[methodName].apply(this, arguments);
                console.log('[*] Return: ' + result);
                
                return result;
            };
        } catch(e) {
            console.log('[!] Cannot hook: ' + methodName);
        }
    });
});
```

## Magisk Module Configuration

### Install Custom Module

```bash
# Structure of a Magisk module
# modules/magisk/custom-module/
# ├── module.prop
# ├── install.sh
# ├── uninstall.sh
# └── system/
#     └── [files to overlay]

# Example module.prop
cat > modules/magisk/ssl-unpinning/module.prop << EOF
id=ssl_unpinning
name=SSL Unpinning
version=1.0
versionCode=1
author=JAMBOREE
description=Systemless SSL certificate unpinning
EOF

# Install via adb
adb push modules/magisk/ssl-unpinning /sdcard/
adb shell su -c 'magisk --install-module /sdcard/ssl-unpinning.zip'
adb reboot
```

### Enable BusyBox

```bash
# Install BusyBox module
./modules/magisk/install-busybox.sh

# Verify installation
adb shell su -c 'busybox --help'
```

## Configuration Files

### Network Proxy Configuration

```bash
# configurations/network/proxy.conf
PROXY_HOST=127.0.0.1
PROXY_PORT=8080
PROXY_TYPE=http
UPSTREAM_PROXY=
UPSTREAM_PORT=
```

**Apply configuration:**

```bash
source configurations/network/proxy.conf
adb shell settings put global http_proxy $PROXY_HOST:$PROXY_PORT
```

### Security Testing Profile

```bash
# configurations/security/testing-profile.conf
TARGET_PACKAGE=com.example.app
ENABLE_SSL_BYPASS=true
ENABLE_ROOT_BYPASS=true
ENABLE_DEBUGGER_CHECK_BYPASS=true
LOG_LEVEL=verbose
OUTPUT_DIR=./logs
```

**Load profile:**

```bash
./orchestration/load-profile.sh configurations/security/testing-profile.conf
```

## Troubleshooting

### Frida Server Not Running

```bash
# Check if frida-server is present
adb shell ls -l /data/local/tmp/frida-server

# If missing, push frida-server
wget https://github.com/frida/frida/releases/download/[VERSION]/frida-server-[VERSION]-android-x86_64.xz
unxz frida-server-*.xz
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server

# Start frida-server with root
adb shell "su -c '/data/local/tmp/frida-server &'"

# Verify it's running
adb shell "su -c 'ps | grep frida'"
```

### Certificate Pinning Still Active

```bash
# Check if certificate is installed
adb shell "su -c 'ls -l /system/etc/security/cacerts/ | grep burp'"

# If not present, reinstall
./modules/burp/install-cert.sh --force

# Use alternative Frida script
frida -U -l modules/objection/scripts/universal-ssl-bypass.js -f com.example.app
```

### Objection Won't Connect

```bash
# List USB devices
frida-ls-devices

# If no device listed, check ADB
adb devices

# Restart ADB server
adb kill-server
adb start-server

# Reconnect Objection
objection -N -g com.example.app explore
```

### Magisk Module Not Loading

```bash
# Check Magisk logs
adb shell "su -c 'logcat | grep Magisk'"

# Boot into safe mode (disable all modules)
adb reboot

# Re-enable modules one by one via Magisk Manager
adb shell "su -c 'magisk --list-modules'"
```

### Emulator Performance Issues

```bash
# Increase emulator RAM and CPU
emulator -avd security-test-avd -memory 4096 -cores 4

# Disable animations
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0

# Use hardware acceleration
emulator -avd security-test-avd -gpu host
```

## Environment Variables

```bash
# Set these in your shell profile or .env file

export ANDROID_SDK_ROOT=/path/to/android-sdk
export ANDROID_HOME=$ANDROID_SDK_ROOT
export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools
export PATH=$PATH:$ANDROID_SDK_ROOT/emulator

# Burp Suite configuration
export BURP_PROXY_HOST=127.0.0.1
export BURP_PROXY_PORT=8080

# Frida configuration
export FRIDA_SERVER_PATH=/data/local/tmp/frida-server

# Objection logging
export OBJECTION_LOG_LEVEL=debug

# JAMBOREE workspace
export JAMBOREE_HOME=/path/to/Android-Mobile-Security-Sandbox-Testing
export JAMBOREE_OUTPUT_DIR=$JAMBOREE_HOME/logs
```

## Best Practices

1. **Always use isolated test environments** — Never test on production devices
2. **Keep Frida and Objection updated** — Compatibility issues arise with outdated versions
3. **Document your Magisk module stack** — Conflicts can be difficult to debug
4. **Use version control for custom scripts** — Track changes to your Frida hooks
5. **Validate certificate installations** — SSL issues are the #1 troubleshooting point
6. **Monitor logs continuously** — Use `adb logcat` in a separate terminal during testing

## Integration with CI/CD

```yaml
# Example GitHub Actions workflow
# .github/workflows/android-security-scan.yml
name: Android Security Scan

on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up JAMBOREE
        run: |
          git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
          cd Android-Mobile-Security-Sandbox-Testing
          ./orchestration/deploy.sh --validate --deploy
      
      - name: Start Emulator
        run: |
          ./orchestration/create-avd.sh --name ci-test --android-version 13
          emulator -avd ci-test -no-window -no-audio &
          adb wait-for-device
      
      - name: Install APK
        run: adb install app-debug.apk
      
      - name: Run Objection Scan
        run: |
          objection -g com.example.app explore --quiet --startup-command "android sslpinning disable"
          objection -g com.example.app run custom-scan.js
      
      - name: Upload Results
        uses: actions/upload-artifact@v3
        with:
          name: security-scan-results
          path: ${{ env.JAMBOREE_OUTPUT_DIR }}
```

This skill provides comprehensive guidance for AI coding agents to assist developers in configuring and using the JAMBOREE Android security testing framework effectively.
