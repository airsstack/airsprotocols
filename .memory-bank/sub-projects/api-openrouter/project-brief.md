# api-openrouter - Project Brief

**Version**: 0.1.0 (Planning)  
**Last Updated**: 2025-12-17  
**Status**: 🚀 New Project - Planning Phase  
**Type**: Library Crate

## Vision

Provide the Rust ecosystem with a production-ready, type-safe, and ergonomic client library for the OpenRouter API, enabling developers to access multiple LLM providers through a unified interface with cost optimization and reliability features.

## Mission

Deliver a comprehensive OpenRouter Rust client that:
1. Implements OpenRouter API with full feature coverage
2. Provides type-safe request/response structures
3. Offers ergonomic builder patterns for API calls
4. Enables streaming responses with async/await
5. Includes cost tracking and usage monitoring
6. Maintains excellent documentation and examples

## Project Overview

`airsprotocols-api-openrouter` is a Rust client library for OpenRouter - an LLM API aggregator that provides unified access to multiple providers (OpenAI, Anthropic, Google, Meta, Mistral, and more) through a single API endpoint.

**Key Features**:
- **Multi-Provider Access**: Single API for GPT-4, Claude, Gemini, Llama, and 100+ models
- **Chat Completions**: Streaming and non-streaming chat with message history
- **Type Safety**: Strongly-typed models, messages, and responses
- **Cost Optimization**: Built-in cost tracking and provider routing
- **Async-First**: Built on tokio for high-performance concurrent operations
- **Error Handling**: Production-grade error handling with actionable errors

## Core Objectives

### 1. API Implementation ⏳ PLANNED
- [ ] OpenRouter authentication (API key via bearer token)
- [ ] Chat completions endpoint (POST /api/v1/chat/completions)
- [ ] Streaming responses via Server-Sent Events (SSE)
- [ ] Model listing and metadata
- [ ] Usage and cost tracking
- [ ] Rate limit handling

### 2. Type-Safe API ⏳ PLANNED
- [ ] Strongly-typed message structures (System, User, Assistant)
- [ ] Model definitions with provider metadata
- [ ] Request builders with validation
- [ ] Response parsers with error handling
- [ ] Type-safe streaming iterators

### 3. Client Architecture ⏳ PLANNED
- [ ] OpenRouterClient with async methods
- [ ] Request builder pattern for ergonomic API
- [ ] Connection pooling with reqwest
- [ ] Retry logic with exponential backoff
- [ ] Timeout configuration

### 4. Streaming Support ⏳ PLANNED
- [ ] SSE-based streaming implementation
- [ ] Async stream abstraction (tokio-stream)
- [ ] Chunk parsing and reassembly
- [ ] Error handling in streams
- [ ] Stream cancellation support

### 5. Observability ⏳ PLANNED
- [ ] Cost tracking per request
- [ ] Token usage monitoring
- [ ] Request/response logging with tracing
- [ ] Metrics for latency and throughput
- [ ] Error categorization

### 6. Documentation ⏳ PLANNED
- [ ] API reference (rustdoc)
- [ ] Quick start guide
- [ ] Example projects (basic chat, streaming, multi-model)
- [ ] Integration patterns
- [ ] Migration guide (from OpenAI SDK)

## Target Audience

### Primary Users
1. **Rust Developers** building AI applications with LLMs
2. **Multi-Model Teams** needing flexibility across providers
3. **Cost-Conscious Projects** optimizing LLM expenses
4. **Enterprise Teams** requiring provider redundancy

### Secondary Users
1. AI researchers prototyping with multiple models
2. SaaS developers building LLM-powered features
3. Tool developers creating AI integrations

## Success Criteria

### Technical Excellence
- [ ] Full OpenRouter API coverage (v1)
- [ ] <100ms client-side overhead (beyond network latency)
- [ ] >80% test coverage
- [ ] Zero unsafe code in public APIs
- [ ] Production-grade error handling with thiserror

### Developer Experience
- [ ] Clear API documentation with examples
- [ ] Quick start guide (<10 min to first request)
- [ ] Intuitive builder APIs
- [ ] Helpful error messages
- [ ] Migration examples from OpenAI SDK

### Ecosystem Impact
- [ ] Published to crates.io (target: Q2 2025)
- [ ] Referenced in OpenRouter community
- [ ] Integration examples with popular frameworks
- [ ] Active community engagement

## Project Boundaries

### In Scope
- OpenRouter API client implementation
- Chat completions (streaming and non-streaming)
- Model management and discovery
- Cost tracking and usage monitoring
- Request/response type definitions
- Error handling and retry logic
- Documentation and examples

### Out of Scope
- LLM model inference or training
- OpenRouter API server implementation
- UI/UX components
- Database adapters
- Prompt engineering frameworks
- Multi-tenancy or user management
- Custom model deployment

## Technical Constraints

