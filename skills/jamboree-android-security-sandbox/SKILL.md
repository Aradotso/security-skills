---
name: jamboree-android-security-sandbox
description: Configure and orchestrate Android security testing environments with Magisk, Burp Suite, Objection, and rooted emulators for penetration testing.
triggers:
  - set up android security testing environment
  - configure burp suite with android emulator
  - use objection to hook android app
  - bypass android certificate pinning
  - set up magisk modules for testing
  - configure rooted android emulator for pentesting
  - intercept android app traffic with burp
  - hook android runtime with frida objection
---

# JAMBOREE Android Security Sandbox Skill

> Skill by [ara.so](https://ara.so) — Security Skills collection

## Overview

JAMBOREE (Java Android Magisk Burp Objection Root Emulator Easy) is a unified Android security testing framework that integrates:

- **Magisk**: Systemless root and module management
- **Burp Suite**: HTTPS traffic interception and analysis
- **Objection**: Frida-powered runtime instrumentation
- **Rooted Emulator**: Pre-configured Android Virtual Devices

This skill enables AI agents to help developers configure, deploy, and use JAMBOREE for Android application security assessment, penetration testing, and reverse engineering.

## Installation

### Prerequisites Verification

Before deployment, verify the environment has required dependencies:

```bash
# Check Java version (JDK 11+)
java -version

# Verify Android SDK and platform tools
adb version
avdmanager list

# Confirm Python 3 for Objection
python3 --version

# Check Burp Suite installation
which burpsuite || echo "Burp Suite not found in PATH"
```

### Core Installation Steps

1. **Clone and Initialize**
```bash
git clone https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing.git
cd Android-Mobile-Security-Sandbox-Testing

# Run environment validation
./orchestration/validators/check-prerequisites.sh
```

2. **Deploy Core Components**
```bash
# Phase 1: Environment validation
./orchestration/validators/validate-all.sh

# Phase 2: Deploy Magisk modules
./orchestration/deployers/deploy-magisk.sh

# Phase 3: Configure Burp Suite integration
./orchestration/deployers/setup-burp-proxy.sh

# Phase 4: Install Objection environment
./orchestration/deployers/install-objection.sh
```

3. **Calibrate Settings**
```bash
# Customize configuration
cp configurations/android/default.conf configurations/android/custom.conf
nano configurations/android/custom.conf

# Apply calibration
./orchestration/calibrators/apply-config.sh custom
```

## Configuration

### Android Emulator Setup

Create or modify an AVD for security testing:

```bash
# List available system images
avdmanager list targets

# Create rooted AVD (x86_64, Android 13)
avdmanager create avd \
  -n jamboree-test \
  -k "system-images;android-33;google_apis_playstore;x86_64" \
  -d "pixel_5" \
  -c 4096M

# Start emulator with writable system
emulator -avd jamboree-test -writable-system -no-snapshot-load &

# Wait for boot
adb wait-for-device
adb root
adb remount
```

### Magisk Module Configuration

Deploy core modules for testing:

```bash
# Push Magisk Manager APK
adb install modules/magisk/MagiskManager.apk

# Install BusyBox module
adb push modules/magisk/systemless/busybox.zip /sdcard/
adb shell magisk --install-module /sdcard/busybox.zip

# Install certificate pinning bypass module
adb push modules/magisk/systemless/ssl-unpinning.zip /sdcard/
adb shell magisk --install-module /sdcard/ssl-unpinning.zip

# Reboot to apply
adb reboot
adb wait-for-device
```

### Burp Suite Proxy Setup

Configure system-wide traffic interception:

```bash
# Generate and install Burp CA certificate
openssl x509 -inform DER -in burp-cert.der -out burp-cert.pem
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-cert.pem | head -1)

# Push to system certificate store
adb root
adb remount
adb push burp-cert.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0

# Configure proxy settings (Wi-Fi)
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# Or use iptables redirection
adb shell iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination ${BURP_HOST}:${BURP_PORT}
adb shell iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination ${BURP_HOST}:${BURP_PORT}
```

Configuration file example (`configurations/network/burp-proxy.conf`):

```ini
[proxy]
host = 192.168.1.100
port = 8080
upstream_proxy = 
cert_path = ./modules/burp/certificates/burp-ca.pem

[interception]
mode = transparent
dns_spoofing = true
ssl_passthrough = false

[filters]
include_hosts = *.example.com,*.target-app.io
exclude_hosts = *.google.com,*.android.com
```

### Objection Environment

Install and configure Objection with Frida:

```bash
# Install Objection via pip
pip3 install objection

# Install Frida server on device
FRIDA_VERSION=$(frida --version)
wget https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-x86_64.xz
unxz frida-server-${FRIDA_VERSION}-android-x86_64.xz

adb push frida-server-${FRIDA_VERSION}-android-x86_64 /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

## Key Commands and Workflows

### Basic Application Analysis

```bash
# List installed packages
adb shell pm list packages | grep -i target

# Get package information
adb shell dumpsys package com.example.targetapp

# Pull APK for static analysis
adb shell pm path com.example.targetapp
adb pull /data/app/~~random~~/com.example.targetapp-hash==/base.apk
```

### Objection Runtime Hooking

Launch Objection against a running app:

```bash
# Spawn app with Objection
objection -g com.example.targetapp explore

# Or attach to running process
objection -g $(adb shell pidof com.example.targetapp) explore
```

Common Objection commands in the interactive shell:

```javascript
// List activities and classes
android hooking list activities
android hooking list classes

// Search for methods
android hooking search methods encrypt
android hooking search classes database

// Hook method and print arguments
android hooking watch class_method com.example.crypto.AES.encrypt --dump-args --dump-return

// Bypass root detection
android root disable

// Bypass SSL pinning
android sslpinning disable

// Explore SQLite databases
sqlite connect /data/data/com.example.targetapp/databases/app.db
.tables
SELECT * FROM users;

// Dump SharedPreferences
android hooking list shared_preferences
android hooking get shared_preference com.example.targetapp_preferences

// Memory operations
memory dump all com.example.targetapp
memory search "password" --string
```

### Advanced Frida Scripts

Custom hooking scripts in `modules/objection/scripts/`:

**hook-crypto.js** - Monitor cryptographic operations:

```javascript
Java.perform(function() {
    var Cipher = Java.use('javax.crypto.Cipher');
    
    Cipher.doFinal.overload('[B').implementation = function(input) {
        console.log('[Cipher.doFinal] Input length: ' + input.length);
        console.log('[Cipher.doFinal] Input hex: ' + bytesToHex(input));
        
        var result = this.doFinal(input);
        
        console.log('[Cipher.doFinal] Output length: ' + result.length);
        console.log('[Cipher.doFinal] Output hex: ' + bytesToHex(result));
        
        return result;
    };
    
    function bytesToHex(bytes) {
        var hex = [];
        for (var i = 0; i < bytes.length && i < 32; i++) {
            hex.push(('0' + (bytes[i] & 0xFF).toString(16)).slice(-2));
        }
        return hex.join(' ') + (bytes.length > 32 ? '...' : '');
    }
    
    console.log('[+] Cipher.doFinal hooked');
});
```

**bypass-root-check.js** - Defeat root detection:

```javascript
Java.perform(function() {
    var RootDetection = Java.use('com.example.security.RootDetection');
    
    RootDetection.isRooted.implementation = function() {
        console.log('[Root Detection] Bypass triggered');
        return false;
    };
    
    RootDetection.checkSuperUser.implementation = function() {
        console.log('[Root Detection] SuperUser check bypassed');
        return false;
    };
    
    RootDetection.checkBuildTags.implementation = function() {
        console.log('[Root Detection] Build tags check bypassed');
        return false;
    };
    
    console.log('[+] Root detection bypassed');
});
```

Load custom scripts:

```bash
# Via Objection
objection -g com.example.targetapp explore -s modules/objection/scripts/hook-crypto.js

# Via Frida directly
frida -U -l modules/objection/scripts/bypass-root-check.js -f com.example.targetapp
```

### Burp Suite Automation

Extend Burp with Python extensions in `modules/burp/extensions/`:

**auto-scanner.py** - Automated endpoint discovery:

```python
from burp import IBurpExtender, IHttpListener, ITab
from javax.swing import JPanel, JTextArea, JScrollPane
import json

class BurpExtender(IBurpExtender, IHttpListener, ITab):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.setExtensionName("JAMBOREE Auto Scanner")
        
        self.endpoints = set()
        self.setup_ui()
        
        callbacks.registerHttpListener(self)
        
    def processHttpMessage(self, toolFlag, messageIsRequest, messageInfo):
        if not messageIsRequest:
            return
            
        request = messageInfo.getRequest()
        analyzedRequest = self._helpers.analyzeRequest(messageInfo)
        
        url = analyzedRequest.getUrl()
        endpoint = f"{url.getProtocol()}://{url.getHost()}{url.getPath()}"
        
        if endpoint not in self.endpoints:
            self.endpoints.add(endpoint)
            self.update_display()
            
    def setup_ui(self):
        self.panel = JPanel()
        self.textarea = JTextArea(20, 80)
        self.textarea.setEditable(False)
        scrollPane = JScrollPane(self.textarea)
        self.panel.add(scrollPane)
        
        self._callbacks.addSuiteTab(self)
        
    def update_display(self):
        output = "\n".join(sorted(self.endpoints))
        self.textarea.setText(f"Discovered Endpoints ({len(self.endpoints)}):\n\n{output}")
        
    def getTabCaption(self):
        return "JAMBOREE Scanner"
        
    def getUiComponent(self):
        return self.panel
```

### Traffic Analysis Workflow

Complete workflow for intercepting and analyzing app traffic:

```bash
# 1. Start Frida server
adb shell "/data/local/tmp/frida-server &"

# 2. Launch Burp Suite
burpsuite --proxy-port=${BURP_PORT} &

# 3. Configure device proxy
adb shell settings put global http_proxy ${BURP_HOST}:${BURP_PORT}

# 4. Disable SSL pinning with Objection
objection -g com.example.targetapp explore <<EOF
android sslpinning disable
exit
EOF

# 5. Launch app
adb shell am start -n com.example.targetapp/.MainActivity

# 6. Monitor traffic in Burp and extract patterns
# Burp Suite captures all HTTP/HTTPS traffic automatically

# 7. Export findings
# From Burp: Proxy > HTTP History > Save items
```

## Common Patterns

### Pattern 1: API Endpoint Discovery and Analysis

```bash
#!/bin/bash
# discover-apis.sh

PACKAGE=$1
DURATION=${2:-300}  # 5 minutes default

echo "[+] Starting API discovery for ${PACKAGE}"

# Start network capture
adb shell tcpdump -i any -w /sdcard/capture.pcap &
TCPDUMP_PID=$!

# Launch app with Objection hooks
objection -g ${PACKAGE} explore <<EOF
android hooking watch class_method okhttp3.OkHttpClient.newCall --dump-args --dump-return
android hooking watch class_method retrofit2.Retrofit.create --dump-args
exit
EOF &

# Wait for specified duration
sleep ${DURATION}

# Stop capture
kill ${TCPDUMP_PID}
adb pull /sdcard/capture.pcap ./analysis/

# Parse with tshark
tshark -r ./analysis/capture.pcap -Y "http || tls" -T fields \
  -e http.request.full_uri \
  -e tls.handshake.extensions_server_name | sort -u > ./analysis/endpoints.txt

echo "[+] Discovered $(wc -l < ./analysis/endpoints.txt) unique endpoints"
cat ./analysis/endpoints.txt
```

### Pattern 2: Database Extraction and Analysis

```bash
#!/bin/bash
# extract-databases.sh

PACKAGE=$1
OUTPUT_DIR="./databases/${PACKAGE}"

mkdir -p ${OUTPUT_DIR}

# Find all SQLite databases
adb shell "run-as ${PACKAGE} find /data/data/${PACKAGE} -name '*.db'" | while read DB_PATH; do
    DB_NAME=$(basename ${DB_PATH})
    
    echo "[+] Extracting ${DB_NAME}"
    
    # Copy to accessible location
    adb shell "run-as ${PACKAGE} cp ${DB_PATH} /sdcard/${DB_NAME}"
    adb pull /sdcard/${DB_NAME} ${OUTPUT_DIR}/
    
    # Analyze schema
    sqlite3 ${OUTPUT_DIR}/${DB_NAME} ".schema" > ${OUTPUT_DIR}/${DB_NAME}.schema.sql
    
    # Dump data
    sqlite3 ${OUTPUT_DIR}/${DB_NAME} ".dump" > ${OUTPUT_DIR}/${DB_NAME}.dump.sql
    
    echo "[+] ${DB_NAME}: $(sqlite3 ${OUTPUT_DIR}/${DB_NAME} 'SELECT COUNT(*) FROM sqlite_master WHERE type=\"table\"') tables"
done
```

### Pattern 3: Automated Certificate Pinning Bypass

Create a persistent bypass module:

```javascript
// modules/objection/scripts/persistent-ssl-bypass.js

Java.perform(function() {
    console.log('[+] Loading persistent SSL bypass');
    
    // OkHttp3 CertificatePinner
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
            console.log('[SSL] OkHttp3 pinning bypassed for: ' + arguments[0]);
        };
    } catch(e) {}
    
    // TrustManager bypass
    try {
        var X509TrustManager = Java.use('javax.net.ssl.X509TrustManager');
        var SSLContext = Java.use('javax.net.ssl.SSLContext');
        
        var TrustManager = Java.registerClass({
            name: 'com.jamboree.TrustManager',
            implements: [X509TrustManager],
            methods: {
                checkClientTrusted: function(chain, authType) {},
                checkServerTrusted: function(chain, authType) {},
                getAcceptedIssuers: function() { return []; }
            }
        });
        
        var TrustManagers = [TrustManager.$new()];
        var SSLContext_init = SSLContext.init.overload(
            '[Ljavax.net.ssl.KeyManager;',
            '[Ljavax.net.ssl.TrustManager;',
            'java.security.SecureRandom'
        );
        
        SSLContext_init.implementation = function(keyManager, trustManager, secureRandom) {
            console.log('[SSL] SSLContext.init hooked');
            SSLContext_init.call(this, keyManager, TrustManagers, secureRandom);
        };
    } catch(e) {}
    
    // Universal Android SSL bypass
    try {
        var SSLContext = Java.use('javax.net.ssl.SSLContext');
        var TrustManager = Java.registerClass({
            name: 'com.jamboree.UniversalTrust',
            implements: [Java.use('javax.net.ssl.X509TrustManager')],
            methods: {
                checkClientTrusted: function() {},
                checkServerTrusted: function() {},
                getAcceptedIssuers: function() { return []; }
            }
        });
        
        SSLContext.getDefault.implementation = function() {
            var context = SSLContext.getInstance('TLS');
            context.init(null, [TrustManager.$new()], null);
            return context;
        };
    } catch(e) {}
    
    console.log('[+] SSL bypass hooks installed');
});
```

Deploy automatically on app launch:

```bash
# Add to ~/.frida/scripts/
mkdir -p ~/.frida/scripts
cp modules/objection/scripts/persistent-ssl-bypass.js ~/.frida/scripts/

