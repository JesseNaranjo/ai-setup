---
name: reviewing-performance
description: Use when user mentions "check performance", "review for performance issues", "find slow code", "optimize", "check for memory leaks", "find N+1 queries", "check complexity", "profile code", or "latency issues".
---

# Performance Code Review Skill

Identify algorithmic inefficiencies, memory leaks, database query problems, and other performance bottlenecks through targeted performance-focused code review.

## Agent

`code-review:performance-agent` (Opus)

Uses thorough mode with focus areas below.

## Performance Categories Checked

**Algorithmic Complexity (Major to Critical):**
- O(n^2) nested loops that could be O(n)
- Repeated expensive operations inside loops
- Unnecessary full collection iterations
- Inefficient sorting or searching

**Database Performance (Critical):**
- N+1 query problems (query inside loop)
- Missing indexes on filtered/joined columns
- Unbounded SELECT queries
- Inefficient joins or subqueries

**Memory Issues (Critical):**
- Memory leaks from unreleased resources
- Growing caches without eviction
- Event listeners not removed
- Large object allocations in hot paths

**Async/Await Patterns (Major):**
- Sequential awaits for independent operations
- Blocking event loop with sync operations
- Missing Promise.all for parallel work
- Floating promises (no await or catch)

**Language-Specific Issues:**

*Node.js/TypeScript:*
- Blocking event loop with sync I/O
- String concatenation in loops
- Unnecessary array copies (spread in loops)

*.NET/C#:*
- Boxing/unboxing in hot loops
- LINQ allocations in frequently called code
- async void instead of async Task
- Missing ConfigureAwait in libraries

---

## Auto-Validated Patterns

High-confidence patterns that skip validation. For full definitions, see `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`.

---

## Scope Prioritization

### Hot Path Identification

Automatically prioritize:
- Files with "api", "handler", "controller", "route" in name (entry points)
- Files with "service", "repository", "dao", "query" in name (data access)
- Files with "util", "helper", "common" in name (frequently called)

### Related Database Code

For N+1 and query performance detection, also read:
- ORM model definitions (`models/*.ts`, `entities/*.cs`)
- Repository implementations
- Migration files (to understand indexes)

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section.

**Performance-specific additions** - do NOT flag:
- O(n^2) on small bounded data (max 10 items)
- Startup/initialization code (one-time cost)
- Code with caching/memoization elsewhere
- Theoretical improvements requiring benchmark proof

---

## Complexity Analysis

Always include when relevant:

- **Current**: What is the Big O now?
- **Optimized**: What could it be?
- **Data Scale**: At what data size does this matter?

Example: "Current O(n*m) where n=orders, m=items. With 100 orders and 1000 items, this executes 100,000 comparisons. Optimized O(n+m) would be 1,100 operations."

---

## Detailed Performance Patterns

For comprehensive code examples and detection patterns by category, see `references/performance-patterns.md`.

---

## Example Output

See `examples/example-output.md` for a sample showing:
- N+1 query with diff fix and complexity analysis
- Sequential awaits with Promise.all fix
- Unbounded query with pagination prompt

