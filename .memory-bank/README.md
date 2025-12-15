# AirsProtocols Memory Bank

⚠️ **CRITICAL: READ BEFORE MAKING CHANGES** ⚠️

This is the **Memory Bank** for the AirsProtocols workspace - a structured documentation system that tracks context, progress, and decisions across all sub-projects.

## 🚨 Important Guidelines

**MUST READ FIRST**: Before making any changes to the Memory Bank, you **MUST** read and follow the instructions in:

📖 **[`.aiassisted/instructions/multi-project-memory-bank.instructions.md`](../.aiassisted/instructions/multi-project-memory-bank.instructions.md)**

This file contains:
- Complete Memory Bank structure and conventions
- Workflow for planning, implementation, and verification
- Task management guidelines
- Documentation standards
- Command reference

## 📁 Structure Overview

```
.memory-bank/
├── README.md                  # This file - entry point with critical warnings
├── current-context.md         # Tracks active sub-project
├── workspace/                 # Workspace-level shared context
│   ├── project-brief.md
│   ├── shared-patterns.md
│   ├── workspace-architecture.md
│   └── workspace-progress.md
├── templates/                 # Documentation templates
│   └── docs/
├── context-snapshots/         # Historical snapshots
└── sub-projects/              # Individual project contexts
    ├── protocols-mcp/
    └── mcp-server-filesystem/
```

## 🎯 Active Sub-Projects

1. **`protocols-mcp`** - Core MCP protocol implementation
   - Protocol message handling, authentication, transport layers
   
2. **`mcp-server-filesystem`** - Filesystem MCP server
   - Security-first filesystem bridge for AI tools

## 🔧 Quick Commands

- **Switch Context**: Update `current-context.md` to change active project
- **View Context**: Check `current-context.md` for current active project
- **Add Task**: Create task file in `sub-projects/[project]/tasks/`
- **Save Snapshot**: Capture current state in `context-snapshots/`

## 📚 Documentation Standards

All Memory Bank files follow:
- **Naming**: `kebab-case` for all files and directories
- **Format**: Markdown for all documentation
- **Updates**: Real-time updates during work
- **Templates**: Use templates from `templates/docs/` for consistency

---

**Remember**: Always refer to the instructions file before making changes!
