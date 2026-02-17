# Documentation Review Orchestration

- Use git CLI (not GitHub CLI). Create todo list before starting.
- Cite issues with `file:line` (e.g., `docs/api.md:42-48`). Quote exact AI instruction rules when violated.
- Paths relative to repo root. Line numbers reference file lines (not diff lines).

## Agent Invocation

### Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `MODE` | string | Yes | `thorough`, `gaps`, or `quick` |
| `project_type` | string | Yes | `nodejs`, `dotnet`, or both |
| `files_to_review` | list | Yes | See File Entry Schema below |
| `ai_instructions` | list | Optional | Summary of AI instruction files |
| `previous_findings` | list | Gaps only | Prior findings for dedup. Each: `title`, `file`, `line`, `range` (string or null), `category`, `severity` |
| `skill_instructions` | object | Optional | From `--skills`. Fields: `focus_areas`, `checklist` [{category, severity, items}], `auto_validate`, `false_positive_rules`, `methodology` {approach, steps, questions} |
| `additional_instructions` | string | Optional | Settings body + `--prompt` + orchestrator-injected rules |

### File Entry Schema

**Critical files** (has_changes: true, tier: "critical"): `path`, `diff` (unified), `full_content`

**Peripheral files** (staged — unchanged context, has_changes: false, tier: "peripheral"): `path`, `preview` (first ~50 lines), `line_count`, `full_content_available`: true (agent uses Read tool)

End prompt with: `Return findings as YAML per agent examples in your agent file.`

### Content Distribution

Per agent, append to `additional_instructions`:
1. Output schema, MODE definition, FP rules from Agent Common Instructions below
2. Category-specific FP rules — agent's category only

Synthesis agents skip distribution.

### Agent Common Instructions

**MODE:** thorough (all issues), gaps (subtle issues missed; dedup against previous_findings), quick (critical/merge-blocking only). Deep reviews skip pre-existing and silenced issues. Quick: only blocking issues, skip theoretical edge cases.

**False Positive Rules — do NOT flag:**
- Pre-existing issues not modified in current changes
- General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- Code with lint-disable/suppress comments or documented suppressions

**Output Schema:** Each issue: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`. See agent file for category-specific extra fields.

### Category-Specific False Positive Rules

- **Accuracy**: Intentionally simplified examples (marked "simplified"/"basic example"); pseudocode marked as illustrative; documentation for planned features marked as such
- **Clarity**: Jargon appropriate for stated expert audience; acronyms defined earlier in same document; intentionally terse reference docs (vs tutorials)
- **Completeness**: Internal/private APIs not for external use; features marked experimental/unstable; sections that would duplicate linked content
- **Consistency**: Code/API names that must match implementation (even if inconsistent with prose); quoted text preserving original formatting; version-specific sections that intentionally differ
- **Examples**: Pseudocode marked as illustrative (not runnable); partial examples with "..." for omitted code; examples showing error cases (intentionally incorrect); shell examples with placeholder values like `<your-token>`
- **Structure**: Intentionally orphaned archive/historical documents; heading hierarchy violations in code-generated documentation; AI instruction files in projects not using AI assistants (if explicitly stated)

## Gaps Mode

When MODE=gaps, agents receive `previous_findings` from thorough mode.

**Duplicate Detection:** Skip issues in same file within +/-5 lines of prior findings. Skip same issue type on same function/method. For range findings (lines A-B): skip zone = [A-5, B+5]. See Prompt Schema `previous_findings` field for schema.

**Constraints:** Only Major/Critical severity (skip Minor/Suggestion). Maximum 5 new findings per agent. Model: always Sonnet.

**Agents:** accuracy, completeness, consistency. See each agent file for category-specific gaps focus areas.

## Deep Docs Review Sequence (13 invocations)

**Phase 1 — Thorough** (6 parallel): accuracy, clarity, completeness, consistency, examples, structure. MODE: thorough. Models: Opus for accuracy, completeness, examples; Sonnet for rest. Pass: doc contents, related code snippets, project metadata, AI instruction file status, additional prompt instructions.
**CRITICAL: WAIT for ALL 6 before Phase 2.**

**Phase 2 — Gaps** (3 parallel Sonnet): accuracy, completeness, consistency. MODE: gaps. Input: Phase 1 findings as `previous_findings`. Distribute Gaps Mode rules to each agent's `additional_instructions`.
**CRITICAL: WAIT for ALL 3 before Synthesis.**

**Synthesis** (4 parallel): 4 instances of synthesis-docs-agent. Input: ALL Phase 1+2 findings.
**CRITICAL: WAIT for Phase 1 AND Phase 2 fully complete before starting.**
Pairs: Accuracy+Examples ("Do code examples match documented behavior?"), Clarity+Structure ("Does poor structure contribute to clarity issues?"), Completeness+Consistency ("Are missing sections causing terminology inconsistencies?"), Consistency+Structure ("Do formatting inconsistencies reflect structural problems?")

**Post-review:** Validate all issues → filter invalid → deduplicate → generate report.

## Quick Docs Review Sequence (7 invocations)

**Review** (4 parallel): accuracy, clarity, examples, structure. MODE: quick. Models: accuracy, examples (Opus); clarity, structure (Sonnet). Pass: doc contents, related code snippets, AI instruction file status, additional prompt instructions.
- Accuracy: Critical mismatches, wrong function names, broken examples
- Clarity: Incomprehensible sections, undefined critical acronyms
- Examples: Syntax errors, missing critical imports, wrong API calls
- Structure: Broken links, major navigation issues, AI instruction file errors

**Synthesis** (3 parallel): Accuracy+Examples ("Do examples match documented behavior?"), Clarity+Structure ("Does structure contribute to clarity issues?"), Examples+Structure ("Are example placements structurally sound?")

**Post-review:** Same as deep.

## Model Selection (Authoritative)

Default: **Sonnet** for all agents and modes.
**Opus exceptions (thorough):** accuracy, completeness, examples
**Opus exceptions (quick):** accuracy, examples
**Gaps:** Always Sonnet (no exceptions)

## Synthesis Invocation

Invoke synthesis agents multiple times in parallel with different category pairs. See `${CLAUDE_PLUGIN_ROOT}/agents/docs/synthesis-docs-agent.md`.

---

Validation, aggregation, and auto-validation: See `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-docs.md`.
