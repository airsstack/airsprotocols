# OpenRouter Streaming Guide

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/streaming

## Overview

OpenRouter supports streaming responses from **any model** using Server-Sent Events (SSE). Streaming enables real-time response generation, perfect for chat interfaces and interactive applications.

## Enabling Streaming

Set `stream: true` in your request:

```json
{
  "model": "openai/gpt-4o",
  "messages": [...],
  "stream": true
}
```

## Server-Sent Events (SSE) Format

### Event Structure

Each event follows SSE specification:

```
data: {json_object}\n\n
```

### Example Stream

```
data: {"id":"gen-abc","choices":[{"delta":{"content":"Hello"}}]}

data: {"id":"gen-abc","choices":[{"delta":{"content":" there"}}]}

data: {"id":"gen-abc","choices":[{"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

### Stream Termination

Final event signals completion:

```
data: [DONE]
```

## Implementation Examples

### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'YOUR_API_KEY',
});

const stream = await openRouter.chat.send({
  model: 'openai/gpt-4o',
  messages: [{ role: 'user', content: 'Write a story' }],
  stream: true,
  streamOptions: { includeUsage: true }
});

for await (const chunk of stream) {
  const content = chunk.choices?.[0]?.delta?.content;
  if (content) {
    console.log(content);
  }

  // Final chunk includes usage stats
  if (chunk.usage) {
    console.log('Usage:', chunk.usage);
  }
}
```

### Python

```python
import requests
import json

response = requests.post(
  "https://openrouter.ai/api/v1/chat/completions",
  headers={
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
  },
  json={
    "model": "openai/gpt-4o",
    "messages": [{"role": "user", "content": "Write a story"}],
    "stream": True
  },
  stream=True
)

buffer = ""
for chunk in response.iter_content(chunk_size=1024, decode_unicode=True):
  buffer += chunk
  while True:
    line_end = buffer.find('\n')
    if line_end == -1:
      break

    line = buffer[:line_end].strip()
    buffer = buffer[line_end + 1:]

    if line.startswith('data: '):
      data = line[6:]
      if data == '[DONE]':
        break

      try:
        data_obj = json.loads(data)
        content = data_obj["choices"][0]["delta"].get("content")
        if content:
          print(content, end="", flush=True)
      except json.JSONDecodeError:
        pass
```

### TypeScript (fetch)

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${apiKey}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-4o',
    messages: [{ role: 'user', content: 'Write a story' }],
    stream: true,
  }),
});

const reader = response.body?.getReader();
const decoder = new TextDecoder();
let buffer = '';

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  buffer += decoder.decode(value, { stream: true });

  while (true) {
    const lineEnd = buffer.indexOf('\n');
    if (lineEnd === -1) break;

    const line = buffer.slice(0, lineEnd).trim();
    buffer = buffer.slice(lineEnd + 1);

    if (line.startsWith('data: ')) {
      const data = line.slice(6);
      if (data === '[DONE]') break;

      try {
        const parsed = JSON.parse(data);
        const content = parsed.choices[0].delta.content;
        if (content) {
          console.log(content);
        }
      } catch (e) {
        // Ignore invalid JSON
      }
    }
  }
}
```

## SSE Comments

OpenRouter occasionally sends comments to prevent connection timeouts:

```
: OPENROUTER PROCESSING
```

These are standard SSE comments and should be ignored per [SSE specification](https://html.spec.whatwg.org/multipage/server-sent-events.html#event-stream-interpretation).

**Usage**: You can leverage these to show loading indicators or dynamic status messages.

## Recommended SSE Clients

Some SSE client implementations don't parse comments correctly. We recommend:

- [eventsource-parser](https://github.com/rexxars/eventsource-parser)
- [OpenAI SDK](https://www.npmjs.com/package/openai)
- [Vercel AI SDK](https://www.npmjs.com/package/ai)

## Stream Cancellation

### Supported Providers

Cancelling streaming requests **immediately stops processing and billing** for these providers:

✅ **Supported**:
- OpenAI, Azure, Anthropic
- Fireworks, Mancer, Recursal
- AnyScale, Lepton, OctoAI
- Novita, DeepInfra, Together
- Cohere, Hyperbolic, Infermatic
- Avian, XAI, Cloudflare
- SFCompute, Nineteen, Liquid
- Friendli, Chutes, DeepSeek

❌ **Not Currently Supported**:
- AWS Bedrock, Groq, Modal
- Google, Google AI Studio, Minimax
- HuggingFace, Replicate, Perplexity
- Mistral, AI21, Featherless
- Lynn, Lambda, Reflection
- SambaNova, Inflection, ZeroOneAI
- AionLabs, Alibaba, Nebius
- Kluster, Targon, InferenceNet

### Implementation

#### TypeScript SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'YOUR_API_KEY',
});

const controller = new AbortController();

try {
  const stream = await openRouter.chat.send({
    model: 'openai/gpt-4o',
    messages: [{ role: 'user', content: 'Write a story' }],
    stream: true,
  }, {
    signal: controller.signal,
  });

  for await (const chunk of stream) {
    const content = chunk.choices?.[0]?.delta?.content;
    if (content) {
      console.log(content);
    }
  }
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Stream cancelled');
  } else {
    throw error;
  }
}

// To cancel the stream:
controller.abort();
```

#### Python

