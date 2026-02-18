# File Processing: Validation and Content Gathering

## Input Validation

Parse arguments:
- **File paths**: All non-flag arguments (required, space-separated)
- **Output file path**: From `--output-file` flag
- **Language override**: From `--language` flag (`nodejs`, `react`, `dotnet`)

Validate files: git repo presence (fail if not), existence (exclude missing), change status (uncommitted vs none). Fail if no valid files remain.

**Output:** Valid file paths (split by has_changes / no changes), output file path, language override (if any).

## Content Gathering

Launch Sonnet agent to gather:

**File content:** For files with uncommitted changes: diff + full content. For files without changes: full content only.

**Related test files:** Read test files from Context Discovery step (patterns per `languages/nodejs.md` and `languages/dotnet.md`).

**Summary:** Files with changes vs reviewed in entirety, detected project type(s), related test files found.

**Output:** Current branch name. For each file: diff (if has_changes), full content, has_changes flag. Related test file contents. Review summary.
