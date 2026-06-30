---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for mobile app penetration testing
triggers:
  - set up android security testing environment
  - configure burp suite for android app testing
  - integrate objection with android emulator
  - install magisk modules for security testing
  - bypass ssl pinning on android
  - set up frida for android runtime hooking
  - create android penetration testing sandbox
  - configure android emulator for security research
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a comprehensive Android security testing framework that integrates:

- **Magisk** - Systemless root and module management
- **Burp Suite** - HTTP/HTTPS proxy and certificate management
- **Objection** - Runtime mobile exploration toolkit powered by Frida
- **Android Emulator** - Optimized AVD configurations with anti-detection

This unified environment eliminates fragmentation in Android security workflows by providing pre-configured components that work together seamlessly.

## Installation

### Prerequisites

Ensure these tools are installed and available in PATH:

```bash
# Verify Java (JDK 11+)
java -version

# Verify Android SDK
adb version
emulator -version

# Verify Python 3.8+
python3 --version

# Install Frida tools
pip3 install frida-tools objection
```

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration setup script
./orchestration/deploy.sh

# Verify installation
./orchestration/validators/check_environment.sh
```

### Configuration Files

Create a configuration directory structure:

```bash
mkdir -p ~/.jamboree/{android,network,security}
```

**Network Configuration** (`~/.jamboree/network/proxy.conf`):

```ini
[proxy]
host=127.0.0.1
port=8080
upstream_proxy=
ssl_passthrough=false

[certificate]
ca_path=/path/to/burp-ca-cert.der
system_trust=true
```

**Android Configuration** (`~/.jamboree/android/emulator.conf`):

```ini
[device]
api_level=30
arch=x86_64
ram=4096
storage=8192

[security]
root_method=magisk
hide_root=true
hide_emulator=true
```

## Core Components

### 1. Magisk Module Management

Deploy Magisk modules for systemless modifications:

```bash
# List available modules
ls modules/magisk/

# Install a specific module
adb push modules/magisk/systemless/busybox.zip /sdcard/
adb shell su -c "magisk --install-module /sdcard/busybox.zip"

# Verify module installation
adb shell su -c "magisk --list"
```

**Custom Module Creation**:

```bash
# Create module structure
mkdir -p custom_module/{META-INF/com/google/android,system}

# Create module.prop
cat > custom_module/module.prop <<EOF
id=jamboree_custom
name=JAMBOREE Custom Module
version=1.0
versionCode=1
author=Your Name
description=Custom security testing module
EOF

# Package module
cd custom_module && zip -r ../custom_module.zip * && cd ..
```

### 2. Burp Suite Integration

Configure Burp Suite proxy and certificate trust:

```bash
# Export Burp CA certificate (from Burp Suite UI)
# Proxy > Options > Import/Export CA Certificate > Export Certificate in DER format

# Convert DER to PEM
openssl x509 -inform DER -in burp-ca-cert.der -out burp-ca-cert.pem

# Get certificate hash
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca-cert.pem | head -1)

# Install as system certificate
adb root
adb remount
adb push burp-ca-cert.pem /sdcard/
adb shell "su -c 'mv /sdcard/burp-ca-cert.pem /system/etc/security/cacerts/${CERT_HASH}.0'"
adb shell "su -c 'chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0'"
adb reboot
```

**Configure Android Proxy**:

```bash
# Set global proxy
adb shell settings put global http_proxy 127.0.0.1:8080

# Or use per-WiFi proxy in emulator settings
adb shell am start -a android.settings.WIFI_SETTINGS
```

### 3. Objection Runtime Exploration

Launch Objection against a target application:

```bash
# List running applications
frida-ps -U

# Attach to running app
objection -g com.example.app explore

# Spawn app with Objection
objection -g com.example.app -S explore
```

**Common Objection Commands**:

```bash
# Inside Objection REPL

# Bypass SSL pinning
android sslpinning disable

# List activities
android hooking list activities

# List classes and methods
android hooking list classes
android hooking search methods MainActivity

# Hook a specific method
android hooking watch class_method com.example.app.MainActivity.onCreate --dump-args --dump-return

# Explore SQLite databases
android sqlite status
android sqlite connect /data/data/com.example.app/databases/user.db
android sqlite execute "SELECT * FROM users;"

# Dump SharedPreferences
android pref dump
android pref set com.example.app premium_user true

