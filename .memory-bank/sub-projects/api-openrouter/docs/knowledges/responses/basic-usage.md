# Responses API Beta - Basic Usage

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/responses/basic-usage

⚠️ **Beta API**: This API is in beta stage and may have breaking changes.

## Overview

The Responses API Beta supports both simple string input and structured message arrays, making it easy to get started with basic text generation.

## Simple String Input

The simplest way to use the API:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/responses', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_OPENROUTER_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/o4-mini',
    input: 'What is the meaning of life?',
    max_output_tokens: 9000,
  }),
});

const result = await response.json();
```

## Structured Message Input

For complex conversations:

```typescript
{
  "model": "openai/o4-mini",
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": [
        {
          "type": "input_text",
          "text": "Tell me a joke about programming"
        }
      ]
    }
  ],
  "max_output_tokens": 9000
}
```

## Response Format

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
          "text": "Response content...",
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

## Streaming

Enable real-time response generation:

```typescript
{
  "model": "openai/o4-mini",
  "input": "Write a short story",
  "stream": true,
  "max_output_tokens": 9000
}
```

Process stream events:

```typescript
for await (const chunk of stream) {
  if (chunk.type === 'response.content_part.delta') {
    console.log(chunk.delta);
  }
}
```

## Multi-Turn Conversations

Since the API is stateless, include full conversation history:

```typescript
{
  "model": "openai/o4-mini",
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "What is the capital of France?"}]
    },
    {
      "type": "message",
      "role": "assistant",
      "id": "msg_123",
      "status": "completed",
      "content": [
        {"type": "output_text", "text": "The capital of France is Paris.", "annotations": []}
      ]
    },
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "What is the population?"}]
    }
  ]
}
```

ℹ️ **Required Fields**: The `id` and `status` fields are required for assistant messages in conversation history.

## Common Parameters

| Parameter           | Type    | Description                            |
|---------------------|---------|----------------------------------------|
| `model`             | string  | **Required.** Model identifier         |
| `input`             | mixed   | **Required.** String or message array  |
| `stream`            | boolean | Enable streaming (default: false)      |
| `max_output_tokens` | integer | Maximum tokens to generate             |
| `temperature`       | number  | Sampling temperature (0-2)             |
| `top_p`             | number  | Nucleus sampling (0-1)                 |

## Error Handling

```typescript
try {
  const response = await fetch(/* ... */);
  
  if (!response.ok) {
    const error = await response.json();
    console.error('API Error:', error.error.message);
    return;
  }
  
  const result = await response.json();
  console.log(result);
} catch (error) {
  console.error('Network Error:', error);
}
```

## Related Documentation

- [Overview](overview.md)
- [Reasoning](reasoning.md)
- [Tool Calling](tool-calling.md)
- [Web Search](web-search.md)
