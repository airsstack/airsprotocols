# api-openrouter - Product Context

**Last Updated**: 2025-12-17  
**Status**: Planning Phase

## Problem Statement

### The Challenge

Developers building AI applications in Rust face several challenges when working with Large Language Models (LLMs):

1. **Provider Lock-In**: Committing to a single LLM provider (OpenAI, Anthropic, etc.) creates vendor dependency and limits flexibility.

2. **Cost Management**: LLM costs vary significantly across providers and models. Developers lack tools to optimize spending or implement cost-based routing.

3. **Redundancy & Reliability**: Production systems need fallback options when a provider experiences downtime or rate limiting.

4. **Multi-Model Experimentation**: Comparing models across providers requires integrating multiple SDKs with different APIs.

5. **Limited Rust Support**: Most LLM providers offer official SDKs for Python/JavaScript, but Rust support is limited or non-existent.

6. **Type Safety**: Existing solutions often use stringly-typed APIs prone to runtime errors.

### Why OpenRouter?

**OpenRouter** is an LLM API aggregator that solves these challenges by:
- Providing a single API endpoint for 100+ models across 20+ providers
- Offering competitive pricing with automatic cost tracking
- Enabling model routing and fallback strategies
- Maintaining OpenAI-compatible API surface for easy migration
- Supporting streaming responses via Server-Sent Events

### The Gap

Currently, there is **no production-ready Rust client** for OpenRouter. Rust developers must either:
- Write their own client (duplicating effort)
- Use unsafe or stringly-typed HTTP calls
- Forgo the benefits of OpenRouter entirely

## Solution Overview

### What We're Building

`airsprotocols-api-openrouter` is a production-ready Rust client library that provides:

1. **Type-Safe API**: Strongly-typed requests, responses, and models leveraging Rust's type system
2. **Ergonomic Interface**: Builder patterns and async/await for intuitive usage
3. **Streaming Support**: Efficient SSE-based streaming with tokio-stream
4. **Cost Tracking**: Built-in usage and cost monitoring per request
5. **Production Features**: Retry logic, rate limiting, connection pooling, comprehensive error handling
6. **OpenRouter-Specific**: Full support for multi-provider routing, fallbacks, and provider preferences

### How It Works

```rust
// Example: Basic chat completion
use airsprotocols_api_openrouter::{OpenRouterClient, Message};

#[tokio::main]
async fn main() -> Result<()> {
    let client = OpenRouterClient::builder()
        .api_key(env::var("OPENROUTER_API_KEY")?)
        .build()?;
    
    let response = client
        .chat()
        .model("openai/gpt-4")
        .messages(vec![
            Message::user("What is the capital of France?"),
        ])
        .send()
        .await?;
    
    println!("Response: {}", response.content());
    println!("Cost: ${:.6}", response.usage().total_cost());
    
    Ok(())
}
```

```rust
// Example: Streaming chat
let mut stream = client
    .chat()
    .model("anthropic/claude-3-5-sonnet")
    .stream()
    .messages(vec![
        Message::user("Write a short story"),
    ])
    .send_stream()
    .await?;

while let Some(chunk) = stream.next().await {
    let chunk = chunk?;
    if let Some(content) = chunk.content() {
        print!("{}", content);
    }
}
```

### Key Differentiators

1. **Type Safety**: Compile-time validation of API usage (no stringly-typed endpoints)
2. **Rust-Native**: Built for Rust developers using idiomatic patterns
3. **Production-Ready**: Enterprise-grade error handling, retries, observability
4. **OpenRouter-First**: Designed specifically for OpenRouter's features (routing, costs, multi-provider)
5. **Zero-Overhead Abstractions**: Performance-conscious design with minimal overhead

## User Personas

### Primary Persona 1: Full-Stack Rust Developer

**Name**: Sarah, Senior Backend Engineer  
**Context**: Building a Rust-based SaaS product with AI features  
**Goals**:
- Integrate LLM capabilities into existing Rust services
- Keep costs under control with usage tracking
- Avoid vendor lock-in to single LLM provider
- Type-safe APIs to catch errors at compile-time

