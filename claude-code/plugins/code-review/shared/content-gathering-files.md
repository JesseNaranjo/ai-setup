# Content Gathering: File-Based Commands

Shared content gathering logic for file-based review commands (`/deep-review`, `/quick-review`).

## Process

Launch a Sonnet agent to gather the content to review:

### 1. Get Current Branch

```bash
git branch --show-current
```

### 2. Gather Content for Files with Changes

For each file that has uncommitted changes:
- Run `git diff HEAD -- <file>` to get the diff
- Also read the full file content for broader context

### 3. Gather Content for Files without Changes

For each file that has no uncommitted changes:
- Read the entire file content

### 4. Read Related Test Files

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-common.md`.

### 5. Create Summary

Summarize what will be reviewed:
- Which files have changes being reviewed
- Which files are being reviewed in their entirety (no changes)
- Detected project type(s)
- Related test files found

## Output

Return to the review step:
- Current branch name
- For each file: diff (if has changes), full content, has_changes flag
- Related test file contents
- Summary of what's being reviewed
