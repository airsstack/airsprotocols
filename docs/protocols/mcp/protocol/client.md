# Client Feature Specifications

## Client Methods

The MCP client provides high-level methods for interacting with MCP servers. All methods return `McpResult<T>` and automatically handle JSON-RPC protocol details.

### Initialization

```rust
/// Initialize the MCP session with the server
/// Returns server capabilities after successful handshake
pub async fn initialize(&mut self) -> McpResult<InitializeResponse>
```

### Tool Operations

```rust
/// List available tools from the server
/// Returns Vec<Tool> containing tool definitions
pub async fn list_tools(&mut self) -> McpResult<Vec<Tool>>

/// Execute a tool with the given arguments
/// Returns Vec<Content> with the tool execution results
pub async fn call_tool(
    &mut self,
    name: impl Into<String>,
    arguments: Option<Value>,
) -> McpResult<Vec<Content>>
```

### Resource Operations

```rust
/// List available resources from the server
/// Returns Vec<Resource> containing resource metadata
pub async fn list_resources(&mut self) -> McpResult<Vec<Resource>>

/// Read content from a specific resource
/// Returns Vec<Content> with the resource contents
pub async fn read_resource(
    &mut self, 
    uri: impl Into<String>
) -> McpResult<Vec<Content>>

/// Subscribe to changes for a specific resource
pub async fn subscribe_to_resource(
    &mut self, 
    uri: impl Into<String>
) -> McpResult<()>
```

### Prompt Operations

```rust
/// List available prompts from the server
/// Returns Vec<Prompt> containing prompt definitions
pub async fn list_prompts(&mut self) -> McpResult<Vec<Prompt>>

/// Get a prompt with the given arguments
/// Returns Vec<PromptMessage> containing the rendered prompt
pub async fn get_prompt(
    &mut self,
    name: impl Into<String>,
    arguments: HashMap<String, String>,
) -> McpResult<Vec<PromptMessage>>
```

### Logging Operations

```rust
/// Set logging configuration
pub async fn set_logging_config(
    &mut self, 
    config: LoggingConfig
) -> McpResult<()>
```

### Utility Operations

```rust
/// Close the MCP session
pub async fn close(&mut self) -> McpResult<()>

/// Check if the client is initialized and ready
pub fn is_ready(&self) -> bool

/// Check if server supports a specific capability
pub fn supports_capability(
    &self, 
    check: impl Fn(&ServerCapabilities) -> bool
) -> bool
```

## Sampling: Server-Initiated AI Interactions

**Note:** The sampling feature allows servers to request AI model interactions through the client. This feature is part of the MCP specification but implementation details should be verified with the current codebase.

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SamplingRequest {
    pub messages: Vec<SamplingMessage>,
    pub model_preferences: Option<ModelPreferences>,
    pub system_prompt: Option<String>,
    pub include_context: Option<ContextInclusion>,
    pub temperature: Option<f64>,
    pub max_tokens: Option<u32>,
    pub stop_sequences: Option<Vec<String>>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ModelPreferences {
    pub hints: Option<ModelHints>,
    pub cost_priority: Option<f64>,      // 0.0-1.0
    pub speed_priority: Option<f64>,     // 0.0-1.0  
    pub intelligence_priority: Option<f64>, // 0.0-1.0
}
```

Double Approval Workflow:

```rust
// Step 1: Server requests sampling permission
server.request_sampling_approval(request).await?;

// Step 2: User approves the AI interaction  
if user_approves_sampling(request) {
    let response = client.create_message(request).await?;
    
    // Step 3: User approves the AI response
    if user_approves_response(response) {
        return Ok(response);
    }
}
```

Context Inclusion Controls:

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum ContextInclusion {
    None,           // No additional context
    ThisServer,     // Context from requesting server only
    AllServers,     // Context from all connected servers
}
```
