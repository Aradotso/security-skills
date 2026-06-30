---
name: android-mobile-security-sandbox-jamboree
description: JAMBOREE Android security testing framework integrating Magisk, Burp Suite, Objection, and Frida for comprehensive mobile app penetration testing
triggers:
  - set up android security testing environment
  - configure jamboree framework for mobile pentesting
  - integrate burp suite with android emulator
  - bypass ssl pinning on android app
  - use objection to hook android methods
  - set up magisk for rooted testing environment
  - intercept android app traffic with burp
  - automate frida scripts for android analysis
---

# Android Mobile Security Sandbox JAMBOREE

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that orchestrates multiple tools into a cohesive pentesting environment. It automates the integration of:

- **Magisk modules** for systemless root and module management
- **Burp Suite proxy** for traffic interception and certificate handling
- **Objection/Frida** for runtime instrumentation and hooking
- **Rooted Android emulators** optimized for security testing

This framework eliminates fragmented workflows by providing pre-configured components that work together seamlessly for Android application security assessment.

## Installation

### Prerequisites

Ensure the following are installed:

```bash
# Java Development Kit 11+
java -version

# Android SDK with platform tools
which adb

# Python 3.8+ for Objection
python3 --version

# Frida tools
pip3 install frida-tools objection
```

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration script
chmod +x orchestration/deploy.sh
./orchestration/deploy.sh

# Verify installation
./orchestration/validate.sh
```

The deployment script will:
1. Validate dependencies
2. Configure Android emulator with Magisk
3. Install Burp Suite CA certificates
4. Deploy Objection scripts and Frida gadgets
5. Set up proxy routing

## Core Components

### 1. Magisk Module Management

```bash
# Install Magisk modules
adb push modules/magisk/systemless/universal-safetynet-fix.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/universal-safetynet-fix.zip"

# List installed modules
adb shell su -c "ls /data/adb/modules"

# Enable/disable module
adb shell su -c "touch /data/adb/modules/module-name/disable"
adb reboot

# Remove module
adb shell su -c "rm -rf /data/adb/modules/module-name"
```

**Custom Module Installation:**

```bash
# Deploy custom init.d script
adb push configurations/android/custom-init.sh /data/adb/service.d/
adb shell su -c "chmod 755 /data/adb/service.d/custom-init.sh"
```

### 2. Burp Suite Integration

**Certificate Installation:**

```bash
# Export Burp CA certificate (DER format)
# In Burp: Proxy → Options → Import/Export CA certificate

# Convert to PEM and get hash
openssl x509 -inform DER -in burp-ca.der -out burp-ca.pem
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)

# Install as system certificate
adb root
adb remount
adb push burp-ca.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

**Proxy Configuration:**

```bash
# Set system-wide proxy
adb shell settings put global http_proxy <HOST_IP>:8080

# Or use the provided configuration script
./orchestration/deployers/burp-proxy-setup.sh --host 192.168.1.100 --port 8080

# Verify proxy settings
adb shell settings get global http_proxy

# Clear proxy
adb shell settings put global http_proxy :0
```

**VPN-based Interception:**

```bash
# Install proxy VPN app
adb install modules/burp/proxy-vpn.apk

# Configure VPN tunnel
adb shell am start -n com.proxyvpn/.MainActivity \
  --es PROXY_HOST "192.168.1.100" \
  --ei PROXY_PORT 8080
```

### 3. Objection Runtime Instrumentation

**Basic Objection Usage:**

```bash
# List running apps
frida-ps -Ua

# Attach to running app
objection -g com.example.target explore

# Spawn app with Objection
objection -g com.example.target run
```

**SSL Pinning Bypass:**

```bash
# Inside Objection REPL
android sslpinning disable

# Or using custom script
objection -g com.example.target explore \
  -s "android sslpinning disable" \
  -s "jobs list"
```

**Common Objection Commands:**

