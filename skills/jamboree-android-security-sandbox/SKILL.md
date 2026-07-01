---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and Frida for penetration testing and reverse engineering
triggers:
  - set up android security testing environment
  - configure jamboree android sandbox
  - intercept android app traffic with burp
  - bypass ssl pinning with objection
  - root android emulator with magisk
  - hook android app methods with frida
  - analyze android malware in sandbox
  - configure android pentest toolkit
---

# JAMBOREE Android Security Sandbox

> Skill by [ara.so](https://ara.so) — Security Skills collection.

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates Magisk root management, Burp Suite proxy interception, Objection/Frida runtime instrumentation, and optimized emulator configurations into a single orchestrated environment for penetration testing and reverse engineering.

## Core Capabilities

- **Magisk Module Management**: Systemless root, module installation, and root hiding
- **Burp Suite Integration**: Automated CA certificate installation and traffic interception
- **Objection/Frida Toolkit**: Runtime hooking, SSL pinning bypass, method tracing
- **Emulator Optimization**: AVD configurations that evade anti-emulation checks
- **Automated Orchestration**: Single-command environment setup and validation

## Installation

### Prerequisites

```bash
# Verify Java installation (JDK 11+)
java -version

# Verify Android SDK and platform tools
adb version
emulator -version

# Install Python dependencies for Objection
pip install objection frida-tools

# Ensure Burp Suite is installed (Community or Pro)
```

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run the orchestration script
./orchestration/setup.sh

# Verify installation
./orchestration/validate.sh
```

### Configuration Files

```bash
# Main configuration
configurations/android/emulator.conf
configurations/network/proxy.conf
configurations/security/certificates.conf
```

## Magisk Module Configuration

### Install Magisk on Emulator

```bash
# Download Magisk APK
adb install modules/magisk/Magisk-v26.1.apk

# Push Magisk modules to device
adb push modules/magisk/systemless/ /sdcard/Download/

# Install module via Magisk Manager
adb shell am start -n com.topjohnwu.magisk/.ui.MainActivity
```

### Systemless Root Setup

```bash
# Check root status
adb shell su -c "id"

# Install BusyBox module
adb push modules/magisk/busybox.zip /sdcard/Download/
adb shell su -c "magisk --install-module /sdcard/Download/busybox.zip"

# Reboot to apply
adb reboot
```

### Hide Root from Target Apps

```bash
# Enable MagiskHide (for older versions)
adb shell su -c "magiskhide enable"

# Add app to hide list
adb shell su -c "magiskhide add com.target.app"

# For Magisk 24+, use DenyList
adb shell su -c "magisk --denylist add com.target.app"
```

## Burp Suite Proxy Configuration

### Install CA Certificate

```bash
# Export Burp CA certificate
# In Burp: Proxy > Options > Import/Export CA Certificate > Certificate in DER format
# Save as burp-cert.der

# Convert to PEM format
openssl x509 -inform DER -in burp-cert.der -out burp-cert.pem

# Get certificate hash
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-cert.pem | head -1)

# Rename certificate
cp burp-cert.pem ${CERT_HASH}.0

# Push to system certificate store
adb root
adb remount
adb push ${CERT_HASH}.0 /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### Configure Proxy on Device

```bash
# Set global proxy (requires root)
adb shell settings put global http_proxy <HOST_IP>:8080

# Or configure via ADB
adb shell settings put global http_proxy $(ip route get 1 | awk '{print $7}'):8080

# Verify proxy settings
adb shell settings get global http_proxy

# Clear proxy
adb shell settings put global http_proxy :0
```

### Automated Proxy Setup Script

