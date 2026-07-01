---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for mobile application penetration testing
triggers:
  - set up android security testing environment
  - configure jamboree android sandbox
  - intercept android app traffic with burp
  - bypass ssl pinning on android
  - use objection to hook android app
  - set up magisk for security testing
  - configure frida for android pentest
  - create rooted android emulator for testing
---

# JAMBOREE Android Security Sandbox

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk**: Systemless root and module management
- **Burp Suite**: HTTP/HTTPS traffic interception
- **Objection/Frida**: Runtime instrumentation and hooking
- **Rooted Emulator**: Pre-configured Android Virtual Device

This creates a cohesive environment for penetration testing, reverse engineering, and malware analysis of Android applications.

## Installation

### Prerequisites

Ensure these are installed and in your PATH:

```bash
# Check Java (JDK 11+)
java -version

# Check Android SDK
adb version
emulator -version

# Check Python 3.8+
python3 --version

# Install Frida tools
pip3 install frida-tools objection
```

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Set environment variables
export ANDROID_SDK_ROOT="$HOME/Android/Sdk"
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"
export BURP_JAR="$HOME/tools/burpsuite_pro.jar"

# Run initialization script
./orchestration/deploy.sh --validate
```

### Emulator Deployment

```bash
# Create AVD with Magisk pre-installed
./orchestration/create_avd.sh \
  --name "jamboree-test" \
  --android-version 13 \
  --arch x86_64 \
  --root

# Start emulator
emulator -avd jamboree-test -writable-system -no-snapshot &

# Wait for boot
adb wait-for-device
```

## Core Components

### 1. Magisk Module Management

Magisk provides systemless root and module installation.

```bash
# Push Magisk modules
adb push modules/magisk/ssl-unpinning.zip /sdcard/Download/
adb push modules/magisk/frida-server.zip /sdcard/Download/

# Install via Magisk Manager (automated)
./orchestration/install_modules.sh \
  --module ssl-unpinning \
  --module frida-server \
  --reboot

# Verify module status
adb shell su -c "magisk --list"
```

### 2. Burp Suite Proxy Configuration

Configure device to route traffic through Burp:

```bash
# Export Burp CA certificate
BURP_CERT="$HOME/.burp/cacert.der"

# Convert and install as system certificate
openssl x509 -inform DER -in "$BURP_CERT" -out burp.pem
HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp.pem | head -1)
adb root
adb remount
adb push burp.pem "/system/etc/security/cacerts/${HASH}.0"
adb shell chmod 644 "/system/etc/security/cacerts/${HASH}.0"
adb reboot

# Configure proxy
adb shell settings put global http_proxy "192.168.1.100:8080"

# Or use automated script
./orchestration/configure_burp.sh \
  --burp-host 192.168.1.100 \
  --burp-port 8080 \
  --cert "$BURP_CERT"
```

### 3. Objection Runtime Hooking

Use Objection for dynamic instrumentation:

```bash
# List running apps
frida-ps -Uai

# Launch app with Objection
objection --gadget "com.example.targetapp" explore

# Common Objection commands (inside objection shell)
# Bypass SSL pinning
android sslpinning disable

# Dump activity hierarchy
android hooking list activities

# Watch method calls
android hooking watch class_method com.example.Class.method --dump-args --dump-return

# List shared preferences
android keystore list

# Dump SQLite databases
android sqlite databases
android sqlite connect /data/data/com.example.app/databases/app.db
.tables
SELECT * FROM users;
```

### Custom Objection Scripts

Create reusable Objection scripts:

```javascript
// modules/objection/scripts/bypass-root-detection.js
Java.perform(function() {
    console.log("[*] Bypassing root detection");
    
    // Hook common root detection methods
    var RootBeer = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer.isRooted.implementation = function() {
        console.log("[*] isRooted() called - returning false");
        return false;
    };
    
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.indexOf("su") !== -1 || 
            path.indexOf("magisk") !== -1 ||
            path.indexOf("superuser") !== -1) {
            console.log("[*] Hiding file: " + path);
            return false;
        }
        return this.exists.call(this);
    };
    
    console.log("[+] Root detection bypass active");
});
```

Load custom script:

```bash
# Inject script into running app
objection --gadget "com.example.app" explore --startup-script modules/objection/scripts/bypass-root-detection.js
```

### 4. Frida Scripts

Direct Frida usage for advanced hooking:

```python
# modules/objection/scripts/intercept_crypto.py
import frida
import sys

