# api-openrouter - Technical Context

**Last Updated**: 2025-12-17  
**Version**: 0.1.0 (Planning)  
**Status**: Planning Phase

## Technology Stack

### Core Languages & Frameworks

#### Rust
- **Version**: 1.88.0+ (MSRV - Minimum Supported Rust Version)
- **Edition**: 2021
- **Rationale**: Memory safety, performance, excellent async support
- **Constraints**: Must compile on stable Rust (no nightly features)

#### Async Runtime
- **Framework**: tokio
- **Version**: 1.x (latest stable)
- **Features**: `rt-multi-thread`, `macros`, `time`
- **Rationale**: Industry-standard async runtime, excellent ecosystem
- **Constraints**: All public futures must be `Send`

### Dependencies (Planned)

#### HTTP & Networking
```toml
[dependencies]
reqwest = { version = "0.12", features = ["json", "stream"] }
```
- **Purpose**: HTTP client for API requests
- **Features**: Connection pooling, streaming, TLS
- **Rationale**: Most popular Rust HTTP client, well-maintained
- **Alternative Considered**: hyper directly (lower-level, more complex)

#### Serialization
```toml
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```
- **Purpose**: JSON serialization/deserialization
- **Rationale**: De facto standard in Rust ecosystem
- **Constraints**: Must support OpenRouter JSON schema

#### Streaming
```toml
tokio-stream = "0.1"
eventsource-stream = "0.2"
```
- **Purpose**: Async stream utilities and SSE parsing
- **Rationale**: 
  - `tokio-stream`: Standard async stream trait
  - `eventsource-stream`: Battle-tested SSE parser
- **Alternative Considered**: `async-stream` (more macros, less flexibility)

#### Error Handling
```toml
thiserror = "1.0"
```
- **Purpose**: Derive Error trait for custom errors
- **Rationale**: Workspace standard, ergonomic error definitions
- **Pattern**: Canonical error struct (see System Patterns)

#### Optional Dependencies
```toml
tracing = { version = "0.1", optional = true }
```
- **Purpose**: Structured logging
- **Feature Flag**: `tracing`
- **Rationale**: Production observability, standard in Rust ecosystem

### Build Tools

#### Cargo
- **Version**: 1.88.0+ (matches Rust version)
- **Workspace**: Part of airsprotocols workspace
- **Features**: Feature flags for optional functionality

#### Feature Flags (Planned)
```toml
[features]
default = ["native-tls"]
native-tls = ["reqwest/native-tls"]
rustls = ["reqwest/rustls-tls"]
tracing = ["dep:tracing"]
```

## OpenRouter API

### API Overview

#### Base Information
- **Base URL**: `https://openrouter.ai/api/v1`
- **Protocol**: HTTP/1.1 and HTTP/2
- **Authentication**: Bearer token (API key)
- **Format**: JSON (request and response)
- **Versioning**: API version in URL path

#### API Documentation
- **Official Docs**: https://openrouter.ai/docs
- **OpenAI Compatibility**: Similar to OpenAI API (easy migration)
- **Specification**: RESTful HTTP API (no official OpenAPI spec)

### Authentication

#### API Key Authentication
```http
Authorization: Bearer sk-or-v1-...
```

