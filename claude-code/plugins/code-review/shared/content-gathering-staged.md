# Content Gathering: Staged Commands

Shared content gathering logic for staged review commands (`/deep-review-staged`, `/quick-review-staged`).

## Process

Launch a Sonnet agent to gather the content to review:

### 1. Get Staged Diff

```bash
git diff --cached
```

### 2. Get Current Branch

```bash
git branch --show-current
```

### 3. Read Full File Content

For each staged file, also read the full file content for broader context.

### 4. Read Related Test Files

Read related test files (from Context Discovery step) for context:
- **Node.js**: See `languages/nodejs.md` for test file patterns
- **.NET**: See `languages/dotnet.md` for test file patterns

### 5. Create Summary

Summarize what changes are being made:
- Number of files modified
- Brief description of the changes
- Detected project type(s)
- Related test files found

## Output

Return to the review step:
- Current branch name
- Staged diff
- Full file content for each staged file
- Related test file contents
- Summary of what's being reviewed
