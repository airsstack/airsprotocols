# Phase 1: Repository Structure Setup - Detailed Action Plan

**Status:** ⏳ READY TO START  
**Estimated Time:** 30 minutes  
**Complexity:** Low  
**Risk Level:** Minimal  
**Purpose:** Create the directory structure in airsprotocols to receive the migrated MCP crate

---

## 📋 Overview

Phase 1 is a simple but important preparation phase. We'll create the `protocols/mcp/` directory structure and verify that the workspace configuration is ready to receive the migrated code.

**What this phase does:**
- Creates empty directory structure
- Verifies workspace configuration
- Confirms cargo recognizes the setup
- Documents the preparation

**What this phase does NOT do:**
- Does not copy any files (that's Phase 2)
- Does not modify any code
- Does not run builds (directory will be empty)
- Does not touch airsstack repository

---

## 🎯 Phase 1 Objectives

1. ✅ Create the `protocols/mcp/` directory in airsprotocols
2. ✅ Verify the workspace Cargo.toml is correctly configured
3. ✅ Confirm the structure is ready to receive the migrated code
4. ✅ Document the preparation steps

---

## 📝 Detailed Steps

### **Step 1.1: Create Directory Structure**

**Time:** 5 minutes  
**Risk Level:** Low  
**Goal:** Create the empty protocols/mcp directory

#### Actions:

1. **Navigate to airsprotocols:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols
   ```

2. **Create the directory:**
   ```bash
   mkdir -p protocols/mcp
   ```
   
   The `-p` flag creates parent directories if needed and doesn't error if directory exists.

3. **Verify creation:**
   ```bash
   ls -la protocols/
   ```
   
   **Expected output:**
   ```
   drwxr-xr-x  3 user  staff  96 Dec  6 XX:XX .
   drwxr-xr-x  8 user  staff 256 Dec  6 XX:XX ..
   drwxr-xr-x  2 user  staff  64 Dec  6 XX:XX mcp
   ```

4. **Confirm directory is empty:**
   ```bash
   ls -la protocols/mcp/
   ```
   
   **Expected output:**
   ```
   total 0
   drwxr-xr-x  2 user  staff  64 Dec  6 XX:XX .
   drwxr-xr-x  3 user  staff  96 Dec  6 XX:XX ..
   ```

#### Expected Outcome:
- ✅ Directory `protocols/mcp/` exists
- ✅ Directory is empty (no files)
- ✅ No errors during creation
- ✅ Proper permissions (readable, writable)

#### What Could Go Wrong:
- **Permissions issue:** Unlikely in your own project directory
- **Directory already exists:** Not a problem (mkdir -p handles this)
- **Disk space:** Already verified in Phase 0
- **Typo in path:** Immediate verification catches this

#### Rollback:
```bash
# If needed to undo:
cd /Users/hiraq/Projects/airsstack/airsprotocols
rm -rf protocols/mcp
```

**Checkpoint:**
- [ ] Directory created successfully
- [ ] Directory is empty
- [ ] No errors encountered

---

### **Step 1.2: Verify Workspace Configuration**

**Time:** 10 minutes  
**Risk Level:** Very Low (read-only verification)  
**Goal:** Confirm workspace Cargo.toml is correctly configured

#### Actions:

1. **Read the workspace Cargo.toml:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols
   cat Cargo.toml
   ```

2. **Verify workspace members section:**
   
   **Should contain:**
   ```toml
   [workspace]
   members = [
       "protocols/mcp",
   ]
   resolver = "2"
   ```
   
   This tells Cargo that `protocols/mcp` is a workspace member.

3. **Verify workspace dependencies section:**
   
   **Should contain:**
   ```toml
   [workspace.dependencies]
   # Layer 1: AIRS Protocols Crates
   airsprotocols-mcp = { path = "protocols/mcp" }
   ```
   
   This allows other crates to reference `airsprotocols-mcp` using workspace inheritance.

4. **Verify workspace package metadata:**
   
   **Should contain:**
   ```toml
   [workspace.package]
   version = "0.2.3"
   edition = "2021"
   authors = ["rstlix0x0 <rstlix.dev@gmail.com>"]
   license = "MIT OR Apache-2.0"
   repository = "https://github.com/airsstack/airsprotocols"
   rust-version = "1.88"
   ```
   
   This metadata will be inherited by the MCP crate.

5. **Check for any conflicts:**
   ```bash
   grep -n "protocols/mcp" Cargo.toml
   # Should show the line numbers where it's referenced
   ```

#### Expected Outcome:
- ✅ Workspace members includes "protocols/mcp"
- ✅ Workspace dependencies includes airsprotocols-mcp path
- ✅ Workspace package metadata is complete
- ✅ No syntax errors in Cargo.toml
- ✅ Configuration is migration-ready

**Note:** This configuration should already be correct from the initial repository setup. We're just verifying it's ready.

#### What We're Looking For:
- Correct path references
- No typos
- Proper TOML syntax
- Complete workspace inheritance setup

**Checkpoint:**
- [ ] Workspace members verified
- [ ] Workspace dependencies verified
- [ ] Workspace package metadata verified
- [ ] No configuration issues found

---

### **Step 1.3: Test Workspace Recognition**

**Time:** 10 minutes  
**Risk Level:** Low  
**Goal:** Verify that Cargo recognizes the workspace structure

#### Actions:

1. **Try to build the workspace:**
   ```bash
   cd /Users/hiraq/Projects/airsstack/airsprotocols
   cargo build --workspace 2>&1 | head -20
   ```

2. **Expected error (this is CORRECT):**
   ```
   error: failed to load manifest for workspace member `/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp`
   
   Caused by:
     failed to read `/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp/Cargo.toml`
   
   Caused by:
     No such file or directory (os error 2)
   ```
   
   **This error is EXPECTED and GOOD!** It means:
   - ✅ Cargo sees the workspace configuration
   - ✅ Cargo knows to look for protocols/mcp
   - ✅ The path is correct
   - ✅ We just haven't copied the files yet (Phase 2)

3. **Try cargo metadata:**
   ```bash
   cargo metadata --no-deps --format-version 1 2>&1 | grep -i "protocols/mcp" || echo "Expected error: member not found"
   ```
   
   Will also error, which is expected.

4. **Verify the workspace knows about the member:**
   ```bash
   cat Cargo.toml | grep -A 5 "members"
   ```

#### Expected Outcome:
- ✅ Error message confirms Cargo is looking for protocols/mcp/Cargo.toml
- ✅ Error is "No such file or directory" (correct - files don't exist yet)
- ✅ No workspace configuration syntax errors
- ✅ Path references are correct

#### What This Proves:
- Workspace configuration is valid
- Cargo recognizes protocols/mcp as a member
- Path resolution works correctly
- System is ready for Phase 2 file copy

#### ⚠️ IMPORTANT: Expected Error

**You WILL see this error - it's CORRECT:**

```
error: failed to load manifest for workspace member `protocols/mcp`
```

**Why this is good:**
- Proves workspace sees the member
- Proves path is correct
- Expected because we haven't copied Cargo.toml yet
- Will be resolved in Phase 2

**What would be BAD:**
- Syntax error in Cargo.toml
- Workspace not recognizing members section
- Wrong path reference
- Permission errors

**Checkpoint:**
- [ ] Build error received (expected)
- [ ] Error message mentions protocols/mcp
- [ ] Error is "No such file or directory"
- [ ] No syntax or configuration errors

---

### **Step 1.4: Update Documentation**

**Time:** 5 minutes  
**Risk Level:** None (documentation only)  
**Goal:** Record Phase 1 completion and observations

#### Actions:

1. **Update MIGRATION-PROGRESS-LOG.md:**
   
   Add Phase 1 section:
   ```markdown
   ## Phase 1: Repository Structure Setup
   
   ### Step 1.1: Create Directory Structure ✅
   **Completed:** [TIME]
   **Actions:** Created protocols/mcp directory
   **Result:** Directory exists and is empty
   
   ### Step 1.2: Verify Workspace Configuration ✅
   **Completed:** [TIME]
   **Actions:** Verified Cargo.toml workspace configuration
   **Result:** Configuration correct and ready
   
   ### Step 1.3: Test Workspace Recognition ✅
   **Completed:** [TIME]
   **Actions:** Tested cargo workspace member recognition
   **Result:** Expected error received, workspace configured correctly
   
   ### Step 1.4: Update Documentation ✅
   **Completed:** [TIME]
   **Actions:** Updated all progress tracking
   **Result:** Documentation current
   
   **Phase 1 Status:** ✅ COMPLETE
   **Duration:** [ACTUAL TIME]
   ```

2. **Update MIGRATION-ACTION-PLAN.md header:**
   
   Change progress section:
   ```markdown
   ## 📊 Current Progress
   
   **Phase 0: Pre-Migration Verification** ✅ COMPLETE
   **Phase 1: Repository Structure Setup** ✅ COMPLETE
   - ✅ Step 1.1: Create Directory Structure
   - ✅ Step 1.2: Verify Workspace Configuration
   - ✅ Step 1.3: Test Workspace Recognition
   - ✅ Step 1.4: Update Documentation
   ```

3. **Update MIGRATION-BASELINE-STATE.txt:**
   
   Add Phase 1 completion:
   ```
   Phase 1 Completion Time: [TIME]
   Phase 1 Duration: [DURATION]
   Phase 1 Status: ✅ COMPLETE
   
   Phase 1 Results:
   - protocols/mcp directory created
   - Workspace configuration verified
   - Cargo workspace recognition tested
   - Ready for Phase 2 file copy
   ```

4. **Create Phase 1 summary:**
   ```bash
   echo "Phase 1 completed at $(date)" >> phase1-completion.txt
   ```

#### Expected Outcome:
- ✅ All documentation files updated
- ✅ Progress accurately reflected
- ✅ Timestamps recorded
- ✅ Ready for Phase 2 tracking

**Checkpoint:**
- [ ] MIGRATION-PROGRESS-LOG.md updated
- [ ] MIGRATION-ACTION-PLAN.md updated
- [ ] MIGRATION-BASELINE-STATE.txt updated
- [ ] Phase 1 marked complete

---

## 📊 Phase 1 Deliverables

By the end of Phase 1, you will have:

### Directory Structure:
```
airsprotocols/
├── protocols/
│   └── mcp/                    # ← NEW: Empty, ready for files
├── Cargo.toml                  # ← VERIFIED: Workspace configured
├── README.md
├── MIGRATIONS.md
├── MIGRATION-ACTION-PLAN.md    # ← UPDATED: Phase 1 complete
├── MIGRATION-PROGRESS-LOG.md   # ← UPDATED: Phase 1 logged
├── MIGRATION-BASELINE-STATE.txt # ← UPDATED: Phase 1 recorded
└── PHASE-0-DETAILED-PLAN.md
```

### Verification Complete:
- ✅ Directory `protocols/mcp/` exists
- ✅ Directory is empty (no files yet)
- ✅ Workspace configuration verified correct
- ✅ Cargo recognizes the workspace member
- ✅ Expected error behavior confirmed
- ✅ System ready for Phase 2

### Documentation Updated:
- ✅ Progress logs current
- ✅ All steps documented
- ✅ Timestamps recorded
- ✅ Ready for Phase 2

---

## ✅ Phase 1 Completion Checklist

Before proceeding to Phase 2, verify:

### Directory Creation:
- [ ] `protocols/mcp/` directory exists
- [ ] Directory is empty (0 files)
- [ ] Directory has proper permissions
- [ ] Can write to directory

### Workspace Verification:
- [ ] Cargo.toml contains "protocols/mcp" in members
- [ ] Workspace dependencies include airsprotocols-mcp
- [ ] Workspace package metadata is complete
- [ ] No syntax errors in Cargo.toml

### Recognition Testing:
- [ ] Cargo build shows expected error
- [ ] Error message mentions protocols/mcp
- [ ] Error is "No such file or directory"
- [ ] No unexpected errors

### Documentation:
- [ ] MIGRATION-PROGRESS-LOG.md updated
- [ ] MIGRATION-ACTION-PLAN.md updated
- [ ] MIGRATION-BASELINE-STATE.txt updated
- [ ] Phase 1 completion time recorded

---

## 🔄 Rollback Procedure

If you need to undo Phase 1:

```bash
# Remove the directory
cd /Users/hiraq/Projects/airsstack/airsprotocols
rm -rf protocols/mcp

# Revert documentation (if needed)
git checkout MIGRATION-PROGRESS-LOG.md
git checkout MIGRATION-ACTION-PLAN.md
git checkout MIGRATION-BASELINE-STATE.txt
```

**Phase 1 Changes:**
- Only creates one directory
- Updates documentation files
- No code changes
- Easily reversible

---

## 🎯 Success Criteria

Phase 1 is successfully complete when:

1. ✅ `protocols/mcp/` directory exists and is empty
2. ✅ Workspace Cargo.toml verified correct
3. ✅ Cargo recognizes workspace member (even without files)
4. ✅ Expected error confirms correct configuration
5. ✅ All documentation updated
6. ✅ No unexpected issues encountered

**Target Time:** 30 minutes  
**Typical Time:** 20-25 minutes  
**Complexity:** Very Low  
**Risk:** Minimal

---

## ⚠️ Known Expected Behaviors

### Expected Error in Step 1.3:

```
error: failed to load manifest for workspace member `/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp`

Caused by:
  failed to read `/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp/Cargo.toml`

Caused by:
  No such file or directory (os error 2)
```

**This is CORRECT!** It means:
- ✅ Workspace configuration works
- ✅ Cargo knows to look for the member
- ✅ Path is correct
- ✅ Ready for Phase 2 to add files

### What Would Be Wrong:

❌ No error (would mean workspace doesn't see member)  
❌ Syntax error in Cargo.toml  
❌ Wrong path error  
❌ Permission denied error

---

## 📈 Progress After Phase 1

**Phases Complete:** 2/8 (25%)

- ✅ Phase 0: Pre-Migration Verification (20 min)
- ✅ Phase 1: Repository Structure Setup (20-30 min)
- ⏳ Phase 2: Copy and Adapt (2-3 hours) ← NEXT
- ⏳ Phase 3: Build and Test Verification (3-4 hours)
- ⏳ Phase 4: Python Integration Tests (3-4 hours)
- ⏳ Phase 5: Documentation Updates (1-2 hours)
- ⏳ Phase 6: Git Operations (30 min)
- ⏳ Phase 7: Final Verification (1-2 hours)
- ⏳ Phase 8: Post-Migration Tasks (30 min)

**Estimated Remaining:** 12-16 hours

---

## 🚀 What's Next: Phase 2 Preview

After Phase 1 is complete, Phase 2 will:

1. **Copy files** from `airsstack/crates/airs-mcp/` to `airsprotocols/protocols/mcp/`
2. **Update package name** from `airs-mcp` to `airsprotocols-mcp`
3. **Update all imports** in Rust code
4. **Update documentation** with migration notices
5. **Update example projects** to use new crate name

Phase 2 is the "big one" - it's where the actual migration happens.

---

## 💡 Why Phase 1 Matters

Even though Phase 1 is simple:

1. **Validates structure** - Confirms workspace is ready
2. **Tests configuration** - Ensures cargo setup is correct
3. **Provides checkpoint** - Clean state before big copy
4. **Documents baseline** - Clear record of preparation
5. **Reduces risk** - Catches config issues before file copy
6. **Enables rollback** - Clear point to return to if needed

---

## 📞 Getting Help

If issues arise during Phase 1:

1. **Directory creation fails:**
   - Check disk space: `df -h .`
   - Check permissions: `ls -la protocols/`
   - Verify path: `pwd`

2. **Workspace verification issues:**
   - Validate TOML syntax: `cargo metadata --format-version 1`
   - Check for typos in Cargo.toml
   - Review workspace.members section

3. **Unexpected errors:**
   - Document the error message
   - Check Phase 1 rollback procedure
   - Review Phase 0 baseline state

---

## ✅ Phase 1 Ready

**Prerequisites:**
- ✅ Phase 0 complete
- ✅ Both repositories in clean state
- ✅ Backups created
- ✅ Documentation established

**Phase 1 will:**
- Create one directory
- Verify configuration
- Test workspace
- Update documentation
- Take 20-30 minutes
- Be fully reversible

**Execution Mode:**
- Step-by-step with approval after each step
- Progress updates in real-time
- Documentation updated continuously

---

**Ready to execute Phase 1 when you give the command.**
