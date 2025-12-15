# Contributing to airsprotocols-mcpserver-filesystem

Thank you for your interest in contributing to the AIRS MCP Filesystem Server! This document provides guidelines and instructions for contributing to this project.

## Code of Conduct

This project adheres to a code of professional conduct. By participating, you are expected to uphold respectful and constructive communication.

## Getting Started

### Development Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/airsstack/airsprotocols.git
   cd airsprotocols/mcp/servers/filesystem
   ```

2. **Install Rust** (if not already installed):
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.up | sh
   rustup update stable
   ```

3. **Build the project**:
   ```bash
   cargo build
   ```

4. **Run tests**:
   ```bash
   cargo test
   ```

### Development Workflow

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** following our code standards

3. **Run the full test suite**:
   ```bash
   # Unit tests
   cargo test --lib
   
   # Integration tests
   cargo test --test '*'
   
   # All tests
   cargo test
   ```

4. **Run code quality checks**:
   ```bash
   # Format code
   cargo fmt
   
   # Run clippy
   cargo clippy -- -D warnings
   
   # Check compilation
   cargo check
   ```

5. **Commit your changes** using conventional commits (see below)

6. **Push and create a pull request**

## Code Standards

### Rust Style Guidelines

We follow the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) and [Microsoft Rust Guidelines](../../.aiassisted/guidelines/rust/microsoft-rust-guidelines.md):

- **Formatting**: Use `cargo fmt` (rustfmt) for all code
- **Linting**: Pass `cargo clippy -- -D warnings` without warnings
- **Error Handling**: Use `Result<T, E>` and `?` operator, avoid `unwrap()` in production code
- **Documentation**: Document all public APIs with rustdoc comments
- **Testing**: Write tests for all new functionality
- **Safety**: Avoid `unsafe` code unless absolutely necessary and well-documented

### Code Organization

```rust
// Layer 1: Standard library imports
use std::path::PathBuf;

// Layer 2: Third-party crate imports
use serde::{Deserialize, Serialize};
use tokio::fs;

// Layer 3a: AIRS foundation crates (prioritized)
use airsprotocols_mcp::protocol::Message;

// Layer 3b: Local crate modules
use crate::security::SecurityManager;
```

### Documentation

- **Public APIs**: Must have comprehensive rustdoc comments
- **Modules**: Should have module-level documentation
- **Examples**: Include examples in doc comments when helpful
- **Safety**: Document any unsafe code thoroughly

### Testing

- **Unit Tests**: Test individual functions and methods
- **Integration Tests**: Test end-to-end workflows
- **Security Tests**: Test security boundaries and validation
- **Edge Cases**: Test error conditions and edge cases

Example test structure:
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_functionality_name() {
        // Arrange
        let input = create_test_input();
        
        // Act
        let result = function_under_test(input);
        
        // Assert
        assert_eq!(result, expected);
    }
}
```

## Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples

```
feat(security): add path traversal protection

Implement comprehensive path validation to prevent directory traversal
attacks. Includes canonicalization, symlink resolution, and sandbox
enforcement.

Closes #123
```

```
fix(config): resolve environment variable override precedence

Environment variables now correctly override configuration file
settings in all cases.
```

## Pull Request Process

1. **Ensure all tests pass**: Run `cargo test` and verify 100% pass rate
2. **Update documentation**: Update README, CHANGELOG, or docs as needed
3. **Add tests**: Include tests for new functionality
4. **Follow code standards**: Ensure code passes `cargo fmt` and `cargo clippy`
5. **Update CHANGELOG**: Add entry to CHANGELOG.md under `[Unreleased]`
6. **Create PR**: Open a pull request with a clear description

### Pull Request Template

```markdown
## Description
Brief description of the changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] All tests passing

## Checklist
- [ ] Code follows project style guidelines
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] No new warnings from clippy
```

## Security

### Reporting Security Issues

**Do NOT create public issues for security vulnerabilities.**

Instead, please email: rstlix.dev@gmail.com

Include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Security Guidelines

When contributing code that affects security:

1. **Follow security best practices** for file system operations
2. **Validate all inputs** thoroughly
3. **Test security boundaries** with dedicated tests
4. **Document security implications** in code comments
5. **Consider threat models** when designing features

## Areas for Contribution

### High Priority

- **Performance improvements**: Optimize file operations
- **Test coverage**: Add more integration and security tests
- **Documentation**: Improve guides and examples
- **Bug fixes**: Address open issues

### Feature Requests

Check existing issues for feature requests or create new ones. Discuss major features before implementing.

### Good First Issues

Look for issues tagged with `good first issue` for beginner-friendly contributions.

## Development Tips

### Running Examples

```bash
cargo run --example configuration_demo
```

### Testing Specific Modules

```bash
cargo test --lib security::
cargo test --test integration_security_audit
```

### Debugging

```bash
RUST_LOG=debug cargo run -- serve
```

### Performance Profiling

```bash
cargo bench
```

## Documentation

### Building Documentation

```bash
# Generate API documentation
cargo doc --no-deps --open

# Build mdBook documentation (if still present)
cd docs && mdbook build && mdbook serve
```

### Documentation Guidelines

- Write clear, concise explanations
- Include code examples
- Link to related documentation
- Keep examples up to date

## Community

### Getting Help

- **GitHub Issues**: For bug reports and feature requests
- **GitHub Discussions**: For questions and discussions
- **Documentation**: Check the comprehensive documentation first

### Communication

- Be respectful and constructive
- Provide context and details
- Help others when you can
- Follow up on your issues and PRs

## License

By contributing, you agree that your contributions will be licensed under the same license as the project (MIT OR Apache-2.0).

All contributions must include the appropriate license header:

```rust
// Licensed under either of Apache License, Version 2.0 or MIT license at your option.
```

## Recognition

Contributors are recognized in:
- Git commit history
- Release notes
- Project documentation

Thank you for contributing to airsprotocols-mcpserver-filesystem! 🎉
