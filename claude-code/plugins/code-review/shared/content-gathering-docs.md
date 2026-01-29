# Content Gathering: Documentation Commands

Shared content gathering logic for documentation review commands (`/deep-docs-review`, `/quick-docs-review`).

## Process

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

## Output

Return to the review step:
- Current branch name
- For each doc file: full content, type classification
- Referenced code snippets for verification
- Project metadata (versions, etc.)
- AI instruction file cross-reference status
- Summary of what's being reviewed