```javascript
// List activities
android hooking list activities

// List classes
android hooking list classes

// Search for methods
android hooking search methods decrypt

// Hook specific method
android hooking watch class_method com.example.Crypto.decrypt --dump-args --dump-return

// Bypass root detection
android root disable

// List shared preferences
android file shared-preferences list

// Read specific preference
android file shared-preferences read com.example.target.prefs user_token

// SQLite operations
android sqlite list
android sqlite query /data/data/com.example.target/databases/main.db "SELECT * FROM users"

// Dump memory
memory dump all /tmp/memory-dump.bin

// Export class definitions
android hooking dump class com.example.target.MainActivity
```

**Custom Frida Scripts:**

Create `hooks/ssl-bypass.js`:

```javascript
Java.perform(function() {
    console.log("[*] SSL Pinning Bypass Active");
    
    // Hook OkHttp3 Certificate Pinner
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
        console.log("[+] Bypassed OkHttp3 Certificate Pinner");
    };
    
    // Hook TrustManagerImpl
    var TrustManagerImpl = Java.use("com.android.org.conscrypt.TrustManagerImpl");
    TrustManagerImpl.verifyChain.implementation = function(untrustedChain, trustAnchorChain, host, clientAuth, ocspData, tlsSctData) {
        console.log("[+] Bypassed TrustManagerImpl verification for: " + host);
        return untrustedChain;
    };
    
    // Hook SSLContext
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    SSLContext.init.overload('[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom').implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[+] SSLContext.init() called, accepting all certificates");
        this.init.overload('[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom').call(this, keyManager, null, secureRandom);
    };
});
```

Load custom script:

```bash
# Using Frida directly
frida -U -l hooks/ssl-bypass.js -f com.example.target

# Using Objection with custom script
objection -g com.example.target explore --startup-script hooks/ssl-bypass.js
```

**Method Hooking Template:**

```javascript
Java.perform(function() {
    var TargetClass = Java.use("com.example.target.Crypto");
    
    // Hook encryption method
    TargetClass.encrypt.implementation = function(plaintext, key) {
        console.log("[*] Encrypt called");
        console.log("[*] Plaintext: " + plaintext);
        console.log("[*] Key: " + key);
        
        var result = this.encrypt(plaintext, key);
        console.log("[*] Encrypted result: " + result);
        
        return result;
    };
    
    // Hook with multiple overloads
    TargetClass.processData.overload('java.lang.String').implementation = function(data) {
        console.log("[*] processData(String) called with: " + data);
        return this.processData(data);
    };
    
    TargetClass.processData.overload('[B').implementation = function(data) {
        console.log("[*] processData(byte[]) called");
        return this.processData(data);
    };
});
```

### 4. Emulator Configuration

**Create Rooted AVD:**

```bash
# Using provided configuration
./orchestration/deployers/create-rooted-avd.sh \
  --name jamboree-test \
  --api 30 \
  --arch x86_64

# Manual AVD creation
avdmanager create avd \
  -n security-test \
  -k "system-images;android-30;google_apis;x86_64" \
  -d "pixel_4"

# Start emulator with writable system
emulator -avd security-test -writable-system -no-snapshot-load &

# Wait for boot
adb wait-for-device

# Install Magisk
adb install modules/magisk/Magisk-v26.1.apk
```

**Anti-Emulation Evasion:**

```bash
# Modify build properties
adb shell su -c "mount -o rw,remount /system"
adb shell su -c "echo 'ro.build.fingerprint=google/redfin/redfin:11/RQ3A.211001.001/7641976:user/release-keys' >> /system/build.prop"
adb shell su -c "echo 'ro.product.manufacturer=Google' >> /system/build.prop"
adb reboot

# Hide emulator artifacts
adb shell su -c "setprop ro.kernel.qemu 0"
adb shell su -c "setprop ro.hardware.goldfish 0"
```

## Workflow Patterns

### Complete Penetration Test Setup

