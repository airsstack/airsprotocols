# Migration Guide: airs-mcpserver-fs v0.1.x → airsprotocols-mcpserver-filesystem v1.0.0-rc.1

This guide helps you migrate from the legacy `airs-mcpserver-fs` package (v0.1.2 and earlier) to the new `airsprotocols-mcpserver-filesystem` v1.0.0-rc.1.

## Overview

The AIRS MCP Filesystem server has been migrated from the `airsstack/airsstack` repository to the `airsstack/airsprotocols` organization. This migration provides:

- **Modern Organization**: Part of the airsprotocols ecosystem
- **Updated Dependencies**: Uses `airsprotocols-mcp` v1.0.0-rc.1
- **Zero Functional Changes**: All functionality preserved and tested
- **Cleaner Architecture**: Better separation and organization

## Quick Migration (5 minutes)

### 1. Uninstall Old Package (if installed via cargo)

```bash
cargo uninstall airs-mcpserver-fs
```

### 2. Install New Package

```bash
# From source (recommended during RC phase)
cd /path/to/airsprotocols
cargo build --release -p airsprotocols-mcpserver-filesystem

# Or from crates.io (when published)
# cargo install airsprotocols-mcpserver-filesystem
```

### 3. Update Claude Desktop Configuration

**Location:** `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)

**Before (v0.1.2):**
```json
{
  "mcpServers": {
    "airs-mcpserver-fs": {
      "command": "airs-mcpserver-fs",
      "env": {
        "AIRS_MCPSERVER_FS_ENV": "development",
        "AIRS_MCPSERVER_FS_CONFIG_DIR": "/Users/yourusername/.config/airs-mcpserver-fs",
        "AIRS_MCPSERVER_FS_LOG_DIR": "/Users/yourusername/.local/share/airs-mcpserver-fs/logs"
      }
    }
  }
}
```

**After (v1.0.0-rc.1):**
```json
{
  "mcpServers": {
    "airsprotocols-mcpserver-filesystem": {
      "command": "airsprotocols-mcpserver-filesystem",
      "env": {
        "AIRSPROTOCOLS_MCPSERVER_FS_ENV": "development",
        "AIRSPROTOCOLS_MCPSERVER_FS_CONFIG_DIR": "/Users/yourusername/.config/airsprotocols-mcpserver-filesystem",
        "AIRSPROTOCOLS_MCPSERVER_FS_LOG_DIR": "/Users/yourusername/.local/share/airsprotocols-mcpserver-filesystem/logs"
      }
    }
  }
}
```

### 4. Migrate Configuration Directory (Optional but Recommended)

```bash
# Move configuration to new location
mv ~/.config/airs-mcpserver-fs ~/.config/airsprotocols-mcpserver-filesystem

