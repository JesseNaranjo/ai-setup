# Common Bug Patterns

## Null/Undefined Reference

```javascript
// BUG
function getUserName(user) {
  return user.profile.name; // Crashes if user or profile is null
}
// SAFE
function getUserName(user) {
  return user?.profile?.name ?? 'Unknown';
}
```

```csharp
// BUG
public string GetUserName(User user) {
  return user.Profile.Name; // NullReferenceException
}
// SAFE
public string GetUserName(User user) {
  return user?.Profile?.Name ?? "Unknown";
}
```

**Severity**: Major (crashes), Minor (undefined behavior)

## Off-by-One Errors

```javascript
// BUG - includes non-existent index
for (let i = 0; i <= arr.length; i++) {
  console.log(arr[i]); // arr[arr.length] is undefined
}

// BUG - counts spaces, not words
function countWords(str) {
  return str.split(' ').length - 1;
}
// CORRECT
function countWords(str) {
  return str.trim().split(/\s+/).length;
}
```

**Severity**: Minor to Major

## Async/Promise Issues

### Unhandled Promise Rejection

```javascript
// BUG - unhandled rejection
async function loadData() {
  const data = await fetch('/api/data');
  return data.json();
}
loadData(); // No .catch()
```

### Race Conditions

```javascript
// BUG - state changed between await calls
async function updateUser(id, data) {
  const user = await getUser(id);
  user.count += 1;
  await saveUser(user); // Another request may have updated count
}

// SAFE - atomic update
async function updateUser(id, data) {
  await db.query('UPDATE users SET count = count + 1 WHERE id = ?', [id]);
}
```

**Severity**: Major to Critical

### Floating Promises

```javascript
// BUG - doesn't wait for save
function handleSubmit(data) {
  saveData(data); // Forgot await!
  showSuccess(); // Shows before save completes
}
```

**Severity**: Major

## Type Coercion Issues

```javascript
// UNEXPECTED
0 == ''       // true
null == undefined  // true
[] == false   // true

// BUG - string concat instead of addition
const total = '10' + 5;  // '105'
const sorted = [1, 10, 2].sort();  // [1, 10, 2] (string sort)

// CORRECT
const total = Number('10') + 5;  // 15
const sorted = [1, 10, 2].sort((a, b) => a - b);  // [1, 2, 10]
```

**Severity**: Minor to Major

## State Management

### Mutating Shared State

```javascript
// BUG
function processItems(items) {
  items.sort(); // Mutates original array!
  return items.slice(0, 5);
}
// SAFE
function processItems(items) {
  return [...items].sort().slice(0, 5);
}
```

### Stale Closure

```javascript
// BUG - always logs initial count
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setInterval(() => {
      console.log(count); // Always 0!
    }, 1000);
  }, []);
}
```

**Severity**: Major

## Error Handling

```javascript
// BUG - error silently ignored
try {
  await saveData(data);
} catch (e) {
  // Empty catch
}

// BUG - catches too broad
try {
  JSON.parse(input);
} catch (e) {
  return null; // Also catches programmer errors!
}
// BETTER
try {
  JSON.parse(input);
} catch (e) {
  if (e instanceof SyntaxError) return null;
  throw e;
}
```

**Severity**: Minor to Major

## .NET Specific

### IDisposable Not Disposed

```csharp
// BUG - connection may leak
var connection = new SqlConnection(connString);
connection.Open();
// Missing: connection.Dispose()

// CORRECT
using var connection = new SqlConnection(connString);
connection.Open();
```

### Async Deadlock

```csharp
// DEADLOCK in ASP.NET
public ActionResult Index() {
  var data = GetDataAsync().Result; // Deadlock!
  return View(data);
}

// CORRECT
public async Task<ActionResult> Index() {
  var data = await GetDataAsync();
  return View(data);
}
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
