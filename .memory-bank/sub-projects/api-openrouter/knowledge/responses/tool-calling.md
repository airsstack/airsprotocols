# Responses API Beta - Tool Calling

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/responses/tool-calling

⚠️ **Beta API**: This API is in beta stage and may have breaking changes.

## Overview

The Responses API Beta supports comprehensive tool calling capabilities, allowing models to call functions, execute tools in parallel, and handle complex multi-step workflows.

## Basic Tool Definition

Define tools using OpenAI function calling format:

```typescript
const weatherTool = {
  type: 'function' as const,
  name: 'get_weather',
  description: 'Get the current weather in a location',
  strict: null,
  parameters: {
    type: 'object',
    properties: {
      location: {
        type: 'string',
        description: 'The city and state, e.g. San Francisco, CA',
      },
      unit: {
        type: 'string',
        enum: ['celsius', 'fahrenheit'],
      },
    },
    required: ['location'],
  },
};
```

## Tool Choice Options

| Tool Choice                              | Description                       |
|------------------------------------------|-----------------------------------|
| `auto`                                   | Model decides whether to use tools|
| `none`                                   | Model will not call any tools     |
| `{type: 'function', name: 'tool_name'}`  | Force specific tool call          |

## Using Tools

```typescript
{
  "model": "openai/o4-mini",
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": [
        {"type": "input_text", "text": "What is the weather in San Francisco?"}
      ]
    }
  ],
  "tools": [weatherTool],
  "tool_choice": "auto",
  "max_output_tokens": 9000
}
```

## Multiple Tools

```typescript
const calculatorTool = {
  type: 'function' as const,
  name: 'calculate',
  description: 'Perform mathematical calculations',
  strict: null,
  parameters: {
    type: 'object',
    properties: {
      expression: {
        type: 'string',
        description: 'The mathematical expression to evaluate',
      },
    },
    required: ['expression'],
  },
};

{
  "model": "openai/o4-mini",
  "input": "What is 25 * 4?",
  "tools": [weatherTool, calculatorTool],
  "tool_choice": "auto"
}
```

## Parallel Tool Calls

The API supports calling multiple tools simultaneously:

```typescript
{
  "model": "openai/o4-mini",
  "input": "Calculate 10*5 and also tell me the weather in Miami",
  "tools": [weatherTool, calculatorTool],
  "tool_choice": "auto"
}
```

## Tool Call Response

```json
{
  "id": "resp_123",
  "output": [
    {
      "type": "function_call",
      "id": "fc_abc123",
      "call_id": "call_xyz789",
      "name": "get_weather",
      "arguments": "{\"location\":\"San Francisco, CA\"}"
    }
  ],
  "status": "completed"
}
```

## Tool Responses in Conversation

Include tool responses in follow-up requests:

```typescript
{
  "model": "openai/o4-mini",
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "What is the weather in Boston?"}]
    },
    {
      "type": "function_call",
      "id": "fc_1",
      "call_id": "call_123",
      "name": "get_weather",
      "arguments": "{\"location\": \"Boston, MA\"}"
    },
    {
      "type": "function_call_output",
      "id": "fc_output_1",
      "call_id": "call_123",
      "output": "{\"temperature\": \"72°F\", \"condition\": \"Sunny\"}"
    },
    {
      "type": "message",
      "role": "assistant",
      "id": "msg_abc",
      "status": "completed",
      "content": [
        {"type": "output_text", "text": "The weather in Boston is 72°F and sunny.", "annotations": []}
      ]
    },
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "Is that good for a picnic?"}]
    }
  ]
}
```

ℹ️ **Required Field**: The `id` field is required for `function_call_output` objects.

## Streaming Tool Calls

Monitor tool calls in real-time:

```typescript
for await (const chunk of stream) {
  if (chunk.type === 'response.output_item.added' && 
      chunk.item?.type === 'function_call') {
    console.log('Function call:', chunk.item.name);
  }
  
  if (chunk.type === 'response.function_call_arguments.done') {
    console.log('Arguments:', chunk.arguments);
  }
}
```

## Tool Validation

Required fields for function calls:

- `type`: Always "function_call"
- `id`: Unique identifier for the function call object
- `name`: Function name matching tool definition
- `arguments`: Valid JSON string with function parameters
- `call_id`: Unique identifier for the call

## Best Practices

1. **Clear Descriptions**: Provide detailed function descriptions and parameter explanations
2. **Proper Schemas**: Use valid JSON Schema for parameters
3. **Error Handling**: Handle cases where tools might not be called
4. **Parallel Execution**: Design tools to work independently when possible
5. **Conversation Flow**: Include tool responses in follow-up requests for context

## Related Documentation

- [Basic Usage](basic-usage.md)
- [Reasoning](reasoning.md)
- [Web Search](web-search.md)
- [Overview](overview.md)
