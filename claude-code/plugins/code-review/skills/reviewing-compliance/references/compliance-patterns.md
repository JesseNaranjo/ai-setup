# Compliance Patterns

## Instruction File Locations

Search for instruction files in: current directory, parent directories, and `.github/` folders.

## Rule Classification

| Keyword | Severity | Examples |
|---------|----------|----------|
| MUST, SHALL, REQUIRED, ALWAYS, NEVER | Major | "All API endpoints MUST have authentication" |
| SHOULD, RECOMMENDED, PREFER | Minor | "Functions SHOULD have JSDoc comments" |
| MAY, OPTIONAL, CONSIDER | Suggestion | "You MAY use helper functions" |

## Common Rule Categories

- **Architecture**: direct DB calls in controllers, business logic in controllers, circular dependencies, layer violations
- **Naming**: file names not matching pattern (kebab-case, PascalCase, camelCase), inconsistent naming, missing interface prefixes (C#)
- **Testing**: new functions without tests, test file location, test naming patterns
- **Documentation**: missing JSDoc/XMLDoc on public functions, outdated README after changes
- **Security**: unvalidated user input, SQL string concatenation, hardcoded credentials, missing auth middleware
- **Error Handling**: empty catch blocks, missing error handling, sensitive info in error messages, stack traces in production

## Checking Process

1. **Parse Instructions**: Extract rules, identify keyword (MUST/SHOULD/MAY), note file patterns
2. **Match Files**: Check if rules apply based on file path and content/type
3. **Check Compliance**: Analyze code, classify severity by keyword, include exact rule text
4. **Report**: Include `rule_violated` and `rule_source` fields in output

## Ignoring Rules

Rules silenced with comments (`claude-ignore:`, `eslint-disable`, `@ts-ignore`) should not be flagged.
