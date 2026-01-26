---
name: security-review
description: This skill should be used when the user asks to "security review", "check for vulnerabilities", "audit security", "find security issues", "security scan", "check for injection", "find hardcoded secrets", "OWASP check", or mentions reviewing code specifically for security concerns.
version: 3.2.0
---

# Security Code Review Skill

Identify vulnerabilities, insecure coding patterns, and security misconfigurations through targeted security-focused code review.

## Agent Configuration

Uses **security-agent** (Opus in thorough mode, Sonnet in gaps mode). See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for authoritative model configuration.

### Common Workflow Steps

See `${CLAUDE_PLUGIN_ROOT}/shared/skill-common-workflow.md` for scope determination, context gathering, validation, and reporting procedures.

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

High-confidence patterns that skip validation. For full definitions with regex patterns, see `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`.

**Security patterns:** `hardcoded_password`, `hardcoded_api_key`, `hardcoded_token`, `hardcoded_secret`, `hardcoded_credentials`, `sql_injection_concat`, `sql_injection_template`, `eval_untrusted`, `new_function_untrusted`

---

## Scope Prioritization

When reviewing directories, automatically prioritize:
- Files with "auth", "login", "password", "api", "db" in names
- Configuration files (`config/*.ts`, `appsettings.json`)
- Environment handling (`.env.example`, environment loading code)
- Middleware and interceptor definitions

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section.

**Security-specific additions** - do NOT flag:
- Placeholder values (`"changeme"`, `"TODO"`)
- Vulnerabilities explicitly mitigated elsewhere (documented upstream protection)

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
