# Performance Patterns

## Algorithmic Complexity

O(n^2) nested loops → build Map/index first. Severity: Major (large datasets), Minor (small fixed).
Repeated expensive operations per-item (e.g., config reload in `.map()`) → hoist. Severity: Major.

## Database Performance

N+1 queries → batch fetch. .NET: `_db.Users.Include(u => u.Orders).ToListAsync()`. Severity: Critical (hot paths), Major (otherwise).
Missing indexes on WHERE/JOIN/ORDER BY columns. Severity: Major.
Unbounded queries without LIMIT/OFFSET. Severity: Major.

## Memory Issues

Memory leaks — missing `removeEventListener` in cleanup; unbounded `Map`/cache growth without eviction. Severity: Critical.
Large objects created per-call instead of reusing `Object.freeze()` constants. Severity: Minor to Major.

## Async/Await Performance

Sequential awaits (waterfall) → `Promise.all()` for independent calls. Severity: Major.
Blocking event loop (Node.js) — CPU loop without yield; chunk with `setImmediate()`. Severity: Critical (servers), Major (scripts).

## Cloning and Collection Patterns

JSON.parse/stringify deep clone: O(n) with high constant, fails on circular refs, drops functions/symbols. Use structuredClone() in Node 17+. Severity: Minor to Major.
Array.from() on large iterables more memory-efficient than spread. Severity: Minor.

## .NET Specific

Boxing/Unboxing: `ArrayList` with value types → `List<int>`. Severity: Minor (isolated), Major (hot loops).
LINQ in hot paths: `FirstOrDefault()` allocates — use manual loop. Severity: Minor to Major.
Async void: exceptions lost, can't await — use `async Task`. Severity: Major (also correctness issue).
