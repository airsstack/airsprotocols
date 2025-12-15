# Workspace Shared Patterns

**Last Updated**: 2025-12-15  
**Applies To**: All sub-projects in AirsProtocols workspace

## Authoritative Standards

**CRITICAL**: This workspace follows the **AirS Stack Rust Guidelines** as the authoritative source for all Rust development standards.

📖 **Source**: `../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md`

All projects MUST adhere to these guidelines, which are based on Microsoft's Pragmatic Rust Guidelines and tailored for AI agent collaboration.

## Core Implementation Patterns

### 1. Error Handling Strategy

**Pattern**: Canonical Error Structs (M-ERRORS-CANONICAL-STRUCTS)

```rust
// Libraries use thiserror for structured errors
use thiserror::Error;
use std::backtrace::Backtrace;

#[derive(Debug, Error)]
pub struct ProtocolError {
    kind: ErrorKind,
    backtrace: Backtrace,
}

impl ProtocolError {
    pub fn is_timeout(&self) -> bool { /* ... */ }
    pub fn is_transport(&self) -> bool { /* ... */ }
}

// Applications may use anyhow (M-APP-ERROR)
use anyhow::Result;

fn start_server() -> Result<()> {
    // Application-level error handling
}
```

**Rationale**: 
- Libraries provide structured, actionable errors
- Applications use anyhow for ergonomic error propagation
- Backtraces captured for development debugging

### 2. Async Patterns

**Pattern**: Tokio-First with Futures Abstraction

```rust
use tokio::runtime::Runtime;
use async_trait::async_trait;

#[async_trait]
pub trait MessageHandler: Send + Sync {
    async fn handle(&self, msg: Message) -> Result<Response>;
}

// All public futures must be Send
const _: () = {
    fn assert_send<T: Send>() {}
    assert_send::<impl Future<Output = ()>>();
};
```

**Key Rules**:
- All async operations use `tokio` runtime
- Public futures MUST be `Send` (M-TYPES-SEND)
- Long-running tasks include yield points (M-YIELD-POINTS)
- Optimize for throughput over latency (M-THROUGHPUT)

### 3. Type Safety Patterns

**Pattern**: Strong Types Over Primitives

```rust
// Avoid primitive obsession
use std::path::PathBuf;

// Good: Strong types
pub struct ToolName(String);
pub struct ResourceUri(String);
pub fn read_file(path: PathBuf) -> Result<String> { /* ... */ }

// Bad: Primitive obsession
pub fn read_file(path: String) -> Result<String> { /* ... */ }
```

**Guidelines**:
- Use `PathBuf`/`Path` for filesystem operations (M-STRONG-TYPES)
- Create newtypes for domain concepts
- Leverage type system for compile-time validation

### 4. Authentication & Authorization

**Pattern**: Strategy-Based Authentication with Middleware Authorization

```rust
use async_trait::async_trait;

// Authentication strategies
#[async_trait]
pub trait AuthStrategy: Send + Sync {
    async fn authenticate(&self, req: &Request) -> Result<AuthContext>;
}

pub struct ApiKeyStrategy { /* ... */ }
pub struct OAuth2Strategy { /* ... */ }

// Authorization middleware
pub struct AuthorizationMiddleware {
    policies: Vec<Box<dyn Policy>>,
}

impl AuthorizationMiddleware {
    pub async fn authorize(&self, ctx: &AuthContext, req: &Request) -> Result<()> {
        // Policy evaluation
    }
}
```

**Rationale**:
- Pluggable authentication strategies
- Declarative authorization policies
- Separation of concerns

### 5. Transport Abstraction

**Pattern**: Transport-Agnostic Protocol Layer

```rust
#[async_trait]
pub trait Transport: Send + Sync {
    async fn send(&mut self, msg: JsonRpcMessage) -> Result<()>;
    async fn receive(&mut self) -> Result<JsonRpcMessage>;
}

// Implementations
pub struct StdioTransport { /* ... */ }
pub struct HttpTransport { /* ... */ }
pub struct WebSocketTransport { /* ... */ }
```

**Benefits**:
- Protocol logic independent of transport
- Easy to add new transports
- Testable with mock transports

### 6. Provider Framework

**Pattern**: Capability-Based Providers

