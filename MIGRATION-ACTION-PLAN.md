# Migration Action Plan: airs-mcp → airsprotocols-mcp

**Status:** Ready for Execution  
**Created:** December 6, 2025  
**Scope:** One-way copy migration from airsstack to airsprotocols  
**Impact:** airsstack remains unchanged (except README update)

---

## 📋 Executive Summary

### What This Migration Does

**IN SCOPE:**
- ✅ Copy `airs-mcp` from `airsstack/crates/airs-mcp/` to `airsprotocols/protocols/mcp/`
- ✅ Rename package to `airsprotocols-mcp` in new location
- ✅ Verify all builds, tests, and examples work independently
- ✅ Run full integration test suites (Python tests included)
- ✅ Update documentation in both repositories
- ✅ Commit changes to both repositories

**OUT OF SCOPE:**
- ❌ Modifying `airs-mcpserver-fs` dependencies
- ❌ Removing `airs-mcp` from `airsstack`
- ❌ Changing any imports in `airsstack` code
- ❌ Creating git tags
- ❌ Publishing to crates.io
- ❌ Setting up CI/CD

### Key Decisions

1. **No Git History Preservation** - Clean start with reference to original source
2. **No Git Tags** - Deferred for later
3. **No CI/CD Setup** - Deferred for future work
4. **No crates.io Publishing** - Git dependency only for now
5. **Keep Original** - `airs-mcp` remains in `airsstack` during transition
6. **Full Testing** - Complete integration test suite execution required

### Timeline

- **Optimistic:** 10-12 hours
- **Realistic:** 14-18 hours (recommended estimate)
- **Pessimistic:** 20-24 hours

---

## 🎯 Prerequisites

### Required Tools

- [ ] Rust toolchain (1.88+)
- [ ] Python 3.9+ with pip
- [ ] Git
- [ ] mdBook (`cargo install mdbook`)
- [ ] rsync

### Repository State

- [ ] Both repos in clean git state (no uncommitted changes)
- [ ] All existing `airsstack` builds pass
- [ ] All existing `airsstack` tests pass
- [ ] Sufficient disk space (~2GB for workspace builds)

### Environment Setup

- [ ] Terminal with bash/zsh
- [ ] Text editor ready
- [ ] Internet connection for dependencies

---

## 🚀 PHASE 0: Pre-Migration Verification

**Estimated Time:** 30 minutes

### Step 0.1: Verify Repository State

```bash
# Check airsstack
cd /Users/hiraq/Projects/airsstack/airsstack
git status
# Expected: "nothing to commit, working tree clean"

# Check airsprotocols
cd /Users/hiraq/Projects/airsstack/airsprotocols
git status
# Expected: "nothing to commit, working tree clean"
```

**Checkpoint:** Both repos show clean working directory

### Step 0.2: Verify Current Builds

```bash
# Test airsstack builds and tests
cd /Users/hiraq/Projects/airsstack/airsstack
cargo build --workspace
cargo test --workspace
```

**Checkpoint:** All builds and tests pass

### Step 0.3: Create Backup (Recommended)

```bash
cd /Users/hiraq/Projects/airsstack
tar -czf backup-airsstack-$(date +%Y%m%d-%H%M%S).tar.gz airsstack/
tar -czf backup-airsprotocols-$(date +%Y%m%d-%H%M%S).tar.gz airsprotocols/
```

**Checkpoint:** Backups created successfully

### Step 0.4: Document Current State

**Record:**
- Current `airs-mcp` version: `0.2.3`
- Current `airsprotocols` version: `0.2.3`
- Migration start time: `_________`

**Deliverables:**
- [ ] Clean git state confirmed
- [ ] All builds passing
- [ ] Backups created
- [ ] Current state documented

---

## 🏗️ PHASE 1: Repository Structure Setup

**Estimated Time:** 30 minutes

### Step 1.1: Create Directory Structure

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols
mkdir -p protocols/mcp
```

**Verify:**
```bash
ls -la protocols/
```

**Expected Output:**
```
drwxr-xr-x  3 user  staff  96 Dec  6 XX:XX .
drwxr-xr-x  8 user  staff 256 Dec  6 XX:XX ..
drwxr-xr-x  2 user  staff  64 Dec  6 XX:XX mcp
```

**Checkpoint:** `protocols/mcp/` directory exists

### Step 1.2: Verify Workspace Configuration

```bash
cat Cargo.toml | grep -A 2 "members"
```

**Expected Output:**
```toml
[workspace]
members = [
    "protocols/mcp",
]
```

**Checkpoint:** Workspace already configured correctly ✅

**Deliverables:**
- [ ] `protocols/mcp/` directory created
- [ ] Workspace configuration verified

---

## 📦 PHASE 2: Copy and Adapt

**Estimated Time:** 2-3 hours

### Step 2.1: Copy Source Code with Exclusions

```bash
cd /Users/hiraq/Projects/airsstack/airsstack

