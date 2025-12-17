# OpenRouter Models Guide

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/guides/overview/models

## Overview

OpenRouter provides access to 400+ language models from 20+ providers through a unified API. All models are accessible using the same interface, with OpenRouter handling provider-specific differences automatically.

## Model Discovery

### Browsing Models

- **Website**: [https://openrouter.ai/models](https://openrouter.ai/models)
- **API**: `GET /api/v1/models`
- **RSS Feed**: [https://openrouter.ai/api/v1/models?use_rss=true](https://openrouter.ai/api/v1/models?use_rss=true)

### Models API Endpoint

```http
GET /api/v1/models
```

Returns a standardized list of all available models with comprehensive metadata.

## Models API Standard

OpenRouter's Models API provides freely available, standardized information about all LLMs. The schema is cached at the edge and designed for reliable production integration.

### Response Schema

#### Root Object

```json
{
  "data": [
    /* Array of Model objects */
  ]
}
```

#### Model Object

Each model in the `data` array contains:

| Field                  | Type       | Description                                                   |
|------------------------|------------|---------------------------------------------------------------|
| `id`                   | string     | Unique identifier for API requests (e.g., `"openai/gpt-4o"`) |
| `canonical_slug`       | string     | Permanent slug that never changes                             |
| `name`                 | string     | Human-readable display name                                   |
| `created`              | number     | Unix timestamp of when added to OpenRouter                    |
| `description`          | string     | Detailed capabilities description                             |
| `context_length`       | number     | Maximum context window size in tokens                         |
| `architecture`         | object     | Technical capabilities (see below)                            |
| `pricing`              | object     | Cost structure (see below)                                    |
| `top_provider`         | object     | Primary provider configuration                                |
| `per_request_limits`   | object     | Rate limiting info (null if no limits)                        |
| `supported_parameters` | string[]   | Array of supported API parameters                             |

### Architecture Object

```typescript
{
  "input_modalities": string[],  // e.g., ["file", "image", "text"]
  "output_modalities": string[], // e.g., ["text"]
  "tokenizer": string,           // Tokenization method
  "instruct_type": string | null // Instruction format type
}
```

**Input Modalities**:
- `text` - Text input
- `image` - Image input (vision models)
- `file` - File input

**Output Modalities**:
- `text` - Text generation
- `embeddings` - Vector embeddings

### Pricing Object

All prices in USD per token/request/unit. `"0"` indicates free.

```typescript
{
  "prompt": string,              // Cost per input token
  "completion": string,          // Cost per output token
  "request": string,             // Fixed cost per API request
  "image": string,               // Cost per image input
  "web_search": string,          // Cost per web search operation
  "internal_reasoning": string,  // Cost for internal reasoning tokens
  "input_cache_read": string,    // Cost per cached input token read
  "input_cache_write": string    // Cost per cached input token write
}
```

**Example**:
```json
{
  "prompt": "0.000003",
  "completion": "0.000015",
  "request": "0",
  "image": "0.001445"
}
```

### Top Provider Object

```typescript
{
  "context_length": number,        // Provider-specific context limit
  "max_completion_tokens": number, // Maximum tokens in response
  "is_moderated": boolean         // Whether content moderation is applied
}
```

### Supported Parameters

The `supported_parameters` array indicates which OpenAI-compatible parameters work with each model:

**Core Parameters**:
- `tools` - Function calling capabilities
- `tool_choice` - Tool selection control
- `max_tokens` - Response length limiting
- `temperature` - Randomness control
- `top_p` - Nucleus sampling
- `stop` - Custom stop sequences
- `seed` - Deterministic outputs

**Advanced Parameters**:
- `frequency_penalty` - Repetition reduction
- `presence_penalty` - Topic diversity
- `reasoning` - Internal reasoning mode
- `include_reasoning` - Include reasoning in response
- `structured_outputs` - JSON schema enforcement
- `response_format` - Output format specification

## Model Naming Convention

Models follow the format: `provider/model-name`

**Examples**:
- `openai/gpt-4o`
- `anthropic/claude-3.5-sonnet`
- `google/gemini-2.0-flash-exp`
- `meta-llama/llama-3.3-70b-instruct`

### Model Variants

Some models have special variants:

- `:free` - Free tier variant (limited availability)
- `:online` - Web search enabled variant
- `:extended` - Extended context window

**Examples**:
- `openai/gpt-3.5-turbo:free`
- `openai/gpt-4o:online`

## Tokenization Differences

⚠️ **Important**: Different models tokenize text differently.

- **GPT, Claude, Llama**: Multi-character chunks
- **PaLM**: Character-by-character

**Implications**:
- Token counts vary between models for same text
- Costs vary even with identical inputs/outputs
- Billing uses native tokenizer for accuracy

**Usage Field**: The `usage` field in responses provides token counts for the specific model used.

## Model Selection

### Automatic Selection

If `model` parameter is omitted:
- Uses user's default model
- Or payer's default model

### Manual Selection

Specify exact model from the models list:

```json
{
  "model": "openai/gpt-4o",
  "messages": [...]
}
```

### Fallback Models

Use multiple models with automatic fallback:

```json
{
  "models": ["openai/gpt-4o", "anthropic/claude-3.5-sonnet"],
  "route": "fallback",
  "messages": [...]
}
```

OpenRouter will:
1. Try the first model
2. Fall back on 5xx errors or rate limits
3. Select best available GPU/provider

## Model Categories

### By Provider

- **OpenAI**: GPT-4, GPT-3.5, etc.
- **Anthropic**: Claude models
- **Google**: Gemini models
- **Meta**: Llama models
- **Mistral AI**: Mistral models
- **Many others**: 20+ total providers

### By Capability

- **Vision Models**: Accept image inputs
- **Function Calling**: Support tool/function calling
- **Extended Context**: Large context windows (32K+)
- **Reasoning Models**: Advanced reasoning capabilities
- **Embeddings**: Generate vector embeddings

### By Cost

- **Free Tier**: Models ending in `:free`
- **Budget**: Low-cost models ($0.0001-0.001/1K tokens)
- **Standard**: Mid-range models ($0.001-0.01/1K tokens)
- **Premium**: High-end models ($0.01+/1K tokens)

## Filtering Models

### By Output Modality

Filter embeddings models:
```
https://openrouter.ai/models?fmt=cards&output_modalities=embeddings
```

### By Parameters

Check models supporting specific parameters on the models page and filter by `supported_parameters`.

## Example: Listing Models

### Using cURL

```bash
curl https://openrouter.ai/api/v1/models \
  -H "Authorization: Bearer $OPENROUTER_API_KEY"
```

### Using Python

```python
import requests

response = requests.get(
  "https://openrouter.ai/api/v1/models",
  headers={"Authorization": f"Bearer {api_key}"}
)

models = response.json()
for model in models["data"]:
  print(f"{model['id']}: ${model['pricing']['prompt']} per 1K tokens")
```

### Using TypeScript

```typescript
const response = await fetch('https://openrouter.ai/api/v1/models', {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
  },
});

const data = await response.json();
console.log(data.data.map(m => m.id));
```

## Model Metadata

### Context Length

Maximum input + output tokens supported:
- Small: 2K-8K tokens
- Medium: 16K-32K tokens
- Large: 64K-128K tokens
- Extended: 200K+ tokens

### Max Completion Tokens

Maximum tokens in a single response (varies by provider).

### Content Moderation

Some models have built-in content moderation:
- `is_moderated: true` - Provider applies content filtering
- `is_moderated: false` - No automatic moderation

## For Providers

If you're a model provider interested in listing your models on OpenRouter, visit the [providers page](https://openrouter.ai/docs/use-cases/for-providers).

## Related Documentation

- [API Overview](api-overview.md)
- [Parameters](parameters.md)
- [Authentication](authentication.md)
- [Pricing and Limits](limits-and-quotas.md)
