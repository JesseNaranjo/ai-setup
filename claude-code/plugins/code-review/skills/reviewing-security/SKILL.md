---
name: reviewing-security
description: Detects injection attacks, authentication bypasses, hardcoded secrets, insecure cryptography, and OWASP top 10 issues. Use when checking for security vulnerabilities, conducting vulnerability audits, or scanning for secrets during code review.
---

# Security Code Review Skill

Adds scope prioritization (auth/config/middleware), security-specific FP adjustments, vulnerability pattern references.

## Scope Prioritization

- Files with "auth", "login", "password", "api", "db" in names
- Configuration files (`config/*.ts`, `appsettings.json`)
- Environment handling (`.env.example`, environment loading code)
- Middleware and interceptor definitions

## False Positives

**Security-specific** - do NOT flag:
- Placeholder values (`"changeme"`, `"TODO"`)
- Vulnerabilities mitigated elsewhere (documented upstream protection)

## Pattern References

`references/common-vulnerabilities.md`
