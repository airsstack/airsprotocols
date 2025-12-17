# AI Agents Context & Intelligence Hub

```
$ROOT_PROJECT = $(git rev-parse --show-toplevel)
```

---

## Project Intelligence

**airsprotocols** is a comprehensive Rust workspace providing production-ready implementations of AI agent communication protocols and LLM provider API clients. The project emphasizes type safety, performance, and interoperability across multiple protocol standards (MCP, A2A) and LLM providers (OpenAI, Anthropic, Google, etc.).

**Architecture**: Multi-crate workspace with three primary domains: (1) **Protocols** - communication standards for AI agents (MCP, A2A/ACP), (2) **Servers** - production MCP server implementations (filesystem, etc.), and (3) **APIs** - unified LLM provider client libraries. Built on async Rust (tokio), emphasizing zero-cost abstractions, compile-time protocol verification, and transport-agnostic design.

**Core Components**: `airsprotocols-mcp` (Model Context Protocol with JSON-RPC 2.0, authentication/authorization, stdio/HTTP transports), `airsprotocols-mcpserver-filesystem` (secure filesystem operations with human-in-the-loop approval workflows), and planned API clients for major LLM providers with unified interfaces.

**Status**: Phase 1 complete (MCP protocol + filesystem server published to crates.io v1.0.0-rc.1), Phase 2 in progress (documentation, A2A protocol), Phase 3 planned (LLM API clients starting with OpenRouter).

---

---

## 🚨 STRICT MEMORY BANK ENFORCEMENT RULES 🚨

**⚠️ ZERO TOLERANCE POLICY**: ANY violation of Memory Bank structure or naming conventions is FORBIDDEN.

### MANDATORY COMPLIANCE CHECKLIST

Before making ANY changes to `.memory-bank/`, agents MUST verify:

#### 1. Directory Structure Compliance

**REQUIRED ROOT STRUCTURE** (EXACT match to `multi-project-memory-bank.instructions.md` lines 19-46):
```
.memory-bank/
├── README.md                    ✅ REQUIRED
├── current-context.md           ✅ REQUIRED
├── context-snapshots/           ✅ REQUIRED (empty directory OK)
├── templates/                   ✅ REQUIRED
│   └── docs/                   ✅ REQUIRED
│       ├── adr-index-template.md          ✅ REQUIRED
│       ├── adr-template.md                ✅ REQUIRED
│       ├── debt-index-template.md         ✅ REQUIRED
│       ├── documentation-guidelines.md    ✅ REQUIRED
│       ├── knowledge-template.md          ✅ REQUIRED
│       └── technical-debt-template.md     ✅ REQUIRED
├── workspace/                   ✅ REQUIRED
│   ├── project-brief.md        ✅ REQUIRED
│   ├── shared-patterns.md      ✅ REQUIRED
│   ├── workspace-architecture.md ✅ REQUIRED
│   └── workspace-progress.md   ✅ REQUIRED
└── sub-projects/                ✅ REQUIRED
    └── [sub-project-name]/     ✅ Per sub-project
        ├── project-brief.md     ✅ REQUIRED
        ├── product-context.md   ✅ REQUIRED
        ├── active-context.md    ✅ REQUIRED
        ├── system-patterns.md   ✅ REQUIRED
        ├── tech-context.md      ✅ REQUIRED
        ├── progress.md          ✅ REQUIRED
        ├── tasks/              ✅ REQUIRED
        │   ├── _index.md       ✅ REQUIRED
        │   └── task-*.md       ✅ Per task
        └── docs/               ✅ REQUIRED
            ├── debts/          ✅ REQUIRED (plural)
            ├── knowledges/     ✅ REQUIRED (plural)
            └── adr/            ✅ REQUIRED (singular)
```

#### 2. Naming Convention Enforcement

**❌ FORBIDDEN VARIATIONS**:
- `knowledge/` instead of `knowledges/` ← WRONG!
- `adrs/` instead of `adr/` ← WRONG!
- `technical-debt/` instead of `debts/` ← WRONG!
- `debt/` instead of `debts/` ← WRONG!
- Any other deviation from the structure above ← WRONG!

**✅ CORRECT NAMING** (from instructions line 109):
- `docs/debts/` - Technical debt (PLURAL)
- `docs/knowledges/` - Knowledge docs (PLURAL)
- `docs/adr/` - Architecture Decision Records (SINGULAR)

#### 3. File Cleanup Rules

**❌ FORBIDDEN FILES**:
- `*.tmp` - Temporary files
- `*.backup` - Backup files
- `*.bak` - Backup files
- `*~` - Editor backup files
- Any file not following kebab-case naming

