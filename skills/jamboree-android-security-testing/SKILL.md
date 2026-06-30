---
name: jamboree-android-security-testing
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and rooted emulators for mobile penetration testing
triggers:
  - set up an Android security testing sandbox with JAMBOREE
  - configure Burp Suite proxy for Android app interception
  - deploy Magisk modules for Android security testing
  - use Objection to hook into Android applications
  - bypass SSL certificate pinning on Android
  - create a rooted Android emulator for pentesting
  - intercept Android app traffic with Burp Suite
  - set up JAMBOREE framework for mobile security analysis
---

# JAMBOREE Android Security Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified security testing framework that integrates Magisk module management, Burp Suite proxy configuration, Objection runtime exploration, and rooted Android emulation into a single orchestrated environment for mobile application penetration testing.

## What JAMBOREE Does

JAMBOREE provides:
- **Automated Magisk deployment** for systemless root with module conflict resolution
- **Burp Suite integration** with automatic CA certificate installation and traffic interception
- **Objection/Frida toolkit** for runtime hooking, method tracing, and SSL pinning bypass
- **Optimized emulator configurations** that evade anti-emulation and root detection
- **Unified orchestration** that reduces setup time and eliminates manual configuration errors

## Prerequisites

Before using JAMBOREE, ensure you have:

```bash
# Java Development Kit (11+)
java -version

# Android SDK with platform tools
echo $ANDROID_HOME
adb --version

# Python 3.8+ for Objection/Frida
python3 --version
pip3 --version

# Burp Suite (Community or Professional)
# Download from: https://portswigger.net/burp/releases
```

## Installation

### 1. Clone and Initialize

```bash
# Clone the JAMBOREE repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Make orchestration scripts executable
chmod +x orchestration/deploy.sh
chmod +x orchestration/validate.sh
```

### 2. Install Python Dependencies

```bash
# Install Objection and Frida tools
pip3 install objection frida-tools

# Verify installation
objection --version
frida --version
```

### 3. Configure Environment Variables

```bash
# Set environment variables
export JAMBOREE_HOME="$(pwd)"
export BURP_JAR_PATH="/path/to/burpsuite_community.jar"
export ANDROID_AVD_NAME="JAMBOREE_Security_Testing"
export PROXY_HOST="127.0.0.1"
export PROXY_PORT="8080"

# Add to your shell profile for persistence
echo "export JAMBOREE_HOME=$JAMBOREE_HOME" >> ~/.bashrc
echo "export BURP_JAR_PATH=$BURP_JAR_PATH" >> ~/.bashrc
```

## Environment Deployment

### Phase 1: Validation

```bash
# Run pre-deployment validation
./orchestration/validate.sh

# Expected output:
# ✓ Java JDK detected (version 11.0.18)
# ✓ Android SDK found at /opt/android-sdk
# ✓ ADB accessible in PATH
# ✓ Python 3.9.16 with pip available
# ✓ Objection 1.11.0 installed
# ✓ Frida 16.0.19 installed
```

### Phase 2: Core Deployment

```bash
# Deploy the complete JAMBOREE stack
./orchestration/deploy.sh --full

# Options:
# --magisk-only      Deploy only Magisk modules
# --burp-only        Configure only Burp Suite integration
# --objection-only   Set up only Objection/Frida
# --avd-create       Create a new Android Virtual Device
# --verbose          Enable detailed logging
```

### Phase 3: Launch Environment

```bash
# Start the Android emulator with Magisk
emulator -avd $ANDROID_AVD_NAME -writable-system -no-snapshot-load &

# Wait for boot
adb wait-for-device
adb shell getprop sys.boot_completed

# Start Burp Suite (in separate terminal)
java -jar $BURP_JAR_PATH --project-file=./configurations/burp/jamboree-project.burp
```

## Magisk Module Management

### Deploy Systemless Root

```bash
# Install Magisk APK to emulator
adb install modules/magisk/Magisk-v26.1.apk

# Push custom modules
adb push modules/magisk/systemless/ssl-unpinning.zip /sdcard/Download/
adb push modules/magisk/systemless/busybox-ndk.zip /sdcard/Download/

# Install via Magisk Manager CLI
adb shell su -c "magisk --install-module /sdcard/Download/ssl-unpinning.zip"
adb shell su -c "magisk --install-module /sdcard/Download/busybox-ndk.zip"

# Reboot to apply modules
adb reboot
adb wait-for-device
```

### Verify Magisk Installation

```bash
# Check Magisk version and status
adb shell su -c "magisk -v"
adb shell su -c "magisk --denylist status"

# List installed modules
adb shell su -c "ls -la /data/adb/modules/"

# Check module logs
adb shell su -c "cat /data/adb/magisk.log"
```

## Burp Suite Proxy Configuration

