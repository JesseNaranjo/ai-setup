# Input Validation: Documentation Commands

Shared input validation logic for documentation review commands (`/deep-docs-review`, `/quick-docs-review`).

## Process

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

## Output

Return to the next step:
- List of documentation files to review
- AI instruction file standardization status
- Parsed output file path
- Parsed additional instructions (if any)
