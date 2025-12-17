# api-openrouter - System Patterns

**Last Updated**: 2025-12-17  
**Version**: 0.1.0 (Planning)  
**Status**: Draft - Architecture Design Phase

## Overview

This document captures the system architecture, design patterns, and technical decisions for the `airsprotocols-api-openrouter` sub-project. It extends the workspace-level patterns in `../../workspace/shared-patterns.md` with OpenRouter-specific implementations.

## Architecture

### Layered Architecture

The crate follows a layered architecture optimized for API client design:

```
┌─────────────────────────────────────┐
│   Integration Layer                 │  ← OpenRouterClient (high-level API)
│   (Client builder and facade)       │
├─────────────────────────────────────┤
│   Request Builder Layer             │  ← ChatRequestBuilder, etc.
│   (Ergonomic request construction)  │
├─────────────────────────────────────┤
│   API Layer                          │  ← Endpoint implementations
│   (HTTP request/response handling)  │
├─────────────────────────────────────┤
│   Streaming Layer                    │  ← SSE parsing and async streams
│   (Streaming response processing)   │
├─────────────────────────────────────┤
│   Types Layer                        │  ← Message, Model, Request, Response
│   (Protocol type definitions)       │
├─────────────────────────────────────┤
│   HTTP Transport Layer               │  ← reqwest client, connection pool
│   (Network communication)           │
└─────────────────────────────────────┘
```

**Key Principles**:
1. **Client simplicity**: High-level API hides complexity
2. **Builder ergonomics**: Intuitive request construction
3. **Type safety**: Strong types prevent API misuse
4. **Testability**: Mockable HTTP layer

### Module Structure (Planned)

```
src/
├── lib.rs                    # Public API exports
├── client.rs                 # OpenRouterClient main struct
├── types/                    # Type definitions
│   ├── mod.rs
│   ├── message.rs           # Message (System, User, Assistant, Function)
│   ├── model.rs             # Model definitions and metadata
│   ├── request.rs           # ChatRequest, CompletionRequest
│   ├── response.rs          # ChatResponse, StreamChunk
│   ├── usage.rs             # Token usage and cost tracking
│   └── error.rs             # Error types
├── builder/                  # Request builders
│   ├── mod.rs
│   ├── chat.rs              # ChatRequestBuilder
│   └── streaming.rs         # StreamingRequestBuilder
├── api/                      # API endpoint implementations
│   ├── mod.rs
│   ├── chat.rs              # Chat completions endpoint
│   ├── models.rs            # Model listing (future)
│   └── streaming.rs         # SSE streaming handler
├── http/                     # HTTP transport layer
│   ├── mod.rs
│   ├── client.rs            # reqwest client wrapper
│   ├── retry.rs             # Retry logic with backoff
│   └── rate_limit.rs        # Rate limit handling
├── stream/                   # Streaming utilities
│   ├── mod.rs
│   ├── sse.rs               # SSE parsing
│   └── adapter.rs           # Stream type adapters
└── cost/                     # Cost tracking
    ├── mod.rs
    ├── tracker.rs           # Cost calculation and tracking
    └── pricing.rs           # Model pricing data
```

## Core Design Patterns

### 1. Client Builder Pattern

**Pattern**: Fluent builder for client configuration

**Implementation** (Draft):
```rust
pub struct OpenRouterClient {
    http_client: reqwest::Client,
    api_key: String,
    base_url: String,
    cost_tracker: Option<Arc<CostTracker>>,
}

pub struct OpenRouterClientBuilder {
    api_key: Option<String>,
    base_url: Option<String>,
    timeout: Option<Duration>,
    enable_cost_tracking: bool,
    retry_config: RetryConfig,
}

impl OpenRouterClientBuilder {
    pub fn new() -> Self {
        Self {
            api_key: None,
            base_url: Some("https://openrouter.ai/api/v1".to_string()),
            timeout: Some(Duration::from_secs(60)),
            enable_cost_tracking: true,
            retry_config: RetryConfig::default(),
        }
    }
    
    pub fn api_key(mut self, key: impl Into<String>) -> Self {
        self.api_key = Some(key.into());
        self
    }
    
    pub fn timeout(mut self, timeout: Duration) -> Self {
        self.timeout = Some(timeout);
        self
    }
    
    pub fn build(self) -> Result<OpenRouterClient> {
        let api_key = self.api_key.ok_or(Error::MissingApiKey)?;
        
        let http_client = reqwest::Client::builder()
            .timeout(self.timeout.unwrap())
            .build()?;
        
        Ok(OpenRouterClient {
            http_client,
            api_key,
            base_url: self.base_url.unwrap(),
            cost_tracker: if self.enable_cost_tracking {
                Some(Arc::new(CostTracker::new()))
            } else {
                None
            },
        })
    }
}
```

