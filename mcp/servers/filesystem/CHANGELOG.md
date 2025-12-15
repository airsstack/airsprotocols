# Changelog

All notable changes to airsprotocols-mcpserver-fs will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned
- Performance optimizations and additional security features

## [1.0.0-rc.1] - 2025-12-15

### 🎉 Major Migration Release

This release represents the migration of `airs-mcpserver-fs` to the `airsprotocols` organization with a complete package rename and modernization.

#### Package Migration
- **New package name:** `airsprotocols-mcpserver-fs` (previously `airs-mcpserver-fs`)
- **New repository:** https://github.com/airsstack/airsprotocols/tree/main/mcp/servers/filesystem
- **New binary name:** `airsprotocols-mcpserver-fs` (previously `airs-mcpserver-fs`)
- **Updated dependency:** Now uses `airsprotocols-mcp` v1.0.0-rc.1 (previously `airs-mcp` v0.2.3)

#### Breaking Changes
- Package name changed from `airs-mcpserver-fs` to `airsprotocols-mcpserver-fs`
- Binary name changed from `airs-mcpserver-fs` to `airsprotocols-mcpserver-fs`
- Environment variable prefix changed from `AIRS_MCPSERVER_FS_*` to `AIRSPROTOCOLS_MCPSERVER_FS_*`
- Crate import path changed from `airs_mcpserver_fs::` to `airsprotocols_mcpserver_fs::`
- Repository moved from `airsstack/airsstack` to `airsstack/airsprotocols`
- Default configuration directory changed from `~/.airs-mcpserver-fs` to `~/.airsprotocols-mcpserver-fs`

#### Migration Guide
For users migrating from `airs-mcpserver-fs` v0.1.x, see [MIGRATION.md](./MIGRATION.md)

#### Features (Carried Over from v0.1.2)
All features from `airs-mcpserver-fs` v0.1.2 are included and fully functional:
- Security-first filesystem operations
- Human-in-the-loop approval workflows
- Claude Desktop integration
- Comprehensive audit logging
- Path traversal protection
- Binary file detection
- Configurable security policies
- Multi-environment configuration
- Sub-100ms response times

#### Technical Details
- Updated all imports from `airs_mcp::` to `airsprotocols_mcp::`
- Updated all internal crate references
- Migrated to airsprotocols workspace dependencies
- All tests passing (25/25)
- Zero regression in functionality or performance

---

## Previous Versions (as airs-mcpserver-fs)

## [0.1.2] - 2025-09-24

### Changed
- **Dependency update**: Upgraded airs-mcp from 0.2.2 to 0.2.3
- **Enhanced security management**: Now uses airs-mcp v0.2.3 with comprehensive security audit configuration
- **Latest RSA version**: Updated to RSA 0.10.0-rc.8 (latest available) in development dependencies

### Security
- **Comprehensive security audit**: All known vulnerabilities properly documented and isolated to non-production code
- **Zero production vulnerabilities**: Clean security posture for all production code paths
- **Risk management**: Structured approach to security dependency management

## [0.1.1] - 2025-09-24

### Changed
- **Dependency update**: Upgraded airs-mcp from 0.2.1 to 0.2.2
- **Security enhancement**: Now uses airs-mcp v0.2.2 with RSA vulnerability fix (RUSTSEC-2023-0071 resolved)

### Security
- **Improved security posture**: Indirect resolution of RSA Marvin Attack vulnerability through upstream dependency update

## [0.1.0] - 2025-09-24

### Added
- **Initial release** of airsprotocols-mcpserver-fs as dedicated MCP filesystem server
- **Architectural migration** from crates/airs-mcp-fs to mcp-servers/airsprotocols-mcpserver-fs
- **Complete filesystem operations** through MCP protocol (read, write, list, create, delete, move, copy)
- **Security-first design** with 5-layer security validation system
- **Binary file restriction** with comprehensive content and extension-based detection
- **Human-in-the-loop approval** workflows for write/delete operations
- **Multi-environment configuration** system (development, staging, production)
- **Claude Desktop integration** with step-by-step setup guide
- **Comprehensive audit logging** with correlation IDs and security event tracking
- **Path validation and sanitization** with unicode normalization and traversal prevention
- **Performance optimization** with sub-100ms response times

### Security
- **Zero production vulnerabilities** - All security audits passed
- **5-layer security framework** - Path validation, permissions, policies, approval, audit
- **Binary file blocking** - Prevents entire classes of binary-based attacks
- **Memory safety** - Rust-based implementation with comprehensive error handling
- **Audit compliance** - Complete operation logging with security event tracking

### Technical Implementation
- **188 comprehensive tests** (146 unit + 17 integration + 25 doc tests)
- **Zero compilation warnings** - Professional code quality maintained
- **Workspace integration** - Clean dependency management with airs-mcp v0.2.1
- **Modular architecture** - Separation of concerns with clean interfaces
- **Cross-platform support** - Windows, macOS, and Linux compatibility

### Dependencies
- **airs-mcp v0.2.1** - Foundation MCP protocol implementation
- **Security dependencies** - Modern cryptography and validation libraries
- **Testing framework** - Comprehensive test coverage with multiple test types
