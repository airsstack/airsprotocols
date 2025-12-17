# OpenRouter API Knowledge Base

**Last Updated**: 2025-12-17
**Purpose**: Comprehensive reference documentation for OpenRouter API implementation

## Overview

This knowledge base contains detailed documentation about the OpenRouter API, organized for easy reference during Rust client library development. All content is sourced from official OpenRouter documentation and structured for implementation clarity.

## 📚 Documentation Structure

### Core API Documentation

| File | Description | Key Topics |
|------|-------------|------------|
| [api-overview.md](api-overview.md) | API structure and conventions | Request/response schemas, endpoints, model routing |
| [authentication.md](authentication.md) | Authentication methods | Bearer tokens, API keys, security best practices |
| [models.md](models.md) | Model listing and capabilities | Model discovery, pricing, supported parameters |
| [parameters.md](parameters.md) | Request parameters reference | Temperature, top_p, penalties, sampling methods |
| [streaming.md](streaming.md) | Server-Sent Events streaming | SSE format, cancellation, error handling |
| [embeddings.md](embeddings.md) | Embeddings API | Vector embeddings, semantic search, batch processing |
| [limits-and-quotas.md](limits-and-quotas.md) | Rate limits and credits | Credit system, rate limiting, usage tracking |
| [errors.md](errors.md) | Error handling and debugging | Error codes, metadata, debugging options |

### Responses API Beta (Advanced)

| File | Description | Key Topics |
|------|-------------|------------|
| [responses/overview.md](responses/overview.md) | Responses API introduction | Beta API overview, stateless design |
| [responses/basic-usage.md](responses/basic-usage.md) | Basic usage patterns | Simple/structured input, streaming |
| [responses/reasoning.md](responses/reasoning.md) | Reasoning capabilities | Effort levels, reasoning output |
| [responses/tool-calling.md](responses/tool-calling.md) | Function calling | Tool definitions, parallel execution |
| [responses/web-search.md](responses/web-search.md) | Web search integration | Plugins, citations, annotations |
| [responses/error-handling.md](responses/error-handling.md) | Responses API errors | Error codes, retry logic |

## 🚀 Quick Start

### For Implementation Reference

1. **Authentication**: Start with [authentication.md](authentication.md) for API key setup
2. **Basic Requests**: See [api-overview.md](api-overview.md) for request/response structure
3. **Streaming**: Reference [streaming.md](streaming.md) for SSE implementation
4. **Error Handling**: Check [errors.md](errors.md) for comprehensive error handling

### For Feature Implementation

- **Chat Completions**: [api-overview.md](api-overview.md) + [parameters.md](parameters.md)
- **Embeddings**: [embeddings.md](embeddings.md)
- **Tool/Function Calling**: [responses/tool-calling.md](responses/tool-calling.md)
- **Web Search**: [responses/web-search.md](responses/web-search.md)
- **Reasoning**: [responses/reasoning.md](responses/reasoning.md)

## 🎯 Key Implementation Details

### Base URLs

- **Production**: `https://openrouter.ai/api/v1`
- **Responses API Beta**: `https://openrouter.ai/api/v1/responses`

### Core Endpoints

```
POST /api/v1/chat/completions       # Chat completions (streaming/non-streaming)
POST /api/v1/embeddings              # Generate embeddings
GET  /api/v1/models                  # List available models
GET  /api/v1/key                     # Check API key status/credits
GET  /api/v1/generation?id={id}      # Query generation stats
```

### Authentication

```http
Authorization: Bearer sk-or-v1-{your-key}
Content-Type: application/json
```

Optional headers:
```http
HTTP-Referer: {your-app-url}
X-Title: {your-app-name}
```

### Request Format

```json
{
  "model": "openai/gpt-4o",
  "messages": [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "max_tokens": 500,
  "stream": false
}
```

### Response Format

```json
{
  "id": "gen-xxxxx",
  "model": "openai/gpt-4o",
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "Hello! How can I help you?"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 15,
    "completion_tokens": 10,
    "total_tokens": 25
  }
}
```

### Streaming Format (SSE)

```
data: {"id":"gen-abc","choices":[{"delta":{"content":"Hello"}}]}

data: {"id":"gen-abc","choices":[{"delta":{"content":" there"}}]}

data: {"id":"gen-abc","choices":[{"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

## 📊 Model Information

### Model Naming Convention

Format: `provider/model-name`

Examples:
- `openai/gpt-4o`
- `anthropic/claude-3.5-sonnet`
- `google/gemini-2.0-flash-exp`
- `meta-llama/llama-3.3-70b-instruct`

### Model Variants

- `:free` - Free tier variant
- `:online` - Web search enabled
- `:extended` - Extended context window

### Supported Parameters

Check model's `supported_parameters` field:
- `tools` - Function calling
- `temperature`, `top_p` - Sampling control
- `response_format` - JSON mode
- `seed` - Deterministic output
- And more...

## 🔧 Rust Implementation Notes

### Required Dependencies

Based on API requirements:

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json", "stream"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
tokio-stream = "0.1"
eventsource-stream = "0.2"  # For SSE parsing
thiserror = "1.0"
```

