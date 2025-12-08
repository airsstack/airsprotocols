# Getting Started

This guide will help you get up and running with AIRS Protocols quickly.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Rust** 1.88 or later ([install instructions](https://www.rust-lang.org/tools/install))
- **Cargo** (comes with Rust)
- A code editor with Rust support (e.g., VS Code with rust-analyzer)

### Verify Installation

```bash
rustc --version
cargo --version
```

## Installation

### Using MCP (Model Context Protocol)

Add AIRS Protocols MCP to your `Cargo.toml`:

```toml
[dependencies]
# From git repository (current):
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }

# Or when published to crates.io:
# airsprotocols-mcp = "1.0.0-rc.1"

# Required async runtime:
tokio = { version = "1.47", features = ["full"] }
```

### Future Packages

Other packages will be available as they're released:

```toml
[dependencies]
# Coming soon:
# airsprotocols-a2a = "0.1.0"
# airsprotocols-anthropic = "0.1.0"
# airsprotocols-openai = "0.1.0"
```

## Quick Start: MCP Client

Here's a minimal example of creating an MCP client:

```rust
use airsprotocols_mcp::McpClientBuilder;
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create an MCP client
    let client = McpClientBuilder::new()
        .client_info("my-app", "0.1.0")
        .timeout(Duration::from_secs(30))
        .build(transport)  // transport implementation needed
        .await?;

    // List available tools from the server
    let tools = client.list_tools().await?;
    println!("Available tools: {}", tools.len());

    Ok(())
}
```

!!! note "Transport Required"
    The example above requires a transport implementation. See [MCP Quick Start](protocols/mcp/quick-start.md) for complete examples with stdio and HTTP transports.

## Quick Start: MCP Server

Create a simple MCP server that provides tools:

```rust
use std::sync::Arc;
use async_trait::async_trait;
use serde_json::json;
use airsprotocols_mcp::protocol::{JsonRpcMessage, MessageContext, MessageHandler};
use airsprotocols_mcp::providers::{ToolProvider, ToolCall, ToolResult};
use airsprotocols_mcp::transport::adapters::stdio::StdioTransport;
use airsprotocols_mcp::IntegrationError;

// Implement a simple tool provider
struct MyToolProvider;

#[async_trait]
impl ToolProvider for MyToolProvider {
    async fn call_tool(&self, tool_call: ToolCall) -> Result<ToolResult, IntegrationError> {
        match tool_call.name.as_str() {
            "add" => {
                let params = tool_call.params.unwrap_or_default();
                let a = params["a"].as_i64().unwrap_or(0);
                let b = params["b"].as_i64().unwrap_or(0);
                let result = a + b;
                Ok(ToolResult::success(json!({ "result": result })))
            }
            _ => Err(IntegrationError::MethodNotFound(tool_call.name))
        }
    }
    
    // Other required methods...
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let tool_provider = Arc::new(MyToolProvider);
    // Set up handler and transport...
    
    Ok(())
}
```

For complete server examples, see [MCP Server Implementation](protocols/mcp/protocol/server.md).

## Project Structure

When working with AIRS Protocols in your project:

```
my-project/
├── Cargo.toml              # Dependencies
├── src/
│   ├── main.rs            # Application entry point
│   └── ...
└── examples/              # Optional examples
    └── ...
```

### Recommended Project Setup

```toml
[package]
name = "my-ai-app"
version = "0.1.0"
edition = "2021"

[dependencies]
# AIRS Protocols
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }

# Async runtime
tokio = { version = "1.47", features = ["full"] }

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# Error handling
thiserror = "1.0"

# Logging
tracing = "0.1"
tracing-subscriber = "0.3"
```

## Common Patterns

### Error Handling

Use Rust's Result type for error handling:

```rust
use airsprotocols_mcp::IntegrationError;

async fn my_function() -> Result<(), IntegrationError> {
    // Your code here
    Ok(())
}
```

### Async/Await

All AIRS Protocols APIs are async:

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = create_client().await?;
    let result = client.list_tools().await?;
    Ok(())
}
```

### Logging

Add logging to your application:

```rust
use tracing::{info, error};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Initialize logging
    tracing_subscriber::fmt::init();
    
    info!("Starting application");
    
    // Your code here
    
    Ok(())
}
```

## Next Steps

### Learn More About MCP

- **[MCP Quick Start](protocols/mcp/quick-start.md)** - Detailed MCP tutorial
- **[Basic Examples](protocols/mcp/usage/basic-examples.md)** - Common usage patterns
- **[Advanced Patterns](protocols/mcp/usage/advanced-patterns.md)** - Advanced techniques

### Explore the Architecture

- **[Architecture Overview](architecture.md)** - System design
- **[MCP Architecture](protocols/mcp/architecture/core.md)** - MCP internals

### See Examples

- **[Examples Overview](examples/index.md)** - Runnable examples
- **[Claude Integration](protocols/mcp/usage/claude-integration.md)** - Integrate with Claude Desktop

### API Reference

- **[MCP API Reference](protocols/mcp/api-reference.md)** - Complete API documentation
- **[docs.rs](https://docs.rs/airsprotocols-mcp)** - Generated Rust documentation

## Development Workflow

### Building Your Project

```bash
# Build in debug mode
cargo build

# Build optimized release
cargo build --release

# Run your application
cargo run

# Run with logging
RUST_LOG=info cargo run
```

### Testing

```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run with output
cargo test -- --nocapture
```

### Code Quality

```bash
# Format code
cargo fmt

# Check for issues
cargo clippy

# Check without building
cargo check
```

## Troubleshooting

### Common Issues

#### Compilation Errors

Make sure you're using Rust 1.88 or later:

```bash
rustup update
```

#### Async Runtime Not Found

Ensure tokio is in your dependencies:

```toml
[dependencies]
tokio = { version = "1.47", features = ["full"] }
```

#### Transport Connection Issues

Check that your transport is properly configured and the server is running.

### Getting Help

If you encounter issues:

1. Check the [documentation](protocols/mcp/index.md)
2. Review [examples](examples/index.md)
3. Search [existing issues](https://github.com/airsstack/airsprotocols/issues)
4. Ask in [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)
5. Open a [new issue](https://github.com/airsstack/airsprotocols/issues/new)

## Resources

### Documentation

- [MCP Protocol Documentation](protocols/mcp/index.md)
- [Architecture Guide](architecture.md)
- [API Reference](protocols/mcp/api-reference.md)

### External Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Rust Book](https://doc.rust-lang.org/book/)
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)

### Community

- [GitHub Repository](https://github.com/airsstack/airsprotocols)
- [Issue Tracker](https://github.com/airsstack/airsprotocols/issues)
- [Discussions](https://github.com/airsstack/airsprotocols/discussions)

---

Ready to dive deeper? Continue to the [MCP Quick Start](protocols/mcp/quick-start.md) for a complete tutorial!
