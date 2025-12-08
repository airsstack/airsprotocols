# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0-rc.1] - 2025-12-08

### 🚀 Initial Release

- **New Library**: Initial release of `airsprotocols-mcp`, resetting versioning to `1.0.0-rc.1`.
- **Source**: Migrated from the original `airsstack/airsstack` repository (previously `airs-mcp` v0.2.3).
- **Goal**: To serve as a standalone, stable library for the Model Context Protocol implementation.

### 🔄 Migration Guide

**BREAKING CHANGE:** Crate renamed and repository moved.

- **Crate Renamed:** `airs-mcp` → `airsprotocols-mcp`
- **Repository:** https://github.com/airsstack/airsprotocols

**Dependency Update:**
```toml
# Old
airs-mcp = "0.2.3"

# New
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
```

**Import Update:**
```rust
// Old
use airs_mcp::McpClient;

// New
use airsprotocols_mcp::McpClient;
```
All imports must be updated from `airs_mcp` to `airsprotocols_mcp`.