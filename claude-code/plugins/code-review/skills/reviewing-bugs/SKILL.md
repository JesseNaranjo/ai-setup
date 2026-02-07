---
name: reviewing-bugs
description: Use when user mentions "find bugs", "check for bugs", "review for errors", "find logical errors", "check for null references", "find edge cases", "check for race conditions", or "debug this code".
---

# Bug Detection Code Review Skill

Identify logical errors, null reference issues, race conditions, off-by-one errors, and other potential bugs through targeted bug-focused code review.

## Agent

`code-review:bug-detection-agent` (Opus)

Uses thorough mode with focus areas below.

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

---

## Auto-Validated Patterns

High-confidence patterns that skip validation. For full definitions, see `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`.

---

## Bug Investigation Mode

When investigating a specific bug:
- Ask for reproduction steps or stack trace
- Focus on code paths mentioned in error
- Include related error handling code
- Read recent git commits touching affected files

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` "False Positive Rules" section.

**Bug-specific additions** - do NOT flag:
- Guarded elsewhere (null check in caller)
- Framework guarantee (framework ensures non-null)
- Intentional behavior (documented as expected)
- Unreachable conditions (requires impossible state)

---

## Reproduction Conditions

For each bug, describe:

- **Preconditions**: What state must exist?
- **Trigger**: What action causes the bug?
- **Frequency**: How often can this occur?
- **Impact**: What goes wrong?

Example: "When two users simultaneously withdraw from the same account (concurrent requests), and the balance check passes for both before either write completes, the second write overwrites the first, resulting in only one deduction being recorded."

---

## Detailed Bug Patterns

For comprehensive code examples and detection patterns by category, see `references/common-bugs.md`.

---

## Example Output

See `examples/example-output.md` for a sample showing:
- Race condition with transaction fix
- Null reference with optional chaining fix
- Unhandled promise with try/catch fix

