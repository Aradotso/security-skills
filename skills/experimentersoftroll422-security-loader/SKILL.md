---
name: experimentersoftroll422-security-loader
description: Windows filesystem security monitoring, access control, and encryption workflow integration using EaseFilter SDK with Rust bindings
triggers:
  - how do I monitor file activity on Windows with experimentersoftroll422
  - set up file access policies with experimentersoftroll422 security loader
  - integrate EaseFilter SDK with Rust bindings for filesystem security
  - configure encryption workflows in experimentersoftroll422
  - implement Windows file monitoring and access control
  - troubleshoot experimentersoftroll422 filesystem security issues
  - use experimentersoftroll422 for file policy enforcement
  - create Windows filesystem security rules with experimentersoftroll422
---

# experimentersoftroll422-security-loader

> Skill by [ara.so](https://ara.so) — Security Skills collection.

## Overview

Experimentersoftroll422 Security Loader is a Windows filesystem security project that provides file activity monitoring, access policy enforcement, and encryption workflow support through Rust bindings to the EaseFilter File Security SDK. It enables developers to build custom filesystem security solutions for Windows environments with policy-based file access control.

## Installation

### Prerequisites

- Windows operating system
- Rust toolchain (rustc, cargo)
- EaseFilter File Security SDK
- Administrator privileges for filesystem monitoring operations

### Clone and Setup

```bash
git clone https://github.com/tomw286/experimentersoftroll422-security-loader.git
cd experimentersoftroll422-security-loader
```

### Rust Dependencies

Add to your `Cargo.toml`:

```toml
[dependencies]
easefilter-sdk = "0.1"  # Adjust version based on SDK binding
winapi = { version = "0.3", features = ["fileapi", "winnt", "handleapi"] }
serde = { version = "1.0", features = ["derive"] }
toml = "0.5"
log = "0.4"
env_logger = "0.9"
```

## Configuration

### Basic Configuration File

Create `config.toml` in your project root:

```toml
[filesystem]
monitor_files = true
control_access = true
encryption = true
log_level = "info"

[monitoring]
watched_paths = [
    "C:\\Users\\*\\Documents",
    "C:\\ProgramData\\Sensitive"
]
excluded_extensions = [".tmp", ".log"]
real_time = true

[access_control]
enabled = true
default_policy = "deny"
audit_mode = false

[encryption]
enabled = true
algorithm = "AES256"
key_storage = "registry"

[integration]
runtime = "rust"
sdk = "EaseFilter File Security SDK"
sdk_path = "C:\\Program Files\\EaseFilter"
```

### Loading Configuration

```rust
use serde::Deserialize;
use std::fs;

#[derive(Debug, Deserialize)]
struct FilesystemConfig {
    monitor_files: bool,
    control_access: bool,
    encryption: bool,
    log_level: String,
}

#[derive(Debug, Deserialize)]
struct MonitoringConfig {
    watched_paths: Vec<String>,
    excluded_extensions: Vec<String>,
    real_time: bool,
}

#[derive(Debug, Deserialize)]
struct AccessControlConfig {
    enabled: bool,
    default_policy: String,
    audit_mode: bool,
}

#[derive(Debug, Deserialize)]
struct Config {
    filesystem: FilesystemConfig,
    monitoring: MonitoringConfig,
    access_control: AccessControlConfig,
}

fn load_config(path: &str) -> Result<Config, Box<dyn std::error::Error>> {
    let contents = fs::read_to_string(path)?;
    let config: Config = toml::from_str(&contents)?;
    Ok(config)
}
```

## Core API Usage

### Initializing the Security Loader

```rust
use std::env;
use log::{info, error};

struct SecurityLoader {
    config: Config,
    sdk_handle: Option<SDKHandle>,
}

impl SecurityLoader {
    fn new(config_path: &str) -> Result<Self, Box<dyn std::error::Error>> {
        env_logger::init();
        let config = load_config(config_path)?;
        
        info!("Initializing Security Loader with config: {:?}", config);
        
        Ok(SecurityLoader {
            config,
            sdk_handle: None,
        })
    }
    
    fn initialize_sdk(&mut self) -> Result<(), Box<dyn std::error::Error>> {
        let sdk_path = env::var("EASEFILTER_SDK_PATH")
            .unwrap_or_else(|_| "C:\\Program Files\\EaseFilter".to_string());
        
        info!("Loading EaseFilter SDK from: {}", sdk_path);
        
        // Initialize SDK connection
        let handle = easefilter_sdk::initialize(&sdk_path)?;
        self.sdk_handle = Some(handle);
        
        info!("SDK initialized successfully");
        Ok(())
    }
}
```

### File Monitoring Setup

```rust
use std::path::PathBuf;

struct FileMonitor {
    watched_paths: Vec<PathBuf>,
    callback: Box<dyn Fn(FileEvent) + Send>,
}

#[derive(Debug, Clone)]
enum FileEvent {
    Created(PathBuf),
    Modified(PathBuf),
    Deleted(PathBuf),
    Accessed(PathBuf),
}

impl FileMonitor {
    fn new(paths: Vec<String>) -> Self {
        let watched_paths = paths.into_iter()
            .map(PathBuf::from)
            .collect();
        
        FileMonitor {
            watched_paths,
            callback: Box::new(|event| {
                info!("File event detected: {:?}", event);
            }),
        }
    }
    
    fn start(&self) -> Result<(), Box<dyn std::error::Error>> {
        info!("Starting file monitoring on {} paths", self.watched_paths.len());
        
        for path in &self.watched_paths {
            self.register_path(path)?;
        }
        
        Ok(())
    }
    
    fn register_path(&self, path: &PathBuf) -> Result<(), Box<dyn std::error::Error>> {
        info!("Registering monitor for: {:?}", path);
        
        // Register with EaseFilter SDK
        easefilter_sdk::register_monitor(
            path.to_str().unwrap(),
            FileEventMask::ALL,
        )?;
        
        Ok(())
    }
    
    fn set_callback<F>(&mut self, callback: F)
    where
        F: Fn(FileEvent) + Send + 'static,
    {
        self.callback = Box::new(callback);
    }
}
```

### Access Control Policy Engine

```rust
use std::collections::HashMap;

#[derive(Debug, Clone, PartialEq)]
enum AccessDecision {
    Allow,
    Deny,
    AuditAndAllow,
    AuditAndDeny,
}

struct AccessPolicy {
    rules: Vec<PolicyRule>,
    default: AccessDecision,
}

#[derive(Debug, Clone)]
struct PolicyRule {
    path_pattern: String,
    allowed_operations: Vec<FileOperation>,
    denied_operations: Vec<FileOperation>,
    priority: u32,
}

#[derive(Debug, Clone, PartialEq)]
enum FileOperation {
    Read,
    Write,
    Delete,
    Execute,
    Rename,
}

impl AccessPolicy {
    fn new(default: AccessDecision) -> Self {
        AccessPolicy {
            rules: Vec::new(),
            default,
        }
    }
    
    fn add_rule(&mut self, rule: PolicyRule) {
        self.rules.push(rule);
        self.rules.sort_by(|a, b| b.priority.cmp(&a.priority));
    }
    
    fn evaluate(&self, path: &str, operation: &FileOperation) -> AccessDecision {
        for rule in &self.rules {
            if self.path_matches(&rule.path_pattern, path) {
                if rule.denied_operations.contains(operation) {
                    return AccessDecision::Deny;
                }
                if rule.allowed_operations.contains(operation) {
                    return AccessDecision::Allow;
                }
            }
        }
        
        self.default.clone()
    }
    
    fn path_matches(&self, pattern: &str, path: &str) -> bool {
        // Simplified wildcard matching
        if pattern.contains('*') {
            let pattern_parts: Vec<&str> = pattern.split('*').collect();
            let mut pos = 0;
            
            for (i, part) in pattern_parts.iter().enumerate() {
                if i == 0 && !part.is_empty() && !path.starts_with(part) {
                    return false;
                }
                if let Some(found_pos) = path[pos..].find(part) {
                    pos += found_pos + part.len();
                } else {
                    return false;
                }
            }
            true
        } else {
            path == pattern
        }
    }
}
```

### Encryption Workflow Integration

```rust
use std::io::{Read, Write};
use std::fs::File;

struct EncryptionManager {
    algorithm: String,
    key: Vec<u8>,
}

impl EncryptionManager {
    fn new() -> Result<Self, Box<dyn std::error::Error>> {
        let key = Self::load_key_from_env()?;
        
        Ok(EncryptionManager {
            algorithm: "AES256".to_string(),
            key,
        })
    }
    
    fn load_key_from_env() -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        let key_b64 = env::var("ENCRYPTION_KEY")?;
        let key = base64::decode(key_b64)?;
        Ok(key)
    }
    
    fn encrypt_file(&self, path: &str) -> Result<(), Box<dyn std::error::Error>> {
        info!("Encrypting file: {}", path);
        
        let mut file = File::open(path)?;
        let mut contents = Vec::new();
        file.read_to_end(&mut contents)?;
        
        let encrypted = self.encrypt_data(&contents)?;
        
        let mut output = File::create(format!("{}.encrypted", path))?;
        output.write_all(&encrypted)?;
        
        Ok(())
    }
    
    fn decrypt_file(&self, path: &str) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        info!("Decrypting file: {}", path);
        
        let mut file = File::open(path)?;
        let mut contents = Vec::new();
        file.read_to_end(&mut contents)?;
        
        self.decrypt_data(&contents)
    }
    
    fn encrypt_data(&self, data: &[u8]) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        // Integration with encryption SDK
        easefilter_sdk::encrypt(data, &self.key, &self.algorithm)
    }
    
    fn decrypt_data(&self, data: &[u8]) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        easefilter_sdk::decrypt(data, &self.key, &self.algorithm)
    }
}
```

## Complete Working Example

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load configuration
    let config = load_config("config.toml")?;
    
    // Initialize security loader
    let mut loader = SecurityLoader::new("config.toml")?;
    loader.initialize_sdk()?;
    
    // Set up file monitoring
    let monitor = Arc::new(Mutex::new(
        FileMonitor::new(config.monitoring.watched_paths.clone())
    ));
    
    let monitor_clone = monitor.clone();
    monitor.lock().unwrap().set_callback(move |event| {
        match event {
            FileEvent::Created(path) => {
                info!("New file created: {:?}", path);
            }
            FileEvent::Modified(path) => {
                info!("File modified: {:?}", path);
            }
            FileEvent::Accessed(path) => {
                info!("File accessed: {:?}", path);
            }
            FileEvent::Deleted(path) => {
                info!("File deleted: {:?}", path);
            }
        }
    });
    
    monitor.lock().unwrap().start()?;
    
    // Set up access control
    let mut policy = AccessPolicy::new(AccessDecision::Deny);
    
    policy.add_rule(PolicyRule {
        path_pattern: "C:\\Users\\*\\Documents\\*.docx".to_string(),
        allowed_operations: vec![FileOperation::Read, FileOperation::Write],
        denied_operations: vec![FileOperation::Delete],
        priority: 100,
    });
    
    policy.add_rule(PolicyRule {
        path_pattern: "C:\\ProgramData\\Sensitive\\*".to_string(),
        allowed_operations: vec![FileOperation::Read],
        denied_operations: vec![FileOperation::Write, FileOperation::Delete],
        priority: 200,
    });
    
    let policy = Arc::new(policy);
    
    // Set up encryption manager
    let encryption = EncryptionManager::new()?;
    
    // Example: Check access and encrypt
    let test_path = "C:\\Users\\TestUser\\Documents\\report.docx";
    let decision = policy.evaluate(test_path, &FileOperation::Write);
    
    match decision {
        AccessDecision::Allow => {
            info!("Access granted for: {}", test_path);
            encryption.encrypt_file(test_path)?;
        }
        AccessDecision::Deny => {
            error!("Access denied for: {}", test_path);
        }
        _ => {}
    }
    
    // Keep running
    info!("Security loader active. Press Ctrl+C to exit.");
    loop {
        thread::sleep(Duration::from_secs(1));
    }
}
```

## Common Patterns

### Pattern: Real-time File Event Handler

```rust
fn setup_realtime_handler(config: &Config) -> FileMonitor {
    let mut monitor = FileMonitor::new(config.monitoring.watched_paths.clone());
    
    monitor.set_callback(|event| {
        match event {
            FileEvent::Modified(path) | FileEvent::Created(path) => {
                if should_scan_file(&path) {
                    scan_for_threats(&path);
                }
            }
            _ => {}
        }
    });
    
    monitor
}

