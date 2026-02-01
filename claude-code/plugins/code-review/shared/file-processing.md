# File Processing: Validation and Content Gathering

This document combines input validation and content gathering for file-based commands (`/deep-code-review`, `/quick-code-review`).

## Contents

- [Input Validation](#input-validation)
- [Content Gathering](#content-gathering)

---

## Input Validation

Validate the input and check for changes:

### 1. Parse Arguments

Extract from the command arguments:
- **File paths**: Everything except flags (required, space-separated)
- **Output file path**: From `--output-file` flag (command provides default)
- **Language override**: From `--language` flag (`nodejs` or `dotnet`, default: auto-detect per file)

### 2. Verify Git Repository

Check if the current directory is a git repository:
```bash
git rev-parse --git-dir
```

If not a git repository, stop and inform the user: "Not a git repository."

### 3. Verify File Existence

For each specified file path:
- Use `ls` or check file paths to verify existence
- If a file doesn't exist, note it and exclude from review
- Track which files are valid

### 4. Check for Uncommitted Changes

For each valid file, check if there are uncommitted changes:
```bash
git diff HEAD -- <file>
```

Create two lists:
- **Files with changes**: Files that have uncommitted changes (review the changes)
- **Files without changes**: Files that have no uncommitted changes (review entire file)

### 5. Validate Scope

If no valid files were specified, stop and inform the user: "No valid files specified for review."

### Validation Output

Pass to Content Gathering:
- List of valid file paths
- Files with changes vs files without changes
- Parsed output file path
- Parsed language override (if any)

---

## Content Gathering

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

Read related test files (from Context Discovery step) for context:
- **Node.js**: See `languages/nodejs.md` for test file patterns
- **.NET**: See `languages/dotnet.md` for test file patterns

### 5. Create Summary

Summarize what will be reviewed:
- Which files have changes being reviewed
- Which files are being reviewed in their entirety (no changes)
- Detected project type(s)
- Related test files found

### Gathering Output

Return to the review step:
- Current branch name
- For each file: diff (if has changes), full content, has_changes flag
- Related test file contents
- Summary of what's being reviewed
