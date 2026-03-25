Write comprehensive tests for the specified code/feature/bug fix.

Test target: $ARGUMENTS

## Workflow

1. **MUST ALWAYS Read @docs/testing.md** to understand how to write/execute tests
2. Check for existing tests for the subject - extend them if possible. Only create new test files if none exist or extending doesn't make sense.
3. **Write tests** following the testing pyramid:
   - **Unit Tests**: Test in isolation, must be instant, mock all external deps, follow FIRST principles (Fast, Independent, Repeatable, Self-Validating, Timely)
   - **Integration Tests**: Test workflows, mock only external services, cover critical user journeys
4. **Coverage**: Happy path, error cases, edge cases, permissions, side effects
5. **Run tests** to verify they pass

## After Writing Tests

1. Run newly created tests to verify they pass
2. Check test execution time (unit tests must be instant)
3. If tests fail, fix the code or the test UNTIL THEY PASS
4. Run ALL tests for ALL apps and verify they pass too
5. Never skip/delete/remove failing tests