def on_message(message, data):
    if message['type'] == 'send':
        print(f"[*] {message['payload']}")
    else:
        print(message)

# Frida script to hook AES encryption
script_code = """
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");
    
    Cipher.doFinal.overload('[B').implementation = function(data) {
        send({
            "operation": "Cipher.doFinal",
            "data": Java.use("android.util.Base64").encodeToString(data, 0),
            "algorithm": this.getAlgorithm()
        });
        return this.doFinal(data);
    };
    
    console.log("[+] Crypto hooks installed");
});
"""

# Connect to device
device = frida.get_usb_device()
pid = device.spawn(["com.example.app"])
session = device.attach(pid)

# Load script
script = session.create_script(script_code)
script.on('message', on_message)
script.load()

device.resume(pid)
sys.stdin.read()
```

Run Frida script:

```bash
python3 modules/objection/scripts/intercept_crypto.py
```

## Configuration Files

### Network Configuration

```yaml
# configurations/network/proxy.yml
proxy:
  burp:
    host: 192.168.1.100
    port: 8080
    ssl: true
  
  certificate:
    path: ~/.burp/cacert.der
    system_install: true
  
  bypass_apps:
    - com.google.android.gms
    - com.android.vending

iptables:
  redirect_port: 8080
  exclude_local: true
```

### Magisk Module Configuration

```ini
# modules/magisk/frida-server/module.prop
id=frida-server
name=Frida Server
version=16.1.4
versionCode=16104
author=JAMBOREE
description=Automated Frida server for Android instrumentation
```

```bash
# modules/magisk/frida-server/service.sh
#!/system/bin/sh

