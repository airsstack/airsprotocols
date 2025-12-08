# Contributing to AIRS Protocols

Thank you for your interest in contributing to AIRS Protocols! This guide will help you get started.

## Ways to Contribute

There are many ways to contribute to AIRS Protocols:

- 🐛 **Report Bugs**: Find and report issues
- 💡 **Suggest Features**: Propose new capabilities
- 📝 **Improve Documentation**: Fix typos, clarify instructions, add examples
- 🔧 **Write Code**: Fix bugs, implement features, optimize performance
- 🧪 **Write Tests**: Add test coverage, create integration tests
- 💬 **Help Others**: Answer questions in discussions

## Getting Started

### Prerequisites

- **Rust** 1.88 or later
- **Git** for version control
- **Cargo** (comes with Rust)
- A GitHub account

### Development Setup

1. **Fork the Repository**

   Visit [github.com/airsstack/airsprotocols](https://github.com/airsstack/airsprotocols) and click "Fork"

2. **Clone Your Fork**

   ```bash
   git clone https://github.com/YOUR_USERNAME/airsprotocols.git
   cd airsprotocols
   ```

3. **Add Upstream Remote**

   ```bash
   git remote add upstream https://github.com/airsstack/airsprotocols.git
   ```

4. **Install Development Tools**

   ```bash
   # Install rustfmt for code formatting
   rustup component add rustfmt
   
   # Install clippy for linting
   rustup component add clippy
   
   # Install mdBook for documentation (optional)
   cargo install mdbook
   ```

5. **Build the Project**

   ```bash
   cargo build
   ```

6. **Run Tests**

   ```bash
   cargo test --workspace
   ```

## Development Workflow

### 1. Create a Branch

Create a branch for your changes:

```bash
git checkout -b feature/your-feature-name
```

Branch naming conventions:

- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test additions or modifications

### 2. Make Changes

Follow our coding standards (see below) and make your changes.

### 3. Test Your Changes

```bash
# Run all tests
cargo test --workspace

# Run specific package tests
cargo test -p airsprotocols-mcp

# Run with output
cargo test -- --nocapture

# Run integration tests
cargo test --test '*' -- --test-threads=1
```

### 4. Format and Lint

```bash
# Format code
cargo fmt --all

# Run clippy
cargo clippy --workspace --all-targets -- -D warnings
```

### 5. Commit Your Changes

Follow conventional commit message format:

```
type(scope): short description

Longer description if needed.

Fixes #issue_number
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Test additions/modifications
- `chore`: Maintenance tasks

Example:

```bash
git add .
git commit -m "feat(mcp): add OAuth2 token refresh support

Implements automatic token refresh for OAuth2 authentication
to improve user experience and reduce authentication failures.

Fixes #123"
```

### 6. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then create a pull request on GitHub.

## Coding Standards

### Rust Code Style

Follow these guidelines:

1. **Use `rustfmt`**: All code must be formatted with `cargo fmt`

2. **Follow Clippy**: Address all clippy warnings with `cargo clippy`

3. **Document Public APIs**: All public items must have documentation comments

   ```rust
   /// Describes what this function does.
   ///
   /// # Arguments
   ///
   /// * `param` - Description of parameter
   ///
   /// # Returns
   ///
   /// Description of return value
   ///
   /// # Errors
   ///
   /// Description of error conditions
   pub fn my_function(param: Type) -> Result<ReturnType, Error> {
       // implementation
   }
   ```

4. **Use Explicit Error Handling**: Avoid `unwrap()` and `expect()` in library code

   ```rust
   // Bad
   let value = some_result.unwrap();
   
   // Good
   let value = some_result?;
   // or
   let value = some_result.map_err(|e| MyError::from(e))?;
   ```

5. **Follow Workspace Lint Rules**: The workspace defines mandatory lints

   ```rust
   // These are denied in production code:
   // - unwrap_used
   // - expect_used
   // - panic
   ```

### Testing Standards

1. **Write Tests for New Code**: All new features should include tests

2. **Test Public APIs**: Focus on testing the public interface

3. **Use Descriptive Test Names**:

   ```rust
   #[test]
   fn test_tool_provider_returns_correct_result() {
       // test implementation
   }
   ```

4. **Include Integration Tests**: For cross-component functionality

5. **Use Property-Based Testing**: For complex algorithms (using proptest)

### Documentation Standards

1. **Clear and Concise**: Write for your audience

2. **Include Examples**: Show how to use the API

   ```rust
   /// # Example
   ///
   /// ```rust
   /// let client = McpClientBuilder::new()
   ///     .build(transport)
   ///     .await?;
   /// ```
   ```

3. **Update Related Docs**: Keep documentation synchronized with code

4. **Follow Diataxis Framework**: Use the documentation guidelines in `.aiassisted/guidelines/documentation/`

## Pull Request Process

### Before Submitting

- [ ] Code is formatted (`cargo fmt`)
- [ ] All tests pass (`cargo test`)
- [ ] No clippy warnings (`cargo clippy`)
- [ ] Documentation is updated
- [ ] Commit messages follow conventions
- [ ] Branch is up to date with main

### PR Description

Include in your PR description:

1. **What**: What does this PR do?
2. **Why**: Why is this change needed?
3. **How**: How does it work?
4. **Testing**: How was it tested?
5. **Issues**: References to related issues

Example:

```markdown
## What

Adds OAuth2 token refresh support to the MCP HTTP client.

## Why

Users were experiencing authentication failures when tokens expired,
requiring manual re-authentication.

## How

Implements automatic token refresh using the refresh token. The client
now monitors token expiration and proactively refreshes before expiry.

## Testing

- Added unit tests for token refresh logic
- Added integration test with mock OAuth2 server
- Tested with Claude Desktop integration

## Issues

Fixes #123
Related to #456
```

### Review Process

1. **Automated Checks**: CI runs tests and linting
2. **Code Review**: Maintainers review your code
3. **Address Feedback**: Make requested changes
4. **Approval**: PR is approved
5. **Merge**: Maintainers merge your PR

## Code Review Guidelines

### For Contributors

- **Be Receptive**: Accept feedback gracefully
- **Ask Questions**: If feedback is unclear, ask for clarification
- **Make Changes**: Address review comments promptly
- **Learn**: Use reviews as learning opportunities

### For Reviewers

- **Be Respectful**: Provide constructive feedback
- **Be Specific**: Explain why changes are needed
- **Praise Good Work**: Acknowledge good solutions
- **Focus on Code**: Review the code, not the person

## Development Guidelines

### Architecture

Follow the established architecture patterns:

- **Integration Layer**: High-level APIs
- **Provider Layer**: Capability implementations
- **Protocol Layer**: Message types and validation
- **Transport Layer**: Communication abstractions

See [Architecture Guide](architecture.md) for details.

### Error Handling

Use Result types and custom error types:

```rust
#[derive(Debug, thiserror::Error)]
pub enum MyError {
    #[error("connection failed: {0}")]
    ConnectionFailed(String),
    
    #[error("protocol error: {0}")]
    ProtocolError(String),
}
```

### Async Code

All I/O should be async:

```rust
#[async_trait]
pub trait MyTrait: Send + Sync {
    async fn my_method(&self) -> Result<T, Error>;
}
```

## Community Guidelines

### Code of Conduct

Be respectful, inclusive, and professional:

- **Be Welcoming**: Welcome newcomers
- **Be Respectful**: Respect differing viewpoints
- **Be Constructive**: Provide constructive criticism
- **Be Patient**: Help others learn

### Getting Help

- **Documentation**: Check the docs first
- **Discussions**: Ask in [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)
- **Issues**: Report bugs in [GitHub Issues](https://github.com/airsstack/airsprotocols/issues)

## Project Structure

```
airsprotocols/
├── protocols/          # Protocol implementations
│   ├── mcp/           # Model Context Protocol
│   └── a2a/           # Agent-to-Agent (planned)
├── apis/              # LLM APIs (planned)
├── docs/              # Documentation source
├── site-mkdocs/       # MkDocs configuration
└── .aiassisted/       # Development guidelines
```

## Release Process

Maintainers handle releases, but here's the process:

1. **Version Bump**: Update version in `Cargo.toml`
2. **Changelog**: Update `CHANGELOG.md`
3. **Tag**: Create git tag
4. **Publish**: Publish to crates.io
5. **Announce**: Announce release

## Recognition

Contributors are recognized in:

- Repository contributor list
- Release notes
- Documentation acknowledgments

## Questions?

If you have questions about contributing:

- Check this guide
- Ask in [GitHub Discussions](https://github.com/airsstack/airsprotocols/discussions)
- Review existing PRs for examples
- Contact maintainers

---

Thank you for contributing to AIRS Protocols! 🦀
