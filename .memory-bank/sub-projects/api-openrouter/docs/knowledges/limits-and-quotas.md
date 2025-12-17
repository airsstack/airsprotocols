# OpenRouter API Limits and Quotas

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/limits

## Overview

OpenRouter implements rate limiting and credit-based quotas to ensure fair usage and system stability. Limits are managed globally across your account, not per API key.

⚠️ **Important**: Making additional accounts or API keys will **not** affect your rate limits, as capacity is governed globally. However, different models have different rate limits, so you can share load across models.

## Checking Rate Limits and Credits

### Endpoint

```
GET /api/v1/key
```

Returns information about your API key's rate limits, credit usage, and remaining balance.

### Example Request

**TypeScript SDK**:
```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'YOUR_API_KEY',
});

const keyInfo = await openRouter.apiKeys.getCurrent();
console.log(keyInfo);
```

**Python**:
```python
import requests

response = requests.get(
  url="https://openrouter.ai/api/v1/key",
  headers={
    "Authorization": f"Bearer {api_key}"
  }
)

print(response.json())
```

**cURL**:
```bash
curl https://openrouter.ai/api/v1/key \
  -H "Authorization: Bearer $OPENROUTER_API_KEY"
```

### Response Format

```typescript
type Key = {
  data: {
    label: string;
    limit: number | null;              // Credit limit, or null if unlimited
    limit_reset: string | null;        // Reset type, or null if never resets
    limit_remaining: number | null;    // Remaining credits, or null if unlimited
    include_byok_in_limit: boolean;    // Include external BYOK usage in limit

    usage: number;                     // Credits used (all time)
    usage_daily: number;               // Credits used (current UTC day)
    usage_weekly: number;              // Credits used (current UTC week, Monday start)
    usage_monthly: number;             // Credits used (current UTC month)

    byok_usage: number;                // External BYOK usage (all time)
    byok_usage_daily: number;          // External BYOK (current UTC day)
    byok_usage_weekly: number;         // External BYOK (current UTC week)
    byok_usage_monthly: number;        // External BYOK (current UTC month)

    is_free_tier: boolean;             // Whether user has paid for credits
    // rate_limit: { ... }              // Deprecated, safe to ignore
  };
};
```

### Example Response

```json
{
  "data": {
    "label": "My API Key",
    "limit": 100.0,
    "limit_reset": "monthly",
    "limit_remaining": 75.5,
    "include_byok_in_limit": false,
    "usage": 24.5,
    "usage_daily": 5.2,
    "usage_weekly": 15.3,
    "usage_monthly": 24.5,
    "byok_usage": 0,
    "byok_usage_daily": 0,
    "byok_usage_weekly": 0,
    "byok_usage_monthly": 0,
    "is_free_tier": false
  }
}
```

## Rate Limits

### Free Model Limits

Free models (IDs ending in `:free`) have specific rate limits:

**Requests per Minute**: Up to a certain threshold (model-specific)

**Requests per Day**:
- **No credits purchased**: Limited daily requests
- **≥$5 credits purchased**: Increased daily limit

### Model-Specific Limits

