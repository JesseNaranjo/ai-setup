---
name: performance-review
description: This skill should be used when the user asks to "check performance", "review for performance issues", "find slow code", "optimize", "check for memory leaks", "find N+1 queries", "check complexity", "profile code", "latency issues", or mentions improving code performance.
version: 3.1.3
---

# Performance Code Review Skill

Identify algorithmic inefficiencies, memory leaks, database query problems, and other performance bottlenecks through targeted performance-focused code review.

## Performance-Specific Configuration

### Agent Parameters

- **Agent:** `${CLAUDE_PLUGIN_ROOT}/agents/performance-agent.md`
- **Model:** Opus (for deep performance analysis)
- **Modes:** thorough (first pass), gaps (second pass)

### Performance Categories Checked

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

These high-confidence patterns skip validation:

| Pattern | Description |
|---------|-------------|
| `n_plus_one_query` | Loop + await + findOne/findById/query |
| `nested_loop_includes` | Nested for + `.includes()`/`.indexOf()` |
| `select_star_in_loop` | Loop + `SELECT *` |

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

## Performance-Specific False Positives

Do NOT flag:
- O(n^2) on max 10 items (not a problem)
- Startup/initialization code (one-time cost)
- Debug/logging code (functionality > performance)
- Code with caching/memoization elsewhere
- Test code (doesn't need optimization)
- Theoretical improvements requiring benchmark proof

---

## Complexity Analysis

Always include when relevant:

- **Current**: What is the Big O now?
- **Optimized**: What could it be?
- **Data Scale**: At what data size does this matter?

Example: "Current O(n*m) where n=orders, m=items. With 100 orders and 1000 items, this executes 100,000 comparisons. Optimized O(n+m) would be 1,100 operations."

---

## Example Output

See `examples/example-output.md` for a sample showing:
- N+1 query with diff fix and complexity analysis
- Sequential awaits with Promise.all fix
- Unbounded query with pagination prompt

---

## Related Components

See `${CLAUDE_PLUGIN_ROOT}/agents/performance-agent.md` for the agent definition.
