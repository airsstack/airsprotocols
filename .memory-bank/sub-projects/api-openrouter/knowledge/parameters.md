# OpenRouter API Parameters Reference

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/parameters

## Overview

Sampling parameters shape the token generation process of the model. OpenRouter accepts a wide range of parameters and applies sensible defaults when parameters are omitted.

## Default Values

OpenRouter will apply these defaults if parameters are absent:
- `temperature`: 1.0
- Other parameters: Model/provider-specific defaults

## Core Parameters

### Temperature

**Key**: `temperature`  
**Type**: float (0.0 to 2.0)  
**Default**: 1.0  
**Video**: [Explainer](https://youtu.be/ezgqHnWvua8)

Controls response variety and randomness:
- **Lower values (0.0-0.5)**: More predictable, deterministic, typical responses
- **Higher values (1.5-2.0)**: More diverse, creative, less common responses
- **0**: Always returns same response for given input

**Use Cases**:
- `0.0-0.3`: Factual tasks, code generation, precise answers
- `0.7-1.0`: Balanced responses, general chat
- `1.2-2.0`: Creative writing, brainstorming

**Example**:
```json
{
  "temperature": 0.7,
  "messages": [...]
}
```

### Top P

**Key**: `top_p`  
**Type**: float (0.0 to 1.0)  
**Default**: 1.0  
**Video**: [Explainer](https://youtu.be/wQP-im_HInk)

Nucleus sampling - limits model choices to top tokens whose probabilities sum to P:
- **Lower values (0.1-0.5)**: More focused, predictable choices
- **1.0**: Full range of token choices (default)
- Think of it as dynamic Top-K

**Use Cases**:
- `0.1-0.3`: Deterministic, focused responses
- `0.5-0.7`: Balanced creativity
- `0.9-1.0`: Maximum diversity

**Example**:
```json
{
  "top_p": 0.9,
  "messages": [...]
}
```

### Top K

**Key**: `top_k`  
**Type**: integer (0 or above)  
**Default**: 0 (disabled)  
**Video**: [Explainer](https://youtu.be/EbZv6-N8Xlk)

Limits token choices to K most likely tokens at each step:
- **1**: Always picks most likely token (deterministic)
- **0**: Disabled, considers all choices
- **10-50**: Good balance for most tasks

**Note**: Not available for OpenAI models.

**Example**:
```json
{
  "top_k": 40,
  "messages": [...]
}
```

### Max Tokens

**Key**: `max_tokens`  
**Type**: integer (1 or above)  
**Default**: Model-specific

Upper limit for generated tokens:
- Cannot exceed (context_length - prompt_length)
- Response stops when limit reached
- Use to control costs and response length

**Example**:
```json
{
  "max_tokens": 500,
  "messages": [...]
}
```

### Stop Sequences

**Key**: `stop`  
**Type**: string or string[]  
**Default**: None

Stop generation when specific tokens/strings encountered:

**Single stop sequence**:
```json
{
  "stop": "\n\n",
  "messages": [...]
}
```

**Multiple stop sequences**:
```json
{
  "stop": ["\n\n", "END", "###"],
  "messages": [...]
}
```

## Penalty Parameters

### Frequency Penalty

**Key**: `frequency_penalty`  
**Type**: float (-2.0 to 2.0)  
**Default**: 0.0  
**Video**: [Explainer](https://youtu.be/p4gl6fqI0_w)

Controls repetition based on token frequency:
- **Positive values**: Reduce repetition (proportional to occurrence count)
- **Negative values**: Encourage repetition
- Penalty scales with frequency

**Use Cases**:
- `0.5-1.0`: Reduce repetitive text
- `-0.5 to 0`: Allow or encourage repetition
- `1.0-2.0`: Strongly discourage repetition

**Example**:
```json
{
  "frequency_penalty": 0.5,
  "messages": [...]
}
```

### Presence Penalty

**Key**: `presence_penalty`  
**Type**: float (-2.0 to 2.0)  
**Default**: 0.0  
**Video**: [Explainer](https://youtu.be/MwHG5HL-P74)

Controls repetition of tokens already used:
- **Positive values**: Discourage reusing tokens
- **Negative values**: Encourage reuse
- **Does NOT scale** with occurrence count

**Difference from Frequency Penalty**:
- Presence: Binary (used or not used)
- Frequency: Proportional to how often used

**Example**:
```json
{
  "presence_penalty": 0.6,
  "messages": [...]
}
```

### Repetition Penalty

**Key**: `repetition_penalty`  
**Type**: float (0.0 to 2.0)  
**Default**: 1.0  
**Video**: [Explainer](https://youtu.be/LHjGAnLm3DM)

Reduces repetition from input:
- **> 1.0**: Discourages repetition
- **1.0**: No penalty (default)
- **< 1.0**: Encourages repetition
- Too high values can cause run-on sentences

**Use Cases**:
- `1.1-1.3`: Gentle repetition reduction
- `1.0`: Balanced (default)
- `0.8-0.9`: Allow more repetition

**Example**:
```json
{
  "repetition_penalty": 1.1,
  "messages": [...]
}
```

## Advanced Sampling

### Min P

**Key**: `min_p`  
**Type**: float (0.0 to 1.0)  
**Default**: 0.0

Minimum probability threshold relative to most likely token:
- Filters tokens below (min_p × max_probability)
- Dynamic threshold based on confidence
- `0.1` = only tokens with ≥10% of top token's probability

**Example**:
```json
{
  "min_p": 0.1,
  "messages": [...]
}
```

### Top A

**Key**: `top_a`  
**Type**: float (0.0 to 1.0)  
**Default**: 0.0

Dynamic filtering based on maximum probability:
- Lower values: Narrower scope based on top token
- Higher values: Doesn't increase creativity, refines filtering
- Think of it as adaptive Top-P

**Example**:
```json
{
  "top_a": 0.2,
  "messages": [...]
}
```

### Seed

**Key**: `seed`  
**Type**: integer  
**Default**: None

Enable deterministic sampling:
- Same seed + parameters = same result (usually)
- Useful for reproducibility
- Not guaranteed for all models

**Example**:
```json
{
  "seed": 12345,
  "messages": [...]
}
```

## Output Control

### Response Format

**Key**: `response_format`  
**Type**: object  
**Default**: None

Force specific output format:

**JSON Mode**:
```json
{
  "response_format": { "type": "json_object" },
  "messages": [
    {"role": "system", "content": "Return JSON with keys: name, age"},
    {"role": "user", "content": "Tell me about Alice, 30 years old"}
  ]
}
```

**Note**: 
- Supported by OpenAI models, Nitro models, some others
- Must instruct model to produce JSON via system/user message
- Check model's supported_parameters

### Structured Outputs

**Key**: `structured_outputs`  
**Type**: boolean  
**Default**: false

Enable JSON schema enforcement with `response_format`:

```json
{
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "person",
      "schema": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "age": {"type": "number"}
        },
        "required": ["name", "age"]
      }
    }
  },
  "structured_outputs": true
}
```

## Probability Parameters

### Logit Bias

**Key**: `logit_bias`  
**Type**: object (map of token_id → bias)  
**Default**: None

Bias specific tokens:
- Token ID → bias value (-100 to 100)
- Added to logits before sampling
- `-100` or `100`: Ban or force selection
- `-1` to `1`: Decrease/increase likelihood

**Example**:
```json
{
  "logit_bias": {
    "2435": 10,   // Increase likelihood of token 2435
    "5678": -10   // Decrease likelihood of token 5678
  }
}
```

### Logprobs

**Key**: `logprobs`  
**Type**: boolean  
**Default**: false

Return log probabilities for output tokens:

```json
{
  "logprobs": true,
  "messages": [...]
}
```

### Top Logprobs

**Key**: `top_logprobs`  
**Type**: integer (0 to 20)  
**Default**: None

Return N most likely tokens at each position:
- Requires `logprobs: true`
- Useful for uncertainty analysis

**Example**:
```json
{
  "logprobs": true,
  "top_logprobs": 5,
  "messages": [...]
}
```

## Tool Calling Parameters

### Tools

**Key**: `tools`  
**Type**: array of Tool objects  
**Default**: None

Define available functions:

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {"type": "string"}
          }
        }
      }
    }
  ]
}
```

See [API Overview](api-overview.md) for full tool calling guide.

### Tool Choice

**Key**: `tool_choice`  
**Type**: string or object  
**Default**: `"auto"`

Control tool calling behavior:
- `"none"`: Never call tools
- `"auto"`: Model decides
- `"required"`: Must call at least one tool
- `{"type": "function", "function": {"name": "..."}}`: Force specific tool

**Example**:
```json
{
  "tool_choice": "auto",
  "tools": [...]
}
```

### Parallel Tool Calls

**Key**: `parallel_tool_calls`  
**Type**: boolean  
**Default**: true

Enable simultaneous function calls:
- `true`: Call multiple functions in parallel
- `false`: Call functions sequentially

**Example**:
```json
{
  "parallel_tool_calls": true,
  "tools": [...]
}
```

## Provider-Specific Parameters

OpenRouter forwards provider-specific parameters directly:

**Mistral**:
```json
{
  "safe_prompt": true
}
```

**Hyperbolic**:
```json
{
  "raw_mode": true
}
```

**Note**: Check model's provider documentation for supported parameters.

## Verbosity

**Key**: `verbosity`  
**Type**: enum ("low", "medium", "high")  
**Default**: "medium"

Control response length and detail:
- `"low"`: Concise responses
- `"medium"`: Balanced (default)
- `"high"`: Detailed, comprehensive responses

**Example**:
```json
{
  "verbosity": "high",
  "messages": [...]
}
```

## Parameter Compatibility

### Non-Standard Parameters

If a model doesn't support a parameter:
- Parameter is **silently ignored**
- No error is returned
- Other parameters still processed

**Examples**:
- `logit_bias`: Only OpenAI models
- `top_k`: Not available for OpenAI models

### Checking Compatibility

Use the Models API to check `supported_parameters`:

```bash
curl https://openrouter.ai/api/v1/models | jq '.data[] | {id, supported_parameters}'
```

## Best Practices

### For Factual Tasks
```json
{
  "temperature": 0.2,
  "top_p": 0.9,
  "frequency_penalty": 0.3
}
```

### For Creative Tasks
```json
{
  "temperature": 1.2,
  "top_p": 0.95,
  "presence_penalty": 0.6
}
```

### For Code Generation
```json
{
  "temperature": 0.0,
  "top_p": 0.95,
  "frequency_penalty": 0.0,
  "stop": ["```", "\n\n\n"]
}
```

### For Deterministic Output
```json
{
  "temperature": 0.0,
  "seed": 42,
  "top_p": 1.0
}
```

## Related Documentation

- [API Overview](api-overview.md)
- [Models](models.md)
- [Streaming](streaming.md)
- [Tool Calling](api-overview.md#tool-definitions)