```python
import requests
from threading import Event, Thread

def stream_with_cancellation(prompt: str, cancel_event: Event):
    with requests.Session() as session:
        response = session.post(
            "https://openrouter.ai/api/v1/chat/completions",
            headers={"Authorization": f"Bearer {api_key}"},
            json={
                "model": "openai/gpt-4o",
                "messages": [{"role": "user", "content": prompt}],
                "stream": True
            },
            stream=True
        )

        try:
            for line in response.iter_lines():
                if cancel_event.is_set():
                    response.close()
                    return
                if line:
                    print(line.decode(), end="", flush=True)
        finally:
            response.close()

# Usage:
cancel_event = Event()
stream_thread = Thread(target=lambda: stream_with_cancellation("Write a story", cancel_event))
stream_thread.start()

# To cancel:
cancel_event.set()
```

#### TypeScript (fetch)

```typescript
const controller = new AbortController();

try {
  const response = await fetch(
    'https://openrouter.ai/api/v1/chat/completions',
    {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'openai/gpt-4o',
        messages: [{ role: 'user', content: 'Write a story' }],
        stream: true,
      }),
      signal: controller.signal,
    },
  );

  // Process stream...
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Stream cancelled');
  } else {
    throw error;
  }
}

// To cancel:
controller.abort();
```

⚠️ **Warning**: Cancellation only works for streaming requests with supported providers. For non-streaming or unsupported providers, the model continues processing and you will be billed for the complete response.

## Error Handling During Streaming

### Pre-Stream Errors

Errors **before any tokens** are sent return standard JSON with HTTP status codes:

```json
{
  "error": {
    "code": 400,
    "message": "Invalid model specified"
  }
}
```

**Common Status Codes**:
- `400`: Bad Request (invalid parameters)
- `401`: Unauthorized (invalid API key)
- `402`: Payment Required (insufficient credits)
- `429`: Too Many Requests (rate limited)
- `502`: Bad Gateway (provider error)
- `503`: Service Unavailable (no available providers)

### Mid-Stream Errors

Errors **after tokens have been sent** cannot change HTTP status (already 200 OK). Error sent as SSE event:

```
data: {"id":"cmpl-abc123","object":"chat.completion.chunk","created":1234567890,"model":"gpt-3.5-turbo","provider":"openai","error":{"code":"server_error","message":"Provider disconnected"},"choices":[{"index":0,"delta":{"content":""},"finish_reason":"error"}]}
```

**Key characteristics**:
- Error at **top level** alongside standard fields
- `choices` array with `finish_reason: "error"` to terminate stream
- HTTP status remains 200 OK
- Stream terminates after error event

### Error Handling Example

#### TypeScript SDK

```typescript
async function streamWithErrorHandling(prompt: string) {
  try {
    const stream = await openRouter.chat.send({
      model: 'openai/gpt-4o',
      messages: [{ role: 'user', content: prompt }],
      stream: true,
    });

    for await (const chunk of stream) {
      // Check for errors in chunk
      if ('error' in chunk) {
        console.error(`Stream error: ${chunk.error.message}`);
        if (chunk.choices?.[0]?.finish_reason === 'error') {
          console.log('Stream terminated due to error');
        }
        return;
      }

      // Process normal content
      const content = chunk.choices?.[0]?.delta?.content;
      if (content) {
        console.log(content);
      }
    }
  } catch (error) {
    // Handle pre-stream errors
    console.error(`Error: ${error.message}`);
  }
}
```

#### Python

```python
import requests
import json

def stream_with_error_handling(prompt):
    response = requests.post(
        'https://openrouter.ai/api/v1/chat/completions',
        headers={'Authorization': f'Bearer {api_key}'},
        json={
            'model': 'openai/gpt-4o',
            'messages': [{'role': 'user', 'content': prompt}],
            'stream': True
        },
        stream=True
    )

    # Check initial HTTP status for pre-stream errors
    if response.status_code != 200:
        error_data = response.json()
        print(f"Error: {error_data['error']['message']}")
        return

    # Process stream and handle mid-stream errors
    for line in response.iter_lines():
        if line:
            line_text = line.decode('utf-8')
            if line_text.startswith('data: '):
                data = line_text[6:]
                if data == '[DONE]':
                    break

                try:
                    parsed = json.loads(data)

                    # Check for mid-stream error
                    if 'error' in parsed:
                        print(f"Stream error: {parsed['error']['message']}")
                        if parsed.get('choices', [{}])[0].get('finish_reason') == 'error':
                            print("Stream terminated due to error")
                        break

                    # Process normal content
                    content = parsed['choices'][0]['delta'].get('content')
                    if content:
                        print(content, end='', flush=True)

                except json.JSONDecodeError:
                    pass
```

## Usage Statistics with Streaming

When streaming, usage statistics are sent in the **final chunk** with an empty `choices` array:

```
data: {"id":"gen-abc","choices":[],"usage":{"prompt_tokens":10,"completion_tokens":50,"total_tokens":60}}

data: [DONE]
```

To receive usage with TypeScript SDK:

```typescript
const stream = await openRouter.chat.send({
  model: 'openai/gpt-4o',
  messages: [...],
  stream: true,
  streamOptions: { includeUsage: true }
});

for await (const chunk of stream) {
  if (chunk.usage) {
    console.log('Total tokens:', chunk.usage.total_tokens);
  }
}
```

## Best Practices

1. **Buffer Handling**: Always buffer incomplete lines when parsing SSE
2. **Error Checking**: Check for both pre-stream and mid-stream errors
3. **Graceful Degradation**: Handle cases where streaming is interrupted
4. **Usage Tracking**: Capture usage stats from final chunk
5. **Cancellation**: Implement cancellation for better UX
6. **Comment Handling**: Ignore SSE comments (lines starting with `:`)
7. **Use Recommended Clients**: Use battle-tested SSE parsers

## Related Documentation

- [API Overview](api-overview.md)
- [Errors](errors.md)
- [Parameters](parameters.md)
- [Authentication](authentication.md)
