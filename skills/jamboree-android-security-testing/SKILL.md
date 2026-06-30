---
name: jamboree-android-security-testing
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for penetration testing and reverse engineering
triggers:
  - set up android security testing environment
  - configure burp suite with android emulator
  - install magisk modules for security testing
  - intercept android app traffic with burp
  - use objection to hook android app
  - bypass ssl pinning on android
  - create rooted android testing sandbox
  - automate frida injection for android
---

# JAMBOREE Android Security Testing

> Skill by [ara.so](https://ara.so) — Security Skills collection.

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified framework for Android application security testing that integrates Magisk modules, Burp Suite proxy configuration, Objection/Frida runtime hooking, and rooted emulator management into a single orchestrated environment.

## What It Does

JAMBOREE automates the complex setup of Android security testing environments by:
- Installing and configuring Magisk modules for systemless root
- Setting up Burp Suite proxy with automatic CA certificate installation
- Deploying Objection/Frida for runtime instrumentation
- Optimizing Android emulators with anti-detection measures
- Providing pre-built scripts for common security testing scenarios

## Installation

### Prerequisites

Ensure you have the required dependencies:

```bash
# Check Java version (requires JDK 11+)
java -version

# Verify Android SDK installation
adb version

# Install Python dependencies for Objection/Frida
pip install frida-tools objection
```

### Core Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run environment validation
./orchestration/validators/check_environment.sh

# Deploy the full stack (Magisk + Burp + Objection)
./orchestration/deployers/deploy_all.sh

# Calibrate configuration for your environment
./orchestration/calibrators/configure.sh
```

## Key Components

### 1. Magisk Module Management

Install systemless root and security testing modules:

```bash
# Push Magisk modules to device
adb push modules/magisk/systemless/MagiskHideProps.zip /sdcard/
adb push modules/magisk/systemless/BusyBox.zip /sdcard/

# Install via Magisk Manager (programmatically)
adb shell su -c "magisk --install-module /sdcard/MagiskHideProps.zip"
adb shell su -c "magisk --install-module /sdcard/BusyBox.zip"

# Verify installation
adb shell su -c "ls /data/adb/modules"
```

### 2. Burp Suite Certificate Installation

Automatically install and trust Burp's CA certificate:

```bash
# Export Burp certificate (DER format)
# From Burp: Proxy > Options > Import/Export CA Certificate

# Convert to PEM and calculate hash
openssl x509 -inform DER -in burp_cert.der -out burp_cert.pem
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp_cert.pem | head -1)

# Install as system certificate
adb root
adb remount
adb push burp_cert.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

Using the JAMBOREE automated script:

```bash
# Automated certificate installation
./modules/burp/certificates/install_burp_cert.sh --burp-cert /path/to/burp_cert.der
```

### 3. Proxy Configuration

Set up device-wide proxy routing:

```bash
# Configure WiFi proxy via ADB
adb shell settings put global http_proxy "192.168.1.100:8080"

# Or use iptables for transparent proxying
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080"
adb shell su -c "iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 192.168.1.100:8080"

# Verify proxy settings
adb shell settings get global http_proxy
```

JAMBOREE proxy setup script:

```bash
# Configure proxy with certificate pinning bypass
./modules/burp/configure_proxy.sh \
  --proxy-host 192.168.1.100 \
  --proxy-port 8080 \
  --bypass-pinning true
```

### 4. Objection Runtime Hooking

Launch Objection to hook into running applications:

```bash
# List running applications
frida-ps -U

# Spawn application with Objection
objection -g com.example.target explore

# Attach to running process
objection -g com.example.target explore --attach
```

Common Objection commands within the REPL:

```javascript
// Bypass SSL pinning
android sslpinning disable

// List activities
android hooking list activities

// Watch class methods
android hooking watch class_method com.example.MainActivity.onCreate --dump-args --dump-return

// Get current activity
android hooking get current_activity

// Dump SharedPreferences
android keystore list

// List loaded classes
android hooking list classes
```

### 5. Frida Script Injection

Use pre-built Frida scripts from the JAMBOREE collection:

```javascript
// modules/objection/scripts/ssl_bypass.js
Java.perform(function() {
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    
    SSLContext.init.overload(
        "[Ljavax.net.ssl.KeyManager;",
        "[Ljavax.net.ssl.TrustManager;",
        "java.security.SecureRandom"
    ).implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[+] SSLContext.init() hooked");
        
        var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
        var X509TrustManager = Java.registerClass({
            name: "com.sensepost.test.TrustManager",
            implements: [TrustManager],
            methods: {
                checkClientTrusted: function(chain, authType) {},
                checkServerTrusted: function(chain, authType) {},
                getAcceptedIssuers: function() { return []; }
            }
        });
        
        var TrustManagers = [X509TrustManager.$new()];
        this.init(keyManager, TrustManagers, secureRandom);
    };
});
```

Run the script:

