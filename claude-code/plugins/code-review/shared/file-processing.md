# File Processing: Validation and Content Gathering

## Input Validation

Parse arguments:
- **File paths**: All non-flag arguments (required, space-separated)
- **Output file path**: From `--output-file` flag
- **Language override**: From `--language` flag (`nodejs`, `react`, `dotnet`)

Files must be validated for:
- Git repository presence (fail if not git repo)
- Existence (exclude non-existent files)
- Change status (uncommitted changes vs no changes)

Fail if no valid files remain after validation.

**Validation output:**
- Valid file paths
- Files with changes vs files without changes
- Output file path
- Language override (if any)

## Content Gathering

Launch Sonnet agent to gather:

**For files with uncommitted changes:**
- Diff (uncommitted changes only)
- Full file content (broader context)

**For files without changes:**
- Full file content

**Related test files:**
- Read test files discovered in Context Discovery step
- Test file patterns per `languages/nodejs.md` and `languages/dotnet.md`

**Summary:**
- Which files have changes being reviewed
- Which files reviewed in entirety (no changes)
- Detected project type(s)
- Related test files found

**Gathering output:**
- Current branch name
- For each file: diff (if has_changes), full content, has_changes flag
- Related test file contents
- Review summary
