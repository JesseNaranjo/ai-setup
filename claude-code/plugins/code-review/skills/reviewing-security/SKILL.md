---
name: reviewing-security
description: Detects injection attacks, authentication bypasses, hardcoded secrets, insecure cryptography, and OWASP top 10 issues. Use when checking for security vulnerabilities, conducting vulnerability audits, or scanning for secrets during code review.
---

# Security Code Review Skill

## Agent

`code-review:security-agent` (Opus) in thorough mode.

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

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions with regex patterns: `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

## Scope Prioritization

Prioritize:
- Files with "auth", "login", "password", "api", "db" in names
- Configuration files (`config/*.ts`, `appsettings.json`)
- Environment handling (`.env.example`, environment loading code)
- Middleware and interceptor definitions

## False Positives

Apply `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "False Positive Rules".

**Security-specific additions** - do NOT flag:
- Placeholder values (`"changeme"`, `"TODO"`)
- Vulnerabilities explicitly mitigated elsewhere (documented upstream protection)

## Security References

`references/common-vulnerabilities.md`

## Example Output

`examples/example-output.md`

