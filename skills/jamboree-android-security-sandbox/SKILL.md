---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for penetration testing
triggers:
  - set up android security testing environment
  - configure jamboree android sandbox
  - intercept android app traffic with burp
  - use objection to bypass root detection
  - configure magisk modules for testing
  - set up frida hooks for android app
  - bypass ssl pinning on android
  - create android emulator for pentesting
---

# JAMBOREE Android Security Sandbox

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is an integrated Android security testing framework that combines Magisk modules, Burp Suite proxy configuration, Objection/Frida runtime instrumentation, and optimized emulator setups into a unified testing environment. It streamlines the workflow for penetration testers and security researchers conducting Android application assessments.

## Installation

### Prerequisites

```bash
# Verify Java Development Kit
java -version  # Requires JDK 11+

# Verify Android SDK and tools
echo $ANDROID_HOME
adb --version
emulator -version

# Install Python and required packages
python3 -m pip install objection frida-tools
```

### Framework Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration script
chmod +x orchestration/deploy.sh
./orchestration/deploy.sh

# The script will:
# 1. Validate environment dependencies
# 2. Deploy Magisk modules
# 3. Configure Burp Suite integration
# 4. Set up Objection scripts
```

### Emulator Configuration

```bash
# Create a new AVD with Google APIs (for root support)
avdmanager create avd -n jamboree_test \
  -k "system-images;android-30;google_apis;x86_64" \
  -d "pixel_4"

# Start emulator with writable system partition
emulator -avd jamboree_test -writable-system -no-snapshot-load &

# Wait for boot
adb wait-for-device
```

## Core Components

### 1. Magisk Module Management

```bash
# Install Magisk to emulator/device
# Download Magisk APK and install
adb install modules/magisk/Magisk-v26.1.apk

# Push and install Magisk modules
adb push modules/magisk/systemless/busybox.zip /sdcard/
adb push modules/magisk/systemless/ssl-unpinning.zip /sdcard/

# Install via Magisk Manager or command line
adb shell su -c "magisk --install-module /sdcard/busybox.zip"
adb shell su -c "magisk --install-module /sdcard/ssl-unpinning.zip"

# Reboot to apply modules
adb reboot
adb wait-for-device
```

### 2. Burp Suite Proxy Configuration

```bash
# Export Burp CA certificate (from Burp Suite)
# Navigate to: Proxy > Options > Import/Export CA Certificate
# Export as DER format: burp_ca.der

# Convert to PEM format
openssl x509 -inform DER -in burp_ca.der -out burp_ca.pem

# Calculate certificate hash
HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp_ca.pem | head -1)

# Rename and push to system trust store
mv burp_ca.pem ${HASH}.0
adb root
adb remount
adb push ${HASH}.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/${HASH}.0
adb reboot
```

### 3. Network Proxy Setup

```bash
# Configure device to use Burp Suite proxy
# Get host machine IP (from device perspective)
HOST_IP="10.0.2.2"  # Default for Android emulator
BURP_PORT="8080"

# Set global proxy
adb shell settings put global http_proxy ${HOST_IP}:${BURP_PORT}

# Or use per-WiFi settings for physical devices
adb shell am start -a android.settings.WIFI_SETTINGS

# Clear proxy when done
adb shell settings put global http_proxy :0
```

### 4. Objection Runtime Instrumentation

```bash
# List running applications
frida-ps -Ua

# Inject Objection into running app
objection -g com.example.targetapp explore

# Or spawn app with Objection
objection -g com.example.targetapp explore --startup-command \
  "android sslpinning disable"
```

## Objection Commands and Scripts

### Basic Exploration

```bash
# Inside objection REPL

# Explore app structure
env

# List activities
android hooking list activities

# List services
android hooking list services

# List classes
android hooking list classes

# Search for specific class
android hooking search classes database

# Search for methods
android hooking search methods decrypt
```

### SSL Pinning Bypass

```javascript
// Save as modules/objection/scripts/ssl-bypass.js
Java.perform(function() {
    console.log("[*] Universal SSL Pinning Bypass");
    
    // TrustManagerImpl bypass
    var TrustManagerImpl = Java.use("com.android.org.conscrypt.TrustManagerImpl");
    TrustManagerImpl.verifyChain.implementation = function(untrustedChain, trustAnchorChain, host, clientAuth, ocspData, tlsSctData) {
        console.log("[+] Bypassing TrustManagerImpl.verifyChain for: " + host);
        return untrustedChain;
    };
    
    // OkHttp CertificatePinner bypass
    try {
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(hostname, peerCertificates) {
            console.log("[+] Bypassing OkHttp CertificatePinner for: " + hostname);
            return;
        };
    } catch(err) {
        console.log("[-] OkHttp not found");
    }
});
```

```bash
# Load custom script
objection -g com.example.targetapp explore \
  --startup-script modules/objection/scripts/ssl-bypass.js
