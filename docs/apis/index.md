# LLM Provider APIs

<span class="version-badge">Planned</span>

!!! info "Status: Future Release"
    LLM provider API implementations are planned for future releases. This page outlines the vision and roadmap for unified LLM client libraries.

## Overview

AIRS Protocols will include production-ready client implementations for major LLM (Large Language Model) providers, offering a unified Rust interface for interacting with different AI services.

## Vision

As developers build AI applications, they often need to:

- Switch between different LLM providers based on cost, performance, or features
- Implement fallback logic when one provider is unavailable
- Compare outputs across multiple providers
- Abstract provider-specific details from application logic

AIRS APIs aims to solve these challenges by providing:

- **Unified Interface**: Common API across all providers
- **Type Safety**: Rust's type system for compile-time guarantees
- **Async Native**: Efficient async/await implementation
- **Error Handling**: Comprehensive error types and recovery
- **Production Ready**: Battle-tested with extensive testing

## Planned Providers

### Anthropic Claude API

<span class="version-badge">Planned</span>

Client for Anthropic's Claude language models.

**Planned Features:**

- Chat completions API
- Streaming responses
- Function calling
- System prompts and examples
- Token counting utilities
- Error handling and retries

### OpenAI GPT API

<span class="version-badge">Planned</span>

Client for OpenAI's GPT models (GPT-4, GPT-3.5, etc.).

**Planned Features:**

- Chat completions API
- Streaming responses
- Function calling and tools
- Vision capabilities
- Audio transcription/generation
- Fine-tuning support

### Google Gemini API

<span class="version-badge">Planned</span>

Client for Google's Gemini models.

**Planned Features:**

- Multi-modal input support
- Streaming responses
- Function declarations
- Safety settings
- Content caching
- System instructions

### Ollama

<span class="version-badge">Planned</span>

Client for locally-run models via Ollama.

**Planned Features:**

- Local model management
- Streaming generation
- Model library integration
- Performance optimization
- Custom model support

## Unified Interface (Draft)

All provider clients will implement a common trait:

```rust
#[async_trait]
pub trait LlmProvider: Send + Sync {
    /// Generate a chat completion
    async fn chat_completion(
        &self,
        request: ChatCompletionRequest,
    ) -> Result<ChatCompletion, LlmError>;
    
    /// Stream a chat completion
    async fn chat_completion_stream(
        &self,
        request: ChatCompletionRequest,
    ) -> Result<ChatCompletionStream, LlmError>;
    
    /// List available models
    async fn list_models(&self) -> Result<Vec<Model>, LlmError>;
    
    /// Count tokens for a message
    async fn count_tokens(&self, messages: &[Message]) -> Result<usize, LlmError>;
}
```

### Common Types

```rust
pub struct ChatCompletionRequest {
    pub model: String,
    pub messages: Vec<Message>,
    pub temperature: Option<f32>,
    pub max_tokens: Option<usize>,
    pub stream: bool,
    pub tools: Option<Vec<Tool>>,
}

pub struct Message {
    pub role: Role,
    pub content: Content,
    pub name: Option<String>,
    pub tool_calls: Option<Vec<ToolCall>>,
}

pub enum Role {
    System,
    User,
    Assistant,
    Tool,
}

pub enum Content {
    Text(String),
    MultiModal(Vec<ContentPart>),
}
```

## Example Usage (Conceptual)

### Basic Chat Completion

```rust
use airsprotocols_anthropic::AnthropicClient;
use airsprotocols_common::{LlmProvider, ChatCompletionRequest, Message, Role, Content};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create client
    let client = AnthropicClient::new("api-key")?;
    
    // Create request
    let request = ChatCompletionRequest {
        model: "claude-3-opus-20240229".to_string(),
        messages: vec![
            Message {
                role: Role::User,
                content: Content::Text("Hello, Claude!".to_string()),
                name: None,
                tool_calls: None,
            }
        ],
        temperature: Some(0.7),
        max_tokens: Some(1000),
        stream: false,
        tools: None,
    };
    
    // Get completion
    let completion = client.chat_completion(request).await?;
    println!("Response: {}", completion.content);
    
    Ok(())
}
```

### Provider Switching

```rust
// Abstract over providers
fn create_client(provider: &str) -> Box<dyn LlmProvider> {
    match provider {
        "anthropic" => Box::new(AnthropicClient::new(api_key)?),
        "openai" => Box::new(OpenAiClient::new(api_key)?),
        "gemini" => Box::new(GeminiClient::new(api_key)?),
        _ => panic!("Unknown provider"),
    }
}

// Use any provider
let client = create_client("anthropic");
let completion = client.chat_completion(request).await?;
```

### Streaming Responses

```rust
use tokio_stream::StreamExt;

let mut stream = client.chat_completion_stream(request).await?;

while let Some(chunk) = stream.next().await {
    match chunk {
        Ok(delta) => print!("{}", delta.content),
        Err(e) => eprintln!("Stream error: {}", e),
    }
}
```

