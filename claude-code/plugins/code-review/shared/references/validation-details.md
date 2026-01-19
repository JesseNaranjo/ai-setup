# Validation Details

Detailed validation procedures for review findings. Referenced by `skill-common-workflow.md` Step 4.

## Auto-Validated Patterns

High-confidence patterns skip validation and are marked `auto_validated: true`. Each skill defines its own auto-validation patterns (e.g., hardcoded secrets, SQL injection concatenation).

## Batch Validation Process

For issues not matching auto-validation patterns:

1. Group all issues for the same file
2. Launch one validator per file (not per issue)
3. Validator receives file content + all issues + full context
4. Validator returns: VALID, INVALID, or DOWNGRADE for each

## Common False Positive Indicators

Instruct validators to check for:

- **Test code:** Issues in test files may be intentional
- **Comments/docs:** Code in documentation is not executable
- **Disabled code:** Commented-out code is not active
- **Placeholder values:** `"changeme"` or `"TODO"` are not real values
- **Framework guarantees:** Framework may provide protection elsewhere

## Validation Prompt Template

```
Validate these findings against the file content:

File: {file_path}
Content: {file_content}

Issues to validate:
{issues_list}

For each issue, determine:
- VALID: Issue is real and actionable
- INVALID: Issue is a false positive (explain why)
- DOWNGRADE: Issue exists but severity should be lower (explain why)
```

## Validator Model Assignment

| Issue Severity | Validator Model |
|---------------|-----------------|
| Critical | Opus |
| Major | Sonnet |
| Minor | Haiku |
| Suggestion | Haiku |

## Deduplication Rules

Before validation, deduplicate issues:
- Same file + overlapping line ranges = duplicate
- Keep higher severity version
- Merge descriptions if both add value
