---
name: experimentersoftroll422-windows-filesystem-security
description: Windows filesystem security monitoring, access control, and encryption workflows using EaseFilter SDK with Rust bindings
triggers:
  - how do I monitor file access on Windows
  - set up filesystem security policies
  - configure EaseFilter SDK with Rust
  - implement Windows file encryption workflow
  - create file access control rules
  - monitor filesystem events in Windows
  - integrate EaseFilter file security
  - set up Windows file activity monitoring
---

# experimentersoftroll422-windows-filesystem-security

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

Experimentersoftroll422 is a Windows-centered filesystem security project that provides file activity monitoring, access policy enforcement, and encryption workflow support through Rust bindings to the EaseFilter File Security SDK. It's designed for developers building Windows file policy solutions with security-focused file management capabilities.

**Key capabilities:**
- Real-time file activity observation and logging
- Policy-based access control enforcement
- Filesystem encryption workflow integration
- Windows-native deployment focus
- Rust bindings for type-safe integration
- EaseFilter File Security SDK foundation

## Installation

### Prerequisites

Ensure you have:
- Windows 10/11 or Windows Server 2019+
- Rust toolchain (1.70+): `rustup install stable`
- EaseFilter File Security SDK (license required)
- Administrator privileges for driver installation

### Clone and Setup

```bash
git clone https://github.com/tomw286/experimentersoftroll422-security-loader.git
cd experimentersoftroll422-security-loader
```

### Rust Dependencies

Add to your `Cargo.toml`:

```toml
[dependencies]
windows = { version = "0.52", features = ["Win32_Storage_FileSystem", "Win32_Security"] }
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.35", features = ["full"] }
```

### SDK Integration

Place the EaseFilter SDK files in the expected location:

```
project_root/
├── sdk/
│   ├── EaseFilter.dll
│   ├── EaseFilter.lib
│   └── FilterDriver.sys
└── src/
    └── main.rs
```

## Configuration

### Filesystem Security Settings

Create `config.toml` in the project root:

```toml
[filesystem]
monitor_files = true
control_access = true
encryption = true
log_level = "info"

[monitoring]
# Paths to monitor (supports wildcards)
watch_paths = [
    "C:\\Users\\*\\Documents",
    "C:\\ProgramData\\Sensitive\\*"
]

# File operations to track
operations = ["read", "write", "delete", "rename", "create"]

[access_control]
# Default policy: "allow" or "deny"
default_policy = "allow"

# Denied extensions
blocked_extensions = [".exe", ".dll", ".sys"]

[encryption]
# Enable automatic encryption for specific paths
auto_encrypt_paths = ["C:\\Secure\\*"]
algorithm = "AES256"
key_storage = "windows_credential_manager"

[integration]
runtime = "rust"
sdk = "EaseFilter File Security SDK"
sdk_path = "./sdk"

[logging]
output_path = "C:\\ProgramData\\ExperimenterSoftroll\\logs"
max_log_size_mb = 100
rotation_count = 5
```

### Environment Variables

Set required environment variables:

```bash
# Windows Command Prompt
set EASEFILTER_SDK_PATH=C:\path\to\sdk
set EASEFILTER_LICENSE_KEY=%YOUR_LICENSE_KEY%
set SECURITY_LOG_PATH=C:\ProgramData\ExperimenterSoftroll\logs

# PowerShell
$env:EASEFILTER_SDK_PATH="C:\path\to\sdk"
$env:EASEFILTER_LICENSE_KEY=$env:YOUR_LICENSE_KEY
$env:SECURITY_LOG_PATH="C:\ProgramData\ExperimenterSoftroll\logs"
```

## Core Rust Integration Patterns

### Initialize File Monitor

