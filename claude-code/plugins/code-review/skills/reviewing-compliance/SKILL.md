---
name: reviewing-compliance
description: Reviews adherence to project-specific coding standards, CLAUDE.md guidelines, and AI agent instruction files. Use when checking coding standards compliance or convention adherence during code review.
---

# Compliance Code Review Skill

Adds rule citation guidelines, compliance scoring, compliance-specific FP adjustments.

## Pattern References

### Instruction File Locations

Search in: current directory, parent directories, `.github/` folders.

### Rule Classification

| Keyword | Severity | Examples |
|---------|----------|----------|
| MUST, SHALL, REQUIRED, ALWAYS, NEVER | Major | "All API endpoints MUST have authentication" |
| SHOULD, RECOMMENDED, PREFER | Minor | "Functions SHOULD have JSDoc comments" |
| MAY, OPTIONAL, CONSIDER | Suggestion | "You MAY use helper functions" |

### Common Rule Categories

- **Architecture**: direct DB in controllers, business logic in controllers, circular deps, layer violations
- **Naming**: file names not matching pattern (kebab-case/PascalCase/camelCase), inconsistent naming, missing interface prefixes (C#)
- **Testing**: new functions without tests, test file location, test naming
- **Documentation**: missing JSDoc/XMLDoc on public functions, outdated README
- **Security**: unvalidated input, SQL concatenation, hardcoded credentials, missing auth middleware
- **Error Handling**: empty catches, missing error handling, sensitive info in errors, stack traces in production

### Rule Conflicts

Per-directory overrides > root-level. Ambiguous rules: apply less restrictive, flag as Suggestion.

### Ignoring Rules

Rules silenced with comments (`claude-ignore:`, `eslint-disable`, `@ts-ignore`, `# noinspection`, `@SuppressWarnings`, `[SuppressMessage]`) should not be flagged.

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