# Copy with exclusions
rsync -av --progress \
  --exclude='target' \
  --exclude='Cargo.lock' \
  --exclude='.DS_Store' \
  --exclude='__pycache__' \
  --exclude='.pytest_cache' \
  --exclude='*.pyc' \
  --exclude='venv' \
  --exclude='.venv' \
  --exclude='*.egg-info' \
  crates/airs-mcp/ \
  ../airsprotocols/protocols/mcp/
```

**Verify Copy:**
```bash
cd ../airsprotocols/protocols/mcp
ls -la
```

**Expected Files:**
- Cargo.toml
- README.md
- CHANGELOG.md
- src/
- examples/
- tests/
- benches/
- docs/

**Check Size:**
```bash
find . -type f | wc -l    # Should be ~200-300 files
du -sh .                   # Should be ~1-10 MB (without target/)
```

**Checkpoint:** All source files copied successfully

### Step 2.2: Update Package Metadata

**Edit:** `protocols/mcp/Cargo.toml`

**Find and Replace:**
```toml
# Line 2: Change package name
[package]
name = "airsprotocols-mcp"  # WAS: "airs-mcp"
```

**Update URLs (around lines 14-15):**
```toml
documentation = "https://docs.rs/airsprotocols-mcp"
homepage = "https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp"
repository = "https://github.com/airsstack/airsprotocols"
```

**Verify Workspace Inheritance:**
```toml
# These should remain as-is:
version.workspace = true
edition.workspace = true
authors.workspace = true
license.workspace = true
```

**Checkpoint:** Package metadata updated

### Step 2.3: Update Example Projects' Cargo.toml

**Examples with Cargo.toml files:**
1. `examples/http-apikey-client-integration/Cargo.toml`
2. `examples/http-apikey-server-integration/` (check if Cargo.toml exists)
3. `examples/http-oauth2-client-integration/Cargo.toml`
4. `examples/http-oauth2-server-integration/Cargo.toml`
5. `examples/stdio-server-integration/` (check if Cargo.toml exists)

**For each Cargo.toml found, update dependency:**

```toml
# BEFORE:
[dependencies]
airs-mcp = { path = "../../" }

# AFTER:
[dependencies]
airsprotocols-mcp = { path = "../../" }
```

**Verification Command:**
```bash
find examples -name "Cargo.toml" -exec grep -l "airs-mcp" {} \;
# Should return empty after updates
```

**Checkpoint:** All example Cargo.toml files updated

### Step 2.4: Update Source Code Imports

**Update src/ directory:**
```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

find src -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} +
find src -name "*.bak" -delete
```

**Update tests/ directory:**
```bash
find tests -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} + 2>/dev/null || true
find tests -name "*.bak" -delete 2>/dev/null || true
```

**Update examples/ directory:**
```bash
find examples -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} +
find examples -name "*.bak" -delete
```

**Update benches/ directory:**
```bash
find benches -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} + 2>/dev/null || true
find benches -name "*.bak" -delete 2>/dev/null || true
```

**Verification:**
```bash
# Should return nothing:
grep -r "use airs_mcp" src/ tests/ examples/ benches/ 2>/dev/null || echo "✅ All imports updated"

# Also check for crate name in comments/docs:
grep -r "airs-mcp" src/ | grep -v "airsprotocols-mcp" || echo "✅ No old references"
```

**Checkpoint:** All Rust imports updated

### Step 2.5: Update Documentation

**Update Markdown files:**
```bash
find . -name "*.md" -type f -exec sed -i.bak 's/airs-mcp/airsprotocols-mcp/g' {} +
find . -name "*.md" -type f -exec sed -i.bak 's/airs_mcp/airsprotocols_mcp/g' {} +
find . -name "*.bak" -delete
```

**Manually update README.md:**

Add migration notice at the top (after title):

```markdown
# airsprotocols-mcp