**Pain Points**:
- Python/JS SDKs don't integrate with Rust codebase
- Managing multiple provider SDKs is complex
- Runtime errors from stringly-typed HTTP clients
- Difficult to track and optimize LLM costs

**How We Help**:
- Native Rust library with async/await
- Single client for all providers
- Type-safe API surface
- Built-in cost tracking and monitoring

### Primary Persona 2: AI Researcher/Engineer

**Name**: David, Machine Learning Engineer  
**Context**: Experimenting with different LLM models for research  
**Goals**:
- Compare performance across models and providers
- Quick prototyping with multiple LLMs
- Cost-effective experimentation
- Stream responses for interactive testing

**Pain Points**:
- Setting up multiple provider SDKs is time-consuming
- Comparing models requires different code paths
- Costs add up during experimentation
- Python ecosystem but prefers Rust for production

**How We Help**:
- Single API for 100+ models
- Easy model switching (just change model string)
- Cost tracking to monitor spending
- Streaming support for real-time feedback

### Primary Persona 3: Enterprise Platform Team

**Name**: Marcus, Platform Engineer  
**Context**: Building internal AI platform for company  
**Goals**:
- Reliable LLM access with fallbacks
- Multi-tenant cost allocation
- Compliance and audit logging
- High availability and performance

**Pain Points**:
- Single provider creates reliability risk
- Tracking costs per tenant/department
- Need detailed audit logs
- Python services in Rust infrastructure

**How We Help**:
- Multi-provider redundancy via OpenRouter
- Per-request cost tracking
- Integration with tracing for audit logs
- Rust-native fits infrastructure stack

### Secondary Persona: Indie Developer

**Name**: Alex, Independent Developer  
**Context**: Building AI-powered side project/startup  
**Goals**:
- Get started quickly with LLMs
- Minimize costs on limited budget
- Access to best models without commitments
- Simple, clear documentation

**Pain Points**:
- Can't afford OpenAI/Anthropic enterprise pricing
- Scared of surprise API bills
- Limited time for complex integrations
- Wants flexibility to switch providers

**How We Help**:
- OpenRouter's competitive pricing
- Cost tracking prevents surprises
- Simple, ergonomic API
- Quick start guide (<10 minutes)

## Key Benefits

### For Developers

1. **Rapid Development**: Builder patterns and sensible defaults enable quick integration
2. **Type Safety**: Catch API misuse at compile-time, not in production
3. **Ergonomic API**: Intuitive, Rust-idiomatic interface reduces cognitive load
4. **Excellent Documentation**: Examples, guides, and API reference for all use cases

### For Teams

1. **Cost Optimization**: Built-in tracking enables data-driven decisions on model selection
2. **Flexibility**: Easy to switch models/providers without code changes
3. **Reliability**: Retry logic and fallback support improve uptime
4. **Maintainability**: Type-safe code reduces bugs and improves refactoring confidence

### For Businesses

1. **Reduced Vendor Risk**: Multi-provider access prevents lock-in
2. **Cost Control**: Visibility into LLM spending per feature/request
3. **Faster Time-to-Market**: Production-ready client accelerates development
4. **Compliance**: Structured logging and audit trails for governance

## Market Context

### LLM API Landscape

**Direct Provider APIs**:
- OpenAI, Anthropic, Google, Meta, etc.
- Pros: Official support, latest features
- Cons: Vendor lock-in, inconsistent APIs, higher costs

**API Aggregators**:
- **OpenRouter**: Multi-provider, competitive pricing, unified API
- **Anyscale**: Focused on open-source models
- **Replicate**: Infrastructure for running models

**Why OpenRouter**:
- Largest model selection (100+ models)
- OpenAI-compatible API (easy migration)
- Cost optimization features
- Active community and updates

### Rust Ecosystem

**Existing Rust LLM Clients**:
- `async-openai`: OpenAI-specific client (unofficial)
- `anthropic-sdk`: Community-driven Anthropic client
- Various incomplete or unmaintained clients

**Gap**: No production-ready OpenRouter client exists in Rust ecosystem.

