---
description: Write comprehensive tests for code, features, or bug fixes
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

Write comprehensive tests for the specified code or feature.

Test target: $ARGUMENTS

## Workflow

### 1. Read Testing Documentation

Read `docs/testing.md` directly to understand:

- Test structure and organization
- Unit vs Integration test guidelines
- Fixtures and mocking patterns
- How to run tests
- Coverage requirements

### 2. Check Existing Tests

Before creating new tests:

- Search for existing test files for the subject
- If tests exist and make sense to extend, add new test cases
- Only create new test files if none exist or extending doesn't make sense

### 3. Write Comprehensive Tests

Follow project patterns from `docs/testing.md`:

- Use the project's test framework
- Include proper imports and fixtures
- Mock external services and dependencies
- Follow documented testing conventions

### Testing Pyramid

#### Unit Tests (Foundation)

- **Purpose**: Test single function/method/component in isolation
- **Speed**: Must be instant - fast feedback loop
- **Scope**: Pure business logic, calculations, transformations
- **Dependencies**: Mock all external dependencies (databases, APIs, file systems, services)
- **Follow FIRST Principles**:
  - **Fast**: Tests run in milliseconds
  - **Independent**: No dependencies between tests, can run in any order
  - **Repeatable**: Same result every time, no flaky tests
  - **Self-Validating**: Clear pass/fail, no manual inspection needed
  - **Timely**: Written alongside or before implementation code

#### Integration Tests (Top Layer)

- **Purpose**: Test how components work together, full workflows
- **Speed**: Slower, more expensive to run
- **Scope**: End-to-end workflows, API endpoints, database interactions
- **Dependencies**: Mock only external services (3rd party APIs, external systems)
- **Coverage**: Critical user journeys, main business flows

### Test Coverage Checklist

- Happy path (valid inputs, expected behavior)
- Error cases (invalid inputs, constraints)
- Edge cases (null values, empty lists, boundaries)
- Permissions (different user roles if applicable)
- Side effects (database state, async tasks)

### 4. Run Tests

1. Run newly created tests to verify they pass
2. Check test execution time (unit tests must be instant)
3. If tests fail, fix the code or test until they pass
4. Run ALL tests for ALL apps and verify they pass too
5. Never skip/delete/remove failing tests even if they are not related to changes made
