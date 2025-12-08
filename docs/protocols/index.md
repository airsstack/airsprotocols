# Protocols Overview

Communication protocols that enable AI agents to interact with tools, resources, and other agents.

## Available Protocols

### Model Context Protocol (MCP)

<span class="version-badge">v1.0.0-rc.1</span> **Status: Available**

The Model Context Protocol enables AI models to securely access external tools, data sources, and context through a standardized interface.

**Key Features:**

- 🔧 **Tool Calling** - Expose functions for AI models to execute
- 📦 **Resource Access** - Provide structured access to data and content
- 📝 **Prompt Templates** - Supply templates for generating structured prompts
- 🔐 **Authentication** - API Key and OAuth2 support
- 🚀 **Multiple Transports** - Stdio and HTTP implementations

[→ Go to MCP Documentation](mcp/index.md)

---

### Agent-to-Agent Protocol (A2A)

<span class="version-badge">Planned</span> **Status: In Development**

The Agent-to-Agent Protocol (formerly ACP) enables communication and collaboration between independent AI agents.

**Planned Features:**

- 🤝 **Agent Communication** - Standardized messaging between agents
- 📋 **Task Management** - Task delegation and lifecycle tracking
- 🔄 **Multiple Bindings** - JSON-RPC, gRPC, and REST support
- 🔍 **Discovery** - Agent discovery and capability negotiation
- 📊 **Streaming** - Real-time data streaming between agents

[→ View A2A Roadmap](a2a/index.md)

---

## Protocol Comparison

| Feature | MCP | A2A |
|---------|-----|-----|
| **Purpose** | Connect AI to tools/resources | Agent-to-agent communication |
| **Status** | ✅ Available | 🚧 Planned |
| **Transport** | Stdio, HTTP | JSON-RPC, gRPC, REST |
| **Use Case** | Tool integration | Multi-agent systems |
| **Authentication** | API Key, OAuth2 | TBD |

## When to Use Which Protocol

### Use MCP When:

- Connecting AI models to external tools and APIs
- Providing structured data access to language models
- Building tool providers for AI applications
- Integrating with Claude Desktop or similar clients
- Need stdio or HTTP communication

### Use A2A When:

- Building multi-agent systems
- Enabling agent collaboration and coordination
- Implementing task delegation between agents
- Creating agent networks and workflows
- Need distributed agent architectures

## Getting Started

### With MCP

```toml
[dependencies]
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
```

See the [MCP Quick Start Guide](mcp/quick-start.md) for detailed instructions.

### With A2A

Coming soon! A2A is currently in the planning phase.

## Protocol Standards

All AIRS protocol implementations follow these principles:

1. **Spec Compliance** - Faithful implementation of official specifications
2. **Type Safety** - Strong typing and compile-time validation
3. **Async-First** - Built on tokio for efficient async operations
4. **Extensibility** - Clear extension points for custom implementations
5. **Testing** - Comprehensive test coverage

## Resources

### MCP Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP Documentation](mcp/index.md)
- [MCP Examples](../examples/index.md)

### A2A Resources

- [Agent-to-Agent Protocol Specification](https://a2a-protocol.org/)
- [Linux Foundation Project Page](https://www.linuxfoundation.org/)

## Contributing

We welcome contributions to both protocol implementations! See our [Contributing Guide](../contributing.md) for details.

### Current Focus

- ✅ MCP: Feature complete, accepting enhancements and bug fixes
- 🚧 A2A: Design and planning phase, architecture discussions welcome

---

Have questions? Open a [discussion](https://github.com/airsstack/airsprotocols/discussions) or check the documentation for each protocol.
