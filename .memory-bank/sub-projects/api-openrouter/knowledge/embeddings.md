# OpenRouter Embeddings API

**Last Updated**: 2025-12-17
**Source**: https://openrouter.ai/docs/api/reference/embeddings

## Overview

OpenRouter's Embeddings API provides unified access to vector embedding models from multiple providers. Embeddings convert text into numerical vectors that capture semantic meaning, enabling semantic search, RAG systems, clustering, and more.

## What are Embeddings?

Embeddings transform text into high-dimensional vectors where semantically similar texts are positioned closer together in vector space.

**Example**:
- "cat" and "kitten" → Similar embeddings (close in vector space)
- "cat" and "airplane" → Different embeddings (far apart)

## Common Use Cases

### RAG (Retrieval-Augmented Generation)
Build systems that retrieve relevant context from a knowledge base before generating answers. Embeddings help find the most relevant documents.

### Semantic Search
Find documents by meaning rather than keyword matching. More accurate than traditional search because it understands context.

### Recommendation Systems
Generate embeddings for items and user preferences to recommend similar items based on semantic similarity.

### Clustering and Classification
Group similar documents or classify text into categories by analyzing embedding patterns.

### Duplicate Detection
Identify duplicate or near-duplicate content by comparing embedding similarity, even when text is paraphrased.

### Anomaly Detection
Detect unusual content by identifying embeddings that deviate from typical patterns.

## Basic Usage

### Endpoint

```
POST /api/v1/embeddings
```

### Simple Request

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'YOUR_API_KEY',
});

const response = await openRouter.embeddings.generate({
  model: 'openai/text-embedding-3-small',
  input: 'The quick brown fox jumps over the lazy dog',
});

console.log(response.data[0].embedding);
```

### cURL Example

```bash
curl https://openrouter.ai/api/v1/embeddings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -d '{
    "model": "openai/text-embedding-3-small",
    "input": "The quick brown fox jumps over the lazy dog"
  }'
```

### Python Example

```python
import requests

response = requests.post(
  "https://openrouter.ai/api/v1/embeddings",
  headers={
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json",
  },
  json={
    "model": "openai/text-embedding-3-small",
    "input": "The quick brown fox jumps over the lazy dog"
  }
)

data = response.json()
embedding = data["data"][0]["embedding"]
print(f"Embedding dimension: {len(embedding)}")
```

## Batch Processing

Generate embeddings for multiple texts in a single request:

### TypeScript

```typescript
const response = await openRouter.embeddings.generate({
  model: 'openai/text-embedding-3-small',
  input: [
    'Machine learning is a subset of artificial intelligence',
    'Deep learning uses neural networks with multiple layers',
    'Natural language processing enables computers to understand text'
  ],
});

response.data.forEach((item, index) => {
  console.log(`Embedding ${index}: ${item.embedding.length} dimensions`);
});
```

### Python

```python
response = requests.post(
  "https://openrouter.ai/api/v1/embeddings",
  headers={
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json",
  },
  json={
    "model": "openai/text-embedding-3-small",
    "input": [
      "Machine learning is a subset of artificial intelligence",
      "Deep learning uses neural networks with multiple layers",
      "Natural language processing enables computers to understand text"
    ]
  }
)

data = response.json()
for i, item in enumerate(data["data"]):
  print(f"Embedding {i}: {len(item['embedding'])} dimensions")
```

## Response Format

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [0.0023, -0.0042, 0.0056, ...],
      "index": 0
    }
  ],
  "model": "openai/text-embedding-3-small",
  "usage": {
    "prompt_tokens": 10,
    "total_tokens": 10
  }
}
```

## Available Models

### Viewing Embeddings Models

**Website**: [https://openrouter.ai/models?output_modalities=embeddings](https://openrouter.ai/models?fmt=cards&output_modalities=embeddings)

**API**:
```bash
curl https://openrouter.ai/api/v1/embeddings/models \
  -H "Authorization: Bearer $OPENROUTER_API_KEY"
```

### Popular Models

- `openai/text-embedding-3-small` - Balanced quality/cost
- `openai/text-embedding-3-large` - Highest quality
- `qwen/qwen3-embedding-0.6b` - Fast and cheap
- Provider-specific models from various vendors

## Semantic Search Example

Complete example building a semantic search system:

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: 'YOUR_API_KEY',
});

// Sample documents
const documents = [
  "The cat sat on the mat",
  "Dogs are loyal companions",
  "Python is a programming language",
  "Machine learning models require training data",
  "The weather is sunny today"
];

// Cosine similarity function
function cosineSimilarity(a: number[], b: number[]): number {
  const dotProduct = a.reduce((sum, val, i) => sum + val * b[i], 0);
  const magnitudeA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
  const magnitudeB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
  return dotProduct / (magnitudeA * magnitudeB);
}

async function semanticSearch(query: string, documents: string[]) {
  // Generate embeddings for all documents and the query
  const response = await openRouter.embeddings.generate({
    model: 'openai/text-embedding-3-small',
    input: [query, ...documents],
  });

  const queryEmbedding = response.data[0].embedding;
  const docEmbeddings = response.data.slice(1);

  // Calculate similarity scores
  const results = documents.map((doc, i) => ({
    document: doc,
    similarity: cosineSimilarity(
      queryEmbedding as number[],
      docEmbeddings[i].embedding as number[]
    ),
  }));

  // Sort by similarity (highest first)
  results.sort((a, b) => b.similarity - a.similarity);

  return results;
}