# File operations
file download /data/data/com.example.app/files/config.json ./config.json
file upload ./modified_config.json /data/data/com.example.app/files/config.json
```

### 4. Custom Frida Scripts

Create custom Frida scripts for advanced hooking:

**ssl-pinning-bypass.js**:

```javascript
// SSL Pinning Bypass Script
Java.perform(function() {
    console.log("[*] Starting SSL Pinning Bypass");
    
    // Bypass OkHttp3 CertificatePinner
    var CertificatePinner = Java.use('okhttp3.CertificatePinner');
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(str, list) {
        console.log("[+] Bypassing OkHttp3 CertificatePinner.check for: " + str);
        return;
    };
    
    // Bypass TrustManager
    var X509TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    var TrustManager = Java.registerClass({
        name: 'com.jamboree.TrustManager',
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    // Hook SSLContext
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    SSLContext.init.overload('[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom').implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[+] Overriding SSLContext.init with custom TrustManager");
        this.init.overload('[Ljavax.net.ssl.KeyManager;', '[Ljavax.net.ssl.TrustManager;', 'java.security.SecureRandom').call(this, keyManager, [TrustManager.$new()], secureRandom);
    };
    
    console.log("[*] SSL Pinning Bypass Complete");
});
```

**Run custom script**:

```bash
frida -U -f com.example.app -l ssl-pinning-bypass.js --no-pause
```

**method-tracer.js**:

```javascript
// Trace specific class methods
Java.perform(function() {
    var targetClass = Java.use('com.example.app.AuthManager');
    
    targetClass.login.implementation = function(username, password) {
        console.log("[+] login() called");
        console.log("    Username: " + username);
        console.log("    Password: " + password);
        
        var result = this.login(username, password);
        
        console.log("    Return value: " + result);
        return result;
    };
    
    targetClass.generateToken.implementation = function() {
        console.log("[+] generateToken() called");
        var token = this.generateToken();
        console.log("    Token: " + token);
        return token;
    };
});
```

## Complete Workflow Examples

### Example 1: Basic App Security Assessment

```bash
# 1. Start emulator with proxy
emulator -avd JAMBOREE_Security_AVD -http-proxy 127.0.0.1:8080 &

# 2. Start Burp Suite (ensure listener on 127.0.0.1:8080)

# 3. Verify device connection
adb devices

# 4. Install target app
adb install target_app.apk

# 5. Launch Objection
objection -g com.target.app explore

# 6. In Objection REPL, disable SSL pinning
android sslpinning disable

# 7. Start app and monitor traffic in Burp
# 8. Explore runtime with Objection commands
```

### Example 2: Bypass Root Detection

```bash
# Launch app with root detection bypass script
cat > root-bypass.js <<'EOF'
Java.perform(function() {
    // Common root detection methods
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    RootBeer.isRooted.implementation = function() {
        console.log("[+] Root detection bypassed");
        return false;
    };
    
    // Bypass Magisk detection
    var Runtime = Java.use('java.lang.Runtime');
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.indexOf("su") !== -1 || cmd.indexOf("magisk") !== -1) {
            console.log("[+] Blocked command: " + cmd);
            throw new Error("Command not found");
        }
        return this.exec(cmd);
    };
    
    // File existence checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.indexOf("su") !== -1 || path.indexOf("magisk") !== -1 || path.indexOf("Superuser.apk") !== -1) {
            console.log("[+] Hiding file: " + path);
            return false;
        }
        return this.exists();
    };
});
EOF

frida -U -f com.target.app -l root-bypass.js --no-pause
```

### Example 3: Extract Encrypted Data

```bash
# Launch Objection
objection -g com.target.app explore

# Inside Objection REPL:
# 1. Find crypto methods
android hooking search methods AES
android hooking search methods decrypt

# 2. Hook decryption method
android hooking watch class_method com.target.app.CryptoUtil.decrypt --dump-args --dump-return

# 3. Trigger decryption in app
# 4. Extract decrypted data from logs

# Alternative: Dump memory
memory dump all /sdcard/memory_dump.bin
exit

# Pull memory dump
adb pull /sdcard/memory_dump.bin ./
strings memory_dump.bin | grep -i "password\|token\|api"
```

## Advanced Configuration

### Custom Emulator Creation

```bash
# Create AVD with specific configuration
avdmanager create avd \
  -n JAMBOREE_Security_AVD \
  -k "system-images;android-30;google_apis_playstore;x86_64" \
  -d "pixel_4" \
  -p ~/.android/avd/JAMBOREE_Security_AVD