```bash
frida -U -f com.example.target -l modules/objection/scripts/ssl_bypass.js --no-pause
```

### 6. Emulator Optimization

Create an optimized AVD for security testing:

```bash
# Create AVD with Google APIs (for root access)
avdmanager create avd \
  -n SecurityTesting \
  -k "system-images;android-30;google_apis;x86_64" \
  -d "pixel_3a"

# Launch with writable system partition
emulator -avd SecurityTesting -writable-system -no-snapshot-load

# Install Magisk on emulator
adb root
adb remount
adb push configurations/android/magisk_patched_boot.img /sdcard/
adb shell su -c "dd if=/sdcard/magisk_patched_boot.img of=/dev/block/by-name/boot"
adb reboot
```

JAMBOREE automated emulator setup:

```bash
./configurations/android/create_security_avd.sh \
  --android-version 30 \
  --device-profile pixel_3a \
  --enable-magisk true \
  --enable-root-hiding true
```

## Configuration Files

### Network Configuration

Edit `configurations/network/proxy_rules.json`:

```json
{
  "upstream_proxy": {
    "host": "192.168.1.100",
    "port": 8080,
    "protocol": "http"
  },
  "bypass_domains": [
    "*.google.com",
    "play.googleapis.com"
  ],
  "interception_mode": "transparent",
  "certificate_pinning_bypass": true
}
```

### Security Configuration

Edit `configurations/security/testing_profile.yaml`:

```yaml
magisk:
  hide_root_from:
    - com.example.bankingapp
    - com.android.chrome
  modules_enabled:
    - MagiskHideProps
    - BusyBox
    - SystemlessHosts

frida:
  spawn_mode: true
  auto_attach: false
  script_timeout: 30000

burp:
  auto_install_cert: true
  trust_user_certs: true
  invisible_proxy_mode: false
```

## Common Patterns

### Pattern 1: Full Application Assessment

```bash
#!/bin/bash
# Complete assessment workflow

TARGET_APP="com.example.target"
BURP_HOST="192.168.1.100"
BURP_PORT="8080"

# 1. Setup environment
./orchestration/deployers/deploy_all.sh

# 2. Configure proxy
adb shell settings put global http_proxy "${BURP_HOST}:${BURP_PORT}"

# 3. Install target application
adb install target_app.apk

# 4. Launch with Objection
objection -g ${TARGET_APP} explore <<EOF
android sslpinning disable
android hooking watch class com.example.target.* --dump-args --dump-return
exit
EOF

# 5. Interact with app (manual testing)
echo "App is ready for testing. Press Enter when done."
read

# 6. Cleanup
adb shell settings delete global http_proxy
adb uninstall ${TARGET_APP}
```

### Pattern 2: SSL Pinning Bypass

```javascript
// modules/objection/scripts/universal_ssl_bypass.js
Java.perform(function() {
    console.log("[*] Universal SSL Bypass loaded");
    
    // Hook OkHttp3
    try {
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
            console.log("[+] OkHttp3 CertificatePinner.check() bypassed");
        };
    } catch(err) {
        console.log("[-] OkHttp3 not found");
    }
    
    // Hook TrustManager
    var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    
    var TrustManager = Java.registerClass({
        name: "dev.jamboree.TrustManager",
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
        "[Ljavax.net.ssl.KeyManager;",
        "[Ljavax.net.ssl.TrustManager;",
        "java.security.SecureRandom"
    );
    
    SSLContext_init.implementation = function(keyManager, trustManager, secureRandom) {
        console.log("[+] SSLContext.init() bypassed");
        SSLContext_init.call(this, keyManager, TrustManagers, secureRandom);
    };
});
```

### Pattern 3: Runtime Method Tracing

```javascript
// modules/objection/scripts/method_tracer.js
Java.perform(function() {
    var targetClass = "com.example.target.SecurityUtils";
    var SecurityUtils = Java.use(targetClass);
    
    SecurityUtils.encrypt.overload('java.lang.String').implementation = function(plaintext) {
        console.log("[+] encrypt() called");
        console.log("    Input: " + plaintext);
        
        var result = this.encrypt(plaintext);
        
        console.log("    Output: " + result);
        return result;
    };
    
    SecurityUtils.decrypt.overload('java.lang.String').implementation = function(ciphertext) {
        console.log("[+] decrypt() called");
        console.log("    Input: " + ciphertext);
        
        var result = this.decrypt(ciphertext);
        
        console.log("    Output: " + result);
        return result;
    };
    
    console.log("[*] Tracing " + targetClass);
});
```

Run the tracer:

```bash
frida -U -f com.example.target -l modules/objection/scripts/method_tracer.js --no-pause
```

### Pattern 4: Root Detection Bypass

