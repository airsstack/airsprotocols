# OpenRouter API Authentication

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/authentication

## Overview

OpenRouter API authenticates requests using Bearer tokens with API keys. This allows you to use `curl` or the OpenAI SDK directly with OpenRouter by simply changing the base URL and API key.

## API Key Management

### Creating an API Key

1. Visit [https://openrouter.ai/keys](https://openrouter.ai/keys)
2. Click "Create Key"
3. Give it a descriptive name
4. Optionally set a credit limit for the key

### API Key Features

- Set credit limits per key
- Can be used in OAuth flows
- More powerful than direct model API keys
- Track usage per key

## Authentication Methods

### Bearer Token Authentication

Include your API key in the `Authorization` header as a Bearer token:

```http
Authorization: Bearer sk-or-v1-...
```

### Optional Headers

OpenRouter allows you to specify optional headers to identify your app:

- `HTTP-Referer`: Identifies your app on openrouter.ai (for rankings)
- `X-Title`: Sets/modifies your app's title on openrouter.ai

## Implementation Examples

### Using cURL

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -d '{
  "model": "openai/gpt-4o",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ]
}'
```

### Using OpenAI Python SDK

```python
from openai import OpenAI

client = OpenAI(
  base_url="https://openrouter.ai/api/v1",
  api_key="<OPENROUTER_API_KEY>",
)

response = client.chat.completions.create(
  extra_headers={
    "HTTP-Referer": "<YOUR_SITE_URL>",
    "X-Title": "<YOUR_SITE_NAME>",
  },
  model="openai/gpt-4o",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ],
)

reply = response.choices[0].message
```

### Using OpenAI TypeScript SDK

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: '<OPENROUTER_API_KEY>',
  defaultHeaders: {
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
  },
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: 'openai/gpt-4o',
    messages: [{ role: 'user', content: 'Say this is a test' }],
  });

  console.log(completion.choices[0].message);
}

main();
```

### Raw API with Fetch

```typescript
fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'HTTP-Referer': '<YOUR_SITE_URL>',
    'X-Title': '<YOUR_SITE_NAME>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-4o',
    messages: [
      {
        role: 'user',
        content: 'What is the meaning of life?',
      },
    ],
  }),
});
```

## Security Best Practices

### API Key Protection

⚠️ **CRITICAL**: You must protect your API keys and never commit them to public repositories.

- Store keys in environment variables
- Keep keys out of your codebase
- Never commit keys to version control
- Use secret management systems for production

### Key Compromise Response

If your key has been exposed:

1. **Immediate Action**: Visit [your key settings page](https://openrouter.ai/settings/keys)
2. **Delete** the compromised key
3. **Create** a new key
4. **Update** your applications with the new key

### Detection Systems

OpenRouter is a GitHub secret scanning partner and has multiple methods to detect exposed keys. You will receive an email notification if your key is compromised.

## Environment Variables

Recommended approach for storing API keys:

```bash
# Set in your environment
export OPENROUTER_API_KEY="sk-or-v1-..."

# Use in your application
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  ...
```

## API Key Format

OpenRouter API keys follow this format:

```
sk-or-v1-{random_string}
```

- `sk`: Indicates a secret key
- `or`: OpenRouter identifier
- `v1`: Version identifier
- Random string: Unique identifier for your key

## Related Documentation

- [API Overview](api-overview.md)
- [Limits and Quotas](limits-and-quotas.md)
- [Errors](errors.md)
