---
name: reviewing-performance
description: Use when checking for performance issues, optimization opportunities, or latency concerns during code review.
---

# Performance Code Review Skill

Adds hot path identification, database code prioritization, Big O analysis format, performance-specific FP adjustments, performance pattern references.

## Scope Prioritization

### Hot Path Identification

- Files with "api", "handler", "controller", "route" in name (entry points)
- Files with "service", "repository", "dao", "query" in name (data access)
- Files with "util", "helper", "common" in name (frequently called)

### Related Database Code

For N+1 and query performance, also read:
- ORM model definitions (`models/*.ts`, `entities/*.cs`)
- Repository implementations
- Migration files (index information)

## False Positives

**Performance-specific** - do NOT flag:
- O(n^2) on small bounded data (max 10 items)
- Startup/initialization code (one-time cost)
- Code with caching/memoization elsewhere
- Theoretical improvements requiring benchmark proof

## Complexity Analysis

Always include: **Current** (Big O now), **Optimized** (achievable Big O), **Data Scale** (threshold where it matters).

Example: "Current O(n*m) where n=orders, m=items. With 100 orders and 1000 items, this executes 100,000 comparisons. Optimized O(n+m) would be 1,100 operations."

## Pattern References

`references/performance-patterns.md`
