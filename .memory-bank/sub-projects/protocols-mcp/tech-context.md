# protocols-mcp - Tech Context

**Last Updated**: 2025-12-17  
**Version**: 1.0.0-rc.1

## Technology Stack

### Core Language & Runtime

**Rust**: 1.88.0+
- **Why**: Type safety, memory safety, zero-cost abstractions, performance
- **MSRV**: 1.88.0 (workspace standard)
- **Edition**: 2021

**Tokio**: ^1.41.1 (Async Runtime)
- **Why**: Industry-standard async runtime, excellent ecosystem support
- **Features**: `full` (all features enabled)
- **Alternatives Considered**: async-std (rejected - less ecosystem support)

### Serialization & Data

**serde**: ^1.0.215
- **Why**: De-facto standard for Rust serialization
- **Features**: `derive` for automatic implementations

**serde_json**: ^1.0.133
- **Why**: JSON-RPC 2.0 requires JSON serialization
- **Performance**: Zero-copy deserialization where possible

### HTTP & Web

**axum**: ^0.8.1
- **Why**: Modern, ergonomic web framework built on tokio
- **Features**: Excellent middleware support, type-safe extractors

**hyper**: ^1.5.1
- **Why**: Low-level HTTP implementation, used by axum
- **Features**: HTTP/1 and HTTP/2 support

**tower**: ^0.5.2
- **Why**: Middleware and service abstractions
- **Features**: Composable services, extensive middleware

**tower-http**: ^0.6.2
- **Why**: HTTP-specific middleware (CORS, tracing, compression)

**reqwest**: ^0.12.12
- **Why**: Client-side HTTP requests (for OAuth2 JWKS fetching)
- **Features**: JSON support, async

### Authentication & Security

**oauth2**: ^4.4.2
- **Why**: OAuth 2.0 client implementation
- **Features**: Authorization code flow, token management

**jsonwebtoken**: ^9.3.0
- **Why**: JWT encoding/decoding and validation
- **Features**: JWKS support, multiple algorithms

**base64**: ^0.22.1
- **Why**: Base64 encoding/decoding for tokens
- **Features**: Standard base64 implementation

**url**: ^2.5.4
- **Why**: URL parsing and manipulation
- **Features**: Serde support for URL types

### Utilities

**async-trait**: ^0.1.83
- **Why**: Async methods in traits (until Rust stabilizes async traits)
- **Tradeoff**: Small allocation for async trait methods

**thiserror**: ^2.0.9
- **Why**: Derive macro for error types (M-ERRORS-CANONICAL-STRUCTS)
- **Alternative**: anyhow (used in applications, not libraries)

**tracing**: ^0.1.41
- **Why**: Structured logging and instrumentation
- **Features**: Async support, OpenTelemetry integration

**tracing-subscriber**: ^0.3.19
- **Why**: Tracing output formatting and filtering
- **Features**: JSON formatter, environment filter

**chrono**: ^0.4.39
- **Why**: Date and time handling
- **Features**: Serde support, timezone handling

**uuid**: ^1.11.0
- **Why**: Unique identifier generation
- **Features**: V4 (random) UUIDs, serde support

**bytes**: ^1.9.0
- **Why**: Efficient byte buffer handling
- **Features**: Zero-copy operations

**tokio-util**: ^0.7.13
- **Why**: Tokio utilities (codecs, time)
- **Features**: Codec helpers for transport

**dashmap**: ^6.1.0
- **Why**: Concurrent hashmap for caching
- **Features**: Lock-free reads, concurrent writes

**futures**: ^0.3.31
- **Why**: Additional future combinators
- **Features**: Stream utilities

**tokio-stream**: ^0.1.17
- **Why**: Stream utilities for tokio
- **Features**: Stream adapters

**deadpool**: ^0.12.1
- **Why**: Connection pooling (for HTTP transport)
- **Features**: Async pool management

### Development Dependencies

**tokio-test**: ^0.4.4
- **Why**: Testing utilities for tokio-based code
- **Features**: Async test helpers

**proptest**: ^1.5.0
- **Why**: Property-based testing
- **Use**: Concurrency testing, message validation

