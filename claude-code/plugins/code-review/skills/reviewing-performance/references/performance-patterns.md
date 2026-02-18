# Performance Patterns

## Algorithmic Complexity

### O(n^2) Nested Loops

```javascript
// SLOW — O(n^2)
for (const user of users) {
  for (const order of orders) {
    if (order.userId === user.id) { /* process */ }
  }
}
// FAST: build Map of orders by userId first — O(n)
```

**Severity**: Major (large datasets), Minor (small fixed datasets)

### Repeated Expensive Operations

```javascript
// SLOW — called per item
return items.map(item => transform(item, loadConfigFromDisk()));
```

## Database Performance

### N+1 Query Problem

```javascript
// SLOW — N+1 queries
for (const user of users) {
  user.orders = await Order.findByUserId(user.id);
}
// FAST: fetch all orders in one query, groupBy userId
```

```csharp
// SLOW: per-user query in loop
// FAST
var users = await _db.Users.Include(u => u.Orders).ToListAsync();
```

**Severity**: Critical (hot paths), Major (otherwise)

### Missing Indexes

WHERE/JOIN/ORDER BY on non-indexed columns.

**Severity**: Major

### Unbounded Queries

```javascript
// DANGEROUS
const allUsers = await db.query('SELECT * FROM users');
// SAFE: add LIMIT/OFFSET pagination
```

## Memory Issues

### Memory Leaks

```javascript
// LEAK — no removeEventListener in cleanup
window.addEventListener('resize', this.onResize);

// LEAK — unbounded Map growth
const cache = new Map();
function getCached(key) { if (!cache.has(key)) cache.set(key, expensiveComputation(key)); return cache.get(key); }
```

**Severity**: Critical

### Large Object Allocation

Large objects created per call instead of reusing constants (`Object.freeze()`). **Severity**: Minor to Major

## Async/Await Performance

### Sequential Awaits (Waterfall)

```javascript
// SLOW — sequential (3 seconds total)
const users = await fetchUsers();
const orders = await fetchOrders();
// FAST: Promise.all([fetchUsers(), fetchOrders(), fetchProducts()])
```

### Blocking Event Loop (Node.js)

```javascript
// BLOCKING — CPU loop without yield; chunk with setImmediate()
for (let i = 0; i < data.length; i++) { /* process */ }
```

**Severity**: Critical (servers), Major (scripts)

## String Concatenation in Loops

```javascript
// SLOW — new string each iteration
let result = '';
for (const item of items) { result += item.toString(); }
// FAST: items.map(i => i.toString()).join('')
```

```csharp
// SLOW — O(n^2)
string result = "";
foreach (var item in items) { result += item.ToString(); }
// FAST: StringBuilder
```

## .NET Specific

### Boxing/Unboxing

```csharp
// BOXING; use List<int> instead
var list = new ArrayList(); list.Add(42);
```

**Severity**: Minor (isolated), Major (hot loops)

### LINQ in Hot Paths

```csharp
// ALLOCATES — use manual loop in hot paths
var first = items.FirstOrDefault(x => x.Id == id);
```

### Async Void

```csharp
// BAD — exceptions lost, can't await; use async Task
async void ProcessAsync() { ... }
```

**Severity**: Major (also correctness issue)

## Quick Reference

| Issue | Severity | Fix Difficulty |
|-------|----------|----------------|
| N+1 Queries | Critical/Major | Medium |
| O(n^2) Loops | Major | Medium |
| Memory Leaks | Critical | High |
| Sequential Awaits | Major | Low |
| String Concat in Loop | Major | Low |
| Unbounded Queries | Major | Low |
| Boxing in Loops | Minor/Major | Low |
