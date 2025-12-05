# Migration Plan: airs-mcp → airsprotocols

This document outlines the plan for migrating the `airs-mcp` crate from the `airsstack` repository to the new `airsprotocols` repository.

---

## 📋 Overview

### What's Moving
- **Source:** `airsstack/crates/airs-mcp/`
- **Destination:** `airsprotocols/protocols/mcp/`
- **New Crate Name:** `airsprotocols-mcp` (was `airs-mcp`)
- **Version:** 0.2.3 (maintained during migration)

### Why Migrate
- **Better Organization:** Separate protocol implementations from applications
- **Clearer Scope:** `airsstack` focuses on applications, `airsprotocols` on protocols
- **Future Growth:** Room for A2A, LLM APIs, and other protocols
- **Reusability:** Make protocols more accessible to external projects

---

## 🎯 Migration Goals

1. ✅ **Zero Regression** - All functionality remains intact
2. ✅ **Preserve History** - Maintain git history (where feasible)
3. ✅ **Update Consumers** - Fix all dependent projects
4. ✅ **Complete Documentation** - Update all references and guides
5. ✅ **Clean Transition** - Clear path for external users

---

## 🏗️ Structure Changes

### Before (airsstack)
```
airsstack/
└── crates/
    └── airs-mcp/              # Crate name: airs-mcp
        ├── Cargo.toml
        ├── src/
        ├── examples/
        └── ...
```

### After (airsprotocols)
```
airsprotocols/
└── protocols/
    └── mcp/                   # Crate name: airsprotocols-mcp
        ├── Cargo.toml         # Updated package name
        ├── src/
        ├── examples/
        └── ...
```

### Package Name Change
```toml
# Before
[package]
name = "airs-mcp"

# After
[package]
name = "airsprotocols-mcp"
```

### Import Changes
```rust
// Before
use airs_mcp::{McpClient, McpServer};

// After
use airsprotocols_mcp::{McpClient, McpServer};
```

---

## 📅 Migration Phases

### **Phase 1: Repository Setup** ✅
**Timeline:** Day 1 (2-3 hours)

- [x] Initialize `airsprotocols` repository structure
- [x] Create workspace `Cargo.toml`
- [x] Add license files (MIT, Apache-2.0)
- [x] Create documentation structure
- [ ] Set up CI/CD workflow

**Deliverables:**
- Clean repository structure
- Workspace configuration
- Basic documentation

---

### **Phase 2: Copy and Adapt** 🚧
**Timeline:** Day 1-2 (4-6 hours)

**Step 2.1: Copy Source Code**
```bash
# Copy complete crate structure
rsync -av --exclude='target' \
          --exclude='.DS_Store' \
          airsstack/crates/airs-mcp/ \
          airsprotocols/protocols/mcp/
```

**Step 2.2: Update Package Metadata**

File: `protocols/mcp/Cargo.toml`
```toml
[package]
name = "airsprotocols-mcp"          # Changed
version = "0.2.3"
edition.workspace = true
# ... other workspace inherited fields

# Update URLs
homepage = "https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp"
repository = "https://github.com/airsstack/airsprotocols"
documentation = "https://docs.rs/airsprotocols-mcp"

[dependencies]
# All dependencies use .workspace = true
serde.workspace = true
tokio.workspace = true
# ...
```

**Step 2.3: Update Example Projects**

Each example's `Cargo.toml` needs updating:
```toml
# Before
[dependencies]
airs-mcp = { path = "../../" }

# After
[dependencies]
airsprotocols-mcp = { path = "../../" }
```

**Step 2.4: Update Source Code Imports**

Use search/replace across all files:
```bash
# Update crate references in source
find protocols/mcp -name "*.rs" -type f -exec sed -i '' 's/airs_mcp/airsprotocols_mcp/g' {} +

# Update in documentation
find protocols/mcp -name "*.md" -type f -exec sed -i '' 's/airs-mcp/airsprotocols-mcp/g' {} +
```

**Deliverables:**
- Complete MCP crate in new location
- Updated package names and imports
- All examples updated

---

### **Phase 3: Verification** 🚧
**Timeline:** Day 2 (3-4 hours)

**Step 3.1: Build Testing**
```bash
cd airsprotocols

# Full workspace build
cargo build --workspace

# Run all tests
cargo test --workspace

# Build all examples
cargo build --workspace --examples

# Check documentation
cargo doc --workspace --no-deps --open
```

**Step 3.2: Example Verification**
```bash
# Test each example individually
cd protocols/mcp
cargo run --example stdio-server
cargo run --example http-oauth2-server
# ... test all 6 examples
```

**Step 3.3: Documentation Build**
```bash
cd protocols/mcp/docs
mdbook build
mdbook serve  # Verify at http://localhost:3000
```

**Success Criteria:**
- [ ] All builds pass without warnings
- [ ] All tests pass (unit + integration)
- [ ] All examples compile and run
- [ ] Documentation builds correctly
- [ ] Benchmarks compile

---

### **Phase 4: Update Consumers in airsstack** 🚧
**Timeline:** Day 2-3 (3-5 hours)

**Step 4.1: Update airs-mcpserver-fs**