> **Migration Notice:** This crate was previously `airs-mcp` in the 
> [airsstack](https://github.com/airsstack/airsstack) repository. 
> Migrated to `airsprotocols` as of December 2025 for better organization.

## Migrating from `airs-mcp`

If you're updating from `airs-mcp`:

1. Update `Cargo.toml`:
   ```toml
   # Before
   airs-mcp = "0.2.3"
   
   # After
   airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
   ```

2. Update imports:
   ```rust
   // Before
   use airs_mcp::McpClient;
   
   // After
   use airsprotocols_mcp::McpClient;
   ```

[Continue with regular README content...]
```

**Update CHANGELOG.md:**

Add entry at the top:

```markdown
# Changelog

## [Unreleased]

### Changed
- **MIGRATION:** Crate renamed from `airs-mcp` to `airsprotocols-mcp`
- **MIGRATION:** Moved from `airsstack` repository to `airsprotocols` repository
- Repository URL: https://github.com/airsstack/airsprotocols
- Original source: https://github.com/airsstack/airsstack/tree/main/crates/airs-mcp

## [0.2.3] - 2024-XX-XX

[Previous changelog entries...]
```

**Check docs/ directory:**
```bash
grep -r "airs-mcp" docs/ 2>/dev/null | wc -l
# If returns >0, manually review and update
```

**Verification:**
```bash
# Check for remaining old references:
grep -r "airs-mcp" . --include="*.md" --include="*.rs" --include="*.toml" \
  | grep -v "airsprotocols-mcp" \
  | grep -v "was airs-mcp" \
  | grep -v "previously airs-mcp" \
  || echo "✅ All documentation updated"
```

**Checkpoint:** All documentation updated

**Deliverables:**
- [ ] Source code copied successfully
- [ ] Package name updated to `airsprotocols-mcp`
- [ ] All example Cargo.toml files updated
- [ ] All Rust imports updated (src, tests, examples, benches)
- [ ] All documentation files updated
- [ ] README.md has migration notice
- [ ] CHANGELOG.md updated
- [ ] No remaining `airs-mcp` or `airs_mcp` references in code

---

## ✅ PHASE 3: Build and Test Verification

**Estimated Time:** 3-4 hours

### Step 3.1: Clean Build

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Clean any previous build artifacts
cargo clean

# Build entire workspace
cargo build --workspace --all-features
```

**Expected:** Build succeeds with 0 errors

**If warnings appear:** Document them for review

**Checkpoint:** Workspace builds successfully

### Step 3.2: Run Unit and Integration Tests

```bash
# Run all Rust tests
cargo test --workspace --all-features
```

**Expected:** All tests pass

**Document:**
- Number of tests run: `_________`
- Pass rate: `_________`
- Any failures: `_________`

**Checkpoint:** All Rust tests pass

### Step 3.3: Build All Examples

```bash
cargo build --workspace --examples
```

**Expected:** All examples compile

**Checkpoint:** Examples build successfully

### Step 3.4: Generate Documentation

```bash
cargo doc --workspace --no-deps --open
```

**Manual Verification:**
- [ ] Documentation opens in browser
- [ ] Crate name shows as `airsprotocols_mcp`
- [ ] All modules documented
- [ ] No broken links visible

**Checkpoint:** Documentation generates correctly

### Step 3.5: Compile Benchmarks

```bash
cd protocols/mcp
cargo build --benches
```

**Expected:** Benchmarks compile (don't need to run them)

**Checkpoint:** Benchmarks compile

### Step 3.6: Build mdBook Documentation

```bash
# Check if mdBook is installed
which mdbook || echo "Install with: cargo install mdbook"

# Build documentation
cd protocols/mcp/docs
mdbook build

# Serve locally for verification
mdbook serve
```

**Manual Verification:**
- Open http://localhost:3000
- [ ] Documentation renders correctly
- [ ] Navigation works
- [ ] Code examples render
- [ ] No broken links

Press Ctrl+C to stop server

**Checkpoint:** mdBook documentation builds and renders

**Deliverables:**
- [ ] `cargo build --workspace` succeeds
- [ ] `cargo test --workspace` passes (document test count)
- [ ] All examples compile
- [ ] Cargo docs generate successfully
- [ ] Benchmarks compile
- [ ] mdBook documentation builds and serves
- [ ] No errors in any build step

---

## 🧪 PHASE 4: Python Integration Tests

**Estimated Time:** 3-4 hours

### Step 4.1: Setup Python Environment

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# Check Python version
python3 --version
# Should be 3.9 or higher
```

**Checkpoint:** Python 3.9+ available

### Step 4.2: Test stdio-server-integration

```bash
cd examples/stdio-server-integration/tests

# Install dependencies
pip3 install -r requirements.txt

# Run tests
python3 test_stdio_integration.py
```

**Expected:** Tests pass

**Document Results:**
```
Test Suite: stdio-server-integration
Status: PASS/FAIL
Tests Run: _________
Duration: _________
```

**Checkpoint:** STDIO tests pass

### Step 4.3: Test http-apikey-client-integration

```bash
cd ../../http-apikey-client-integration/tests

# Install dependencies
pip3 install -r requirements.txt

# Run test scripts
./run_tests.sh
# OR individually:
python3 test_http_client_mock_integration.py
python3 test_http_client_production_integration.py
```

**Expected:** All tests pass

**Document Results:**
```
Test Suite: http-apikey-client-integration
Status: PASS/FAIL
Tests Run: _________
Duration: _________
```

**Checkpoint:** API Key client tests pass

### Step 4.4: Test http-apikey-server-integration

```bash
cd ../../http-apikey-server-integration/tests

# Install dependencies
pip3 install -r requirements.txt

# Run tests
./run_tests.sh
# OR:
python3 test_http_apikey_integration.py
```

**Expected:** Tests pass

**Document Results:**
```
Test Suite: http-apikey-server-integration
Status: PASS/FAIL
Tests Run: _________
Duration: _________
```

**Checkpoint:** API Key server tests pass

### Step 4.5: Test http-oauth2-client-integration

```bash
cd ../../http-oauth2-client-integration

# Check if test keys exist
ls -la test_keys/
# Should show: private_key.pem, public_key.pem

# If keys missing, generate them:
cd scripts
./setup_keys.sh
# OR:
python3 setup_keys.py

cd ../tests

# Install dependencies
pip3 install -r requirements.txt

# Run tests (if test script exists)
# Check what test files exist:
ls -la *.py
```

**Expected:** Tests pass (if test files exist)

**Document Results:**
```
Test Suite: http-oauth2-client-integration
Status: PASS/FAIL/SKIPPED
Tests Run: _________
Notes: _________
```

**Checkpoint:** OAuth2 client tests complete

### Step 4.6: Test http-oauth2-server-integration

```bash
cd ../../http-oauth2-server-integration

# Check if test keys exist
ls -la keys/
# Should show: test_rsa_key.pem

cd tests

# Install dependencies
pip3 install -r requirements.txt

# Run test suite
python3 run_tests.py
# OR individually:
python3 test_oauth2_integration.py
python3 test_oauth2_comprehensive.py
```

**Expected:** All tests pass

**Document Results:**
```
Test Suite: http-oauth2-server-integration
Status: PASS/FAIL
Tests Run: _________
Duration: _________
```

**Checkpoint:** OAuth2 server tests pass

### Step 4.7: Integration Test Summary

**Create Summary Document:**

```
=== INTEGRATION TEST RESULTS ===
Date: _________
Total Suites: 5-6
Total Tests: _________
Pass Rate: _________/%

Suite Details:
1. stdio-server-integration: PASS/FAIL
2. http-apikey-client-integration: PASS/FAIL
3. http-apikey-server-integration: PASS/FAIL
4. http-oauth2-client-integration: PASS/FAIL
5. http-oauth2-server-integration: PASS/FAIL

Issues Found:
- _________
- _________

Notes:
- _________
```

**Deliverables:**
- [ ] Python 3.9+ environment ready
- [ ] All test dependencies installed
- [ ] STDIO integration tests pass
- [ ] HTTP API Key client tests pass
- [ ] HTTP API Key server tests pass
- [ ] HTTP OAuth2 client tests complete
- [ ] HTTP OAuth2 server tests pass
- [ ] All test keys present or generated
- [ ] Integration test summary documented
- [ ] All test suites pass OR issues documented

---

## 📚 PHASE 5: Documentation Updates

**Estimated Time:** 1-2 hours

### Step 5.1: Update airsprotocols Root README

**Edit:** `/Users/hiraq/Projects/airsstack/airsprotocols/README.md`

**Update MCP Status Section (around line 15-20):**

```markdown
- **[`airsprotocols-mcp`](protocols/mcp/)** - Model Context Protocol (MCP)
  - Connect AI models to tools, resources, and context
  - JSON-RPC 2.0 foundation with transport abstraction
  - Built-in authentication (API Key, OAuth2) and authorization
  - Stdio and HTTP transport implementations
  - **Status:** ✅ Available (v0.2.3 - migrated from airsstack, Dec 2025)
```

**Update Quick Start Example (if needed):**

Ensure code examples use `airsprotocols_mcp` instead of `airs_mcp`

**Checkpoint:** airsprotocols README updated

### Step 5.2: Update airsprotocols MIGRATIONS.md

**Edit:** `/Users/hiraq/Projects/airsstack/airsprotocols/MIGRATIONS.md`

**Add Completion Status at Top:**

```markdown
# Migration Plan: airs-mcp → airsprotocols

**Status:** ✅ COMPLETED - December 6, 2025

> **Migration Summary:** The `airs-mcp` crate has been successfully migrated 
> to `airsprotocols` as `airsprotocols-mcp` v0.2.3. All builds, tests, and 
> integration tests pass. The original `airs-mcp` remains in the `airsstack` 
> repository during the transition period.

---

[Rest of document continues...]
```

**Update Phase Statuses:**

```markdown
### **Phase 1: Repository Setup** ✅ COMPLETE
### **Phase 2: Copy and Adapt** ✅ COMPLETE
### **Phase 3: Verification** ✅ COMPLETE
### **Phase 4: Documentation Updates** ✅ COMPLETE
### **Phase 5: Git Operations** ✅ COMPLETE
```

**Add "Actual Migration Date":**

```markdown
**Status:** ✅ Migration Complete  
**Completed:** December 6, 2025  
**Duration:** [ACTUAL HOURS]
```

**Checkpoint:** MIGRATIONS.md updated with completion status

### Step 5.3: Update airsstack README

**Edit:** `/Users/hiraq/Projects/airsstack/airsstack/README.md`

**Add Migration Notice (after line 21, before "Working Components"):**

```markdown
## 📦 Protocol Migration

> **Note:** The Model Context Protocol (`airs-mcp`) implementation has been 
> migrated to the [`airsprotocols`](https://github.com/airsstack/airsprotocols) 
> repository as `airsprotocols-mcp` for better organization. The local 
> `airs-mcp` crate remains available during the transition period.
>
> **New Location:** https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp  
> **Status:** Both versions maintained at v0.2.3  
> **Timeline:** Full migration to new location planned after stabilization

### Working Components
```

**Update Links (if they point to airs-mcp examples):**

Consider adding notes like:
```markdown
[**See MCP Server Example →**](crates/airs-mcp/examples/simple-mcp-server/)
*(Note: MCP is now primarily developed in [airsprotocols](https://github.com/airsstack/airsprotocols))*
```

**Checkpoint:** airsstack README updated with migration notice

### Step 5.4: Verify All Documentation Links

**Check for broken links:**

```bash
# In airsprotocols
cd /Users/hiraq/Projects/airsstack/airsprotocols
grep -r "github.com" README.md MIGRATIONS.md protocols/mcp/README.md | grep -v "airsstack"
# Verify URLs are correct

# In airsstack
cd /Users/hiraq/Projects/airsstack/airsstack
grep "airsprotocols" README.md
# Verify new links work
```

**Manual checks:**
- [ ] Open each GitHub link in browser
- [ ] Verify relative links work (e.g., `[MCP](protocols/mcp/)`)
- [ ] Check documentation examples render correctly

**Checkpoint:** All documentation links verified

**Deliverables:**
- [ ] airsprotocols README.md updated
- [ ] airsprotocols MIGRATIONS.md marked complete
- [ ] airsstack README.md has migration notice
- [ ] All documentation links verified
- [ ] No broken links found

---

## 🗂️ PHASE 6: Git Operations

**Estimated Time:** 30 minutes

### Step 6.1: Review Changes in airsprotocols

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Review what will be committed
git status

# Review actual changes
git diff

# Check untracked files
git ls-files --others --exclude-standard
```

**Expected Changes:**
- New directory: `protocols/mcp/` (with all content)
- Modified: `README.md`
- Modified: `MIGRATIONS.md`

**Checkpoint:** Changes look correct

### Step 6.2: Commit to airsprotocols

**Stage changes:**
```bash
git add protocols/mcp/
git add README.md
git add MIGRATIONS.md
git add MIGRATION-ACTION-PLAN.md
```

**Verify staged changes:**
```bash
git status
```

**Commit with detailed message:**
```bash
git commit -m "feat(mcp): migrate Model Context Protocol implementation from airsstack

Migrate airs-mcp from airsstack repository as airsprotocols-mcp v0.2.3.

This migration includes:
- Complete MCP protocol implementation (JSON-RPC 2.0 + MCP specification)
- Transport layers (stdio, HTTP) with connection pooling
- Authentication (API Key, OAuth2) and authorization
- Provider implementations (Tool, Resource, Prompt)
- 5 comprehensive integration examples with test suites
- Complete test suite (unit, integration, property-based, Python)
- Performance benchmarks
- Documentation (mdBook + inline docs)

Migration details:
- Source: https://github.com/airsstack/airsstack/tree/main/crates/airs-mcp
- Original crate: airs-mcp v0.2.3
- New crate: airsprotocols-mcp v0.2.3
- Migration date: December 6, 2025
- Rationale: Separate protocol implementations from applications

Testing:
- All Rust unit tests pass
- All Rust integration tests pass
- All Python integration tests pass
- All examples compile and run
- Documentation builds successfully

The original airs-mcp crate remains in airsstack during transition period
for backward compatibility.

Breaking changes for external users:
- Crate name: airs-mcp → airsprotocols-mcp
- Import paths: airs_mcp::* → airsprotocols_mcp::*
- Repository URL changed

Migration guide: /MIGRATIONS.md
Action plan: /MIGRATION-ACTION-PLAN.md"
```

**Checkpoint:** Changes committed

### Step 6.3: Push to Remote

```bash
# Push commits
git push origin main
```

**Verify:**
```bash
git log --oneline -n 3
```

**Checkpoint:** airsprotocols pushed to remote

### Step 6.4: Review Changes in airsstack

```bash
cd /Users/hiraq/Projects/airsstack/airsstack

# Review changes
git status
git diff
```

**Expected Changes:**
- Modified: `README.md` (migration notice added)

**Checkpoint:** Only README.md changed

### Step 6.5: Commit to airsstack

**Stage changes:**
```bash
git add README.md
```

**Commit:**
```bash
git commit -m "docs: add MCP migration notice to README

Document the migration of the airs-mcp crate to the airsprotocols repository.

The airs-mcp crate has been migrated to airsprotocols as airsprotocols-mcp
for better organization of protocol implementations separate from applications.

Changes:
- Add migration notice section in README
- Link to new airsprotocols repository
- Explain transition period and dual maintenance

The airs-mcp crate remains in this repository during the transition period.
Both versions are maintained at v0.2.3.

New MCP location: https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp
Migration details: https://github.com/airsstack/airsprotocols/blob/main/MIGRATIONS.md"
```

**Checkpoint:** Changes committed

### Step 6.6: Push airsstack to Remote

```bash
git push origin main
```

**Verify:**
```bash
git log --oneline -n 3
```

**Checkpoint:** airsstack pushed to remote

**Deliverables:**
- [ ] airsprotocols changes reviewed and staged
- [ ] airsprotocols committed with detailed message
- [ ] airsprotocols pushed to remote
- [ ] airsstack changes reviewed
- [ ] airsstack README committed
- [ ] airsstack pushed to remote
- [ ] Both repositories synced with remotes

---

## ✅ PHASE 7: Final Verification

**Estimated Time:** 1-2 hours

### Step 7.1: Fresh Clone Test (Recommended)

**Create test directory:**
```bash
cd /tmp
mkdir migration-test
cd migration-test
```

**Clone both repositories:**
```bash
git clone https://github.com/airsstack/airsprotocols.git
git clone https://github.com/airsstack/airsstack.git
```

**Test airsprotocols:**
```bash
cd airsprotocols
cargo build --workspace --release
cargo test --workspace
cargo build --workspace --examples
```

**Expected:** All succeed

**Test airsstack:**
```bash
cd ../airsstack
cargo build --workspace --release
cargo test --workspace
```

**Expected:** All succeed

**Cleanup:**
```bash
cd /tmp
rm -rf migration-test
```

**Checkpoint:** Fresh clones build successfully

### Step 7.2: Comprehensive Verification Checklist

**airsprotocols Repository:**
- [ ] Repository accessible at https://github.com/airsstack/airsprotocols
- [ ] `protocols/mcp/` directory exists with full content
- [ ] `cargo build --workspace` passes (0 errors)
- [ ] `cargo test --workspace` passes (all tests)
- [ ] `cargo build --workspace --examples` succeeds
- [ ] `cargo doc --workspace --no-deps` generates docs
- [ ] All 5 integration examples compile
- [ ] All Python integration tests pass
- [ ] mdBook documentation builds (`mdbook build`)
- [ ] README.md has migration notice
- [ ] README.md code examples use `airsprotocols_mcp`
- [ ] MIGRATIONS.md marked as complete
- [ ] No `airs-mcp` references in source code (except migration notes)
- [ ] No `airs_mcp` references in Rust code (except migration notes)

**airsstack Repository:**
- [ ] Repository accessible at https://github.com/airsstack/airsstack
- [ ] `airs-mcp` crate still present in `crates/airs-mcp/`
- [ ] `airs-mcp` still in workspace members
- [ ] `airs-mcpserver-fs` still uses `airs-mcp` (not changed)
- [ ] `cargo build --workspace` passes
- [ ] `cargo test --workspace` passes
- [ ] `airs-mcpserver-fs` builds successfully
- [ ] `airs-mcpserver-fs` tests pass
- [ ] Binary `airs-mcpserver-fs` runs: `./target/release/airs-mcpserver-fs --version`
- [ ] README.md has migration notice
- [ ] Links to airsprotocols work

**Documentation:**
- [ ] All README files accurate
- [ ] Migration guide complete
- [ ] Code examples use correct crate names
- [ ] No broken links (test manually)
- [ ] Installation instructions correct

**Git:**
- [ ] Commits are descriptive and clear
- [ ] Remote branches synced
- [ ] No uncommitted changes in either repo

**Checkpoint:** All checklist items verified

### Step 7.3: Test Binary Functionality

```bash
cd /Users/hiraq/Projects/airsstack/airsstack/mcp-servers/airs-mcpserver-fs

# Build release binary
cargo build --release

# Test help
./target/release/airs-mcpserver-fs --help

# Test version
./target/release/airs-mcpserver-fs --version

# Test subcommands (if applicable)
./target/release/airs-mcpserver-fs setup --help
./target/release/airs-mcpserver-fs serve --help
```

**Expected:** All commands work correctly

**Checkpoint:** Binary functionality confirmed

### Step 7.4: Smoke Test Examples

**Test one example from each category:**

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# STDIO example
cargo run --example stdio-server-integration &
STDIO_PID=$!
sleep 2
kill $STDIO_PID || true

# HTTP API Key example (just verify it starts)
timeout 5 cargo run --example http-apikey-server-integration -- --help || true

# OAuth2 example (just verify it starts)
timeout 5 cargo run --example http-oauth2-server-integration -- --help || true
```

**Expected:** Examples start without panic/errors

**Checkpoint:** Example smoke tests pass

### Step 7.5: Document Migration Results

**Create Results Summary:**

```
=== MIGRATION COMPLETION REPORT ===
Date: [ACTUAL DATE]
Start Time: [START]
End Time: [END]
Total Duration: [HOURS]

Status: ✅ SUCCESS / ⚠️ PARTIAL / ❌ FAILED

Phase Results:
- Phase 0 (Pre-Migration): ✅
- Phase 1 (Setup): ✅
- Phase 2 (Copy & Adapt): ✅
- Phase 3 (Build & Test): ✅
- Phase 4 (Python Tests): ✅
- Phase 5 (Documentation): ✅
- Phase 6 (Git Operations): ✅
- Phase 7 (Verification): ✅

Build Results:
- airsprotocols builds: ✅ 0 errors, [N] warnings
- airsprotocols tests: ✅ [N] passed
- airsstack builds: ✅ 0 errors
- airsstack tests: ✅ [N] passed
- Integration tests: ✅ [N]/[N] passed

Issues Encountered:
- [None / List any issues]

Verification:
- Fresh clone builds: ✅
- Documentation generates: ✅
- All examples compile: ✅
- Binary functionality: ✅

Next Steps:
1. Monitor both repositories for issues
2. Update external documentation (if any)
3. Notify stakeholders/users (if applicable)
4. Plan timeline for eventual removal of airs-mcp from airsstack
```

**Save to:** `migration-results-[DATE].txt`

**Checkpoint:** Migration results documented

**Deliverables:**
- [ ] Fresh clone test passed (or documented as skipped)
- [ ] All verification checklist items complete
- [ ] Binary functionality tested
- [ ] Example smoke tests passed
- [ ] Migration completion report created
- [ ] No critical issues remaining

---

## 🎉 PHASE 8: Post-Migration Tasks

**Estimated Time:** 30 minutes

### Step 8.1: Update GitHub Repository Settings

**airsprotocols repository:**

Visit: https://github.com/airsstack/airsprotocols/settings

- [ ] **Description:** "Common AI Communication Protocols and LLM API Implementations in Rust"
- [ ] **Website:** (optional)
- [ ] **Topics:** `rust`, `mcp`, `model-context-protocol`, `protocols`, `ai`, `agents`
- [ ] **License:** MIT (verify license files present)

**airsstack repository:**

Visit: https://github.com/airsstack/airsstack/settings

- [ ] **Description:** Check if update needed to mention protocol migration
- [ ] **Topics:** Verify accuracy

**Checkpoint:** Repository settings updated

### Step 8.2: Create Migration Announcement (Optional)

**In airsprotocols:**

Create GitHub Discussion or Issue:
- **Title:** "MCP v0.2.3 Migration Complete"
- **Content:**
  ```markdown
  The Model Context Protocol (MCP) implementation has been successfully 
  migrated from the `airsstack` repository as `airsprotocols-mcp` v0.2.3.
  
  ## What Changed
  
  - **Crate Name:** `airs-mcp` → `airsprotocols-mcp`
  - **Repository:** Now in `airsprotocols` for better organization
  - **Version:** 0.2.3 (unchanged)
  - **Functionality:** All features preserved
  
  ## For Users
  
  See the [migration guide](MIGRATIONS.md) for updating your dependencies.
  
  ## Status
  
  - ✅ All builds pass
  - ✅ All tests pass (Rust + Python integration)
  - ✅ Documentation updated
  - ✅ Examples verified
  
  The original `airs-mcp` remains available in `airsstack` during transition.
  ```

**Checkpoint:** Announcement created (if desired)

### Step 8.3: Update Personal Documentation

**Document for future reference:**

```
Project: airsprotocols
Migration: airs-mcp → airsprotocols-mcp
Completed: [DATE]
Duration: [HOURS]

Key Locations:
- New MCP: /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
- Old MCP: /Users/hiraq/Projects/airsstack/airsstack/crates/airs-mcp (kept)
- Consumer: /Users/hiraq/Projects/airsstack/airsstack/mcp-servers/airs-mcpserver-fs

Status:
- airsprotocols: Primary development location
- airsstack: airs-mcp maintained during transition
- airs-mcpserver-fs: Still uses local airs-mcp

Future Actions:
- Monitor stability (estimated 2-4 weeks)
- Update airs-mcpserver-fs to use airsprotocols-mcp
- Eventually remove airs-mcp from airsstack
- Consider crates.io publication
- Consider creating git tags

Notes:
- [Any important observations]
```

**Checkpoint:** Personal docs updated

### Step 8.4: Monitor for Issues

**Set up monitoring:**

- [ ] Watch both GitHub repositories for issues
- [ ] Test builds periodically over next week
- [ ] Check for community feedback (if applicable)

**Create reminder:** Check migration status in 1 week

**Checkpoint:** Monitoring in place

**Deliverables:**
- [ ] Repository settings verified/updated
- [ ] Migration announcement created (optional)
- [ ] Personal documentation updated
- [ ] Monitoring plan in place
- [ ] Migration fully complete

---

## 🔄 Rollback Procedures

If critical issues are discovered during or after migration:

### During Migration (Before Git Push)

```bash
# Discard all changes in airsprotocols
cd /Users/hiraq/Projects/airsstack/airsprotocols
git reset --hard HEAD
git clean -fd

# Discard all changes in airsstack
cd /Users/hiraq/Projects/airsstack/airsstack
git reset --hard HEAD
git clean -fd
```

### After Git Push (Committed)

**Revert airsprotocols:**
```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols
git revert HEAD
git push origin main
```

**Revert airsstack:**
```bash
cd /Users/hiraq/Projects/airsstack/airsstack
git revert HEAD
git push origin main
```

### Partial Rollback (Keep airsprotocols, revert airsstack)

```bash
# Only revert airsstack README changes if needed
cd /Users/hiraq/Projects/airsstack/airsstack
git revert [commit-hash]
git push origin main
```

---

## 📊 Success Metrics

### Required for Success

- [ ] airsprotocols builds without errors
- [ ] All Rust tests pass in airsprotocols
- [ ] All Python integration tests pass
- [ ] All examples compile
- [ ] Documentation builds (cargo doc + mdBook)
- [ ] airsstack builds without changes (except README)
- [ ] airs-mcpserver-fs still functions correctly
- [ ] Git commits pushed to both repos

### Quality Indicators

- [ ] 0 compiler warnings (ideal, not required)
- [ ] 100% test pass rate
- [ ] All documentation links work
- [ ] Fresh clone builds successfully
- [ ] No user-facing breaking changes in airsstack

### Migration Completion Criteria

✅ Migration is COMPLETE when:

1. airsprotocols-mcp builds, tests, and runs independently
2. All integration tests pass
3. Documentation is accurate and updated
4. Both repositories pushed to GitHub
5. No regressions in airsstack
6. Verification checklist 100% complete

---

## 📞 Troubleshooting

### Common Issues

**Issue: Build errors after copying**
- **Cause:** Import paths not fully updated
- **Solution:** Run grep for remaining `airs_mcp` references:
  ```bash
  grep -r "airs_mcp" src/ tests/ examples/ benches/
  ```

**Issue: Python tests fail**
- **Cause:** Dependencies not installed or wrong Python version
- **Solution:** Verify Python 3.9+, reinstall requirements:
  ```bash
  python3 --version
  pip3 install --upgrade -r requirements.txt
  ```

**Issue: Examples don't compile**
- **Cause:** Cargo.toml not updated in example subdirectories
- **Solution:** Check each example's Cargo.toml:
  ```bash
  find examples -name "Cargo.toml" -exec grep "airs-mcp" {} + -print
  ```

**Issue: mdBook build fails**
- **Cause:** mdBook not installed or docs have issues
- **Solution:** 
  ```bash
  cargo install mdbook
  cd docs && mdbook build --verbose
  ```

**Issue: Tests pass locally but fail in fresh clone**
- **Cause:** Uncommitted files or local configuration
- **Solution:** Check git status, ensure all necessary files committed

### Getting Help

If issues persist:
1. Review this action plan step-by-step
2. Check the original MIGRATIONS.md for context
3. Review git diff to see what changed
4. Test in isolation (fresh clone to /tmp)
5. Document the issue for troubleshooting

---

## 📋 Quick Reference

### Key Paths

```
Source:
/Users/hiraq/Projects/airsstack/airsstack/crates/airs-mcp/

Destination:
/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp/

Consumer (unchanged):
/Users/hiraq/Projects/airsstack/airsstack/mcp-servers/airs-mcpserver-fs/
```

### Key Commands

```bash
# Build workspace
cargo build --workspace

# Test workspace
cargo test --workspace

# Build examples
cargo build --workspace --examples

# Generate docs
cargo doc --workspace --no-deps

# Build mdBook
cd docs && mdbook build

# Run Python tests
cd examples/*/tests && pip3 install -r requirements.txt && python3 test_*.py
```

### Important Files to Update

- `protocols/mcp/Cargo.toml` - Package name and URLs
- `protocols/mcp/README.md` - Migration notice
- `protocols/mcp/CHANGELOG.md` - Migration entry
- `protocols/mcp/examples/*/Cargo.toml` - Dependency names
- All `*.rs` files - Import statements
- `airsprotocols/README.md` - Status update
- `airsprotocols/MIGRATIONS.md` - Completion status
- `airsstack/README.md` - Migration notice

---

## ✅ Final Checklist

Before considering migration complete:

### Pre-Migration
- [ ] Backups created
- [ ] Both repos in clean state
- [ ] All current builds pass

### Migration Execution
- [ ] Phase 0: Pre-Migration ✅
- [ ] Phase 1: Setup ✅
- [ ] Phase 2: Copy & Adapt ✅
- [ ] Phase 3: Build & Test ✅
- [ ] Phase 4: Python Tests ✅
- [ ] Phase 5: Documentation ✅
- [ ] Phase 6: Git Operations ✅
- [ ] Phase 7: Verification ✅
- [ ] Phase 8: Post-Migration ✅

### Verification
- [ ] airsprotocols builds and tests pass
- [ ] airsstack builds and tests pass (unchanged)
- [ ] All Python integration tests pass
- [ ] Documentation accurate
- [ ] Git pushed successfully
- [ ] Fresh clone test passed

### Documentation
- [ ] Migration completion report created
- [ ] Issues documented (if any)
- [ ] Personal notes updated
- [ ] Monitoring plan in place

---

**Migration Action Plan Complete**

This document provides step-by-step instructions for migrating `airs-mcp` 
to `airsprotocols-mcp`. Follow each phase sequentially, checking off items 
as you complete them. If issues arise, consult the Troubleshooting section 
or Rollback Procedures.

Good luck! 🚀
