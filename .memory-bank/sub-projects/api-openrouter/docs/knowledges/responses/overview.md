# OpenRouter Responses API Beta - Overview

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/responses/overview

⚠️ **Beta API**: This API is in beta stage and may have breaking changes. Use with caution in production environments.

ℹ️ **Stateless Only**: This API is stateless - each request is independent and no conversation state is persisted between requests. You must include the full conversation history in each request.

## Overview

OpenRouter's Responses API Beta provides OpenAI-compatible access to multiple AI models through a unified interface, designed as a drop-in replacement for OpenAI's Responses API. This stateless API offers enhanced capabilities including reasoning, tool calling, and web search integration.

## Base URL

```
https://openrouter.ai/api/v1/responses
```

## Authentication

All requests require authentication using your OpenRouter API key in the Authorization header:

**TypeScript**:
```typescript
const response = await fetch('https://openrouter.ai/api/v1/responses', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_OPENROUTER_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/o4-mini',
    input: 'Hello, world!',
  }),
});
```

**Python**:
```python
import requests

response = requests.post(
    'https://openrouter.ai/api/v1/responses',
    headers={
        'Authorization': 'Bearer YOUR_OPENROUTER_API_KEY',
        'Content-Type': 'application/json',
    },
    json={
        'model': 'openai/o4-mini',
        'input': 'Hello, world!',
    }
)
```

**cURL**:
```bash
curl -X POST https://openrouter.ai/api/v1/responses \
  -H "Authorization: Bearer YOUR_OPENROUTER_API_KEY" \
  -H "Content-Type": 'application/json" \
  -d '{
    "model": "openai/o4-mini",
    "input": "Hello, world!"
  }'
```

## Core Features

### [Basic Usage](basic-usage.md)
Learn the fundamentals of making requests with simple text input and handling responses.

**Key Topics**:
- Simple string input
- Structured message arrays
- Streaming responses
- Multi-turn conversations

### [Reasoning](reasoning.md)
Access advanced reasoning capabilities with configurable effort levels and encrypted reasoning chains.

**Key Topics**:
- Reasoning configuration
- Effort levels (minimal, low, medium, high)
- Reasoning in conversation context
- Streaming reasoning

### [Tool Calling](tool-calling.md)
Integrate function calling with support for parallel execution and complex tool interactions.

**Key Topics**:
- Tool definitions
- Tool choice options
- Multiple tools
- Parallel tool calls
- Tool responses in conversation

### [Web Search](web-search.md)
Enable web search capabilities with real-time information retrieval and citation annotations.

**Key Topics**:
- Web search plugin
- Online model variants
- Citation annotations
- Complex search queries

## Common Parameters

| Parameter           | Type            | Required | Description                                      |
|---------------------|-----------------|----------|--------------------------------------------------|
| `model`             | string          | ✅       | Model to use (e.g., `openai/o4-mini`)            |
| `input`             | string or array | ✅       | Text or message array                            |
| `stream`            | boolean         | ❌       | Enable streaming responses (default: false)      |
| `max_output_tokens` | integer         | ❌       | Maximum tokens to generate                       |
| `temperature`       | number          | ❌       | Sampling temperature (0-2)                       |
| `top_p`             | number          | ❌       | Nucleus sampling parameter (0-1)                 |
| `reasoning`         | object          | ❌       | Reasoning configuration                          |
| `tools`             | array           | ❌       | Function calling tools                           |
| `plugins`           | array           | ❌       | Plugins (e.g., web search)                       |

## Response Format

### Basic Response

```json
{
  "id": "resp_1234567890",
  "object": "response",
  "created_at": 1234567890,
  "model": "openai/o4-mini",
  "output": [
    {
      "type": "message",
      "id": "msg_abc123",
      "status": "completed",
      "role": "assistant",
      "content": [
        {
          "type": "output_text",
          "text": "Response content here...",
          "annotations": []
        }
      ]
    }
  ],
  "usage": {
    "input_tokens": 12,
    "output_tokens": 45,
    "total_tokens": 57
  },
  "status": "completed"
}
```

### Response Fields

| Field        | Type    | Description                                |
|--------------|---------|-------------------------------------------|
| `id`         | string  | Unique response identifier                 |
| `object`     | string  | Always "response"                          |
| `created_at` | number  | Unix timestamp                             |
| `model`      | string  | Model that generated the response          |
| `output`     | array   | Array of output items (messages, etc.)     |
| `usage`      | object  | Token usage statistics                     |
| `status`     | string  | Response status (completed, in_progress)   |

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "invalid_prompt",
    "message": "Missing required parameter: 'model'."
  },
  "metadata": null
}
```

### Error Codes

| Code                  | Description               | HTTP Status |
|-----------------------|---------------------------|-------------|
| `invalid_prompt`      | Request validation failed | 400         |
| `rate_limit_exceeded` | Too many requests         | 429         |
| `server_error`        | Internal server error     | 500+        |

For comprehensive error handling, see [Error Handling](error-handling.md).

## Stateless Design

The Responses API Beta is **stateless**, meaning:
- No server-side conversation state
- Each request is independent
- Must include full conversation history in each request
- No session management required

**Example Multi-Turn**:
```typescript
// First request
const firstResponse = await fetch(/* ... */, {
  body: JSON.stringify({
    model: 'openai/o4-mini',
    input: 'What is the capital of France?'
  })
});

// Second request - include previous conversation
const secondResponse = await fetch(/* ... */, {
  body: JSON.stringify({
    model: 'openai/o4-mini',
    input: [
      {
        type: 'message',
        role: 'user',
        content: [{ type: 'input_text', text: 'What is the capital of France?' }]
      },
      {
        type: 'message',
        role: 'assistant',
        id: 'msg_123',
        status: 'completed',
        content: [{ type: 'output_text', text: 'The capital of France is Paris.', annotations: [] }]
      },
      {
        type: 'message',
        role: 'user',
        content: [{ type: 'input_text', text: 'What is the population?' }]
      }
    ]
  })
});
```

## Rate Limits

Standard OpenRouter rate limits apply. See [API Limits](../limits-and-quotas.md) for details.

## Comparison with Chat Completions API

| Feature              | Responses API Beta         | Chat Completions API  |
|----------------------|----------------------------|-----------------------|
| **Endpoint**         | `/api/v1/responses`        | `/api/v1/chat/completions` |
| **State Management** | Stateless                  | Stateless             |
| **Input Format**     | `input` (string or array)  | `messages` (array)    |
| **Reasoning**        | ✅ Explicit configuration  | ❌ Not supported      |
| **Web Search**       | ✅ Plugin system           | `:online` variants    |
| **Tool Calling**     | ✅ Supported               | ✅ Supported          |
| **Streaming**        | ✅ SSE format              | ✅ SSE format         |

## Next Steps

- [Basic Usage](basic-usage.md) - Get started with simple requests
- [Reasoning](reasoning.md) - Enable advanced reasoning
- [Tool Calling](tool-calling.md) - Integrate function calling
- [Web Search](web-search.md) - Add web search capabilities
- [Error Handling](error-handling.md) - Handle errors gracefully

## Related Documentation

- [Chat Completions API](../api-overview.md)
- [Authentication](../authentication.md)
- [Models](../models.md)
- [Streaming](../streaming.md)
