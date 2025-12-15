# AirsProtocols Workspace - Project Brief

**Version**: 1.0  
**Last Updated**: 2025-12-15  
**Workspace Type**: Multi-Project Rust Monorepo

## Vision

Build production-ready, type-safe implementations of AI communication protocols and LLM provider APIs in Rust, enabling the next generation of intelligent applications with performance, safety, and interoperability at their core.

## Mission

Provide the Rust ecosystem with comprehensive, standards-compliant implementations of:
1. Agent communication protocols (MCP, A2A)
2. LLM provider client libraries (Anthropic, OpenAI, Google, Ollama)
3. Interoperability bridges between protocols
4. Production-ready server implementations

## Core Objectives

### 1. Protocol Foundation
- **Complete MCP Implementation** ✅
  - Full JSON-RPC 2.0 specification
  - Multiple transport layers (stdio, HTTP, WebSocket)
  - Authentication & authorization
  - Provider framework

- **A2A Protocol** 🚧 Planned
  - Agent-to-agent communication
  - Task lifecycle management
  - Multiple protocol bindings

### 2. LLM API Clients 🚧 Planned
- Unified interface across providers
- Streaming support
- Type-safe message structures
- Async-first design

### 3. Production Servers
- **Filesystem Server** ✅
  - Security-first design
  - Comprehensive path validation
  - Human-in-the-loop approval
  - Audit logging

### 4. Ecosystem Integration
- Interoperability with Python/TypeScript SDKs
- Standards compliance
- Community adoption
- Documentation excellence

## Target Audience

### Primary Users
1. **Rust Developers** building AI applications
2. **Enterprise Teams** requiring type-safe, performant AI integrations
3. **Open Source Contributors** to the AI agent ecosystem

### Secondary Users
1. AI researchers prototyping agents
2. Platform developers building AI infrastructure
3. Tool developers creating MCP servers

## Success Criteria

### Technical Excellence
- [ ] 100% MCP specification compliance
- [ ] <100ms average response time for protocol operations
- [ ] Comprehensive test coverage (>80%)
- [ ] Zero-cost abstractions where possible
- [ ] Production-grade error handling

### Ecosystem Impact
- [ ] Adoption by major Rust AI projects
- [ ] Compatible with official SDKs (Python, TypeScript)
- [ ] Active community contributions
- [ ] Referenced in official protocol documentation

### Developer Experience
- [ ] Clear, comprehensive documentation
- [ ] Working examples for all major use cases
- [ ] Quick start guides (<5 min to first working app)
- [ ] Active issue resolution (<48h response time)

## Architecture Philosophy

### Type Safety First
Leverage Rust's type system for compile-time verification of protocol compliance and message structures.

### Protocol Agnostic
Clean separation between protocol specification and transport implementation allows easy addition of new transports.

### Async Native
Built on `tokio` for non-blocking I/O and efficient concurrent operations throughout the stack.

### Production Ready
- Comprehensive error handling
- Security-first design
- Performance optimization
- Extensive testing (unit, integration, property-based)

### Standards Compliant
Strict adherence to official specifications with interoperability testing against reference implementations.

## Project Boundaries

### In Scope
- Protocol implementations (MCP, A2A)
- LLM provider clients
- Core MCP servers (filesystem, etc.)
- Transport layers and authentication
- Documentation and examples

### Out of Scope
- Application-level business logic
- UI/UX components
- Model training or inference
- Database adapters
- Frontend frameworks

## Constraints

### Technical
- **Rust Version**: 1.88.0+ (workspace standard)
- **Async Runtime**: Tokio (for consistency)
- **MSRV Policy**: Support last 2 stable Rust versions

### Organizational
- Dual licensing (MIT/Apache-2.0)
- Open source development
- Community-driven governance
- Semantic versioning

### Resource
- Initial focus on MCP before A2A
- Prioritize core functionality over optional features
- Balance between features and maintenance burden

## Roadmap Overview

### Phase 1: Foundation ✅
- [x] Workspace setup
- [x] MCP protocol complete
- [x] Documentation site
- [ ] CI/CD pipeline

### Phase 2: Core Protocols (Current)
- [x] MCP documentation complete
- [ ] A2A protocol implementation
- [ ] MCP ↔ A2A bridges

### Phase 3: LLM APIs
- [ ] Anthropic client
- [ ] OpenAI client
- [ ] Unified interface

### Phase 4: Ecosystem
- [ ] Additional servers
- [ ] Performance optimization
- [ ] Advanced features

## Related Documentation

- **Architecture**: `workspace-architecture.md`
- **Patterns**: `shared-patterns.md`
- **Progress**: `workspace-progress.md`
- **Rust Guidelines**: `../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md`

---

This brief serves as the north star for all development decisions across the AirsProtocols workspace.