```javascript
// modules/objection/scripts/root_detection_bypass.js
Java.perform(function() {
    console.log("[*] Root Detection Bypass loaded");
    
    // Bypass common root detection methods
    var RootBeer = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer.isRooted.implementation = function() {
        console.log("[+] RootBeer.isRooted() bypassed");
        return false;
    };
    
    // Bypass file existence checks
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.indexOf("su") > -1 || 
            path.indexOf("magisk") > -1 || 
            path.indexOf("busybox") > -1) {
            console.log("[+] File.exists() bypassed for: " + path);
            return false;
        }
        return this.exists();
    };
    
    // Bypass package manager checks
    var PackageManager = Java.use("android.content.pm.PackageManager");
    PackageManager.getPackageInfo.overload('java.lang.String', 'int').implementation = function(packageName, flags) {
        if (packageName.indexOf("magisk") > -1 || 
            packageName.indexOf("supersu") > -1) {
            console.log("[+] PackageManager.getPackageInfo() bypassed for: " + packageName);
            throw Java.use("android.content.pm.PackageManager$NameNotFoundException").$new();
        }
        return this.getPackageInfo(packageName, flags);
    };
});
```

## Troubleshooting

### Issue: Burp Certificate Not Trusted

**Symptom**: HTTPS connections fail with certificate errors despite installing Burp's certificate.

**Solution**:

```bash
# Verify certificate is in system store
adb shell ls -la /system/etc/security/cacerts/ | grep $(openssl x509 -inform PEM -subject_hash_old -in burp_cert.pem | head -1)

# If missing, reinstall with correct permissions
adb root && adb remount
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp_cert.pem | head -1)
adb push burp_cert.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chown root:root /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### Issue: Frida Cannot Attach to Process

**Symptom**: `frida.ServerNotRunningError` or connection timeout.

**Solution**:

```bash
# Download and push frida-server matching your Frida version
FRIDA_VERSION=$(frida --version)
wget https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-x86_64.xz
unxz frida-server-${FRIDA_VERSION}-android-x86_64.xz

adb push frida-server-${FRIDA_VERSION}-android-x86_64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell su -c "/data/local/tmp/frida-server &"

# Verify frida-server is running
adb shell ps | grep frida-server
```

### Issue: Magisk Modules Not Loading

**Symptom**: Modules show as installed but don't function.

**Solution**:

```bash
# Check Magisk log
adb shell su -c "cat /cache/magisk.log"

# Remove conflicting modules
adb shell su -c "rm -rf /data/adb/modules/conflicting_module"

# Clear Magisk cache and reboot
adb shell su -c "rm -rf /data/adb/magisk.db"
adb reboot

# Reinstall modules after boot
adb shell su -c "magisk --install-module /sdcard/MagiskModule.zip"
```

### Issue: Objection Commands Fail

**Symptom**: `android sslpinning disable` or other commands return errors.

**Solution**:

```bash
# Update Objection and Frida to latest versions
pip install --upgrade objection frida-tools

# Clear Objection cache
rm -rf ~/.objection

# Launch with verbose logging
objection -g com.example.target explore --debug
```

### Issue: Emulator Performance Issues

**Symptom**: Slow emulator performance during testing.

**Solution**:

```bash
# Launch emulator with more resources
emulator -avd SecurityTesting \
  -memory 4096 \
  -cores 4 \
  -gpu host \
  -writable-system \
  -no-snapshot-load

# Disable animations to speed up testing
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0
```

## Integration with Automation

### GitHub Actions CI/CD Example

```yaml
# .github/workflows/security-test.yml
name: Android Security Testing

on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
      
      - name: Install JAMBOREE
        run: |
          git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git jamboree
          cd jamboree
          ./orchestration/deployers/deploy_all.sh
      
      - name: Start Emulator
        run: |
          echo "no" | avdmanager create avd -n test -k "system-images;android-30;google_apis;x86_64"
          emulator -avd test -no-window -no-audio -no-boot-anim &
          adb wait-for-device
      
      - name: Run Security Tests
        run: |
          frida -U -f com.example.app -l jamboree/modules/objection/scripts/ssl_bypass.js --no-pause
```

## Advanced Usage

### Custom Frida Script with Persistent Storage

```javascript
// modules/objection/scripts/data_exfiltration.js
Java.perform(function() {
    var FileOutputStream = Java.use("java.io.FileOutputStream");
    var results = [];
    
    FileOutputStream.write.overload('[B').implementation = function(bytes) {
        var data = bytes.map(function(b) { return String.fromCharCode(b & 0xff); }).join('');
        
        if (data.indexOf("password") > -1 || data.indexOf("token") > -1) {
            console.log("[!] Sensitive data detected:");
            console.log(data);
            results.push({
                timestamp: Date.now(),
                data: data
            });
        }
        
        return this.write(bytes);
    };
    
    // Export results on exit
    Java.use("android.app.Activity").onDestroy.implementation = function() {
        console.log("[*] Exporting " + results.length + " results");
        send({type: "results", data: results});
        this.onDestroy();
    };
});
```

This skill provides comprehensive guidance for using JAMBOREE to set up and conduct Android security testing with industry-standard tools in an automated, repeatable environment.