File: `airsstack/mcp-servers/airs-mcpserver-fs/Cargo.toml`

**Option A: Git Dependency** (Recommended)
```toml
[dependencies]
# Before
airs-mcp = { version = "0.2.3", path = "../../crates/airs-mcp" }

# After
airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols", tag = "mcp-v0.2.3" }
```

**Option B: Path Dependency** (Development)
```toml
[dependencies]
airsprotocols-mcp = { path = "../../../airsprotocols/protocols/mcp" }
```

**Step 4.2: Update Source Imports**
```bash
cd airsstack/mcp-servers/airs-mcpserver-fs

# Update all imports
find src -name "*.rs" -type f -exec sed -i '' 's/use airs_mcp/use airsprotocols_mcp/g' {} +
```

**Step 4.3: Update airsstack Workspace**

File: `airsstack/Cargo.toml`
```toml
[workspace]
members = [
    # "crates/airs-mcp",  # REMOVED - migrated to airsprotocols
    "crates/airs-memspec",
    "mcp-servers/airs-mcpserver-fs",
]

[workspace.dependencies]
# Remove or update airs-mcp reference
# airsprotocols-mcp = { git = "...", tag = "..." }  # If needed
```

**Step 4.4: Verify airsstack Build**
```bash
cd airsstack

# Clean build to catch any issues
cargo clean
cargo build --workspace
cargo test --workspace

# Specifically test airs-mcpserver-fs
cd mcp-servers/airs-mcpserver-fs
cargo test
cargo build --release
```

**Deliverables:**
- `airs-mcpserver-fs` uses new crate location
- All airsstack builds pass
- Integration tests pass

---

### **Phase 5: Documentation Updates** 🚧
**Timeline:** Day 3-4 (2-3 hours)

**Step 5.1: Update Protocol Documentation**

File: `airsprotocols/protocols/mcp/README.md`
```markdown
# airsprotocols-mcp

> **Note:** This crate was previously published as `airs-mcp` and has been 
> migrated to the `airsprotocols` repository as part of a reorganization 
> to better separate protocol implementations from applications.

## Migration from `airs-mcp`

If you're migrating from `airs-mcp`:

1. Update your `Cargo.toml`:
   ```toml
   # Before
   airs-mcp = "0.2.3"
   
   # After
   airsprotocols-mcp = "0.2.3"
   ```

2. Update your imports:
   ```rust
   // Before
   use airs_mcp::McpClient;
   
   // After
   use airsprotocols_mcp::McpClient;
   ```

[Rest of README continues...]
```

**Step 5.2: Add Migration Notice to airsstack**

Create: `airsstack/crates/MOVED-airs-mcp.md`
```markdown
# airs-mcp has moved!

The `airs-mcp` crate has been relocated to the `airsprotocols` repository.

**New location:** https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp
**New crate name:** `airsprotocols-mcp`

## Why?

To better organize the codebase:
- `airsstack` → Application-level implementations
- `airsprotocols` → Protocol and API implementations

## How to Update

See the [migration guide](https://github.com/airsstack/airsprotocols/blob/main/MIGRATIONS.md)
```

**Step 5.3: Update Links**

Search and replace in documentation:
- Repository URLs
- Documentation links
- Example code references

**Deliverables:**
- Updated README files
- Migration notices in place
- All documentation links working

---

### **Phase 6: Git Operations** 🚧
**Timeline:** Day 4 (2-3 hours)

**Step 6.1: Commit to airsprotocols**
```bash
cd airsprotocols

git add .
git commit -m "Add MCP protocol implementation

Migrate airs-mcp from airsstack repository as airsprotocols-mcp.

This is a complete migration of the Model Context Protocol implementation
including:
- Complete protocol implementation (JSON-RPC 2.0 + MCP)
- Transport layers (stdio, HTTP)
- Authentication and authorization
- Provider implementations (Tool, Resource, Prompt)
- 6 integration examples
- Comprehensive test suite
- Documentation and benchmarks

Original repository: https://github.com/airsstack/airsstack
Original path: crates/airs-mcp/
Original crate name: airs-mcp
New crate name: airsprotocols-mcp

Version: 0.2.3"

# Tag the migration
git tag -a mcp-v0.2.3 -m "MCP protocol v0.2.3 - Initial migration from airsstack"

# Push to remote
git push origin main
git push origin --tags
```

**Step 6.2: Update airsstack Repository**
```bash
cd airsstack

# Option A: Remove directory completely
rm -rf crates/airs-mcp

# Option B: Replace with stub (if transition period needed)
# Keep directory but add notice

git add .
git commit -m "Migrate airs-mcp to airsprotocols repository

The airs-mcp crate has been moved to the airsprotocols repository
for better organization of protocol implementations.

New location: https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp
New crate name: airsprotocols-mcp

Updated:
- Removed crates/airs-mcp/ (or added migration notice)
- Updated workspace Cargo.toml
- Updated airs-mcpserver-fs dependencies
- Added migration documentation

For migration instructions, see:
https://github.com/airsstack/airsprotocols/blob/main/MIGRATIONS.md"

git push origin main
```

