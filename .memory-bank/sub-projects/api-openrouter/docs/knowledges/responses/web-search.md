# Responses API Beta - Web Search

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/responses/web-search

⚠️ **Beta API**: This API is in beta stage and may have breaking changes.

## Overview

The Responses API Beta supports web search integration, allowing models to access real-time information from the internet and provide responses with proper citations and annotations.

## Web Search Plugin

Enable web search using the `plugins` parameter:

```typescript
{
  "model": "openai/o4-mini",
  "input": "What is OpenRouter?",
  "plugins": [{ "id": "web", "max_results": 3 }],
  "max_output_tokens": 9000
}
```

## Plugin Configuration

| Parameter     | Type    | Description                         |
|---------------|---------|-------------------------------------|
| `id`          | string  | **Required.** Must be "web"         |
| `max_results` | integer | Maximum search results (1-10)       |

## Structured Message with Web Search

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
          "text": "What was a positive news story from today?"
        }
      ]
    }
  ],
  "plugins": [{ "id": "web", "max_results": 2 }]
}
```

## Online Model Variants

Some models have built-in web search via `:online` variant:

```typescript
{
  "model": "openai/o4-mini:online",
  "input": "What was a positive news story from today?",
  "max_output_tokens": 9000
}
```

## Response with Annotations

Web search responses include citation annotations:

```json
{
  "id": "resp_123",
  "output": [
    {
      "type": "message",
      "id": "msg_abc",
      "status": "completed",
      "role": "assistant",
      "content": [
        {
          "type": "output_text",
          "text": "OpenRouter is a unified API for accessing multiple LLM providers...",
          "annotations": [
            {
              "type": "url_citation",
              "url": "https://openrouter.ai/docs",
              "start_index": 0,
              "end_index": 85
            }
          ]
        }
      ]
    }
  ]
}
```

## Annotation Types

### URL Citation

```json
{
  "type": "url_citation",
  "url": "https://example.com/article",
  "start_index": 0,
  "end_index": 50
}
```

## Complex Search Queries

```typescript
{
  "model": "openai/o4-mini",
  "input": "Compare OpenAI and Anthropic latest models",
  "plugins": [{ "id": "web", "max_results": 5 }]
}
```

## Web Search in Conversation

```typescript
{
  "model": "openai/o4-mini",
  "input": [
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "What is the latest version of React?"}]
    },
    {
      "type": "message",
      "id": "msg_1",
      "status": "in_progress",
      "role": "assistant",
      "content": [
        {"type": "output_text", "text": "Let me search for that.", "annotations": []}
      ]
    },
    {
      "type": "message",
      "role": "user",
      "content": [{"type": "input_text", "text": "Yes, please find the most recent information"}]
    }
  ],
  "plugins": [{ "id": "web", "max_results": 2 }]
}
```

## Streaming Web Search

```typescript
for await (const chunk of stream) {
  if (chunk.type === 'response.completed') {
    const annotations = chunk.response?.output
      ?.find(o => o.type === 'message')
      ?.content?.find(c => c.type === 'output_text')
      ?.annotations || [];
    console.log('Citations:', annotations.length);
  }
}
```

## Annotation Processing

Extract and process citations:

```typescript
function extractCitations(response: any) {
  const messageOutput = response.output?.find((o: any) => o.type === 'message');
  const textContent = messageOutput?.content?.find((c: any) => c.type === 'output_text');
  const annotations = textContent?.annotations || [];

  return annotations
    .filter((a: any) => a.type === 'url_citation')
    .map((a: any) => ({
      url: a.url,
      text: textContent.text.slice(a.start_index, a.end_index),
      startIndex: a.start_index,
      endIndex: a.end_index,
    }));
}
```

## Best Practices

1. **Limit Results**: Use appropriate `max_results` to balance quality and speed
2. **Handle Annotations**: Process citation annotations for proper attribution
3. **Query Specificity**: Make search queries specific for better results
4. **Error Handling**: Handle cases where web search might fail
5. **Rate Limits**: Be mindful of search rate limits

## Related Documentation

- [Basic Usage](basic-usage.md)
- [Tool Calling](tool-calling.md)
- [Reasoning](reasoning.md)
- [Overview](overview.md)
