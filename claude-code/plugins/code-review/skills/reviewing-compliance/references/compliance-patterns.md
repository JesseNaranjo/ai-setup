# Compliance Patterns

## Instruction File Locations

Search in: current directory, parent directories, `.github/` folders.

## Rule Classification

| Keyword | Severity | Examples |
|---------|----------|----------|
| MUST, SHALL, REQUIRED, ALWAYS, NEVER | Major | "All API endpoints MUST have authentication" |
| SHOULD, RECOMMENDED, PREFER | Minor | "Functions SHOULD have JSDoc comments" |
| MAY, OPTIONAL, CONSIDER | Suggestion | "You MAY use helper functions" |

## Common Rule Categories

- **Architecture**: direct DB in controllers, business logic in controllers, circular deps, layer violations
- **Naming**: file names not matching pattern (kebab-case/PascalCase/camelCase), inconsistent naming, missing interface prefixes (C#)
- **Testing**: new functions without tests, test file location, test naming
- **Documentation**: missing JSDoc/XMLDoc on public functions, outdated README
- **Security**: unvalidated input, SQL concatenation, hardcoded credentials, missing auth middleware
- **Error Handling**: empty catches, missing error handling, sensitive info in errors, stack traces in production

## Checking Process

1. **Parse**: Extract rules, identify keyword (MUST/SHOULD/MAY), note file patterns
2. **Match**: Check rule applicability by file path and content/type
3. **Check**: Analyze code, classify severity by keyword, include exact rule text
4. **Report**: Include `rule_violated` and `rule_source` fields

## Ignoring Rules

Rules silenced with comments (`claude-ignore:`, `eslint-disable`, `@ts-ignore`) should not be flagged.
