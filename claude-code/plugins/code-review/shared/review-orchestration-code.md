# Code Review Orchestration

- Use git CLI (not GitHub CLI). Cite issues with `file:line` (e.g., `src/utils.ts:42-48`). Quote exact AI instruction rules when violated.
- Paths relative to repo root. Line numbers reference file lines (not diff lines). Full file reviews (no changes): focus on clear bugs, not style.

## Agent Invocation

### Prompt Schema

Required: `MODE` (thorough/gaps/quick), `project_type` (nodejs/dotnet/both), `files_to_review` (see File Entry Schema).
Conditional: `ai_instructions` (full content for architecture/compliance; summary-only for others), `related_tests` (bug-detection, technical-debt, test-coverage only), `previous_findings` (gaps only; each: `title`, `file`, `line`, `range`, `category`, `severity`).
Optional: `skill_instructions` ({focus_areas, checklist [{category, severity, items}], auto_validate, false_positive_rules, methodology {approach, steps, questions}}), `additional_instructions` (settings body + review prompt + orchestrator-injected rules).

### File Entry Schema

**Critical files** (has_changes: true, tier: "critical"): `path`, `diff` (unified), `full_content`

**Peripheral files** (staged — unchanged context, has_changes: false, tier: "peripheral"): `path`, `preview` (first ~50 lines), `line_count`, `full_content_available`: true (agent uses Read tool)

End prompt with: `Return findings as YAML per agent examples in your agent file.`

### Content Distribution

Per agent, append to `additional_instructions`: (1) Language checks from detected language files using agent anchor. (2) LSP codes from `lsp-nodejs.md` or `lsp-dotnet.md` (per detected language) using agent category.

**Anchors:** architecture→#architecture, bugs→#bugs, errors→#errors, performance→#performance, security→#security, debt→#debt, tests→#tests. No anchors: api-contracts, compliance. Synthesis: skip all.

**Framework tags:** Language checks include framework-tagged entries (e.g., `[Express]`, `[Next.js]`, `[EF Core]`, `[Docker]`). Agents apply framework-tagged checks when reviewed file's framework context matches. No orchestrator filtering needed — agents use contextual judgment.

**LSP:** architecture→Architecture, bugs→Bugs, errors→Error Handling, performance→Performance, security→Security. Unavailable → skip.

**Selective:** `related_tests` → bug-detection, technical-debt, test-coverage only. Full `ai_instructions` → architecture, compliance only (others: path summary).

## Gaps Mode

Input: `previous_findings` from thorough mode. **Dedup:** Skip same file within +/-5 lines of prior findings; skip same issue type on same function/method; range (A-B): skip zone [A-5, B+5]. **Constraints:** Major/Critical only. Max 5 new per agent. Always Sonnet. **Agents:** bug-detection, compliance, performance, security, technical-debt.

## Deep Code Review Sequence (up to 21 invocations)

**Phase 1 — Thorough** (9 parallel): api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage. MODE: thorough.
**CRITICAL: WAIT for ALL 9 before Phase 2.**

**Phase 2 — Gaps** (5 parallel Sonnet): bug-detection, compliance, performance, security, technical-debt. MODE: gaps. Input: Phase 1 findings as `previous_findings`. Content: diff only (agents Read for deeper). Distribute Gaps Mode rules via `additional_instructions`.
**CRITICAL: WAIT for ALL 5 before Synthesis.**

**Synthesis** (up to 7 parallel): 7 instances of synthesis-code-agent. Input: ALL Phase 1+2 findings. Content: file paths only (agents Read to cross-reference). **Skip pairs where either category has zero Phase 1+2 findings.**
**CRITICAL: WAIT for Phase 1 AND Phase 2 fully complete before starting.**
Pairs: Architecture+Performance ("Do architectural changes introduce performance issues in hot paths?"), Architecture+Test Coverage ("Are architectural changes covered by tests?"), Bugs+Compliance ("Do compliance violations introduce or mask bugs?"), Bugs+Error Handling ("Do bugs have proper error handling in fix paths?"), Compliance+Technical Debt ("Do compliance violations indicate or worsen debt?"), Performance+Security ("Do security fixes introduce performance issues?"), Security+Test Coverage ("Are security fixes covered by corresponding security tests?")

**Post-review:** Validate all issues → filter invalid → deduplicate → generate report.

## Quick Code Review Sequence (up to 7 invocations)

**Review** (4 parallel): bug-detection, error-handling, security, test-coverage. MODE: quick.

**Synthesis** (up to 3 parallel): Bugs+Error Handling ("Do bugs have proper error handling?"), Bugs+Security ("Do security issues relate to bugs?"), Bugs+Test Coverage ("Are bugs covered by tests?"). **Skip pairs where either category has zero findings.**

**Post-review:** Same as deep.

## Model Selection

Default: agent `model` frontmatter. **Override:** Gaps → always Sonnet. Quick Opus exceptions: security.

## Language-Specific Focus

Load ONLY for detected languages: `detected_languages.dotnet` → `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`, `detected_languages.nodejs` → `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`, `detected_frameworks.react` → also `${CLAUDE_PLUGIN_ROOT}/languages/react.md`

## Synthesis Invocation

Parallel instances with different category pairs. See `${CLAUDE_PLUGIN_ROOT}/agents/code/synthesis-code-agent.md`.

---

Validation, aggregation, and auto-validation: See `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`.