```python
#!/usr/bin/env python3
import subprocess
import socket

def get_host_ip():
    """Get host machine IP visible to emulator"""
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    s.connect(("8.8.8.8", 80))
    ip = s.getsockname()[0]
    s.close()
    return ip

def configure_proxy(port=8080):
    """Configure Android device to use Burp proxy"""
    host_ip = get_host_ip()
    proxy_setting = f"{host_ip}:{port}"
    
    # Set proxy
    subprocess.run([
        "adb", "shell", "settings", "put", "global", 
        "http_proxy", proxy_setting
    ])
    
    print(f"Proxy configured: {proxy_setting}")
    
    # Verify
    result = subprocess.run(
        ["adb", "shell", "settings", "get", "global", "http_proxy"],
        capture_output=True, text=True
    )
    print(f"Current proxy: {result.stdout.strip()}")

if __name__ == "__main__":
    configure_proxy()
```

## Objection Runtime Instrumentation

### Basic Objection Commands

```bash
# List running applications
frida-ps -Ua

# Attach to running app
objection -g com.target.app explore

# Spawn app with Objection
objection -g com.target.app explore --startup-command "android hooking watch class_method com.target.app.MainActivity.onCreate"
```

### SSL Pinning Bypass

```bash
# Inside Objection REPL
android sslpinning disable

# Verify bypass
android sslpinning disable --quiet

# Alternative: Use Frida script
frida -U -f com.target.app -l modules/objection/scripts/ssl-bypass.js --no-pause
```

### SSL Pinning Bypass Script (JavaScript)

```javascript
// modules/objection/scripts/ssl-bypass.js
Java.perform(function() {
    console.log("[*] Starting SSL Pinning Bypass");
    
    // TrustManager bypass
    var TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    
    var TrustManagerImpl = Java.registerClass({
        name: 'com.custom.TrustManagerImpl',
        implements: [TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    var TrustManagers = [TrustManagerImpl.$new()];
    var SSLContextInstance = SSLContext.getInstance("TLS");
    SSLContextInstance.init(null, TrustManagers, null);
    SSLContext.setDefault(SSLContextInstance);
    
    console.log("[+] SSL Pinning bypassed");
});
```

### Method Hooking

```bash
# Watch method calls
android hooking watch class_method com.target.app.crypto.AES.encrypt

# Hook method and modify return value
android hooking set return_value com.target.app.auth.validateToken true

# List classes in memory
android hooking list classes

# Search for specific classes
android hooking search classes auth

# List methods of a class
android hooking list class_methods com.target.app.MainActivity
```

### Runtime Method Hooking Script

```javascript
// modules/objection/scripts/hook-auth.js
Java.perform(function() {
    var MainActivity = Java.use('com.target.app.MainActivity');
    
    // Hook onCreate method
    MainActivity.onCreate.overload('android.os.Bundle').implementation = function(bundle) {
        console.log("[*] MainActivity.onCreate called");
        console.log("[*] Bundle: " + bundle);
        
        // Call original method
        this.onCreate(bundle);
        
        console.log("[+] onCreate executed");
    };
    
    // Hook authentication method
    var AuthManager = Java.use('com.target.app.auth.AuthManager');
    AuthManager.isUserAuthenticated.implementation = function() {
        console.log("[*] isUserAuthenticated called");
        console.log("[!] Bypassing authentication check");
        return true; // Always return authenticated
    };
    
    // Hook encryption method
    var CryptoUtils = Java.use('com.target.app.crypto.CryptoUtils');
    CryptoUtils.encrypt.implementation = function(data, key) {
        console.log("[*] Encrypt called");
        console.log("[*] Data: " + data);
        console.log("[*] Key: " + key);
        
        var result = this.encrypt(data, key);
        console.log("[*] Encrypted: " + result);
        return result;
    };
});
```

### Database Exploration

```bash
# List SQLite databases
android hooking list activities
android sqlite connect /data/data/com.target.app/databases/app.db

# Query database
android sqlite execute /data/data/com.target.app/databases/app.db "SELECT * FROM users"

# Export database
android file download /data/data/com.target.app/databases/app.db ./app.db
```

### SharedPreferences Manipulation

```bash
# List SharedPreferences
android prefs list

# Read preference value
android prefs get shared_prefs com.target.app_preferences token

# Write preference value
android prefs set com.target.app_preferences is_premium true --type boolean
```

## Emulator Configuration

### Create Optimized AVD

