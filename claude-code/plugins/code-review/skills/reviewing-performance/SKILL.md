---
name: reviewing-performance
description: Detects algorithmic complexity problems, memory leaks, N+1 queries, blocking operations, and hot path inefficiencies. Use when checking for performance issues, optimization opportunities, or latency concerns during code review.
---

# Performance Code Review Skill

Enhancement: Adds hot path identification, database code prioritization, complexity analysis format (Big O), performance-specific false positive adjustments, and detailed performance pattern references.

## Agent

`code-review:performance-agent` (Opus) in thorough mode.

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions: `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`.

## Scope Prioritization

### Hot Path Identification

Prioritize:
- Files with "api", "handler", "controller", "route" in name (entry points)
- Files with "service", "repository", "dao", "query" in name (data access)
- Files with "util", "helper", "common" in name (frequently called)

### Related Database Code

For N+1 and query performance, also read:
- ORM model definitions (`models/*.ts`, `entities/*.cs`)
- Repository implementations
- Migration files (to understand indexes)

## False Positives

Apply `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "False Positive Rules".

**Performance-specific additions** - do NOT flag:
- O(n^2) on small bounded data (max 10 items)
- Startup/initialization code (one-time cost)
- Code with caching/memoization elsewhere
- Theoretical improvements requiring benchmark proof

## Complexity Analysis

Always include:
- **Current**: What is the Big O now?
- **Optimized**: What could it be?
- **Data Scale**: At what data size does this matter?

Example: "Current O(n*m) where n=orders, m=items. With 100 orders and 1000 items, this executes 100,000 comparisons. Optimized O(n+m) would be 1,100 operations."

## Detailed Performance Patterns

`references/performance-patterns.md`

## Example Output

`examples/example-output.md`