**criterion**: ^0.5.1
- **Why**: Micro-benchmarking framework
- **Use**: Performance regression detection

**wiremock**: ^0.6.2
- **Why**: HTTP mocking for tests
- **Use**: OAuth2 JWKS endpoint testing

**rsa**: ^0.9.7
- **Why**: RSA key generation for OAuth2 examples
- **Use**: Dev dependencies only

**sha2**: ^0.10.8
- **Why**: SHA-256 hashing for OAuth2
- **Use**: Dev dependencies only

**tempfile**: ^3.14.0
- **Why**: Temporary file/directory creation for tests
- **Use**: Testing file-based configuration

**num_cpus**: ^1.16.0
- **Why**: CPU count detection for benchmarks
- **Use**: Benchmark parallelism tuning

### CLI Dependencies (Examples Only)

**clap**: ^4.5.23
- **Why**: Command-line argument parsing
- **Features**: Derive API for declarative CLI
- **Use**: Example applications only

## Development Environment

### Required Tools

**Rust Toolchain**: 1.88.0+
```bash
rustup update stable
rustup default stable
```

**Cargo**: Bundled with Rust
- Build system and package manager
- Workspace management

**Optional Tools**:
- `cargo-watch` - Auto-recompile on file changes
- `cargo-expand` - Macro expansion for debugging
- `cargo-audit` - Security vulnerability scanning
- `cargo-outdated` - Dependency update checking

### Editor/IDE Setup

**VS Code** (Recommended):
- **rust-analyzer**: LSP for Rust
- **CodeLLDB**: Debugging support
- **Even Better TOML**: TOML syntax highlighting
- **crates**: Cargo.toml dependency management

**IntelliJ IDEA/CLion**:
- **Rust Plugin**: Official JetBrains Rust support

**Vim/Neovim**:
- **rust.vim**: Rust syntax highlighting
- **coc-rust-analyzer**: LSP integration

### Build Configuration

**Development Build**:
```bash
cargo build --workspace
```

**Release Build**:
```bash
cargo build --release --workspace
```

**Feature Flags**:
```toml
[features]
default = ["stdio", "http"]
stdio = []
http = ["axum", "hyper", "tower", "tower-http"]
oauth2 = ["oauth2-crate", "jsonwebtoken"]
test-util = []
```

**Build Profile** (workspace-level):
```toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
```

## Testing Infrastructure

### Unit Tests
```bash
cargo test --lib
```

### Integration Tests
```bash
cargo test --test '*'
```

### Property Tests
```bash
cargo test --release  # Property tests run faster in release mode
```

### Benchmarks
```bash
cargo bench
```

### Test Coverage
```bash
cargo tarpaulin --workspace --out Html
```

## Performance Characteristics

### Message Processing
- **Latency**: <1ms average per message
- **Throughput**: 10,000+ messages/second
- **Memory**: <10MB baseline, scales linearly

### Transport Performance
- **Stdio**: <100μs overhead
- **HTTP**: <5ms overhead (including network)
- **SSE**: <10ms for stream setup

### Optimization Techniques
1. **Zero-Copy Deserialization**: Use `&str` over `String` where possible
2. **Connection Pooling**: Reuse HTTP connections
3. **Token Caching**: Cache validated JWT keys
4. **Buffer Reuse**: Reuse buffers in transport loops

## Technical Constraints

### Language Constraints
- **Rust Edition**: 2021 (latest stable features)
- **MSRV**: 1.88.0 (cannot use features from newer versions)
- **no_std**: Not supported (requires std for tokio)

### Runtime Constraints
- **Async Runtime**: Tokio only (no async-std support)
- **Thread Model**: Work-stealing thread pool
- **Blocking Operations**: Must use `tokio::task::spawn_blocking`

### Platform Constraints
- **Operating Systems**: Linux, macOS, Windows
- **Architectures**: x86_64, aarch64 (ARM64)
- **Minimum Requirements**: 
  - 2 CPU cores recommended
  - 100MB RAM minimum
  - Network connectivity for HTTP transport

### Security Constraints
- **TLS**: Required for production HTTP deployments
- **Token Storage**: Tokens must not be logged
- **Key Rotation**: OAuth2 JWKS refresh every 24 hours
- **Rate Limiting**: Recommended for production (not built-in yet)

