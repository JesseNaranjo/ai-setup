# Staged Processing: Validation and Content Gathering

## Input Validation

**Git Repository Check**: Verify current directory is a git repository. If not, stop: "Not a git repository."

**Staged Changes Check**: Verify staged changes exist. If none, stop: "No staged changes to review."

**Argument Parsing**: Extract from command arguments:
- **Output file path**: From `--output-file` flag (command provides default)
- **Language override**: From `--language` flag (`nodejs` or `dotnet`, default: auto-detect per file)

**Validation Output**: Pass to Content Gathering: staged file paths, parsed output file path, parsed language override (if any)

## Content Gathering

Launch Sonnet agent to gather:

**1. Staged Diff**: Get full staged diff with line markers

**2. Current Branch**: Get branch name

**3. Full File Content (Tiered)**:
- **Changed files (has_changes=true)**: Read full file content (critical tier)
- **Unchanged files referenced in imports/context (has_changes=false)**: Read preview only (peripheral tier)

**4. Related Test Files**: Read test files from Context Discovery step (see `languages/nodejs.md` and `languages/dotnet.md` for patterns). Always critical tier.

**5. Summary**: Number of files modified, brief description of changes, detected project types, related test files found, tier classification summary

**Gathering Output**: Return to review step:
- Current branch name
- For each file:
  - path, has_changes (true/false), **tier** ("critical" or "peripheral")
  - **Critical files** (has_changes=true): diff (full diff content), full_content (full file content)
  - **Peripheral files** (has_changes=false): preview (first 50 lines), line_count (total lines), full_content_available: true
- Related test files (always critical tier)
- Summary of what's being reviewed

**Tier Classification**:

| File Type | Tier | Rationale |
|-----------|------|-----------|
| Files with staged changes | critical | Primary review focus |
| Related test files | critical | Need full content for coverage analysis |
| AI instruction files | critical | Always fully loaded |
| Unchanged source files | peripheral | Context only - agents Read on-demand |

## Pre-Existing Issue Detection (For Staged Reviews)

**CRITICAL**: When reviewing staged changes or diffs, agents must only flag issues in CHANGED lines.

**Context provided to agents:**
1. Diff content with line markers (lines starting with `+` are additions)
2. Surrounding unchanged lines (for understanding context only)
3. Full file content (for reference only, not for flagging issues)

**Rules for what to flag:**
- Issue is in a line starting with `+` in the diff (newly added code)
- Change INTRODUCES the issue (e.g., removes null check that protected existing code)
- Change WORSENS an existing issue (e.g., increases scope of vulnerability)

**Do NOT flag:**
- Issues in unchanged code (lines without `+` prefix)
- Pre-existing problems not made worse by the change
- Style issues in untouched code nearby
- Issues visible in "full file" context but not in the diff

## Tiered Context Agent Guidance

For staged reviews, inject the following into each agent's `additional_instructions`:

**For `tier: "critical"` files:**
- Full content is provided - analyze thoroughly
- This is the primary review focus

**For `tier: "peripheral"` files:**
- Only a preview (first 50 lines) is provided
- Use the preview to understand file purpose
- If cross-file analysis discovers relevance, use Read tool to get full content
