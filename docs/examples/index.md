# Examples

Practical examples and tutorials for working with AIRS Protocols.

## MCP Examples

### Getting Started

- **[Quick Start](../protocols/mcp/quick-start.md)** - Your first MCP client and server
- **[Installation](../protocols/mcp/installation.md)** - Setup and configuration
- **[Basic Examples](../protocols/mcp/usage/basic-examples.md)** - Common usage patterns

### Usage Guides

- **[Advanced Patterns](../protocols/mcp/usage/advanced-patterns.md)** - Complex scenarios and best practices
- **[Claude Integration](../protocols/mcp/usage/claude-integration.md)** - Integrate with Claude Desktop
- **[Custom Transports](../protocols/mcp/usage/custom-transports.md)** - Implement custom transport layers

### Running Examples

The airsprotocols repository includes several runnable examples in the `protocols/mcp/examples/` directory.

#### HTTP API Key Client Integration

A complete example showing HTTP transport with API key authentication:

```bash
cd protocols/mcp/examples/http-apikey-client-integration
cargo run
```

**Features demonstrated:**

- HTTP transport client setup
- API key authentication
- Mock server for testing
- Request/response handling
- Error handling patterns

## Example Code Snippets

### Creating an MCP Client

```rust
use airsprotocols_mcp::McpClientBuilder;
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = McpClientBuilder::new()
        .client_info("my-app", "1.0.0")
        .timeout(Duration::from_secs(30))
        .build(transport)
        .await?;

    let tools = client.list_tools().await?;
    println!("Found {} tools", tools.len());

    Ok(())
}
```

### Implementing a Tool Provider

```rust
use async_trait::async_trait;
use airsprotocols_mcp::providers::{ToolProvider, ToolCall, ToolResult};
use airsprotocols_mcp::IntegrationError;
use serde_json::json;

struct CalculatorProvider;

#[async_trait]
impl ToolProvider for CalculatorProvider {
    async fn call_tool(&self, call: ToolCall) -> Result<ToolResult, IntegrationError> {
        match call.name.as_str() {
            "add" => {
                let params = call.params.unwrap_or_default();
                let a = params["a"].as_i64().unwrap_or(0);
                let b = params["b"].as_i64().unwrap_or(0);
                Ok(ToolResult::success(json!({ "result": a + b })))
            }
            _ => Err(IntegrationError::MethodNotFound(call.name))
        }
    }
    
    // Other required methods...
}
```

### HTTP Server with Authentication

```rust
use airsprotocols_mcp::transport::adapters::http::AxumHttpServer;
use std::sync::Arc;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let server = AxumHttpServer::new_with_empty_handlers(
        connection_manager,
        session_manager,
        jsonrpc_processor,
        config,
    ).await?;
    
    // Add authentication if needed
    let auth_server = server.with_authentication(auth_adapter, auth_config);
    
    // Start server
    auth_server.serve("0.0.0.0:3000").await?;
    
    Ok(())
}
```

## Community Examples

We encourage the community to share their examples! Here's how:

### Sharing Your Example

1. **Create Your Example**: Build something useful with AIRS Protocols
2. **Open an Issue**: Share your example in a [GitHub issue](https://github.com/airsstack/airsprotocols/issues)
3. **Submit a PR**: Add your example to the repository
4. **Write a Blog Post**: Share your experience on your blog

### Example Ideas

- Integration with popular AI services
- Custom transport implementations
- Authentication strategies
- Real-world tool providers
- Performance optimization techniques
- Testing patterns

## Learning Resources

### Documentation

- **[Getting Started Guide](../getting-started.md)** - Quick start tutorial
- **[MCP Documentation](../protocols/mcp/index.md)** - Comprehensive MCP guide
- **[Architecture](../architecture.md)** - System design overview

### External Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Rust Async Book](https://rust-lang.github.io/async-book/)
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)

### Video Tutorials

Coming soon! We plan to create video tutorials covering:

- Getting started with MCP
- Building your first tool provider
- Integrating with Claude Desktop
- Advanced authentication patterns

## Example Projects

### Built with AIRS Protocols

Projects using AIRS Protocols in production or development:

!!! note "Share Your Project"
    Using AIRS Protocols in your project? Let us know in [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)!

## Troubleshooting Examples

Common issues and solutions:

### Connection Errors

```rust
// Problem: Connection timeout
// Solution: Increase timeout duration

let client = McpClientBuilder::new()
    .timeout(Duration::from_secs(60))  // Increased from 30
    .build(transport)
    .await?;
```

### Authentication Failures

```rust
// Problem: Invalid API key
// Solution: Verify key and source configuration

let auth = ApiKeyAuth {
    key: "your-valid-key".to_string(),
    source: ApiKeySource::AuthorizationBearer,
};
```

### Serialization Errors

```rust
// Problem: JSON parsing error
// Solution: Validate JSON structure before sending

let params = serde_json::from_str(&json_string)?;
let result = client.call_tool(ToolCall {
    name: "tool_name".to_string(),
    params: Some(params),
}).await?;
```

## Testing Examples

### Unit Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_tool_provider() {
        let provider = CalculatorProvider;
        let call = ToolCall {
            name: "add".to_string(),
            params: Some(json!({"a": 2, "b": 3})),
        };
        
        let result = provider.call_tool(call).await.unwrap();
        assert_eq!(result.content["result"], 5);
    }
}
```

### Integration Testing

```rust
#[tokio::test]
async fn test_mcp_client() {
    let transport = create_test_transport().await;
    let client = McpClientBuilder::new()
        .build(transport)
        .await
        .unwrap();
    
    let tools = client.list_tools().await.unwrap();
    assert!(!tools.is_empty());
}
```

## Next Steps

- **Try the examples** in the repository
- **Build your own** MCP server or client
- **Share your work** with the community
- **Contribute** improvements and new examples

---

Need help? Ask in [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions) or check the [documentation](../protocols/mcp/index.md).