```rust
use std::env;
use std::path::PathBuf;
use serde::Deserialize;

#[derive(Deserialize)]
struct Config {
    filesystem: FilesystemConfig,
    monitoring: MonitoringConfig,
}

#[derive(Deserialize)]
struct FilesystemConfig {
    monitor_files: bool,
    control_access: bool,
    encryption: bool,
}

#[derive(Deserialize)]
struct MonitoringConfig {
    watch_paths: Vec<String>,
    operations: Vec<String>,
}

fn load_config() -> Result<Config, Box<dyn std::error::Error>> {
    let config_path = PathBuf::from("config.toml");
    let contents = std::fs::read_to_string(config_path)?;
    let config: Config = toml::from_str(&contents)?;
    Ok(config)
}

fn initialize_monitor() -> Result<(), Box<dyn std::error::Error>> {
    let config = load_config()?;
    let sdk_path = env::var("EASEFILTER_SDK_PATH")?;
    let license_key = env::var("EASEFILTER_LICENSE_KEY")?;
    
    println!("Initializing EaseFilter SDK from: {}", sdk_path);
    
    // Load EaseFilter SDK (pseudo-code, adapt to actual SDK API)
    unsafe {
        let sdk = load_easefilter_sdk(&sdk_path)?;
        sdk.initialize(&license_key)?;
        
        for path in &config.monitoring.watch_paths {
            sdk.add_monitor_path(path)?;
            println!("Monitoring: {}", path);
        }
    }
    
    Ok(())
}
```

### File Activity Callback Handler

```rust
use std::sync::mpsc::{channel, Sender, Receiver};
use std::thread;

#[derive(Debug, Clone)]
struct FileEvent {
    operation: String,
    path: String,
    process_id: u32,
    timestamp: std::time::SystemTime,
    user: String,
}

fn setup_file_event_handler() -> Receiver<FileEvent> {
    let (tx, rx) = channel();
    
    thread::spawn(move || {
        // Register callback with EaseFilter SDK
        register_callback(tx);
    });
    
    rx
}

fn register_callback(tx: Sender<FileEvent>) {
    // Pseudo-code for SDK callback registration
    unsafe {
        set_file_event_callback(Box::new(move |operation, path, pid, user| {
            let event = FileEvent {
                operation: operation.to_string(),
                path: path.to_string(),
                process_id: pid,
                timestamp: std::time::SystemTime::now(),
                user: user.to_string(),
            };
            
            let _ = tx.send(event);
        }));
    }
}

fn process_events(rx: Receiver<FileEvent>) {
    for event in rx {
        println!("[{}] {} on {} by {} (PID: {})",
            event.timestamp.elapsed().unwrap_or_default().as_secs(),
            event.operation,
            event.path,
            event.user,
            event.process_id
        );
        
        // Apply access control logic
        if should_block_event(&event) {
            block_file_operation(&event);
        }
    }
}
```

### Access Control Policy Engine

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
enum AccessPolicy {
    Allow,
    Deny,
    Audit,
}

struct PolicyEngine {
    rules: HashMap<String, AccessPolicy>,
    blocked_extensions: Vec<String>,
    default_policy: AccessPolicy,
}

impl PolicyEngine {
    fn new(config: &Config) -> Self {
        let mut rules = HashMap::new();
        
        // Load rules from config
        for path in &config.monitoring.watch_paths {
            rules.insert(path.clone(), AccessPolicy::Audit);
        }
        
        Self {
            rules,
            blocked_extensions: vec![".exe".to_string(), ".dll".to_string()],
            default_policy: AccessPolicy::Allow,
        }
    }
    
    fn evaluate(&self, event: &FileEvent) -> AccessPolicy {
        // Check extension blacklist
        if self.is_blocked_extension(&event.path) {
            return AccessPolicy::Deny;
        }
        
        // Check path-specific rules
        for (pattern, policy) in &self.rules {
            if self.path_matches(pattern, &event.path) {
                return policy.clone();
            }
        }
        
        self.default_policy.clone()
    }
    
    fn is_blocked_extension(&self, path: &str) -> bool {
        self.blocked_extensions.iter()
            .any(|ext| path.to_lowercase().ends_with(ext))
    }
    
    fn path_matches(&self, pattern: &str, path: &str) -> bool {
        // Simple wildcard matching (extend as needed)
        if pattern.ends_with("*") {
            let prefix = &pattern[..pattern.len() - 1];
            path.starts_with(prefix)
        } else {
            pattern == path
        }
    }
}

