# OpenRouter API Errors and Debugging

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/errors-and-debugging

## Overview

OpenRouter returns structured error responses that provide detailed information about what went wrong and how to fix it.

## Error Response Format

All errors follow this structure:

```typescript
type ErrorResponse = {
  error: {
    code: number;
    message: string;
    metadata?: Record<string, unknown>;
  };
};
```

### Example

```json
{
  "error": {
    "code": 400,
    "message": "Invalid model specified",
    "metadata": {
      "provided_model": "invalid/model-name"
    }
  }
}
```

## HTTP Status Codes vs Response Codes

### Request Errors (Status Code = error.code)

If your **original request is invalid** or your **account has issues**, the HTTP status code matches `error.code`:

```
HTTP/1.1 400 Bad Request
{
  "error": {
    "code": 400,
    "message": "Missing required parameter: messages"
  }
}
```

### Model Processing Errors (Status Code = 200)

If an error occurs **while the LLM is processing**, the HTTP status is `200 OK` and the error is in the response body:

```
HTTP/1.1 200 OK
{
  "error": {
    "code": 502,
    "message": "Provider error: Model timeout"
  }
}
```

## Error Codes

### 400 Bad Request

**Causes**:
- Invalid or missing parameters
- Malformed JSON
- CORS issues
- Invalid model name

**Examples**:
```json
{
  "error": {
    "code": 400,
    "message": "Missing required parameter: 'model'"
  }
}
```

**Solutions**:
- Validate request parameters
- Check JSON syntax
- Verify model name exists
- Ensure proper CORS headers

### 401 Unauthorized

**Causes**:
- Invalid API key
- Missing Authorization header
- OAuth session expired
- Disabled API key

**Example**:
```json
{
  "error": {
    "code": 401,
    "message": "Invalid API key provided"
  }
}
```

**Solutions**:
- Check API key is correct
- Verify Authorization header format: `Bearer sk-or-v1-...`
- Ensure key hasn't been deleted
- Regenerate key if compromised

### 402 Payment Required

**Causes**:
- Insufficient credits
- Negative account balance
- Credit limit reached on key

**Example**:
```json
{
  "error": {
    "code": 402,
    "message": "Insufficient credits. Please add credits to your account."
  }
}
```

