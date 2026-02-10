# Staged Processing: Validation and Content Gathering

This document combines input validation and content gathering for staged commands (`/deep-code-review-staged`, `/quick-code-review-staged`).

## Contents

- [Input Validation](#input-validation)
- [Content Gathering](#content-gathering)

---

## Input Validation

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

### Validation Output

Pass to Content Gathering:
- List of staged file paths
- Parsed output file path
- Parsed language override (if any)

---

## Content Gathering

Launch a Sonnet agent to gather the content to review:

### 1. Get Staged Diff

```bash
git diff --cached
```

### 2. Get Current Branch

```bash
git branch --show-current
```

### 3. Read Full File Content (Tiered)

For each staged file:
- **Changed files (has_changes=true)**: Read full file content (critical tier)
- **Unchanged files referenced in imports/context (has_changes=false)**: Read preview only (peripheral tier)

### 4. Read Related Test Files

Read related test files (from Context Discovery step) for context:
- **Node.js**: See `languages/nodejs.md` for test file patterns
- **.NET**: See `languages/dotnet.md` for test file patterns

Test files are always treated as critical tier.

### 5. Create Summary

Summarize what changes are being made:
- Number of files modified
- Brief description of the changes
- Detected project type(s)
- Related test files found
- Tier classification summary

### Gathering Output

Return to the review step:
- Current branch name
- For each file:
  - path
  - has_changes (true/false)
  - **tier**: "critical" or "peripheral"
  - For **critical** files (has_changes=true):
    - diff: full diff content
    - full_content: full file content
  - For **peripheral** files (has_changes=false):
    - preview: first 50 lines of file
    - line_count: total lines in file
    - full_content_available: true
- Related test files (always critical tier)
- Summary of what's being reviewed

### Tier Classification

| File Type | Tier | Rationale |
|-----------|------|-----------|
| Files with staged changes | critical | Primary review focus |
| Related test files | critical | Need full content for coverage analysis |
| AI instruction files | critical | Always fully loaded |
| Unchanged source files | peripheral | Context only - agents Read on-demand |

## Tiered Context Behavior

Staged reviews use tiered context to balance thoroughness with context efficiency:
- Changed files receive full content (critical tier)
- Unchanged files receive preview + metadata (peripheral tier)
- Agents can Read peripheral files on-demand if cross-file analysis discovers relevance

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

When files include tier information (staged reviews):

**For `tier: "critical"` files:**
- Full content is provided - analyze thoroughly
- This is the primary review focus

**For `tier: "peripheral"` files:**
- Only a preview (first 50 lines) is provided
- Use the preview to understand file purpose
- If cross-file analysis discovers relevance, use Read tool to get full content