**Rationale**:
- Ergonomic configuration with sensible defaults
- Compile-time validation of required fields
- Extensible for future options
- Follows workspace builder pattern

**Usage**:
```rust
let client = OpenRouterClient::builder()
    .api_key(env::var("OPENROUTER_API_KEY")?)
    .timeout(Duration::from_secs(30))
    .build()?;
```

### 2. Request Builder Pattern

**Pattern**: Fluent builder for API requests

**Implementation** (Draft):
```rust
pub struct ChatRequestBuilder<'a> {
    client: &'a OpenRouterClient,
    model: Option<String>,
    messages: Vec<Message>,
    temperature: Option<f32>,
    max_tokens: Option<u32>,
    stream: bool,
    // ... other parameters
}

impl<'a> ChatRequestBuilder<'a> {
    pub fn model(mut self, model: impl Into<String>) -> Self {
        self.model = Some(model.into());
        self
    }
    
    pub fn messages(mut self, messages: Vec<Message>) -> Self {
        self.messages = messages;
        self
    }
    
    pub fn temperature(mut self, temp: f32) -> Self {
        self.temperature = Some(temp);
        self
    }
    
    pub async fn send(self) -> Result<ChatResponse> {
        let request = self.build_request()?;
        self.client.send_chat_request(request).await
    }
    
    pub async fn send_stream(mut self) -> Result<ChatStream> {
        self.stream = true;
        let request = self.build_request()?;
        self.client.send_stream_request(request).await
    }
}

impl OpenRouterClient {
    pub fn chat(&self) -> ChatRequestBuilder {
        ChatRequestBuilder::new(self)
    }
}
```

**Rationale**:
- Intuitive, chainable API
- Type-safe parameter configuration
- Clear distinction between streaming/non-streaming
- Prevents invalid request construction

**Usage**:
```rust
let response = client
    .chat()
    .model("openai/gpt-4")
    .messages(vec![Message::user("Hello!")])
    .temperature(0.7)
    .send()
    .await?;
```

### 3. Message Type System

**Pattern**: Sealed enum for message variants

**Implementation** (Draft):
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(tag = "role", content = "content")]
pub enum Message {
    #[serde(rename = "system")]
    System(String),
    
    #[serde(rename = "user")]
    User(String),
    
    #[serde(rename = "assistant")]
    Assistant(String),
    
    #[serde(rename = "function")]
    Function {
        name: String,
        content: String,
    },
}

impl Message {
    pub fn system(content: impl Into<String>) -> Self {
        Self::System(content.into())
    }
    
    pub fn user(content: impl Into<String>) -> Self {
        Self::User(content.into())
    }
    
    pub fn assistant(content: impl Into<String>) -> Self {
        Self::Assistant(content.into())
    }
    
    pub fn role(&self) -> &str {
        match self {
            Self::System(_) => "system",
            Self::User(_) => "user",
            Self::Assistant(_) => "assistant",
            Self::Function { .. } => "function",
        }
    }
    
    pub fn content(&self) -> &str {
        match self {
            Self::System(s) | Self::User(s) | Self::Assistant(s) => s,
            Self::Function { content, .. } => content,
        }
    }
}
```

**Rationale**:
- Type-safe message construction
- Impossible to create invalid messages
- Clear, ergonomic API
- Efficient serialization with serde

**Decision**: Enum over trait hierarchy
- **Pro**: Simpler, more idiomatic Rust
- **Pro**: Exhaustive pattern matching
- **Pro**: Efficient memory layout
- **Con**: Less extensible (but OpenRouter API is stable)

### 4. Streaming with Async Streams

**Pattern**: SSE-based streaming with tokio-stream

**Implementation** (Draft):
```rust
use tokio_stream::Stream;
use futures::stream::StreamExt;

