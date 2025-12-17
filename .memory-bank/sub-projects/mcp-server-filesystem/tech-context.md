# mcp-server-filesystem - Tech Context

**Last Updated**: 2025-12-17  
**Version**: 1.0.0-rc.1

## Technology Stack

### Core Dependencies

**airsprotocols-mcp**: Foundation
- **Why**: Provides MCP protocol implementation
- **Version**: Workspace dependency
- **Features**: Transport, providers, integration APIs

**Rust**: 1.88.0+
- **Why**: Security, performance, type safety
- **MSRV**: 1.88.0
- **Edition**: 2021

**tokio**: Async Runtime
- **Why**: Non-blocking file I/O
- **Features**: `fs`, `io-util`, `macros`

### Filesystem & Security

**path-clean**: ^1.0
- **Why**: Path normalization
- **Use**: Resolve `.` and `..` in paths

**globset**: ^0.4
- **Why**: Glob pattern matching
- **Use**: Path allowlist/denylist matching

**glob**: ^0.3
- **Why**: Pattern expansion
- **Use**: Directory scanning with patterns

**walkdir**: ^2.5
- **Why**: Recursive directory traversal
- **Use**: List directory contents

**infer**: ^0.16
- **Why**: File type detection
- **Use**: Binary file detection by magic numbers

**image**: ^0.25 (Optional)
- **Why**: Image format validation
- **Use**: Additional binary detection

**unicode-normalization**: ^0.1
- **Why**: Unicode path handling
- **Use**: Prevent Unicode-based attacks

**urlencoding**: ^2.1
- **Why**: URL decoding
- **Use**: Decode encoded paths in attacks

**dirs**: ^5.0
- **Why**: Platform-specific directories
- **Use**: Configuration and log directories

### Configuration

**config**: ^0.14
- **Why**: Multi-source configuration
- **Features**: TOML, environment variables, defaults

**toml**: ^0.8 (Optional feature)
- **Why**: TOML configuration format
- **Use**: Human-readable config files

### CLI

**clap**: ^4.5
- **Why**: Command-line parsing
- **Features**: Derive API for declarative CLI
- **Use**: Setup, config, run commands

### Utilities

**serde**: ^1.0
- **Why**: Serialization
- **Features**: `derive`

**serde_json**: ^1.0
- **Why**: JSON serialization
- **Use**: MCP message handling

**thiserror**: ^2.0
- **Why**: Error types (library pattern)

**anyhow**: ^1.0
- **Why**: Error handling (application pattern)

**tracing**: ^0.1
- **Why**: Structured logging

**tracing-subscriber**: ^0.3
- **Why**: Log formatting

**chrono**: ^0.4
- **Why**: Timestamps for audit logs

**uuid**: ^1.11
- **Why**: Session identifiers

**async-trait**: ^0.1
- **Why**: Async traits for providers

**base64**: ^0.22
- **Why**: Encoding utilities

**regex**: ^1.11
- **Why**: Pattern matching

### Development Dependencies

**tokio-test**: ^0.4
- **Why**: Async test utilities

**assert_fs**: ^1.1
- **Why**: Filesystem test utilities
- **Use**: Create temporary test structures

**tempfile**: ^3.14
- **Why**: Temporary files/dirs for tests

## Development Environment

### Build Commands

```bash
# Development build
cargo build -p airsprotocols-mcpserver-filesystem

# Release build
cargo build --release -p airsprotocols-mcpserver-filesystem

# Run tests
cargo test -p airsprotocols-mcpserver-filesystem

# Run server
cargo run -p airsprotocols-mcpserver-filesystem
```

### Configuration Paths

**macOS**:
- Config: `~/.config/airsprotocols-mcpserver-filesystem/`
- Logs: `~/.local/share/airsprotocols-mcpserver-filesystem/logs/`

**Linux**:
- Config: `~/.config/airsprotocols-mcpserver-filesystem/`
- Logs: `~/.local/share/airsprotocols-mcpserver-filesystem/logs/`

**Windows**:
- Config: `%APPDATA%\airsprotocols-mcpserver-filesystem\`
- Logs: `%LOCALAPPDATA%\airsprotocols-mcpserver-filesystem\logs\`

## Performance Characteristics

### File Operations
- **Read**: <50ms typical
- **Write**: <100ms typical
- **List**: <100ms for typical directories
- **Path validation**: <1ms per path

### Memory Usage
- **Baseline**: <50MB
- **Scaling**: + ~10MB per concurrent operation

## Security Constraints

### Path Security
- All paths canonicalized
- Directory traversal blocked
- Symlink resolution controlled
- Allowlist/denylist enforced

### Binary File Handling
- Extension-based detection
- Content-based detection (magic numbers)
- Null byte detection
- MIME type checking

### Approval Requirements
- Write operations require approval
- Delete operations require approval
- Move/copy operations require approval
- Read/list operations allowed (if path valid)

## Testing Infrastructure

### Test Categories

1. **Unit Tests**: Module-level functionality
2. **Integration Tests**: End-to-end workflows
3. **Security Tests**: Attack vector validation
4. **Configuration Tests**: Config loading and validation

### Running Tests

```bash
# All tests
cargo test -p airsprotocols-mcpserver-filesystem

# Security tests only
cargo test -p airsprotocols-mcpserver-filesystem security

# Integration tests
cargo test -p airsprotocols-mcpserver-filesystem --test integration
```

## Deployment

### Binary Installation

```bash
# From crates.io (when published)
cargo install airsprotocols-mcpserver-filesystem

# From source
cargo install --path mcp/servers/filesystem
```

### Claude Desktop Integration

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "/path/to/airsprotocols-mcpserver-filesystem",
      "env": {
        "AIRSPROTOCOLS_MCPSERVER_FS_ENV": "development",
        "AIRSPROTOCOLS_MCPSERVER_FS_CONFIG_DIR": "/path/to/config",
        "AIRSPROTOCOLS_MCPSERVER_FS_LOG_DIR": "/path/to/logs"
      }
    }
  }
}
```

## Related Documentation

- **Configuration Guide**: `../../../mcp/servers/filesystem/CONFIGURATION.md`
- **Security Audit**: `../../../mcp/servers/filesystem/SECURITY_AUDIT.md`
- **System Patterns**: `system-patterns.md`
- **Active Context**: `active-context.md`

---

This tech context provides the technical foundation for the filesystem server.
