# Docs Processing: Validation and Content Gathering

## Input Validation

### Parse Arguments

Extract:
- **File paths**: Optional specific files (space-separated)
- **Output file path**: From `--output-file` flag
- **Additional instructions**: From `--prompt` flag

### Git Repository Check

Verify git repository exists (`git rev-parse --git-dir`). If not, stop: "Not a git repository."

### Discover Documentation Files

If specific files provided, validate those. Otherwise, discover:

**Standard documentation:** Root markdown (README.md, CHANGELOG.md, CLAUDE.md, CODE_OF_CONDUCT.md, CONTRIBUTING.md, LICENSE.md) + `docs/**/*.md`, `documentation/**/*.md`

**AI Agent Instruction Files:**
- `.ai/AI-AGENT-INSTRUCTIONS.md` (standard location)
- `.github/copilot-instructions.md`
- `.github/ISSUE_TEMPLATE/*.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `AI-AGENT-INSTRUCTIONS.md` (root - should be in `.ai/`)

### Track AI Instruction File Standardization

Track for reporting:

**AI-AGENT-INSTRUCTIONS.md location:**
- `.ai/`: ✅ Correct
- Root but not `.ai/`: ⚠️ Should move to `.ai/`
- Missing: ⚠️ Should create

**CLAUDE.md header:**
- Check references `.ai/AI-AGENT-INSTRUCTIONS.md`

**copilot-instructions.md:**
- `.github/`: ✅ Exists
- Missing: ⚠️ Should create

### Filter by Scope

**Specific files provided:** Verify existence, review only those. **No files:** Include all discovered; prioritize README > CLAUDE.md > CHANGELOG > docs/

If no files found, stop: "No documentation files found. Ensure your project has README.md, docs/, or other markdown documentation."

**Validation Output:** Documentation files to review, AI instruction file standardization status, output file path, additional instructions

---

## Content Gathering

Launch Sonnet agent to gather documentation content and code references.

### Required Data

**Current branch:** `git branch --show-current`

**Documentation files:** Read full content, classify type (README, API docs, tutorial, etc.)

**Code references:** Scan documentation for verifiable code references (function/class names, file paths, import statements, API endpoints, CLI commands). For referenced code files that exist, read relevant sections (signatures, definitions, exports) for accuracy verification.

**Project metadata:** Read `package.json`, `*.csproj`, `pyproject.toml`/`setup.py`, `VERSION` file, git tags for version verification.

**AI instruction file context (if reviewing AI instruction files):**
- Read `.ai/AI-AGENT-INSTRUCTIONS.md`, `CLAUDE.md`, `.github/copilot-instructions.md` if they exist
- Check header consistency
- Verify cross-references are valid

### Gathering Output

Return:
- Current branch name
- For each doc file: full content, type classification
- Referenced code snippets for verification
- Project metadata (versions, etc.)
- AI instruction file cross-reference status
- Summary of what's being reviewed