```bash
# Create AVD with Google APIs (for root)
avdmanager create avd \
  -n jamboree_test \
  -k "system-images;android-30;google_apis;x86_64" \
  -d pixel_5 \
  --force

# Launch emulator with writable system
emulator -avd jamboree_test -writable-system -no-snapshot-load &

# Wait for boot
adb wait-for-device

# Remount system as writable
adb root
adb remount
```

### Anti-Emulation Evasion Configuration

```bash
# Modify build.prop to mimic real device
adb root
adb remount
adb pull /system/build.prop ./build.prop

# Edit build.prop
cat >> build.prop << EOF
ro.product.manufacturer=samsung
ro.product.model=SM-G998B
ro.build.fingerprint=samsung/p3sxxx/p3s:12/SP1A.210812.016/G998BXXU5CVEK:user/release-keys
ro.hardware=qcom
EOF

# Push modified build.prop
adb push build.prop /system/build.prop
adb shell chmod 644 /system/build.prop
adb reboot
```

## Automated Testing Workflow

### Complete Setup Script

```python
#!/usr/bin/env python3
import subprocess
import time
import os

class JamboreeOrchestrator:
    def __init__(self, target_package):
        self.target_package = target_package
        self.burp_port = 8080
        
    def start_emulator(self):
        """Launch Android emulator"""
        print("[*] Starting emulator...")
        subprocess.Popen([
            "emulator", "-avd", "jamboree_test",
            "-writable-system", "-no-snapshot-load"
        ])
        subprocess.run(["adb", "wait-for-device"])
        time.sleep(10)
        print("[+] Emulator ready")
        
    def configure_proxy(self):
        """Set up Burp Suite proxy"""
        print("[*] Configuring proxy...")
        subprocess.run(["adb", "root"])
        subprocess.run(["adb", "remount"])
        
        # Install certificate
        subprocess.run([
            "adb", "push", 
            "modules/burp/certificates/9a5ba575.0",
            "/system/etc/security/cacerts/"
        ])
        
        # Set proxy
        host_ip = self._get_host_ip()
        subprocess.run([
            "adb", "shell", "settings", "put", "global",
            "http_proxy", f"{host_ip}:{self.burp_port}"
        ])
        print(f"[+] Proxy configured: {host_ip}:{self.burp_port}")
        
    def install_app(self, apk_path):
        """Install target APK"""
        print(f"[*] Installing {apk_path}...")
        subprocess.run(["adb", "install", "-r", apk_path])
        print("[+] App installed")
        
    def start_objection(self):
        """Launch Objection with SSL bypass"""
        print("[*] Starting Objection...")
        subprocess.Popen([
            "objection", "-g", self.target_package, "explore",
            "--startup-command", "android sslpinning disable"
        ])
        
    def _get_host_ip(self):
        """Get host IP for emulator"""
        import socket
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.connect(("8.8.8.8", 80))
        ip = s.getsockname()[0]
        s.close()
        return ip
        
    def run(self, apk_path):
        """Execute full workflow"""
        self.start_emulator()
        self.configure_proxy()
        self.install_app(apk_path)
        self.start_objection()
        print("[+] JAMBOREE environment ready")

if __name__ == "__main__":
    orchestrator = JamboreeOrchestrator("com.target.app")
    orchestrator.run("target_app.apk")
```

## Common Testing Patterns

### Traffic Interception Workflow

```bash
# 1. Start emulator
emulator -avd jamboree_test -writable-system &

# 2. Configure proxy
adb root && adb remount
./scripts/configure-proxy.sh

# 3. Install target app
adb install -r target.apk

# 4. Start Burp Suite
# Configure listener on 0.0.0.0:8080

# 5. Launch app and monitor traffic in Burp
adb shell monkey -p com.target.app 1
```

### Root Detection Bypass