# Create wrapper
cat > ~/bin/jamboree-launch <<'EOF'
#!/bin/bash
PACKAGE=$1
frida -U -l ~/.frida/scripts/persistent-ssl-bypass.js -f ${PACKAGE} --no-pause
EOF

chmod +x ~/bin/jamboree-launch
```

## Troubleshooting

### Issue: Certificate Not Trusted

**Symptoms**: HTTPS traffic not captured, SSL errors in app

**Solution**:
```bash
# Verify certificate installation
adb shell ls -la /system/etc/security/cacerts/ | grep -i burp

# Check certificate hash matches filename
openssl x509 -inform PEM -subject_hash_old -in burp-cert.pem | head -1

# Force reinstall
adb root && adb remount
CERT_HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-cert.pem | head -1)
adb push burp-cert.pem /system/etc/security/cacerts/${CERT_HASH}.0
adb shell chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0
adb reboot
```

### Issue: Frida Connection Failed

**Symptoms**: `Failed to spawn: unable to find process with name 'com.example.app'`

**Solution**:
```bash
# Check Frida server is running
adb shell "ps -A | grep frida"

# Restart Frida server
adb shell "killall frida-server"
adb shell "/data/local/tmp/frida-server &"

# Verify Frida version compatibility
frida --version
adb shell "/data/local/tmp/frida-server --version"

