---
name: reviewing-bugs
description: Detects runtime errors, null references, off-by-one errors, boundary conditions, race conditions, and state management issues. Use when hunting for bugs, edge cases, or logical errors during code review.
---

# Bug Detection Code Review Skill

## Agent

`code-review:bug-detection-agent` (Opus) in thorough mode.

## Bug Categories Checked

**Null/Undefined References (Major to Critical):**
- Accessing properties on potentially null objects
- Missing null checks after database lookups
- Optional chaining gaps
- Nullable type misuse

**Off-by-One Errors (Minor to Major):**
- Array index bounds (`<=` vs `<`)
- Fence post errors in counting
- Pagination calculations
- Loop termination conditions

**Async/Promise Issues (Major to Critical):**
- Unhandled promise rejections
- Race conditions between async operations
- Floating promises (missing await)
- TOCTOU (time-of-check to time-of-use)

**Type Coercion Bugs (Major):**
- Loose equality (`==`) vs strict (`===`)
- String/number confusion
- Truthy/falsy misunderstandings
- Type narrowing gaps

**State Management (Major to Critical):**
- Mutating shared objects
- Stale closure captures
- React state update issues
- Redux action misuse

**Error Handling (Major):**
- Swallowed exceptions
- Wrong error type caught
- Missing error propagation
- Incomplete cleanup in finally

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions: `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

## Bug Investigation Mode

When investigating specific bugs:
- Ask for reproduction steps or stack trace
- Focus on code paths mentioned in error
- Include related error handling code
- Read recent git commits touching affected files

## False Positives

Apply `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "False Positive Rules".

**Bug-specific additions** - do NOT flag:
- Guarded elsewhere (null check in caller)
- Framework guarantee (framework ensures non-null)
- Intentional behavior (documented as expected)
- Unreachable conditions (requires impossible state)

## Reproduction Conditions

For each bug:
- **Preconditions**: What state must exist?
- **Trigger**: What action causes the bug?
- **Frequency**: How often can this occur?
- **Impact**: What goes wrong?

Example: "When two users simultaneously withdraw from the same account (concurrent requests), and the balance check passes for both before either write completes, the second write overwrites the first, resulting in only one deduction being recorded."

## Detailed Bug Patterns

`references/common-bugs.md`

## Example Output

`examples/example-output.md`

