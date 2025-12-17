# protocols-mcp - System Patterns

**Last Updated**: 2025-12-17  
**Version**: 1.0.0-rc.1

## Overview

This document captures the system architecture, design patterns, and technical decisions specific to the `protocols-mcp` sub-project. It extends the workspace-level patterns in `../../workspace/shared-patterns.md` with MCP-specific implementations.

## Architecture

### Layered Architecture

The crate follows a strict layered architecture where dependencies flow downward:

```
┌─────────────────────────────────────┐
│   Integration Layer                 │  ← McpServer, McpClient builders
│   (High-level convenience APIs)     │
├─────────────────────────────────────┤
│   Providers Layer                   │  ← Tool/Resource/Prompt providers
│   (Capability implementations)      │
├─────────────────────────────────────┤
│   Protocol Layer                    │  ← JSON-RPC, MCP message types
│   (Core protocol structures)        │
├─────────────────────────────────────┤
│   Auth/Authz Layer                  │  ← Security middleware
│   (Security enforcement)            │
├─────────────────────────────────────┤
│   Transport Layer                   │  ← Stdio, HTTP, SSE adapters
│   (Communication channels)          │
└─────────────────────────────────────┘
```

**Key Principles**:
1. **Each layer depends only on layers below**
2. **No circular dependencies**
3. **Clear responsibilities per layer**
4. **Testable in isolation**

### Module Structure

```
src/
├── lib.rs                    # Public API exports
├── protocol/                 # Core protocol types
│   ├── mod.rs
│   ├── message.rs           # JsonRpcMessage
│   ├── types.rs             # MCP type definitions
│   ├── errors.rs            # Protocol errors
│   └── constants.rs         # Protocol constants
├── transport/                # Transport abstraction
│   ├── mod.rs               # Transport trait
│   ├── error.rs             # Transport errors
│   └── adapters/
│       ├── stdio.rs         # Stdio transport
│       ├── http.rs          # HTTP transport
│       └── sse.rs           # SSE support
├── authentication/           # Auth strategies
│   ├── mod.rs
│   ├── manager.rs           # Auth manager
│   ├── context.rs           # Auth context
│   ├── error.rs             # Auth errors
│   └── strategies/
│       ├── apikey.rs        # API Key auth
│       └── oauth2.rs        # OAuth2 auth
├── authorization/            # Authorization middleware
│   ├── mod.rs
│   ├── middleware.rs        # Policy enforcement
│   ├── policy.rs            # Policy definitions
│   ├── context.rs           # Authz context
│   └── extractor.rs         # Request extraction
├── oauth2/                   # OAuth2 lifecycle
│   ├── mod.rs
│   ├── lifecycle/           # Token management
│   ├── middleware/          # HTTP middleware
│   └── validator/           # JWT validation
├── providers/                # Provider framework
│   ├── mod.rs
│   ├── tool.rs              # ToolProvider trait
│   ├── resource.rs          # ResourceProvider trait
│   ├── prompt.rs            # PromptProvider trait
│   └── logging.rs           # LoggingProvider trait
└── integration/              # High-level APIs
    ├── mod.rs
    ├── server.rs            # McpServer builder
    ├── client.rs            # McpClient builder
    ├── error.rs             # Integration errors
    └── constants.rs         # Integration constants
```

## Core Design Patterns

### 1. Transport Abstraction Pattern

**Pattern**: Trait-based transport abstraction with async operations

**Implementation**:
```rust
#[async_trait]
pub trait Transport: Send + Sync {
    async fn send(&mut self, msg: JsonRpcMessage) -> Result<()>;
    async fn receive(&mut self) -> Result<JsonRpcMessage>;
}

// Concrete implementations
pub struct StdioTransport { /* ... */ }
pub struct HttpTransport { /* ... */ }
```

**Rationale**:
- Protocol logic independent of transport mechanism
- Easy to add new transports (WebSocket, gRPC)
- Testable with mock transports
- Clean separation of concerns

**Usage**:
```rust
// Works with any transport
let server = McpServerBuilder::new()
    .build(stdio_transport)
    .await?;

// Or HTTP transport
let server = McpServerBuilder::new()
    .build(http_transport)
    .await?;
```

### 2. Strategy Pattern for Authentication

**Pattern**: Pluggable authentication strategies with trait abstraction