**Opportunity**: First-mover advantage to become the de facto OpenRouter Rust client.

## Success Metrics

### Adoption Metrics
- [ ] 1,000+ downloads on crates.io in first month
- [ ] 10+ projects using in production within 6 months
- [ ] 5+ community contributors

### Usage Metrics
- [ ] 50+ GitHub stars in first quarter
- [ ] Active community engagement (issues, discussions)
- [ ] Integration examples shared by users

### Quality Metrics
- [ ] >80% test coverage maintained
- [ ] <5 critical bugs reported in first release
- [ ] Response time to issues <48 hours
- [ ] Documentation rated helpful by users

### Ecosystem Impact
- [ ] Referenced in OpenRouter documentation
- [ ] Featured in Rust LLM ecosystem lists
- [ ] Integration guides by community
- [ ] Talks/blog posts about the library

## Competitive Analysis

### vs. Direct Provider Clients (async-openai, etc.)

**Advantages**:
- ✅ Multi-provider access (100+ models vs 1 provider)
- ✅ Cost optimization via OpenRouter
- ✅ Provider redundancy and fallbacks
- ✅ Competitive pricing

**Tradeoffs**:
- ⚠️ Indirect access (via OpenRouter)
- ⚠️ May lag behind provider feature releases
- ⚠️ Additional network hop (OpenRouter routing)

### vs. Python/JS OpenRouter Clients

**Advantages**:
- ✅ Type safety at compile-time
- ✅ Rust performance and memory safety
- ✅ Native async/await with tokio
- ✅ Zero-cost abstractions

**Tradeoffs**:
- ⚠️ Smaller ecosystem than Python
- ⚠️ More verbose than dynamic languages
- ⚠️ Learning curve for Rust newcomers

### vs. Raw HTTP Calls

**Advantages**:
- ✅ Type-safe request/response structures
- ✅ Automatic retry and error handling
- ✅ Connection pooling and performance optimization
- ✅ Streaming abstraction
- ✅ Cost tracking built-in

**Tradeoffs**:
- ⚠️ Additional dependency
- ⚠️ Abstraction layer (minimal overhead)

## Feature Prioritization

### Must-Have (Phase 1)
- ✅ Chat completions (non-streaming)
- ✅ API key authentication
- ✅ Type-safe message structures
- ✅ Basic error handling
- ✅ Core documentation

### Should-Have (Phase 2)
- ✅ Streaming responses
- ✅ Retry logic with backoff
- ✅ Rate limit handling
- ✅ Connection pooling
- ✅ Example projects

### Nice-to-Have (Phase 3+)
- ⏳ Model metadata API
- ⏳ Advanced routing strategies
- ⏳ Custom provider parameters
- ⏳ MCP provider integration
- ⏳ CLI tool for testing

### Future Consideration
- 🔮 Function calling support
- 🔮 Image generation endpoints
- 🔮 Audio/video models
- 🔮 Fine-tuning API (if OpenRouter adds)

## User Journey

### Getting Started
1. **Discovery**: User finds library via crates.io or Rust LLM ecosystem
2. **Evaluation**: Reads quick start guide and examples
3. **Setup**: Adds dependency, configures API key
4. **First Request**: Sends chat completion in <10 minutes
5. **Success**: Gets response, sees cost tracking

### Integration
1. **Exploration**: Tries different models and providers
2. **Customization**: Configures retry, timeout, logging
3. **Streaming**: Implements streaming responses
4. **Testing**: Writes tests with mock responses
5. **Production**: Deploys with monitoring and alerts

### Optimization
1. **Monitoring**: Reviews cost and latency metrics
2. **Tuning**: Adjusts model selection based on cost/quality
3. **Scaling**: Leverages connection pooling for high throughput
4. **Reliability**: Implements fallback strategies

## Related Documentation

- **Project Brief**: `project-brief.md`
- **System Patterns**: `system-patterns.md`
- **Tech Context**: `tech-context.md`
- **Active Context**: `active-context.md`
- **Progress**: `progress.md`

---

This product context guides feature development and prioritization for the api-openrouter project.
