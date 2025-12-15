# Usage Guide

Common patterns and best practices for using the AIRS MCP Filesystem Server.

## Basic Operations

### Reading Files

```python
# In Claude Desktop
"Read the contents of my README.md file"
```

Claude will use the `read_file` tool to access and display the file contents.

### Writing Files

```python
"Create a new file called notes.txt with 'Hello World'"
```

This triggers the `write_file` tool with approval workflow.

### Listing Directories

```python
"Show me all files in my Documents folder"
```

Uses the `list_directory` tool to browse filesystem.

## Common Workflows

### Code Analysis

```python
"Analyze all TypeScript files in my project for potential bugs"
```

The server will:
1. List files matching `**/*.ts`
2. Read each TypeScript file
3. Analyze code patterns
4. Report findings

### Documentation Generation

```python
"Create API documentation from my source code comments"
```

Workflow:
1. Read source files
2. Extract documentation
3. Generate markdown files
4. Write to docs directory

### Project Setup

```python
"Create a new Next.js project structure"
```

The server will:
1. Create directory structure
2. Generate configuration files
3. Set up initial templates

## Security Patterns

### Safe File Access

Always configure allowed paths:

```toml
[security.filesystem]
allowed_paths = [
    "~/projects/**/*",
    "~/Documents/**/*"
]
```

### Approval Workflows

Enable approval for sensitive operations:

```toml
[security.operations]
write_requires_policy = true
delete_requires_explicit_allow = true
```

## Advanced Patterns

See [Examples & Use Cases](./examples.md) for detailed examples.

See [Integration Guide](./integration.md) for ecosystem integration.
