# Welcome to AIRS Protocols

**Common AI Communication Protocols and LLM API Implementations in Rust**

A comprehensive Rust workspace providing production-ready implementations of agent communication protocols and LLM provider APIs. Built with performance, type safety, and interoperability in mind.

---

## 🚀 Quick Links

### 🚀 [Getting Started](getting-started.md)
Get up and running with AIRS Protocols in minutes.

### 🔧 [Protocols](protocols/index.md)
Explore communication protocols for AI agents.

### 🌐 [APIs](apis/index.md)
LLM provider client implementations (Coming Soon).

### 📚 [Examples](examples/index.md)
Learn from practical examples and tutorials.

---

## 📦 What's Inside

### **Protocols** (`protocols/`)

Communication protocols for AI agents and intelligent systems:

- **[MCP - Model Context Protocol](protocols/mcp/index.md)** <span class="version-badge">v1.0.0-rc.1</span>
  - Connect AI models to tools, resources, and context
  - JSON-RPC 2.0 foundation with transport abstraction
  - Built-in authentication (API Key, OAuth2) and authorization
  - Stdio and HTTP transport implementations

- **A2A - Agent-to-Agent Protocol** <span class="version-badge">Planned</span>
  - Enable communication between independent AI agents
  - Multiple protocol bindings (JSON-RPC, gRPC, REST)
  - Task lifecycle management and streaming support

### **APIs** (`apis/`)

LLM provider client implementations with unified interfaces (Coming Soon):

- Anthropic Claude API client
- OpenAI GPT API client
- Google Gemini API client
- Ollama local models client

---

## ✨ Key Features

### Type Safety First
Leverage Rust's type system for protocol compliance with compile-time verification and zero-cost abstractions.

### Protocol Agnostic
Clean separation between protocol specification and transport with multiple implementations (stdio, HTTP, WebSocket).

### Production Ready
Comprehensive error handling, built-in authentication and authorization, extensive testing, and performance benchmarks.

### Async Native
Built on `tokio` async runtime with non-blocking I/O throughout and efficient concurrent operations.

### Interoperability
Standards-compliant implementations compatible with official SDKs (Python, TypeScript, etc.) and bridge adapters for protocol interop.

---

## 🏗️ Project Structure

```
airsprotocols/
├── protocols/          # Communication protocols
│   ├── mcp/           # Model Context Protocol
│   └── a2a/           # Agent-to-Agent Protocol (Planned)
│
└── apis/              # LLM provider clients (Planned)
    ├── anthropic/     # Claude API
    ├── openai/        # GPT API
    ├── google/        # Gemini API
    └── ollama/        # Local models
```

---

## 🎯 Use Cases

- **AI Agent Development**: Build intelligent agents that communicate using standard protocols
- **LLM Integration**: Connect language models to external tools and data sources
- **Multi-Agent Systems**: Enable collaboration between independent AI agents
- **Tool Providers**: Expose capabilities to AI models through standardized interfaces
- **Resource Management**: Provide structured access to data and content for AI systems

---

## 🚀 Quick Example

Add to your `Cargo.toml`:

```toml
[dependencies]
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
```

Create an MCP client:

```rust
use airsprotocols_mcp::McpClientBuilder;
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = McpClientBuilder::new()
        .client_info("my-app", "0.1.0")
        .timeout(Duration::from_secs(30))
        .build(transport)
        .await?;

    let tools = client.list_tools().await?;
    println!("Available tools: {:?}", tools);

    Ok(())
}
```

For detailed guides, see [Getting Started](getting-started.md).

---

## 📚 Documentation

- **[Overview](overview.md)** - Detailed project overview and philosophy
- **[Getting Started](getting-started.md)** - Quick start guide
- **[Architecture](architecture.md)** - High-level architecture
- **[Protocols](protocols/index.md)** - Protocol implementations
- **[Examples](examples/index.md)** - Practical examples
- **[Contributing](contributing.md)** - Contribution guidelines

---

## 🔗 Related Projects

- **[airsstack](https://github.com/airsstack/airsstack)** - Application-level agent implementations
  - `airs-mcpserver-fs` - Secure filesystem MCP server
  - Example applications and servers

---

## 📄 License

Dual licensed under:
- Apache License, Version 2.0 ([LICENSE-APACHE](https://github.com/airsstack/airsprotocols/blob/main/LICENSE-APACHE))
- MIT License ([LICENSE-MIT](https://github.com/airsstack/airsprotocols/blob/main/LICENSE-MIT))

You may choose either license for your use.

---

## 🤝 Contributing

Contributions are welcome! See our [Contributing Guide](contributing.md) for details on:

- Development setup
- Code standards
- Testing requirements
- Pull request process

---

**Built with 🦀 Rust** | **Powered by the AI Agent Ecosystem**
