# Server Feature Specifications

## Server Implementation Pattern

MCP servers in AIRS use a `MessageHandler` pattern rather than a direct server builder. The server processes JSON-RPC messages and returns responses.

### Basic Server Structure

```rust
use airsprotocols_mcp::protocol::{MessageHandler, JsonRpcRequest, JsonRpcResponse};

struct MyMcpServer {
    // Your server state
}

#[async_trait]
impl MessageHandler for MyMcpServer {
    async fn handle_message(&mut self, request: JsonRpcRequest) -> Result<JsonRpcResponse, Error> {
        // Route and handle MCP methods
        match request.method.as_str() {
            "initialize" => self.handle_initialize(request).await,
            "tools/list" => self.handle_list_tools(request).await,
            "tools/call" => self.handle_call_tool(request).await,
            "resources/list" => self.handle_list_resources(request).await,
            "resources/read" => self.handle_read_resource(request).await,
            "prompts/list" => self.handle_list_prompts(request).await,
            "prompts/get" => self.handle_get_prompt(request).await,
            _ => Err(Error::method_not_found(request.method)),
        }
    }
}
```

## Resources: URI-Addressable Content

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Resource {
    pub uri: String,           // Unique identifier (custom schemes allowed)
    pub name: String,          // Human-readable name
    pub description: Option<String>,
    pub mime_type: Option<String>,
    pub size: Option<u64>,     // Size in bytes
}

// URI Template support (RFC 6570)
#[derive(Debug, Clone)]
pub struct ResourceTemplate {
    pub uri_template: String,  // e.g., "file://{path}"
    pub name: String,
    pub description: Option<String>,
}
```

Resource Operations:

```rust
// Core resource methods
"resources/list" → Vec<Resource>
"resources/read" → ResourceContent  
"resources/templates/list" → Vec<ResourceTemplate>

// Subscription support (real-time updates)
"resources/subscribe" → subscription_id
"resources/unsubscribe" → confirmation
// Server sends notifications: "notifications/resources/updated"
```

Implementation Requirements:

- URI Template Engine: RFC 6570 compliance for parameterized resources
- Subscription Management: Real-time update delivery with proper cleanup
- Content Handling: Binary (base64) and text content with MIME type support
- Pagination: Cursor-based pagination for large resource lists

## Tools: Executable Functions with Safety Controls

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Tool {
    pub name: String,
    pub description: String,
    pub input_schema: serde_json::Value, // JSON Schema
    pub output_schema: Option<serde_json::Value>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ToolCall {
    pub name: String,
    pub arguments: serde_json::Value,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ToolResult {
    pub content: Vec<Content>,    // Multi-modal results
    pub is_error: Option<bool>,   // Error indication
    pub meta: Option<serde_json::Value>, // Additional metadata
}
```

Tool Safety Framework:

```rust
#[derive(Debug, Clone)]
pub enum SafetyLevel {
    Safe,        // No approval required
    Moderate,    // User confirmation required  
    Dangerous,   // Explicit approval with risk warning
}

#[async_trait]
pub trait ToolExecutor {
    async fn execute_tool(
        &self, 
        call: ToolCall,
        approval: Option<UserApproval>
    ) -> Result<ToolResult, ToolError>;
    
    fn get_safety_level(&self, tool_name: &str) -> SafetyLevel;
}
```

Human-in-the-Loop Approval:

- Risk Assessment: Automatic safety level determination
- Approval Workflow: User approval before dangerous tool execution
- Audit Trail: Complete logging of tool executions and approvals

## Prompts: Templated AI Interactions

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Prompt {
    pub name: String,
    pub description: String,
    pub arguments: Option<Vec<PromptArgument>>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PromptArgument {
    pub name: String,
    pub description: String,
    pub required: Option<bool>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PromptMessage {
    pub role: MessageRole,  // "user", "assistant", "system"
    pub content: Content,   // Text or multi-modal content
}
```

Prompt Operations:

```rust
"prompts/list" → Vec<Prompt>
"prompts/get" → PromptGetResult { messages: Vec<PromptMessage> }

// Autocompletion support
"completion/complete" → CompletionResult
```

Advanced Features:

- Parameter Substitution: Dynamic argument injection into templates
- Multi-modal Content: Text and image content support
- Resource Embedding: Direct resource inclusion in prompt context
- Autocompletion: Intelligent completion for prompt arguments
