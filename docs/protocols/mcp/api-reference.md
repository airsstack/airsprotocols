# API Reference

The complete API reference for AIRS MCP is available on [docs.rs](https://docs.rs).

## Rust API Documentation

Once published to crates.io, the full API documentation will be available at:

**https://docs.rs/airsprotocols-mcp**

## Building Documentation Locally

You can also build and view the API documentation locally using cargo:

```bash
# Build and open the API documentation
cargo doc --package airsprotocols-mcp --open

# Build documentation with all features
cargo doc --package airsprotocols-mcp --all-features --open
```

## Key Modules

### Integration Layer

- **`integration::client`** - High-level MCP client implementation
- **`integration::server`** - MCP server framework
- **`integration::builder`** - Builder patterns for client and server configuration

### Protocol Layer

- **`protocol::types`** - MCP message types and protocol definitions
- **`protocol::jsonrpc`** - JSON-RPC 2.0 implementation
- **`protocol::messages`** - Request and response message types

### Transport Layer

- **`transport`** - Transport abstraction and implementations
- **`transport::adapters::stdio`** - STDIO transport for local communication
- **`transport::adapters::http`** - HTTP transport for network communication

### Provider System

- **`providers::tool`** - Tool provider trait and implementations
- **`providers::resource`** - Resource provider trait and implementations
- **`providers::prompt`** - Prompt provider trait and implementations

### Authentication & Authorization

- **`authentication`** - Authentication strategies and middleware
- **`authorization`** - Authorization policies and access control

## Examples

The [examples directory](https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp/examples) contains complete working examples demonstrating various use cases:

- **stdio-server-integration** - Local server with STDIO transport
- **http-oauth2-server-integration** - HTTP server with OAuth2 authentication
- **basic-client** - Simple MCP client example
- **advanced-patterns** - Complex usage patterns and integrations

## Related Documentation

- [Quick Start Guide](quick-start.md) - Get started quickly
- [Usage Examples](usage/basic-examples.md) - Common usage patterns
- [Architecture Guide](architecture/core.md) - Internal design details
