# airsprotocols

**Common AI Communication Protocols and LLM API Implementations in Rust**

A comprehensive Rust workspace providing production-ready implementations of agent communication protocols and LLM provider APIs. Built with performance, type safety, and interoperability in mind.

[![Documentation](https://img.shields.io/badge/docs-online-blue?style=flat-square)](https://airsstack.github.io/airsprotocols/)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue?style=flat-square)](LICENSE-MIT)

📚 **[Read the Documentation](https://airsstack.github.io/airsprotocols/)** | 🚀 **[Get Started](https://airsstack.github.io/airsprotocols/getting-started/)** | 🔧 **[MCP Protocol](https://airsstack.github.io/airsprotocols/protocols/mcp/)**

---

## 📦 What's Inside

### **Protocols** (`protocols/`)

Communication protocols for AI agents and intelligent systems:

- **[`airsprotocols-mcp`](protocols/mcp/)** - Model Context Protocol (MCP)
  - Connect AI models to tools, resources, and context
  - JSON-RPC 2.0 foundation with transport abstraction
  - Built-in authentication (API Key, OAuth2) and authorization
  - Stdio and HTTP transport implementations
  - **Status:** ✅ Available on [crates.io](https://crates.io/crates/airsprotocols-mcp) (v1.0.0-rc.1)

- **`airsprotocols-a2a`** - Agent-to-Agent Protocol (A2A/ACP)
  - Enable communication between independent AI agents
  - Multiple protocol bindings (JSON-RPC, gRPC, REST)
  - Task lifecycle management and streaming support
  - **Status:** 🚧 Planned

### **Servers** (`mcp/servers/`)

Production-ready MCP server implementations:

- **[`airsprotocols-mcpserver-filesystem`](mcp/servers/filesystem/)** - Secure Filesystem MCP Server
  - Security-first filesystem operations for AI agents
  - Human-in-the-loop approval workflows with configurable policies
  - Complete file operations: read, write, create, delete, move, copy
  - Binary file protection and comprehensive audit logging
  - Sub-100ms performance with efficient memory management
  - **Status:** ✅ Available on [crates.io](https://crates.io/crates/airsprotocols-mcpserver-filesystem) (v1.0.0-rc.1)
  - **Install:** `cargo install airsprotocols-mcpserver-filesystem`

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
├── mcp/
│   └── servers/       # MCP Server implementations
│       └── filesystem/ # Secure filesystem server
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
airsprotocols-mcp = "1.0.0-rc.1"
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

### Using MCP Filesystem Server

Install the filesystem server:

```bash
cargo install airsprotocols-mcpserver-filesystem
```

Configure Claude Desktop (add to `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "airsprotocols-mcpserver-filesystem",
      "args": ["serve", "--allowed-path", "/path/to/your/project"]
    }
  }
}
```

Now Claude Desktop can securely read and write files in your project directory!

For detailed documentation, see:
- **[Installation Guide](https://airsstack.github.io/airsprotocols/servers/filesystem/installation/)**
- **[Configuration Guide](https://airsstack.github.io/airsprotocols/servers/filesystem/configuration/)**
- **[Security Guide](https://airsstack.github.io/airsprotocols/servers/filesystem/security/)**

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

### Security First
- Human-in-the-loop approval workflows
- Configurable security policies
- Comprehensive audit logging
- Path validation and threat detection

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

**📖 [Complete Documentation](https://airsstack.github.io/airsprotocols/)** - Comprehensive guides, API references, and examples

- **[Getting Started](https://airsstack.github.io/airsprotocols/getting-started/)** - Quick start guide
- **[MCP Protocol Guide](https://airsstack.github.io/airsprotocols/protocols/mcp/)** - Complete MCP implementation guide
- **[Filesystem Server Guide](https://airsstack.github.io/airsprotocols/servers/filesystem/)** - Secure filesystem server documentation
- **[Architecture](https://airsstack.github.io/airsprotocols/architecture/)** - System design and architecture
- **[Examples](https://airsstack.github.io/airsprotocols/examples/)** - Practical examples and tutorials
- **[Contributing](https://airsstack.github.io/airsprotocols/contributing/)** - Contribution guidelines

### Protocol Specifications
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Agent-to-Agent Protocol](https://a2a-protocol.org/)

---

## 📦 Published Packages

### Available on crates.io

- **[`airsprotocols-mcp`](https://crates.io/crates/airsprotocols-mcp)** v1.0.0-rc.1
  - Model Context Protocol implementation
  - `cargo add airsprotocols-mcp`

- **[`airsprotocols-mcpserver-filesystem`](https://crates.io/crates/airsprotocols-mcpserver-filesystem)** v1.0.0-rc.1
  - Secure filesystem MCP server
  - `cargo install airsprotocols-mcpserver-filesystem`

---

## 🔗 Related Projects

- **[airsstack](https://github.com/airsstack/airsstack)** - Application-level agent implementations
  - Example applications and integrations
  - Additional MCP server implementations

---

## 🤝 Contributing

Contributions are welcome! This project is in active development.

### Current Focus
- ✅ MCP protocol implementation complete
- ✅ MCP filesystem server published
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
- [x] MCP filesystem server published
- [ ] CI/CD pipeline

### Phase 2: Core Protocols
- [x] Complete MCP documentation
- [x] Documentation website (https://airsstack.github.io/airsprotocols/)
- [ ] A2A protocol implementation
- [ ] MCP ↔ A2A bridge adapters

### Phase 3: LLM APIs
- [ ] Anthropic API client
- [ ] OpenAI API client
- [ ] Unified LLM interface

### Phase 4: Ecosystem
- [ ] Additional MCP servers
- [ ] Additional LLM providers
- [ ] Performance optimizations
- [ ] Advanced features and extensions

---

**Built with 🦀 Rust** | **Powered by the AI Agent Ecosystem**
