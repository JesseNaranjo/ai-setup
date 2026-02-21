---
name: reviewing-bugs
description: Detects runtime errors, null references, off-by-one errors, boundary conditions, race conditions, and state management issues. Use when hunting for bugs, edge cases, or logical errors during code review.
---

# Bug Detection Code Review Skill

Adds bug investigation mode, reproduction conditions framework, bug-specific FP adjustments, common bug pattern references.

## Bug Investigation Mode

When reviewing code for bugs:
- Check related error handling code
- Read recent git commits touching affected files
- Use Grep to trace data flow through suspicious paths
- Verify boundary conditions in loops, array access, and arithmetic

## False Positives

**Bug-specific** - do NOT flag:
- Guarded elsewhere (null check in caller)
- Framework guarantee (ensures non-null)
- Intentional behavior (documented)
- Unreachable conditions (requires impossible state)

## Reproduction Conditions

For each bug: **Preconditions** (required state), **Trigger** (causing action), **Frequency** (likelihood), **Impact** (consequence).

Example: "When two users simultaneously withdraw from the same account (concurrent requests), and the balance check passes for both before either write completes, the second write overwrites the first, resulting in only one deduction being recorded."

## Detailed Bug Patterns

`references/common-bugs.md`