### Technology Stack
- **Language**: Rust 1.88.0+
- **Async Runtime**: Tokio
- **HTTP Client**: reqwest with connection pooling
- **Serialization**: serde + serde_json
- **Streaming**: tokio-stream, eventsource-stream
- **Error Handling**: thiserror

### Design Constraints
- Must support async/await patterns
- Must handle streaming responses efficiently
- Zero-cost abstractions where possible
- Strong type safety (no stringly-typed APIs)
- Minimal external dependencies

### Performance Constraints
- Client-side overhead: <100ms per request
- Streaming latency: <50ms first token
- Memory footprint: <20MB baseline
- Concurrent requests: 100+ simultaneous
- Connection reuse via pooling

## Architecture Philosophy

### Type Safety First
Leverage Rust's type system to prevent API misuse at compile-time. Strongly-typed models, messages, and responses.

### Async Native
Built on tokio for non-blocking I/O and efficient concurrent operations. All public futures are Send.

### Ergonomic API
Builder patterns for intuitive request construction. Sensible defaults with easy customization.

### Production Ready
- Comprehensive error handling with actionable errors
- Retry logic with exponential backoff
- Rate limit handling
- Cost tracking and monitoring
- Extensive testing

### OpenRouter-Specific Features
- Multi-provider routing and fallbacks
- Cost optimization strategies
- Provider-specific parameters
- Model metadata and capabilities

## Current Status

### Version: 0.1.0 (Planning)
**Status**: 🚀 New Project - Planning Phase

### Next Steps
1. Research OpenRouter API documentation
2. Define core type structures (Message, Model, Request, Response)
3. Implement basic client with authentication
4. Add chat completions endpoint (non-streaming)
5. Add streaming support with SSE
6. Create example projects
7. Write comprehensive documentation

### Known Considerations
- OpenRouter API is similar to OpenAI but with provider routing
- Need to handle provider-specific parameters
- Cost tracking requires parsing response headers
- Streaming uses Server-Sent Events (SSE)
- Rate limits are per-API-key and vary by tier

## Dependencies

### Runtime Dependencies (Planned)
- `tokio` - Async runtime
- `reqwest` - HTTP client with connection pooling
- `serde`, `serde_json` - Serialization
- `thiserror` - Error handling
- `tokio-stream` - Async stream utilities
- `eventsource-stream` - SSE parsing
- `async-trait` - Async trait support (if needed)

### Build Dependencies
- Rust 1.88.0+ toolchain
- cargo for building and testing

### Optional Dependencies
- `tracing` - Structured logging
- `metrics` - Metrics collection
- `airsprotocols-mcp` - For MCP provider integration (future)

## Roadmap

### Phase 1: Foundation (Q2 2025) ⏳
- [ ] Project setup and structure
- [ ] Core type definitions
- [ ] Basic client implementation
- [ ] Authentication with API key
- [ ] Chat completions (non-streaming)
- [ ] Basic error handling

### Phase 2: Streaming & Reliability (Q2 2025) ⏳
- [ ] Streaming chat completions via SSE
- [ ] Retry logic with exponential backoff
- [ ] Rate limit handling
- [ ] Connection pooling optimization
- [ ] Timeout configuration

### Phase 3: Observability (Q2 2025) ⏳
- [ ] Cost tracking and reporting
- [ ] Token usage monitoring
- [ ] Structured logging with tracing
- [ ] Request/response metrics
- [ ] Error categorization

### Phase 4: Documentation & Release (Q2 2025) ⏳
- [ ] Comprehensive API documentation
- [ ] Example projects (basic, streaming, multi-model)
- [ ] Integration guides
- [ ] Migration guide from OpenAI SDK
- [ ] Publish to crates.io (v0.1.0)

### Future Enhancements (Q3 2025+)
- [ ] Model metadata and capabilities API
- [ ] Provider preference and routing strategies
- [ ] Advanced cost optimization
- [ ] Integration with protocols-mcp as MCP provider
- [ ] CLI tool for testing and exploration

## Integration with AirsProtocols Workspace

### Relationship to Other Projects
- **Independent Library**: Unlike `protocols-mcp`, this is a standalone API client
- **No Direct Dependencies**: Doesn't depend on protocols-mcp initially
- **Future Integration**: Could be wrapped as MCP provider for unified interface
- **Shared Patterns**: Follows workspace patterns for consistency

### Workspace Position
- Part of Phase 3 (LLM APIs) in workspace roadmap
- Alongside future `apis-anthropic`, `apis-openai` clients
- Demonstrates workspace pattern for API client libraries

## Related Documentation

- **Workspace Brief**: `../../workspace/project-brief.md`
- **Shared Patterns**: `../../workspace/shared-patterns.md`
- **Workspace Architecture**: `../../workspace/workspace-architecture.md`
- **Product Context**: `product-context.md`
- **System Patterns**: `system-patterns.md`
- **Tech Context**: `tech-context.md`
- **Active Context**: `active-context.md`
- **Progress**: `progress.md`

---

This brief serves as the foundation document for the api-openrouter sub-project.
