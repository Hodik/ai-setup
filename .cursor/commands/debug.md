**Your task**: Find and prove the root cause. **Do NOT fix it yet.**

Bug description: $ARGUMENTS

## Process

1. **Debug** - Examine code paths, data flows, and state. Form a clear hypothesis about the root cause.
2. **Prove** - Write a unit test (follow @docs/testing.md) that isolates the buggy behavior, demonstrates the failure clearly, and is runnable and reproducible.
3. **Run** - Execute the test to confirm your hypothesis. If the test passes (doesn't prove the bug), your hypothesis is wrong — revise and try again.
4. **Report** - Respond with a short summary only. Do NOT propose fixes.

## Output Format

```
ROOT CAUSE: [Brief description of the actual problem]

TEST RESULT: [Pass/Fail status - should fail if bug is proven]

EVIDENCE: [1-2 sentences explaining how the test proves the cause]
```

## Guidelines

- **Systematic Approach**: Narrow down possibilities through code examination
- **Test-Driven Diagnosis**: Tests are your proof mechanism
- **Iterate If Needed**: If test doesn't prove it, revise hypothesis and test again
- **No Fixing**: Your job ends at proving the cause, not solving it
- **Clear Communication**: Make the root cause obvious for whoever implements the fix

**Diagnosis without prescription.** You find what's broken and prove it's broken. Someone else will fix it.