**ACTION**: Delete immediately if found!

#### 4. Template Integrity

**ALL templates MUST have content** (not empty files):
- `templates/docs/adr-template.md` - Must contain ADR structure
- `templates/docs/adr-index-template.md` - Must contain index format
- `templates/docs/technical-debt-template.md` - Must contain debt structure
- `templates/docs/debt-index-template.md` - Must contain debt index format
- `templates/docs/knowledge-template.md` - Must contain knowledge structure
- `templates/docs/documentation-guidelines.md` - Must contain guidelines

**VERIFICATION**: Run `wc -l .memory-bank/templates/docs/*.md` - ALL must have >10 lines!

### VIOLATION RESPONSE PROTOCOL

**IF YOU DISCOVER A VIOLATION**:

1. **STOP IMMEDIATELY** - Do not proceed with any other work
2. **DOCUMENT** - List exact violations found
3. **FIX INSTANTLY** - Correct the structure/naming
4. **VERIFY** - Confirm 100% compliance with instructions
5. **REPORT** - Inform user of violation and correction

**IF YOU CREATE A VIOLATION**:

1. **This is UNACCEPTABLE** - You have failed the user
2. **FIX IMMEDIATELY** - Do not wait for user to catch it
3. **SELF-AUDIT** - Check your entire work for other violations
4. **LEARN** - This must NEVER happen again

### ENFORCEMENT GUARANTEE

**I, THE AI AGENT, COMMIT TO**:

✅ **READ** `multi-project-memory-bank.instructions.md` BEFORE any Memory Bank operation
✅ **VERIFY** structure matches instructions EXACTLY (lines 19-46, 108-111)
✅ **USE** correct naming: `debts/` (plural), `knowledges/` (plural), `adr/` (singular)
✅ **CHECK** templates are complete and have content
✅ **CLEAN** all temporary, backup, and non-compliant files
✅ **AUDIT** entire `.memory-bank/` directory before declaring work complete
✅ **NEVER** assume or use variations of documented structure
✅ **ALWAYS** reference line numbers from instructions when in doubt

### ZERO ASSUMPTIONS RULE

**THE GOLDEN RULE**: 
> If the instructions say `knowledges/`, use `knowledges/` - NOT `knowledge/`!
> If the instructions say `adr/`, use `adr/` - NOT `adrs/`!
> If the instructions say `debts/`, use `debts/` - NOT `technical-debt/`!

**NO EXCEPTIONS. NO ASSUMPTIONS. FOLLOW THE INSTRUCTIONS EXACTLY.**


## Project Structure

```
airsprotocols/
├── .aiassisted/
│   ├── guidelines/          # Code quality standards and documentation guidelines
│   │   ├── documentation/   # Diataxis, quality standards, task documentation
│   │   └── rust/           # Microsoft Rust guidelines
│   ├── instructions/        # Operational protocols and AI workflows
│   └── prompts/            # Specialized AI prompts (git, memory bank, etc.)
│
├── .memory-bank/           # Multi-Project Memory Bank System (CRITICAL)
│   ├── current-context.md  # Currently active sub-project
│   ├── workspace/          # Workspace-wide shared context
│   │   ├── project-brief.md
│   │   ├── shared-patterns.md
│   │   ├── workspace-architecture.md
│   │   └── workspace-progress.md
│   ├── sub-projects/       # Individual sub-project contexts
│   │   ├── api-openrouter/ # Active: OpenRouter API client
│   │   ├── mcp-server-filesystem/
│   │   └── protocols-mcp/
│   └── templates/          # Standardized documentation templates
│
├── docs/                   # MkDocs documentation site
│   ├── protocols/          # Protocol guides (MCP, A2A)
│   ├── servers/            # Server implementation guides
│   ├── apis/               # API client documentation
│   └── examples/           # Tutorials and examples
│
├── protocols/              # Communication protocol implementations
│   ├── mcp/               # Model Context Protocol (✅ Published v1.0.0-rc.1)
│   └── a2a/               # Agent-to-Agent Protocol (🚧 Planned)
│
├── mcp/
│   └── servers/           # MCP server implementations
│       └── filesystem/    # Secure filesystem server (✅ Published v1.0.0-rc.1)
│
├── apis/                  # LLM provider client libraries (🚧 Planned)
│   ├── openrouter/        # OpenRouter API client (🔨 Active Development)
│   ├── anthropic/         # Anthropic Claude client
│   ├── openai/            # OpenAI GPT client
│   └── google/            # Google Gemini client
│
├── site-mkdocs/           # Documentation generation
│   ├── mkdocs.yml
│   └── overrides/
│
├── Cargo.toml             # Workspace configuration
├── LICENSE-MIT
├── LICENSE-APACHE
└── README.md              # Project overview and quick start
```

