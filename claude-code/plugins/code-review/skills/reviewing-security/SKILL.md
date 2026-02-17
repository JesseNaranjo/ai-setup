---
name: reviewing-security
description: Detects injection attacks, authentication bypasses, hardcoded secrets, insecure cryptography, and OWASP top 10 issues. Use when checking for security vulnerabilities, conducting vulnerability audits, or scanning for secrets during code review.
---

# Security Code Review Skill

Enhancement: Adds scope prioritization (auth/config/middleware files), security-specific false positive adjustments, and detailed vulnerability pattern references.

## Agent

`code-review:security-agent` (Opus) in thorough mode.

## Auto-Validated Patterns

High-confidence patterns skip validation. Full definitions with regex patterns: `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`.

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

