---
name: security-review
description: This skill should be used when the user asks to "security review", "check for vulnerabilities", "audit security", "find security issues", "security scan", "check for injection", "find hardcoded secrets", "OWASP check", or mentions reviewing code specifically for security concerns.
version: 3.1.3
---

# Security Code Review Skill

Identify vulnerabilities, insecure coding patterns, and security misconfigurations through targeted security-focused code review.

## Security-Specific Configuration

### Agent Parameters

- **Agent:** `${CLAUDE_PLUGIN_ROOT}/agents/security-agent.md`
- **Model:** Opus (for thorough vulnerability detection)
- **Modes:** thorough (first pass), gaps (second pass)

### Security Categories Checked

**Injection Vulnerabilities (Critical):**
- SQL injection via string concatenation
- Command injection through shell execution
- XSS via unescaped HTML output
- Template injection, LDAP injection

**Authentication & Authorization (Critical):**
- Missing authentication on endpoints
- Broken access control (IDOR)
- Session management weaknesses
- Missing CSRF protection

**Sensitive Data Exposure (Critical):**
- Hardcoded secrets, API keys, passwords
- Sensitive data in logs
- Unencrypted data transmission
- Excessive data exposure in APIs

**Cryptographic Issues (Major):**
- Weak hashing (MD5, SHA1 for passwords)
- Insecure random number generation
- Missing encryption for sensitive data

**Input Validation (Major):**
- Path traversal vulnerabilities
- ReDoS (regex denial of service)
- Unsafe deserialization
- Missing input sanitization

---

## Auto-Validated Patterns

These high-confidence patterns skip validation:

| Pattern | Description |
|---------|-------------|
| `hardcoded_password` | `password = "..."` assignments |
| `hardcoded_api_key` | `api_key = "..."` assignments |
| `hardcoded_token` | `token = "..."`, `bearer = "..."` assignments |
| `hardcoded_secret` | `secret = "..."`, `private_key = "..."` assignments |
| `hardcoded_credentials` | `credentials = "..."`, `connection_string = "..."` assignments |
| `sql_injection_concat` | SQL + string concatenation with user input |
| `sql_injection_template` | SQL + template literal interpolation with user input |
| `eval_untrusted` | `eval()` with request/user input |
| `new_function_untrusted` | `new Function()` with request/user input |

---

## Scope Prioritization

When reviewing directories, automatically prioritize:
- Files with "auth", "login", "password", "api", "db" in names
- Configuration files (`config/*.ts`, `appsettings.json`)
- Environment handling (`.env.example`, environment loading code)
- Middleware and interceptor definitions

---

## Security-Specific False Positives

Do NOT flag:
- Security issues in test files (usually intentional)
- Internal-only code with no untrusted input exposure
- Code with explicit security comments explaining design
- Placeholder values (`"changeme"`, `"TODO"`)
- Vulnerabilities already mitigated elsewhere

---

## OWASP Top 10 Quick Reference

For OWASP Top 10 reference, see `references/owasp-reference.md`.

---

## Example Output

See `examples/example-output.md` for a sample showing:
- SQL injection finding with diff fix
- Hardcoded API key with prompt fix
- Missing rate limiting with prompt fix

---

## Related Components

See `${CLAUDE_PLUGIN_ROOT}/agents/security-agent.md` for the agent definition.