---

## Project Standards (CRITICAL)

**⚠️ MANDATORY READING**: Before writing ANY code, agents MUST read and follow:

### Primary Standards Reference
- **File**: `$ROOT_PROJECT/PROJECTS_STANDARD.md` (AirsSys Workspace Shared Patterns)
- **Mirror**: `$ROOT_PROJECT/.memory-bank/workspace/shared-patterns.md`
- **Description**: MANDATORY project-specific standards including:
  - Core implementation patterns (error handling, async patterns, type design)
  - Architecture patterns (transport abstraction, protocol design)
  - Methodology patterns (testing, documentation, versioning)
- **Authority**: These standards OVERRIDE generic guidelines if conflicts occur
- **Integration**: All sub-projects inherit these workspace patterns unless explicitly overridden

### Core Mandatory Patterns (Quick Reference)

**§2.1 3-Layer Import Organization** (ALL Rust files):
```rust
// Layer 1: Standard library imports
use std::collections::HashMap;

// Layer 2: Third-party crate imports  
use serde::{Deserialize, Serialize};

// Layer 3: Internal module imports
use crate::protocol::core::McpMethod;
```

**§3.2 chrono DateTime<Utc> Standard**:
- ALL time operations MUST use `chrono::DateTime<Utc>`
- FORBIDDEN: `std::time::SystemTime` (except `Instant` for performance measuring only)

**§4.3 Module Architecture**:
- `mod.rs` files contain ONLY module declarations and re-exports
- NO implementation code in `mod.rs`

**§5.1 Dependency Management**:
- Workspace dependencies in 3 layers: (1) AirsSys Foundation Crates, (2) Core Runtime, (3) External Dependencies
- Always check `[workspace.dependencies]` before adding new deps

**§6.1 YAGNI Principles**:
- Build only what is currently required
- No speculative features or future-proofing
- Remove unused abstractions immediately

**§6.2 Avoid `dyn` Patterns**:
- Prefer concrete types > generics > `dyn` trait objects
- Use static dispatch for compile-time type safety

**§6.4 Implementation Quality Gates**:
- Zero `unsafe` blocks without justification
- Zero warnings (clippy clean)
- >90% test coverage
- Security logging for all operations

**Full standards documentation**: See `$ROOT_PROJECT/PROJECTS_STANDARD.md` for complete details.

### Memory Bank Integration
- **Multi-Project System**: This project uses the Multi-Project Memory Bank for context management
- **Required Reading**: ALWAYS read workspace + active sub-project Memory Bank files before starting work
- **Active Sub-Project**: Check `$ROOT_PROJECT/.memory-bank/current-context.md` for current focus
- **Context Files**: Each sub-project has dedicated context files (see Memory Bank section below)

---

## Operational Protocols

These instruction files define HOW AI agents should operate within this project:

- **Multi-Project Memory Bank System**: `$ROOT_PROJECT/.aiassisted/instructions/multi-project-memory-bank.instructions.md`  
  CRITICAL: Complete memory bank framework with workspace/sub-project structure, task taxonomy (Task→Phase→Subtask), single-file-per-task mandate, technical documentation templates (ADRs, technical debt, knowledge docs), context snapshots, and validation system. This is the PRIMARY operational framework.

- **Rust Development Standards**: `$ROOT_PROJECT/.aiassisted/instructions/rust.instructions.md`  
  Rust-specific development workflows, code patterns, error handling, async programming, testing strategies, and workspace management for this project.

- **Conventional Commits**: `$ROOT_PROJECT/.aiassisted/instructions/conventional-commits.instructions.md`  
  Git commit message format and standards for this project (follows conventional commits specification).

- **AI Prompt Engineering Safety**: `$ROOT_PROJECT/.aiassisted/instructions/ai-prompt-engineering-safety-best-practices.instructions.md`  
  Best practices for AI-assisted development, prompt safety, and code generation guidelines.

- **Agents Context Setup**: `$ROOT_PROJECT/.aiassisted/instructions/setup-agents-context.instructions.md`  
  Instructions for generating and maintaining this AGENTS.md file (this document).

---

## Guidelines & Standards

These files define code quality standards and documentation frameworks:

### Documentation Guidelines