```rust
#[async_trait]
pub trait ToolProvider: Send + Sync {
    async fn list_tools(&self) -> Result<Vec<Tool>>;
    async fn call_tool(&self, call: ToolCall) -> Result<ToolResult>;
}

#[async_trait]
pub trait ResourceProvider: Send + Sync {
    async fn list_resources(&self) -> Result<Vec<Resource>>;
    async fn read_resource(&self, uri: &str) -> Result<ResourceContent>;
}

#[async_trait]
pub trait PromptProvider: Send + Sync {
    async fn list_prompts(&self) -> Result<Vec<Prompt>>;
    async fn get_prompt(&self, name: &str) -> Result<PromptTemplate>;
}
```

**Design Principles**:
- Narrow, focused traits (M-DI-HIERARCHY)
- Async-first APIs
- Composable capabilities

## Architecture Patterns

### 1. Layered Architecture

```
┌─────────────────────────────────────┐
│   Integration Layer                 │  ← McpServer, McpClient
│   (High-level builders)             │
├─────────────────────────────────────┤
│   Providers Layer                   │  ← Tool/Resource/Prompt providers
│   (Capability implementation)       │
├─────────────────────────────────────┤
│   Protocol Layer                    │  ← JSON-RPC, MCP types
│   (Message structures)              │
├─────────────────────────────────────┤
│   Transport Layer                   │  ← Stdio, HTTP, WebSocket
│   (Communication channels)          │
├─────────────────────────────────────┤
│   Auth/Authz Layer                  │  ← Security middleware
│   (Security enforcement)            │
└─────────────────────────────────────┘
```

**Principles**:
- Clear separation of concerns
- Dependencies flow downward
- Each layer has well-defined responsibilities

### 2. Dependency Injection

**Pattern**: Constructor Injection with Arc

```rust
use std::sync::Arc;

pub struct McpServer {
    tool_provider: Arc<dyn ToolProvider>,
    resource_provider: Arc<dyn ResourceProvider>,
    transport: Arc<dyn Transport>,
}

impl McpServer {
    pub fn new(
        tool_provider: Arc<dyn ToolProvider>,
        resource_provider: Arc<dyn ResourceProvider>,
        transport: Arc<dyn Transport>,
    ) -> Self {
        Self {
            tool_provider,
            resource_provider,
            transport,
        }
    }
}
```

**Guidelines**:
- Prefer concrete types over generics (M-DI-HIERARCHY)
- Use Arc for shared ownership
- Avoid smart pointer proliferation in public APIs (M-AVOID-WRAPPERS)

### 3. Configuration Management

**Pattern**: Environment-Aware Configuration

```rust
use config::{Config, Environment, File};
use serde::Deserialize;

#[derive(Debug, Deserialize)]
pub struct ServerConfig {
    pub server: ServerSettings,
    pub security: SecuritySettings,
    pub logging: LoggingSettings,
}

impl ServerConfig {
    pub fn load() -> Result<Self> {
        Config::builder()
            .add_source(File::with_name("config/default"))
            .add_source(File::with_name(&format!("config/{}", env)))
            .add_source(Environment::with_prefix("APP"))
            .build()?
            .try_deserialize()
    }
}
```

**Features**:
- Environment-specific overrides
- Environment variable support
- Type-safe configuration

## Testing Patterns

### 1. Unit Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_message_validation() {
        let msg = create_test_message();
        assert!(msg.validate().is_ok());
    }

    #[tokio::test]
    async fn test_async_handler() {
        let handler = MockHandler::new();
        let result = handler.handle(test_message()).await;
        assert!(result.is_ok());
    }
}
```

### 2. Integration Testing

```rust
#[tokio::test]
async fn test_full_mcp_flow() {
    // Setup server
    let server = setup_test_server().await;
    
    // Setup client
    let client = setup_test_client().await;
    
    // Test interaction
    let tools = client.list_tools().await.unwrap();
    assert!(!tools.is_empty());
}
```

### 3. Property-Based Testing

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_path_validation(path in ".*") {
        let result = validate_path(&path);
        // Invariants that must hold for all inputs
    }
}
```

## Performance Patterns

### 1. Hot Path Optimization (M-HOTPATH)

- Identify performance-critical paths early
- Use `criterion` for benchmarking
- Profile with VTune/Superluminal
- Document performance-sensitive areas

### 2. Allocation Optimization