**Deliverables:**
- Both repositories committed and pushed
- Version tagged in airsprotocols
- Clear commit messages

---

### **Phase 7: Final Verification** 🚧
**Timeline:** Day 4-5 (2-3 hours)

**Checklist:**

**airsprotocols repository:**
- [ ] `cargo build --workspace` passes
- [ ] `cargo test --workspace` passes
- [ ] `cargo build --workspace --examples` passes
- [ ] All 6 examples run successfully
- [ ] Documentation builds (`cargo doc`)
- [ ] mdBook builds (`mdbook build`)
- [ ] CI/CD pipeline passes (if configured)

**airsstack repository:**
- [ ] `cargo build --workspace` passes
- [ ] `cargo test --workspace` passes
- [ ] `airs-mcpserver-fs` builds and tests pass
- [ ] Integration tests pass
- [ ] No references to old path remain

**Documentation:**
- [ ] README files are accurate
- [ ] Migration guide is complete
- [ ] All links work (no 404s)
- [ ] Examples show correct usage

**Git:**
- [ ] Tags are in place
- [ ] Commit history is clean
- [ ] Remote branches are synced

---

## 🔄 Rollback Plan

If issues arise during migration:

### Immediate Rollback (During Migration)
```bash
# In airsstack - undo changes
cd airsstack
git reset --hard HEAD~1

# In airsprotocols - undo changes
cd airsprotocols
git reset --hard HEAD~1
```

### Post-Migration Rollback
```bash
# Revert airs-mcpserver-fs to use local path
cd airsstack/mcp-servers/airs-mcpserver-fs
# Edit Cargo.toml to restore path dependency
git commit -am "Rollback: Restore local airs-mcp dependency"

# Restore airs-mcp to workspace
cd ../..
# Edit Cargo.toml to re-add crates/airs-mcp
# Copy back crate from backup or git history
git commit -am "Rollback: Restore airs-mcp to workspace"
```

---

## 📊 Impact Analysis

### Files Changed

**airsprotocols:**
- New files: ~253 files (entire MCP crate)
- Modified: `Cargo.toml`, `README.md`, `MIGRATIONS.md`

**airsstack:**
- Removed: `crates/airs-mcp/` (~253 files)
- Modified: `Cargo.toml`, `mcp-servers/airs-mcpserver-fs/Cargo.toml`
- Modified: All `.rs` files in `airs-mcpserver-fs` (import changes)

### Breaking Changes

**For External Users:**
1. **Crate name change:** `airs-mcp` → `airsprotocols-mcp`
2. **Import path change:** `airs_mcp::*` → `airsprotocols_mcp::*`
3. **Repository change:** URL updates needed
4. **Dependency specification:** Path/git/registry changes

**Mitigation:**
- Clear migration guide (this document)
- Updated examples showing new usage
- Version maintained (0.2.3)
- Functionality unchanged

---

## ✅ Success Criteria

Migration is complete when:

1. **Build & Test:**
   - [ ] airsprotocols workspace builds cleanly
   - [ ] All tests pass in airsprotocols
   - [ ] airsstack workspace builds cleanly
   - [ ] All tests pass in airsstack
   - [ ] No warnings or errors

2. **Functionality:**
   - [ ] All MCP features work in new location
   - [ ] airs-mcpserver-fs functions correctly
   - [ ] Examples run successfully
   - [ ] No regressions detected

3. **Documentation:**
   - [ ] README files updated
   - [ ] Migration guide complete
   - [ ] All links working
   - [ ] Examples updated

4. **Repository:**
   - [ ] Clean commits in both repos
   - [ ] Proper tags applied
   - [ ] CI/CD passing
   - [ ] Remote synced

---

## 🎯 Timeline Summary

| Phase | Duration | Status |
|-------|----------|--------|
| 1. Repository Setup | 2-3 hours | ✅ Complete |
| 2. Copy and Adapt | 4-6 hours | 🚧 In Progress |
| 3. Verification | 3-4 hours | ⏳ Pending |
| 4. Update Consumers | 3-5 hours | ⏳ Pending |
| 5. Documentation | 2-3 hours | ⏳ Pending |
| 6. Git Operations | 2-3 hours | ⏳ Pending |
| 7. Final Verification | 2-3 hours | ⏳ Pending |
| **Total** | **18-27 hours** | **3-4 days** |

---

## 📞 Questions or Issues?

If you encounter issues during or after migration:

1. **Check this document** for rollback procedures
2. **Review the checklist** to ensure all steps completed
3. **Open an issue** on the appropriate repository:
   - airsprotocols: Protocol implementation issues
   - airsstack: Application integration issues
4. **Consult examples** for correct usage patterns

---

## 📚 References

- **MCP Specification:** https://modelcontextprotocol.io/
- **airsprotocols Repository:** https://github.com/airsstack/airsprotocols
- **airsstack Repository:** https://github.com/airsstack/airsstack
- **Original airs-mcp Documentation:** See `protocols/mcp/docs/`

---

**Status:** 🚧 Migration In Progress  
**Last Updated:** 2025-12-05  
**Estimated Completion:** 2025-12-08
