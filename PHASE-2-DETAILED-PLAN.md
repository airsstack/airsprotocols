# Phase 2: Copy and Adapt - Detailed Action Plan

**Status:** ⏳ READY TO START  
**Estimated Time:** 2-3 hours  
**Complexity:** High  
**Risk Level:** Medium  
**Purpose:** Copy airs-mcp source code to airsprotocols and adapt it to the new package name and repository structure

---

## 📋 Overview

Phase 2 is the core migration phase where we actually copy the source code from `airsstack/crates/airs-mcp/` to `airsprotocols/protocols/mcp/` and perform all necessary adaptations to make it work as `airsprotocols-mcp`.

**What this phase does:**
- Copies all source files (with appropriate exclusions)
- Renames the package from `airs-mcp` to `airsprotocols-mcp`
- Updates all Rust imports throughout the codebase
- Updates example projects to use the new package name
- Updates all documentation and markdown files
- Adds migration notices to key files

**What this phase does NOT do:**
- Does not build or test (that's Phase 3)
- Does not modify `airsstack` repository
- Does not remove files from `airsstack`
- Does not publish to crates.io

---

## 🎯 Phase 2 Objectives

1. ✅ Copy all source files from airsstack to airsprotocols (excluding build artifacts)
2. ✅ Update package name in Cargo.toml
3. ✅ Update all example projects' Cargo.toml files
4. ✅ Update all Rust imports (use statements)
5. ✅ Update all documentation with new package name
6. ✅ Add migration notices to README and CHANGELOG
7. ✅ Verify no old references remain

---

## 📝 Detailed Steps

### **Step 2.1: Copy Source Code with Exclusions**

**Time:** 10-15 minutes  
**Risk Level:** Low  
**Goal:** Copy all source files while excluding build artifacts and temporary files

#### Pre-Copy Verification:

1. **Verify source directory exists:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsstack
   ls -la crates/airs-mcp/
   ```
   
   **Expected:** Should show Cargo.toml, src/, examples/, tests/, etc.

2. **Count source files to copy:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsstack/crates/airs-mcp
   find . -type f \
     ! -path "*/target/*" \
     ! -path "*/.git/*" \
     ! -path "*/__pycache__/*" \
     ! -path "*/.pytest_cache/*" \
     ! -name "*.pyc" \
     ! -name "Cargo.lock" \
     ! -name ".DS_Store" \
     | wc -l
   ```
   
   **Expected:** Around 200-300 files

3. **Verify destination directory is empty:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
   ls -la
   ```
   
   **Expected:** Empty directory (just . and ..)

#### Copy Operation:

```bash
cd /Users/hiraq/Projects/airsstack/airsstack

# Use rsync with exclusions
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
  --exclude='.git' \
  crates/airs-mcp/ \
  ../airsprotocols/protocols/mcp/
```

**Alternative (if rsync not preferred):**
```bash
cd /Users/hiraq/Projects/airsstack/airsstack
cp -r crates/airs-mcp/* ../airsprotocols/protocols/mcp/

# Then clean up unwanted files
cd ../airsprotocols/protocols/mcp
rm -rf target/
rm -f Cargo.lock
find . -name ".DS_Store" -delete
find . -name "*.pyc" -delete
find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find . -type d -name ".pytest_cache" -exec rm -rf {} + 2>/dev/null
```

#### Post-Copy Verification:

1. **Verify core files exist:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
   ls -la
   ```
   
   **Expected files:**
   - Cargo.toml ✅
   - README.md ✅
   - CHANGELOG.md ✅
   - src/ ✅
   - examples/ ✅
   - tests/ ✅
   - benches/ ✅
   - docs/ ✅

2. **Count copied files:**
   ```bash
   find . -type f | wc -l
   ```
   
   **Expected:** Around 200-300 files (should match source count)

3. **Check total size:**
   ```bash
   du -sh .
   ```
   
   **Expected:** 1-10 MB (without target/ directory)

4. **Verify no target/ directory:**
   ```bash
   ls -la | grep target || echo "✅ No target directory"
   ```

5. **Verify Cargo.toml is readable:**
   ```bash
   head -20 Cargo.toml
   ```
   
   **Expected:** Should show `[package]` section with `name = "airs-mcp"`

#### Expected Outcome:
- ✅ All source files copied successfully
- ✅ No build artifacts (target/) copied
- ✅ File count matches source
- ✅ All directories preserved (src/, tests/, examples/, etc.)
- ✅ No permission issues

#### What Could Go Wrong:
- **Rsync not available:** Use cp alternative method
- **Permission denied:** Check directory permissions
- **Unexpected files copied:** Review exclusion patterns
- **Missing files:** Check source directory structure

#### Rollback:
```bash
# If copy goes wrong:
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
rm -rf *
rm -rf .??*  # Remove hidden files
```

**Checkpoint:**
- [ ] All source files copied to protocols/mcp/
- [ ] File count matches source (~200-300 files)
- [ ] No target/ directory present
- [ ] Cargo.toml exists and readable
- [ ] src/, tests/, examples/ directories present

---

### **Step 2.2: Update Package Metadata in Cargo.toml**

**Time:** 10 minutes  
**Risk Level:** Low  
**Goal:** Update the main package Cargo.toml with new name and repository information

#### Actions:

1. **Read current Cargo.toml:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
   cat Cargo.toml | head -30
   ```

2. **Update package name (around line 2):**
   
   **FIND:**
   ```toml
   [package]
   name = "airs-mcp"
   ```
   
   **REPLACE WITH:**
   ```toml
   [package]
   name = "airsprotocols-mcp"
   ```

3. **Update documentation URL (around line 14):**
   
   **FIND:**
   ```toml
   documentation = "https://docs.rs/airs-mcp"
   ```
   
   **REPLACE WITH:**
   ```toml
   documentation = "https://docs.rs/airsprotocols-mcp"
   ```

4. **Update homepage URL (around line 15):**
   
   **FIND:**
   ```toml
   homepage = "https://github.com/airsstack/airsstack/tree/main/crates/airs-mcp"
   ```
   
   **REPLACE WITH:**
   ```toml
   homepage = "https://github.com/airsstack/airsprotocols/tree/main/protocols/mcp"
   ```

5. **Update repository URL (if present):**
   
   **FIND:**
   ```toml
   repository = "https://github.com/airsstack/airsstack"
   ```
   
   **REPLACE WITH:**
   ```toml
   repository = "https://github.com/airsstack/airsprotocols"
   ```

6. **Verify workspace inheritance remains intact:**
   
   These lines should remain UNCHANGED:
   ```toml
   version.workspace = true
   edition.workspace = true
   authors.workspace = true
   license.workspace = true
   repository.workspace = true  # If using workspace inheritance
   rust-version.workspace = true
   ```

7. **Verify no internal airs-mcp references:**
   ```bash
   grep -n "airs-mcp" Cargo.toml | grep -v "airsprotocols-mcp"
   ```
   
   **Expected:** No results (or only in comments explaining migration)

#### Verification:

```bash
# Check package name
grep "^name = " Cargo.toml
# Should show: name = "airsprotocols-mcp"

# Check URLs
grep "documentation\|homepage\|repository" Cargo.toml | head -5
# Should show new airsprotocols URLs

# Validate TOML syntax
cargo metadata --format-version 1 --no-deps 2>&1 | head -10
# Should show package metadata or expected "manifest" error (we haven't updated imports yet)
```

#### Expected Outcome:
- ✅ Package name is `airsprotocols-mcp`
- ✅ All URLs point to airsprotocols repository
- ✅ Workspace inheritance preserved
- ✅ TOML syntax is valid
- ✅ No old `airs-mcp` references (except in comments)

**Checkpoint:**
- [ ] Package name updated to "airsprotocols-mcp"
- [ ] Documentation URL updated
- [ ] Homepage URL updated
- [ ] Repository URL updated
- [ ] Workspace inheritance intact
- [ ] TOML syntax valid

---

### **Step 2.3: Update Example Projects' Cargo.toml Files**

**Time:** 15-20 minutes  
**Risk Level:** Low  
**Goal:** Update all example project Cargo.toml files to use the new package name

#### Discover Example Projects:

1. **Find all example Cargo.toml files:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
   find examples -name "Cargo.toml" -type f
   ```
   
   **Expected examples:**
   - `examples/http-apikey-client-integration/Cargo.toml`
   - `examples/http-apikey-server-integration/Cargo.toml` (if exists)
   - `examples/http-oauth2-client-integration/Cargo.toml`
   - `examples/http-oauth2-server-integration/Cargo.toml`
   - `examples/stdio-server-integration/Cargo.toml` (if exists)
   - Plus any others

2. **Count example projects:**
   ```bash
   find examples -name "Cargo.toml" | wc -l
   ```
   
   **Expected:** 4-8 example projects

#### Update Each Example:

For each Cargo.toml found in examples/, update the dependency:

**FIND:**
```toml
[dependencies]
airs-mcp = { path = "../../" }
# OR
airs-mcp = { path = "../../", features = [...] }
```

**REPLACE WITH:**
```toml
[dependencies]
airsprotocols-mcp = { path = "../../" }
# OR
airsprotocols-mcp = { path = "../../", features = [...] }
```

#### Automated Update Method:

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# Update all example Cargo.toml files
find examples -name "Cargo.toml" -type f -exec sed -i.bak 's/airs-mcp/airsprotocols-mcp/g' {} +

# Remove backup files
find examples -name "*.bak" -delete
```

#### Manual Verification:

For safety, manually check each file:

```bash
# List all example Cargo.toml files
find examples -name "Cargo.toml" -type f -print0 | while IFS= read -r -d '' file; do
  echo "=== $file ==="
  grep -A 2 "\[dependencies\]" "$file" | head -5
  echo ""
done
```

**Expected:** Each should show `airsprotocols-mcp = { path = "../../" }`

#### Verification:

1. **Check for remaining old references:**
   ```bash
   find examples -name "Cargo.toml" -type f -exec grep -l "airs-mcp" {} \;
   ```
   
   **Expected:** No results (empty output)

2. **Confirm new references exist:**
   ```bash
   find examples -name "Cargo.toml" -type f -exec grep -l "airsprotocols-mcp" {} \;
   ```
   
   **Expected:** Lists all example Cargo.toml files

3. **Verify path references are correct:**
   ```bash
   find examples -name "Cargo.toml" -exec grep "path = " {} \;
   ```
   
   **Expected:** All paths should be `path = "../../"` (relative to example location)

#### Expected Outcome:
- ✅ All example Cargo.toml files updated
- ✅ Dependency name changed to `airsprotocols-mcp`
- ✅ Path references remain correct (`../../`)
- ✅ No old `airs-mcp` references in example Cargo.toml files
- ✅ TOML syntax remains valid

**Checkpoint:**
- [ ] All example Cargo.toml files found
- [ ] Dependency name updated in each file
- [ ] Path references verified correct
- [ ] No remaining "airs-mcp" in example Cargo.toml files
- [ ] Backup files cleaned up

---

### **Step 2.4: Update Source Code Imports**

**Time:** 20-30 minutes  
**Risk Level:** Medium  
**Goal:** Update all Rust `use` statements from `airs_mcp` to `airsprotocols_mcp`

#### Pre-Update Analysis:

1. **Count occurrences to update:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
   
   # In src/
   grep -r "use airs_mcp" src/ | wc -l
   
   # In tests/
   grep -r "use airs_mcp" tests/ 2>/dev/null | wc -l || echo "0"
   
   # In examples/
   grep -r "use airs_mcp" examples/ | wc -l
   
   # In benches/
   grep -r "use airs_mcp" benches/ 2>/dev/null | wc -l || echo "0"
   ```
   
   **Expected:** Dozens to hundreds of occurrences

2. **Sample current imports:**
   ```bash
   grep -r "use airs_mcp" src/ | head -10
   ```
   
   **Expected format:**
   ```rust
   use airs_mcp::McpClient;
   use airs_mcp::types::*;
   use airs_mcp::{Error, Result};
   ```

#### Update Operations:

**Update src/ directory:**
```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# Update all .rs files in src/
find src -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} +
find src -name "*.rs" -type f -exec sed -i '' 's/extern crate airs_mcp/extern crate airsprotocols_mcp/g' {} +

# Clean backup files
find src -name "*.bak" -delete
```

**Update tests/ directory:**
```bash
# Update all .rs files in tests/ (if directory exists)
if [ -d tests ]; then
  find tests -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} +
  find tests -name "*.rs" -type f -exec sed -i '' 's/extern crate airs_mcp/extern crate airsprotocols_mcp/g' {} +
  find tests -name "*.bak" -delete
fi
```

**Update examples/ directory:**
```bash
# Update all .rs files in examples/
find examples -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} +
find examples -name "*.rs" -type f -exec sed -i '' 's/extern crate airs_mcp/extern crate airsprotocols_mcp/g' {} +

# Clean backup files
find examples -name "*.bak" -delete
```

**Update benches/ directory:**
```bash
# Update all .rs files in benches/ (if directory exists)
if [ -d benches ]; then
  find benches -name "*.rs" -type f -exec sed -i.bak 's/use airs_mcp/use airsprotocols_mcp/g' {} +
  find benches -name "*.rs" -type f -exec sed -i '' 's/extern crate airs_mcp/extern crate airsprotocols_mcp/g' {} +
  find benches -name "*.bak" -delete
fi
```

#### Comprehensive Verification:

1. **Check for remaining old imports:**
   ```bash
   # Should return NO results:
   grep -r "use airs_mcp" src/ tests/ examples/ benches/ 2>/dev/null || echo "✅ All imports updated"
   ```

2. **Verify new imports exist:**
   ```bash
   # Should return results:
   grep -r "use airsprotocols_mcp" src/ | head -5
   ```
   
   **Expected:**
   ```rust
   use airsprotocols_mcp::McpClient;
   use airsprotocols_mcp::types::*;
   ```

3. **Check for old crate name in comments/docs:**
   ```bash
   # This will show places in comments - manual review needed
   grep -r "airs-mcp" src/ --include="*.rs" | grep -v "airsprotocols-mcp"
   ```
   
   **Action:** Review results and update documentation comments if needed

4. **Check for old crate name in macro definitions:**
   ```bash
   grep -r "airs_mcp!" src/ 2>/dev/null || echo "✅ No macro issues"
   ```

5. **Verify no old extern crate:**
   ```bash
   grep -r "extern crate airs_mcp" src/ tests/ examples/ benches/ 2>/dev/null || echo "✅ No extern crate issues"
   ```

#### Sample Files to Manually Verify:

After automated updates, spot-check these key files:

```bash
# Check main lib.rs
head -30 src/lib.rs

# Check a module file
head -20 src/client/mod.rs  # Adjust path as needed

# Check an example
head -20 examples/*/src/main.rs | head -40  # Check first example
```

**Look for:**
- ✅ `use airsprotocols_mcp::...` (correct)
- ❌ `use airs_mcp::...` (wrong - needs fixing)

#### Expected Outcome:
- ✅ All `use airs_mcp` changed to `use airsprotocols_mcp`
- ✅ All `extern crate airs_mcp` changed (if any exist)
- ✅ No old imports remaining in .rs files
- ✅ New imports verified working (will test in Phase 3)
- ✅ Documentation comments updated (where appropriate)

#### What Could Go Wrong:
- **Sed syntax differs on macOS:** Use `sed -i ''` instead of `sed -i` on macOS
- **Some imports missed:** Run manual grep to find any stragglers
- **Code in strings:** Review any matches in string literals (probably okay)
- **Macro expansions:** Check for macro-generated code with old names

**Checkpoint:**
- [ ] src/ directory imports updated
- [ ] tests/ directory imports updated
- [ ] examples/ directory imports updated
- [ ] benches/ directory imports updated
- [ ] No "use airs_mcp" found in any .rs files
- [ ] Sample files manually verified
- [ ] Backup files cleaned up

---

### **Step 2.5: Update Documentation Files**

**Time:** 20-30 minutes  
**Risk Level:** Low  
**Goal:** Update all markdown files and add migration notices

#### Update All Markdown Files (Automated):

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# Update package name references
find . -name "*.md" -type f -exec sed -i.bak 's/airs-mcp/airsprotocols-mcp/g' {} +

# Update crate name references (underscore version)
find . -name "*.md" -type f -exec sed -i '' 's/airs_mcp/airsprotocols_mcp/g' {} +

# Update repository URLs
find . -name "*.md" -type f -exec sed -i '' 's|github.com/airsstack/airsstack/tree/main/crates/airs-mcp|github.com/airsstack/airsprotocols/tree/main/protocols/mcp|g' {} +

# Clean backup files
find . -name "*.bak" -delete
```

#### Manual README.md Update:

1. **Read current README header:**
   ```bash
   head -30 README.md
   ```

2. **Add migration notice after the title:**
   
   Find the main title (line 1-3), then add this section:
   
   ```markdown
   # airsprotocols-mcp
   
   > **📦 Migration Notice:** This crate was previously published as `airs-mcp` in the 
   > [airsstack](https://github.com/airsstack/airsstack) repository. It has been migrated 
   > to the `airsprotocols` repository as of December 2025 for better organization of 
   > protocol implementations.
   >
   > **Original Source:** https://github.com/airsstack/airsstack/tree/main/crates/airs-mcp
   
   ## Migrating from `airs-mcp` to `airsprotocols-mcp`
   
   If you're updating from the old `airs-mcp` crate, follow these steps:
   
   ### 1. Update your `Cargo.toml`:
   
   ```toml
   # Before (old)
   [dependencies]
   airs-mcp = "0.2.3"
   
   # After (new)
   [dependencies]
   airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
   ```
   
   ### 2. Update your imports:
   
   ```rust
   // Before (old)
   use airs_mcp::client::McpClient;
   use airs_mcp::types::*;
   
   // After (new)
   use airsprotocols_mcp::client::McpClient;
   use airsprotocols_mcp::types::*;
   ```
   
   ### 3. Update any documentation or examples:
   
   Replace all references to `airs-mcp` with `airsprotocols-mcp` in your project 
   documentation, examples, and comments.
   
   ---
   
   [Rest of existing README content follows...]
   ```

3. **Update installation instructions:**
   
   Find the installation section and update:
   
   ```markdown
   ## Installation
   
   Add this to your `Cargo.toml`:
   
   ```toml
   [dependencies]
   airsprotocols-mcp = { git = "https://github.com/airsstack/airsprotocols" }
   ```
   
   Or use a specific version when published:
   
   ```toml
   [dependencies]
   airsprotocols-mcp = "0.2.3"
   ```
   ```

#### Manual CHANGELOG.md Update:

1. **Read current changelog:**
   ```bash
   head -20 CHANGELOG.md
   ```

2. **Add migration entry at the top:**
   
   ```markdown
   # Changelog
   
   All notable changes to this project will be documented in this file.
   
   The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
   and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
   
   ## [Unreleased]
   
   ### Changed
   
   - **BREAKING MIGRATION:** Crate renamed from `airs-mcp` to `airsprotocols-mcp`
   - **BREAKING MIGRATION:** Moved from `airsstack` repository to `airsprotocols` repository
   - **Repository:** https://github.com/airsstack/airsprotocols
   - **Original Source:** https://github.com/airsstack/airsstack/tree/main/crates/airs-mcp
   - All imports must be updated from `use airs_mcp::` to `use airsprotocols_mcp::`
   - All Cargo.toml dependencies must be updated from `airs-mcp` to `airsprotocols-mcp`
   
   ### Migration Guide
   
   See README.md for detailed migration instructions.
   
   ## [0.2.3] - 2024-XX-XX
   
   [Previous changelog entries continue below...]
   ```

#### Update docs/ Directory (if exists):

```bash
# Check if docs/ exists and has markdown files
if [ -d docs ]; then
  find docs -name "*.md" -type f | wc -l
  
  # Update all docs
  find docs -name "*.md" -type f -exec sed -i.bak 's/airs-mcp/airsprotocols-mcp/g' {} +
  find docs -name "*.md" -type f -exec sed -i '' 's/airs_mcp/airsprotocols_mcp/g' {} +
  find docs -name "*.bak" -delete
fi
```

#### Verification:

1. **Check for remaining old references:**
   ```bash
   # Look for old references (excluding intentional migration notices)
   grep -r "airs-mcp" . --include="*.md" \
     | grep -v "airsprotocols-mcp" \
     | grep -v "was airs-mcp" \
     | grep -v "previously.*airs-mcp" \
     | grep -v "renamed from.*airs-mcp" \
     | grep -v "Original Source"
   ```
   
   **Expected:** Very few or no results (only legitimate historical references)

2. **Verify README has migration notice:**
   ```bash
   grep -A 3 "Migration Notice" README.md
   ```
   
   **Expected:** Shows the migration notice section

3. **Verify CHANGELOG has migration entry:**
   ```bash
   head -30 CHANGELOG.md | grep -i "migration"
   ```
   
   **Expected:** Shows migration entries

4. **Check code examples in documentation:**
   ```bash
   # Look for code blocks with old crate name
   grep -B 2 -A 2 "use airs_mcp" *.md docs/*.md 2>/dev/null | grep -v "Before\|Old"
   ```
   
   **Expected:** Only in "before/after" migration examples

#### Expected Outcome:
- ✅ All markdown files updated with new crate name
- ✅ README.md has migration notice at top
- ✅ CHANGELOG.md has migration entry
- ✅ Code examples in docs use new name
- ✅ Old references only in migration instructions
- ✅ Repository URLs updated

**Checkpoint:**
- [ ] All .md files processed
- [ ] README.md has migration notice
- [ ] CHANGELOG.md updated
- [ ] Installation instructions updated
- [ ] Code examples in docs updated
- [ ] docs/ directory updated (if exists)
- [ ] Minimal old references remain (only in migration context)

---

### **Step 2.6: Final Verification and Documentation**

**Time:** 10-15 minutes  
**Risk Level:** None (verification only)  
**Goal:** Comprehensive verification that all adaptations are complete

#### Comprehensive Search for Old References:

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# Search all text files for "airs-mcp" (hyphen version)
grep -r "airs-mcp" . \
  --include="*.rs" \
  --include="*.toml" \
  --include="*.md" \
  --include="*.txt" \
  --exclude-dir="target" \
  | grep -v "airsprotocols-mcp" \
  | grep -v "was airs-mcp" \
  | grep -v "previously.*airs-mcp" \
  | grep -v "renamed from" \
  | grep -v "migrat" \
  || echo "✅ No problematic airs-mcp references"
```

```bash
# Search all text files for "airs_mcp" (underscore version)
grep -r "airs_mcp" . \
  --include="*.rs" \
  --include="*.toml" \
  --include="*.md" \
  --exclude-dir="target" \
  | grep -v "airsprotocols_mcp" \
  | grep -v "was airs_mcp" \
  | grep -v "previously" \
  | grep -v "migrat" \
  || echo "✅ No problematic airs_mcp references"
```

#### File-by-File Verification:

```bash
# 1. Main Cargo.toml
echo "=== Main Cargo.toml ==="
grep "^name = " Cargo.toml
grep "documentation\|homepage\|repository" Cargo.toml | head -3
echo ""

# 2. Example Cargo.toml files
echo "=== Example Cargo.toml files ==="
find examples -name "Cargo.toml" -exec grep -H "airsprotocols-mcp" {} \; | head -5
echo ""

# 3. Source files sample
echo "=== Sample source imports ==="
grep -h "use airsprotocols_mcp" src/*.rs src/*/*.rs 2>/dev/null | head -5
echo ""

# 4. README.md
echo "=== README.md Migration Notice ==="
head -20 README.md | grep -A 2 "Migration"
echo ""

# 5. CHANGELOG.md
echo "=== CHANGELOG.md Migration Entry ==="
head -20 CHANGELOG.md | grep -i "migration\|renamed"
echo ""
```

#### Count Changes:

```bash
# Count files with new crate name
echo "Files referencing airsprotocols-mcp:"
grep -r "airsprotocols-mcp" . --include="*.rs" --include="*.toml" --include="*.md" | wc -l

echo "Files referencing airsprotocols_mcp:"
grep -r "airsprotocols_mcp" . --include="*.rs" | wc -l
```

#### Generate Change Summary:

```bash
# Create a summary file
cat > PHASE-2-CHANGES-SUMMARY.txt << 'EOF'
Phase 2 Changes Summary
=======================
Date: $(date)
Location: protocols/mcp/

Files Copied: ~200-300 files from airsstack/crates/airs-mcp/

Changes Made:
=============

1. Package Metadata (Cargo.toml):
   - Package name: airs-mcp → airsprotocols-mcp
   - Documentation URL: updated to docs.rs/airsprotocols-mcp
   - Homepage URL: updated to airsprotocols repo
   - Repository URL: updated to airsprotocols repo

2. Example Projects:
   - Updated all example Cargo.toml dependency references
   - Count: $(find examples -name "Cargo.toml" | wc -l) example projects

3. Source Code Imports:
   - Updated all use statements: airs_mcp → airsprotocols_mcp
   - Directories updated: src/, tests/, examples/, benches/
   - Approximate import updates: $(grep -r "use airsprotocols_mcp" src/ | wc -l) in src/

4. Documentation:
   - Updated all markdown files
   - Added migration notice to README.md
   - Added migration entry to CHANGELOG.md
   - Updated code examples in documentation

Verification:
=============

Old references remaining:
  "airs-mcp" (hyphen): $(grep -r "airs-mcp" . --include="*.rs" --include="*.toml" --include="*.md" 2>/dev/null | grep -v "airsprotocols-mcp" | grep -v "migrat" | wc -l)
  "airs_mcp" (underscore): $(grep -r "airs_mcp" . --include="*.rs" 2>/dev/null | grep -v "airsprotocols_mcp" | grep -v "migrat" | wc -l)

New references added:
  "airsprotocols-mcp": $(grep -r "airsprotocols-mcp" . --include="*.rs" --include="*.toml" --include="*.md" 2>/dev/null | wc -l)
  "airsprotocols_mcp": $(grep -r "airsprotocols_mcp" . --include="*.rs" 2>/dev/null | wc -l)

Status: Phase 2 Complete - Ready for Phase 3 (Build Verification)
EOF

cat PHASE-2-CHANGES-SUMMARY.txt
```

#### Update Progress Documentation:

Update the migration progress log with Phase 2 completion:

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Note: This will be done interactively during execution
echo "Remember to update:"
echo "- MIGRATION-PROGRESS-LOG.md"
echo "- MIGRATION-ACTION-PLAN.md"
echo "- MIGRATION-BASELINE-STATE.txt"
```

#### Expected Outcome:
- ✅ No unwanted old references found
- ✅ All key files verified correct
- ✅ Change summary generated
- ✅ Ready for Phase 3 (build testing)

**Checkpoint:**
- [ ] Comprehensive search completed
- [ ] No problematic old references
- [ ] Key files spot-checked
- [ ] Change summary generated
- [ ] Progress documentation noted for update

---

## 📊 Phase 2 Deliverables

By the end of Phase 2, you will have:

### File Structure:
```
airsprotocols/protocols/mcp/
├── Cargo.toml                  # ✅ Package name: airsprotocols-mcp
├── README.md                   # ✅ Migration notice added
├── CHANGELOG.md                # ✅ Migration entry added
├── src/                        # ✅ All imports updated
├── tests/                      # ✅ All imports updated
├── examples/                   # ✅ All imports and Cargo.toml updated
├── benches/                    # ✅ All imports updated
├── docs/                       # ✅ All documentation updated
└── PHASE-2-CHANGES-SUMMARY.txt # ✅ Summary of all changes
```

### Changes Complete:
- ✅ ~200-300 source files copied
- ✅ Package name changed to `airsprotocols-mcp`
- ✅ All repository URLs updated
- ✅ All example Cargo.toml files updated
- ✅ All Rust imports updated (use statements)
- ✅ All documentation files updated
- ✅ Migration notices added to README and CHANGELOG
- ✅ No remaining problematic old references

### Verification:
- ✅ Comprehensive grep searches passed
- ✅ Key files manually verified
- ✅ Change summary documented
- ✅ Ready for build testing (Phase 3)

---

## ✅ Phase 2 Completion Checklist

Before proceeding to Phase 3, verify:

### File Copy:
- [ ] All source files copied from airsstack
- [ ] File count matches source (~200-300)
- [ ] No build artifacts copied (no target/)
- [ ] All key directories present (src/, tests/, examples/)

### Package Metadata:
- [ ] Package name is "airsprotocols-mcp"
- [ ] Documentation URL updated
- [ ] Homepage URL updated
- [ ] Repository URL updated
- [ ] Workspace inheritance intact

### Example Projects:
- [ ] All example Cargo.toml files found
- [ ] All dependencies updated to airsprotocols-mcp
- [ ] No "airs-mcp" in example Cargo.toml files

### Source Code:
- [ ] All src/ imports updated
- [ ] All tests/ imports updated
- [ ] All examples/ imports updated
- [ ] All benches/ imports updated
- [ ] No "use airs_mcp" in any .rs files

### Documentation:
- [ ] All .md files updated
- [ ] README.md has migration notice
- [ ] CHANGELOG.md has migration entry
- [ ] Installation instructions updated
- [ ] Code examples use new crate name

### Final Verification:
- [ ] Comprehensive grep search passed
- [ ] No unwanted old references remain
- [ ] Change summary generated
- [ ] Progress logs ready for update

---

## 🔄 Rollback Procedure

If Phase 2 needs to be rolled back:

```bash
# Complete rollback - remove all copied files
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp
rm -rf *
rm -rf .??*  # Remove hidden files

# Verify empty
ls -la
# Should only show . and ..

# Or restore from Phase 1 state (empty directory)
# The directory structure remains for Phase 2 retry
```

**Phase 2 Characteristics:**
- Only modifies airsprotocols repository
- No changes to airsstack repository
- Fully reversible (just delete copied files)
- No builds run (Phase 3 handles that)

---

## 🎯 Success Criteria

Phase 2 is successfully complete when:

1. ✅ All source files copied to protocols/mcp/
2. ✅ Package name is `airsprotocols-mcp` in Cargo.toml
3. ✅ All repository URLs point to airsprotocols
4. ✅ All example Cargo.toml files use new crate name
5. ✅ All Rust imports use `airsprotocols_mcp`
6. ✅ All documentation updated with new name
7. ✅ Migration notices in README and CHANGELOG
8. ✅ No unwanted old references remain
9. ✅ Ready for build testing (Phase 3)

**Target Time:** 2-3 hours  
**Typical Time:** 1.5-2.5 hours with automation  
**Complexity:** High  
**Risk:** Medium (but fully reversible)

---

## ⚠️ Important Notes

### About String Replacements:

- The automated `sed` commands are safe for this codebase
- They only change imports and package names
- They don't affect string literals or comments in harmful ways
- Always review the grep results after bulk changes

### About Backup Files:

- `sed -i.bak` creates .bak backup files
- We delete them after verification
- If you want to keep them for safety, skip the `find .bak -delete` commands
- They don't affect functionality but will be included in git

### About Migration Notices:

- The migration notice in README is important for users
- The CHANGELOG entry documents the breaking change
- These should mention both repositories for transparency
- Include original source URL for historical reference

---

## 📈 Progress After Phase 2

**Phases Complete:** 3/8 (37.5%)

- ✅ Phase 0: Pre-Migration Verification (20 min)
- ✅ Phase 1: Repository Structure Setup (6 min)
- ✅ Phase 2: Copy and Adapt (2-3 hours) ← COMPLETING
- ⏳ Phase 3: Build and Test Verification (3-4 hours) ← NEXT
- ⏳ Phase 4: Python Integration Tests (3-4 hours)
- ⏳ Phase 5: Documentation Updates (1-2 hours)
- ⏳ Phase 6: Git Operations (30 min)
- ⏳ Phase 7: Final Verification (1-2 hours)
- ⏳ Phase 8: Post-Migration Tasks (30 min)

**Estimated Remaining:** 10-13 hours

---

## 🚀 What's Next: Phase 3 Preview

After Phase 2 is complete and verified, Phase 3 will:

1. **Clean build** the entire airsprotocols workspace
2. **Run all tests** (unit, integration, doc tests)
3. **Build all examples** to verify they compile
4. **Generate documentation** to verify doc links work
5. **Address any build issues** that arise

Phase 3 is where we verify that all the changes from Phase 2 actually work correctly.

---

## 💡 Tips for Phase 2

1. **Use automation:** The sed commands save significant time
2. **Verify incrementally:** Check results after each step
3. **Keep backups:** The .bak files are your safety net
4. **Review key files:** Spot-check important files manually
5. **Document issues:** Note any unexpected findings
6. **Take breaks:** Phase 2 is long, take breaks between steps

---

## 📞 Troubleshooting

### Common Issues:

**1. Rsync not found:**
- Solution: Use the `cp -r` alternative method provided

**2. Sed syntax errors on macOS:**
- Solution: Use `sed -i ''` instead of `sed -i`

**3. Too many old references found:**
- Review each occurrence
- Some in comments/docs are okay
- Update migration notices to exclude them from grep

**4. File count mismatch:**
- Check for hidden files (`.??*`)
- Verify exclusion patterns
- Compare directory structures

**5. Permission denied:**
- Check directory ownership: `ls -la`
- Verify write permissions: `touch test && rm test`

---

**Ready to execute Phase 2 when you give the command.**

**Execution Mode:** Step-by-step with approval after each step (2.1 through 2.6)
