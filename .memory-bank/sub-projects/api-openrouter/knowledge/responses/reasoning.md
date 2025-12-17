# Responses API Beta - Reasoning

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/responses/reasoning

⚠️ **Beta API**: This API is in beta stage and may have breaking changes.

## Overview

The Responses API Beta supports advanced reasoning capabilities, allowing models to show their internal reasoning process with configurable effort levels.

## Reasoning Configuration

Enable reasoning with the `reasoning` parameter:

```typescript
{
  "model": "openai/o4-mini",
  "input": "What is the meaning of life?",
  "reasoning": {
    "effort": "high"
  },
  "max_output_tokens": 9000
}
```

## Effort Levels

| Level      | Description                                   | Use Cases                          |
|------------|-----------------------------------------------|------------------------------------|
| `minimal`  | Basic reasoning, minimal computation          | Simple questions, quick answers    |
| `low`      | Light reasoning for simple problems           | Straightforward tasks              |
| `medium`   | Balanced reasoning for moderate complexity    | General problem-solving            |
| `high`     | Deep reasoning for complex problems           | Math, logic, complex analysis      |

## Complex Reasoning Example

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
          "text": "Was 1995 30 years ago? Please show your reasoning."
        }
      ]
    }
  ],
  "reasoning": {
    "effort": "high"
  },
  "max_output_tokens": 9000
}
```

## Response with Reasoning

```json
{
  "id": "resp_123",
  "output": [
    {
      "type": "reasoning",
      "id": "rs_abc",
      "encrypted_content": "gAAAAABotI9-FK1PbhZhaZk4yMrZw3XDI1AWFa...",
      "summary": [
        "First, I need to determine the current year",
        "Then calculate the difference from 1995",
        "Finally, compare that to 30 years"
      ]
    },
    {
      "type": "message",
      "id": "msg_xyz",
      "status": "completed",
      "role": "assistant",
      "content": [
        {
          "type": "output_text",
          "text": "Yes. In 2025, 1995 was 30 years ago...",
          "annotations": []
        }
      ]
    }
  ],
  "usage": {
    "input_tokens": 15,
    "output_tokens": 85,
    "output_tokens_details": {
      "reasoning_tokens": 45
    },
    "total_tokens": 100
  }
}
```

## Streaming Reasoning

See reasoning develop in real-time:

```typescript
{
  "model": "openai/o4-mini",
  "input": "Solve: If a train travels 60 mph for 2.5 hours, how far does it go?",
  "reasoning": {
    "effort": "medium"
  },
  "stream": true
}
```

Process reasoning deltas:

```typescript
for await (const chunk of stream) {
  if (chunk.type === 'response.reasoning.delta') {
    console.log('Reasoning:', chunk.delta);
  }
}
```

## Reasoning in Conversation

```typescript
{
  "model": "openai/o4-mini",
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "What is your favorite color?"}]
    },
    {
      "type": "message",
      "role": "assistant",
      "id": "msg_1",
      "status": "completed",
      "content": [
        {"type": "output_text", "text": "I don't have a favorite color.", "annotations": []}
      ]
    },
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "How many Earths can fit on Mars?"}]
    }
  ],
  "reasoning": {
    "effort": "high"
  }
}
```

## Best Practices

1. **Choose Appropriate Effort**: Use `high` for complex problems, `low` for simple tasks
2. **Consider Token Usage**: Reasoning increases token consumption
3. **Use Streaming**: For long reasoning chains, streaming provides better UX
4. **Include Context**: Provide sufficient context for effective reasoning

## Usage Tracking

Reasoning tokens are tracked separately in `usage.output_tokens_details.reasoning_tokens`.

## Related Documentation

- [Basic Usage](basic-usage.md)
- [Tool Calling](tool-calling.md)
- [Web Search](web-search.md)
- [Overview](overview.md)
