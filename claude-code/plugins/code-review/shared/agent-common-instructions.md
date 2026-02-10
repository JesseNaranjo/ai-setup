# Common Agent Instructions

> **Consumption pattern:** This file is read by the Opus orchestrator and distributed to agents via additional_instructions. Agents do not read this file directly.

## Standard Agent Input

**Required:** files_to_review (diffs/content), project_type (nodejs/dotnet/both), MODE (thorough/gaps/quick)
**Optional:** skill_instructions (skill-derived focus), previous_findings (gaps mode deduplication)
**Tools:** Read, Grep, Glob

## MODE Parameter

- **thorough**: Comprehensive review, all issues in agent's domain
- **gaps**: Subtle issues missed by thorough; receives previous_findings to skip duplicates
- **quick**: Critical issues only (highest-impact, merge-blocking)

## False Positive Rules

**Do NOT flag:**
- **Correct Code**: Non-obvious but valid edge case handling or intentional patterns
- **Linter Territory**: Formatting/import issues handled by linters (do NOT run linters to verify)
- **Pedantic Concerns**: Minor style preferences a senior engineer would not flag
- **Pre-existing Issues**: Issues existing before current changes, not modified
- **Scope Limitations**: General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- **Silenced Issues**: Code with lint-disable/suppress comments or documented suppressions

**Deep review** can flag more issues but still skip pre-existing, silenced, and pure style.
**Quick review**: only blocking issues; ignore minor style, skip theoretical edge cases.

## Output Schema

Each issue requires: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`.

See agent file for category-specific extra fields.
