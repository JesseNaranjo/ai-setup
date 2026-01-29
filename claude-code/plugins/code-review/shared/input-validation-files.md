# Input Validation: File-Based Commands

Shared input validation logic for file-based review commands (`/deep-code-review`, `/quick-code-review`).

## Process

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

## Output

Return to the next step:
- List of valid file paths
- Files with changes vs files without changes
- Parsed output file path
- Parsed language override (if any)