### Install CA Certificate as System Trust

```bash
# Export Burp CA certificate
# In Burp Suite: Proxy → Options → Import/Export CA certificate → Certificate in DER format
# Save as burp-ca.der

# Convert to PEM format with correct hash name
openssl x509 -inform DER -in burp-ca.der -out burp-ca.pem
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)
cp burp-ca.pem ${CERT_HASH}.0

# Push to system certificate store
adb root
adb remount
adb push ${CERT_HASH}.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### Configure Proxy Settings

```bash
# Set global proxy (Wi-Fi)
adb shell settings put global http_proxy ${PROXY_HOST}:${PROXY_PORT}

# Or use iptables redirect (requires root)
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination ${PROXY_HOST}:${PROXY_PORT}"
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination ${PROXY_HOST}:${PROXY_PORT}"

# Verify proxy is active
adb shell settings get global http_proxy
```

### Automated Certificate Installation Script

```bash
#!/bin/bash
# File: scripts/install-burp-cert.sh

set -e

BURP_CERT_DER="${1:-./configurations/burp/certificates/burp-ca.der}"

if [ ! -f "$BURP_CERT_DER" ]; then
    echo "Error: Burp certificate not found at $BURP_CERT_DER"
    exit 1
fi

# Convert and hash certificate
openssl x509 -inform DER -in "$BURP_CERT_DER" -out /tmp/burp-ca.pem
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in /tmp/burp-ca.pem | head -1)
cp /tmp/burp-ca.pem "/tmp/${CERT_HASH}.0"

# Install to system
adb root
adb wait-for-device
adb remount
adb push "/tmp/${CERT_HASH}.0" /system/etc/security/cacerts/
adb shell chmod 644 "/system/etc/security/cacerts/${CERT_HASH}.0"
adb shell chown root:root "/system/etc/security/cacerts/${CERT_HASH}.0"

echo "✓ Burp CA certificate installed as ${CERT_HASH}.0"
echo "Rebooting device..."
adb reboot
```

## Objection Runtime Hooking

### Basic Objection Commands

```bash
# List installed packages
adb shell pm list packages | grep -i target

# Spawn application with Objection
objection --gadget "com.example.targetapp" explore

# Or attach to running process
objection --gadget "com.example.targetapp" explore --startup-command "android hooking watch class_method com.example.TargetClass.sensitiveMethod --dump-args --dump-return"
```

### SSL Pinning Bypass

```objection
# Inside Objection REPL
android sslpinning disable

# Verify pinning is disabled
android hooking list activities
android intent launch_activity com.example.targetapp/.MainActivity

# Watch for SSL validation calls
android hooking watch class javax.net.ssl.SSLContext
```

### Common Objection Scripts

```javascript
// File: modules/objection/scripts/dump-shared-prefs.js
// Usage: objection --gadget com.example.app explore --startup-script dump-shared-prefs.js

Java.perform(function() {
    var Context = Java.use("android.content.Context");
    var SharedPreferences = Java.use("android.content.SharedPreferences");
    
    console.log("[*] Enumerating SharedPreferences files...");
    
    var currentApp = Java.use("android.app.ActivityThread")
        .currentApplication()
        .getApplicationContext();
    
    var prefsDir = currentApp.getFilesDir().getParent() + "/shared_prefs";
    var File = Java.use("java.io.File");
    var prefsFolder = File.$new(prefsDir);
    
    var files = prefsFolder.listFiles();
    if (files) {
        for (var i = 0; i < files.length; i++) {
            var fileName = files[i].getName().replace(".xml", "");
            console.log("[+] Found: " + fileName);
            
            var prefs = currentApp.getSharedPreferences(fileName, 0);
            var allEntries = prefs.getAll();
            var entries = allEntries.keySet().toArray();
            
            for (var j = 0; j < entries.length; j++) {
                var key = entries[j];
                var value = allEntries.get(key);
                console.log("    " + key + " = " + value);
            }
        }
    }
});
```

### Hook Cryptographic Operations

```javascript
// File: modules/objection/scripts/crypto-hook.js

