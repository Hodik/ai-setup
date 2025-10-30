---
description: Find and prove the root cause of a bug through test-driven diagnosis
category: debugging
argument-hint: <bug_description>
---

# Debug Command

This command invokes the `debugger` subagent to diagnose bugs through systematic, test-driven investigation.

## Usage

```
/debug <bug_description>
```

## What It Does

The debugger agent will:

1. **Identify** - Analyze code to find the most likely cause
2. **Prove** - Write a unit test that demonstrates the issue
3. **Run** - Execute the test to confirm the hypothesis
4. **Iterate** - If test doesn't prove it, continue until root cause is found
5. **Report** - Provide structured summary with evidence

## Important: Diagnosis Only

The debugger **does NOT fix bugs**. It only:
- Finds the root cause
- Proves it with a failing test
- Reports the evidence

This separation ensures clear diagnosis before implementation.

## Example

```
/debug API endpoint returns 500 error for users without profiles
```

The debugger will:
- Examine the code path for that endpoint
- Form hypothesis about null reference to profile
- Write test calling endpoint with user lacking profile
- Run test to confirm it produces 500 error
- Report:
  ```
  ROOT CAUSE: NoneType error when user.profile is None
  TEST RESULT: Test fails with 500 error as expected
  EVIDENCE: Serializer accesses profile.email without null check
  ```

## Output Format

```
ROOT CAUSE: [Brief description of the actual problem]

TEST RESULT: [Pass/Fail status - should fail if bug is proven]

EVIDENCE: [1-2 sentences explaining how the test proves the cause]
```

## When to Use

Use this command when:
- You have a bug that needs systematic diagnosis
- You want test-driven proof of the root cause
- You want to ensure the problem is clearly identified before fixing
- You need reproducible evidence of the issue

After the debugger proves the root cause, you can use `/implement` to fix it, confident in your understanding of the problem.

The debugger ensures bugs are properly understood before solutions are attempted.

