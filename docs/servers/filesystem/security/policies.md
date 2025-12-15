# Security Policies

Comprehensive guide to configuring security policies for the AIRS MCP Filesystem Server.

## Overview

Security policies define fine-grained access control rules for filesystem operations. They allow you to:

- Define operation-specific permissions (read, write, delete, etc.)
- Create path-based access rules using glob patterns
- Assign risk levels to different operations
- Require human approval for sensitive operations

## Policy Structure

Each security policy consists of:

```toml
[security.policies.policy_name]
patterns = ["/path/to/files/**/*"]
operations = ["read", "write", "list"]
risk_level = "low"
description = "Description of what this policy covers"
```

### Policy Fields

- **patterns**: Glob patterns matching file paths
- **operations**: List of allowed operations
- **risk_level**: Risk assessment (low, medium, high, critical)
- **description**: Human-readable explanation

## Available Operations

- `read` - Read file contents
- `write` - Create or modify files
- `delete` - Remove files or directories
- `list` - List directory contents
- `create` - Create new directories
- `move` - Move/rename files
- `copy` - Copy files

## Example Policies

### Development Files

```toml
[security.policies.dev_files]
patterns = [
    "~/projects/**/*.rs",
    "~/projects/**/*.toml",
    "~/projects/**/*.md"
]
operations = ["read", "write", "list"]
risk_level = "low"
description = "Development source code and configuration"
```

### Configuration Files

```toml
[security.policies.config_files]
patterns = [
    "~/.config/**/*",
    "~/projects/**/config/**/*"
]
operations = ["read", "list"]
risk_level = "medium"
description = "Configuration files - read-only"
```

### System Files (Restricted)

```toml
[security.policies.system_protected]
patterns = [
    "/etc/**/*",
    "/usr/**/*",
    "/System/**/*"
]
operations = []  # No operations allowed
risk_level = "critical"
description = "System files - completely restricted"
```

## Policy Evaluation

Policies are evaluated in order:

1. **Path matching**: Check if file path matches any policy pattern
2. **Operation check**: Verify requested operation is in allowed list
3. **Risk assessment**: Apply risk-based approval requirements
4. **Approval workflow**: Trigger human approval if required

## Risk Levels

### Low Risk
- Standard development files
- Documentation
- Personal notes
- No approval required

### Medium Risk
- Configuration files
- Shared resources
- May require approval based on settings

### High Risk
- System configuration
- Sensitive data
- Requires approval

### Critical Risk
- System files
- Protected resources
- Blocked by default

## Best Practices

### 1. Principle of Least Privilege

Grant only necessary permissions:

```toml
[security.policies.project_readonly]
patterns = ["~/projects/important/**/*"]
operations = ["read", "list"]  # No write/delete
risk_level = "medium"
```

### 2. Separate Policies by Purpose

Create distinct policies for different use cases:

```toml
# Source code
[security.policies.source_code]
patterns = ["**/*.rs", "**/*.py", "**/*.js"]
operations = ["read", "write"]
risk_level = "low"

# Test files
[security.policies.test_files]
patterns = ["**/tests/**/*", "**/*_test.*"]
operations = ["read", "write", "delete"]
risk_level = "low"
```

### 3. Use Specific Patterns

Prefer specific patterns over wildcards:

```toml
# Good - specific
patterns = ["~/projects/myapp/**/*.rs"]

# Avoid - too broad
patterns = ["**/*"]
```

### 4. Document Policies

Always include clear descriptions:

```toml
[security.policies.api_keys]
patterns = ["**/.env", "**/*.key", "**/secrets/**/*"]
operations = []
risk_level = "critical"
description = "API keys and secrets - no access allowed"
```

## Environment-Specific Policies

### Development Environment

```toml
[security.operations]
write_requires_policy = false
delete_requires_explicit_allow = true

[security.policies.dev_workspace]
patterns = ["~/dev/**/*"]
operations = ["read", "write", "list", "create", "move", "copy"]
risk_level = "low"
```

### Production Environment

```toml
[security.operations]
write_requires_policy = true
delete_requires_explicit_allow = true

[security.policies.prod_readonly]
patterns = ["~/prod/**/*"]
operations = ["read", "list"]  # Read-only in production
risk_level = "high"
```

## Troubleshooting

### "Operation not allowed by policy"

**Cause**: Requested operation not in policy's allowed operations

**Solution**: Add the operation to your policy:
```toml
operations = ["read", "write", "list"]  # Add needed operations
```

### "No matching policy found"

**Cause**: File path doesn't match any policy pattern

**Solution**: Add a policy covering the path:
```toml
[security.policies.new_area]
patterns = ["/path/to/files/**/*"]
operations = ["read"]
risk_level = "medium"
```

## See Also

- [Configuration Guide](../configuration/index.md)
- [Security Model](./index.md)
- [Troubleshooting](../configuration/troubleshooting.md)