### Function Calling

```rust
use serde_json::json;

let request = ChatCompletionRequest {
    model: "claude-3-opus".to_string(),
    messages: vec![
        Message::user("What's the weather in San Francisco?")
    ],
    tools: Some(vec![
        Tool {
            name: "get_weather".to_string(),
            description: "Get current weather for a location".to_string(),
            parameters: json!({
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "units": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["location"]
            }),
        }
    ]),
    ..Default::default()
};

let completion = client.chat_completion(request).await?;
if let Some(tool_calls) = completion.tool_calls {
    for call in tool_calls {
        println!("Tool: {}, Args: {}", call.name, call.arguments);
    }
}
```

## Architecture (Planned)

```
airsprotocols-common/          # Common types and traits
├── traits.rs                  # LlmProvider trait
├── types.rs                   # Message, Request, Response
└── errors.rs                  # Common error types

airsprotocols-anthropic/       # Anthropic implementation
├── client.rs                  # AnthropicClient
├── types.rs                   # Provider-specific types
└── error.rs                   # Error conversions

airsprotocols-openai/          # OpenAI implementation
airsprotocols-google/          # Google implementation
airsprotocols-ollama/          # Ollama implementation
```

## Roadmap

### Phase 1: Foundation (Q2 2025)

- [ ] Design common trait interface
- [ ] Define common types (Message, Request, Response)
- [ ] Error handling strategy
- [ ] Testing framework

### Phase 2: Initial Providers (Q3 2025)

- [ ] Anthropic Claude client
- [ ] OpenAI GPT client
- [ ] Streaming support
- [ ] Basic function calling
- [ ] Documentation

### Phase 3: Additional Providers (Q4 2025)

- [ ] Google Gemini client
- [ ] Ollama client
- [ ] Multi-modal support
- [ ] Advanced features
- [ ] Performance optimization

### Phase 4: Ecosystem (2026)

- [ ] Provider comparison tools
- [ ] Fallback and retry logic
- [ ] Cost optimization utilities
- [ ] Rate limiting and quotas
- [ ] Caching layer

## Design Principles

### 1. Provider Agnostic

The common interface abstracts provider differences while allowing access to provider-specific features when needed.

### 2. Type Safe

Leverage Rust's type system to prevent common errors:

- Compile-time model name validation (where possible)
- Strong typing for messages and responses
- Explicit error handling

### 3. Async First

All operations are async for optimal performance:

- Non-blocking API calls
- Efficient streaming
- Concurrent request handling

### 4. Production Ready

Built for real-world usage:

- Comprehensive error handling
- Retry logic and backoff
- Rate limiting
- Timeout handling
- Logging and observability

### 5. Extensible

Easy to add new providers and features:

- Clear trait boundaries
- Provider-specific extensions
- Feature flags for optional capabilities

## Integration with Protocols

### MCP Integration

LLM API clients can work seamlessly with MCP:

```rust
// Use MCP tools with LLM
let tools = mcp_client.list_tools().await?;
let llm_tools = convert_mcp_to_llm_tools(tools);

let request = ChatCompletionRequest {
    tools: Some(llm_tools),
    ..request
};

let completion = llm_client.chat_completion(request).await?;
```

### A2A Integration

Agents using A2A can leverage LLM APIs:

```rust
// Agent powered by LLM
struct LlmAgent {
    llm_client: Box<dyn LlmProvider>,
    a2a_client: A2AClient,
}

impl Agent for LlmAgent {
    async fn handle_task(&self, task: Task) -> Result<TaskResult, Error> {
        let request = create_llm_request(task);
        let completion = self.llm_client.chat_completion(request).await?;
        Ok(TaskResult::from_completion(completion))
    }
}
```

## Contributing

We welcome input on the API design!

### How to Contribute

1. **Design Feedback**: Share thoughts in [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)
2. **Provider Requests**: Suggest additional providers to support
3. **Feature Ideas**: Propose features and improvements
4. **Early Testing**: Join when beta releases are available

### Areas of Interest

- Common trait interface design
- Provider-specific feature exposure
- Error handling strategies
- Streaming API design
- Function calling patterns

## Resources

### External Resources

- [Anthropic API Documentation](https://docs.anthropic.com/)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Google Gemini Documentation](https://ai.google.dev/)
- [Ollama Documentation](https://ollama.ai/)

### Related Documentation

- [AIRS Protocols Overview](../index.md)
- [MCP Documentation](../protocols/mcp/index.md)
- [Architecture Guide](../architecture.md)

## Stay Updated

Follow development progress:

- **GitHub**: Watch the [repository](https://github.com/airsstack/airsprotocols) for updates
- **Discussions**: Join [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)
- **Issues**: Track [API-related issues](https://github.com/airsstack/airsprotocols/issues?q=label%3Aapi)

---

Interested in LLM APIs? Star the repository and watch for announcements!