# Edit config.ini for security testing
cat >> ~/.android/avd/JAMBOREE_Security_AVD.avd/config.ini <<EOF
hw.sensors.proximity=yes
hw.sensors.magnetic_field=yes
hw.gps=yes
hw.ramSize=4096
vm.heapSize=512
hw.keyboard=yes
EOF

# Launch with additional options
emulator @JAMBOREE_Security_AVD \
  -http-proxy 127.0.0.1:8080 \
  -writable-system \
  -no-snapshot-load \
  -gpu swiftshader_indirect
```

### Automated Setup Script

```bash
#!/bin/bash
# jamboree-setup.sh - Automated JAMBOREE environment setup

set -e

BURP_PROXY="127.0.0.1:8080"
AVD_NAME="JAMBOREE_Security_AVD"
TARGET_PACKAGE="$1"

if [ -z "$TARGET_PACKAGE" ]; then
    echo "Usage: $0 <target.package.name>"
    exit 1
fi

echo "[*] Starting emulator..."
emulator @$AVD_NAME -http-proxy $BURP_PROXY -no-window &
EMULATOR_PID=$!

echo "[*] Waiting for device..."
adb wait-for-device

echo "[*] Installing Burp certificate..."
./orchestration/deployers/install_burp_cert.sh

echo "[*] Configuring proxy..."
adb shell settings put global http_proxy $BURP_PROXY

echo "[*] Launching Objection..."
objection -g $TARGET_PACKAGE explore

# Cleanup on exit
trap "kill $EMULATOR_PID" EXIT
```

## Troubleshooting

### Issue: SSL Pinning Bypass Not Working

**Solution 1 - Use Frida Gadget**:

```bash
# Repackage app with Frida gadget
objection patchapk -s target_app.apk

# Install patched app
adb install target_app.objection.apk

# Connect to gadget
objection explore
```

**Solution 2 - Check Android Version**:

```bash
# Android 7+ requires system certificate
# Verify certificate is in system store
adb shell "ls -la /system/etc/security/cacerts/ | grep burp"

# If missing, remount system as writable
adb root
adb disable-verity
adb reboot
adb wait-for-device
adb root
adb remount
# Re-install certificate
```

### Issue: Objection Cannot Connect

```bash
# Check Frida server is running
adb shell "ps -A | grep frida"

# If not running, start Frida server
adb push frida-server-16.0.19-android-x86_64 /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"

# Verify Frida connection
frida-ps -U
```

### Issue: Magisk Module Not Loading

```bash
# Check module status
adb shell "su -c 'magisk --list'"

# Review Magisk logs
adb shell "su -c 'logcat -b main -b system | grep -i magisk'"

# Boot to safe mode (removes all modules temporarily)
adb reboot
# During boot, hold Volume Down

# Re-install module after fixing conflicts
```

### Issue: Emulator Detection by App

```bash
# Use emulator property modification
adb root
adb shell "setprop ro.build.fingerprint 'google/redfin/redfin:11/RQ3A.210805.001.A1/7474174:user/release-keys'"
adb shell "setprop ro.product.manufacturer 'Google'"
adb shell "setprop ro.product.model 'Pixel 5'"
adb shell "setprop ro.build.product 'redfin'"

# Reboot to apply
adb reboot
```

## Integration with CI/CD

```yaml
# .github/workflows/security-scan.yml
name: Android Security Scan

on: [push, pull_request]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup JAMBOREE
        run: |
          ./orchestration/deploy.sh
          
      - name: Start Emulator
        run: |
          emulator @JAMBOREE_Security_AVD -no-window &
          adb wait-for-device
          
      - name: Run Security Tests
        run: |
          objection -g com.example.app -S explore <<EOF
          android sslpinning disable
          android hooking list activities
          exit
          EOF
          
      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: security-report
          path: ./reports/
```

## Best Practices

1. **Always use isolated environments** - Never test on production devices or networks
2. **Maintain separate AVDs** - Create dedicated emulators per project to avoid conflicts
3. **Version control your scripts** - Keep Frida scripts and configurations in version control
4. **Document bypass techniques** - Record successful bypass methods for future reference
5. **Regular updates** - Keep Frida, Objection, and Magisk updated for latest features
6. **Secure your workstation** - Testing tools have elevated privileges; protect your environment

## Environment Variables

```bash
# Export these in your shell profile
export JAMBOREE_HOME="$HOME/.jamboree"
export BURP_PROXY_HOST="127.0.0.1"
export BURP_PROXY_PORT="8080"
export ANDROID_SDK_ROOT="$HOME/Android/Sdk"
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk"
```