# Move logs to new location
mv ~/.local/share/airs-mcpserver-fs/logs ~/.local/share/airsprotocols-mcpserver-filesystem/logs
```

### 5. Restart Claude Desktop

Restart Claude Desktop to load the new configuration.

## Detailed Migration Steps

### Environment Variables Migration

All environment variables have been updated with the new prefix:

| Old Variable (v0.1.x) | New Variable (v1.0.0-rc.1) |
|----------------------|----------------------------|
| `AIRS_MCPSERVER_FS_ENV` | `AIRSPROTOCOLS_MCPSERVER_FS_ENV` |
| `AIRS_MCPSERVER_FS_CONFIG_DIR` | `AIRSPROTOCOLS_MCPSERVER_FS_CONFIG_DIR` |
| `AIRS_MCPSERVER_FS_LOG_DIR` | `AIRSPROTOCOLS_MCPSERVER_FS_LOG_DIR` |
| `AIRS_MCP_FS_SECURITY_*` | `AIRSPROTOCOLS_MCPSERVER_FS_SECURITY_*` |

### Configuration File Migration

#### Option 1: Update Configuration Files
```bash
# Update your existing config files to use new binary name
sed -i 's/airsprotocols-mcpserver-filesystem/airsprotocols-mcpserver-filesystem/g' ~/.config/airsprotocols-mcpserver-filesystem/*.toml
```

#### Option 2: Use Legacy Configuration
Your existing configuration files will continue to work without changes. The new binary maintains full backward compatibility.

### Shell Scripts and Automation

Update any shell scripts or automation that references the old binary:

```bash
# Old
./target/release/airsprotocols-mcpserver-filesystem --config development.toml

# New  
./target/release/airsprotocols-mcpserver-filesystem --config development.toml
```

## Validation Steps

After migration, verify everything works correctly:

### 1. Build Test
```bash
cargo build --release --bin airsprotocols-mcpserver-filesystem
echo "Build status: $?"  # Should be 0
```

### 2. Configuration Test
```bash
./target/release/airsprotocols-mcpserver-filesystem --help
# Should show help without errors
```

### 3. Claude Desktop Integration Test
1. Restart Claude Desktop
2. Test basic commands:
   - "List files in my current directory"
   - "Read the contents of README.md"
   - "Create a test file"

### 4. Performance Validation
- Response times should remain sub-100ms
- Memory usage should be unchanged
- All security features should work identically

## Rollback Plan

If you encounter issues, you can easily rollback:

### 1. Restore Claude Desktop Configuration
```bash
# Restore your backup of claude_desktop_config.json
cp claude_desktop_config.json.backup ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### 2. Use Legacy Binary
```bash
# Build and use the legacy version
cargo build --release --bin airsprotocols-mcpserver-filesystem
# Update Claude config to point to old binary
```

### 3. Restore Environment Variables
```bash
# Reset to legacy environment variables
export AIRS_MCP_FS_ENV=development
export AIRS_MCP_FS_CONFIG_DIR=~/.config/airsprotocols-mcpserver-filesystem
```

## Backward Compatibility

The workspace maintains both versions during the transition period:

- **Legacy**: `crates/airsprotocols-mcpserver-filesystem` (preserved for compatibility)
- **New**: `mcp-servers/airsprotocols-mcpserver-filesystem` (recommended for new installations)

Both versions:
- ✅ Provide identical functionality
- ✅ Use the same configuration format
- ✅ Maintain the same performance characteristics
- ✅ Support the same security features

## Troubleshooting

### Side-by-Side Configuration Comparison

**Legacy Configuration (airsprotocols-mcpserver-filesystem):**
```json
{
  "mcpServers": {
    "airsprotocols-mcpserver-filesystem": {
      "command": "/path/to/airs/target/release/airsprotocols-mcpserver-filesystem",
      "args": ["serve"],
      "env": {
        "AIRS_MCP_FS_ENV": "development",
        "AIRS_MCP_FS_CONFIG_DIR": "/Users/username/.config/airsprotocols-mcpserver-filesystem",
        "AIRS_MCP_FS_ROOT_DIR": "/Users/username/projects",
        "AIRS_MCP_FS_ALLOWED_PATHS": "/Users/username/projects,/Users/username/docs"
      }
    }
  }
}
```

**New Configuration (airsprotocols-mcpserver-filesystem):**
```json
{
  "mcpServers": {
    "airsprotocols-mcpserver-filesystem": {
      "command": "/path/to/airs/target/release/airsprotocols-mcpserver-filesystem",
      "args": ["serve"],
      "env": {
        "AIRSPROTOCOLS_MCPSERVER_FS_ENV": "development",
        "AIRSPROTOCOLS_MCPSERVER_FS_CONFIG_DIR": "/Users/username/.config/airsprotocols-mcpserver-filesystem",
        "AIRSPROTOCOLS_MCPSERVER_FS_ROOT_DIR": "/Users/username/projects",
        "AIRSPROTOCOLS_MCPSERVER_FS_ALLOWED_PATHS": "/Users/username/projects,/Users/username/docs"
      }
    }
  }
}
```

### Common Migration Issues

#### Issue: "Binary not found" error
**Symptoms:** Claude Desktop shows connection error or "command not found"
**Solution:**
1. Verify binary exists: `ls -la target/release/airsprotocols-mcpserver-filesystem`
2. Check permissions: `chmod +x target/release/airsprotocols-mcpserver-filesystem`
3. Use absolute path in Claude config: `/full/path/to/airs/target/release/airsprotocols-mcpserver-filesystem`

#### Issue: Environment variables not recognized
**Symptoms:** Server starts but can't access expected directories
**Solution:**
1. Double-check variable name conversion: `AIRS_MCP_FS_*` → `AIRSPROTOCOLS_MCPSERVER_FS_*`
2. Verify environment variable values with: `echo $AIRSPROTOCOLS_MCPSERVER_FS_ROOT_DIR`
3. Test with minimal config (no env vars) first

#### Issue: Claude Desktop can't connect to server
**Symptoms:** "Failed to connect to MCP server" in Claude Desktop
**Solution:**
1. Test server manually: `./target/release/airsprotocols-mcpserver-filesystem --help`
2. Check Claude Desktop logs: `~/Library/Logs/Claude/`
3. Try temporary config with minimal settings
4. Verify JSON syntax in claude_desktop_config.json

#### Issue: Configuration directory not found
**Symptoms:** Server complains about missing config files
**Solution:**
1. Create new config directory: `mkdir -p ~/.config/airsprotocols-mcpserver-filesystem`
2. Copy existing config: `cp -r ~/.config/airsprotocols-mcpserver-filesystem/* ~/.config/airsprotocols-mcpserver-filesystem/`
3. Or keep existing path: Use `AIRSPROTOCOLS_MCPSERVER_FS_CONFIG_DIR=~/.config/airsprotocols-mcpserver-filesystem`

### Validation Commands

**Test new binary:**
```bash
# Build and verify
cargo build --release --package airsprotocols-mcpserver-filesystem
./target/release/airsprotocols-mcpserver-filesystem --help

# Test startup (should show deprecation warning for comparison)
echo 'exit' | ./target/release/airsprotocols-mcpserver-filesystem serve
echo 'exit' | ./target/release/airsprotocols-mcpserver-filesystem serve
```

**Verify environment:**
```bash
# Check all new environment variables
env | grep AIRSPROTOCOLS_MCPSERVER_FS_

# Compare with legacy (should be empty after migration)
env | grep AIRS_MCP_FS_
```

### Debugging Steps

1. **Isolate the Issue:**
   - Does legacy version still work? `./target/release/airsprotocols-mcpserver-filesystem --help`
   - Does new binary build? `cargo build --release --package airsprotocols-mcpserver-filesystem`
   - Can you run new binary? `./target/release/airsprotocols-mcpserver-filesystem --help`

2. **Test Configuration:**
   - Validate JSON syntax: Use online JSON validator or `python -m json.tool claude_desktop_config.json`
   - Check file permissions: `ls -la ~/Library/Application\ Support/Claude/claude_desktop_config.json`

3. **Progressive Testing:**
   - Start with minimal config (just command path)
   - Add environment variables one by one
   - Test each change with Claude Desktop restart

### Performance Validation

**Expected identical performance:**
```bash
# Time comparison (should be nearly identical)
time ./target/release/airsprotocols-mcpserver-filesystem --help
time ./target/release/airsprotocols-mcpserver-filesystem --help

# Memory usage (using Activity Monitor or htop)
# Response times should remain sub-100ms
```

## FAQ

### Q: Will my existing configuration work?
**A:** Yes! The new binary maintains full backward compatibility with existing configuration files and directories.

### Q: Do I need to migrate my configuration directory?
**A:** No, it's optional. Your existing `~/.config/airsprotocols-mcpserver-filesystem/` directory will continue to work. Migration to `~/.config/airsprotocols-mcpserver-filesystem/` is recommended but not required.

### Q: What if I have custom automation or scripts?
**A:** Update any hardcoded paths to use `airsprotocols-mcpserver-filesystem` instead of `airsprotocols-mcpserver-filesystem`. The command-line interface remains identical.

### Q: Can I run both versions simultaneously?
**A:** Yes, during the transition period you can have both configured in Claude Desktop with different names (e.g., "airsprotocols-mcpserver-filesystem-legacy" and "airsprotocols-mcpserver-filesystem").

### Q: When will the legacy version be deprecated?
**A:** The legacy version will be maintained until **December 31, 2025** to ensure smooth transition. Here's the timeline:

**Migration Timeline:**
- **Phase 1 (September 2025)**: New architecture available, both versions supported
- **Phase 2 (October-November 2025)**: Migration notices active, documentation updated
- **Phase 3 (December 2025)**: Final month for migration, increased deprecation warnings
- **Phase 4 (January 1, 2026)**: Legacy version removed from workspace

**Deprecation Notices:**
- Legacy binary shows migration notice on startup
- Legacy README prominently displays migration information
- Documentation consistently points to new version

## Support

If you encounter issues during migration:

1. **Check the troubleshooting section** in the main README
2. **Verify your configuration** against the examples in this guide
3. **Test with the legacy version** to isolate migration-specific issues
4. **Report issues** with detailed steps to reproduce

## Migration Checklist

- [ ] Build new binary: `cargo build --release --bin airsprotocols-mcpserver-filesystem`
- [ ] Update Claude Desktop configuration JSON
- [ ] Update environment variables (if using)
- [ ] Update any scripts or automation
- [ ] Test basic functionality with Claude Desktop
- [ ] Verify performance characteristics
- [ ] Update bookmarks/documentation to point to new binary
- [ ] (Optional) Migrate configuration directory
- [ ] Create backup of working legacy configuration

**Estimated migration time:** 5-15 minutes depending on complexity of your setup.

## What's Next

After successful migration, you're ready to take advantage of:

- **Future MCP servers** that will be added to the `mcp-servers/` directory
- **Improved documentation** with clearer examples and guides
- **Better project organization** that scales with the AIRS ecosystem

Welcome to the new AIRS MCP Server architecture! 🚀