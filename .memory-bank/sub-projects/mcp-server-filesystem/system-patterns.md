# mcp-server-filesystem - System Patterns

**Last Updated**: 2025-12-17  
**Version**: 1.0.0-rc.1

## Overview

This document captures filesystem-specific patterns that extend workspace patterns for secure file operations.

## Security-First Architecture

### Defense-in-Depth Pattern

```
┌─────────────────────────────────────┐
│  1. Input Validation                │  ← Validate all inputs
├─────────────────────────────────────┤
│  2. Path Validation                 │  ← Prevent traversal
├─────────────────────────────────────┤
│  3. Binary Detection                │  ← Block binary files
├─────────────────────────────────────┤
│  4. Policy Check                    │  ← Check permissions
├─────────────────────────────────────┤
│  5. Human Approval                  │  ← Confirm write ops
├─────────────────────────────────────┤
│  6. Operation Execution             │  ← Execute safely
├─────────────────────────────────────┤
│  7. Audit Logging                   │  ← Track everything
└─────────────────────────────────────┘
```

### Path Validation Pattern

```rust
pub fn validate_path(path: &Path, config: &SecurityConfig) -> Result<PathBuf> {
    // 1. Normalize path (resolve .., symlinks)
    let canonical = path.canonicalize()
        .map_err(|_| SecurityError::InvalidPath)?;
    
    // 2. Check against allowed paths
    let is_allowed = config.allowed_paths.iter()
        .any(|pattern| pattern.matches_path(&canonical));
    
    if !is_allowed {
        return Err(SecurityError::PathNotAllowed);
    }
    
    // 3. Check against denied paths
    let is_denied = config.denied_paths.iter()
        .any(|pattern| pattern.matches_path(&canonical));
    
    if is_denied {
        return Err(SecurityError::PathDenied);
    }
    
    Ok(canonical)
}
```

### Binary Detection Pattern

```rust
pub fn is_binary_file(path: &Path) -> Result<bool> {
    // Layer 1: Extension check
    if let Some(ext) = path.extension() {
        if BINARY_EXTENSIONS.contains(&ext.to_string_lossy().as_ref()) {
            return Ok(true);
        }
    }
    
    // Layer 2: Content-based detection
    let mut file = File::open(path)?;
    let mut buffer = [0; 8192];
    let bytes_read = file.read(&mut buffer)?;
    
    // Check for null bytes (common in binary files)
    if buffer[..bytes_read].contains(&0) {
        return Ok(true);
    }
    
    // Layer 3: Magic number detection
    if let Some(kind) = infer::get(&buffer[..bytes_read]) {
        return Ok(!kind.mime_type().starts_with("text/"));
    }
    
    Ok(false)
}
```

## Configuration Patterns

### Environment-Aware Configuration

```rust
pub struct ConfigLoader {
    env: Environment,
    config_dir: PathBuf,
}

impl ConfigLoader {
    pub fn load(&self) -> Result<ServerConfig> {
        Config::builder()
            // 1. Defaults
            .add_source(Config::try_from(&DefaultConfig::default())?)
            // 2. Environment-specific file
            .add_source(File::with_name(&format!(
                "{}/config/{}.toml",
                self.config_dir.display(),
                self.env
            )).required(false))
            // 3. Environment variables
            .add_source(Environment::with_prefix("AIRSPROTOCOLS_MCPSERVER_FS"))
            .build()?
            .try_deserialize()
    }
}
```

### Policy-Based Security

```rust
pub struct SecurityPolicy {
    pub patterns: Vec<String>,
    pub operations: Vec<Operation>,
    pub risk_level: RiskLevel,
}

impl SecurityPolicy {
    pub fn allows(&self, path: &Path, op: Operation) -> bool {
        // Check if path matches policy patterns
        let matches_pattern = self.patterns.iter()
            .any(|pattern| glob_match(pattern, path));
        
        // Check if operation is allowed
        let operation_allowed = self.operations.contains(&op);
        
        matches_pattern && operation_allowed
    }
}
```

## File Operation Patterns

### Safe File Write Pattern

```rust
pub async fn write_file(
    path: &Path,
    content: &str,
    approval: &ApprovalService,
) -> Result<()> {
    // 1. Validate path
    let canonical_path = validate_path(path, &config)?;
    
    // 2. Check if binary
    if is_binary_file(&canonical_path)? {
        return Err(Error::BinaryFileNotAllowed);
    }
    
    // 3. Request approval
    if !approval.request_approval(&canonical_path, Operation::Write).await? {
        return Err(Error::ApprovalDenied);
    }
    
    // 4. Write atomically
    let temp_path = canonical_path.with_extension("tmp");
    tokio::fs::write(&temp_path, content).await?;
    tokio::fs::rename(&temp_path, &canonical_path).await?;
    
    // 5. Audit log
    audit_log(AuditEvent::FileWritten {
        path: canonical_path,
        size: content.len(),
    }).await;
    
    Ok(())
}
```

### Atomic File Operations

```rust
pub async fn move_file(src: &Path, dst: &Path) -> Result<()> {
    // Validate both paths
    let src_canonical = validate_path(src, &config)?;
    let dst_canonical = validate_path(dst, &config)?;
    
    // Use atomic rename when possible
    tokio::fs::rename(&src_canonical, &dst_canonical).await?;
    
    // Audit the move
    audit_log(AuditEvent::FileMoved {
        from: src_canonical,
        to: dst_canonical,
    }).await;
    
    Ok(())
}
```

## Audit Logging Pattern

```rust
pub async fn audit_log(event: AuditEvent) {
    let entry = AuditLogEntry {
        timestamp: Utc::now(),
        event,
        user: current_user(),
        session_id: current_session_id(),
    };
    
    // Log to file
    let log_file = audit_log_path();
    append_to_file(log_file, &serde_json::to_string(&entry)?).await;
    
    // Log to tracing
    info!(
        name: event.name(),
        timestamp = %entry.timestamp,
        user = %entry.user,
        session = %entry.session_id,
        "Audit event: {}", event.description()
    );
}
```

## Error Handling Pattern

```rust
#[derive(Error, Debug)]
pub enum FilesystemError {
    #[error("Path validation failed: {0}")]
    PathValidation(String),
    
    #[error("Binary file not allowed: {0}")]
    BinaryFileNotAllowed(PathBuf),
    
    #[error("Approval denied for operation")]
    ApprovalDenied,
    
    #[error("Security policy violation: {0}")]
    PolicyViolation(String),
    
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
}
```

## Testing Patterns

### Security Test Pattern

```rust
#[tokio::test]
async fn test_path_traversal_prevention() {
    let attack_paths = vec![
        "../../../etc/passwd",
        "..\\..\\..\\windows\\system32",
        "/etc/passwd",
        "%2e%2e%2f%2e%2e%2f",  // URL encoded
    ];
    
    for attack_path in attack_paths {
        let result = validate_path(Path::new(attack_path), &config);
        assert!(
            result.is_err(),
            "Attack path should be blocked: {}",
            attack_path
        );
    }
}
```

## Related Documentation

- **Workspace Patterns**: `../../workspace/shared-patterns.md`
- **Security Audit**: `../../../mcp/servers/filesystem/SECURITY_AUDIT.md`
- **Tech Context**: `tech-context.md`

---

These patterns ensure secure, reliable filesystem operations.
