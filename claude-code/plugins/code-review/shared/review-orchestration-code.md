# Code Review Orchestration

- Use git CLI (not GitHub CLI). Create todo list before starting.
- Cite issues with `file:line` (e.g., `src/utils.ts:42-48`). Quote exact AI instruction rules when violated.
- Paths relative to repo root. Line numbers reference file lines (not diff lines). Full file reviews (no changes): focus on clear bugs, not style.

## Agent Invocation

### Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `MODE` | string | Yes | `thorough`, `gaps`, or `quick` |
| `project_type` | string | Yes | `nodejs`, `dotnet`, or both |
| `files_to_review` | list | Yes | See File Entry Schema below |
| `ai_instructions` | list | Conditional | Full content for architecture/compliance agents; summary-only for others |
| `related_tests` | list | Conditional | Only for bug-detection, technical-debt, test-coverage agents |
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
3. Language-specific checks — agent's anchor from detected language files
4. LSP diagnostic codes — agent's category rows + usage guidelines from lsp-integration.md

**Language anchors:** architecture→#architecture, bugs→#bugs, errors→#errors, performance→#performance, security→#security, debt→#debt, tests→#tests. Agents without anchors (api-contracts, compliance) skip step 3. Synthesis agents skip all distribution.

**LSP categories:** architecture→Architecture, bugs→Bugs, errors→Error Handling, performance→Performance, security→Security. If unavailable, skip.

**Selective:** `related_tests` → bug-detection, technical-debt, test-coverage only. Full `ai_instructions` → architecture, compliance only (others get path summary).

### Agent Common Instructions

**MODE:** thorough (all issues), gaps (subtle issues missed; dedup against previous_findings), quick (critical/merge-blocking only). Deep reviews skip pre-existing and silenced issues. Quick: only blocking issues, skip theoretical edge cases.

**False Positive Rules — do NOT flag:**
- Pre-existing issues not modified in current changes
- General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- Code with lint-disable/suppress comments or documented suppressions

**Output Schema:** Each issue: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`. See agent file for category-specific extra fields.

### Category-Specific False Positive Rules

- **API Contracts**: Additive-only changes; beta/experimental APIs marked unstable; changes following established deprecation
- **Architecture**: Pragmatic compromises with clear justification; patterns overkill for project scale
- **Bug Detection**: Code with explicit comments explaining why it's correct
- **Compliance**: Explicit override comments; ambiguous rules with reasonable compliance; style preferences not stated as rules
- **Error Handling**: Errors intentionally ignored with explicit comments; logging-only catch blocks as intended
- **Performance**: Micro-optimizations without measurable impact; code that runs rarely
- **Security**: Internal-only code with no untrusted input; vulnerabilities mitigated elsewhere
- **Technical Debt**: Dependencies pinned for compatibility (documented); dead code conditionally compiled (build flags); TODO with issue tracking (TODO(#123)); workarounds with documented upstream bugs; class components in projects supporting older React
- **Test Coverage**: Code impractical to unit test (better for integration); code covered by higher-level tests; generated code/boilerplate; dead code to remove rather than test

## Gaps Mode

When MODE=gaps, agents receive `previous_findings` from thorough mode.

**Duplicate Detection:** Skip issues in same file within +/-5 lines of prior findings. Skip same issue type on same function/method. For range findings (lines A-B): skip zone = [A-5, B+5]. See Prompt Schema `previous_findings` field for schema.

**Constraints:** Only Major/Critical severity (skip Minor/Suggestion). Maximum 5 new findings per agent. Model: always Sonnet.

**Agents:** bug-detection, compliance, performance, security, technical-debt. See each agent file for category-specific gaps focus areas.

## Deep Code Review Sequence (19 invocations)

**Phase 1 — Thorough** (9 parallel): api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage. MODE: thorough. Models: Opus for architecture, bug-detection, performance, security, technical-debt; Sonnet for rest.
**CRITICAL: WAIT for ALL 9 before Phase 2.**

**Phase 2 — Gaps** (5 parallel Sonnet): bug-detection, compliance, performance, security, technical-debt. MODE: gaps. Input: Phase 1 findings as `previous_findings`. Content: diff only (agents Read for deeper analysis). Distribute Gaps Mode rules to each agent's `additional_instructions`.
**CRITICAL: WAIT for ALL 5 before Synthesis.**

**Synthesis** (5 parallel): 5 instances of synthesis-code-agent. Input: ALL Phase 1+2 findings. Content: file paths only (agents Read for cross-reference).
**CRITICAL: WAIT for Phase 1 AND Phase 2 fully complete before starting.**
Pairs: Architecture+Test Coverage ("Are architectural changes covered by tests?"), Bugs+Compliance ("Do compliance violations introduce or mask bugs?"), Bugs+Error Handling ("Do bugs have proper error handling in fix paths?"), Compliance+Technical Debt ("Do compliance violations indicate or worsen debt?"), Performance+Security ("Do security fixes introduce performance issues?")

**Post-review:** Validate all issues → filter invalid → deduplicate → generate report.

## Quick Code Review Sequence (7 invocations)

**Review** (4 parallel): bug-detection, error-handling, security, test-coverage. MODE: quick.

**Synthesis** (3 parallel): Bugs+Error Handling ("Do bugs have proper error handling?"), Bugs+Security ("Do security issues relate to bugs?"), Bugs+Test Coverage ("Are bugs covered by tests?")

**Post-review:** Same as deep.

## Model Selection (Authoritative)

Default: **Sonnet** for all agents and modes.
**Opus exceptions (thorough):** architecture, bug-detection, performance, security, technical-debt
**Opus exceptions (quick):** bug-detection, security
**Gaps:** Always Sonnet (no exceptions)

## Language-Specific Focus

Load ONLY for detected languages: `detected_languages.dotnet` → `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`, `detected_languages.nodejs` → `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`, `detected_frameworks.react` → also `${CLAUDE_PLUGIN_ROOT}/languages/react.md`

## Synthesis Invocation

Invoke synthesis agents multiple times in parallel with different category pairs. See `${CLAUDE_PLUGIN_ROOT}/agents/code/synthesis-code-agent.md`.

---

Validation, aggregation, and auto-validation: See `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`.
