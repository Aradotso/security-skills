---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and rooted emulators for mobile penetration testing
triggers:
  - set up android security testing environment
  - configure burp suite with android emulator
  - install magisk modules for android pentesting
  - set up objection frida hooks on android
  - create rooted android testing sandbox
  - intercept android app traffic with burp
  - bypass ssl pinning on android app
  - configure jamboree android toolkit
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk modules** for systemless root and privilege escalation
- **Burp Suite** for HTTPS traffic interception and manipulation
- **Objection/Frida** for runtime instrumentation and method hooking
- **Rooted Android emulators** optimized for security testing

This framework eliminates the fragmentation of traditional Android pentesting by providing a cohesive, pre-configured environment for application security audits, malware analysis, and reverse engineering.

## Installation

### Prerequisites

Ensure the following are installed:

```bash
# Verify Java Development Kit (11+)
java -version

# Verify Android SDK and platform tools
adb version
echo $ANDROID_HOME

# Verify Python (for Objection)
python3 --version
pip3 --version

# Verify Frida tools
frida --version
```

### Core Framework Setup

```bash
# Clone the JAMBOREE repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration setup script
chmod +x orchestration/setup.sh
./orchestration/setup.sh

# Verify installation
./orchestration/validate.sh
```

### Install Objection

```bash
pip3 install objection
objection --help
```

### Configure Burp Suite Certificate

```bash
# Export Burp Suite CA certificate (DER format)
# Place it in: modules/burp/certificates/burp-ca.der

# Convert to PEM for Android system trust
openssl x509 -inform DER -in modules/burp/certificates/burp-ca.der \
  -out modules/burp/certificates/burp-ca.pem

# Calculate certificate hash for system store
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old \
  -in modules/burp/certificates/burp-ca.pem | head -1)
cp modules/burp/certificates/burp-ca.pem \
  modules/burp/certificates/${CERT_HASH}.0
```

## Key Components

### 1. Emulator Configuration

Create a rooted Android emulator with Magisk:

```bash
# Create AVD with Google APIs (rootable)
avdmanager create avd -n jamboree-test \
  -k "system-images;android-30;google_apis;x86_64" \
  -d "pixel_4"

# Start emulator with writable system partition
emulator -avd jamboree-test -writable-system -no-snapshot-load &

# Wait for emulator to boot
adb wait-for-device

# Remount system as writable
adb root
adb remount

# Install Magisk via orchestration script
./orchestration/deployers/deploy_magisk.sh
```

### 2. Magisk Module Deployment

```bash
# List available Magisk modules
ls modules/magisk/

# Deploy systemless modules
./orchestration/deployers/deploy_magisk_modules.sh

# Verify Magisk installation
adb shell su -c "magisk -v"

# Install custom modules
adb push modules/magisk/systemless/custom-module.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/custom-module.zip"

# Reboot to apply modules
adb reboot
adb wait-for-device
```

### 3. Burp Suite Proxy Configuration

```bash
# Configure device to use Burp proxy
# Burp typically runs on 127.0.0.1:8080

# Set up ADB port forwarding
adb reverse tcp:8080 tcp:8080

# Configure global proxy via ADB
adb shell settings put global http_proxy $(adb shell ip route | awk '{print $9}'):8080

# Install Burp CA certificate as system certificate
adb root
adb remount
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old \
  -in modules/burp/certificates/burp-ca.pem | head -1)
adb push modules/burp/certificates/${CERT_HASH}.0 \
  /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### 4. Objection Runtime Instrumentation

```bash
# Explore a running application
objection --gadget "com.example.app" explore

# Common Objection commands (interactive mode)
# List activities
android hooking list activities

# List classes
android hooking list classes

# Search for specific methods
android hooking search methods <keyword>

# Bypass SSL pinning
android sslpinning disable

# Dump memory
memory dump all <output-dir>

# List loaded classes
android hooking list class_methods <class-name>

# Hook a specific method
android hooking watch class_method <class>.<method> --dump-args --dump-return
```

## Common Workflows

### Workflow 1: Basic App Traffic Interception

```bash
#!/bin/bash
# intercept_app.sh

APP_PACKAGE="com.target.app"
PROXY_HOST="127.0.0.1"
PROXY_PORT="8080"

# Start emulator with proxy
adb reverse tcp:${PROXY_PORT} tcp:${PROXY_PORT}

# Install target app
adb install target_app.apk

# Launch app with Objection
objection --gadget "${APP_PACKAGE}" explore