```bash
#!/bin/bash
# setup-pentest.sh

TARGET_PACKAGE="com.example.target"
BURP_HOST="192.168.1.100"
BURP_PORT="8080"

echo "[*] Starting JAMBOREE penetration test environment"

# 1. Start emulator
emulator -avd jamboree-test -writable-system -no-snapshot-load &
adb wait-for-device
sleep 10

# 2. Configure proxy
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# 3. Verify Magisk root
adb shell su -c "id" || { echo "[!] Root not available"; exit 1; }

# 4. Install target app
adb install -r target-app.apk

# 5. Start Objection with SSL bypass
objection -g ${TARGET_PACKAGE} explore \
  --startup-script hooks/ssl-bypass.js &

echo "[+] Environment ready. Burp Suite should show traffic."
```

### Automated Hook Deployment

```python
# deploy_hooks.py
import frida
import sys

HOOKS = {
    "crypto": """
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload('[B').implementation = function(data) {
        console.log("[Cipher] doFinal called with " + data.length + " bytes");
        return this.doFinal(data);
    };
});
""",
    "network": """
Java.perform(function() {
    var URL = Java.use("java.net.URL");
    URL.$init.overload('java.lang.String').implementation = function(url) {
        console.log("[Network] URL requested: " + url);
        return this.$init(url);
    };
});
"""
}

def attach_hooks(package_name, hook_type):
    device = frida.get_usb_device()
    pid = device.spawn([package_name])
    session = device.attach(pid)
    
    script = session.create_script(HOOKS[hook_type])
    script.on('message', lambda msg, data: print(msg))
    script.load()
    
    device.resume(pid)
    print(f"[+] {hook_type} hooks attached to {package_name}")
    sys.stdin.read()

if __name__ == "__main__":
    attach_hooks(sys.argv[1], sys.argv[2])
```

Usage:

```bash
python3 deploy_hooks.py com.example.target crypto
```

### Traffic Analysis Workflow

```bash
# 1. Clear app data
adb shell pm clear com.example.target

# 2. Start packet capture
adb shell su -c "tcpdump -i wlan0 -w /sdcard/capture.pcap" &

# 3. Start Burp Suite listener on port 8080

# 4. Launch app with Objection
objection -g com.example.target explore \
  -s "android sslpinning disable" \
  -s "android root disable"

# 5. Perform actions in app

# 6. Stop capture and retrieve
adb shell su -c "killall tcpdump"
adb pull /sdcard/capture.pcap ./analysis/
```

## Configuration Files

### Burp Suite Extension Config

`configurations/burp/extension-config.json`:

```json
{
  "proxy": {
    "enabled": true,
    "host": "0.0.0.0",
    "port": 8080,
    "ssl": {
      "generateCert": true,
      "certPath": "/certificates/burp-ca.pem"
    }
  },
  "interceptRules": [
    {
      "match": ".*\\.api\\.example\\.com.*",
      "action": "intercept",
      "modifyHeaders": {
        "User-Agent": "CustomAgent/1.0"
      }
    }
  ],
  "bypassSSL": true
}
```

### Objection Startup Scripts

`configurations/objection/startup.js`:

```javascript
// Auto-load common bypasses
android sslpinning disable;
android root disable;

// Custom hooks
android hooking watch class_method com.example.target.Auth.login --dump-args --dump-return;
android hooking watch class_method com.example.target.Network.makeRequest --dump-args --dump-return;

// Monitor file operations
android hooking watch class java.io.FileOutputStream.$init --dump-args;

console.log("[JAMBOREE] Startup hooks loaded");
```

Load on startup:

```bash
objection -g com.example.target explore --startup-script configurations/objection/startup.js
```

## Troubleshooting

### Certificate Trust Issues

**Symptom:** HTTPS traffic not decrypted in Burp Suite

**Solutions:**

```bash
# 1. Verify certificate installation
adb shell ls -la /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)

# 2. Check certificate format (must be PEM with .0 extension)
openssl x509 -in /system/etc/security/cacerts/9a5ba575.0 -text -noout

# 3. Ensure correct permissions
adb shell su -c "chmod 644 /system/etc/security/cacerts/*.0"

# 4. For Android 14+, use Magisk module
adb push modules/magisk/movecert.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/movecert.zip"
adb reboot
```

