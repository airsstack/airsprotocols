# Installation

Complete installation instructions for the AIRS MCP Filesystem Server.

## System Requirements

- **Rust**: Version 1.88.0 or later
- **Operating System**: macOS, Linux, or Windows
- **Disk Space**: ~50MB for binary and dependencies
- **Claude Desktop**: Latest version (optional, for Claude integration)

## Installation Methods

### Method 1: From crates.io (Recommended)

The simplest way to install:

```bash
cargo install airsprotocols-mcpserver-filesystem
```

The binary will be installed to `~/.cargo/bin/airsprotocols-mcpserver-filesystem`.

### Method 2: From Source

For the latest development version:

```bash
# Clone the repository
git clone https://github.com/airsstack/airsprotocols.git
cd airsprotocols

# Build and install
cargo install --path mcp/servers/filesystem
```

### Method 3: Pre-built Binaries

Download pre-built binaries from the [releases page](https://github.com/airsstack/airsprotocols/releases).

## Verification

Verify the installation:

```bash
airsprotocols-mcpserver-filesystem --version
```

Expected output:
```
airsprotocols-mcpserver-filesystem 1.0.0-rc.1
```

## Post-Installation Setup

### 1. Create Directory Structure

```bash
airsprotocols-mcpserver-filesystem setup
```

This creates:
- Configuration directory: `~/.config/airsprotocols-mcpserver-filesystem/`
- Log directory: `~/.local/share/airsprotocols-mcpserver-filesystem/logs/`

### 2. Generate Initial Configuration

```bash
airsprotocols-mcpserver-filesystem config --env development
```

This generates `~/.config/airsprotocols-mcpserver-filesystem/development.toml` with sensible defaults.

## Integration with Claude Desktop

See the [Quick Start Guide](./quick-start.md) for Claude Desktop integration instructions.

## Updating

To update to the latest version:

```bash
cargo install airsprotocols-mcpserver-filesystem --force
```

## Uninstallation

To remove the server:

```bash
cargo uninstall airsprotocols-mcpserver-filesystem

# Optionally remove configuration and logs
rm -rf ~/.config/airsprotocols-mcpserver-filesystem
rm -rf ~/.local/share/airsprotocols-mcpserver-filesystem
```

## Troubleshooting

### "cargo: command not found"

Install Rust:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.sh | sh
```

### "failed to compile"

Ensure you have the latest Rust version:
```bash
rustup update stable
```

### Binary not in PATH

Add cargo bin directory to PATH:
```bash
# Add to ~/.bashrc or ~/.zshrc
export PATH="$HOME/.cargo/bin:$PATH"
```

## Next Steps

- [Quick Start Guide](./quick-start.md): Get running in 5 minutes
- [Configuration](./configuration/index.md): Customize your setup
- [Usage Examples](./usage/index.md): Learn common patterns
