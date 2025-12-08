# Installation

## Adding to Your Project

Add `airsprotocols-mcp` to your project's `Cargo.toml`:

```toml
[dependencies]
# From git repository:
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }

# Or when published to crates.io:
# airsprotocols-mcp = "1.0.0-rc.1"
```

## Required Dependencies

For most use cases, you'll also want to include:

```toml
[dependencies]
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
tokio = { version = "1.35", features = ["full"] }
serde_json = "1.0"
```

## Building from Source

To build the library from source:

```bash
# Clone the repository
git clone https://github.com/airsstack/airsprotocols
cd airsprotocols

# Build the MCP protocol implementation
cargo build --package airsprotocols-mcp

# Run tests
cargo test --package airsprotocols-mcp
```

## Documentation Setup

This crate includes comprehensive documentation. The source files for the documentation are located in the `docs/src` directory within this crate.

### Building Documentation Locally

To build and view the documentation locally:

```bash
# Navigate to the documentation directory
cd docs

# Serve the documentation (auto-reloads on changes)
mkdocs serve
```

This will make the documentation available at `http://localhost:8000`. The server will automatically rebuild the documentation and reload your browser whenever you make changes to the source files.

## Next Steps

- [Quick Start Guide](quick-start.md) - Get started with your first MCP client
- [Basic Examples](usage/basic-examples.md) - Learn common usage patterns
- [API Reference](api-reference.md) - Explore the full API documentation