fn should_block_event(event: &FileEvent) -> bool {
    let config = load_config().unwrap();
    let engine = PolicyEngine::new(&config);
    
    match engine.evaluate(event) {
        AccessPolicy::Deny => true,
        _ => false,
    }
}
```

### Encryption Workflow Integration

```rust
use std::fs::{File, OpenOptions};
use std::io::{Read, Write};

struct EncryptionService {
    auto_encrypt_paths: Vec<String>,
    algorithm: String,
}

impl EncryptionService {
    fn from_config(config: &Config) -> Self {
        Self {
            auto_encrypt_paths: config.encryption.auto_encrypt_paths.clone(),
            algorithm: config.encryption.algorithm.clone(),
        }
    }
    
    fn should_auto_encrypt(&self, path: &str) -> bool {
        self.auto_encrypt_paths.iter()
            .any(|pattern| path.starts_with(pattern.trim_end_matches('*')))
    }
    
    fn encrypt_file(&self, path: &str) -> Result<(), Box<dyn std::error::Error>> {
        // Retrieve encryption key from Windows Credential Manager
        let key = self.get_encryption_key()?;
        
        let mut file = File::open(path)?;
        let mut contents = Vec::new();
        file.read_to_end(&mut contents)?;
        
        // Encrypt using EaseFilter SDK or external crypto library
        let encrypted = self.perform_encryption(&contents, &key)?;
        
        let mut output = OpenOptions::new()
            .write(true)
            .truncate(true)
            .open(path)?;
        output.write_all(&encrypted)?;
        
        println!("Encrypted: {}", path);
        Ok(())
    }
    
    fn get_encryption_key(&self) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        // Windows Credential Manager integration
        // Use windows-rs crate for actual implementation
        let key = std::env::var("ENCRYPTION_KEY_BASE64")
            .map(|k| base64::decode(k).unwrap())?;
        Ok(key)
    }
    
    fn perform_encryption(&self, data: &[u8], key: &[u8]) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        // Delegate to EaseFilter SDK encryption API or use crypto crate
        // This is a placeholder for the actual encryption call
        Ok(data.to_vec())
    }
}

fn handle_file_creation(event: &FileEvent, encryption: &EncryptionService) {
    if encryption.should_auto_encrypt(&event.path) {
        if let Err(e) = encryption.encrypt_file(&event.path) {
            eprintln!("Encryption failed for {}: {}", event.path, e);
        }
    }
}
```

### Complete Application Example

```rust
use tokio::runtime::Runtime;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load configuration
    let config = load_config()?;
    
    // Initialize SDK
    initialize_monitor()?;
    
    // Set up policy engine
    let policy_engine = PolicyEngine::new(&config);
    
    // Set up encryption service
    let encryption_service = EncryptionService::from_config(&config);
    
    // Start event processing
    let event_rx = setup_file_event_handler();
    
    println!("File security monitor started. Press Ctrl+C to exit.");
    
    // Process events
    let rt = Runtime::new()?;
    rt.block_on(async {
        for event in event_rx {
            // Log event
            log_event(&event);
            
            // Evaluate policy
            let decision = policy_engine.evaluate(&event);
            match decision {
                AccessPolicy::Deny => {
                    println!("BLOCKED: {:?}", event);
                    block_file_operation(&event);
                }
                AccessPolicy::Audit => {
                    println!("AUDITED: {:?}", event);
                    audit_log(&event);
                }
                AccessPolicy::Allow => {
                    // Check for auto-encryption on create
                    if event.operation == "create" {
                        handle_file_creation(&event, &encryption_service);
                    }
                }
            }
        }
    });
    
    Ok(())
}

fn log_event(event: &FileEvent) {
    let log_path = std::env::var("SECURITY_LOG_PATH")
        .unwrap_or_else(|_| "C:\\ProgramData\\ExperimenterSoftroll\\logs".to_string());
    
    // Append to log file
    let log_entry = format!("{:?}\n", event);
    let _ = std::fs::write(format!("{}\\events.log", log_path), log_entry);
}