Java.perform(function() {
    console.log("[*] Hooking cryptographic operations...");
    
    // Hook AES cipher operations
    var Cipher = Java.use("javax.crypto.Cipher");
    
    Cipher.doFinal.overload('[B').implementation = function(data) {
        console.log("[AES] doFinal called");
        console.log("    Algorithm: " + this.getAlgorithm());
        console.log("    Input length: " + data.length);
        console.log("    Input (hex): " + bytesToHex(data));
        
        var result = this.doFinal(data);
        console.log("    Output (hex): " + bytesToHex(result));
        
        return result;
    };
    
    // Hook MessageDigest (hashing)
    var MessageDigest = Java.use("java.security.MessageDigest");
    
    MessageDigest.digest.overload('[B').implementation = function(data) {
        var algorithm = this.getAlgorithm();
        console.log("[Hash] " + algorithm + " digest called");
        console.log("    Input (hex): " + bytesToHex(data));
        
        var result = this.digest(data);
        console.log("    Hash (hex): " + bytesToHex(result));
        
        return result;
    };
    
    function bytesToHex(bytes) {
        var hex = [];
        for (var i = 0; i < bytes.length; i++) {
            hex.push(('0' + (bytes[i] & 0xFF).toString(16)).slice(-2));
        }
        return hex.join('');
    }
});
```

### Spawn and Hook Pattern

```bash
# Kill app if running
adb shell am force-stop com.example.targetapp

# Spawn with Objection and load script
objection --gadget "com.example.targetapp" explore \
    --startup-command "import modules/objection/scripts/crypto-hook.js" \
    --startup-command "android sslpinning disable"

# Alternatively, use a startup script file
cat > /tmp/startup.objection <<EOF
import modules/objection/scripts/crypto-hook.js
android sslpinning disable
android hooking watch class_method com.example.api.ApiClient.makeRequest --dump-args --dump-return
EOF

objection --gadget "com.example.targetapp" explore --startup-script /tmp/startup.objection
```

## Emulator Configuration

### Create Optimized AVD

```bash
# Create AVD with Google APIs (for Play Store apps)
avdmanager create avd \
    --name JAMBOREE_Security_Testing \
    --package "system-images;android-29;google_apis;x86_64" \
    --device "pixel_4" \
    --sdcard 2048M

# Configure AVD with optimal settings
cat > ~/.android/avd/JAMBOREE_Security_Testing.avd/config.ini <<EOF
hw.ramSize=4096
hw.keyboard=yes
hw.gpu.enabled=yes
hw.gpu.mode=host
fastboot.forceFastBoot=no
disk.dataPartition.size=8G
vm.heapSize=512
EOF

# Launch with writeable system partition
emulator -avd JAMBOREE_Security_Testing \
    -writable-system \
    -no-snapshot-load \
    -http-proxy ${PROXY_HOST}:${PROXY_PORT} &
```

### Anti-Detection Configuration

```bash
# Modify build properties to appear as physical device
adb root
adb remount

# Create custom build.prop overlay
cat > /tmp/build.prop.overlay <<EOF
ro.build.fingerprint=google/redfin/redfin:11/RQ3A.211001.001/7641976:user/release-keys
ro.build.characteristics=nosdcard
ro.product.model=Pixel 5
ro.product.manufacturer=Google
ro.hardware=redfin
ro.build.tags=release-keys
dalvik.vm.heapsize=512m
EOF

# Apply overlay
adb push /tmp/build.prop.overlay /system/build.prop
adb shell chmod 644 /system/build.prop

# Hide emulator artifacts
adb shell su -c "setprop ro.kernel.qemu 0"
adb shell su -c "setprop ro.hardware.goldfish 0"

adb reboot
```

## Complete Testing Workflow

### Typical Application Assessment

```bash
#!/bin/bash
# File: workflows/full-app-assessment.sh

set -e

TARGET_PACKAGE="com.example.targetapp"
TARGET_APK="./target-app.apk"

echo "[1/6] Starting JAMBOREE environment..."
emulator -avd JAMBOREE_Security_Testing -writable-system -no-snapshot-load &
adb wait-for-device
sleep 10

echo "[2/6] Installing target application..."
adb install -r "$TARGET_APK"

echo "[3/6] Starting Burp Suite proxy..."
java -jar "$BURP_JAR_PATH" --project-file=./configurations/burp/jamboree-project.burp &
sleep 15

echo "[4/6] Configuring proxy settings..."
adb shell settings put global http_proxy ${PROXY_HOST}:${PROXY_PORT}

echo "[5/6] Launching Objection with SSL pinning bypass..."
objection --gadget "$TARGET_PACKAGE" explore \
    --startup-command "android sslpinning disable" \
    --startup-command "android hooking watch class_method ${TARGET_PACKAGE}.api.* --dump-args --dump-return" &

echo "[6/6] Starting application..."
adb shell monkey -p "$TARGET_PACKAGE" -c android.intent.category.LAUNCHER 1

echo ""
echo "✓ Environment ready for testing"
echo "  - Burp Suite listening on ${PROXY_HOST}:${PROXY_PORT}"
echo "  - SSL pinning disabled"
echo "  - API calls being logged"
echo ""
echo "Interact with the application to capture traffic."
```

## Troubleshooting

### Certificate Not Trusted

**Problem**: Apps still reject Burp certificate despite installation.

**Solution**: Verify certificate hash and Android 7+ network security config:

```bash
# Check installed certificates
adb shell "ls -la /system/etc/security/cacerts/ | grep -i burp"

