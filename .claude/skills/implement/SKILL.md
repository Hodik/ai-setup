---
description: Implement a feature or code change with tests and quality checks
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

Implement the following feature/change with full quality checks:

$ARGUMENTS

## Phase 1: Read Documentation First

**MUST read** these documentation files before implementing:

- `docs/conventions.md` - Naming patterns, file structure, standard practices
- `docs/formatting.md` - Code style, formatting rules, import organization
- `docs/system_patterns.md` - Architectural patterns, design decisions
- `docs/tech_context.md` - Technologies, constraints, dependencies
- `docs/testing.md` - Testing requirements and patterns

## Phase 2: Implement the Feature

### Follow Established Patterns

- Don't invent new patterns when existing ones work
- Match the style and structure of similar code
- Use existing utilities and abstractions
- Respect module boundaries and dependencies

### Code Quality Standards

- **Readability**: Code is simple and clear
- **Naming**: Functions and variables are well-named following `docs/conventions.md`
- **DRY**: No duplicated code
- **Error Handling**: Proper error handling and validation
- **Security**: No exposed secrets, proper input validation
- **Performance**: Consider efficiency and scalability

### Documentation in Code

- Clear function/class docstrings
- Comments for complex logic
- Type hints where beneficial

## Phase 3: Write & Run Tests

1. **Read `docs/testing.md`** to understand project test patterns
2. **Check existing tests** for the subject - extend them if possible. Only create new test files if none exist or extending doesn't make sense.
3. **Write comprehensive tests** following the testing pyramid:

### Unit Tests (Foundation)

- Test single function/method/component in isolation
- Must be instant - fast feedback loop
- Mock all external dependencies (databases, APIs, file systems, services)
- **FIRST Principles**: Fast, Independent, Repeatable, Self-Validating, Timely

### Integration Tests (Top Layer)

- Test how components work together, full workflows
- Mock only external services (3rd party APIs, external systems)
- Cover critical user journeys, main business flows

### Test Coverage Checklist

- Happy path (valid inputs, expected behavior)
- Error cases (invalid inputs, constraints)
- Edge cases (null values, empty lists, boundaries)
- Permissions (different user roles if applicable)
- Side effects (database state, async tasks)

4. **Run newly created tests** to verify they pass
5. **If tests fail**, fix the code or test until they pass
6. **Run ALL tests** to ensure no regressions
7. **Never skip or delete failing tests**

## Phase 4: Quality Checks

- Run linter and fix any errors
- Run formatters if documented in `docs/formatting.md`
- Quick self-review for critical issues:
  - Logic errors, null handling, edge cases, off-by-one errors, race conditions
  - Security vulnerabilities (SQL injection, XSS, auth bypasses, data exposure, missing input validation)
  - Performance issues (N+1 queries, memory leaks, unnecessary database hits, missing indexes)
  - Correctness (data integrity, state management, transaction handling)
  - Pattern violations against `docs/system_patterns.md`
- Ensure no debug code or comments left behind
- Confirm proper error handling
- Validate input/output contracts

## Completion Criteria

Implementation is complete when:

1. Feature/change is fully implemented
2. Code follows all documented standards
3. Tests are written and passing
4. Linter and formatter checks pass
5. No debug code remains
