# Docs Processing: Validation and Content Gathering

This document combines input validation and content gathering for documentation review commands (`/deep-docs-review`, `/quick-docs-review`).

## Contents

- [Input Validation](#input-validation)
- [Content Gathering](#content-gathering)

---

## Input Validation

Validate the input and discover documentation files:

### 1. Parse Arguments

Extract from the command arguments:
- **File paths**: Optional specific files to review (space-separated)
- **Output file path**: From `--output-file` flag (command provides default)
- **Additional instructions**: From `--prompt` flag

### 2. Verify Git Repository

Check if the current directory is a git repository:
```bash
git rev-parse --git-dir
```

If not a git repository, stop and inform the user: "Not a git repository."

### 3. Discover Documentation Files

If specific files provided, validate those. Otherwise, discover documentation:

**Standard documentation files (check root):**
- `README.md`
- `CLAUDE.md`
- `CHANGELOG.md`
- `CONTRIBUTING.md`
- `LICENSE.md`
- `CODE_OF_CONDUCT.md`

**Documentation directories:**
- `docs/**/*.md`
- `documentation/**/*.md`

**AI Agent Instruction Files:**
- `.ai/AI-AGENT-INSTRUCTIONS.md` (standard location)
- `AI-AGENT-INSTRUCTIONS.md` (root - should be moved to .ai/)
- `.github/copilot-instructions.md`
- `.github/ISSUE_TEMPLATE/*.md`
- `.github/PULL_REQUEST_TEMPLATE.md`

**Discovery commands:**
```bash
# Find standard docs
ls -la README.md CLAUDE.md CHANGELOG.md CONTRIBUTING.md 2>/dev/null

# Find docs directories
find docs documentation -name "*.md" -type f 2>/dev/null

# Find AI instruction files
ls -la .ai/AI-AGENT-INSTRUCTIONS.md AI-AGENT-INSTRUCTIONS.md .github/copilot-instructions.md 2>/dev/null
```

### 4. Check for AI Instruction File Standardization

Track standardization status for reporting:

**AI-AGENT-INSTRUCTIONS.md location:**
- If exists in `.ai/`: ✅ Correct location
- If exists in root but not `.ai/`: ⚠️ Should be moved to `.ai/`
- If missing entirely: ⚠️ Should be created

**CLAUDE.md header:**
- Check if references `.ai/AI-AGENT-INSTRUCTIONS.md`

**copilot-instructions.md:**
- If exists in `.github/`: ✅ Exists
- If missing: ⚠️ Should be created

### 5. Filter by Scope

If specific files were provided:
- Verify each file exists
- Only review specified files

If no files specified:
- Include all discovered documentation files
- Prioritize by importance: README > CLAUDE.md > CHANGELOG > docs/

### 6. Validate Scope

If no documentation files found, stop and inform the user: "No documentation files found. Ensure your project has README.md, docs/, or other markdown documentation."

### Validation Output

Pass to Content Gathering:
- List of documentation files to review
- AI instruction file standardization status
- Parsed output file path
- Parsed additional instructions (if any)

---

## Content Gathering

Launch a Sonnet agent to gather documentation content and related code references:

### 1. Get Current Branch

```bash
git branch --show-current
```

### 2. Read Documentation Files

For each documentation file identified in validation:
- Read the full file content
- Note the file's location and purpose (README, API docs, tutorial, etc.)

### 3. Extract Code References

Scan documentation for code references that need verification:

**Inline code references:**
- Function/method names mentioned in backticks
- Class/type names
- CLI commands and flags
- Configuration keys

**Code block references:**
- File paths mentioned above code blocks (e.g., "In `src/utils.ts`:")
- Import statements in examples
- Function signatures shown

**API documentation:**
- Endpoint paths
- Request/response schemas
- Parameter names and types

### 4. Gather Related Code Files

For accuracy verification, read the actual code files referenced:

```bash
# Find files mentioned in documentation
grep -oE '`[a-zA-Z0-9_/.-]+\.(ts|js|py|cs|go|rs)`' <doc-file> | tr -d '`' | sort -u
```

For each referenced code file that exists:
- Read the file (or relevant sections)
- Extract function signatures, class definitions, exports
- Note any version-specific behavior

### 5. Check Package/Project Metadata

Read project metadata for version verification:

**Node.js:**
- `package.json` (version, scripts, dependencies)

**.NET:**
- `*.csproj` (version, package references)

**Python:**
- `pyproject.toml` or `setup.py` (version, dependencies)

**General:**
- `VERSION` file if present
- Git tags for release versions

### 6. Gather AI Instruction File Context

If reviewing AI instruction files, gather cross-reference context:

- Read all three files if they exist:
  - `.ai/AI-AGENT-INSTRUCTIONS.md`
  - `CLAUDE.md`
  - `.github/copilot-instructions.md`
- Check for header consistency
- Verify cross-references are valid

### 7. Create Summary

Summarize what will be reviewed:
- Documentation files and their types
- Code files gathered for accuracy verification
- Project metadata found
- AI instruction file standardization status

### Gathering Output

Return to the review step:
- Current branch name
- For each doc file: full content, type classification
- Referenced code snippets for verification
- Project metadata (versions, etc.)
- AI instruction file cross-reference status
- Summary of what's being reviewed