# If versions mismatch, download matching server version
```

### Issue: Magisk Module Not Loading

**Symptoms**: Module shows as installed but features not working

**Solution**:
```bash
# Check module status
adb shell magisk --list

# View Magisk logs
adb shell cat /cache/magisk.log

# Reinstall in recovery mode
adb reboot recovery
# Use volume keys to navigate to "Install" > Select module ZIP

# Check for conflicts
adb shell magisk --remove-module conflicting-module
adb reboot
```

### Issue: Objection Hooks Not Triggering

**Symptoms**: Hooks installed but no output when methods called

**Solution**:
```javascript
// Verify class/method exists first
Java.perform(function() {
    // List all classes matching pattern
    Java.enumerateLoadedClasses({
        onMatch: function(className) {
            if (className.indexOf('example') !== -1) {
                console.log('[+] Found: ' + className);
                
                // List methods
                var clazz = Java.use(className);
                var methods = clazz.class.getDeclaredMethods();
                methods.forEach(function(method) {
                    console.log('  -> ' + method);
                });
            }
        },
        onComplete: function() {}
    });
});
```

### Issue: Emulator Detection

**Symptoms**: App exits or shows warning about running on emulator

**Solution**:
```bash
# Modify build.prop to mimic real device
adb root && adb remount
adb shell "cat >> /system/build.prop <<EOF
ro.product.manufacturer=samsung
ro.product.model=SM-G991B
ro.product.brand=samsung
ro.build.fingerprint=samsung/o1sxxx/o1s:13/TP1A.220624.014/G991BXXU5DVLB:user/release-keys
EOF"