- Avoid unnecessary cloning
- Use `&str` over `String` where possible
- Reuse allocations in loops
- Consider bump allocators for short-lived data

### 3. Concurrent Operations

- Batch operations where possible
- Use connection pooling
- Exploit CPU caches
- Avoid hot-spinning

## Security Patterns

### 1. Path Validation

```rust
use std::path::{Path, PathBuf};
use path_clean::PathClean;

pub fn validate_path(path: &Path, allowed_roots: &[PathBuf]) -> Result<PathBuf> {
    // 1. Canonicalize
    let canonical = path.canonicalize()?;
    
    // 2. Check against allowed roots
    let is_allowed = allowed_roots.iter()
        .any(|root| canonical.starts_with(root));
    
    if !is_allowed {
        return Err(SecurityError::PathNotAllowed);
    }
    
    Ok(canonical)
}
```

### 2. Input Validation

- Validate all external inputs
- Use strong types
- Fail early and explicitly
- Log validation failures

### 3. Audit Logging

```rust
use tracing::{info, warn};

info!(
    action = "file.read",
    path = ?file_path,
    user_id = user.id,
    "File read operation"
);
```

## Documentation Patterns

### 1. Module Documentation (M-MODULE-DOCS)

```rust
//! # Protocol Layer
//!
//! This module contains the core MCP protocol types and message structures.
//!
//! ## Examples
//!
//! ```rust
//! use airsprotocols_mcp::protocol::JsonRpcMessage;
//! let msg = JsonRpcMessage::request(/* ... */);
//! ```
```

### 2. Function Documentation (M-CANONICAL-DOCS)

```rust
/// Validates and canonicalizes a filesystem path.
///
/// # Examples
///
/// ```rust
/// let path = validate_path("/home/user/file.txt")?;
/// ```
///
/// # Errors
///
/// Returns `SecurityError::PathNotAllowed` if the path is outside allowed roots.
///
/// # Security
///
/// This function prevents directory traversal attacks by canonicalizing paths.
pub fn validate_path(path: &Path) -> Result<PathBuf> {
    // ...
}
```

## Logging Patterns (M-LOG-STRUCTURED)

**Pattern**: Structured Logging with Tracing

```rust
use tracing::{info, warn, error, event, Level};

// Named events with properties
event!(
    name: "file.write.success",
    Level::INFO,
    file.path = path.display(),
    file.size = bytes_written,
    "Wrote {{file.size}} bytes to {{file.path}}"
);

// Error logging
error!(
    name: "auth.failed",
    reason = "invalid_token",
    user_id = ?user_id,
    "Authentication failed: {{reason}}"
);
```

**Guidelines**:
- Use message templates (not string formatting)
- Follow OpenTelemetry semantic conventions
- Redact sensitive data
- Name all events

## Code Organization

### 1. Module Structure

```
src/
├── lib.rs              # Public API exports
├── protocol/           # Protocol types
│   ├── mod.rs
│   ├── message.rs
│   └── types.rs
├── transport/          # Transport implementations
│   ├── mod.rs
│   ├── stdio.rs
│   └── http.rs
├── providers/          # Capability providers
│   ├── mod.rs
│   ├── tool.rs
│   └── resource.rs
└── integration/        # High-level APIs
    ├── mod.rs
    ├── server.rs
    └── client.rs
```

### 2. Feature Flags

```toml
[features]
default = ["stdio"]
stdio = []
http = ["axum", "hyper"]
oauth2 = ["oauth2-crate", "jsonwebtoken"]
test-util = []
```

**Guidelines** (M-FEATURES-ADDITIVE):
- All features are additive
- No `no-std` feature (use `std` feature instead)
- Test utilities behind `test-util` flag

## Linting & Verification (M-STATIC-VERIFICATION)

### Workspace Lints

```toml
[workspace.lints.rust]
unsafe_op_in_unsafe_fn = "warn"
missing_debug_implementations = "warn"

[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "deny"
panic = "deny"
```

**Enforcement**:
- CI pipeline checks all lints
- Pre-commit hooks recommended
- Use `#[expect]` for overrides (M-LINT-OVERRIDE-EXPECT)

## Related Documentation

- **Rust Guidelines**: `../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md`
- **Architecture**: `workspace-architecture.md`
- **Progress**: `workspace-progress.md`

---

These patterns ensure consistency, maintainability, and quality across all AirsProtocols sub-projects.