**Implementation**:
```rust
#[async_trait]
pub trait AuthStrategy: Send + Sync {
    async fn authenticate(&self, req: &Request) -> Result<AuthContext>;
}

// Concrete strategies
pub struct ApiKeyStrategy { /* ... */ }
pub struct OAuth2Strategy { /* ... */ }
```

**Rationale**:
- Different servers have different auth requirements
- Easy to add custom auth mechanisms
- Strategy can be swapped without changing server code
- Testable independently

**Usage**:
```rust
// API Key for internal tools
let auth = ApiKeyStrategy::new(api_keys);

// OAuth2 for public APIs
let auth = OAuth2Strategy::new(oauth_config);

let server = McpServerBuilder::new()
    .with_auth(auth)
    .build(transport)
    .await?;
```

### 3. Provider Framework Pattern

**Pattern**: Capability-based providers with focused traits

**Implementation**:
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

**Rationale**:
- Focused, single-responsibility traits (M-DI-HIERARCHY)
- Composable capabilities
- Easy to implement and test
- Clear contract for capability providers

**Usage**:
```rust
struct MyToolProvider;

#[async_trait]
impl ToolProvider for MyToolProvider {
    async fn call_tool(&self, call: ToolCall) -> Result<ToolResult> {
        // Implementation
    }
}

let server = McpServerBuilder::new()
    .with_tool_provider(Arc::new(MyToolProvider))
    .build(transport)
    .await?;
```

### 4. Builder Pattern for Integration

**Pattern**: Fluent builder APIs for server and client configuration

**Implementation**:
```rust
pub struct McpServerBuilder {
    tool_provider: Option<Arc<dyn ToolProvider>>,
    resource_provider: Option<Arc<dyn ResourceProvider>>,
    auth_strategy: Option<Arc<dyn AuthStrategy>>,
    // ... other configuration
}

impl McpServerBuilder {
    pub fn new() -> Self { /* ... */ }
    
    pub fn with_tool_provider(mut self, provider: Arc<dyn ToolProvider>) -> Self {
        self.tool_provider = Some(provider);
        self
    }
    
    pub fn with_auth(mut self, auth: Arc<dyn AuthStrategy>) -> Self {
        self.auth_strategy = Some(auth);
        self
    }
    
    pub async fn build<T: Transport>(self, transport: T) -> Result<McpServer> {
        // Construct server with validation
    }
}
```

**Rationale**:
- Ergonomic API for configuration
- Compile-time type safety
- Clear default values
- Extensible for future options

**Usage**:
```rust
let server = McpServerBuilder::new()
    .with_tool_provider(tools)
    .with_resource_provider(resources)
    .with_auth(api_key_auth)
    .with_timeout(Duration::from_secs(30))
    .build(transport)
    .await?;
```

### 5. Error Handling Pattern

**Pattern**: Canonical error structs with thiserror (M-ERRORS-CANONICAL-STRUCTS)

**Implementation**:
```rust
use thiserror::Error;
use std::backtrace::Backtrace;

#[derive(Debug, Error)]
pub struct ProtocolError {
    kind: ErrorKind,
    backtrace: Backtrace,
}

#[derive(Debug, Error)]
pub enum ErrorKind {
    #[error("Invalid message format: {0}")]
    InvalidMessage(String),
    
    #[error("Transport error: {0}")]
    Transport(#[from] TransportError),
    
    #[error("Authentication failed")]
    AuthenticationFailed,
}

impl ProtocolError {
    pub fn is_timeout(&self) -> bool { /* ... */ }
    pub fn is_transport(&self) -> bool { /* ... */ }
}
```

**Rationale**:
- Structured errors for library consumers
- Backtraces for debugging
- Type checking on error kinds
- Actionable error information

**Usage**:
```rust
match server.handle_message(msg).await {
    Err(e) if e.is_timeout() => { /* retry */ },
    Err(e) if e.is_transport() => { /* reconnect */ },
    Err(e) => { /* handle other errors */ },
    Ok(response) => { /* success */ },
}
```

## Key Technical Decisions

### Decision: Transport Trait Design

**Context**: Need to support multiple transport mechanisms (stdio, HTTP, WebSocket) without coupling protocol logic to specific transports.

**Options Considered**:
1. Enum-based transport with match statements
2. Trait-based abstraction with async methods
3. Callback-based design

**Decision**: Trait-based abstraction with async methods

**Rationale**:
- Most flexible for adding new transports
- Clean separation of concerns
- Testable with mock transports
- Follows Rust ecosystem patterns

**Tradeoffs**:
- Requires trait objects (minimal overhead)
- Async traits need `async-trait` crate
- Cannot use associated types for send/receive