fn block_file_operation(event: &FileEvent) {
    // Call EaseFilter SDK to block the operation
    unsafe {
        // SDK-specific blocking call
        println!("Blocking operation on: {}", event.path);
    }
}

fn audit_log(event: &FileEvent) {
    // Write to audit log
    println!("AUDIT: {:?}", event);
}
```

## CLI Commands

If the project includes a CLI tool, use these commands:

```bash
# Start monitoring service
experimentersoftroll422-monitor start --config config.toml

# Stop monitoring service
experimentersoftroll422-monitor stop

# View live events
experimentersoftroll422-monitor watch --filter "*.doc"

# Apply policy from file
experimentersoftroll422-policy apply --file security-policy.json

# Encrypt directory
experimentersoftroll422-encrypt --path "C:\Secure" --recursive

# Check driver status
experimentersoftroll422-monitor status

# Generate audit report
experimentersoftroll422-audit report --start "2026-01-01" --end "2026-12-31"
```

## Common Troubleshooting

### Driver Not Loaded

**Symptom:** "EaseFilter driver not found" error

**Solution:**
```bash
# Install driver (requires admin)
sc create EaseFilter binPath= "C:\path\to\FilterDriver.sys" type= kernel
sc start EaseFilter

# Verify driver status
sc query EaseFilter
```

### Access Denied Errors

**Symptom:** "Access denied" when monitoring system directories

**Solution:**
- Run as Administrator
- Check Windows security policies: `gpedit.msc` → Computer Configuration → Windows Settings → Security Settings
- Verify process has `SeSecurityPrivilege`

### Events Not Captured

**Symptom:** No file events appear in logs

**Solution:**
```rust
// Verify paths are correctly formatted (Windows-style)
let path = "C:\\Users\\Documents"; // Correct
// NOT: "C:/Users/Documents"

// Check path wildcards
for path in &config.monitoring.watch_paths {
    println!("Monitoring pattern: {}", path);
    // Ensure patterns match actual filesystem structure
}
```

### High CPU Usage

**Symptom:** Monitor process consuming excessive CPU

**Solution:**
```toml
[monitoring]
# Reduce monitoring scope
watch_paths = ["C:\\Sensitive\\*"]  # Not "C:\\*"

# Filter operations
operations = ["write", "delete"]  # Ignore reads

[performance]
batch_events = true
batch_size = 100
flush_interval_ms = 1000
```

### License Validation Failure

**Symptom:** "Invalid license key" error

**Solution:**
```bash
# Verify environment variable is set
echo %EASEFILTER_LICENSE_KEY%

# Check key format (no extra whitespace)
set EASEFILTER_LICENSE_KEY=YOUR-ACTUAL-KEY-HERE

# Restart application after setting variable
```

### Encryption Key Not Found

**Symptom:** "Encryption key not available" error

**Solution:**
```rust
// Store key in Windows Credential Manager
use windows::Win32::Security::Credentials::*;

fn store_encryption_key(key: &[u8]) -> Result<(), Box<dyn std::error::Error>> {
    let key_b64 = base64::encode(key);
    // Use CredWrite API to store in Windows Credential Manager
    // Or set environment variable as fallback
    std::env::set_var("ENCRYPTION_KEY_BASE64", key_b64);
    Ok(())
}
```

## Best Practices

1. **Always run with appropriate privileges** - File system drivers require administrator rights
2. **Test policies in audit mode first** - Use `AccessPolicy::Audit` before enforcing denials
3. **Implement rate limiting** - Prevent log flooding during high-activity periods
4. **Use Windows Credential Manager** - Never hardcode encryption keys in source
5. **Monitor driver health** - Implement watchdog for SDK/driver crashes
6. **Rotate logs regularly** - Prevent disk space exhaustion from audit logs

## Additional Resources

- EaseFilter SDK Documentation: Refer to vendor-provided materials
- Windows Driver Development: [Microsoft Docs](https://docs.microsoft.com/windows-hardware/drivers/)
- Rust Windows Bindings: [windows-rs crate](https://github.com/microsoft/windows-rs)
