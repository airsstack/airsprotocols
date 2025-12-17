# api-openrouter - Active Context

**Last Updated**: 2025-12-17  
**Current Sprint**: Planning & Architecture (Sprint 0)  
**Status**: 🚀 New Project - Planning Phase

## Current Focus

### This Week
- ✅ Memory Bank setup (project structure initialization)
- 🔄 OpenRouter API research and documentation review
- ⏳ Core type definitions design (Message, Model, Request, Response)
- ⏳ Project structure and crate layout planning

### This Sprint (Sprint 0 - Planning)
**Duration**: Week of 2025-12-17  
**Goal**: Complete planning and architectural design

**Objectives**:
1. ✅ Initialize Memory Bank structure
2. ⏳ Research OpenRouter API documentation thoroughly
3. ⏳ Design core type system (messages, models, requests, responses)
4. ⏳ Plan crate structure and module organization
5. ⏳ Create ADR for key architectural decisions
6. ⏳ Draft API surface design (public interface)

## Recent Activity

### 2025-12-17
- ✅ Created Memory Bank structure for api-openrouter sub-project
- ✅ Wrote project brief defining vision, scope, and objectives
- ✅ Documented product context with user personas and benefits
- ✅ Initialized docs/ structure (debts/, knowledges/, adr/)
- 📝 Status: Project officially started in planning phase

### Previous Work
- N/A (New project)

## Active Decisions

### Pending Decisions (Require Research)

1. **Type System Design**
   - **Question**: How to structure Message types (enum vs trait hierarchy)?
   - **Options**: 
     - A) Sealed enum with variants (System, User, Assistant, Function)
     - B) Trait-based with marker types
     - C) Struct-based with role field
   - **Next Steps**: Research OpenRouter message format, study async-openai approach
   - **Target Date**: 2025-12-18

2. **Streaming Implementation**
   - **Question**: Which SSE parsing library to use?
   - **Options**:
     - A) `eventsource-stream` (mature, well-maintained)
     - B) `reqwest-eventsource` (reqwest integration)
     - C) Custom SSE parser (full control)
   - **Next Steps**: Benchmark libraries, check OpenRouter SSE format
   - **Target Date**: 2025-12-19

3. **Error Handling Strategy**
   - **Question**: How granular should error types be?
   - **Options**:
     - A) Flat enum with all error types
     - B) Hierarchical errors (ApiError, NetworkError, etc.)
     - C) Canonical struct with ErrorKind (following workspace pattern)
   - **Next Steps**: Review workspace error patterns, define error categories
   - **Target Date**: 2025-12-18

4. **Cost Tracking Design**
   - **Question**: Where to extract cost information?
   - **Options**:
     - A) Parse from response headers (X-OpenRouter-Cost)
     - B) Calculate from token usage + model pricing
     - C) Both (with reconciliation)
   - **Next Steps**: Review OpenRouter response format, check header availability
   - **Target Date**: 2025-12-19

### Recently Resolved Decisions
- N/A (New project)

## Next Steps