```javascript
// modules/objection/scripts/root-bypass.js
Java.perform(function() {
    // RootBeer library bypass
    var RootBeer = Java.use('com.scottyab.rootbeer.RootBeer');
    RootBeer.isRooted.implementation = function() {
        console.log("[*] RootBeer.isRooted bypassed");
        return false;
    };
    
    // Common root detection methods
    var Runtime = Java.use('java.lang.Runtime');
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.includes("su") || cmd.includes("busybox")) {
            console.log("[!] Blocked root check: " + cmd);
            throw new Error("Command not found");
        }
        return this.exec(cmd);
    };
    
    // File existence checks
    var File = Java.use('java.io.File');
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        if (path.includes("su") || path.includes("magisk")) {
            console.log("[!] Hiding file: " + path);
            return false;
        }
        return this.exists();
    };
});
```

### API Response Manipulation

```javascript
// modules/objection/scripts/api-intercept.js
Java.perform(function() {
    var OkHttpClient = Java.use('okhttp3.OkHttpClient');
    var Interceptor = Java.use('okhttp3.Interceptor');
    
    var CustomInterceptor = Java.registerClass({
        name: 'com.custom.APIInterceptor',
        implements: [Interceptor],
        methods: {
            intercept: function(chain) {
                var request = chain.request();
                console.log("[*] Request: " + request.url());
                
                var response = chain.proceed(request);
                var responseBody = response.body().string();
                
                console.log("[*] Response: " + responseBody);
                
                // Modify response
                if (responseBody.includes('"premium":false')) {
                    responseBody = responseBody.replace(
                        '"premium":false', 
                        '"premium":true'
                    );
                    console.log("[!] Modified response to premium");
                }
                
                var MediaType = Java.use('okhttp3.MediaType');
                var ResponseBody = Java.use('okhttp3.ResponseBody');
                var newBody = ResponseBody.create(
                    MediaType.parse("application/json"),
                    responseBody
                );
                
                return response.newBuilder().body(newBody).build();
            }
        }
    });
    
    console.log("[+] Custom interceptor registered");
});
```

## Troubleshooting

### Certificate Installation Fails

```bash
# Verify system partition is writable
adb root
adb remount
adb shell mount | grep system

# If read-only, restart emulator with -writable-system
emulator -avd jamboree_test -writable-system -no-snapshot-load

# Manually mount as writable
adb shell mount -o rw,remount /system
```

### Objection Connection Fails

```bash
# Check Frida server is running
adb shell ps | grep frida

# Install/restart Frida server
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

# Verify Frida version compatibility
frida --version
objection --version
```

### Magisk Module Not Loading

```bash
# Check module status
adb shell su -c "ls -la /data/adb/modules/"

# View Magisk logs
adb shell su -c "cat /cache/magisk.log"

# Remove conflicting module
adb shell su -c "rm -rf /data/adb/modules/<module_name>"
adb reboot
```

### Proxy Traffic Not Captured

```bash
# Verify proxy settings
adb shell settings get global http_proxy

# Check connectivity to Burp
adb shell ping -c 3 <HOST_IP>

# Test with curl
adb shell curl -x <HOST_IP>:8080 http://example.com

# Check iptables rules (if using transparent proxy)
adb shell su -c "iptables -t nat -L"
```

## Environment Variables

```bash
# Set in configurations/security/env.conf
export BURP_HOST=${BURP_HOST:-"0.0.0.0"}
export BURP_PORT=${BURP_PORT:-8080}
export ANDROID_SERIAL=${ANDROID_SERIAL:-"emulator-5554"}
export FRIDA_SERVER_PATH=${FRIDA_SERVER_PATH:-"/data/local/tmp/frida-server"}
export MAGISK_MODULE_DIR=${MAGISK_MODULE_DIR:-"./modules/magisk"}
```

## Best Practices

1. **Snapshot Management**: Create AVD snapshots after successful configuration for quick recovery
2. **Logging**: Enable verbose logging in `configurations/security/logging.conf` for debugging
3. **Version Compatibility**: Match Frida/Objection versions with target Android API level
4. **Network Isolation**: Use separate network interfaces for testing to avoid conflicts
5. **Module Testing**: Test Magisk modules individually before combining
6. **Certificate Pinning**: Always test SSL bypass scripts before live assessment
7. **Clean State**: Factory reset emulator between different target applications