**References**:
- Workspace patterns: `../../workspace/shared-patterns.md#transport-abstraction`
- Implementation: `src/transport/mod.rs`

### Decision: Authentication Strategy Pattern

**Context**: Different deployments require different authentication mechanisms (API Key, OAuth2, custom).

**Options Considered**:
1. Hardcode auth mechanisms
2. Strategy pattern with trait
3. Middleware-only approach

**Decision**: Strategy pattern with trait-based abstraction

**Rationale**:
- Pluggable authentication strategies
- Easy to implement custom auth
- Testable independently
- Composable with authorization

**Tradeoffs**:
- Requires trait objects
- More complex initial setup
- Additional abstraction layer

**References**:
- Implementation: `src/authentication/`
- Examples: `examples/http-apikey-server-integration/`

### Decision: Provider Framework with Focused Traits

**Context**: MCP defines three capability types (tools, resources, prompts) that servers can provide.

**Options Considered**:
1. Single `CapabilityProvider` trait with all methods
2. Separate traits per capability
3. Enum-based capability system

**Decision**: Separate traits per capability type

**Rationale**:
- Single-responsibility principle
- Servers can implement only needed capabilities
- Easier to understand and test
- Follows M-DI-HIERARCHY guideline

**Tradeoffs**:
- More traits to manage
- Slightly more complex builder API
- Users must understand three traits

**References**:
- Workspace patterns: `../../workspace/shared-patterns.md#provider-framework`
- Implementation: `src/providers/`

### Decision: Builder Pattern for Integration APIs

**Context**: Need ergonomic APIs for constructing servers and clients with many configuration options.

**Options Considered**:
1. Constructor with many parameters
2. Struct with public fields
3. Builder pattern with fluent API

**Decision**: Builder pattern with fluent API

**Rationale**:
- Ergonomic configuration
- Clear default values
- Extensible for future options
- Compile-time validation

**Tradeoffs**:
- More code to maintain
- Two APIs (builder + server/client)
- Learning curve for new users

**References**:
- Implementation: `src/integration/server.rs`, `src/integration/client.rs`
- Examples: All example projects

### Decision: Async-First APIs

**Context**: All I/O operations (transport, auth, providers) are inherently asynchronous.

**Options Considered**:
1. Blocking APIs with thread pool
2. Async APIs with tokio
3. Hybrid approach

**Decision**: Async-first with tokio runtime

**Rationale**:
- Non-blocking I/O essential for performance
- tokio is Rust ecosystem standard
- Composable with other async code
- Efficient resource usage

**Tradeoffs**:
- Requires async runtime
- Users must understand async Rust
- Cannot use in no_std environments

**References**:
- Workspace patterns: `../../workspace/shared-patterns.md#async-patterns`
- All implementations use async/await

## MCP-Specific Patterns

### JSON-RPC 2.0 Message Handling

**Pattern**: Type-safe message structures with serde

```rust
#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "jsonrpc")]
pub enum JsonRpcMessage {
    Request(JsonRpcRequest),
    Response(JsonRpcResponse),
    Notification(JsonRpcNotification),
    Error(JsonRpcError),
}
```

**Key Features**:
- Compile-time validation of message format
- Automatic serialization/deserialization
- Type-safe error handling
- Protocol version checking

### Protocol Type Definitions

**Pattern**: Strong types for all MCP concepts

```rust
pub struct ToolName(String);
pub struct ResourceUri(String);

impl ToolName {
    pub fn new(name: impl Into<String>) -> Result<Self> {
        let name = name.into();
        validate_tool_name(&name)?;
        Ok(Self(name))
    }
}
```

**Benefits**:
- Prevent invalid identifiers at compile time
- Self-documenting code
- Clear error messages
- Type-directed API design

### Message Context Pattern

**Pattern**: Thread-safe context for message handling

```rust
pub struct MessageContext<T> {
    auth: Option<AuthContext>,
    transport_meta: TransportMetadata,
    custom: Arc<T>,
}
```

**Usage**:
- Pass authentication context through handlers
- Track request metadata
- Enable custom per-request data
- Support distributed tracing

## Performance Patterns

### Hot Path Optimization (M-HOTPATH)

**Critical Paths**:
1. Message deserialization (JSON-RPC parsing)
2. Transport send/receive
3. Provider invocation
4. Auth token validation

**Optimizations Applied**:
- Zero-copy deserialization where possible
- Connection pooling for HTTP transport
- Token caching for repeated validation
- Allocation reuse in message loops