```

### Root Detection Bypass

```javascript
// Save as modules/objection/scripts/root-bypass.js
Java.perform(function() {
    console.log("[*] Root Detection Bypass");
    
    // Common root check methods
    var RootBeer = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer.isRooted.implementation = function() {
        console.log("[+] Bypassing RootBeer.isRooted");
        return false;
    };
    
    // File existence checks
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes("su") || path.includes("magisk")) {
            console.log("[+] Hiding file: " + path);
            return false;
        }
        return this.exists.call(this);
    };
});
```

### Memory and Storage Exploration

```bash
# Inside objection REPL

# Dump SharedPreferences
android shared_prefs dump

# List SQLite databases
sqlite connect /data/data/com.example.app/databases/app.db

# Execute SQL query
sqlite execute schema

# Download database for offline analysis
file download /data/data/com.example.app/databases/app.db ./app.db

# Memory search for strings
memory search "api_key" --string
```

## Frida Scripting Patterns

### Method Hooking

```javascript
// Save as modules/objection/scripts/method-hook.js
Java.perform(function() {
    var TargetClass = Java.use("com.example.app.crypto.Encryption");
    
    // Hook encryption method
    TargetClass.encrypt.overload('java.lang.String').implementation = function(plaintext) {
        console.log("[*] encrypt() called");
        console.log("[+] Input: " + plaintext);
        
        var result = this.encrypt(plaintext);
        
        console.log("[+] Output: " + result);
        return result;
    };
    
    // Hook with parameter modification
    TargetClass.verify.implementation = function(token) {
        console.log("[*] verify() called with token: " + token);
        // Always return true
        return true;
    };
});
```

### Runtime Class Inspection

```javascript
// Save as modules/objection/scripts/class-inspect.js
Java.perform(function() {
    var targetClass = "com.example.app.MainActivity";
    
    Java.choose(targetClass, {
        onMatch: function(instance) {
            console.log("[+] Found instance: " + instance);
            
            // List all methods
            var methods = instance.class.getDeclaredMethods();
            methods.forEach(function(method) {
                console.log("[*] Method: " + method.getName());
            });
            
            // List all fields
            var fields = instance.class.getDeclaredFields();
            fields.forEach(function(field) {
                field.setAccessible(true);
                console.log("[*] Field: " + field.getName() + " = " + field.get(instance));
            });
        },
        onComplete: function() {
            console.log("[*] Search complete");
        }
    });
});
```

### API Response Modification

```javascript
// Save as modules/objection/scripts/api-modify.js
Java.perform(function() {
    var JSONObject = Java.use("org.json.JSONObject");
    
    JSONObject.$init.overload('java.lang.String').implementation = function(json) {
        console.log("[*] JSONObject created with: " + json);
        
        // Modify specific responses
        if (json.includes("isPremium")) {
            var modified = json.replace('"isPremium":false', '"isPremium":true');
            console.log("[+] Modified to: " + modified);
            return this.$init(modified);
        }
        
        return this.$init(json);
    };
});
```

## Configuration Files

### Burp Suite Extension Configuration

```python
# configurations/burp/auto-responder.py
# Example Burp Suite extension for JAMBOREE

from burp import IBurpExtender, IHttpListener

class BurpExtender(IBurpExtender, IHttpListener):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.setExtensionName("JAMBOREE Auto-Modifier")
        callbacks.registerHttpListener(self)
        
    def processHttpMessage(self, toolFlag, messageIsRequest, messageInfo):
        if not messageIsRequest:
            response = messageInfo.getResponse()
            responseInfo = self._helpers.analyzeResponse(response)
            body = response[responseInfo.getBodyOffset():].tostring()
            
            # Modify specific API responses
            if '"status":"unauthorized"' in body:
                modified = body.replace(
                    '"status":"unauthorized"',
                    '"status":"authorized"'
                )
                messageInfo.setResponse(
                    self._helpers.buildHttpMessage(
                        responseInfo.getHeaders(),
                        modified
                    )
                )
```

### Android Network Security Config

```xml
<!-- configurations/android/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
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

```bash
# Inject into target APK
apktool d target.apk -o target_decoded
cp configurations/android/network_security_config.xml \
   target_decoded/res/xml/

# Modify AndroidManifest.xml to reference it
# Add to <application> tag:
# android:networkSecurityConfig="@xml/network_security_config"

apktool b target_decoded -o target_patched.apk
jarsigner -keystore keystore.jks target_patched.apk alias_name
adb install target_patched.apk
```

## Automation Scripts

### Full Environment Deployment

