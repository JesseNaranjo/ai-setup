---
name: reviewing-security
description: Detects injection attacks, authentication bypasses, hardcoded secrets, insecure cryptography, and OWASP top 10 issues. Use when checking for security vulnerabilities, conducting vulnerability audits, or scanning for secrets during code review.
---

# Security Code Review Skill

Identify vulnerabilities, insecure coding patterns, and security misconfigurations through targeted security-focused code review.

## Agent

`code-review:security-agent` (Opus)

Uses thorough mode with focus areas below.

## Security Categories Checked

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

High-confidence patterns that skip validation. For full definitions with regex patterns, see `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-code.md`.

---

## Scope Prioritization

When reviewing directories, automatically prioritize:
- Files with "auth", "login", "password", "api", "db" in names
- Configuration files (`config/*.ts`, `appsettings.json`)
- Environment handling (`.env.example`, environment loading code)
- Middleware and interceptor definitions

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "False Positive Rules" section.

**Security-specific additions** - do NOT flag:
- Placeholder values (`"changeme"`, `"TODO"`)
- Vulnerabilities explicitly mitigated elsewhere (documented upstream protection)

---

## Security References

For detailed vulnerability patterns with code examples, see `references/common-vulnerabilities.md`.

For OWASP Top 10 category reference, see `references/owasp-reference.md`.

---

## Example Output

See `examples/example-output.md` for a sample showing:
- SQL injection finding with diff fix
- Hardcoded API key with prompt fix
- Missing rate limiting with prompt fix

