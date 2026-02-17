# Common Bug Patterns

## Null/Undefined Reference

```javascript
// BUG
return user.profile.name; // Crashes if user or profile is null
// SAFE: user?.profile?.name ?? 'Unknown'
```

```csharp
// BUG
return user.Profile.Name; // NullReferenceException
// SAFE: user?.Profile?.Name ?? "Unknown"
```

**Severity**: Major (crashes), Minor (undefined behavior)

## Off-by-One Errors

```javascript
// BUG — includes non-existent index
for (let i = 0; i <= arr.length; i++) { ... }

// BUG — counts spaces, not words
return str.split(' ').length - 1;
// CORRECT: str.trim().split(/\s+/).length
```

**Severity**: Minor to Major

## Async/Promise Issues

### Unhandled Promise Rejection

```javascript
// BUG — no .catch()
loadData(); // async function called without await or .catch()
```

### Race Conditions

```javascript
// BUG — state changed between awaits
const user = await getUser(id);
user.count += 1;
await saveUser(user); // Another request may have updated count
// SAFE: atomic db.query('UPDATE users SET count = count + 1 WHERE id = ?', [id])
```

**Severity**: Major to Critical

### Floating Promises

```javascript
// BUG — missing await
saveData(data); // Shows success before save completes
```

**Severity**: Major

## Type Coercion Issues

```javascript
// UNEXPECTED
0 == ''       // true
null == undefined  // true
[] == false   // true

// BUG — string concat instead of addition
const total = '10' + 5;  // '105'
const sorted = [1, 10, 2].sort();  // [1, 10, 2] (string sort)
```

**Severity**: Minor to Major

## State Management

### Mutating Shared State

```javascript
// BUG — mutates original array
items.sort();
// SAFE: [...items].sort().slice(0, 5)
```

### Stale Closure

```javascript
// BUG — always logs initial count (0)
useEffect(() => {
  setInterval(() => { console.log(count); }, 1000);
}, []);
```

**Severity**: Major

## Error Handling

```javascript
// BUG — error silently ignored
try { await saveData(data); } catch (e) { /* Empty catch */ }

// BUG — catches too broad; re-throw non-SyntaxError
try { JSON.parse(input); } catch (e) { return null; }
```

**Severity**: Minor to Major

## .NET Specific

### IDisposable Not Disposed

```csharp
// BUG — connection may leak
var connection = new SqlConnection(connString);
connection.Open();
// CORRECT: using var connection = new SqlConnection(connString);
```

### Async Deadlock

```csharp
// DEADLOCK in ASP.NET
var data = GetDataAsync().Result;
// CORRECT: async Task<ActionResult> with await GetDataAsync()
```

**Severity**: Critical

## Quick Reference

| Bug Type | Severity | Detection |
|----------|----------|-----------|
| Null Reference | Major | Optional chaining check |
| Off-by-One | Major | Loop bounds, array access |
| Unhandled Promise | Major | Missing await/catch |
| Race Condition | Critical | Concurrent state access |
| Empty Catch | Major | Catch blocks without handling |
| IDisposable Leak | Major | Using statement check |
| Async Deadlock | Critical | .Result/.Wait() usage |