```bash
#!/bin/bash
# orchestration/validators/environment-check.sh

set -e

echo "[*] JAMBOREE Environment Validation"

# Check Java
if ! command -v java &> /dev/null; then
    echo "[!] Java not found. Install JDK 11+"
    exit 1
fi
echo "[+] Java: $(java -version 2>&1 | head -n 1)"

# Check Android SDK
if [ -z "$ANDROID_HOME" ]; then
    echo "[!] ANDROID_HOME not set"
    exit 1
fi
echo "[+] Android SDK: $ANDROID_HOME"

# Check ADB
if ! command -v adb &> /dev/null; then
    echo "[!] ADB not found"
    exit 1
fi
echo "[+] ADB: $(adb --version | head -n 1)"

# Check Python tools
if ! command -v objection &> /dev/null; then
    echo "[!] Objection not installed"
    echo "    pip3 install objection"
    exit 1
fi
echo "[+] Objection: $(objection version)"

if ! command -v frida &> /dev/null; then
    echo "[!] Frida not installed"
    echo "    pip3 install frida-tools"
    exit 1
fi
echo "[+] Frida: $(frida --version)"

echo "[+] Environment validated successfully"
```

### Automated Testing Workflow

```bash
#!/bin/bash
# orchestration/deployers/test-app.sh

APP_PACKAGE="$1"
BURP_HOST="${BURP_HOST:-10.0.2.2}"
BURP_PORT="${BURP_PORT:-8080}"

if [ -z "$APP_PACKAGE" ]; then
    echo "Usage: $0 <app.package.name>"
    exit 1
fi

echo "[*] Starting JAMBOREE test workflow for $APP_PACKAGE"

# Configure proxy
echo "[*] Setting up proxy to $BURP_HOST:$BURP_PORT"
adb shell settings put global http_proxy $BURP_HOST:$BURP_PORT

# Launch with Objection
echo "[*] Launching with Objection"
objection -g $APP_PACKAGE explore --startup-command \
  "android sslpinning disable" \
  --startup-command "android root disable" &

OBJECTION_PID=$!

# Wait for user testing
echo "[*] Press Ctrl+C when testing is complete"
wait $OBJECTION_PID

# Clear proxy
echo "[*] Clearing proxy configuration"
adb shell settings put global http_proxy :0

echo "[+] Workflow complete"
```

## Troubleshooting

### Certificate Trust Issues

```bash
# Verify certificate installation
adb shell ls -la /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in burp_ca.pem | head -1)

# Check certificate format
openssl x509 -in burp_ca.pem -text -noout

# Force system remount (Android 10+)
adb root
adb disable-verity
adb reboot
adb wait-for-device
adb root
adb remount
```

### Frida Connection Problems

```bash
# Check Frida server version matches client
frida --version
adb shell "/data/local/tmp/frida-server --version"

# Download matching version
FRIDA_VERSION=$(frida --version)
wget https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-x86_64.xz
unxz frida-server-${FRIDA_VERSION}-android-x86_64.xz

# Push and start server
adb push frida-server-${FRIDA_VERSION}-android-x86_64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server &"

# Verify connection
frida-ps -U
```

### Magisk Module Not Loading

```bash
# Check module status
adb shell su -c "magisk --list"

# View Magisk logs
adb shell su -c "logcat -s Magisk:*"

# Remove and reinstall module
adb shell su -c "magisk --remove-module <module-id>"
adb shell su -c "magisk --install-module /sdcard/module.zip"
adb reboot
```

### App Crashes on Hook

```javascript
// Add error handling to Frida scripts
Java.perform(function() {
    try {
        var TargetClass = Java.use("com.example.TargetClass");
        TargetClass.targetMethod.implementation = function() {
            // Your hook logic
        };
    } catch(err) {
        console.log("[-] Error: " + err.message);
        console.log("[-] Stack: " + err.stack);
    }
});

// Use non-blocking hooks
Java.performNow = Java.perform;  // Immediate execution
```

## Best Practices

1. **Always test in isolated environments** - Use dedicated emulators or test devices
2. **Version match Frida components** - Ensure frida-server and frida-tools versions match
3. **Use environment variables for sensitive data** - Store proxy configs in `$BURP_HOST`, `$BURP_PORT`
4. **Implement logging** - Save Objection and Frida output for later analysis
5. **Clean up after testing** - Remove proxy settings, clear app data, disable root

## Environment Variables

```bash
# Set these in your shell profile
export ANDROID_HOME="/path/to/Android/Sdk"
export BURP_HOST="10.0.2.2"
export BURP_PORT="8080"
export JAMBOREE_ROOT="/path/to/Android-Mobile-Security-Sandbox-Testing"
export FRIDA_SERVER_PATH="/data/local/tmp/frida-server"
```
