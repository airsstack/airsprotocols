# Technical Documentation Guidelines

This document defines when and how to create technical documentation in the Memory Bank system.

## Documentation Types

### 1. Architecture Decision Records (ADRs)

**When to Create**:
- Selecting between technology alternatives
- Choosing architectural patterns that affect multiple components
- Making decisions that impact system scalability, performance, or security
- Changing fundamental design approaches

**Location**: `docs/adr/`
**Template**: Use `adr-template.md`
**Index**: Maintain `docs/adr/_index.md`

### 2. Technical Debt Records

**When to Create**:
- Any `TODO(DEBT)` comment in code
- Architectural shortcuts or compromises
- Incomplete implementations
- Performance issues that need addressing
- Security concerns requiring future work

**Location**: `docs/debts/`
**Template**: Use `technical-debt-template.md`
**Index**: Maintain `docs/debts/_index.md`

### 3. Knowledge Documentation

**When to Create**:
- Complex algorithms or data structures
- Reusable patterns discovered during development
- External API integration details
- Performance optimization techniques
- Domain-specific knowledge that's not obvious
- Migration guides or setup procedures

**Location**: `docs/knowledges/`
**Template**: Use `knowledge-template.md`
**Organization**: Use subdirectories (architecture/, patterns/, performance/, integration/, security/, domain/)

## Quality Standards

### Code Examples

- All code examples MUST compile and run correctly
- Include necessary imports and context
- Use realistic examples, not trivial "foo/bar"
- Add comments explaining non-obvious parts

### Cross-References

- Link to related ADRs, tasks, and debt items
- Reference external documentation when applicable
- Keep links up-to-date during refactoring

### Maintenance

- **ADRs**: Review when revisiting related decisions (typically during major refactors)
- **Technical Debt**: Review monthly or when prioritizing work
- **Knowledge Docs**: Review quarterly or when underlying implementation changes

## Workflow Integration

### During Task Planning

1. Identify if task requires ADR (major architectural decision)
2. Check existing debt registry for related items
3. Plan knowledge doc creation for complex implementations

### During Implementation

1. Create `TODO(DEBT)` comments with corresponding DEBT-### file
2. Document patterns discovered during development
3. Update related knowledge docs if implementation changes

### During Task Completion

1. Create required ADRs for decisions made
2. Document any new technical debt created
3. Write knowledge docs for reusable patterns
4. Update all relevant indexes

### During Code Review

1. Verify all `TODO(DEBT)` comments have corresponding DEBT files
2. Check that complex code has knowledge documentation
3. Ensure ADRs exist for architectural decisions
4. Validate that all indexes are up-to-date

## Naming Conventions

- **ADRs**: `ADR-###-kebab-case-title.md` (sequential numbering)
- **Debt**: `DEBT-###-kebab-case-title.md` (sequential numbering)
- **Knowledge**: `kebab-case-descriptive-title.md` (no numbering)

## Index Maintenance

Each documentation type requires an index file:

- `docs/adr/_index.md`: Chronological list of all ADRs with status
- `docs/debts/_index.md`: Registry of all debt items by priority
- `docs/knowledges/` subdirectories: README.md files organizing knowledge by category

Update indexes whenever creating, updating, or resolving documentation items.

## Review Schedule

### Quarterly Reviews

- All knowledge documentation
- Active technical debt items
- Deprecated ADRs for potential removal

### Monthly Reviews

- High and critical priority technical debt
- Recently created ADRs (first 3 months)

### Annual Reviews

- Complete audit of all documentation
- Archive outdated content
- Restructure categories if needed

## Validation Checklist

Before considering documentation complete:

- [ ] Used correct template
- [ ] All code examples compile
- [ ] Cross-references are valid
- [ ] Index file updated
- [ ] Follows naming conventions
- [ ] Review schedule specified
- [ ] Related items linked
