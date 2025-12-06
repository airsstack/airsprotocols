# airsprotocols

**Common AI Communication Protocols and LLM API Implementations in Rust**

A comprehensive Rust workspace providing production-ready implementations of agent communication protocols and LLM provider APIs. Built with performance, type safety, and interoperability in mind.

---

## 📦 What's Inside

### **Protocols** (`protocols/`)

Communication protocols for AI agents and intelligent systems:

- **[`airsprotocols-mcp`](protocols/mcp/)** - Model Context Protocol (MCP)
  - Connect AI models to tools, resources, and context
  - JSON-RPC 2.0 foundation with transport abstraction
  - Built-in authentication (API Key, OAuth2) and authorization
  - Stdio and HTTP transport implementations
  - **Status:** ✅ Available (v0.2.3)

- **`airsprotocols-a2a`** - Agent-to-Agent Protocol (A2A/ACP)
  - Enable communication between independent AI agents
  - Multiple protocol bindings (JSON-RPC, gRPC, REST)
  - Task lifecycle management and streaming support
  - **Status:** 🚧 Planned

### **APIs** (`apis/`)

LLM provider client implementations with unified interfaces:

- **`airsprotocols-anthropic`** - Anthropic Claude API client
  - **Status:** 🚧 Planned

- **`airsprotocols-openai`** - OpenAI GPT API client
  - **Status:** 🚧 Planned

- **`airsprotocols-google`** - Google Gemini API client
  - **Status:** 🚧 Planned

- **`airsprotocols-ollama`** - Ollama local models client
  - **Status:** 🚧 Planned

---

## 🏗️ Project Structure

```
airsprotocols/
├── protocols/          # Communication protocols
│   ├── mcp/           # Model Context Protocol
│   └── a2a/           # Agent-to-Agent Protocol
│
└── apis/              # LLM provider clients
    ├── anthropic/     # Claude API
    ├── openai/        # GPT API
    ├── google/        # Gemini API
    └── ollama/        # Local models
```

---

## 🚀 Quick Start

### Using MCP (Model Context Protocol)

Add to your `Cargo.toml`:

```toml
[dependencies]
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
# Or when published to crates.io:
# airsprotocols-mcp = "0.2.3"
```

Example usage:

```rust
use airsprotocols_mcp::{McpClientBuilder, StdioTransport};
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create an MCP client
    let client = McpClientBuilder::new()
        .client_info("my-app", "0.1.0")
        .timeout(Duration::from_secs(30))
        .build(transport)
        .await?;

    // List available tools
    let tools = client.list_tools().await?;
    println!("Available tools: {:?}", tools);

    Ok(())
}
```

For detailed documentation, see [`protocols/mcp/README.md`](protocols/mcp/README.md)

---

## 🎯 Design Philosophy

### Type Safety First
- Leverage Rust's type system for protocol compliance
- Compile-time verification of message structures
- Zero-cost abstractions

### Protocol Agnostic
- Clean separation between protocol specification and transport
- Multiple transport implementations (stdio, HTTP, WebSocket)
- Easy to add custom transports

### Production Ready
- Comprehensive error handling
- Built-in authentication and authorization
- Extensive testing (unit, integration, property-based)
- Performance benchmarks

### Async Native
- Built on `tokio` async runtime
- Non-blocking I/O throughout
- Efficient concurrent operations

### Interoperability
- Standards-compliant implementations
- Compatible with official SDKs (Python, TypeScript, etc.)
- Bridge adapters for protocol interop

---

## 📚 Documentation

- **[MCP Documentation](protocols/mcp/docs/)** - Complete MCP implementation guide
- **Protocol Specifications:**
  - [Model Context Protocol](https://modelcontextprotocol.io/)
  - [Agent-to-Agent Protocol](https://a2a-protocol.org/)

---

## 🔗 Related Projects

- **[airsstack](https://github.com/airsstack/airsstack)** - Application-level agent implementations
  - `airs-mcpserver-fs` - Secure filesystem MCP server
  - Example applications and servers

---

## 🤝 Contributing

Contributions are welcome! This project is in active development.

### Current Focus
- ✅ MCP protocol implementation complete
- 🚧 A2A protocol implementation
- 🚧 LLM provider APIs

### How to Contribute
1. Check open issues and discussions
2. Follow Rust best practices and conventions
3. Include tests for new features
4. Update documentation

---

## 📄 License

Dual licensed under:
- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT License ([LICENSE-MIT](LICENSE-MIT))

You may choose either license for your use.

---

## 🏛️ Governance

This project follows open-source best practices:
- **MCP:** Anthropic's Model Context Protocol specification
- **A2A/ACP:** Linux Foundation governance (merged protocols)
- **Community-driven:** Open to contributions and feedback

---

## 📮 Contact & Support

- **Issues:** [GitHub Issues](https://github.com/airsstack/airsprotocols/issues)
- **Discussions:** [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)
- **Main Project:** [airsstack](https://github.com/airsstack/airsstack)

---

## 🗺️ Roadmap

### Phase 1: Foundation ✅ Complete
- [x] Project structure setup
- [x] Workspace configuration
- [x] MCP protocol implementation
- [ ] CI/CD pipeline

### Phase 2: Core Protocols
- [ ] Complete MCP documentation
- [ ] A2A protocol implementation
- [ ] MCP ↔ A2A bridge adapters

### Phase 3: LLM APIs
- [ ] Anthropic API client
- [ ] OpenAI API client
- [ ] Unified LLM interface

### Phase 4: Ecosystem
- [ ] Additional LLM providers
- [ ] Performance optimizations
- [ ] Advanced features and extensions

---

**Built with 🦀 Rust** | **Powered by the AI Agent Ecosystem**
