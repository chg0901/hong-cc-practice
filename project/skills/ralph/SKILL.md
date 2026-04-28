---
name: ralph
description: |
  Persistent iterative loop for well-defined tasks with clear completion criteria.
  Use when you need TDD cycles, batch refactoring, or getting tests to pass iteratively.
  Trigger: "ralph", "loop until done", "keep going until", "iterate until all tests pass".
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Grep
  - Glob
---

# Ralph — Persistent Iterative Loop

A self-referential development loop inspired by the Ralph Wiggum technique. Unlike `/loop` (timer-based wakeup), Ralph is **completion-condition-driven**: keep iterating until specific criteria are met.

## When to Use

- **TDD Cycles**: Write failing test → implement → run → fix → repeat until all pass
- **Batch Refactoring**: Apply same change pattern across many files
- **Large-Scale Code Cleanup**: Remove deprecated patterns, fix linting errors
- **Test Suite Debugging**: Run tests → identify failures → fix → re-run → repeat

## When NOT to Use

- Tasks requiring human judgment or design decisions
- One-shot operations (write once, done)
- Tasks with unclear success criteria
- Production debugging (use systematic-debugging instead)

## Workflow

### Step 1: Define the Loop

```
Task: <what to accomplish>
Completion Promise: <exact string that signals done>
Max Iterations: <safety cap, default 20>
Verification: <how to check progress each iteration>
```

### Step 2: Execute Iterations

For each iteration:
1. Assess current state (read files, run tests, check git status)
2. Identify remaining work
3. Make one focused change
4. Verify the change (run test/lint/build)
5. Check if completion promise is met
6. If met → output `<promise>COMPLETE</promise>` and stop
7. If max iterations reached → report progress and blockers

### Step 3: Report Progress

After each iteration, briefly report:
```
Iteration N/M | Done: X | Remaining: Y | Status: on-track|blocked
```

## Completion Promise Format

In your final output, include:
```
<promise>COMPLETE</promise>
```

The user (or a future hook) can detect this to know the loop finished successfully.

## Safety Rules

1. **Always set max iterations** — prevent infinite loops on impossible tasks
2. **One change per iteration** — focused, atomic modifications
3. **Verify after each change** — don't accumulate untested modifications
4. **Report blockers immediately** — if stuck 3+ iterations, explain why
5. **Never modify files outside scope** — stay within defined boundaries

## Examples

### TDD Loop
```
Task: Implement and pass all tests in test_codes/test_xxx.py
Completion Promise: ALL_TESTS_PASS
Max Iterations: 15
Verification: Run pytest after each change
```

### Refactoring Loop
```
Task: Replace all `os.getenv("VAR", "default")` with `os.getenv("VAR")` + explicit error
Completion Promise: NO_HARDCODED_DEFAULTS
Max Iterations: 10
Verification: grep for pattern after each change
```

## Integration with Five-Layer System

| Component | Integration |
|-----------|------------|
| deviation-handling.md | Auto-fix bugs found during iteration (Rule 1) |
| test-runner agent | Can dispatch test execution |
| context-management.md | Compact between iterations if context grows |
| systematic-debugging | Switch to this if stuck for 3+ iterations |

## Difference from /loop

| Feature | /loop | Ralph |
|---------|-------|-------|
| Trigger | Timer-based (interval) | Completion-condition |
| Duration | Until cancelled | Until done or max iterations |
| Use case | Monitoring, polling | Implementation, fixing |
| Context | Fresh each wake | Continuous within session |