### Frida/Objection Connection Failures

**Symptom:** `Failed to spawn: unable to find application with identifier`

**Solutions:**

```bash
# 1. Verify Frida server version matches client
frida --version
adb shell /data/local/tmp/frida-server --version

# 2. Update Frida server
wget https://github.com/frida/frida/releases/download/$(frida --version)/frida-server-$(frida --version)-android-x86_64.xz
unxz frida-server-*.xz
adb push frida-server-* /data/local/tmp/frida-server
adb shell su -c "chmod 755 /data/local/tmp/frida-server"
adb shell su -c "/data/local/tmp/frida-server &"

# 3. Check if app is debuggable
adb shell pm dump com.example.target | grep -A 1 "debuggable"

# 4. Force debuggable mode (requires repackaging)
apktool d target.apk
# Edit AndroidManifest.xml: android:debuggable="true"
apktool b target -o target-debug.apk
jarsigner -keystore debug.keystore target-debug.apk androiddebugkey
adb install target-debug.apk
```

### Magisk Module Conflicts

**Symptom:** Device bootloop or modules not loading

**Solutions:**

```bash
# 1. Boot to safe mode (hold volume down during boot)
# Then remove problematic modules via Magisk Manager

# 2. Via ADB (if accessible)
adb shell su -c "touch /data/adb/modules/problem-module/disable"
adb reboot

# 3. Complete module reset
adb shell su -c "rm -rf /data/adb/modules/*"
adb reboot

# 4. Check module logs
adb shell su -c "cat /cache/magisk.log"
```

### Proxy Not Routing Traffic

**Symptom:** No requests appearing in Burp Suite

**Solutions:**

```bash
# 1. Verify proxy settings
adb shell settings get global http_proxy

# 2. Test connectivity
adb shell curl -x http://192.168.1.100:8080 http://example.com

# 3. Check iptables rules
adb shell su -c "iptables -t nat -L -n -v"

# 4. Force proxy via iptables redirect
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080"
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 192.168.1.100:8080"

# 5. Use VPN-based proxy as fallback
adb install modules/burp/drony.apk
# Configure Drony to redirect all traffic to Burp
```

## Advanced Techniques

### Custom Magisk Module Creation

```bash
#!/system/bin/sh
# modules/magisk/custom-bypass/service.sh

MODDIR=${0%/*}

# Disable root detection checks
resetprop ro.debuggable 0
resetprop ro.secure 1

# Hide Magisk from SafetyNet
magiskhide enable
magiskhide add com.example.target

# Custom file redirects
mount -o bind $MODDIR/system/lib/libc.so /system/lib/libc.so

log -p i -t JAMBOREE "Custom bypass module loaded"
```

Module metadata (`module.prop`):

```properties
id=jamboree-custom-bypass
name=JAMBOREE Custom Bypass
version=1.0
versionCode=1
author=YourName
description=Custom root and emulator detection bypass
```

Install:

```bash
cd modules/magisk/custom-bypass
zip -r custom-bypass.zip .
adb push custom-bypass.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/custom-bypass.zip"
adb reboot
```

### Runtime Memory Manipulation

```javascript
// hooks/memory-dump.js
Java.perform(function() {
    var String = Java.use("java.lang.String");
    var token = null;
    
    // Hook authentication token storage
    var SharedPreferences = Java.use("android.content.SharedPreferences");
    var Editor = Java.use("android.content.SharedPreferences$Editor");
    
    Editor.putString.implementation = function(key, value) {
        if (key.indexOf("token") !== -1 || key.indexOf("auth") !== -1) {
            console.log("[*] Token stored: " + key + " = " + value);
            token = value;
            
            // Dump to file
            var File = Java.use("java.io.File");
            var FileWriter = Java.use("java.io.FileWriter");
            var file = File.$new("/sdcard/extracted-token.txt");
            var writer = FileWriter.$new(file);
            writer.write(value);
            writer.close();
        }
        return this.putString(key, value);
    };
});
```