**Benchmarks**: See `benches/message_processing.rs`

### Memory Management

**Patterns**:
- Use `Arc` for shared providers (immutable)
- Avoid unnecessary cloning with `&str` over `String`
- Stream large payloads instead of buffering
- Reuse buffers in transport loops

**Guidelines**:
- Profile with VTune/Superluminal
- Monitor allocation patterns
- Use `criterion` for micro-benchmarks
- Track memory usage in production

## Security Patterns

### Path Validation (Not Applicable)

This library doesn't handle file paths, but consumers (like filesystem server) must validate paths. See workspace patterns.

### Token Validation

**Pattern**: JWT validation with caching

```rust
pub struct JwtValidator {
    jwks_url: String,
    cache: Arc<DashMap<String, DecodingKey>>,
}

impl JwtValidator {
    pub async fn validate(&self, token: &str) -> Result<Claims> {
        // 1. Parse without verification
        // 2. Get key from cache or fetch
        // 3. Verify signature
        // 4. Validate claims (exp, iss, aud)
    }
}
```

**Security Measures**:
- Verify signature with JWKS
- Check expiration time
- Validate issuer and audience
- Rate limit JWKS fetches
- Cache keys safely

### Authorization Policy Enforcement

**Pattern**: Middleware-based policy evaluation

```rust
pub struct AuthorizationMiddleware {
    policies: Vec<Box<dyn Policy>>,
}

#[async_trait]
pub trait Policy: Send + Sync {
    async fn evaluate(&self, ctx: &AuthContext, req: &Request) -> Result<Decision>;
}
```

**Features**:
- Declarative policy definitions
- Composable policies
- Fail-safe defaults (deny unless allowed)
- Audit all authorization decisions

## Testing Patterns

### Unit Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_message_validation() {
        let msg = JsonRpcMessage::request(/* ... */);
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

### Integration Testing

```rust
#[tokio::test]
async fn test_full_mcp_flow() {
    // Setup server with real providers
    let server = setup_test_server().await;
    
    // Setup client
    let client = setup_test_client().await;
    
    // Test interaction
    let tools = client.list_tools().await.unwrap();
    assert!(!tools.is_empty());
    
    let result = client.call_tool("add", json!({"a": 1, "b": 2})).await.unwrap();
    assert_eq!(result, json!({"result": 3}));
}
```

### Mock Transports

```rust
pub struct MockTransport {
    sent: Arc<Mutex<Vec<JsonRpcMessage>>>,
    to_receive: Arc<Mutex<VecDeque<JsonRpcMessage>>>,
}

#[async_trait]
impl Transport for MockTransport {
    async fn send(&mut self, msg: JsonRpcMessage) -> Result<()> {
        self.sent.lock().await.push(msg);
        Ok(())
    }
    
    async fn receive(&mut self) -> Result<JsonRpcMessage> {
        self.to_receive.lock().await.pop_front()
            .ok_or(Error::NoMessage)
    }
}
```

## Documentation Patterns

### Module Documentation (M-MODULE-DOCS)

```rust
//! # Protocol Layer
//!
//! This module contains the core MCP protocol types and message structures.
//! It implements the JSON-RPC 2.0 specification with MCP extensions.
//!
//! ## Examples
//!
//! ```rust
//! use airsprotocols_mcp::protocol::JsonRpcMessage;
//! 
//! let request = JsonRpcMessage::request("initialize", json!({
//!     "protocolVersion": "2024-11-05"
//! }));
//! ```
```

### Function Documentation (M-CANONICAL-DOCS)

```rust
/// Validates and normalizes a tool name.
///
/// # Examples
///
/// ```rust
/// let name = ToolName::new("my_tool")?;
/// ```
///
/// # Errors
///
/// Returns `ValidationError::InvalidName` if the name contains invalid characters.
///
/// # Security
///
/// This function prevents injection attacks by validating tool names.
pub fn validate_tool_name(name: &str) -> Result<()> {
    // ...
}
```

## Related Documentation

- **Workspace Patterns**: `../../workspace/shared-patterns.md` - Core implementation patterns
- **Architecture**: `../../workspace/workspace-architecture.md` - Overall workspace structure
- **Tech Context**: `tech-context.md` - Technologies and constraints
- **Rust Guidelines**: `../../../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md` - Authoritative Rust standards

---

These patterns ensure consistency, quality, and maintainability in the protocols-mcp codebase.
