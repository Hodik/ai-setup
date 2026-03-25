---
description: Review code changes for critical security, performance, and correctness issues
allowed-tools: Read, Grep, Glob, Bash
---

Review code changes for critical issues.

Focus area (if specified): $ARGUMENTS

## Workflow

### 1. Get Changes

Use git read commands to see what changed:

```bash
git status                 # See modified files
git diff                   # Unstaged changes
git diff --staged          # Staged changes
git diff main...HEAD       # All branch changes vs main
git log -p -n 5           # Recent commits with diffs
```

### 2. Understand Documented Patterns

Read documentation files directly to understand existing patterns and conventions:

- `docs/system_patterns.md` - Architectural patterns and design decisions
- `docs/conventions.md` - Naming conventions, file structure, standard practices
- `docs/formatting.md` - Code style and formatting rules
- `docs/tech_context.md` - Technical constraints and dependencies

### 3. Review Against Standards

Identify if changes break established:

- Architectural patterns
- Design decisions
- Naming conventions
- Code formatting standards
- Module relationships
- Technical constraints

### 4. Check for Critical Issues

Focus on **critical issues only**:

**Bugs**:
- Logic errors
- Null/undefined handling
- Edge cases not handled
- Off-by-one errors
- Race conditions

**Performance**:
- N+1 queries
- Inefficient algorithms
- Memory leaks
- Unnecessary database hits
- Missing indexes

**Security**:
- SQL injection vulnerabilities
- XSS vulnerabilities
- Authentication bypasses
- Authorization issues
- Data exposure
- Exposed secrets or API keys
- Missing input validation

**Correctness**:
- Business logic errors
- Data integrity violations
- State management issues
- Transaction handling

**Pattern Breaks**:
- Violations of established architectural patterns
- Inconsistent with documented conventions
- Breaks module boundaries
- Introduces tech debt

### 5. Analyze Context

Don't just review the diff itself:

- Read surrounding code to understand context
- Check how changed code is called
- Verify error handling paths
- Understand data flow through the change
- Look for downstream impacts

## Output Format

### If Critical Issues Found

```
ISSUES FOUND

- [Brief critical issue description with file:line reference]
- [Another critical issue with specific details]
- [Pattern violation with explanation]
```

### If No Critical Issues

```
APPROVED

No critical issues. Ready to merge.
```

## What NOT to Report

**Skip these**:

- Minor style issues (handled by formatters)
- Nitpicks about variable names (unless genuinely confusing)
- Personal preferences
- Non-critical suggestions
- Micro-optimizations
- Formatting inconsistencies (handled by tools)

## Review Priorities

1. **Security vulnerabilities** - Highest priority
2. **Bugs that will cause failures** - High priority
3. **Performance issues** - Medium priority
4. **Pattern violations** - Medium priority
5. **Code quality concerns** - Low priority (only if critical)

## Be Specific

When reporting issues:

- Reference specific file and line numbers
- Explain the problem clearly
- Describe the potential impact
- Suggest how to verify the issue