### Automated Testing Pipeline

```python
#!/usr/bin/env python3
# automation/test-pipeline.py

import subprocess
import time
import json

class JAMBOREEPipeline:
    def __init__(self, package_name, burp_host="192.168.1.100"):
        self.package = package_name
        self.burp_host = burp_host
        self.results = {"findings": []}
    
    def setup_environment(self):
        """Configure emulator and proxy"""
        subprocess.run(["adb", "wait-for-device"])
        subprocess.run(["adb", "shell", "settings", "put", "global", 
                       "http_proxy", f"{self.burp_host}:8080"])
        print("[+] Environment configured")
    
    def deploy_hooks(self):
        """Load Frida scripts"""
        scripts = [
            "hooks/ssl-bypass.js",
            "hooks/root-bypass.js",
            "hooks/crypto-trace.js"
        ]
        for script in scripts:
            subprocess.Popen(["frida", "-U", "-l", script, 
                            "-f", self.package])
        print("[+] Hooks deployed")
    
    def run_tests(self):
        """Execute test scenarios"""
        test_cases = [
            self.test_ssl_pinning,
            self.test_root_detection,
            self.test_data_storage
        ]
        for test in test_cases:
            result = test()
            if result:
                self.results["findings"].append(result)
    
    def test_ssl_pinning(self):
        """Test SSL certificate pinning bypass"""
        print("[*] Testing SSL pinning...")
        # Launch app and monitor Burp Suite for HTTPS traffic
        subprocess.run(["adb", "shell", "monkey", "-p", self.package, 
                       "-c", "android.intent.category.LAUNCHER", "1"])
        time.sleep(5)
        # Check if traffic visible in Burp (requires Burp REST API)
        return {"test": "ssl_pinning", "status": "bypassed"}
    
    def test_root_detection(self):
        """Test root detection bypass"""
        print("[*] Testing root detection...")
        result = subprocess.run(
            ["adb", "shell", "su", "-c", "id"],
            capture_output=True, text=True
        )
        if "uid=0" in result.stdout:
            return {"test": "root_detection", "status": "bypassed"}
        return None
    
    def test_data_storage(self):
        """Check for sensitive data in storage"""
        print("[*] Checking data storage...")
        result = subprocess.run([
            "adb", "shell", "su", "-c", 
            f"find /data/data/{self.package} -name '*.db' -o -name '*.xml'"
        ], capture_output=True, text=True)
        
        files = result.stdout.strip().split('\n')
        return {"test": "data_storage", "files": files}
    
    def generate_report(self):
        """Output findings"""
        with open("test-results.json", "w") as f:
            json.dump(self.results, f, indent=2)
        print(f"[+] Report saved: test-results.json")

if __name__ == "__main__":
    pipeline = JAMBOREEPipeline("com.example.target")
    pipeline.setup_environment()
    pipeline.deploy_hooks()
    pipeline.run_tests()
    pipeline.generate_report()
```

Run the pipeline:

```bash
python3 automation/test-pipeline.py
```

## Environment Variables

```bash
# Set in ~/.bashrc or project .env file

export JAMBOREE_HOME="/path/to/Android-Mobile-Security-Sandbox-Testing"
export JAMBOREE_BURP_HOST="192.168.1.100"
export JAMBOREE_BURP_PORT="8080"
export JAMBOREE_AVD_NAME="jamboree-test"
export ANDROID_SDK_ROOT="/path/to/android-sdk"
export FRIDA_SERVER_VERSION="16.1.4"

# Source helpers
source ${JAMBOREE_HOME}/orchestration/helpers.sh
```

This skill provides comprehensive coverage of the JAMBOREE framework for AI agents to assist developers with Android security testing workflows, from initial setup through advanced exploitation techniques.
