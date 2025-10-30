---
description: Write comprehensive tests for specified code, feature, or bug fix
category: testing
argument-hint: <test_subject>
---

# Test Command

This command invokes the `tester` subagent to create comprehensive, high-quality tests following project standards.

## Usage

```
/test <test_subject>
```

## What It Does

The tester agent will:

1. **Read Testing Docs** - Understand project test patterns from `docs/testing.md`
2. **Check Existing Tests** - Look for tests to extend rather than duplicate
3. **Write Comprehensive Tests** - Cover happy path, errors, edge cases, permissions
4. **Run Tests** - Execute and verify they pass
5. **Fix Issues** - If tests fail, fix until they pass
6. **Verify All Tests** - Ensure no regressions in existing test suite

## Test Coverage

The tester ensures coverage of:
- **Happy path** - Valid inputs, expected behavior
- **Error cases** - Invalid inputs, constraint violations
- **Edge cases** - Null values, empty lists, boundaries
- **Permissions** - Different user roles (if applicable)
- **Side effects** - Database state, async tasks

## Example

```
/test UserProfile endpoint with avatar upload
```

The tester will:
- Read `docs/testing.md` for patterns
- Check for existing UserProfile tests
- Write unit tests for avatar validation logic
- Write integration tests for endpoint CRUD operations
- Mock external services (file storage)
- Test error cases (invalid file types, size limits)
- Run all tests and verify they pass

## Test Types

**Unit Tests**:
- Purpose: Test single function/method in isolation
- Speed: Instant (< 100ms per test)
- Mocking: Mock all external services
- Location: `tests/unit/test_<module>.py`

**Integration Tests**:
- Purpose: Test full workflows through API
- Speed: Slower, uses real database
- Mocking: Only external services
- Location: `tests/integration/test_<feature>.py`

## Quality Standards

The tester enforces:
- Proper pytest structure with fixtures
- Mocking of external services (Google, Asana, WordPress, APIs)
- Multi-database marking if required
- Faker for test data
- Never skip or delete failing tests
- All tests must pass before completion

## When to Use

Use this command when:
- You want explicit test creation for a feature
- You've implemented code and need tests
- You're practicing TDD and want tests first
- You need to extend test coverage

For most implementation work, the `implementer` agent will automatically invoke the tester. Use `/test` when you want focused test creation without implementation.

## After Testing

The tester ensures:
- All newly created tests pass
- Execution time is appropriate (unit tests instant)
- No existing tests are broken
- Code coverage meets standards
- Tests document expected behavior

Tests are your safety net - the tester makes sure it's strong.

