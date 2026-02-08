---
name: reviewing-compliance
description: Reviews adherence to project-specific coding standards, CLAUDE.md guidelines, and AI agent instruction files. Use when user mentions "check CLAUDE.md compliance", "review against coding standards", "check AI agent instructions", "verify guidelines", "check coding conventions", or "check naming conventions".
---

# Compliance Code Review Skill

Verify code adherence to AI Agent Instructions (CLAUDE.md, copilot-instructions, and similar files) and project-specific rules through targeted compliance-focused code review.

## Agent

`code-review:compliance-agent` (Sonnet)

Uses thorough mode with focus areas below.

---

## Rule Classification

For rule classification (MUST/SHOULD/MAY keywords and severity mapping), see `references/compliance-patterns.md`.

---

## Compliance Categories Checked

**Architecture Violations (Major):**
- Layer violations (UI importing from data layer)
- Business logic in wrong location (controller, view)
- Direct database access bypassing repositories
- Circular dependencies

**Naming Convention Violations (Minor to Major):**
- File naming patterns (kebab-case, PascalCase)
- Class/function/variable naming
- Constant naming (UPPER_SNAKE_CASE)
- Interface prefixing (I prefix in C#)

**Documentation Requirements (Minor):**
- Missing JSDoc/XMLDoc on public APIs
- Undocumented complex functions
- README updates for new features

**Security Requirements (Major):**
- Missing authentication on endpoints
- Input validation gaps
- Logging requirements for sensitive operations

---

## Auto-Validated Patterns

High-confidence patterns that skip validation. For full definitions, see `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## False Positives

Apply all rules from `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` "False Positive Rules" section.

**Compliance-specific additions** - do NOT flag:
- Explicit override comments (`// claude-ignore: rule-name`)
- Scope mismatch (rule doesn't apply to this file type)
- Ambiguous rules (benefit of doubt to code)
- Framework exemptions (framework provides required behavior)

---

## Rule Citation Guidelines

Always include exact rule citation:

- **Rule text**: Exact quoted text from instruction file
- **Source**: File path and line number
- **Keyword**: The strength keyword (MUST, SHOULD, MAY)

Example: "The endpoint violates 'All API endpoints MUST have authentication' from CLAUDE.md:31"

---

## Compliance Score

Include in output:

```markdown
### Compliance Score

- Rules checked: 28
- Rules passed: 24
- Rules violated: 4
- Compliance rate: 86%
```

---

## Example Output

See `examples/example-output.md` for a sample showing:
- Missing authentication violation with diff fix
- Business logic in controller with prompt fix
- Missing JSDoc violation with diff fix

