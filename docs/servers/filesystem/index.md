# MCP Filesystem Server

Security-first filesystem bridge that enables Claude Desktop and other MCP-compatible AI tools to safely read, write, and manage files in local development environments.

## Overview

`airsprotocols-mcpserver-filesystem` transforms AI assistance from passive consultation to active collaboration by providing secure, standardized filesystem operations through the Model Context Protocol (MCP). AI agents can now both understand your project context and create tangible artifacts directly in your local environment.

## Key Features

- **🔐 Security-First Design**: Human-in-the-loop approval workflows with configurable security policies
- **📁 Complete Filesystem Operations**: Read, write, create, delete, move, and copy files and directories
- **🛡️ Binary File Restriction**: Text-only processing with comprehensive binary file blocking
- **⚡ Performance Optimized**: Sub-100ms response times with efficient memory management
- **🔧 AIRS Ecosystem Integration**: Seamless compatibility with other AIRS MCP tools
- **🔒 Security Implementation**: Path validation, audit logging, threat detection, and binary file protection

## Quick Links

- **[Quick Start](./quick-start.md)**: Get up and running in 5 minutes
- **[Installation](./installation.md)**: Detailed installation instructions
- **[Configuration](./configuration/index.md)**: Complete configuration guide
- **[Security](./security/index.md)**: Security model and best practices
- **[Usage Examples](./usage/index.md)**: Common use cases and examples

## Core Capabilities

### File Operations
- **read_file**: Read text files with automatic encoding detection
- **write_file**: Create or update files with approval workflows
- **list_directory**: Browse filesystem with metadata and filtering
- **create_directory**: Create directory structures recursively
- **delete_file/delete_directory**: Safe deletion with confirmation
- **move_file/copy_file**: File manipulation with atomic operations

### Security Features
- **Binary File Restriction**: Comprehensive blocking of binary files
- **Human Approval**: Interactive confirmation for write/delete operations
- **Path Validation**: Prevent directory traversal and unauthorized access
- **Access Control**: Configurable allowlists and denylists
- **Audit Logging**: Comprehensive operation tracking
- **Threat Detection**: Enhanced security monitoring

## Installation

### From crates.io (Recommended)
```bash
cargo install airsprotocols-mcpserver-filesystem
```

### From Source
```bash
git clone https://github.com/airsstack/airsprotocols.git
cd airsprotocols
cargo build --release --package airsprotocols-mcpserver-filesystem
```

## Quick Start

1. **Install the server**:
   ```bash
   cargo install airsprotocols-mcpserver-filesystem
   ```

2. **Generate configuration**:
   ```bash
   airsprotocols-mcpserver-filesystem config --env development
   ```

3. **Configure Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):
   ```json
   {
     "mcpServers": {
       "airsprotocols-mcpserver-filesystem": {
         "command": "airsprotocols-mcpserver-filesystem",
         "env": {
           "AIRSPROTOCOLS_MCPSERVER_FS_ENV": "development"
         }
       }
     }
   }
   ```

4. **Restart Claude Desktop** and start using filesystem operations!

## Documentation

- **[Configuration Guide](./configuration/index.md)**: Complete configuration system
- **[Security Policies](./security/policies.md)**: Advanced security configuration
- **[Troubleshooting](./configuration/troubleshooting.md)**: Common issues and solutions

## Use Cases

### Development Workflow
- Code analysis and refactoring
- Documentation generation
- Project scaffolding
- Configuration management

### Content Management
- File organization
- Backup creation
- Text processing
- Duplicate detection

### Research & Analysis
- Code metadata extraction
- Content analysis
- Pattern matching
- Data processing

## Support

- **Documentation**: [Complete guides and references](./quick-start.md)
- **Issues**: [GitHub Issues](https://github.com/airsstack/airsprotocols/issues)
- **Discussions**: [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)

## License

Licensed under MIT OR Apache-2.0 at your option.
