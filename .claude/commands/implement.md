---
description: Implement the requested feature or code change following all project standards and conventions
category: development
argument-hint: <feature_description>
---

# Implement Command

This command invokes the `implementer` subagent to handle feature implementation with full adherence to project standards.

## Usage

```
/implement <feature_description>
```

## What It Does

The implementer agent will:

1. **Read Documentation** - Understand all project patterns, conventions, and standards
2. **Implement Feature** - Write code following documented patterns
3. **Create Tests** - Invoke tester agent for comprehensive test coverage
4. **Quality Checks** - Run linter, formatter, and verify tests pass
5. **Update Docs** - Document new patterns if introduced

## Example

```
/implement Add user profile endpoint with avatar upload
```

The implementer will:
- Read docs for endpoint patterns, naming conventions, file structure
- Create ViewSet, serializer, and URLs following conventions
- Implement avatar upload with proper validation
- Create comprehensive tests (unit + integration)
- Run quality checks
- Document any new patterns introduced

## When to Use

Use this command when you want explicit control over when implementation begins. For most tasks, you can simply describe what you want and Claude will automatically delegate to the implementer agent if appropriate.

This command is useful when:
- You want to clearly signal "now implement this"
- You have a specific implementation request
- You want to ensure the full implementation workflow is followed

The implementer ensures your code is production-ready and follows all project standards.

