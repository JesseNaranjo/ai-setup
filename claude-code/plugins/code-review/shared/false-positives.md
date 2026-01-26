# False Positives

Issues that should NOT be flagged by review agents.

## Do NOT Flag

### Pre-existing Issues

- Issues in files with changes that existed before these changes
- Issues in the codebase that are not related to the current changes
- Pre-existing code patterns that weren't modified

### Correct Code

- Something that appears to be a bug but is actually correct behavior
- Intentional patterns that look unusual but serve a purpose
- Code that handles edge cases in non-obvious but valid ways

### Pedantic Concerns

- Nitpicks that a senior engineer would not flag
- Style preferences that don't affect functionality
- Minor formatting inconsistencies

### Linter Territory

- Issues that a linter will catch (do not run the linter to verify)
- Formatting issues covered by prettier/eslint/other formatters
- Import ordering or unused import warnings

### Silenced Issues

- Issues mentioned in AI Agent Instructions but explicitly silenced in code
- Code with lint-disable comments for specific rules
- Intentionally suppressed warnings with documented reasons

### Scope Limitations

- General code quality concerns unless explicitly required in AI Agent Instructions
- Theoretical edge cases that are extremely unlikely in practice
- Suggestions for features or patterns not in scope
- Issues in test code or development-only paths unless specifically reviewing tests

## Quick vs Deep Review Differences

### Quick Review (Stricter)

Quick review is optimized for speed and should be extra conservative:
- Skip theoretical edge cases
- Focus only on blocking issues that must be fixed before merge
- Ignore minor style concerns

### Deep Review (More Thorough)

Deep review can flag more issues but should still avoid:
- Pre-existing issues unrelated to changes
- Pure style preferences
- Issues explicitly silenced in code