pub struct ChatStream {
    inner: Pin<Box<dyn Stream<Item = Result<StreamChunk>> + Send>>,
}

impl ChatStream {
    pub async fn next(&mut self) -> Option<Result<StreamChunk>> {
        self.inner.next().await
    }
}

#[derive(Debug, Deserialize)]
pub struct StreamChunk {
    pub id: String,
    pub model: String,
    pub choices: Vec<StreamChoice>,
    pub usage: Option<Usage>,
}

impl StreamChunk {
    pub fn content(&self) -> Option<&str> {
        self.choices.first()
            .and_then(|c| c.delta.content.as_deref())
    }
    
    pub fn is_done(&self) -> bool {
        self.choices.iter()
            .any(|c| c.finish_reason.is_some())
    }
}
```

**Rationale**:
- Async streams integrate naturally with tokio
- Easy to compose with other streams
- Ergonomic API for consuming chunks
- Type-safe chunk parsing

**Usage**:
```rust
let mut stream = client
    .chat()
    .model("anthropic/claude-3-5-sonnet")
    .stream()
    .messages(vec![Message::user("Tell me a story")])
    .send_stream()
    .await?;

while let Some(chunk) = stream.next().await {
    let chunk = chunk?;
    if let Some(content) = chunk.content() {
        print!("{}", content);
        stdout().flush()?;
    }
}
```

### 5. Error Handling Pattern

**Pattern**: Canonical error struct with thiserror (M-ERRORS-CANONICAL-STRUCTS)

**Implementation** (Draft):
```rust
use thiserror::Error;
use std::backtrace::Backtrace;

#[derive(Debug)]
pub struct OpenRouterError {
    kind: ErrorKind,
    backtrace: Backtrace,
}