**Key Format**: `sk-or-v1-{random}`  
**Obtaining Keys**: OpenRouter dashboard (https://openrouter.ai/keys)  
**Scopes**: Single API key for all operations  
**Rate Limiting**: Based on API key tier (Free, Starter, Pro, etc.)

**Security**:
- Keys should be stored securely (environment variables, secret managers)
- Never commit keys to version control
- Rotate keys periodically
- Use HTTPS only (enforced by OpenRouter)

### Endpoints

#### Chat Completions
```http
POST /api/v1/chat/completions
Content-Type: application/json
Authorization: Bearer {api_key}

{
  "model": "openai/gpt-4",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "stream": false
}
```

**Response** (Non-Streaming):
```json
{
  "id": "gen-abc123",
  "model": "openai/gpt-4",
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "Hello! How can I help you?"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 15,
    "total_tokens": 25
  }
}
```

**Response Headers**:
- `X-OpenRouter-Cost`: Dollar cost of request (e.g., "0.000123")
- `X-RateLimit-Limit`: Request limit per time window
- `X-RateLimit-Remaining`: Remaining requests in window
- `X-RateLimit-Reset`: Unix timestamp when limit resets

#### Streaming Completions
```http
POST /api/v1/chat/completions
Content-Type: application/json
Authorization: Bearer {api_key}

{
  "model": "openai/gpt-4",
  "messages": [...],
  "stream": true
}
```

**Response** (Server-Sent Events):
```
data: {"id":"gen-abc","choices":[{"delta":{"content":"Hello"}}]}

data: {"id":"gen-abc","choices":[{"delta":{"content":" there"}}]}

data: {"id":"gen-abc","choices":[{"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

**SSE Format**:
- Content-Type: `text/event-stream`
- Each event: `data: {json}\n\n`
- Final event: `data: [DONE]\n\n`
- Delta format: Incremental content chunks

#### Model Listing (Future)
```http
GET /api/v1/models
Authorization: Bearer {api_key}
```

**Response**:
```json
{
  "data": [{
    "id": "openai/gpt-4",
    "name": "GPT-4",
    "pricing": {
      "prompt": "0.00003",
      "completion": "0.00006"
    },
    "context_length": 8192
  }]
}
```

### Request Parameters

#### Common Parameters
- `model` (required): Model identifier (e.g., "openai/gpt-4")
- `messages` (required): Array of message objects
- `temperature` (optional): 0.0-2.0, controls randomness
- `max_tokens` (optional): Maximum completion length
- `top_p` (optional): Nucleus sampling parameter
- `stream` (optional): Enable streaming responses

#### OpenRouter-Specific Parameters
- `provider` (optional): Specify provider preferences
- `models` (optional): Array of fallback models
- `route` (optional): Routing strategy ("fallback", "least-cost")

### Rate Limiting

#### Rate Limit Tiers (Example)
- **Free**: 10 requests/minute, 100 requests/day
- **Starter**: 60 requests/minute, unlimited daily
- **Pro**: 200 requests/minute, unlimited daily

#### Handling Rate Limits
- **Status Code**: 429 Too Many Requests
- **Response Header**: `Retry-After` (seconds to wait)
- **Strategy**: Exponential backoff with jitter

**Example Error Response**:
```json
{
  "error": {
    "message": "Rate limit exceeded",
    "type": "rate_limit_exceeded",
    "code": 429
  }
}
```

### Error Handling

#### HTTP Status Codes
- `200`: Success
- `400`: Bad Request (invalid parameters)
- `401`: Unauthorized (invalid API key)
- `429`: Rate Limit Exceeded
- `500`: Server Error
- `503`: Service Unavailable

#### Error Response Format
```json
{
  "error": {
    "message": "Human-readable error message",
    "type": "error_type",
    "code": 400
  }
}
```

**Error Types**:
- `invalid_request_error`: Bad request parameters
- `authentication_error`: Invalid or missing API key
- `rate_limit_exceeded`: Too many requests
- `api_error`: OpenRouter internal error
- `provider_error`: Downstream provider error

## Development Environment

### Required Tools

#### Rust Toolchain
```bash
# Install rustup (if not installed)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install stable Rust 1.88.0+
rustup install stable
rustup default stable

# Verify installation
rustc --version  # Should be 1.88.0 or higher
cargo --version
```

#### Environment Variables
```bash
# Required: OpenRouter API key
export OPENROUTER_API_KEY="sk-or-v1-..."

# Optional: Custom base URL (for testing)
export OPENROUTER_BASE_URL="https://openrouter.ai/api/v1"

# Optional: Enable debug logging
export RUST_LOG="airsprotocols_api_openrouter=debug"
```

#### Testing Account
- **Sign Up**: https://openrouter.ai/
- **Free Tier**: Available for development and testing
- **Test Models**: Use cheaper models for testing (e.g., gpt-3.5-turbo)
- **Cost Tracking**: Monitor usage in OpenRouter dashboard

### Development Setup (Planned)

#### Project Structure
```
apis/openrouter/
├── Cargo.toml
├── README.md
├── src/
│   ├── lib.rs
│   ├── client.rs
│   ├── types/
│   ├── builder/
│   ├── api/
│   ├── http/
│   └── stream/
├── examples/
│   ├── basic_chat.rs
│   ├── streaming_chat.rs
│   └── multi_model.rs
├── tests/
│   ├── integration_tests.rs
│   └── mock_tests.rs
└── benches/
    └── request_performance.rs
```

#### Build Commands
```bash
# Check compilation
cargo check

# Run tests
cargo test

# Run with specific features
cargo test --features tracing

# Build for release
cargo build --release

# Generate documentation
cargo doc --open

# Run examples
cargo run --example basic_chat
```

### Testing Strategy

#### Unit Tests
- Mock HTTP responses with `wiremock` or `mockito`
- Test type serialization/deserialization
- Test builder validation logic
- Test error handling

#### Integration Tests
- Requires valid API key (env var)
- Tests marked with `#[ignore]` by default
- Run with `cargo test -- --ignored`
- Use cheaper models to minimize costs

#### Example Tests
```bash
# Run unit tests (no API key needed)
cargo test

# Run integration tests (requires API key)
cargo test -- --ignored

# Run specific test
cargo test test_chat_completion

# Run with debug output
cargo test -- --nocapture
```

### Performance Considerations

#### Benchmarking
```bash
# Run benchmarks
cargo bench

# Profile with flamegraph
cargo flamegraph --example basic_chat
```

#### Performance Targets
- **Client Overhead**: <100ms per request (beyond network latency)
- **Streaming Latency**: <50ms to first token
- **Memory**: <20MB baseline (excluding response data)
- **Throughput**: 100+ concurrent requests

## Platform Compatibility

### Supported Platforms
- **Linux**: x86_64, aarch64 (primary target)
- **macOS**: x86_64, aarch64 (M1/M2 Macs)
- **Windows**: x86_64 (MSVC toolchain)
- **Others**: Any platform with Rust stable support

### TLS Implementation
- **Default**: `native-tls` (platform-native TLS)
- **Alternative**: `rustls` (pure Rust TLS)
- **Feature Flags**: Choose via Cargo features

### Async Runtime
- **Required**: tokio (not compatible with async-std or smol)
- **Rationale**: Workspace standard, ecosystem dominance

## External Services

### OpenRouter Service
- **Availability**: 99.9% uptime SLA (Pro tier)
- **Regions**: Global (CloudFlare CDN)
- **Latency**: Varies by provider (100ms-2000ms typical)
- **Support**: Email support, community Discord

### Downstream Providers
OpenRouter routes to various LLM providers:
- OpenAI (GPT-4, GPT-3.5, etc.)
- Anthropic (Claude models)
- Google (Gemini models)
- Meta (Llama models)
- Mistral AI
- Many others

**Implication**: Latency and availability depend on downstream provider.

## Security Considerations

### API Key Security
- Store in environment variables or secure vaults
- Never log or expose API keys
- Rotate keys periodically
- Use least-privilege keys if available

### Network Security
- HTTPS only (enforced by OpenRouter)
- TLS 1.2+ required
- Certificate validation enabled by default
- Proxy support via reqwest

### Data Privacy
- All requests to OpenRouter are logged by OpenRouter
- Review OpenRouter privacy policy
- Consider data residency requirements
- OpenRouter may cache responses for optimization

## Monitoring & Observability

### Logging (Planned)
```rust
use tracing::{info, warn, error};

info!(
    model = model,
    prompt_tokens = usage.prompt_tokens,
    cost = usage.cost,
    "Chat completion successful"
);
```

**Log Levels**:
- `ERROR`: API errors, network failures
- `WARN`: Rate limiting, retries
- `INFO`: Successful requests, costs
- `DEBUG`: Request/response details
- `TRACE`: Internal operations

### Metrics (Future)
- Request count by model
- Average latency by model
- Cost tracking over time
- Error rate by type
- Retry counts

### Tracing (Future)
- Integration with OpenTelemetry
- Distributed tracing for multi-service setups
- Span for each API request

## Constraints & Limitations

### Technical Constraints
- Must use tokio async runtime (not async-std)
- MSRV: Rust 1.88.0+
- Requires network access to openrouter.ai
- Requires valid API key for usage

### API Limitations
- Rate limits based on account tier
- Token limits per request (model-dependent)
- Cost per request (varies by model)
- Streaming may not be supported by all models

### Design Constraints
- Must remain OpenRouter-compatible
- Cannot diverge from OpenRouter API schema
- Must handle provider-specific behaviors gracefully

## Related Documentation

- **Project Brief**: `project-brief.md`
- **Product Context**: `product-context.md`
- **System Patterns**: `system-patterns.md`
- **Active Context**: `active-context.md`
- **Progress**: `progress.md`
- **Workspace Patterns**: `../../workspace/shared-patterns.md`
- **Rust Guidelines**: `../../../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md`

## External Resources

### OpenRouter
- Homepage: https://openrouter.ai
- Documentation: https://openrouter.ai/docs
- API Keys: https://openrouter.ai/keys
- Model List: https://openrouter.ai/models
- Pricing: https://openrouter.ai/docs#pricing

### Rust Ecosystem
- reqwest: https://docs.rs/reqwest
- tokio: https://tokio.rs
- serde: https://serde.rs
- eventsource-stream: https://docs.rs/eventsource-stream

---

This technical context provides the foundation for implementation decisions and development setup.
