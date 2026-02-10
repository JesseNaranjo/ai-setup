# Common Agent Instructions

This document contains shared instructions for all review agents. Agents reference this file instead of duplicating content.

## Standard Agent Input

All review agents receive the following inputs. Agents should NOT repeat this in their own documentation.

**Required inputs:**
- **Files to review**: Diffs and/or full content
- **Detected project type**: Node.js, .NET, or both
- **MODE parameter**: thorough, gaps, or quick (see MODE Parameter below)

**Optional inputs:**
- **skill_instructions**: Skill-derived focus areas and methodology
- **previous_findings**: Prior findings for gaps mode deduplication

**Available tools:** `["Read", "Grep", "Glob"]`

## MODE Parameter (Common)

All review agents accept a MODE parameter that controls review depth:

- **thorough**: Comprehensive review checking all issues in the agent's domain
- **gaps**: Focus on subtle issues that might be missed; receives prior findings context to skip duplicates
- **quick**: Fast pass on critical issues only (highest-impact findings)

See each agent file for mode-specific focus areas.

## Language-Specific Checks

When the detected project type includes a specific language, check the corresponding language file for category-specific patterns:

- **Node.js/TypeScript:** `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md#[your-category]`
- **.NET/C#:** `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md#[your-category]`
- **React:** `${CLAUDE_PLUGIN_ROOT}/languages/react.md#[your-category]` (extends Node.js)

## False Positive Rules

Issues that should NOT be flagged by review agents.

### Do NOT Flag

#### Correct Code
Code with non-obvious but valid edge case handling or intentional patterns.

#### Linter Territory
Formatting/import issues handled by linters/formatters (do NOT run linters to verify).

#### Pedantic Concerns
Minor style preferences a senior engineer would not flag.

#### Pre-existing Issues
Issues in the codebase that existed before the current changes and weren't modified.

#### Scope Limitations
General code quality concerns unless required in AI Agent Instructions. Issues in test code unless specifically reviewing tests. Theoretical edge cases extremely unlikely in practice.

#### Silenced Issues
Code with lint-disable/suppress comments or documented intentional suppressions.

### Deep vs Quick Review Differences

**Deep review** can flag more issues but should still avoid pre-existing issues, silenced issues, and pure style preferences.

**Quick review** is optimized for speed: focus only on blocking issues, ignore minor style concerns, skip theoretical edge cases.

### Category-Specific False Positive Rules

- **Code reviews:** See `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-code.md` "Category-Specific False Positive Rules (Code) > [Your Category]"
- **Docs reviews:** See `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-docs.md` "Category-Specific False Positive Rules (Documentation) > [Your Category]"

### Automatic Cross-File Analysis

Agents perform cross-file analysis automatically using tools when code suggests cross-cutting concerns (imports/exports, class/interface definitions, API contracts, shared types). See `${CLAUDE_PLUGIN_ROOT}/agents/code/architecture-agent.md` for detailed triggers.

## Output Schema

Use the YAML schema shown in your agent's examples. Each issue requires these base fields:
- `title`: Brief description
- `file`: File path relative to repo root
- `line`: Primary line number
- `range`: "start-end" for multi-line, null for single-line
- `category`: Agent's category name
- `severity`: Critical, Major, Minor, or Suggestion
- `description`: Detailed explanation
- `fix_type`: "diff" or "prompt"
- `fix_diff` or `fix_prompt`: The suggested fix

## Severity Definitions

| Severity | Threshold | Action |
|----------|-----------|--------|
| Critical | Exploitable vulnerabilities (SQLi, XSS, auth bypass), data loss/corruption, crashes (null deref in critical paths, infinite loops), broken core functionality | Block merge |
| Major | Conditional security issues (CSRF, info disclosure), significant perf degradation (O(n^2) hot paths, N+1, memory leaks), logic errors, breaking API changes | Fix before merge unless time-critical |
| Minor | Code quality (duplication, complexity, naming), non-critical perf, style inconsistencies, uncommon edge cases | Fix when convenient |
| Suggestion | Better alternatives, improvement opportunities, documentation gaps, test coverage ideas | Consider for future |