# For Android 7+, create network security config bypass
cat > /tmp/network_security_config.xml <<EOF
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </base-config>
</network-security-config>
EOF

# Modify APK to include config (requires apktool)
apktool d target-app.apk -o /tmp/decoded
cp /tmp/network_security_config.xml /tmp/decoded/res/xml/
# Add to AndroidManifest.xml: android:networkSecurityConfig="@xml/network_security_config"
apktool b /tmp/decoded -o /tmp/patched.apk
# Sign and install patched APK
```

### Objection Cannot Attach

**Problem**: `Failed to spawn: unable to find gadget`

**Solution**: Manually inject Frida gadget:

```bash
# Download Frida gadget for target architecture
wget https://github.com/frida/frida/releases/download/16.0.19/frida-gadget-16.0.19-android-x86_64.so.xz
unxz frida-gadget-16.0.19-android-x86_64.so.xz

# Inject into APK
objection patchapk -s target-app.apk

# Install patched APK
adb install dist/target-app.objection.apk

# Now connect without --gadget flag
objection explore
```

### Magisk Modules Not Loading

**Problem**: Modules appear installed but don't function.

**Solution**: Check module compatibility and logs:

```bash
# View Magisk logs
adb shell su -c "cat /data/adb/magisk.log | tail -100"

# Check module installation status
adb shell su -c "magisk --list"

# Remove problematic module
adb shell su -c "magisk --remove-modules module-id"

# Reinstall with verbose logging
adb shell su -c "MAGISK_VERBOSE=1 magisk --install-module /sdcard/Download/module.zip"
```

### Proxy Connection Failures

**Problem**: Traffic not appearing in Burp Suite.

**Solution**: Verify iptables rules and proxy settings:

```bash
# Check global proxy setting
adb shell settings get global http_proxy

# Test direct connection to Burp
adb shell curl -x ${PROXY_HOST}:${PROXY_PORT} -v http://example.com

# If using iptables, verify NAT rules
adb shell su -c "iptables -t nat -L -n -v"

# Flush and reapply if needed
adb shell su -c "iptables -t nat -F"
./scripts/configure-iptables-proxy.sh
```

## Configuration Files

### Burp Suite Project Configuration

```json
{
  "proxy": {
    "request_listeners": [
      {
        "certificate_mode": "per_host",
        "listen_mode": "all_interfaces",
        "listener_port": 8080,
        "running": true,
        "support_invisible_proxying": true
      }
    ],
    "intercept_client_requests": {
      "do_intercept": false,
      "automatic_initial_interception": true
    },
    "ssl_pass_through": {
      "automatically_add_entries": true,
      "rules": []
    }
  },
  "target": {
    "scope": {
      "advanced_mode": true,
      "include": [
        {"enabled": true, "protocol": "any", "host": "^.*\\.example\\.com$"}
      ]
    }
  }
}
```

### Environment Variables Reference

```bash
# JAMBOREE Core
export JAMBOREE_HOME="/path/to/JAMBOREE"
export JAMBOREE_LOG_LEVEL="INFO"  # DEBUG, INFO, WARN, ERROR

# Android SDK
export ANDROID_HOME="/opt/android-sdk"
export ANDROID_AVD_NAME="JAMBOREE_Security_Testing"

# Burp Suite
export BURP_JAR_PATH="/path/to/burpsuite.jar"
export BURP_PROJECT_FILE="$JAMBOREE_HOME/configurations/burp/jamboree-project.burp"

# Proxy Settings
export PROXY_HOST="127.0.0.1"
export PROXY_PORT="8080"

# Frida/Objection
export FRIDA_SERVER_VERSION="16.0.19"
export OBJECTION_SCRIPTS_PATH="$JAMBOREE_HOME/modules/objection/scripts"

# Magisk
export MAGISK_VERSION="26.1"
export MAGISK_MODULES_PATH="$JAMBOREE_HOME/modules/magisk/systemless"
```

## Best Practices

1. **Always validate environment** before testing: Run `./orchestration/validate.sh`
2. **Use separate AVDs** for different target Android versions
3. **Keep Burp projects** organized per application being tested
4. **Version control your Objection scripts** for reusability
5. **Document custom Magisk modules** with README files
6. **Test on physical devices** when emulator anti-detection fails
7. **Regularly update** Frida, Objection, and Magisk to latest versions

## Additional Resources

- [Frida JavaScript API](https://frida.re/docs/javascript-api/)
- [Objection Command Reference](https://github.com/sensepost/objection/wiki/Using-objection)
- [Magisk Module Development](https://topjohnwu.github.io/Magisk/guides.html)
- [Burp Suite Extensions API](https://portswigger.net/burp/extender)