#[derive(Debug, Error)]
pub enum ErrorKind {
    #[error("HTTP error: {0}")]
    Http(#[from] reqwest::Error),
    
    #[error("API error: {code} - {message}")]
    Api {
        code: u16,
        message: String,
    },
    
    #[error("Rate limit exceeded. Retry after {retry_after:?}")]
    RateLimited {
        retry_after: Option<Duration>,
    },
    
    #[error("Invalid request: {0}")]
    InvalidRequest(String),
    
    #[error("Streaming error: {0}")]
    Stream(String),
    
    #[error("Missing API key")]
    MissingApiKey,
    
    #[error("Serialization error: {0}")]
    Serialization(#[from] serde_json::Error),
}

impl OpenRouterError {
    pub fn is_rate_limited(&self) -> bool {
        matches!(self.kind, ErrorKind::RateLimited { .. })
    }
    
    pub fn is_retryable(&self) -> bool {
        matches!(
            self.kind,
            ErrorKind::Http(_) | ErrorKind::RateLimited { .. }
        )
    }
    
    pub fn retry_after(&self) -> Option<Duration> {
        match &self.kind {
            ErrorKind::RateLimited { retry_after } => *retry_after,
            _ => None,
        }
    }
}
```

**Rationale**:
- Structured errors for API consumers
- Backtraces for debugging
- Helper methods for error handling
- Follows workspace error pattern

**Usage**:
```rust
match client.chat().send().await {
    Err(e) if e.is_rate_limited() => {
        if let Some(duration) = e.retry_after() {
            tokio::time::sleep(duration).await;
            // retry
        }
    }
    Err(e) if e.is_retryable() => {
        // retry logic
    }
    Err(e) => return Err(e),
    Ok(response) => { /* success */ }
}
```

### 6. Cost Tracking Pattern

**Pattern**: Response header parsing with accumulation

**Implementation** (Draft):
```rust
#[derive(Debug, Clone)]
pub struct Usage {
    pub prompt_tokens: u32,
    pub completion_tokens: u32,
    pub total_tokens: u32,
    pub cost: Option<f64>,  // Parsed from X-OpenRouter-Cost header
}

impl Usage {
    pub fn total_cost(&self) -> f64 {
        self.cost.unwrap_or(0.0)
    }
}

pub struct CostTracker {
    requests: AtomicU64,
    total_cost: AtomicU64,  // Store as integer cents to avoid float atomics
}

impl CostTracker {
    pub fn record(&self, usage: &Usage) {
        self.requests.fetch_add(1, Ordering::Relaxed);
        if let Some(cost) = usage.cost {
            let cents = (cost * 100.0) as u64;
            self.total_cost.fetch_add(cents, Ordering::Relaxed);
        }
    }
    
    pub fn total_requests(&self) -> u64 {
        self.requests.load(Ordering::Relaxed)
    }
    
    pub fn total_cost(&self) -> f64 {
        let cents = self.total_cost.load(Ordering::Relaxed);
        cents as f64 / 100.0
    }
}
```

**Rationale**:
- Extract cost from OpenRouter response headers
- Thread-safe accumulation with atomics
- Per-request and aggregate tracking
- Optional feature (can disable)

**Usage**:
```rust
let response = client.chat().send().await?;
println!("Request cost: ${:.6}", response.usage().total_cost());

// Later, check total
println!("Total spent: ${:.2}", client.total_cost());
```

### 7. Retry Logic with Exponential Backoff

**Pattern**: Configurable retry with exponential backoff

**Implementation** (Draft):
```rust
#[derive(Debug, Clone)]
pub struct RetryConfig {
    pub max_retries: u32,
    pub initial_delay: Duration,
    pub max_delay: Duration,
    pub multiplier: f32,
}

impl Default for RetryConfig {
    fn default() -> Self {
        Self {
            max_retries: 3,
            initial_delay: Duration::from_millis(100),
            max_delay: Duration::from_secs(10),
            multiplier: 2.0,
        }
    }
}

async fn retry_request<T, F, Fut>(
    config: &RetryConfig,
    mut f: F,
) -> Result<T>
where
    F: FnMut() -> Fut,
    Fut: Future<Output = Result<T>>,
{
    let mut attempt = 0;
    let mut delay = config.initial_delay;
    
    loop {
        match f().await {
            Ok(result) => return Ok(result),
            Err(e) if e.is_retryable() && attempt < config.max_retries => {
                attempt += 1;
                tokio::time::sleep(delay).await;
                delay = std::cmp::min(
                    Duration::from_secs_f32(delay.as_secs_f32() * config.multiplier),
                    config.max_delay,
                );
            }
            Err(e) => return Err(e),
        }
    }
}
```

**Rationale**:
- Handles transient failures gracefully
- Exponential backoff prevents overwhelming server
- Configurable per client
- Respects rate limit hints

## Key Technical Decisions

### Decision: reqwest for HTTP Client

**Context**: Need reliable HTTP client with async support, connection pooling, and streaming.

**Options Considered**:
1. `reqwest` - Popular, feature-rich, maintained
2. `hyper` directly - Lower-level, more control
3. Custom HTTP client - Full control but high effort

**Decision**: Use `reqwest`

**Rationale**:
- Mature, widely adopted in Rust ecosystem
- Built-in connection pooling and timeout handling
- Good streaming support (SSE via event-stream)
- Excellent error messages
- Active maintenance

**Tradeoffs**:
- ✅ Batteries-included (less code to write)
- ✅ Well-documented and tested
- ⚠️ Additional dependency (~2MB compile time)
- ⚠️ Less control than hyper directly

**References**:
- reqwest docs: https://docs.rs/reqwest
- Implementation: `src/http/client.rs`

### Decision: Sealed Enum for Message Types

**Context**: Need to represent different message types (system, user, assistant, function).

**Options Considered**:
1. Sealed enum with variants
2. Trait-based hierarchy
3. Struct with `role: String` field

**Decision**: Sealed enum with variants

**Rationale**:
- Type-safe at compile-time
- Exhaustive pattern matching
- Efficient serialization with serde
- Prevents invalid message types
- Clear, idiomatic Rust

**Tradeoffs**:
- ✅ Type safety
- ✅ Performance (no dynamic dispatch)
- ✅ Clear API
- ⚠️ Less extensible (but API is stable)

**References**:
- Implementation: `src/types/message.rs`

### Decision: tokio-stream for Streaming Abstraction

**Context**: Need async stream abstraction for SSE-based streaming responses.

**Options Considered**:
1. tokio-stream + eventsource-stream
2. async-stream macros
3. Custom stream implementation

**Decision**: tokio-stream with eventsource-stream for SSE parsing

**Rationale**:
- Standard async stream trait
- Integrates with tokio ecosystem
- `eventsource-stream` handles SSE parsing
- Composable with other stream utilities

**Tradeoffs**:
- ✅ Standard ecosystem tools
- ✅ Well-tested SSE parsing
- ✅ Composable streams
- ⚠️ Two dependencies (but both small)

**References**:
- tokio-stream: https://docs.rs/tokio-stream
- eventsource-stream: https://docs.rs/eventsource-stream
- Implementation: `src/stream/`

### Decision: Canonical Error Struct Pattern

**Context**: Library needs actionable error types for API consumers.

**Options Considered**:
1. Flat enum with all errors
2. Hierarchical error types
3. Canonical struct with ErrorKind (workspace pattern)

**Decision**: Canonical struct with ErrorKind enum

**Rationale**:
- Follows workspace shared pattern
- Structured errors with backtraces
- Helper methods for error categorization
- Consistent with protocols-mcp

**Tradeoffs**:
- ✅ Actionable for consumers
- ✅ Debuggable with backtraces
- ✅ Workspace consistency
- ⚠️ More verbose than simple enum

**References**:
- Workspace pattern: `../../workspace/shared-patterns.md#error-handling-strategy`
- Implementation: `src/types/error.rs`

## OpenRouter-Specific Patterns

### Authentication Header Pattern

**Pattern**: Bearer token authentication via Authorization header

```rust
impl OpenRouterClient {
    fn add_auth_header(&self, request: reqwest::RequestBuilder) -> reqwest::RequestBuilder {
        request.header(
            "Authorization",
            format!("Bearer {}", self.api_key)
        )
    }
}
```

**Key Points**:
- Simple bearer token authentication
- Header: `Authorization: Bearer sk-or-v1-...`
- API keys from OpenRouter dashboard
- No OAuth2 or complex auth flows

### Cost Header Parsing Pattern

**Pattern**: Extract cost from response headers

```rust
fn parse_cost_from_headers(headers: &reqwest::header::HeaderMap) -> Option<f64> {
    headers
        .get("X-OpenRouter-Cost")
        .and_then(|v| v.to_str().ok())
        .and_then(|s| s.parse::<f64>().ok())
}
```

**Key Points**:
- OpenRouter returns cost in `X-OpenRouter-Cost` header
- Format: Dollar amount as string (e.g., "0.000123")
- Parse and include in Usage struct
- Optional (header may not be present)

### Provider Routing Pattern

**Pattern**: Model string specifies provider

```rust
// User specifies provider in model string
client.chat()
    .model("openai/gpt-4")        // OpenAI provider
    .model("anthropic/claude-3-5-sonnet")  // Anthropic provider
    .model("google/gemini-pro")    // Google provider
```

**Key Points**:
- Format: `provider/model-name`
- OpenRouter routes to appropriate provider
- No additional configuration needed
- Can specify fallbacks (future enhancement)

## Performance Patterns

### Connection Pooling

**Pattern**: reqwest built-in connection pooling

```rust
let http_client = reqwest::Client::builder()
    .pool_max_idle_per_host(10)
    .pool_idle_timeout(Duration::from_secs(90))
    .build()?;
```

**Benefits**:
- Reuse connections across requests
- Reduce latency (no TLS handshake)
- Configurable pool size
- Automatic connection management

### Streaming with Zero-Copy

**Pattern**: Stream chunks without buffering entire response

```rust
// Stream processes chunks as they arrive
while let Some(chunk) = stream.next().await {
    process_chunk(chunk?);  // No buffering
}
```

**Benefits**:
- Constant memory usage
- Lower latency (first token faster)
- Better user experience
- Efficient for large responses

## Security Patterns

### API Key Management

**Pattern**: Environment variable or secure storage

```rust
// Recommended: From environment variable
let api_key = env::var("OPENROUTER_API_KEY")
    .expect("OPENROUTER_API_KEY not set");

let client = OpenRouterClient::builder()
    .api_key(api_key)
    .build()?;
```

**Best Practices**:
- Never hardcode API keys
- Use environment variables
- Rotate keys periodically
- Use least-privilege keys

### Input Validation

**Pattern**: Validate inputs before sending to API

```rust
impl ChatRequestBuilder {
    fn validate(&self) -> Result<()> {
        if self.messages.is_empty() {
            return Err(Error::invalid_request("Messages cannot be empty"));
        }
        
        if self.model.is_none() {
            return Err(Error::invalid_request("Model must be specified"));
        }
        
        if let Some(temp) = self.temperature {
            if temp < 0.0 || temp > 2.0 {
                return Err(Error::invalid_request(
                    "Temperature must be between 0.0 and 2.0"
                ));
            }
        }
        
        Ok(())
    }
}
```

**Benefits**:
- Fail fast with clear errors
- Prevent wasted API calls
- Better error messages
- Type safety at compile-time

## Testing Patterns

### Mock HTTP Client (Planned)

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use wiremock::{MockServer, Mock, ResponseTemplate};
    
    #[tokio::test]
    async fn test_chat_completion() {
        let mock_server = MockServer::start().await;
        
        Mock::given(method("POST"))
            .and(path("/chat/completions"))
            .respond_with(ResponseTemplate::new(200).set_body_json(json!({
                "id": "test-123",
                "model": "openai/gpt-4",
                "choices": [{"message": {"content": "Hello!"}}],
            })))
            .mount(&mock_server)
            .await;
        
        let client = OpenRouterClient::builder()
            .api_key("test-key")
            .base_url(mock_server.uri())
            .build()
            .unwrap();
        
        let response = client
            .chat()
            .model("openai/gpt-4")
            .messages(vec![Message::user("Hi")])
            .send()
            .await
            .unwrap();
        
        assert_eq!(response.content(), "Hello!");
    }
}
```

### Integration Tests (Planned)

```rust
#[tokio::test]
#[ignore]  // Requires API key
async fn test_real_api_call() {
    let client = OpenRouterClient::builder()
        .api_key(env::var("OPENROUTER_API_KEY").unwrap())
        .build()
        .unwrap();
    
    let response = client
        .chat()
        .model("openai/gpt-3.5-turbo")
        .messages(vec![Message::user("Say hello")])
        .send()
        .await
        .unwrap();
    
    assert!(!response.content().is_empty());
}
```

## Documentation Patterns

### Module Documentation (M-MODULE-DOCS)

```rust
//! # OpenRouter API Client
//!
//! This crate provides a production-ready Rust client for the OpenRouter API.
//!
//! ## Quick Start
//!
//! ```rust
//! use airsprotocols_api_openrouter::{OpenRouterClient, Message};
//!
//! #[tokio::main]
//! async fn main() -> Result<()> {
//!     let client = OpenRouterClient::builder()
//!         .api_key(env::var("OPENROUTER_API_KEY")?)
//!         .build()?;
//!
//!     let response = client
//!         .chat()
//!         .model("openai/gpt-4")
//!         .messages(vec![Message::user("Hello!")])
//!         .send()
//!         .await?;
//!
//!     println!("{}", response.content());
//!     Ok(())
//! }
//! ```
```

### Function Documentation (M-CANONICAL-DOCS)

```rust
/// Sends a chat completion request to OpenRouter.
///
/// # Examples
///
/// ```rust
/// let response = client
///     .chat()
///     .model("openai/gpt-4")
///     .messages(vec![Message::user("Hello")])
///     .send()
///     .await?;
/// ```
///
/// # Errors
///
/// Returns `OpenRouterError` if:
/// - Network request fails
/// - API returns an error response
/// - Response cannot be parsed
/// - Rate limit is exceeded
///
/// # Cost Tracking
///
/// The response includes token usage and cost information parsed from
/// the `X-OpenRouter-Cost` response header.
pub async fn send(&self) -> Result<ChatResponse> {
    // ...
}
```

## Related Documentation

- **Workspace Patterns**: `../../workspace/shared-patterns.md` - Core implementation patterns
- **Architecture**: `../../workspace/workspace-architecture.md` - Overall workspace structure
- **Tech Context**: `tech-context.md` - Technologies and constraints
- **Rust Guidelines**: `../../../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md` - Authoritative Rust standards

---

These patterns ensure consistency, quality, and maintainability in the api-openrouter codebase.
