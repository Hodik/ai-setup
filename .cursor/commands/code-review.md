# Code Review

Review code changes for critical issues only.
Not only review the diff itself, but go deep into the context code to understand how it works underneath so that you can find real problems and vulnerabilities.

Focus area (if specified): $ARGUMENTS

## Get Changes

Use git read commands:
- `git status` - see modified files
- `git diff` - unstaged changes
- `git diff --staged` - staged changes
- `git diff main...HEAD` - all branch changes vs main
- `git log -p -n 5` - recent commits with diffs

## Check Patterns

Reference @docs.mdc to understand existing patterns and conventions. Identify if changes break established:
- Architectural patterns
- Design decisions
- Naming conventions, code formatting
- Module relationships

## Analyze Context

Don't just review the diff:
- Read surrounding code to understand context
- Check how changed code is called
- Verify error handling paths and data flow
- Look for downstream impacts

## Review Criteria

Focus on **critical issues only**:
- **Bugs** - Logic errors, null handling, edge cases, off-by-one, race conditions
- **Performance** - N+1 queries, inefficient algorithms, memory leaks, missing indexes
- **Security** - SQL injection, XSS, auth bypasses, data exposure, exposed secrets, missing input validation
- **Correctness** - Business logic errors, data integrity violations, state management, transaction handling
- **Pattern Breaks** - Violations of established architectural patterns, breaks module boundaries

## Output Format

**If critical issues found:**
```
ISSUES FOUND

- [Brief critical issue with file:line]
- [Brief critical issue]
```

**If no critical issues:**
```
APPROVED

No critical issues. Ready to merge.
```

Skip minor style issues, nitpicks, personal preferences, non-critical suggestions, micro-optimizations, formatting.

**Review Priorities**: Security (highest) > Bugs > Performance > Pattern violations > Code quality (lowest, only if critical).

When reporting, be specific: file:line, explain the problem, describe the impact.
