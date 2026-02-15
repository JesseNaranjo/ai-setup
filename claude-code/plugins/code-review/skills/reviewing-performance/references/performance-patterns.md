# Performance Patterns

## Algorithmic Complexity

### O(n^2) Nested Loops

```javascript
// SLOW - O(n^2)
for (const user of users) {
  for (const order of orders) {
    if (order.userId === user.id) { /* process */ }
  }
}

// FAST - O(n)
const ordersByUser = new Map();
for (const order of orders) {
  if (!ordersByUser.has(order.userId)) ordersByUser.set(order.userId, []);
  ordersByUser.get(order.userId).push(order);
}
```

**Severity**: Major (large datasets), Minor (small fixed datasets)

### Repeated Expensive Operations

```javascript
// SLOW
function processItems(items) {
  return items.map(item => {
    const config = loadConfigFromDisk(); // Called for each item!
    return transform(item, config);
  });
}
```

## Database Performance

### N+1 Query Problem

```javascript
// SLOW - N+1 queries
const users = await User.findAll();
for (const user of users) {
  user.orders = await Order.findByUserId(user.id); // N queries!
}

// FAST - 2 queries
const users = await User.findAll();
const orders = await Order.findByUserIds(users.map(u => u.id));
const ordersByUser = groupBy(orders, 'userId');
```

```csharp
// SLOW
foreach (var user in users) {
  user.Orders = await _db.Orders.Where(o => o.UserId == user.Id).ToListAsync();
}
// FAST
var users = await _db.Users.Include(u => u.Orders).ToListAsync();
```

**Severity**: Critical (hot paths), Major (otherwise)

### Missing Indexes

WHERE clauses on non-indexed columns, JOIN on non-indexed foreign keys, ORDER BY on non-indexed columns.

**Severity**: Major

### Unbounded Queries

```javascript
// DANGEROUS
const allUsers = await db.query('SELECT * FROM users');
// SAFE
const users = await db.query('SELECT * FROM users LIMIT 100 OFFSET 0');
```

## Memory Issues

### Memory Leaks

```javascript
// LEAK - listener never removed
class Component {
  constructor() {
    window.addEventListener('resize', this.onResize);
  }
  // Missing: removeEventListener in cleanup
}

// LEAK - growing Map without eviction
const cache = new Map();
function getCached(key) {
  if (!cache.has(key)) cache.set(key, expensiveComputation(key));
  return cache.get(key);
}
```

**Severity**: Critical

### Large Object Allocation

Functions that create large objects per call instead of reusing constants (`Object.freeze()`).

**Severity**: Minor to Major

## Async/Await Performance

### Sequential Awaits (Waterfall)

```javascript
// SLOW - sequential (3 seconds)
const users = await fetchUsers();     // 1s
const orders = await fetchOrders();   // 1s
const products = await fetchProducts(); // 1s

// FAST - parallel (1 second)
const [users, orders, products] = await Promise.all([
  fetchUsers(), fetchOrders(), fetchProducts()
]);
```

### Blocking Event Loop (Node.js)

```javascript
// BLOCKING
function processLargeFile(data) {
  for (let i = 0; i < data.length; i++) { /* CPU-intensive */ }
}

// NON-BLOCKING
async function processLargeFile(data) {
  const CHUNK_SIZE = 1000;
  for (let i = 0; i < data.length; i += CHUNK_SIZE) {
    processChunk(data.slice(i, i + CHUNK_SIZE));
    await setImmediate();
  }
}
```

**Severity**: Critical (servers), Major (scripts)

## String Concatenation in Loops

```javascript
// SLOW
let result = '';
for (const item of items) {
  result += item.toString(); // Creates new string each time
}
// FAST
const result = items.map(i => i.toString()).join('');
```

```csharp
// SLOW - O(n^2)
string result = "";
foreach (var item in items) { result += item.ToString(); }
// FAST
var sb = new StringBuilder();
foreach (var item in items) { sb.Append(item); }
```

## .NET Specific

### Boxing/Unboxing

```csharp
// BOXING
var list = new ArrayList();
list.Add(42); // boxes int

// NO BOXING
var list = new List<int>();
list.Add(42);
```

**Severity**: Minor (isolated), Major (hot loops)

### LINQ in Hot Paths

```csharp
// ALLOCATES
var first = items.FirstOrDefault(x => x.Id == id);
// For hot paths, use manual loop instead
foreach (var item in items) { if (item.Id == id) return item; }
```

### Async Void

```csharp
// BAD - exceptions lost, can't await
async void ProcessAsync() { ... }
// GOOD
async Task ProcessAsync() { ... }
```

**Severity**: Major (also a correctness issue)

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