### Critical API Characteristics

1. **OpenAI Compatible**: Similar to OpenAI API but with provider routing
2. **Model Format**: Always `provider/model-name`
3. **Streaming**: Server-Sent Events (SSE) with `data:` prefix
4. **Token Counting**: 
   - Response uses normalized tokenizer (GPT-4o)
   - Billing uses native tokenizer
   - Use `/api/v1/generation` for precise counts
5. **Error Handling**:
   - Pre-stream errors: Standard JSON with HTTP codes
   - Mid-stream errors: SSE events with error field
6. **Rate Limiting**: Global per account, not per key
7. **Cost Tracking**: `X-OpenRouter-Cost` response header

### Type System Recommendations

```rust
// Core types needed
- Message { role, content, name?, tool_call_id? }
- Choice { message/delta, finish_reason }
- Usage { prompt_tokens, completion_tokens, total_tokens }
- Model { id, name, context_length, pricing, ... }
- Tool { type, function: FunctionDef }
- Error { code, message, metadata? }
```

### Streaming Implementation

Use `eventsource-stream` for SSE parsing:
- Handle `data:` prefix
- Parse JSON per line
- Handle `[DONE]` terminator
- Ignore comments (`:` prefix)
- Buffer incomplete lines

## 📖 Cross-References

### For Chat Completions
- Main: [api-overview.md](api-overview.md)
- Params: [parameters.md](parameters.md)
- Streaming: [streaming.md](streaming.md)
- Errors: [errors.md](errors.md)

### For Embeddings
- Main: [embeddings.md](embeddings.md)
- Models: [models.md](models.md)
- Errors: [errors.md](errors.md)

### For Tool Calling
- Responses API: [responses/tool-calling.md](responses/tool-calling.md)
- Alternative: Tool calls in [api-overview.md](api-overview.md)

### For Rate Limits & Costs
- Limits: [limits-and-quotas.md](limits-and-quotas.md)
- Auth: [authentication.md](authentication.md)
- Errors: [errors.md](errors.md)

## 🔍 Finding Information

### By Topic

- **Authentication & Security**: [authentication.md](authentication.md)
- **Request Parameters**: [parameters.md](parameters.md)
- **Error Codes**: [errors.md](errors.md)
- **Model Selection**: [models.md](models.md)
- **Streaming**: [streaming.md](streaming.md)
- **Cost & Limits**: [limits-and-quotas.md](limits-and-quotas.md)

### By Use Case

- **Building Chat Interface**: api-overview.md → streaming.md → errors.md
- **Semantic Search**: embeddings.md → models.md
- **Function Calling**: responses/tool-calling.md
- **Web-Connected AI**: responses/web-search.md
- **Advanced Reasoning**: responses/reasoning.md

## 📝 Documentation Standards

All documentation files follow:
- **Diataxis Framework**: Reference-style technical documentation
- **Source Attribution**: Each file includes source URL
- **Last Updated**: Date stamp for freshness tracking
- **Code Examples**: Multiple languages (Rust target, TypeScript/Python for reference)
- **Cross-References**: Links to related documentation

## 🔄 Maintenance

### Update Frequency
Documentation should be reviewed when:
- OpenRouter API changes announced
- New models/features added
- Error handling behavior changes
- Breaking changes in beta APIs

### Validation
- All URLs sourced from https://openrouter.ai/docs
- Code examples tested against live API
- Cross-references verified

## 🚧 Beta Features

**Warning**: The Responses API (`/api/v1/responses`) is in beta:
- May have breaking changes
- Stateless design (no server-side conversation state)
- Advanced features: reasoning, web search, enhanced tool calling
- Use standard Chat Completions API for production stability

## 📞 Support & Resources

- **Official Docs**: https://openrouter.ai/docs
- **Models List**: https://openrouter.ai/models
- **API Keys**: https://openrouter.ai/keys
- **Discord**: https://openrouter.ai/discord
- **Status**: Check OpenRouter status page for service updates

---

**For Implementation Questions**: Refer to specific documentation files above. For API changes or clarifications, consult official OpenRouter documentation at https://openrouter.ai/docs.
