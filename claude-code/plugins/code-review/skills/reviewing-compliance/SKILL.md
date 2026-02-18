---
name: reviewing-compliance
description: Reviews adherence to project-specific coding standards, CLAUDE.md guidelines, and AI agent instruction files. Use when checking coding standards compliance or convention adherence during code review.
---

# Compliance Code Review Skill

Adds rule citation guidelines, compliance scoring, compliance-specific FP adjustments.

## Agent

`code-review:compliance-agent` (Sonnet) in thorough mode.

## Rule Classification

MUST/SHOULD/MAY keywords and severity mapping: `references/compliance-patterns.md`.

## False Positives

**Compliance-specific** - do NOT flag:
- Explicit override comments (`// claude-ignore: rule-name`)
- Scope mismatch (rule doesn't apply to file type)
- Ambiguous rules (benefit of doubt to code)
- Framework exemptions (framework provides required behavior)

## Rule Citation Guidelines

Always include:
- **Rule text**: Exact quoted text from instruction file
- **Source**: File path and line number
- **Keyword**: MUST, SHOULD, or MAY

Example: "The endpoint violates 'All API endpoints MUST have authentication' from CLAUDE.md:31"

## Compliance Score

Include:

```markdown
### Compliance Score

- Rules checked: 28
- Rules passed: 24
- Rules violated: 4
- Compliance rate: 86%
```
