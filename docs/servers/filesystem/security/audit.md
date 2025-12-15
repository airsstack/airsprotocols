# Audit Logging

Comprehensive audit logging system for tracking filesystem operations and security events.

## Overview

The AIRS MCP Filesystem Server provides detailed audit logging for:

- All filesystem operations
- Security decisions and approvals
- Policy evaluations
- Access violations
- System events

## Log Locations

### Default Locations

- **macOS**: `~/.local/share/airsprotocols-mcpserver-filesystem/logs/`
- **Linux**: `~/.local/share/airsprotocols-mcpserver-filesystem/logs/`
- **Windows**: `%LOCALAPPDATA%\airsprotocols-mcpserver-filesystem\logs\`

### Custom Location

Configure via environment variable:

```bash
export AIRSPROTOCOLS_MCPSERVER_FS_LOG_DIR=/custom/path/logs
```

## Log Format

Logs use JSON format for structured logging:

```json
{
  "timestamp": "2025-12-15T18:30:45.123Z",
  "level": "INFO",
  "correlation_id": "req_abc123",
  "event_type": "file_operation",
  "operation": "read_file",
  "path": "/home/user/document.txt",
  "user": "claude",
  "result": "success",
  "duration_ms": 12
}
```

## Event Types

### Filesystem Operations

```json
{
  "event_type": "file_operation",
  "operation": "write_file",
  "path": "/home/user/test.txt",
  "size_bytes": 1024,
  "result": "success"
}
```

### Security Events

```json
{
  "event_type": "security_validation",
  "operation": "delete_file",
  "path": "/etc/passwd",
  "result": "denied",
  "reason": "path_not_allowed"
}
```

### Approval Workflow

```json
{
  "event_type": "approval_request",
  "operation": "write_file",
  "path": "/home/user/important.txt",
  "decision": "approved",
  "approval_time_ms": 2500
}
```

### Policy Evaluation

```json
{
  "event_type": "policy_evaluation",
  "policy_name": "dev_files",
  "path": "/home/user/code.rs",
  "matched": true,
  "risk_level": "low"
}
```

## Log Levels

### ERROR
Critical errors requiring attention:
- System failures
- Security violations
- Configuration errors

### WARN
Warning conditions:
- Deprecated usage
- Performance issues
- Configuration warnings

### INFO
Normal operations:
- Successful operations
- Approvals granted
- Policy matches

### DEBUG
Detailed diagnostic information:
- Path resolution
- Policy evaluation details
- Internal state changes

## Correlation IDs

Each request is assigned a unique correlation ID to track related events:

```json
{
  "correlation_id": "req_abc123",
  "event_type": "file_operation",
  ...
}
```

Search logs by correlation ID to trace complete request lifecycle.

## Security Audit Trail

### Access Tracking

All file access is logged:

```bash
# Find all access to specific file
grep '/etc/config.toml' *.log

# Find all write operations
grep '"operation":"write_file"' *.log
```

### Violation Detection

Track security violations:

```bash
# Find denied operations
grep '"result":"denied"' *.log

# Find path traversal attempts
grep 'path_traversal' *.log
```

## Log Analysis

### Common Queries

**Failed operations**:
```bash
jq 'select(.result == "failure")' audit.log
```

**High-risk operations**:
```bash
jq 'select(.risk_level == "high")' audit.log
```

**Operations by user**:
```bash
jq 'select(.user == "claude")' audit.log
```

**Operations in time range**:
```bash
jq 'select(.timestamp >= "2025-12-15T00:00:00Z" and .timestamp <= "2025-12-15T23:59:59Z")' audit.log
```

## Log Rotation

Logs are automatically rotated:

- **Max size**: 100MB per file
- **Max files**: 10 rotated files
- **Retention**: 30 days

Configure rotation:

```toml
[logging]
max_file_size_mb = 100
max_backup_files = 10
retention_days = 30
```

## Compliance

### GDPR Compliance

Logs may contain personal data:
- File paths with usernames
- Operation timestamps
- User identifiers

Implement data retention policies as required.

### SOC 2 / ISO 27001

Audit logs support compliance requirements:
- Complete audit trail
- Immutable logging
- Access tracking
- Incident investigation

## Best Practices

### 1. Regular Review

Review logs regularly for:
- Unusual access patterns
- Repeated failures
- Security violations

### 2. Monitoring Alerts

Set up alerts for:
- Multiple failed operations
- Access to sensitive files
- Security policy violations

### 3. Secure Storage

Protect log files:
- Restricted file permissions
- Separate log storage
- Regular backups

### 4. Log Analysis Tools

Use log analysis tools:
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Splunk
- Datadog
- Custom scripts

## Example: Security Investigation

```bash
# 1. Find suspicious activity
grep '"result":"denied"' audit.log | jq .

# 2. Track correlation ID
grep '"correlation_id":"req_abc123"' audit.log | jq .

# 3. Identify patterns
grep '/etc/' audit.log | jq -r '.path' | sort | uniq -c

# 4. Generate report
jq -r '[.timestamp, .operation, .path, .result] | @csv' audit.log > report.csv
```

## See Also

- [Security Model](./index.md)
- [Security Policies](./policies.md)
- [Troubleshooting](../configuration/troubleshooting.md)
