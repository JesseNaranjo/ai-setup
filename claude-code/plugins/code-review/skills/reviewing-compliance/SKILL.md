---
name: reviewing-compliance
description: Use when checking coding standards compliance or convention adherence during code review.
---

# Compliance Code Review Skill

Adds rule citation guidelines, compliance scoring, compliance-specific FP adjustments.

## Pattern References

### Instruction File Locations

Search in: current directory, parent directories, `.github/` folders.

See `references/compliance-patterns.md` for rule classification, common categories, conflicts, and suppression patterns.

## Scope Prioritization

Explicit violations > implicit gaps > style preferences. Prioritize violations of documented rules over stylistic inconsistencies.

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
