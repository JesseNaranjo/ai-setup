# Documentation Review Orchestration

- Use git CLI (not GitHub CLI). Cite issues with `file:line` (e.g., `docs/api.md:42-48`). Quote exact AI instruction rules when violated.
- Paths relative to repo root. Line numbers reference file lines (not diff lines).

## Agent Invocation

### Prompt Schema

Required: `MODE` (thorough/gaps/quick), `files_to_review` (see File Entry Schema).
Optional: `ai_instructions` (summary of AI instruction files), `previous_findings` (gaps only; each: `title`, `file`, `line`, `range`, `category`, `severity`), `skill_instructions` ({focus_areas, checklist [{category, severity, items}], auto_validate, false_positive_rules, methodology {approach, steps, questions}}), `additional_instructions` (settings body + `--prompt` + orchestrator-injected rules).

### File Entry Schema

**Critical files** (has_changes: true, tier: "critical"): `path`, `diff` (unified), `full_content`

**Peripheral files** (staged — unchanged context, has_changes: false, tier: "peripheral"): `path`, `preview` (first ~50 lines), `line_count`, `full_content_available`: true (agent uses Read tool)

End prompt with: `Return findings as YAML per agent examples in your agent file.`

### Content Distribution

Synthesis: skip `additional_instructions` distribution.

## Gaps Mode

Input: `previous_findings` from thorough mode. **Dedup:** Skip same file within +/-5 lines of prior findings; skip same issue type on same function/method; range (A-B): skip zone [A-5, B+5]. **Constraints:** Major/Critical only. Max 5 new per agent. Always Sonnet. **Agents:** accuracy, completeness, consistency.

## Deep Docs Review Sequence (up to 13 invocations)

**Phase 1 — Thorough** (6 parallel): accuracy, clarity, completeness, consistency, examples, structure. MODE: thorough. Pass: doc contents, related code snippets, project metadata, AI instruction file status, additional prompt instructions.
**CRITICAL: WAIT for ALL 6 before Phase 2.**

**Phase 2 — Gaps** (3 parallel Sonnet): accuracy, completeness, consistency. MODE: gaps. Input: Phase 1 findings as `previous_findings`. Distribute Gaps Mode rules via `additional_instructions`.
**CRITICAL: WAIT for ALL 3 before Synthesis.**

**Synthesis** (up to 4 parallel): 4 instances of synthesis-docs-agent. Input: ALL Phase 1+2 findings. **Skip pairs where either category has zero Phase 1+2 findings.**
**CRITICAL: WAIT for Phase 1 AND Phase 2 fully complete before starting.**
Pairs: Accuracy+Examples ("Do code examples match documented behavior?"), Clarity+Structure ("Does poor structure contribute to clarity issues?"), Completeness+Consistency ("Are missing sections causing terminology inconsistencies?"), Consistency+Structure ("Do formatting inconsistencies reflect structural problems?")

**Post-review:** Validate all issues → filter invalid → deduplicate → generate report.

## Quick Docs Review Sequence (up to 7 invocations)

**Review** (4 parallel): accuracy, clarity, examples, structure. MODE: quick. Pass: doc contents, related code snippets, AI instruction file status, additional prompt instructions.
- Accuracy: Critical mismatches, wrong function names, broken examples
- Clarity: Incomprehensible sections, undefined critical acronyms
- Examples: Syntax errors, missing critical imports, wrong API calls
- Structure: Broken links, major navigation issues, AI instruction file errors

**Synthesis** (up to 3 parallel): Accuracy+Examples ("Do examples match documented behavior?"), Clarity+Structure ("Does structure contribute to clarity issues?"), Examples+Structure ("Are example placements structurally sound?"). **Skip pairs where either category has zero findings.**

**Post-review:** Same as deep.

## Model Selection

Default: agent `model` frontmatter. **Override:** Gaps → always Sonnet. Quick Opus exceptions: accuracy.

## Synthesis Invocation

Parallel instances with different category pairs. See `${CLAUDE_PLUGIN_ROOT}/agents/docs/synthesis-docs-agent.md`.

---

Validation, aggregation, and auto-validation: See `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-docs.md`.
