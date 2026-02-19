# Common Security Vulnerabilities

## .NET-Specific Patterns

SQL: `cmd.Parameters.AddWithValue("@id", userId)` — never string-concat. Severity: Critical.
XSS: `@Html.Raw(userInput)` vulnerable — use `@Html.Encode()` or Razor auto-encoding. Severity: Major (stored=Critical).
Deserialization: `TypeNameHandling.All` allows RCE — use `DeserializeObject<MyType>(input)`. Severity: Critical.

## Detection Regexes — Hardcoded Secrets

- `password\s*=\s*["'][^"']+["']`
- `api[_-]?key\s*=\s*["'][^"']+["']`
- `secret\s*=\s*["'][^"']+["']`
- `token\s*=\s*["'][^"']+["']`

Severity: Critical.

## Cryptographic Issues

MD5/SHA1 for passwords → bcrypt(12). `Math.random()` for tokens → `crypto.randomBytes(32)`. Severity: Critical/Major.

## ReDoS

Nested quantifiers — `/^([a-zA-Z0-9]+)+@/` causes catastrophic backtracking. Severity: Major.

## Security Headers

Required: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY|SAMEORIGIN`, `Strict-Transport-Security`, `X-XSS-Protection: 1; mode=block`. Severity: Minor to Major.

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