- **Diataxis Framework**: `$ROOT_PROJECT/.aiassisted/guidelines/documentation/diataxis-guidelines.md`  
  Systematic approach to technical documentation (Tutorials, How-To Guides, Technical Reference, Explanation). All documentation MUST follow this framework.

- **Documentation Quality Standards**: `$ROOT_PROJECT/.aiassisted/guidelines/documentation/documentation-quality-standards.md`  
  Quality criteria for all documentation including code comments, API docs, guides, and examples. Defines review checklists and maintenance schedules.

- **Task Documentation Standards**: `$ROOT_PROJECT/.aiassisted/guidelines/documentation/task-documentation-standards.md`  
  Standards for documenting tasks in the Memory Bank system. Covers task file structure, progress tracking, completion criteria, and update protocols.

### Rust Code Guidelines

- **Microsoft Rust Guidelines**: `$ROOT_PROJECT/.aiassisted/guidelines/rust/microsoft-rust-guidelines.md`  
  Comprehensive Rust coding standards derived from Microsoft's official guidelines. Covers naming conventions, error handling, async patterns, safety, testing, and performance best practices.

---

## Memory Bank System (CRITICAL)

This project uses the **Multi-Project Memory Bank** for context management. Agents MUST understand and follow this system.

### System Architecture

```
.memory-bank/
├── current-context.md              # Active sub-project tracker
├── README.md                       # Memory bank system overview
├── context-snapshots/              # Historical context snapshots
├── templates/                      # Standardized templates
│   └── docs/
│       ├── adr-template.md
│       ├── adr-index-template.md
│       ├── technical-debt-template.md
│       ├── debt-index-template.md
│       ├── knowledge-template.md
│       └── documentation-guidelines.md
├── workspace/                      # Shared workspace context
│   ├── project-brief.md           # Workspace vision, objectives, architecture
│   ├── shared-patterns.md         # Core patterns inherited by all sub-projects
│   ├── workspace-architecture.md  # High-level workspace structure
│   └── workspace-progress.md      # Cross-crate milestones and decisions
└── sub-projects/                   # Individual sub-project contexts
    └── [sub-project-name]/
        ├── project-brief.md       # Sub-project foundation
        ├── product-context.md     # Why this exists, problems solved
        ├── active-context.md      # Current work focus, recent changes
        ├── system-patterns.md     # Architecture, technical decisions
        ├── tech-context.md        # Technologies, setup, constraints
        ├── progress.md            # Status, what works, what's left
        ├── tasks/                 # Task management
        │   ├── _index.md         # Master task list
        │   └── task-*.md         # Individual task files
        └── docs/                  # Technical documentation
            ├── debts/            # Technical debt tracking (PLURAL)
            ├── knowledges/       # Knowledge documentation (PLURAL)
            └── adr/              # Architecture Decision Records (SINGULAR)
```


### Workflow Requirements

**BEFORE Starting ANY Task**:
1. ✅ Read `$ROOT_PROJECT/.memory-bank/current-context.md` to identify active sub-project
2. ✅ Read ALL workspace files in `$ROOT_PROJECT/.memory-bank/workspace/`
3. ✅ Read ALL sub-project context files for the active sub-project
4. ✅ Review task status in `tasks/_index.md` and relevant task files

**DURING Task Execution**:
1. ✅ Update progress in the single canonical task file (`tasks/task-[id]-[name].md`)
2. ✅ Follow the single-file-per-task mandate (NO separate plan/completion/status files)
3. ✅ Use task taxonomy: Task → Phase (optional, >4 weeks) → Subtask (mandatory)
4. ✅ Update both progress tracking table AND progress log
5. ✅ Update `tasks/_index.md` to reflect current status

**AFTER Task Completion**:
1. ✅ Add completion summary to the task file (inline, not separate file)
2. ✅ Update `progress.md` with achievements
3. ✅ Update `active-context.md` with next steps
4. ✅ Create technical documentation if required (ADR, knowledge doc, technical debt)

### Critical Rules

⚠️ **Single File Per Task Mandate**:
- Each task MUST have exactly ONE file: `tasks/task-[id]-[name].md`
- ALL information goes in this file: plan, progress, logs, completion summary
- ❌ FORBIDDEN: Separate plan files, completion files, status files, checkpoint files

⚠️ **Task Taxonomy**:
- **Task** (top level): `[PREFIX]-TASK-###`, duration 1-12 weeks
- **Phase** (optional): Use only if task >4 weeks, max 8 phases per task
- **Subtask** (mandatory): Max 10 per phase, duration <1 week, numbered `Phase.Subtask`