### Immediate (Next 24-48 hours)
1. [ ] Read OpenRouter API documentation (https://openrouter.ai/docs)
2. [ ] Document API endpoints and request/response formats
3. [ ] Study OpenRouter authentication mechanism
4. [ ] Research SSE streaming format
5. [ ] Review rate limiting and error codes

### Short-Term (This Week)
1. [ ] Design core type structures (draft in ADR)
2. [ ] Create module structure plan
3. [ ] Write ADR-001: Type System Design
4. [ ] Write ADR-002: Streaming Implementation
5. [ ] Draft public API surface (trait signatures)

### Medium-Term (Next Sprint)
1. [ ] Initialize Rust crate with Cargo.toml
2. [ ] Implement core types (Message, Model, Request)
3. [ ] Create OpenRouterClient struct skeleton
4. [ ] Implement authentication mechanism
5. [ ] Add basic HTTP client with reqwest

## Open Questions

### Technical Questions
1. **Does OpenRouter support function calling?** (Need to verify in docs)
2. **What are the rate limiting rules?** (Per-API-key? Per-model?)
3. **How does provider fallback work?** (Automatic? Manual?)
4. **Are there model-specific parameters?** (Provider extensions?)
5. **What's the format of streaming chunks?** (SSE data structure?)

### Design Questions
1. **Should we support custom HTTP clients?** (Dependency injection for reqwest?)
2. **How to handle provider-specific features?** (Extensions trait?)
3. **Should cost tracking be optional?** (Feature flag?)
4. **Do we need connection pooling configuration?** (Exposed or automatic?)
5. **How to version API compatibility?** (OpenRouter API versioning?)

### Documentation Questions
1. **What examples are most valuable?** (Basic chat? Multi-model? Streaming?)
2. **Should we provide migration guide from OpenAI?** (API compatibility?)
3. **Do we need a CLI tool for testing?** (Like curl but typed?)

## Blockers & Risks

### Current Blockers
- ❌ None (Planning phase)

### Potential Risks

1. **Risk**: OpenRouter API changes frequently
   - **Impact**: Medium (requires maintenance)
   - **Mitigation**: Version API compatibility, monitor OpenRouter changelog
   - **Status**: Monitoring

2. **Risk**: Streaming SSE parsing complexity
   - **Impact**: Medium (affects user experience)
   - **Mitigation**: Research proven libraries, comprehensive testing
   - **Status**: Researching

3. **Risk**: Rate limiting edge cases
   - **Impact**: Medium (production reliability)
   - **Mitigation**: Implement robust retry logic, respect rate limits
   - **Status**: Planning

4. **Risk**: Cost tracking accuracy
   - **Impact**: Low (non-critical feature)
   - **Mitigation**: Parse headers when available, document limitations
   - **Status**: Researching

## Dependencies & Coordination

### Upstream Dependencies
- **OpenRouter API**: External service (https://openrouter.ai)
  - Status: Stable, publicly available
  - Documentation: https://openrouter.ai/docs
  - No direct coordination needed

### Internal Dependencies (Workspace)
- **Workspace Patterns**: Following shared patterns from `.memory-bank/workspace/`
  - Error handling patterns
  - Async patterns
  - Testing patterns
  - Documentation standards

### External Crate Dependencies (Planned)
- `tokio` - Async runtime (mature, stable)
- `reqwest` - HTTP client (mature, widely used)
- `serde`, `serde_json` - Serialization (standard)
- `thiserror` - Error handling (workspace standard)
- `eventsource-stream` - SSE parsing (researching)
- `tokio-stream` - Stream utilities (standard)

## Team Context

### Project Team
- **Role**: AI Agent (Memory Bank Planner)
- **User**: Developer providing requirements and guidance
- **Status**: Active planning phase

### Communication
- **Primary**: Memory Bank files (.memory-bank/sub-projects/api-openrouter/)
- **Documentation**: GitHub markdown in project structure
- **Decisions**: ADRs in docs/adr/

## Environment & Setup

### Development Environment
- **Rust Version**: 1.88.0+ (workspace requirement)
- **Toolchain**: Stable Rust, cargo
- **IDE**: Any Rust-compatible IDE (VS Code, RustRover, etc.)
- **Testing**: OpenRouter API key required (free tier available)

### Setup Status
- ✅ Memory Bank structure created
- ⏳ Cargo crate not yet initialized
- ⏳ Dependencies not yet specified
- ⏳ Development environment not yet configured

## Progress Indicators

### Overall Project Progress: 5%
- ✅ Planning: 30% (Memory Bank setup complete, research ongoing)
- ⏳ Design: 0% (Not started)
- ⏳ Implementation: 0% (Not started)
- ⏳ Testing: 0% (Not started)
- ⏳ Documentation: 0% (Not started)

### Sprint 0 Progress: 20%
- ✅ Memory Bank initialization (100%)
- 🔄 API research (20%)
- ⏳ Type system design (0%)
- ⏳ Module structure planning (0%)
- ⏳ ADR drafting (0%)

## Related Contexts

### Workspace Context
- **Active Project**: api-openrouter
- **Workspace Phase**: Phase 3 (LLM APIs)
- **Related Projects**: 
  - protocols-mcp (complete, foundation patterns)
  - mcp-server-filesystem (complete, reference implementation)

### Documentation Links
- **Project Brief**: `project-brief.md` - Vision and objectives
- **Product Context**: `product-context.md` - User needs and benefits
- **System Patterns**: `system-patterns.md` - Architecture and design patterns
- **Tech Context**: `tech-context.md` - Technical stack and constraints
- **Progress**: `progress.md` - Detailed progress tracking
- **Workspace Patterns**: `../../workspace/shared-patterns.md` - Shared guidelines

## Notes & Observations

### Planning Notes
- OpenRouter's OpenAI-compatible API should make design easier
- Can learn from `async-openai` implementation patterns
- Focus on ergonomics: builders, sensible defaults, clear errors
- Cost tracking is unique value-add over generic HTTP clients
- Streaming support is critical for good UX

### Research Notes
- OpenRouter homepage: https://openrouter.ai
- API docs: https://openrouter.ai/docs
- Authentication: Bearer token via `Authorization` header
- Base URL: `https://openrouter.ai/api/v1`
- Main endpoint: `/chat/completions` (OpenAI-compatible)

### Design Considerations
- Follow workspace patterns for consistency
- Async-first with tokio (non-negotiable)
- Type safety over convenience (but aim for both)
- Production-ready from v0.1.0 (no prototype release)
- Documentation as important as code

---

**Last Updated**: 2025-12-17  
**Next Review**: 2025-12-18 (after API research completion)