MODDIR=${0%/*}
FRIDA_BIN="$MODDIR/frida-server"

# Wait for boot
while [ "$(getprop sys.boot_completed)" != "1" ]; do
    sleep 1
done

# Start Frida server
chmod 755 "$FRIDA_BIN"
"$FRIDA_BIN" -l 0.0.0.0:27042 &
```

### Orchestration Configuration

```json
// configurations/jamboree.json
{
  "environment": {
    "android_version": 13,
    "emulator_name": "jamboree-test",
    "architecture": "x86_64"
  },
  "components": {
    "magisk": {
      "enabled": true,
      "version": "26.1",
      "modules": [
        "ssl-unpinning",
        "frida-server",
        "busybox"
      ]
    },
    "burp": {
      "enabled": true,
      "auto_cert": true,
      "upstream_proxy": null
    },
    "objection": {
      "enabled": true,
      "auto_inject": true,
      "startup_scripts": [
        "modules/objection/scripts/bypass-root-detection.js"
      ]
    }
  },
  "logging": {
    "level": "INFO",
    "output": "logs/jamboree.log"
  }
}
```

## Common Testing Workflows

### Workflow 1: SSL Pinning Bypass

```bash
# 1. Start emulator with Burp proxy
./orchestration/start_testing.sh \
  --app com.example.pinned \
  --proxy 192.168.1.100:8080

# 2. Inject Objection and disable pinning
objection --gadget com.example.pinned explore

# Inside objection:
android sslpinning disable
android root disable

# 3. Monitor traffic in Burp Suite
# Application traffic will now appear in Burp HTTP history
```

### Workflow 2: API Key Extraction

```javascript
// modules/objection/scripts/extract-api-keys.js
Java.perform(function() {
    console.log("[*] Searching for API keys");
    
    // Hook SharedPreferences
    var SharedPreferences = Java.use("android.content.SharedPreferences");
    SharedPreferences.getString.implementation = function(key, defValue) {
        var value = this.getString(key, defValue);
        if (key.toLowerCase().indexOf("api") !== -1 || 
            key.toLowerCase().indexOf("key") !== -1 ||
            key.toLowerCase().indexOf("token") !== -1) {
            send({
                "type": "credential",
                "source": "SharedPreferences",
                "key": key,
                "value": value
            });
        }
        return value;
    };
    
    // Hook BuildConfig
    try {
        var BuildConfig = Java.use("com.example.app.BuildConfig");
        var fields = BuildConfig.class.getDeclaredFields();
        fields.forEach(function(field) {
            field.setAccessible(true);
            var name = field.getName();
            if (name.indexOf("API") !== -1 || name.indexOf("KEY") !== -1) {
                var value = field.get(null);
                send({
                    "type": "credential",
                    "source": "BuildConfig",
                    "key": name,
                    "value": value.toString()
                });
            }
        });
    } catch(e) {
        console.log("[-] BuildConfig not found");
    }
});
```

### Workflow 3: Method Tracing

```bash
# Trace all methods in specific class
objection --gadget com.example.app explore

# Inside objection:
android hooking watch class com.example.api.AuthManager --dump-args --dump-return --dump-backtrace

# Generate method trace report
android hooking generate simple com.example.api
```

### Workflow 4: File System Analysis

```bash
# List app data directory
adb shell su -c "ls -la /data/data/com.example.app/"

# Extract databases
adb shell su -c "cp -r /data/data/com.example.app/databases /sdcard/"
adb pull /sdcard/databases ./analysis/

# Analyze SQLite database
sqlite3 ./analysis/databases/app.db
.schema
.tables
SELECT * FROM sensitive_data;

# Extract shared preferences
adb shell su -c "cat /data/data/com.example.app/shared_prefs/config.xml"
```

## Troubleshooting

### Certificate Not Trusted

```bash
# Verify certificate installation
adb shell ls /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in burp.pem | head -1)

# Reinstall if missing
./orchestration/configure_burp.sh --reinstall-cert

# Check system certificate store
adb shell settings get global http_proxy
```

### Frida Connection Failed

```bash
# Check Frida server is running
adb shell su -c "ps | grep frida"

# Restart Frida server
adb shell su -c "pkill frida-server"
adb shell su -c "/data/adb/modules/frida-server/frida-server -l 0.0.0.0:27042 &"

# Verify connectivity
frida-ps -U

# If port forwarding needed
adb forward tcp:27042 tcp:27042
frida-ps -H 127.0.0.1:27042
```

### Magisk Module Not Loading

```bash
# Boot into safe mode
adb reboot

# Check module logs
adb shell su -c "logcat | grep Magisk"

# Manually verify module
adb shell su -c "magisk --list"
adb shell su -c "cat /data/adb/modules/frida-server/module.prop"

# Reinstall module
./orchestration/install_modules.sh --force --module frida-server
```

### App Crashes After Hooking

```javascript
// Add error handling to hooks
Java.perform(function() {
    try {
        var TargetClass = Java.use("com.example.TargetClass");
        TargetClass.targetMethod.implementation = function() {
            console.log("[*] Method called");
            return this.targetMethod.call(this);
        };
    } catch(e) {
        console.log("[-] Hook failed: " + e.message);
    }
});
```

### Proxy Not Intercepting Traffic

```bash
# Check proxy settings
adb shell settings get global http_proxy

# Use iptables for system-wide redirection
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080"
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 192.168.1.100:8080"

# Or use VPN-based interception
./orchestration/configure_burp.sh --vpn-mode
```

## Environment Variables

```bash
# Required
export ANDROID_SDK_ROOT="$HOME/Android/Sdk"
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"

# Optional
export BURP_JAR="$HOME/tools/burpsuite_pro.jar"
export JAMBOREE_CONFIG="./configurations/jamboree.json"
export FRIDA_SERVER_PATH="/data/local/tmp/frida-server"
export MAGISK_MODULES_DIR="./modules/magisk"
```

## Best Practices

1. **Always use isolated environments** - Never test on production devices
2. **Document findings** - Use structured logging for audit trails
3. **Backup AVD snapshots** - Save clean states before testing
4. **Rotate certificates** - Regenerate Burp certificates periodically
5. **Version control hooks** - Keep Frida/Objection scripts in git
6. **Test incrementally** - Hook one component at a time to isolate issues

## References

- Frida documentation: https://frida.re/docs/
- Objection wiki: https://github.com/sensepost/objection/wiki
- Magisk documentation: https://topjohnwu.github.io/Magisk/
- Burp Suite mobile testing: https://portswigger.net/burp/documentation/desktop/mobile
