# Input Validation: Staged Commands

Shared input validation logic for staged review commands (`/deep-review-staged`, `/quick-review-staged`).

## Process

Check if there are any staged changes:

### 1. Verify Git Repository

Check if the current directory is a git repository:
```bash
git rev-parse --git-dir
```

If not a git repository, stop and inform the user: "Not a git repository."

### 2. Check for Staged Changes

Run to see all staged changes:
```bash
git diff --cached --stat
```

If there are no staged changes, stop and inform the user: "No staged changes to review."

### 3. Get Staged File List

Run to get the list of staged files:
```bash
git diff --cached --name-only
```

### 4. Parse Arguments

Extract from the command arguments:
- **Output file path**: From `--output-file` flag (command provides default)
- **Language override**: From `--language` flag (`nodejs` or `dotnet`, default: auto-detect per file)

## Output

Return to the next step:
- List of staged file paths
- Parsed output file path
- Parsed language override (if any)