fn should_scan_file(path: &PathBuf) -> bool {
    if let Some(ext) = path.extension() {
        matches!(ext.to_str(), Some("exe") | Some("dll") | Some("ps1"))
    } else {
        false
    }
}
```

### Pattern: Policy-based Access Gateway

```rust
fn create_access_gateway(policy: Arc<AccessPolicy>) -> impl Fn(&str, FileOperation) -> bool {
    move |path, operation| {
        let decision = policy.evaluate(path, &operation);
        matches!(decision, AccessDecision::Allow | AccessDecision::AuditAndAllow)
    }
}

// Usage
let gateway = create_access_gateway(policy.clone());
if gateway("C:\\Users\\test\\file.txt", FileOperation::Read) {
    // Proceed with file access
}
```

### Pattern: Automatic Encryption on Write

```rust
fn setup_auto_encrypt(paths: Vec<String>) {
    let encryption = EncryptionManager::new().unwrap();
    
    for path in paths {
        if path.contains("Sensitive") {
            thread::spawn(move || {
                if let Err(e) = encryption.encrypt_file(&path) {
                    error!("Auto-encryption failed for {}: {}", path, e);
                }
            });
        }
    }
}
```

## Troubleshooting

### SDK Initialization Fails

**Problem**: EaseFilter SDK fails to initialize.

**Solution**: Ensure SDK path is correct and process has administrator privileges:

```rust
fn check_admin_privileges() -> bool {
    use winapi::um::securitybaseapi::IsUserAnAdmin;
    unsafe { IsUserAnAdmin() != 0 }
}

