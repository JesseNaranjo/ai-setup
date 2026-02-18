# Staged Processing: Validation and Content Gathering

## Input Validation

Verify git repository (stop: "Not a git repository.") and staged changes exist (stop: "No staged changes to review.").

Parse arguments: `--output-file` (command provides default), `--language` (`nodejs` or `dotnet`, default: auto-detect per file).

**Output:** Staged file paths, parsed output file path, parsed language override (if any).

## Content Gathering

Launch Sonnet agent to gather:

**Staged diff:** Full staged diff with line markers.

**Current branch:** Branch name.

**Full file content (tiered):** Changed files (has_changes=true) → full content (critical tier). Unchanged files in imports/context (has_changes=false) → first 50 lines preview only (peripheral tier).

**Related test files:** From Context Discovery step (see `languages/nodejs.md` and `languages/dotnet.md`). Always critical tier.

**Summary:** Files modified count, change description, detected project types, related test files, tier classification.

**Output:** Current branch name. Per file: path, has_changes, tier ("critical"/"peripheral"). Critical files: diff + full_content. Peripheral files: preview (50 lines), line_count, full_content_available: true. Related test files (critical tier). Review summary.

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

Inject into each agent's `additional_instructions` for staged reviews:

**`tier: "critical"`:** Full content provided — analyze thoroughly. Primary review focus.

**`tier: "peripheral"`:** Preview only (first 50 lines). Use to understand file purpose. If cross-file analysis discovers relevance, use Read tool for full content.