// Search for documents related to pets
const results = await semanticSearch("pets and animals", documents);
console.log("Search results:");
results.forEach((result, i) => {
  console.log(`${i + 1}. ${result.document} (similarity: ${result.similarity.toFixed(4)})`);
});
```

### Python Semantic Search

```python
import requests
import numpy as np

api_key = "YOUR_API_KEY"

documents = [
  "The cat sat on the mat",
  "Dogs are loyal companions",
  "Python is a programming language",
  "Machine learning models require training data",
  "The weather is sunny today"
]

def cosine_similarity(a, b):
  """Calculate cosine similarity between two vectors"""
  dot_product = np.dot(a, b)
  magnitude_a = np.linalg.norm(a)
  magnitude_b = np.linalg.norm(b)
  return dot_product / (magnitude_a * magnitude_b)

def semantic_search(query, documents):
  """Perform semantic search using embeddings"""
  response = requests.post(
    "https://openrouter.ai/api/v1/embeddings",
    headers={
      "Authorization": f"Bearer {api_key}",
      "Content-Type": "application/json",
    },
    json={
      "model": "openai/text-embedding-3-small",
      "input": [query] + documents
    }
  )
  
  data = response.json()
  query_embedding = np.array(data["data"][0]["embedding"])
  doc_embeddings = [np.array(item["embedding"]) for item in data["data"][1:]]
  
  # Calculate similarity scores
  results = []
  for i, doc in enumerate(documents):
    similarity = cosine_similarity(query_embedding, doc_embeddings[i])
    results.append({"document": doc, "similarity": similarity})
  
  # Sort by similarity (highest first)
  results.sort(key=lambda x: x["similarity"], reverse=True)
  
  return results

# Search for documents related to pets
results = semantic_search("pets and animals", documents)
print("Search results:")
for i, result in enumerate(results):
  print(f"{i + 1}. {result['document']} (similarity: {result['similarity']:.4f})")
```

## Best Practices

### 1. Choose the Right Model
- **Small models** (qwen-0.6b, text-embedding-3-small): Faster, cheaper, good for most tasks
- **Large models** (text-embedding-3-large): Better quality, higher cost
- Test multiple models to find the best fit

### 2. Batch Your Requests
Send multiple texts in one request rather than individual API calls. Reduces latency and costs.

### 3. Cache Embeddings
Embeddings for the same text are deterministic. Store them in a database or vector store to avoid regenerating.

### 4. Use Cosine Similarity
For comparing embeddings, use cosine similarity rather than Euclidean distance. It's scale-invariant and works better for high-dimensional vectors.

### 5. Consider Context Length
Each model has a maximum input length. Check the model's context window before processing long documents.

### 6. Chunk Long Documents
For long documents, split into meaningful chunks (paragraphs, sections) rather than arbitrary character limits. Preserves semantic coherence.

### 7. Normalize Vectors
Some applications benefit from L2-normalized embeddings. Check your use case requirements.

## Provider Routing

Control which providers serve your embeddings requests:

```typescript
{
  "model": "openai/text-embedding-3-small",
  "input": "Your text here",
  "provider": {
    "order": ["openai", "azure"],
    "allow_fallbacks": true,
    "data_collection": "deny"
  }
}
```

See [API Overview](api-overview.md) for more on provider routing.

## Error Handling

### Common Errors

**400 Bad Request**: Invalid input format or missing parameters
- Check `input` and `model` are correctly formatted

**401 Unauthorized**: Invalid or missing API key
- Verify API key in Authorization header

**402 Payment Required**: Insufficient credits
- Add credits to your account

**404 Not Found**: Model doesn't exist or isn't available for embeddings
- Check model name and verify it's an embedding model

**429 Too Many Requests**: Rate limit exceeded
- Implement exponential backoff and retry logic

**529 Provider Overloaded**: Provider temporarily overloaded
- Enable `allow_fallbacks: true` for automatic backup providers

### Example Error Response

```json
{
  "error": {
    "code": 400,
    "message": "Invalid model specified",
    "type": "invalid_request_error"
  }
}
```

## Limitations

### No Streaming
Embeddings are returned as complete responses. Streaming is not supported.

### Token Limits
Each model has a maximum input length. Texts exceeding this will be truncated or rejected.

### Deterministic Output
Embeddings for the same input are always identical. No temperature or randomness.

### Language Support
Some models are optimized for specific languages. Check model documentation for language capabilities.

## Model Dimensions

Different models produce embeddings with different dimensions:

- `text-embedding-3-small`: 1536 dimensions
- `text-embedding-3-large`: 3072 dimensions
- `qwen3-embedding-0.6b`: 768 dimensions

**Note**: You cannot compare embeddings from different models or mix dimensions.

## Related Documentation

- [Models](models.md)
- [API Overview](api-overview.md)
- [Authentication](authentication.md)
- [Errors](errors.md)