# In Objection shell:
# android sslpinning disable
# android intent launch_activity ${APP_PACKAGE}/.MainActivity
```

### Workflow 2: SSL Pinning Bypass Script

```python
# ssl_pinning_bypass.py
# Run with: objection --gadget com.target.app run ssl_pinning_bypass.py

import frida
import sys

def on_message(message, data):
    print(f"[*] {message}")

js_code = """
Java.perform(function() {
    // Hook OkHttp3 CertificatePinner
    var CertificatePinner = Java.use('okhttp3.CertificatePinner');
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(hostname, peerCertificates) {
        console.log('[+] SSL Pinning bypass for: ' + hostname);
        return;
    };

    // Hook SSLContext
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    SSLContext.init.overload('[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom').implementation = function(keyManager, trustManager, secureRandom) {
        console.log('[+] SSLContext.init() bypassed');
        this.init.overload('[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom').call(this, keyManager, null, secureRandom);
    };

    console.log('[+] SSL Pinning bypass installed');
});
"""

device = frida.get_usb_device()
pid = device.spawn([sys.argv[1]])
session = device.attach(pid)
script = session.create_script(js_code)
script.on('message', on_message)
script.load()
device.resume(pid)
sys.stdin.read()
```

### Workflow 3: Root Detection Bypass

```javascript
// root_detection_bypass.js
// Load with: objection --gadget com.target.app run root_detection_bypass.js

Java.perform(function() {
    // Common root detection methods
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    RootBeer.isRooted.implementation = function() {
        console.log('[+] Root detection bypassed (RootBeer)');
        return false;
    };

    // Hook common file checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.indexOf('su') !== -1 || path.indexOf('magisk') !== -1) {
            console.log('[+] File existence check bypassed: ' + path);
            return false;
        }
        return this.exists.call(this);
    };

    // Hook Runtime.exec
    var Runtime = Java.use('java.lang.Runtime');
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.indexOf('su') !== -1 || cmd.indexOf('which') !== -1) {
            console.log('[+] Runtime.exec bypassed: ' + cmd);
            throw new Error('Command not found');
        }
        return this.exec.call(this, cmd);
    };

    console.log('[+] Root detection bypass installed');
});
```

### Workflow 4: Automated Security Assessment

```bash
#!/bin/bash
# automated_assessment.sh

TARGET_APK="$1"
APP_PACKAGE=$(aapt dump badging "$TARGET_APK" | grep package | awk '{print $2}' | sed "s/name='\(.*\)'/\1/")

echo "[*] Starting automated assessment for: $APP_PACKAGE"

# 1. Install APK
adb install "$TARGET_APK"

# 2. Start Objection with SSL pinning bypass
objection --gadget "$APP_PACKAGE" explore <<EOF
android sslpinning disable
android hooking list activities
android hooking list services
android hooking search methods crypto
android intent launch_activity
memory list modules
exit
EOF

# 3. Capture traffic logs
adb logcat -d | grep -i "$APP_PACKAGE" > logs/${APP_PACKAGE}_logcat.txt

# 4. Extract APK data
adb shell run-as "$APP_PACKAGE" ls -la /data/data/"$APP_PACKAGE"/ > logs/${APP_PACKAGE}_filesystem.txt

echo "[*] Assessment complete. Review logs/ directory for results."
```

## Configuration Files

### Emulator Network Configuration

```bash
# configurations/network/emulator_proxy.conf
HTTP_PROXY=127.0.0.1:8080
HTTPS_PROXY=127.0.0.1:8080
NO_PROXY=localhost,127.0.0.1
```

### Objection Configuration

```yaml
# configurations/security/objection_config.yaml
frida:
  version: "16.0.0"
  server_mode: true
  gadget_path: "/data/local/tmp/frida-gadget.so"

objection:
  auto_ssl_pinning_bypass: true
  log_level: "debug"
  script_paths:
    - "./modules/objection/scripts"
    - "./modules/objection/plugins"

hooks:
  - name: "ssl_pinning"
    enabled: true
    script: "ssl_pinning_bypass.js"
  - name: "root_detection"
    enabled: true
    script: "root_detection_bypass.js"
```

### Burp Suite Extension Configuration

```json
// configurations/burp/extension_config.json
{
  "upstream_proxy": {
    "enabled": false,
    "host": "",
    "port": 0
  },
  "certificate_trust": {
    "auto_install": true,
    "system_level": true
  },
  "interception_rules": [
    {
      "match": "*.googleapis.com",
      "action": "intercept"
    },
    {
      "match": "*/api/*",
      "action": "intercept_and_log"
    }
  ]
}
```

## Troubleshooting

### Issue: ADB Device Not Found

```bash
# Check ADB connection
adb devices