if !check_admin_privileges() {
    error!("Administrator privileges required");
    return Err("Insufficient privileges".into());
}
```

### File Monitoring Not Capturing Events

**Problem**: File events are not being detected.

**Solution**: Verify paths exist and are accessible:

```rust
use std::path::Path;

fn validate_paths(paths: &[String]) -> Vec<String> {
    paths.iter()
        .filter(|p| {
            let valid = Path::new(p).exists();
            if !valid {
                error!("Path does not exist: {}", p);
            }
            valid
        })
        .cloned()
        .collect()
}
```

### Access Denied Errors

**Problem**: Security loader cannot access certain files.

**Solution**: Check Windows ACLs and run with elevated permissions:

```rust
use winapi::um::winnt::GENERIC_READ;
use winapi::um::fileapi::CreateFileW;

fn check_file_access(path: &str) -> bool {
    use std::ffi::OsStr;
    use std::os::windows::ffi::OsStrExt;
    
    let wide: Vec<u16> = OsStr::new(path)
        .encode_wide()
        .chain(Some(0))
        .collect();
    
    unsafe {
        let handle = CreateFileW(
            wide.as_ptr(),
            GENERIC_READ,
            0,
            std::ptr::null_mut(),
            3, // OPEN_EXISTING
            0,
            std::ptr::null_mut(),
        );
        
        !handle.is_null()
    }
}
```

### Encryption Key Not Found

**Problem**: Encryption fails due to missing key.

**Solution**: Set environment variable or use key storage:

```bash
# Set encryption key
set ENCRYPTION_KEY=base64_encoded_key_here
```

```rust
fn get_encryption_key() -> Result<Vec<u8>, Box<dyn std::error::Error>> {
    env::var("ENCRYPTION_KEY")
        .or_else(|_| {
            // Fallback to registry or key store
            read_key_from_registry()
        })
        .and_then(|k| base64::decode(k).map_err(Into::into))
}
```

### Performance Issues with Large File Sets

**Problem**: Monitoring many files causes performance degradation.

**Solution**: Use path filtering and rate limiting:

```rust
fn filter_high_priority_paths(paths: Vec<String>) -> Vec<String> {
    paths.into_iter()
        .filter(|p| {
            !p.contains("\\AppData\\Local\\Temp") && 
            !p.ends_with(".tmp")
        })
        .collect()
}
```

## Environment Variables

- `EASEFILTER_SDK_PATH`: Path to EaseFilter SDK installation
- `ENCRYPTION_KEY`: Base64-encoded encryption key
- `RUST_LOG`: Logging level (debug, info, warn, error)
- `SECURITY_CONFIG`: Path to configuration file (default: `config.toml`)

## Best Practices

1. **Always run with administrator privileges** for filesystem driver integration
2. **Validate configuration** before initializing SDK
3. **Use environment variables** for sensitive data (keys, credentials)
4. **Implement proper logging** for audit trails
5. **Test policies in audit mode** before enforcement
6. **Handle SDK errors gracefully** with proper cleanup
7. **Use thread-safe structures** for concurrent file operations
8. **Regularly update** EaseFilter SDK to latest version
