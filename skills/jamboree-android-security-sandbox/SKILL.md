---
name: jamboree-android-security-sandbox
description: Configure and operate JAMBOREE Android security testing environment with Magisk, Burp Suite, Objection, and root emulation
triggers:
  - "set up JAMBOREE Android security testing environment"
  - "configure Burp Suite proxy for Android emulator"
  - "bypass Android certificate pinning with Objection"
  - "deploy Magisk modules for security testing"
  - "intercept Android app HTTPS traffic"
  - "hook Android runtime methods with Frida"
  - "configure rooted Android emulator for penetration testing"
  - "troubleshoot Android security testing sandbox"
---

# JAMBOREE Android Security Sandbox

> Skill by [ara.so](https://ara.so) — Security Skills collection.

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates Magisk root management, Burp Suite proxy interception, Objection/Frida runtime instrumentation, and optimized emulator configurations into a single orchestrated environment for penetration testing and reverse engineering.

## What JAMBOREE Does

JAMBOREE eliminates the fragmentation of Android security testing by providing:

- **Magisk Module Automation**: Systemless root with BusyBox and custom init scripts
- **Burp Suite Integration**: Automated CA certificate installation and proxy configuration
- **Objection Runtime Toolkit**: Pre-configured Frida scripts for hooking, tracing, and SSL bypass
- **Emulator Optimization**: AVD configurations that evade anti-emulation detection
- **Orchestration Layer**: Automated dependency resolution and configuration validation

## Installation

### Prerequisites

```bash
# Verify Java 11+
java -version

# Verify Android SDK
which adb
adb --version

# Install Python dependencies for Objection
pip3 install objection frida-tools
```

### Initial Setup

```bash
# Clone repository
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run orchestration script (validates environment and deploys components)
./orchestration/deploy.sh

# Configure Burp Suite proxy (edit with your Burp instance details)
nano configurations/network/proxy.conf
```

### Configuration Files

**configurations/network/proxy.conf**
```ini
# Burp Suite proxy settings
PROXY_HOST=127.0.0.1
PROXY_PORT=8080
ENABLE_SSL_PASSTHROUGH=true
UPSTREAM_PROXY=

# Certificate paths
CA_CERT_PATH=./modules/burp/certificates/cacert.der
SYSTEM_CERT_NAME=9a5ba575.0
```

**configurations/android/emulator.conf**
```ini
# AVD configuration
AVD_NAME=JAMBOREE_Security_Test
ANDROID_VERSION=11
ARCH=x86_64
RAM_SIZE=4096
ENABLE_ROOT=true
MAGISK_VERSION=26.1
```

## Key Commands

### Environment Management

```bash
# Start JAMBOREE environment (launches emulator with Magisk)
./jamboree.sh start

# Stop environment
./jamboree.sh stop

# Validate configuration
./orchestration/validators/check_env.sh

# Repair broken components
./orchestration/deployers/repair.sh
```

### Magisk Module Operations

```bash
# List installed Magisk modules
adb shell su -c magisk --list

# Install custom Magisk module
adb push ./modules/magisk/custom_module.zip /sdcard/
adb shell su -c magisk --install-module /sdcard/custom_module.zip

# Enable/disable module
adb shell su -c magisk --enable <module_id>
adb shell su -c magisk --disable <module_id>
```

### Burp Suite Certificate Installation

```bash
# Export Burp CA certificate (from Burp Suite: Proxy > Options > Import/Export CA certificate)
# Save as DER format to ./modules/burp/certificates/cacert.der

# Convert and install as system certificate
./modules/burp/install_cert.sh

# Verify certificate installation
adb shell su -c ls /system/etc/security/cacerts/ | grep 9a5ba575
```

### Objection Usage

```bash
# List running applications
frida-ps -Ua

# Spawn application with Objection
objection -g com.example.targetapp explore

# Attach to running application
objection -g com.example.targetapp attach
```

## Real Code Examples

### Python: Automated SSL Pinning Bypass

```python
#!/usr/bin/env python3
import frida
import sys

def on_message(message, data):
    if message['type'] == 'send':
        print(f"[*] {message['payload']}")
    else:
        print(message)

# Frida script to bypass SSL pinning
bypass_script = """
Java.perform(function() {
    console.log("[*] Bypassing SSL pinning...");
    
    // Hook OkHttp3 CertificatePinner
    var CertificatePinner = Java.use('okhttp3.CertificatePinner');
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(hostname, peerCertificates) {
        console.log('[+] SSL Pinning bypassed for: ' + hostname);
        return;
    };
    
    // Hook TrustManagerImpl
    var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
    TrustManagerImpl.verifyChain.implementation = function(untrustedChain, trustAnchorChain, host, clientAuth, ocspData, tlsSctData) {
        console.log('[+] Certificate verification bypassed for: ' + host);
        return untrustedChain;
    };
    
    console.log("[*] SSL pinning bypass complete");
});
"""

# Connect to device
device = frida.get_usb_device()
pid = device.spawn(["com.example.targetapp"])
session = device.attach(pid)

# Load and execute script
script = session.create_script(bypass_script)
script.on('message', on_message)
script.load()
device.resume(pid)

# Keep script running
sys.stdin.read()
```

### Bash: Automated Proxy Configuration

```bash
#!/bin/bash
# configure_proxy.sh - Set up system-wide proxy for Android emulator

PROXY_HOST="${BURP_PROXY_HOST:-127.0.0.1}"
PROXY_PORT="${BURP_PROXY_PORT:-8080}"

# Configure global proxy settings
adb shell settings put global http_proxy "${PROXY_HOST}:${PROXY_PORT}"

# Configure Wi-Fi proxy (if using Wi-Fi)
adb shell "su -c 'settings put global http_proxy ${PROXY_HOST}:${PROXY_PORT}'"

# Verify proxy configuration
echo "[*] Current proxy settings:"
adb shell settings get global http_proxy

# Add iptables rules for transparent proxying
adb shell "su -c 'iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination ${PROXY_HOST}:${PROXY_PORT}'"
adb shell "su -c 'iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination ${PROXY_HOST}:${PROXY_PORT}'"

echo "[+] Proxy configuration complete"
```

### JavaScript: Objection Runtime Method Tracing

```javascript
// Save as trace_methods.js in modules/objection/scripts/

// Hook all methods in a specific class
function traceClass(className) {
    Java.perform(function() {
        var targetClass = Java.use(className);
        var methods = targetClass.class.getDeclaredMethods();
        
        methods.forEach(function(method) {
            var methodName = method.getName();
            console.log('[*] Hooking: ' + className + '.' + methodName);
            
            try {
                targetClass[methodName].implementation = function() {
                    console.log('[+] Called: ' + className + '.' + methodName);
                    console.log('    Arguments: ' + JSON.stringify(arguments));
                    var result = this[methodName].apply(this, arguments);
                    console.log('    Return: ' + JSON.stringify(result));
                    return result;
                };
            } catch(e) {
                console.log('[-] Failed to hook: ' + methodName);
            }
        });
    });
}

// Usage within Objection session
traceClass('com.example.targetapp.crypto.CryptoHelper');
```

### Python: Automated APK Analysis Pipeline

```python
#!/usr/bin/env python3
import subprocess
import json
import os

class JAMBOREEAnalyzer:
    def __init__(self, apk_path):
        self.apk_path = apk_path
        self.package_name = self._get_package_name()
    
    def _get_package_name(self):
        """Extract package name from APK"""
        result = subprocess.run(
            ['aapt', 'dump', 'badging', self.apk_path],
            capture_output=True,
            text=True
        )
        for line in result.stdout.split('\n'):
            if line.startswith('package: name='):
                return line.split("'")[1]
        return None
    
    def install_app(self):
        """Install APK to emulator"""
        print(f"[*] Installing {self.package_name}")
        subprocess.run(['adb', 'install', '-r', self.apk_path])
    
    def configure_interception(self):
        """Set up Burp proxy for app"""
        print("[*] Configuring network interception")
        subprocess.run([
            'adb', 'shell', 'settings', 'put', 'global',
            'http_proxy', f'{os.getenv("BURP_PROXY_HOST", "127.0.0.1")}:8080'
        ])
    
    def spawn_with_objection(self):
        """Launch app with Objection attached"""
        print(f"[*] Spawning {self.package_name} with Objection")
        subprocess.run([
            'objection', '-g', self.package_name,
            'explore', '--startup-command',
            'android sslpinning disable'
        ])
    
    def extract_data(self):
        """Extract app data using Objection"""
        commands = [
            'env',  # Show environment
            'android keystore list',  # List keystore entries
            'sqlite connect databases/app.db .dump',  # Dump SQLite
            'file download /data/data/{}/shared_prefs ./output/'.format(self.package_name)
        ]
        
        for cmd in commands:
            print(f"[*] Running: {cmd}")
            subprocess.run([
                'objection', '-g', self.package_name,
                'run', cmd
            ])

# Usage
if __name__ == '__main__':
    analyzer = JAMBOREEAnalyzer('/path/to/target.apk')
    analyzer.install_app()
    analyzer.configure_interception()
    analyzer.spawn_with_objection()
    analyzer.extract_data()
```

## Common Patterns

### Pattern 1: Full Application Assessment Workflow

```bash
#!/bin/bash
# full_assessment.sh

APK_PATH="$1"
PACKAGE_NAME=$(aapt dump badging "$APK_PATH" | grep package | awk '{print $2}' | sed s/name=//g | tr -d "'")

echo "[*] Starting JAMBOREE environment"
./jamboree.sh start

echo "[*] Installing application"
adb install -r "$APK_PATH"

echo "[*] Installing Burp certificate"
./modules/burp/install_cert.sh

echo "[*] Configuring proxy"
adb shell settings put global http_proxy "127.0.0.1:8080"

echo "[*] Starting Objection session with SSL bypass"
objection -g "$PACKAGE_NAME" explore --startup-command "android sslpinning disable"
```

### Pattern 2: Certificate Pinning Detection

```javascript
// Run in Objection console
// Detect common pinning implementations

Java.perform(function() {
    console.log("[*] Scanning for certificate pinning implementations...");
    
    // Check OkHttp3
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        console.log('[+] Found: OkHttp3 CertificatePinner');
    } catch(e) {}
    
    // Check TrustKit
    try {
        var TrustKit = Java.use('com.datatheorem.android.trustkit.TrustKit');
        console.log('[+] Found: TrustKit');
    } catch(e) {}
    
    // Check Network Security Config
    try {
        var NetworkSecurityConfig = Java.use('android.security.net.config.NetworkSecurityConfig');
        console.log('[+] Found: Network Security Config');
    } catch(e) {}
});
```

### Pattern 3: Root Detection Bypass

```bash
# Deploy Magisk Hide and configure root hiding
adb shell su -c magisk --denylist add com.example.targetapp

# Hide Magisk Manager from app
adb shell su -c magisk --hide

# Verify root hiding
adb shell su -c magisk --denylist status
```

### Pattern 4: Dynamic Code Analysis

```python
#!/usr/bin/env python3
import frida

# Hook crypto operations
crypto_hook = """
Java.perform(function() {
    var Cipher = Java.use('javax.crypto.Cipher');
    
    Cipher.getInstance.overload('java.lang.String').implementation = function(transformation) {
        console.log('[+] Cipher.getInstance called with: ' + transformation);
        return this.getInstance(transformation);
    };
    
    Cipher.doFinal.overload('[B').implementation = function(input) {
        console.log('[+] Cipher.doFinal input: ' + Java.use('android.util.Base64').encodeToString(input, 0));
        var result = this.doFinal(input);
        console.log('[+] Cipher.doFinal output: ' + Java.use('android.util.Base64').encodeToString(result, 0));
        return result;
    };
});
"""

device = frida.get_usb_device()
session = device.attach("com.example.targetapp")
script = session.create_script(crypto_hook)
script.load()
input()
```

## Configuration Examples

### Burp Suite Extension Configuration

```json
{
  "proxy": {
    "intercept_client_requests": {
      "do_intercept": true,
      "automatic_fix_missing_or_superfluous_content_length_headers": true
    },
    "intercept_server_responses": {
      "do_intercept": false
    },
    "match_replace_rules": [
      {
        "enabled": true,
        "type": "response_header",
        "match": "^Strict-Transport-Security:.*$",
        "replace": ""
      }
    ]
  },
  "target": {
    "scope": {
      "include": [
        {"protocol": "https", "host": "^api\\.example\\.com$"}
      ]
    }
  }
}
```

### Magisk Module Configuration

```
# module.prop for custom Magisk module
id=jamboree_ssl_bypass
name=JAMBOREE SSL Bypass
version=1.0
versionCode=1
author=JAMBOREE
description=System-level SSL certificate pinning bypass
```

## Troubleshooting

### Issue: Burp Certificate Not Trusted

```bash
# Check certificate installation
adb shell su -c ls -la /system/etc/security/cacerts/

# Verify certificate hash
openssl x509 -inform DER -in cacert.der -subject_hash_old -noout

# Reinstall with correct permissions
adb push cacert.der /sdcard/
adb shell su -c mount -o rw,remount /system
adb shell su -c cp /sdcard/cacert.der /system/etc/security/cacerts/9a5ba575.0
adb shell su -c chmod 644 /system/etc/security/cacerts/9a5ba575.0
adb shell su -c reboot
```

### Issue: Objection Cannot Attach

```bash
# Check Frida server is running
adb shell "su -c ps -A | grep frida"

# Start Frida server if not running
adb push frida-server /data/local/tmp/
adb shell "su -c chmod 755 /data/local/tmp/frida-server"
adb shell "su -c /data/local/tmp/frida-server &"

# Verify connection
frida-ps -U
```

### Issue: Proxy Not Intercepting Traffic

```bash
# Check proxy configuration
adb shell settings get global http_proxy

# Clear proxy cache
adb shell am broadcast -a android.intent.action.PROXY_CHANGE

# Verify iptables rules
adb shell su -c iptables -t nat -L -n -v

# Force traffic through proxy
adb shell su -c iptables -t nat -A OUTPUT -p tcp -m multiport --dports 80,443 -j DNAT --to-destination 127.0.0.1:8080
```

### Issue: Magisk Modules Not Loading

```bash
# Boot into safe mode
adb shell su -c magisk --remove-modules

# Check module logs
adb shell su -c cat /cache/magisk.log

# Verify Magisk version compatibility
adb shell su -c magisk -v

# Reinstall Magisk if needed
adb push Magisk.apk /sdcard/
adb shell pm install /sdcard/Magisk.apk
```

## Environment Variables

```bash
# Set in ~/.bashrc or export before running scripts

export BURP_PROXY_HOST="127.0.0.1"
export BURP_PROXY_PORT="8080"
export ANDROID_SDK_ROOT="/path/to/android-sdk"
export JAMBOREE_HOME="/path/to/Android-Mobile-Security-Sandbox-Testing"
export FRIDA_SERVER_VERSION="16.1.4"
```

## Advanced Usage

### Automated Traffic Analysis

```python
#!/usr/bin/env python3
from mitmproxy import http
import json

class JAMBOREEAnalyzer:
    def __init__(self):
        self.api_calls = []
    
    def request(self, flow: http.HTTPFlow):
        if 'api' in flow.request.pretty_host:
            self.api_calls.append({
                'url': flow.request.url,
                'method': flow.request.method,
                'headers': dict(flow.request.headers),
                'body': flow.request.text
            })
    
    def response(self, flow: http.HTTPFlow):
        if flow.response.status_code != 200:
            print(f"[!] Error response: {flow.request.url} - {flow.response.status_code}")
    
    def done(self):
        with open('api_calls.json', 'w') as f:
            json.dump(self.api_calls, f, indent=2)

addons = [JAMBOREEAnalyzer()]
```

This skill provides comprehensive coverage of JAMBOREE's capabilities for AI coding agents assisting with Android security testing workflows.