# Hide Magisk from detection
adb shell magisk --denylist add com.example.targetapp

# Use Shamiko module for advanced hiding
adb push modules/magisk/systemless/shamiko.zip /sdcard/
adb shell magisk --install-module /sdcard/shamiko.zip

adb reboot
```

## Environment Variables

JAMBOREE uses these environment variables:

```bash
# Burp Suite configuration
export BURP_HOST="192.168.1.100"
export BURP_PORT="8080"
export BURP_CERT_PATH="./modules/burp/certificates/burp-ca.pem"

# Android configuration
export ANDROID_SDK_ROOT="${HOME}/Android/Sdk"
export ANDROID_AVD_HOME="${HOME}/.android/avd"

# Frida/Objection
export FRIDA_SERVER_PATH="/data/local/tmp/frida-server"

# Logging
export JAMBOREE_LOG_LEVEL="DEBUG"  # DEBUG, INFO, WARN, ERROR
export JAMBOREE_LOG_PATH="./logs/jamboree.log"
```

## Best Practices

1. **Always work in isolated environments** - Use dedicated emulators or test devices
2. **Document findings** - Keep detailed logs of hooks, modified traffic, and discovered vulnerabilities
3. **Version control configurations** - Track changes to Frida scripts and Burp extensions
4. **Regular updates** - Keep Magisk, Frida, and Objection current for compatibility
5. **Test incrementally** - Apply one bypass at a time to identify what works
6. **Backup emulator snapshots** - Save working configurations before major changes

## Additional Resources

- Project repository: https://github.com/hero-mike/Android-Mobile-Security-Sandbox-Testing
- Frida documentation: https://frida.re/docs/
- Objection guide: https://github.com/sensepost/objection
- Magisk modules: https://github.com/Magisk-Modules-Repo