### Performance Constraints
- **Message Size**: Recommended max 10MB per message
- **Concurrent Requests**: 100+ supported, tested to 1000+
- **Memory Usage**: <10MB + (requests × avg_message_size)

## External Dependencies

### Protocol Specifications
- **MCP Specification**: https://modelcontextprotocol.io/
- **JSON-RPC 2.0**: https://www.jsonrpc.org/specification

### Security Standards
- **OAuth 2.0**: RFC 6749
- **JWT**: RFC 7519
- **JWKS**: RFC 7517

### Related Projects
- **Python MCP SDK**: Official reference implementation
- **TypeScript MCP SDK**: Official reference implementation
- **Claude Desktop**: Primary consumer of MCP servers

## Development Workflow

### Local Development

**1. Clone and Build**:
```bash
git clone https://github.com/airsstack/airsprotocols.git
cd airsprotocols/protocols/mcp
cargo build
```

**2. Run Tests**:
```bash
cargo test --workspace
```

**3. Run Examples**:
```bash
cargo run --example stdio-server-integration
```

**4. Documentation**:
```bash
cargo doc --open --no-deps
```

### Continuous Integration

**GitHub Actions**:
- Build and test on Linux, macOS, Windows
- Clippy linting
- Rustfmt formatting check
- Security audit (cargo-audit)
- Documentation build

### Release Process

**1. Version Bump**:
```toml
[package]
version = "1.0.0-rc.2"  # Update in Cargo.toml
```

**2. Changelog**:
```markdown
## [1.0.0-rc.2] - 2025-01-15
### Added
- WebSocket transport support
...
```

**3. Tag and Publish**:
```bash
git tag -a v1.0.0-rc.2 -m "Release 1.0.0-rc.2"
git push origin v1.0.0-rc.2
cargo publish --dry-run
cargo publish
```

## Monitoring & Observability

### Logging

**Framework**: `tracing` with structured logging

**Log Levels**:
- `ERROR`: Critical failures requiring immediate attention
- `WARN`: Important but non-critical issues
- `INFO`: High-level operational information
- `DEBUG`: Detailed diagnostic information
- `TRACE`: Very detailed trace information

**Configuration**:
```rust
tracing_subscriber::fmt()
    .with_env_filter("airsprotocols_mcp=debug")
    .with_target(false)
    .init();
```

### Metrics (Future)

**Planned**: OpenTelemetry integration for metrics

**Metrics to Track**:
- Message processing latency (p50, p95, p99)
- Request throughput (requests/second)
- Error rates by type
- Connection pool usage
- Memory usage trends

### Tracing (Future)

**Planned**: Distributed tracing with OpenTelemetry

**Trace Points**:
- Message handling lifecycle
- Transport send/receive
- Authentication/authorization
- Provider invocations

## Known Issues & Limitations

### Current Limitations

1. **WebSocket Transport**: Not yet implemented (planned Q1 2025)
2. **Rate Limiting**: No built-in rate limiting (planned Q1 2025)
3. **Metrics**: No built-in metrics collection yet
4. **Multi-Protocol**: Only MCP supported (A2A planned)

### Workarounds

1. **Rate Limiting**: Implement at application level or use reverse proxy
2. **Metrics**: Use tracing events for now, parse logs
3. **WebSocket**: Use HTTP with SSE for now, or contribute WebSocket transport

### Technical Debt

See `docs/debts/` folder for technical debt tracking.

## Migration Guides

### Upgrading from v0.2.x to v1.0.0-rc.1

**Breaking Changes**:
- None (RC maintains compatibility with v0.2.x)

**New Features**:
- Enhanced OAuth2 support
- Improved error messages
- Better builder APIs

**Deprecations**:
- None

## Related Documentation

- **System Patterns**: `system-patterns.md` - Architecture and design patterns
- **Active Context**: `active-context.md` - Current development status
- **Workspace Patterns**: `../../workspace/shared-patterns.md` - Core patterns
- **Rust Guidelines**: `../../../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md` - Coding standards

---

This tech context provides the technical foundation for understanding and contributing to protocols-mcp.