⚠️ **Stale Task Detection**:
- Tasks unchanged for 7+ days MUST be reviewed
- Update status to blocked/pending/abandoned if not actively worked on

### Memory Bank Commands

Agents can use these commands (reference `multi-project-memory-bank.instructions.md`):

- `show-tasks [sub-project] [filter]` - Display tasks
- `add-task [sub-project] [task-name]` - Create new task
- `update-task [sub-project] [task-id]` - Update task file
- `update-memory-bank [sub-project]` - Review and update all context files
- `switch-context [sub-project]` - Change active sub-project
- `save-context [description]` - Create context snapshot
- `show-memory-bank-summary` - Display current state

---

## Current Active Sub-Project

**Status**: Check `$ROOT_PROJECT/.memory-bank/current-context.md` for the currently active sub-project.

**Recent Active Projects**:
- `api-openrouter` - OpenRouter API client library (Rust) - Phase: Planning & Design (Sprint 0)
- `mcp-server-filesystem` - Secure filesystem MCP server (✅ Published v1.0.0-rc.1)
- `protocols-mcp` - Model Context Protocol implementation (✅ Published v1.0.0-rc.1)

**To work on a sub-project**: Always read workspace context + sub-project context files before starting work.

---

## Development Workflow

### For New Features/Tasks

1. **Context Loading**:
   ```bash
   # Check active sub-project
   cat $ROOT_PROJECT/.memory-bank/current-context.md
   
   # Read workspace context
   cat $ROOT_PROJECT/.memory-bank/workspace/*.md
   
   # Read sub-project context
   cat $ROOT_PROJECT/.memory-bank/sub-projects/[active-project]/*.md
   ```

2. **Task Creation**:
   - Create task file: `tasks/task-[id]-[name].md`
   - Use appropriate template (simple vs multi-phase)
   - Update `tasks/_index.md`

3. **Implementation**:
   - Follow `shared-patterns.md` standards
   - Follow Rust guidelines from `.aiassisted/guidelines/rust/`
   - Update task progress regularly

4. **Completion**:
   - Add inline completion summary to task file
   - Update `progress.md` and `active-context.md`
   - Create technical documentation as needed

### For Documentation

1. **Follow Diataxis Framework**: All docs must be Tutorial, How-To, Reference, or Explanation
2. **Use Templates**: Technical debt, knowledge docs, and ADRs use standardized templates
3. **Cross-Reference**: Link related documentation appropriately
4. **Maintain Indexes**: Update `_index.md` files in `docs/` subdirectories

### For Git Commits

1. **Follow Conventional Commits**: Use format from `conventional-commits.instructions.md`
2. **Meaningful Messages**: Explain WHY, not just WHAT
3. **Scope**: Include affected sub-project or workspace component

---

## Key Technologies

- **Language**: Rust 1.88.0+ (2021 edition)
- **Async Runtime**: tokio
- **HTTP**: reqwest, axum
- **Serialization**: serde, serde_json
- **Protocols**: JSON-RPC 2.0, SSE, OAuth2
- **Testing**: tokio-test, wiremock, criterion (benchmarks)
- **Documentation**: rustdoc, MkDocs (material theme)

---

## External Resources

- **MCP Specification**: https://modelcontextprotocol.io/
- **A2A Protocol**: https://a2a-protocol.org/
- **Documentation Site**: https://airsstack.github.io/airsprotocols/
- **Published Crates**:
  - https://crates.io/crates/airsprotocols-mcp
  - https://crates.io/crates/airsprotocols-mcpserver-filesystem

---

## Agent Checklist

Before starting work, ensure you have:

- [ ] Read `$ROOT_PROJECT/.memory-bank/current-context.md`
- [ ] Read ALL workspace context files
- [ ] Read ALL active sub-project context files
- [ ] Reviewed `$ROOT_PROJECT/.memory-bank/workspace/shared-patterns.md`
- [ ] Checked task status in `tasks/_index.md`
- [ ] Understood the single-file-per-task mandate
- [ ] Reviewed relevant guidelines (Rust, documentation, etc.)
- [ ] Understand task taxonomy (Task → Phase → Subtask)

**Remember**: Your memory resets between sessions. The Memory Bank is your ONLY source of truth. Read it EVERY time.

---

**Last Updated**: 2025-12-17  
**Generated By**: Setup instructions from `setup-agents-context.instructions.md`  
**Memory Bank Version**: Multi-Project Memory Bank v2.0 (workspace + sub-projects)
