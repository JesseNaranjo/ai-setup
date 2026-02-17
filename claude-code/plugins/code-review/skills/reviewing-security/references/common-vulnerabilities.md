# Common Security Vulnerabilities

## Injection Vulnerabilities

### SQL Injection

```javascript
// VULNERABLE
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

```csharp
// VULNERABLE
var query = "SELECT * FROM Users WHERE Id = " + userId;

// SAFE
var query = "SELECT * FROM Users WHERE Id = @id";
cmd.Parameters.AddWithValue("@id", userId);
```

**Severity**: Critical

### Command Injection

```javascript
// VULNERABLE
exec(`ls ${userInput}`);
spawn('sh', ['-c', userInput]);
```

```csharp
// VULNERABLE
Process.Start("cmd", "/c " + userInput);
```

**Severity**: Critical

### XSS (Cross-Site Scripting)

```javascript
// VULNERABLE
res.send(`<div>${userInput}</div>`);
element.innerHTML = userInput;
```

```csharp
// VULNERABLE — use @Html.Encode(userInput) or Razor auto-encoding (@userInput)
@Html.Raw(userInput)
```

**Severity**: Major (stored XSS = Critical)

## Authentication & Authorization

### Missing Authentication

```javascript
// VULNERABLE — no auth middleware
app.get('/api/admin/users', (req, res) => { ... });
```

```csharp
// VULNERABLE — missing [Authorize]
[HttpGet("admin/users")]
public IActionResult GetUsers() { ... }
```

**Severity**: Critical

### Broken Access Control

```javascript
// VULNERABLE — no ownership check
const order = await Order.findById(req.params.id);
res.json(order);
// SAFE: add if (order.userId !== req.user.id) return res.status(403).send();
```

**Severity**: Critical

## Sensitive Data Exposure

### Hardcoded Secrets

```javascript
// VULNERABLE
const API_KEY = 'sk-1234567890abcdef';
const password = 'admin123';
```

**Detection regex:**
- `password\s*=\s*["'][^"']+["']`
- `api[_-]?key\s*=\s*["'][^"']+["']`
- `secret\s*=\s*["'][^"']+["']`
- `token\s*=\s*["'][^"']+["']`

**Severity**: Critical

### Sensitive Data in Logs

```javascript
// VULNERABLE
console.log('User login:', { email, password });
logger.info('Request:', req.body);
```

**Severity**: Major

## Cryptographic Issues

```javascript
// VULNERABLE — weak hash
const hash = crypto.createHash('md5').update(password).digest('hex');
// SAFE: bcrypt.hash(password, 12)

// VULNERABLE — insecure random
const token = Math.random().toString(36);
// SAFE: crypto.randomBytes(32).toString('hex')
```

**Severity**: Critical (weak hash), Major (insecure random)

## Input Validation

### Path Traversal

```javascript
// VULNERABLE
const filePath = path.join('/uploads', req.params.filename);
// SAFE: const filename = path.basename(req.params.filename);
```

**Severity**: Critical

### Regex DoS (ReDoS)

```javascript
// VULNERABLE — catastrophic backtracking
const emailRegex = /^([a-zA-Z0-9]+)+@[a-zA-Z0-9]+$/;
```

**Severity**: Major

## Unsafe Deserialization

```javascript
// VULNERABLE
const obj = eval('(' + userInput + ')');
```

```csharp
// VULNERABLE
var obj = JsonConvert.DeserializeObject(input, new JsonSerializerSettings {
    TypeNameHandling = TypeNameHandling.All
});
// SAFE: JsonConvert.DeserializeObject<MyType>(input)
```

**Severity**: Critical

## Security Headers

Required: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY|SAMEORIGIN`, `Strict-Transport-Security`, `X-XSS-Protection: 1; mode=block`

**Severity**: Minor to Major depending on context

## Quick Detection Patterns

| Category | Key Patterns to Search For |
|----------|---------------------------|
| A01 | Missing authz checks, IDOR, directory traversal, `../`, role checks |
| A02 | MD5, SHA1, hardcoded keys, `password`, `secret`, `apiKey` in code |
| A03 | String concatenation in queries, `eval(`, `exec(`, `${}` in SQL |
| A04 | Missing rate limits, no CAPTCHA, business logic without validation |
| A05 | `DEBUG=true`, default ports, verbose errors, missing HTTPS |
| A06 | Old package versions, `npm audit`, `dotnet list`, outdated deps |
| A07 | Weak password regex, missing MFA, session in URL, no lockout |
| A08 | `deserialize(`, `pickle.loads(`, `unserialize(`, unsigned data |
| A09 | Missing `log.`, sensitive data in logs, no audit trail |
| A10 | `fetch(userInput)`, `request(url)`, `curl`, internal IPs |
