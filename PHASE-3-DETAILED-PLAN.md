# Phase 3: Build and Test Verification - Detailed Action Plan

**Status:** 📋 READY TO START  
**Estimated Time:** 3-4 hours  
**Complexity:** Medium-High  
**Risk Level:** Medium  
**Purpose:** Verify that all migrated code builds successfully and passes all tests

---

## 📋 Overview

Phase 3 validates that the migration was successful by building and testing all components. This phase ensures the renamed crate (`airsprotocols-mcp`) compiles correctly, all tests pass, examples work, and documentation generates properly.

**What this phase does:**
- Builds the entire workspace from scratch
- Runs all unit, integration, and doc tests
- Compiles all example projects
- Generates and verifies Rust documentation
- Compiles benchmark suites
- Builds mdBook documentation
- Documents any issues found

**What this phase does NOT do:**
- Does not run Python integration tests (that's Phase 4)
- Does not modify source code (unless critical bugs found)
- Does not publish to crates.io
- Does not run benchmarks (only compiles them)

---

## 🎯 Success Criteria

- ✅ `cargo build --workspace --all-features` succeeds (0 errors)
- ✅ All Rust tests pass (expected: 736 tests from baseline)
- ✅ All 6 examples compile successfully
- ✅ `cargo doc` generates without errors
- ✅ Benchmarks compile (don't need to run)
- ✅ mdBook documentation builds and renders
- ✅ No critical regressions from baseline

---

## 📊 Pre-Phase Checklist

Before starting Phase 3, verify:

- [x] Phase 2 completed successfully (all 6 steps)
- [x] Git working tree is clean
- [x] Currently in airsprotocols repository
- [x] All changes committed (git log shows Phase 2 commits)
- [ ] Baseline test results available (MIGRATION-BASELINE-STATE.txt)

**Verify location:**
```bash
pwd
# Expected: /Users/hiraq/Projects/airsstack/airsprotocols

git status
# Expected: "nothing to commit, working tree clean"
```

---

## 🔧 Phase 3 Steps

### Step 3.1: Clean Build (Est. 5-10 min)

**Goal:** Build entire workspace from scratch to verify compilation

**Duration:** 5-10 minutes (depending on machine)

#### Commands

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Clean all previous build artifacts
cargo clean

# Verify clean
ls -la target/ 2>/dev/null || echo "✅ Target directory clean"

# Build entire workspace with all features
echo "Starting full workspace build..."
time cargo build --workspace --all-features
```

#### Expected Output

**Success indicators:**
```
   Compiling airsprotocols-mcp v0.2.3 (/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp)
   Compiling http-apikey-client-integration v0.1.0 (...)
   Compiling http-apikey-server-integration v0.1.0 (...)
   ...
    Finished `dev` profile [unoptimized + debuginfo] target(s) in XXs
```

**Build time:** Typically 2-5 minutes for first build

#### Verification

```bash
# Check build succeeded
echo $?
# Expected: 0 (success)

# Verify binary artifacts created
ls -lh target/debug/ | grep airsprotocols

# Count compiled artifacts
find target/debug -name "*.rlib" -o -name "*.so" -o -name "*.dylib" | wc -l
```

#### Success Criteria

- ✅ Exit code: 0
- ✅ No compilation errors
- ✅ Binary artifacts created in target/debug/
- ✅ Workspace members compiled successfully

#### Common Issues & Solutions

**Issue 1: Dependency resolution errors**
```
error: failed to select a version for `package-name`
```
**Solution:**
```bash
cargo update
cargo build --workspace --all-features
```

**Issue 2: Feature flag conflicts**
```
error: feature `xyz` is required but not enabled
```
**Solution:** Check Cargo.toml feature dependencies, ensure workspace features are properly configured

**Issue 3: Missing workspace members**
```
error: package `xyz` is not a member of workspace
```
**Solution:** Verify Cargo.toml workspace members list includes all necessary packages

#### Documentation

**Document in MIGRATION-PROGRESS-LOG.md:**
```markdown
## Phase 3 Step 3.1: Clean Build (2025-12-06 - Time: HH:MM)

**Status:** [✅ COMPLETE / ❌ FAILED / ⚠️ WITH WARNINGS]

**Build time:** X minutes Y seconds

**Warnings:** X warnings (list if any)

**Errors:** None / [list if any]

**Build output summary:**
- Compiled X crates
- Generated Y artifacts
- Target size: Z MB
```

---

### Step 3.2: Run All Tests (Est. 30-45 min)

**Goal:** Execute all unit, integration, and doc tests to verify functionality

**Duration:** 30-45 minutes (736 tests expected)

#### Commands

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Run all tests with verbose output
echo "Starting test suite..."
time cargo test --workspace --all-features -- --nocapture 2>&1 | tee test-results.log
```

**Alternative (less verbose):**
```bash
time cargo test --workspace --all-features
```

#### Expected Output

**Success indicators:**
```
running 352 tests (unit tests - airsprotocols-mcp)
...
test result: ok. 352 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

running 112 tests (doc tests - airsprotocols-mcp)
...
test result: ok. 112 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

Doc-tests airsprotocols-mcp
...

test result: ok. 736 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in XXXs
```

**Baseline comparison:**
- Expected total: 736 tests (464 from airsprotocols-mcp + 272 from examples)
- Pass rate: 100%

#### Verification

```bash
# Check test exit code
echo $?
# Expected: 0

# Count test results from log
grep "test result:" test-results.log | tail -1

# Check for any panics or failures
grep -i "FAILED\|panic" test-results.log || echo "✅ No failures detected"

# Summary statistics
echo "Test summary:"
grep "running.*tests" test-results.log | wc -l
grep "test result: ok" test-results.log | wc -l
```

#### Success Criteria

- ✅ Exit code: 0
- ✅ All tests pass (0 failed)
- ✅ Test count matches baseline (~736 tests)
- ✅ No panics or unexpected errors
- ✅ Doc tests pass

#### Test Categories

**Expected test breakdown:**
1. **Unit tests (airsprotocols-mcp):** ~352 tests
   - Protocol types and serialization
   - Transport adapters (stdio, http, sse)
   - Authentication strategies
   - Integration layer
   
2. **Doc tests (airsprotocols-mcp):** ~112 tests
   - Documentation examples
   - Code snippets in comments
   
3. **Example tests:** ~272 tests
   - Integration example tests
   - Mock tests
   - Production integration tests

#### Common Issues & Solutions

**Issue 1: Test failures due to timing**
```
test timeout_test ... FAILED
```
**Solution:** Increase timeout in test or run individually:
```bash
cargo test timeout_test -- --nocapture --test-threads=1
```

**Issue 2: Doc test failures**
```
error: test failed, to rerun pass `--doc`
```
**Solution:** Check documentation examples are up to date with new crate name
```bash
cargo test --doc -- --nocapture
```

**Issue 3: Example test failures**
```
test integration_test ... FAILED
```
**Solution:** Run specific example tests:
```bash
cd protocols/mcp/examples/[example-name]
cargo test -- --nocapture
```

**Issue 4: Port conflicts in tests**
```
error: Address already in use
```
**Solution:** Run tests serially:
```bash
cargo test --workspace --all-features -- --test-threads=1
```

#### Documentation

**Document in MIGRATION-PROGRESS-LOG.md:**
```markdown
## Phase 3 Step 3.2: Run All Tests (2025-12-06 - Time: HH:MM)

**Status:** [✅ COMPLETE / ❌ FAILED / ⚠️ PARTIAL]

**Test duration:** X minutes Y seconds

**Test results:**
- Total tests: XXX
- Passed: XXX
- Failed: XXX
- Ignored: XXX

**Breakdown:**
- Unit tests: XXX passed
- Doc tests: XXX passed
- Integration tests: XXX passed

**Issues:** None / [list if any]

**Comparison to baseline:**
- Baseline: 736 tests
- Current: XXX tests
- Delta: [+/-]XX tests
```

---

### Step 3.3: Build All Examples (Est. 5-10 min)

**Goal:** Verify all example projects compile successfully

**Duration:** 5-10 minutes

#### Commands

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Build all examples
echo "Building all examples..."
time cargo build --workspace --examples --all-features
```

**Alternative (build each example individually):**
```bash
for example in protocols/mcp/examples/*/; do
    echo "Building example: $(basename $example)"
    (cd "$example" && cargo build --all-features)
done
```

#### Expected Output

**Success indicators:**
```
   Compiling http-apikey-client-integration v0.1.0
   Compiling http-apikey-server-integration v0.1.0
   Compiling http-oauth2-client-integration v0.1.0
   Compiling http-oauth2-server-integration v0.1.0
   Compiling stdio-client-integration v0.1.0
   Compiling stdio-server-integration v0.1.0
    Finished `dev` profile [unoptimized + debuginfo] target(s) in XXs
```

#### Verification

```bash
# Verify examples built
ls -la protocols/mcp/examples/*/target/debug/ 2>/dev/null | grep -E "http-|stdio-"

# Count example binaries
find protocols/mcp/examples -name "target" -type d -exec sh -c 'ls {}"/debug/" 2>/dev/null | grep -v "\.d$" | wc -l' \;

# List all example binaries
find protocols/mcp/examples -path "*/target/debug/*" -type f -perm +111 ! -name "*.so" ! -name "*.dylib" ! -name "*.d"
```

#### Expected Examples (6 total)

1. **http-apikey-client-integration** - HTTP client with API key auth
2. **http-apikey-server-integration** - HTTP server with API key auth
3. **http-oauth2-client-integration** - HTTP client with OAuth2
4. **http-oauth2-server-integration** - HTTP server with OAuth2
5. **stdio-client-integration** - Stdio client example
6. **stdio-server-integration** - Stdio server example

#### Success Criteria

- ✅ All 6 examples compile without errors
- ✅ Binary executables created for each example
- ✅ No dependency resolution issues
- ✅ All features compile correctly

#### Common Issues & Solutions

**Issue 1: Missing dependencies in examples**
```
error: package `xyz` not found
```
**Solution:**
```bash
cd protocols/mcp/examples/[example-name]
cargo update
cargo build
```

**Issue 2: Path dependencies not found**
```
error: failed to load manifest for dependency `airsprotocols-mcp`
```
**Solution:** Verify example Cargo.toml has correct path:
```toml
airsprotocols-mcp = { path = "../../" }
```

#### Documentation

**Document in MIGRATION-PROGRESS-LOG.md:**
```markdown
## Phase 3 Step 3.3: Build All Examples (2025-12-06 - Time: HH:MM)

**Status:** [✅ COMPLETE / ❌ FAILED]

**Build duration:** X minutes Y seconds

**Examples built:** 6/6

**List:**
- ✅ http-apikey-client-integration
- ✅ http-apikey-server-integration
- ✅ http-oauth2-client-integration
- ✅ http-oauth2-server-integration
- ✅ stdio-client-integration
- ✅ stdio-server-integration

**Issues:** None / [list if any]
```

---

### Step 3.4: Generate Documentation (Est. 3-5 min)

**Goal:** Generate Rust API documentation and verify correctness

**Duration:** 3-5 minutes

#### Commands

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Generate documentation for workspace
echo "Generating documentation..."
time cargo doc --workspace --no-deps --all-features

# Open in browser (optional)
cargo doc --workspace --no-deps --all-features --open
```

**Alternative (documentation only for airsprotocols-mcp):**
```bash
cd protocols/mcp
cargo doc --no-deps --all-features --open
```

#### Expected Output

**Success indicators:**
```
 Documenting airsprotocols-mcp v0.2.3 (/Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in XXs
    Generated /Users/hiraq/Projects/airsstack/airsprotocols/target/doc/airsprotocols_mcp/index.html
```

#### Verification

**Automated checks:**
```bash
# Verify documentation generated
ls -la target/doc/airsprotocols_mcp/index.html
# Expected: File exists

# Check documentation size
du -sh target/doc/airsprotocols_mcp/
# Expected: Several MB

# Count documented items
find target/doc/airsprotocols_mcp -name "*.html" | wc -l
# Expected: 100+ HTML files
```

**Manual verification (in browser):**
- [ ] Documentation opens successfully
- [ ] Main page shows "airsprotocols_mcp" (with underscore)
- [ ] Crate version shows "0.2.3"
- [ ] Module structure is correct:
  - protocol/
  - transport/
  - integration/
  - authentication/
  - providers/
- [ ] Code examples render correctly
- [ ] Links work (no 404s on internal links)
- [ ] Search functionality works

#### Success Criteria

- ✅ Documentation generates without errors
- ✅ HTML files created in target/doc/airsprotocols_mcp/
- ✅ Main index.html accessible
- ✅ Crate name correct (airsprotocols_mcp)
- ✅ No broken internal links
- ✅ Code examples display correctly

#### Common Issues & Solutions

**Issue 1: Documentation warnings**
```
warning: missing documentation for public item
```
**Solution:** These are warnings, not errors. Document if time permits, but don't block on them.

**Issue 2: Broken links in docs**
```
warning: unresolved link to `xyz`
```
**Solution:** Check if link target exists or needs updating:
```bash
grep -r "xyz" src/ --include="*.rs"
```

**Issue 3: Doc generation fails**
```
error: could not document `airsprotocols-mcp`
```
**Solution:** Check for syntax errors in doc comments:
```bash
cargo doc --workspace --no-deps 2>&1 | grep "error:"
```

#### Documentation

**Document in MIGRATION-PROGRESS-LOG.md:**
```markdown
## Phase 3 Step 3.4: Generate Documentation (2025-12-06 - Time: HH:MM)

**Status:** [✅ COMPLETE / ⚠️ WITH WARNINGS]

**Generation time:** X minutes Y seconds

**Output location:** target/doc/airsprotocols_mcp/

**Statistics:**
- HTML files: XXX
- Documentation size: X.X MB
- Warnings: XX

**Manual verification:**
- [x] Documentation opens in browser
- [x] Crate name correct: airsprotocols_mcp
- [x] Version shown: 0.2.3
- [x] Module structure correct
- [x] Code examples render
- [x] Links work

**Issues:** None / [list warnings if any]
```

---

### Step 3.5: Compile Benchmarks (Est. 2-3 min)

**Goal:** Verify benchmark code compiles (don't need to run them)

**Duration:** 2-3 minutes

#### Commands

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp

# Build benchmarks
echo "Compiling benchmarks..."
time cargo build --benches --all-features
```

**List available benchmarks:**
```bash
ls -la benches/
# Expected: lightweight_benchmarks.rs (and possibly others)
```

#### Expected Output

**Success indicators:**
```
   Compiling airsprotocols-mcp v0.2.3
    Finished `dev` profile [unoptimized + debuginfo] target(s) in XXs
```

#### Verification

```bash
# Check benchmarks compiled
ls -la target/debug/deps/ | grep benchmark

# Count benchmark binaries
find target/debug -name "*benchmark*" -type f | wc -l
```

#### Expected Benchmarks

From baseline state:
- `lightweight_benchmarks.rs` - Core performance benchmarks

#### Success Criteria

- ✅ Benchmarks compile without errors
- ✅ Benchmark binaries created
- ✅ No dependency issues
- ✅ All benchmark dependencies available

#### Common Issues & Solutions

**Issue 1: Benchmark dependency missing**
```
error: package `criterion` not found
```
**Solution:** Check Cargo.toml [dev-dependencies]:
```toml
[dev-dependencies]
criterion = "0.5"
```

**Issue 2: Benchmark code references old crate name**
```
error: unresolved import `airs_mcp`
```
**Solution:** This should have been caught in Phase 2, but if found:
```bash
grep -r "airs_mcp" benches/
# Update any references found
```

#### Optional: Run Benchmarks (Time permitting)

**Not required for migration validation, but useful for baseline:**
```bash
cargo bench --bench lightweight_benchmarks
```

This will take 10-15 minutes and generate performance metrics.

#### Documentation

**Document in MIGRATION-PROGRESS-LOG.md:**
```markdown
## Phase 3 Step 3.5: Compile Benchmarks (2025-12-06 - Time: HH:MM)

**Status:** [✅ COMPLETE / ❌ FAILED]

**Compilation time:** X minutes Y seconds

**Benchmarks found:**
- lightweight_benchmarks.rs

**Compilation result:** Success

**Benchmarks run:** [Yes/No - if time permitted]

**Issues:** None / [list if any]
```

---

### Step 3.6: Build mdBook Documentation (Est. 5-10 min)

**Goal:** Build and verify the mdBook user guide

**Duration:** 5-10 minutes (includes installation if needed)

#### Check mdBook Installation

```bash
# Check if mdBook is installed
which mdbook
# If not found: cargo install mdbook

mdbook --version
# Expected: mdbook v0.4.x or higher
```

**Install if needed:**
```bash
cargo install mdbook
# Takes ~5 minutes
```

#### Commands

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols/protocols/mcp/docs

# Verify book structure
ls -la
# Expected: book.toml, src/, theme/ (if present)

# Build the book
echo "Building mdBook documentation..."
time mdbook build

# Check build output
ls -la book/
# Expected: HTML files generated
```

#### Expected Output

**Success indicators:**
```
[INFO] (mdbook::book): Book building has started
[INFO] (mdbook::book): Running the html backend
[INFO] (mdbook::renderer): Invoking the "html" renderer
```

#### Verification

**Automated checks:**
```bash
# Verify book built
ls -la book/index.html
# Expected: File exists

# Check book size
du -sh book/
# Expected: Several MB

# Count pages
find book -name "*.html" | wc -l
# Expected: 10+ pages
```

**Manual verification (serve locally):**
```bash
# Serve book locally
mdbook serve --open
# Opens http://localhost:3000 in browser
```

**In browser, verify:**
- [ ] Documentation loads at http://localhost:3000
- [ ] Table of contents shows all chapters
- [ ] Navigation works (Previous/Next buttons)
- [ ] Code examples render with syntax highlighting
- [ ] Images load (if any)
- [ ] Search functionality works
- [ ] Links work (no 404s)
- [ ] Responsive design works (test mobile view)

**Press Ctrl+C to stop server**

#### Expected Book Structure

Based on crate documentation:
- Introduction
- Getting Started / Quick Start
- Protocol documentation
- Transport adapters
- Usage examples
- Advanced patterns
- API reference links

#### Success Criteria

- ✅ mdBook builds without errors
- ✅ HTML output generated in book/
- ✅ book/index.html exists and loads
- ✅ All chapters accessible
- ✅ Navigation functional
- ✅ Code examples render correctly
- ✅ No broken links

#### Common Issues & Solutions

**Issue 1: mdBook not installed**
```
command not found: mdbook
```
**Solution:**
```bash
cargo install mdbook
```

**Issue 2: Book build fails**
```
Error: Failed to build book
```
**Solution:** Check book.toml and src/SUMMARY.md:
```bash
cat book.toml
cat src/SUMMARY.md
```

**Issue 3: Missing chapters**
```
Warning: File not found: src/chapter-name.md
```
**Solution:** Check SUMMARY.md references match actual files:
```bash
ls -la src/*.md
```

**Issue 4: Broken links**
```
Warning: Potential broken link
```
**Solution:** Review and fix links in markdown files

#### Documentation

**Document in MIGRATION-PROGRESS-LOG.md:**
```markdown
## Phase 3 Step 3.6: Build mdBook Documentation (2025-12-06 - Time: HH:MM)

**Status:** [✅ COMPLETE / ⚠️ WITH WARNINGS]

**Build time:** X minutes Y seconds

**Output location:** protocols/mcp/docs/book/

**Statistics:**
- HTML pages: XX
- Book size: X.X MB
- Chapters: XX

**Manual verification:**
- [x] Book loads at localhost:3000
- [x] Table of contents complete
- [x] Navigation works
- [x] Code examples render
- [x] Search works
- [x] No broken links

**Issues:** None / [list if any]
```

---

## 📝 Phase 3 Completion

### Final Verification Checklist

After completing all 6 steps, verify:

- [ ] Step 3.1: Workspace builds successfully (0 errors)
- [ ] Step 3.2: All tests pass (~736 tests)
- [ ] Step 3.3: All 6 examples compile
- [ ] Step 3.4: Rust docs generate correctly
- [ ] Step 3.5: Benchmarks compile
- [ ] Step 3.6: mdBook documentation builds

### Generate Phase 3 Summary

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

cat > PHASE-3-COMPLETION-SUMMARY.md << 'SUMMARY'
# Phase 3: Build and Test Verification - Completion Summary

**Date:** 2025-12-06
**Status:** [✅ COMPLETE / ⚠️ COMPLETE WITH ISSUES / ❌ FAILED]
**Total Duration:** X hours Y minutes

## Summary

[Brief overview of Phase 3 results]

## Step Results

### Step 3.1: Clean Build
- Status: [✅/❌/⚠️]
- Duration: X min
- Issues: [None / List]

### Step 3.2: Run All Tests
- Status: [✅/❌/⚠️]
- Duration: X min
- Tests: XXX passed, XXX failed
- Issues: [None / List]

### Step 3.3: Build Examples
- Status: [✅/❌/⚠️]
- Duration: X min
- Examples: 6/6
- Issues: [None / List]

### Step 3.4: Generate Documentation
- Status: [✅/❌/⚠️]
- Duration: X min
- Issues: [None / List]

### Step 3.5: Compile Benchmarks
- Status: [✅/❌/⚠️]
- Duration: X min
- Issues: [None / List]

### Step 3.6: Build mdBook
- Status: [✅/❌/⚠️]
- Duration: X min
- Issues: [None / List]

## Comparison to Baseline

**Baseline (airs-mcp in airsstack):**
- Tests: 736 (464 unit + 112 doc + 160 integration)
- Build time: ~X min
- Examples: 6

**Current (airsprotocols-mcp):**
- Tests: XXX (breakdown)
- Build time: ~X min
- Examples: 6

**Delta:** [analysis]

## Issues Found

[List any issues discovered during Phase 3]

## Recommendations

[Any recommendations for next steps]

## Next Phase

Phase 4: Python Integration Tests (3-4 hours estimated)

---

**Phase 3 Status:** [✅ COMPLETE / ⚠️ COMPLETE WITH ISSUES / ❌ BLOCKED]
SUMMARY
```

### Update Progress Log

Add Phase 3 completion to MIGRATION-PROGRESS-LOG.md:

```markdown
---

## Phase 3: Build and Test Verification (2025-12-06)

**Status:** ✅ COMPLETE
**Duration:** X hours Y minutes

### Summary

All 6 steps completed successfully. The airsprotocols-mcp crate builds, tests pass, examples compile, and documentation generates correctly.

### Results

- ✅ Workspace build: Success
- ✅ Tests: XXX/XXX passed (100%)
- ✅ Examples: 6/6 compiled
- ✅ Documentation: Generated
- ✅ Benchmarks: Compiled
- ✅ mdBook: Built successfully

### Detailed Results

[Link to PHASE-3-COMPLETION-SUMMARY.md for details]

### Next Steps

Proceed to Phase 4: Python Integration Tests
```

### Commit Changes

```bash
cd /Users/hiraq/Projects/airsstack/airsprotocols

# Add summary documents
git add PHASE-3-COMPLETION-SUMMARY.md MIGRATION-PROGRESS-LOG.md

# Commit
git commit -m "Phase 3 complete: Build and test verification

- All workspace builds successful
- XXX tests passed (100%)
- All 6 examples compile
- Documentation generated
- Benchmarks compile
- mdBook builds successfully

Phase 3: Build and Test Verification COMPLETE"
```

---

## 🚨 Troubleshooting

### Build Failures

**Symptom:** Compilation errors
**Diagnosis:**
```bash
cargo build --workspace --all-features 2>&1 | grep "error:"
```
**Common causes:**
- Missing dependencies
- Incorrect import paths
- Feature flag issues
- Syntax errors

**Resolution:**
1. Review error messages carefully
2. Check if Phase 2 missed any references
3. Verify Cargo.toml correctness
4. Run `cargo update` if dependency issues

### Test Failures

**Symptom:** Tests failing
**Diagnosis:**
```bash
cargo test --workspace -- --nocapture 2>&1 | grep -A 10 "FAILED"
```
**Common causes:**
- Timing issues
- Port conflicts
- Path assumptions
- Test interdependencies

**Resolution:**
1. Run failing test individually: `cargo test test_name -- --nocapture`
2. Run serially: `cargo test -- --test-threads=1`
3. Check test assumes nothing about working directory
4. Verify no hardcoded paths or ports

### Documentation Issues

**Symptom:** Doc generation warnings/errors
**Diagnosis:**
```bash
cargo doc --workspace 2>&1 | grep "warning:\|error:"
```
**Common causes:**
- Broken links
- Missing docs
- Invalid markdown
- Incorrect code examples

**Resolution:**
1. Fix broken links in doc comments
2. Update code examples to use new crate name
3. Validate markdown syntax
4. Run doc tests: `cargo test --doc`

### Example Build Failures

**Symptom:** Example won't compile
**Diagnosis:**
```bash
cd protocols/mcp/examples/[example-name]
cargo build 2>&1 | grep "error:"
```
**Common causes:**
- Wrong dependency path
- Missing features
- Outdated dependencies

**Resolution:**
1. Verify Cargo.toml dependency paths correct
2. Check required features enabled
3. Run `cargo update` in example directory

---

## 📊 Baseline Comparison

### Expected Metrics (from Phase 0 baseline)

**Build:**
- Clean build time: 2-5 minutes
- Incremental: <1 minute

**Tests:**
- Total: 736 tests
- airsprotocols-mcp: 464 tests (352 unit + 112 doc)
- Examples: 272 tests (integration)
- Pass rate: 100%

**Examples:**
- Count: 6 examples
- All compile successfully

**Documentation:**
- Rust docs: ~100+ pages
- mdBook: ~10+ chapters

### Acceptable Deltas

- ✅ Test count: ±5% acceptable (700-770 tests)
- ✅ Build time: ±20% acceptable
- ⚠️ Test failures: 0 expected, investigate any failures
- ⚠️ Build errors: 0 expected, must fix before proceeding

---

## ✅ Phase 3 Exit Criteria

Before proceeding to Phase 4, ensure:

- ✅ All 6 steps completed
- ✅ Zero build errors
- ✅ Zero test failures (or documented and acceptable)
- ✅ All examples compile
- ✅ Documentation generates
- ✅ PHASE-3-COMPLETION-SUMMARY.md created
- ✅ MIGRATION-PROGRESS-LOG.md updated
- ✅ Changes committed to git
- ✅ Baseline comparison documented

**If any criteria not met:** Address issues before Phase 4

**If all criteria met:** ✅ Ready for Phase 4 (Python Integration Tests)

---

## 📚 Related Documents

- MIGRATION-ACTION-PLAN.md - Overall migration plan
- PHASE-0-DETAILED-PLAN.md - Pre-migration verification
- PHASE-1-DETAILED-PLAN.md - Repository setup
- PHASE-2-DETAILED-PLAN.md - Copy and adapt
- MIGRATION-PROGRESS-LOG.md - Step-by-step progress log
- MIGRATION-BASELINE-STATE.txt - Pre-migration baseline

---

**Document Status:** 📋 READY FOR USE  
**Created:** 2025-12-06  
**Phase Status:** READY TO START