# Restart ADB server
adb kill-server
adb start-server

# Verify emulator is running
emulator -list-avds
```

### Issue: Magisk Module Won't Load

```bash
# Boot into Magisk safe mode
adb shell setprop persist.sys.safemode 1
adb reboot

# Remove conflicting modules
adb shell su -c "rm -rf /data/adb/modules/<module-name>"

# Re-deploy modules
./orchestration/deployers/deploy_magisk_modules.sh
```

### Issue: Burp Certificate Not Trusted

```bash
# Verify certificate is in system store
adb shell ls -la /system/etc/security/cacerts/ | grep burp

# Check certificate permissions
adb shell ls -l /system/etc/security/cacerts/<cert-hash>.0

# Should be: -rw-r--r-- (644)
# If not, fix permissions:
adb shell su -c "chmod 644 /system/etc/security/cacerts/<cert-hash>.0"

# Reboot device
adb reboot
```

### Issue: Objection Cannot Attach to Process

```bash
# Verify Frida server is running
adb shell "ps | grep frida"

# Start Frida server manually
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell su -c "/data/local/tmp/frida-server &"

# Verify app is debuggable
aapt dump badging target.apk | grep "application: label"

# Force spawn mode instead of attach
objection --gadget "com.target.app" explore --startup-command "android hooking disable"
```

### Issue: SSL Pinning Bypass Not Working

```bash
# Try alternative bypass methods
objection --gadget "com.target.app" explore

# In Objection shell, try multiple techniques:
android sslpinning disable --quiet
android sslpinning disable --all
android sslpinning disable --universal

# If still failing, use custom Frida script
objection --gadget "com.target.app" run modules/objection/scripts/universal_ssl_bypass.js
```

### Issue: Emulator Performance Degradation

```bash
# Allocate more resources to emulator
emulator -avd jamboree-test \
  -memory 4096 \
  -cores 4 \
  -gpu swiftshader_indirect \
  -no-snapshot-load

# Clear emulator cache
rm -rf ~/.android/avd/jamboree-test.avd/cache/*

# Reduce logging overhead
adb shell setprop log.tag.JAMBOREE ERROR
```

## Advanced Usage

### Custom Frida Hook for API Interception

```javascript
// api_interceptor.js
Java.perform(function() {
    var ApiClient = Java.use('com.example.app.network.ApiClient');
    
    ApiClient.makeRequest.implementation = function(endpoint, payload) {
        console.log('[*] API Request Intercepted');
        console.log('    Endpoint: ' + endpoint);
        console.log('    Payload: ' + payload);
        
        var result = this.makeRequest(endpoint, payload);
        
        console.log('[*] API Response: ' + result);
        return result;
    };
});
```

### Automated Credential Extraction

```javascript
// credential_harvester.js
Java.perform(function() {
    var SharedPreferences = Java.use('android.content.SharedPreferences');
    var Editor = Java.use('android.content.SharedPreferences$Editor');
    
    Editor.putString.implementation = function(key, value) {
        if (key.toLowerCase().indexOf('token') !== -1 || 
            key.toLowerCase().indexOf('password') !== -1 ||
            key.toLowerCase().indexOf('secret') !== -1) {
            console.log('[!] Credential Captured');
            console.log('    Key: ' + key);
            console.log('    Value: ' + value);
        }
        return this.putString(key, value);
    };
});
```

## Environment Variables

```bash
# Set these in your shell profile
export JAMBOREE_HOME="/path/to/Android-Mobile-Security-Sandbox-Testing"
export ANDROID_HOME="/path/to/Android/Sdk"
export BURP_PROXY_HOST="127.0.0.1"
export BURP_PROXY_PORT="8080"
export FRIDA_SERVER_PATH="${JAMBOREE_HOME}/tools/frida-server"
```

## Integration with CI/CD

```yaml
# .github/workflows/android-security-scan.yml
name: Android Security Scan
on: [push]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JAMBOREE
        run: |
          git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
          cd Android-Mobile-Security-Sandbox-Testing
          ./orchestration/setup.sh
      - name: Run automated assessment
        run: |
          ./orchestration/automated_assessment.sh ${{ github.workspace }}/app.apk
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: security-scan-results
          path: logs/
```