**Solutions**:
- Add credits at [https://openrouter.ai/credits](https://openrouter.ai/credits)
- Check balance: `GET /api/v1/key`
- Increase key credit limit if needed

### 403 Forbidden

**Causes**:
- Content moderation flag
- Input violated model's content policy

**Example**:
```json
{
  "error": {
    "code": 403,
    "message": "Your input was flagged by the model's content moderation system",
    "metadata": {
      "reasons": ["violence", "hate"],
      "flagged_input": "...",
      "provider_name": "OpenAI",
      "model_slug": "gpt-4"
    }
  }
}
```

**Solutions**:
- Review and modify input content
- Check model's content policy
- Use different model with less strict moderation

### 408 Request Timeout

**Causes**:
- Request took too long to process
- Model timeout
- Network timeout

**Example**:
```json
{
  "error": {
    "code": 408,
    "message": "Request timeout"
  }
}
```

**Solutions**:
- Reduce input length
- Try again with same or different model
- Implement retry logic

### 429 Too Many Requests

**Causes**:
- Rate limit exceeded
- Too many requests in time window

**Example**:
```json
{
  "error": {
    "code": 429,
    "message": "Rate limit exceeded. Please try again later."
  }
}
```

**Response Headers**:
```http
Retry-After: 60
X-RateLimit-Reset: 1234567890
```

**Solutions**:
- Implement exponential backoff
- Check `Retry-After` header for wait time
- Distribute load across multiple models
- Upgrade account tier

### 502 Bad Gateway

**Causes**:
- Model provider is down
- Invalid response from provider
- Provider error

**Example**:
```json
{
  "error": {
    "code": 502,
    "message": "Provider error",
    "metadata": {
      "provider_name": "OpenAI",
      "raw": { /* provider's error */ }
    }
  }
}
```

**Solutions**:
- Retry with exponential backoff
- Enable fallback models
- Try different provider
- Check OpenRouter status page

### 503 Service Unavailable

**Causes**:
- No available providers meet routing requirements
- All providers are at capacity
- Model temporarily unavailable

**Example**:
```json
{
  "error": {
    "code": 503,
    "message": "No available providers meet your routing requirements"
  }
}
```

**Solutions**:
- Enable `allow_fallbacks: true`
- Relax provider requirements
- Try different model
- Wait and retry

## Error Metadata

### Moderation Errors

When content is flagged:

```typescript
type ModerationErrorMetadata = {
  reasons: string[];          // Why input was flagged
  flagged_input: string;      // Text segment flagged (max 100 chars)
  provider_name: string;      // Provider that flagged content
  model_slug: string;         // Specific model
};
```

**Example**:
```json
{
  "error": {
    "code": 403,
    "message": "Content flagged",
    "metadata": {
      "reasons": ["violence", "graphic_content"],
      "flagged_input": "The violent scene depicted...",
      "provider_name": "OpenAI",
      "model_slug": "gpt-4"
    }
  }
}
```

### Provider Errors

When provider encounters an error:

```typescript
type ProviderErrorMetadata = {
  provider_name: string;      // Provider that failed
  raw: unknown;               // Raw error from provider
};
```

**Example**:
```json
{
  "error": {
    "code": 502,
    "message": "Provider error",
    "metadata": {
      "provider_name": "Anthropic",
      "raw": {
        "type": "overloaded_error",
        "message": "Overloaded"
      }
    }
  }
}
```

## No Content Generated

Sometimes models may not generate content:

**Causes**:
- Model warming up from cold start
- System scaling up capacity
- Provider infrastructure issues

**Typical Duration**: Few seconds to few minutes

**Note**: You may still be charged for prompt processing even if no content is generated.

**Solutions**:
- Implement retry logic
- Try different provider/model with recent activity
- Wait and retry

## Streaming Errors

### Pre-Stream Errors

Errors **before streaming starts** use standard format with HTTP status codes:

```
HTTP/1.1 400 Bad Request
{
  "error": {
    "code": 400,
    "message": "Invalid parameter: temperature"
  }
}
```

### Mid-Stream Errors

Errors **during streaming** are sent as SSE events:

```
data: {"id":"cmpl-abc","error":{"code":"server_error","message":"Provider disconnected"},"choices":[{"delta":{"content":""},"finish_reason":"error"}]}
```

**Characteristics**:
- HTTP status remains 200 OK
- Error in top-level `error` field
- `choices` array with `finish_reason: "error"`
- Stream terminates after error event

See [Streaming](streaming.md) for detailed error handling.

## Debugging

### Debug Option

Enable debug mode to see transformed requests:

```typescript
{
  "model": "anthropic/claude-3-haiku",
  "messages": [...],
  "stream": true,
  "debug": {
    "echo_upstream_body": true
  }
}
```

### Debug Response

First chunk includes transformed request:

```json
{
  "id": "gen-xxxxx",
  "provider": "Anthropic",
  "model": "anthropic/claude-3-haiku",
  "choices": [],
  "debug": {
    "echo_upstream_body": {
      "system": [{"type": "text", "text": "You are helpful."}],
      "messages": [{"role": "user", "content": "Hello!"}],
      "model": "claude-3-haiku-20240307",
      "stream": true,
      "max_tokens": 64000,
      "temperature": 1
    }
  }
}
```

### Debug Use Cases

1. **Parameter Transformations**: See how OpenRouter maps parameters
2. **Message Formatting**: Check how messages are transformed
3. **Applied Defaults**: View default values OpenRouter applies
4. **Provider Fallbacks**: Debug shows each attempted provider

### Debug Warnings

⚠️ **Streaming Only**: Debug only works with `stream: true`

⚠️ **Not for Production**: Debug may expose sensitive information. Only use in development.

⚠️ **Automatic Redaction**: OpenRouter redacts potentially sensitive data, but not guaranteed.

## Error Handling Best Practices

### 1. Handle All Error Types

```typescript
async function makeRequest(url: string, options: any) {
  try {
    const response = await fetch(url, options);
    
    // Check HTTP status
    if (!response.ok) {
      const error = await response.json();
      throw new Error(`API Error (${error.error.code}): ${error.error.message}`);
    }
    
    const data = await response.json();
    
    // Check for errors in successful response
    if (data.error) {
      throw new Error(`Model Error: ${data.error.message}`);
    }
    
    return data;
  } catch (error) {
    console.error('Request failed:', error);
    throw error;
  }
}
```

### 2. Implement Retry Logic

```typescript
async function retryWithBackoff(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      const isRetryable = [408, 429, 502, 503].includes(error.code);
      
      if (!isRetryable || i === maxRetries - 1) {
        throw error;
      }
      
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### 3. Log Error Details

```typescript
function logError(error: any, context: any) {
  console.error({
    timestamp: new Date().toISOString(),
    error_code: error.error?.code,
    error_message: error.error?.message,
    error_metadata: error.error?.metadata,
    request_context: context,
  });
}
```

### 4. User-Friendly Messages

```typescript
function getUserMessage(errorCode: number): string {
  switch (errorCode) {
    case 400:
      return 'Invalid request. Please check your input.';
    case 401:
      return 'Authentication failed. Please check your API key.';
    case 402:
      return 'Insufficient credits. Please add credits to continue.';
    case 429:
      return 'Too many requests. Please try again in a moment.';
    case 502:
    case 503:
      return 'Service temporarily unavailable. Please try again.';
    default:
      return 'An unexpected error occurred. Please try again.';
  }
}
```

## Example: Complete Error Handling

```typescript
async function chatCompletion(messages: any[]) {
  const maxRetries = 3;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          model: 'openai/gpt-4o',
          messages,
        }),
      });
      
      const data = await response.json();
      
      // Handle HTTP errors
      if (!response.ok) {
        const { code, message, metadata } = data.error;
        
        // Non-retryable errors
        if ([400, 401, 403].includes(code)) {
          throw new Error(`${message} (code: ${code})`);
        }
        
        // Retryable errors
        if ([408, 429, 502, 503].includes(code) && attempt < maxRetries - 1) {
          const delay = Math.pow(2, attempt) * 1000;
          console.log(`Retrying in ${delay}ms...`);
          await new Promise(resolve => setTimeout(resolve, delay));
          continue;
        }
        
        throw new Error(message);
      }
      
      // Check for model errors
      if (data.error) {
        throw new Error(`Model error: ${data.error.message}`);
      }
      
      return data;
      
    } catch (error) {
      console.error(`Attempt ${attempt + 1} failed:`, error);
      
      if (attempt === maxRetries - 1) {
        throw error;
      }
    }
  }
}
```

## Related Documentation

- [API Overview](api-overview.md)
- [Streaming](streaming.md)
- [Limits and Quotas](limits-and-quotas.md)
- [Authentication](authentication.md)
