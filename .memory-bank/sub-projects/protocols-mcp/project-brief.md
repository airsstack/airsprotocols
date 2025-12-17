# protocols-mcp - Project Brief

**Version**: 1.0.0-rc.1  
**Last Updated**: 2025-12-17  
**Status**: Complete - Active Maintenance  
**Type**: Library Crate

## Vision

Provide the Rust ecosystem with a production-ready, type-safe, and performant implementation of the Model Context Protocol (MCP), enabling developers to build secure and reliable AI agent communication systems.

## Mission

Deliver a comprehensive MCP library that:
1. Fully implements the MCP specification (JSON-RPC 2.0 foundation)
2. Provides transport-agnostic protocol implementation
3. Offers pluggable authentication and authorization
4. Enables rapid development of MCP servers and clients
5. Maintains excellent documentation and examples

## Project Overview

`airsprotocols-mcp` is the foundational protocol implementation for the AirsProtocols workspace. It provides:

- **JSON-RPC 2.0 Core**: Complete message handling with spec compliance
- **Transport Abstraction**: Stdio, HTTP, SSE implementations with trait-based extensibility
- **Provider Framework**: ToolProvider, ResourceProvider, PromptProvider traits
- **Security Layer**: API Key and OAuth2 authentication strategies
- **Integration APIs**: High-level McpServer and McpClient builders

## Core Objectives

### 1. Protocol Compliance ✅ COMPLETE
- [x] JSON-RPC 2.0 message structures
- [x] MCP protocol types and specifications
- [x] Error handling and validation
- [x] Protocol-level testing

### 2. Transport Layer ✅ COMPLETE
- [x] Transport trait abstraction
- [x] Stdio transport (subprocess communication)
- [x] HTTP transport (network communication)
- [x] SSE support for streaming
- [ ] WebSocket transport (planned Q1 2025)

### 3. Authentication & Authorization ✅ COMPLETE
- [x] Strategy pattern for auth mechanisms
- [x] API Key authentication
- [x] OAuth2 authentication with JWT validation
- [x] Authorization middleware with policy support
- [x] Token lifecycle management

### 4. Provider Framework ✅ COMPLETE
- [x] ToolProvider trait and implementations
- [x] ResourceProvider trait and implementations
- [x] PromptProvider trait and implementations
- [x] Capability composition patterns
- [x] Provider testing utilities

### 5. Integration APIs ✅ COMPLETE
- [x] McpServer builder with configuration
- [x] McpClient builder with configuration
- [x] Example implementations
- [x] Integration testing

### 6. Documentation ✅ COMPLETE
- [x] mdBook comprehensive guide
- [x] API documentation (rustdoc)
- [x] Example projects
- [x] Integration tutorials

## Target Audience

### Primary Users
1. **Rust Developers** building MCP servers
2. **Enterprise Teams** requiring type-safe AI integrations
3. **Open Source Contributors** to MCP ecosystem

### Secondary Users
1. AI researchers prototyping agents
2. Tool developers creating MCP servers
3. Platform developers building AI infrastructure

## Success Criteria

### Technical Excellence
- [x] 100% MCP specification compliance
- [x] <1ms message processing latency
- [x] >80% test coverage (currently 82%)
- [x] Zero unsafe code in public APIs
- [x] Production-grade error handling

### Developer Experience
- [x] Clear API documentation
- [x] Working examples for all use cases
- [x] Quick start guide (<5 min to first app)
- [x] Comprehensive integration guides

### Ecosystem Impact
- [ ] Published to crates.io (target: Q1 2025)
- [ ] Referenced in official MCP documentation
- [ ] Active community contributions
- [ ] Adoption by Rust AI projects

## Project Boundaries

### In Scope
- MCP protocol implementation (messages, types, errors)
- Transport abstractions and implementations
- Authentication and authorization frameworks
- Provider trait definitions and utilities
- Server and client integration APIs
- Documentation and examples

### Out of Scope
- Specific application logic (belongs in servers)
- UI/UX components
- Database adapters
- Frontend frameworks
- Model inference or training

## Technical Constraints

### Technology Stack
- **Language**: Rust 1.88.0+
- **Async Runtime**: Tokio
- **Serialization**: serde + serde_json
- **HTTP**: axum + hyper
- **Auth**: oauth2 + jsonwebtoken

### Design Constraints
- Must remain transport-agnostic
- Zero-cost abstractions where possible
- Async-first APIs throughout
- Strong type safety (no stringly-typed APIs)
- Minimal external dependencies

### Performance Constraints
- Message processing: <1ms average
- Transport overhead: <100μs for stdio, <5ms for HTTP
- Memory footprint: <10MB baseline
- Concurrent requests: 100+ simultaneous

## Architecture Philosophy

### Type Safety First
Leverage Rust's type system for compile-time verification of protocol compliance and message structures. No stringly-typed APIs.

### Protocol Agnostic
Clean separation between protocol specification and transport implementation. Easy to add new transports without changing protocol layer.

### Async Native
Built on tokio for non-blocking I/O and efficient concurrent operations. All public futures are Send.

### Production Ready
- Comprehensive error handling with thiserror
- Security-first design with threat modeling
- Performance optimization for hot paths
- Extensive testing (unit, integration, property-based)

### Standards Compliant
Strict adherence to MCP and JSON-RPC 2.0 specifications with interoperability testing against reference implementations.

## Current Status

### Version: 1.0.0-rc.1
**Status**: Release Candidate - Feature Complete, Active Maintenance

### Recent Achievements (v0.2.x → v1.0.0-rc.1)
- ✅ Complete protocol implementation
- ✅ Multiple transport implementations
- ✅ Authentication/authorization framework
- ✅ Provider framework
- ✅ Integration APIs
- ✅ Comprehensive documentation
- ✅ Example implementations

### Known Issues
- None critical
- WebSocket transport planned for Q1 2025
- Rate limiting middleware planned for Q1 2025

## Dependencies

### Runtime Dependencies
- `tokio` - Async runtime
- `serde`, `serde_json` - Serialization
- `axum`, `hyper` - HTTP transport
- `oauth2`, `jsonwebtoken` - Authentication
- `thiserror` - Error handling
- `async-trait` - Async trait support

### Build Dependencies
- Rust 1.88.0+ toolchain
- cargo for building and testing

### Dependents
- `airsprotocols-mcpserver-filesystem` - Filesystem MCP server
- Future MCP servers in the workspace
- External projects using MCP in Rust

## Roadmap

### Q1 2025
- [ ] Publish to crates.io
- [ ] WebSocket transport implementation
- [ ] Rate limiting middleware
- [ ] Performance benchmarking suite
- [ ] Expanded integration examples

### Q2 2025
- [ ] gRPC transport (optional)
- [ ] Advanced authorization policies
- [ ] Monitoring and metrics integration
- [ ] Load testing and optimization

### Q3 2025
- [ ] MCP v2.0 support (if specification evolves)
- [ ] Additional authentication strategies
- [ ] Enhanced developer tooling

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

This brief serves as the foundation document for the protocols-mcp sub-project.