Different models have different rate limits. Check the [models page](https://openrouter.ai/models) for model-specific limits.

### DDoS Protection

Cloudflare's DDoS protection will block requests that dramatically exceed reasonable usage patterns.

## Credit System

### Credit Balance

- **Positive balance**: Full access to all models (paid and free)
- **Negative balance**: May see `402 Payment Required` errors, even for free models
- **Solution**: Add credits to restore balance above zero

### Credit Limits per Key

You can set credit limits when creating API keys:
- Prevents overspending
- Useful for testing or limiting app usage
- Key stops working when limit reached

### Adding Credits

Add credits via the [OpenRouter dashboard](https://openrouter.ai/credits).

## HTTP Status Codes

### 402 Payment Required

**Cause**: Insufficient credits or negative balance

**Solution**:
1. Check credit balance: `GET /api/v1/key`
2. Add credits to your account
3. Retry the request

**Example Error**:
```json
{
  "error": {
    "code": 402,
    "message": "Insufficient credits. Please add credits to your account.",
    "type": "insufficient_credits"
  }
}
```

### 429 Too Many Requests

**Cause**: Rate limit exceeded

**Solution**:
1. Implement exponential backoff
2. Spread requests across different models
3. Wait for rate limit window to reset

**Response Headers**:
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1234567890
```

**Example Error**:
```json
{
  "error": {
    "code": 429,
    "message": "Rate limit exceeded. Please try again later.",
    "type": "rate_limit_exceeded"
  }
}
```

## Rate Limit Headers

OpenRouter includes rate limit information in response headers:

| Header                    | Description                                      |
|---------------------------|--------------------------------------------------|
| `X-RateLimit-Limit`       | Maximum requests allowed in the time window      |
| `X-RateLimit-Remaining`   | Remaining requests in current window             |
| `X-RateLimit-Reset`       | Unix timestamp when the limit resets             |
| `X-OpenRouter-Cost`       | Dollar cost of the request (e.g., "0.000123")   |

### Example

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1234567890
X-OpenRouter-Cost: 0.000456
Content-Type: application/json
```

## Cost Tracking

### Per-Request Cost

Check the `X-OpenRouter-Cost` header for the exact cost of each request:

```typescript
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: { /* ... */ },
  body: JSON.stringify({ /* ... */ }),
});

const cost = response.headers.get('X-OpenRouter-Cost');
console.log(`Request cost: $${cost}`);
```

### Usage Tracking

Use the `/api/v1/key` endpoint to track usage over different time periods:
- **All time**: `usage`
- **Daily**: `usage_daily`
- **Weekly**: `usage_weekly`
- **Monthly**: `usage_monthly`

### Generation Stats

For precise token accounting, query the generation endpoint:

```typescript
const generation = await fetch(
  `https://openrouter.ai/api/v1/generation?id=${generationId}`,
  { headers: { Authorization: `Bearer ${apiKey}` } }
);

const stats = await generation.json();
// Contains native token counts and exact costs
```

## Best Practices

### 1. Implement Retry Logic

Handle rate limits gracefully with exponential backoff:

```typescript
async function requestWithRetry(url: string, options: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      const resetTime = response.headers.get('X-RateLimit-Reset');
      const waitTime = resetTime 
        ? parseInt(resetTime) * 1000 - Date.now()
        : Math.pow(2, i) * 1000;
      
      await new Promise(resolve => setTimeout(resolve, waitTime));
      continue;
    }
    
    return response;
  }
  
  throw new Error('Max retries exceeded');
}
```

### 2. Monitor Usage

Regularly check your credit usage and remaining balance:

```typescript
async function checkUsage() {
  const keyInfo = await fetch('https://openrouter.ai/api/v1/key', {
    headers: { Authorization: `Bearer ${apiKey}` }
  });
  
  const data = await keyInfo.json();
  
  if (data.data.limit_remaining < 10) {
    console.warn('Low credits remaining!');
  }
}
```

### 3. Set Budget Alerts

Create API keys with credit limits to prevent overspending:
1. Go to [https://openrouter.ai/keys](https://openrouter.ai/keys)
2. Create key with specific credit limit
3. Monitor usage via `/api/v1/key`

### 4. Distribute Load

Spread requests across multiple models to utilize different rate limit pools.

### 5. Cache Responses

Cache API responses when appropriate to reduce request volume.

### 6. Use Streaming Wisely

Streaming doesn't bypass rate limits but provides better UX for long responses.

## Account Tiers

### Free Tier

**Characteristics**:
- `is_free_tier: true`
- Access to `:free` model variants
- Limited daily requests
- Lower rate limits

**Upgrade**: Purchase at least $5 in credits

### Paid Tier

**Characteristics**:
- `is_free_tier: false`
- Higher rate limits for `:free` models
- Access to all paid models
- Better rate limit quotas

## BYOK (Bring Your Own Key)

If using your own provider API keys:
- BYOK usage tracked separately
- `include_byok_in_limit`: Controls if BYOK counts toward credit limit
- Separate usage fields: `byok_usage`, `byok_usage_daily`, etc.

## Troubleshooting

### "Rate limit exceeded" but low usage?

- Check if you're hitting model-specific limits
- Different models have different quotas
- Try spreading load across models

### "Payment required" for free models?

- Check if account balance is negative
- Free tier models still require positive balance
- Add credits to restore access

### Can't increase limits with new keys?

- Limits are **global per account**, not per key
- Additional keys won't increase rate limits
- Use different models to access separate limit pools

## Related Documentation

- [Errors](errors.md)
- [Authentication](authentication.md)
- [Models](models.md)
- [API Overview](api-overview.md)
