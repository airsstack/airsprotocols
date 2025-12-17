# Responses API Beta - Error Handling

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/responses/error-handling

⚠️ **Beta API**: This API is in beta stage and may have breaking changes.

ℹ️ **Stateless Only**: This API is stateless - each request is independent.

## Overview

The Responses API Beta returns structured error responses that follow a consistent format.

## Error Response Format

All errors follow this structure:

```json
{
  "error": {
    "code": "invalid_prompt",
    "message": "Detailed error description"
  },
  "metadata": null
}
```

## Error Codes

| Code                  | Description               | HTTP Status |
|-----------------------|---------------------------|-------------|
| `invalid_prompt`      | Request validation failed | 400         |
| `rate_limit_exceeded` | Too many requests         | 429         |
| `server_error`        | Internal server error     | 500+        |

## Example Errors

### Invalid Prompt

```json
{
  "error": {
    "code": "invalid_prompt",
    "message": "Missing required parameter: 'model'."
  },
  "metadata": null
}
```

### Rate Limit Exceeded

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many requests. Please try again later."
  },
  "metadata": null
}
```

### Server Error

```json
{
  "error": {
    "code": "server_error",
    "message": "An internal error occurred. Please try again."
  },
  "metadata": null
}
```

## Error Handling Example

```typescript
async function makeRequest(input: string) {
  try {
    const response = await fetch('https://openrouter.ai/api/v1/responses', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer YOUR_API_KEY',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'openai/o4-mini',
        input,
      }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`API Error (${error.error.code}): ${error.error.message}`);
    }

    return await response.json();
  } catch (error) {
    console.error('Request failed:', error);
    throw error;
  }
}
```

## Retry Logic

Implement exponential backoff for retryable errors:

```typescript
async function requestWithRetry(input: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await makeRequest(input);
    } catch (error: any) {
      const isRetryable = ['rate_limit_exceeded', 'server_error'].includes(error.code);
      
      if (!isRetryable || i === maxRetries - 1) {
        throw error;
      }
      
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

## HTTP Status Codes

| Status | Description               | Action                    |
|--------|---------------------------|---------------------------|
| 400    | Bad Request               | Fix request parameters    |
| 401    | Unauthorized              | Check API key             |
| 429    | Too Many Requests         | Implement retry with backoff |
| 500    | Internal Server Error     | Retry request             |
| 503    | Service Unavailable       | Retry after delay         |

## Related Documentation

- [Overview](overview.md)
- [Basic Usage](basic-usage.md)
- [API Errors](../errors.md) - Complete error reference
