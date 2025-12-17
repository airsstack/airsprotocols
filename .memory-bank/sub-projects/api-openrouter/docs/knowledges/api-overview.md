# OpenRouter API Reference Overview

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/overview

## Overview

OpenRouter's API is very similar to the OpenAI Chat API, with a few small differences. **OpenRouter normalizes the schema across models and providers** so you only need to learn one interface.

### Key Characteristics

- OpenAI-compatible request/response schemas
- Single unified interface for 100+ models
- Automatic provider routing and fallback
- Normalized responses across all models

## Base URL

```
https://openrouter.ai/api/v1
```

## Core Endpoints

### Chat Completions

```
POST /api/v1/chat/completions
```

Primary endpoint for all LLM interactions (streaming and non-streaming).

## Request Format

### Basic Request Structure

```typescript
type Request = {
  // Either "messages" or "prompt" is required
  messages?: Message[];
  prompt?: string;

  // If "model" is unspecified, uses the user's default
  model?: string;

  // Core parameters
  response_format?: { type: 'json_object' };
  stop?: string | string[];
  stream?: boolean;
  max_tokens?: number;
  temperature?: number;

  // Tool calling
  tools?: Tool[];
  tool_choice?: ToolChoice;

  // Advanced parameters
  seed?: number;
  top_p?: number;
  top_k?: number;
  frequency_penalty?: number;
  presence_penalty?: number;
  repetition_penalty?: number;
  logit_bias?: { [key: number]: number };
  top_logprobs?: number;
  min_p?: number;
  top_a?: number;
  prediction?: { type: 'content'; content: string };

  // OpenRouter-specific
  transforms?: string[];
  models?: string[];
  route?: 'fallback';
  provider?: ProviderPreferences;
  user?: string;
  debug?: { echo_upstream_body?: boolean };
};
```

### Message Types

```typescript
type Message =
  | {
      role: 'user' | 'assistant' | 'system';
      content: string | ContentPart[];
      name?: string;
    }
  | {
      role: 'tool';
      content: string;
      tool_call_id: string;
      name?: string;
    };

type ContentPart = TextContent | ImageContentPart;

type TextContent = {
  type: 'text';
  text: string;
};

type ImageContentPart = {
  type: 'image_url';
  image_url: {
    url: string; // URL or base64 encoded image data
    detail?: string;
  };
};
```

### Tool Definitions

```typescript
type Tool = {
  type: 'function';
  function: FunctionDescription;
};

type FunctionDescription = {
  description?: string;
  name: string;
  parameters: object; // JSON Schema object
};

type ToolChoice =
  | 'none'
  | 'auto'
  | {
      type: 'function';
      function: {
        name: string;
      };
    };
```

## Response Format

### Non-Streaming Response

```typescript
type Response = {
  id: string;
  choices: (NonStreamingChoice | StreamingChoice | NonChatChoice)[];
  created: number; // Unix timestamp
  model: string;
  object: 'chat.completion' | 'chat.completion.chunk';
  system_fingerprint?: string;
  usage?: ResponseUsage;
};

type NonStreamingChoice = {
  finish_reason: string | null;
  native_finish_reason: string | null;
  message: {
    content: string | null;
    role: string;
    tool_calls?: ToolCall[];
  };
  error?: ErrorResponse;
};

type ResponseUsage = {
  prompt_tokens: number;
  completion_tokens: number;
  total_tokens: number;
};
```

### Example Response

```json
{
  "id": "gen-xxxxxxxxxxxxxx",
  "choices": [
    {
      "finish_reason": "stop",
      "native_finish_reason": "stop",
      "message": {
        "role": "assistant",
        "content": "Hello there!"
      }
    }
  ],
  "usage": {
    "prompt_tokens": 0,
    "completion_tokens": 4,
    "total_tokens": 4
  },
  "model": "openai/gpt-3.5-turbo"
}
```

## Finish Reasons

OpenRouter normalizes `finish_reason` to one of these values:

- `tool_calls` - Model called a tool/function
- `stop` - Model finished naturally
- `length` - Reached token limit
- `content_filter` - Content filtered
- `error` - Error occurred

The raw provider `finish_reason` is available in `native_finish_reason`.

## Streaming Responses

Enable streaming by setting `stream: true`.

### Server-Sent Events (SSE) Format

```
data: {"id":"gen-abc","choices":[{"delta":{"content":"Hello"}}]}

data: {"id":"gen-abc","choices":[{"delta":{"content":" there"}}]}

data: {"id":"gen-abc","choices":[{"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

### SSE Comment Payloads

OpenRouter occasionally sends comments to prevent connection timeouts:

```
: OPENROUTER PROCESSING
```

These can be safely ignored per SSE specification.

## Special Features

### Assistant Prefill

Guide model responses by including a partial assistant message at the end:

```typescript
{
  "model": "openai/gpt-4o",
  "messages": [
    { "role": "user", "content": "What is the meaning of life?" },
    { "role": "assistant", "content": "I'm not sure, but my best guess is" }
  ]
}
```

### Response Format

Force structured JSON output:

```typescript
{
  "model": "openai/gpt-4o",
  "messages": [...],
  "response_format": { "type": "json_object" }
}
```

**Note**: Only supported by OpenAI models, Nitro models, and some others. Check model capabilities on openrouter.ai/models.

## Token Counting

- Token counts in responses use a **normalized tokenizer** (GPT-4o tokenizer)
- Credit usage and pricing use **native token counts**
- For precise accounting, query the `/api/v1/generation` endpoint

### Querying Generation Stats

```typescript
const generation = await fetch(
  'https://openrouter.ai/api/v1/generation?id=$GENERATION_ID',
  { headers },
);

const stats = await generation.json();
```

Returns detailed token counts and costs using the model's native tokenizer.

## Model Routing

### Default Model Selection

If `model` parameter is omitted, the user's or payer's default model is used.

### Automatic Fallbacks

OpenRouter automatically:
- Selects least expensive and best GPUs
- Falls back to other providers on 5xx errors
- Handles rate limiting gracefully

### Provider Selection

```typescript
{
  "model": "openai/gpt-4o",
  "provider": {
    "order": ["openai", "azure"],
    "allow_fallbacks": true,
    "require_parameters": true
  }
}
```

## Non-Standard Parameters

If a model doesn't support a parameter:
- Parameter is silently ignored
- Rest of parameters are forwarded to the model API
- No error is returned

Examples:
- `logit_bias` not supported by non-OpenAI models
- `top_k` not supported by OpenAI models

## Headers

### Required Headers

```http
Content-Type: application/json
Authorization: Bearer <OPENROUTER_API_KEY>
```

### Optional Headers

```http
HTTP-Referer: <YOUR_SITE_URL>
X-Title: <YOUR_SITE_NAME>
```

These help identify your app on openrouter.ai for rankings.

## Related Documentation

- [Authentication](authentication.md)
- [Parameters](parameters.md)
- [Streaming](streaming.md)
- [Errors](errors.md)
- [Models](models.md)
