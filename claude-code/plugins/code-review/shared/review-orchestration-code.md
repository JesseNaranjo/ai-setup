# Code Review Orchestration

This document defines agent invocation patterns and execution sequences for code review pipelines.

## Orchestrator Notes

- Use git CLI (not GitHub CLI). Create a todo list before starting.
- Cite issues with file path and line numbers (e.g., `src/utils.ts:42-48`). Quote exact AI instruction rules when violated.
- Paths relative to repo root. Line numbers reference actual file lines (not diff lines for file reviews, working copy lines for staged reviews).
- Full file reviews (no changes): be more lenient — focus on clear bugs, not style.

## Agent Invocation

Always pass the `model` parameter explicitly (see Code Review Model Selection section below).

### Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `MODE` | string | Yes | `thorough`, `gaps`, or `quick` |
| `project_type` | string | Yes | `nodejs`, `dotnet`, or both |
| `files_to_review` | list | Yes | See File Entry Schema below |
| `ai_instructions` | list | Conditional | Full content for architecture/compliance agents; summary-only for others |
| `related_tests` | list | Conditional | Only for bug-detection, technical-debt, test-coverage agents |
| `previous_findings` | list | Gaps only | Prior findings for deduplication. Each entry: `title`, `file`, `line`, `range` (string or null), `category`, `severity` |
| `skill_instructions` | object | Optional | From `--skills` argument. Fields: `focus_areas` (list), `checklist` (list of {category, severity, items}), `auto_validate` (list), `false_positive_rules` (list), `methodology` ({approach, steps, questions}) |
| `additional_instructions` | string | Optional | Combined content from settings file body + `--prompt` argument + orchestrator-injected rules (gaps behavior, pre-existing detection) |

### File Entry Schema

**Critical files** (has_changes: true, tier: "critical"):
`path` (string), `diff` (unified diff), `full_content` (complete file)

**Peripheral files** (staged reviews — unchanged context files, has_changes: false, tier: "peripheral"):
`path` (string), `preview` (first ~50 lines), `line_count` (integer), `full_content_available`: true (agent uses Read tool)

End the prompt with: `Return findings as YAML per agent examples in your agent file.`

### Content Distribution Optimization

#### Agent Common Content Distribution

**Per agent** (append to `additional_instructions`):
1. Output schema, MODE definition, and false positive rules from Agent Common Instructions below
2. Category-specific FP rules — extract ONLY the agent's category
3. Language-specific checks — extract agent's anchor from detected language files
4. LSP diagnostic codes — if available, extract agent's matching category rows plus Agent Usage Guidelines from lsp-integration.md

**Language anchors:** architecture→#architecture, bugs→#bugs, errors→#errors, performance→#performance, security→#security, debt→#debt, tests→#tests. Agents without anchors (api-contracts, compliance) skip step 3. Synthesis agents skip all distribution.

**LSP categories:** architecture→Architecture, bugs→Bugs, errors→Error Handling, performance→Performance, security→Security. If LSP unavailable, skip entirely.

**Selective distribution:**
- `related_tests` → bug-detection, technical-debt, test-coverage only (others use Grep/Glob on-demand)
- Full `ai_instructions` → architecture, compliance only (others get path summary)

### Agent Common Instructions (Distributed to All Agents)

#### MODE Parameter

MODE: thorough (all issues), gaps (subtle issues missed; dedup against previous_findings), quick (critical/merge-blocking only). Deep reviews skip pre-existing and silenced issues. Quick reviews: only blocking issues, skip theoretical edge cases.

#### False Positive Rules

Do NOT flag:
- **Pre-existing Issues**: Issues existing before current changes, not modified
- **Scope Limitations**: General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- **Silenced Issues**: Code with lint-disable/suppress comments or documented suppressions

#### Output Schema

Each issue requires: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`.

See agent file for category-specific extra fields.

### Category-Specific False Positive Rules (Code)

Each category has specific exclusions in addition to the general false positive rules above.

- **API Contracts**: Additive-only changes; beta/experimental APIs marked as unstable; changes following established deprecation process
- **Architecture**: Pragmatic compromises with clear justification; patterns overkill for project scale
- **Bug Detection**: Code with explicit comments explaining why it's correct
- **Compliance**: Explicit override comments; ambiguous rules with reasonable compliance; style preferences not stated as rules
- **Error Handling**: Errors intentionally ignored with explicit comments; logging-only catch blocks as intended behavior
- **Performance**: Micro-optimizations without measurable impact; code that runs rarely
- **Security**: Internal-only code with no untrusted input exposure; vulnerabilities mitigated elsewhere in the code
- **Technical Debt**: Dependencies pinned for compatibility (documented reason); dead code that's conditionally compiled (build flags); TODO comments referencing issue tracking (TODO(#123)); workarounds with documented upstream bugs; class components in projects intentionally supporting older React versions
- **Test Coverage**: Code impractical to unit test (better for integration tests); code covered by higher-level tests; generated code/boilerplate; dead code that should be removed rather than tested

## Gaps Mode Behavior

When MODE=gaps, agents receive `previous_findings` from thorough mode to avoid duplicates.

### Duplicate Detection (Common to All Gaps Agents)

- Skip issues in same file within +/-5 lines of prior findings
- Skip same issue type on same function/method
- For range findings (lines A-B): skip zone = [A-5, B+5]

See Prompt Schema table above (`previous_findings` field) for the schema.

### Constraints (Common to All Gaps Agents)

- Only report Major or Critical severity (skip Minor/Suggestion)
- Maximum 5 new findings per agent
- Model: Always Sonnet (cost optimization)

### Gaps-Supporting Agents

**Code review:** bug-detection, compliance, performance, security, technical-debt.

See each agent file for category-specific focus areas (what subtle issues thorough mode misses).

## Deep Code Review Sequence (19 agent invocations)

1. **Steps 1-3: Input, Context, Content** (defined in command file)

2. **Phase 1: Thorough Review** (9 agents in parallel)
   - Launch: api-contracts, architecture, bug-detection, compliance, error-handling, performance, security, technical-debt, test-coverage
   - Models: architecture, bug-detection, performance, security, technical-debt (Opus); api-contracts, compliance, error-handling, test-coverage (Sonnet)
   - MODE: `thorough` for all agents
   - **CRITICAL: WAIT** - DO NOT proceed to Phase 2 until ALL 9 agents complete
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (5 Sonnet agents in parallel)
   - Launch: bug-detection, compliance, performance, security, technical-debt
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - Content: Diff only (no full file content — agents use Read tool for deeper analysis)
   - Distribute Gaps Mode Behavior rules from this document (duplicate detection skip zones, severity constraints, 5-finding cap) to each gaps agent's `additional_instructions`
   - **CRITICAL: WAIT** - DO NOT proceed to Synthesis until ALL 5 agents complete
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (5 agents in parallel)
   - **CRITICAL: DO NOT START until Phase 1 AND Phase 2 are FULLY COMPLETE**
   - Launch: 5 instances of synthesis-code-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Content: File paths only (no file content — agents Read for cross-reference)
   - Pairs and questions:
     - Architecture+Test Coverage: "Are architectural changes covered by tests?"
     - Bugs+Compliance: "Do compliance violations introduce or mask bugs?"
     - Bugs+Error Handling: "Do identified bugs have proper error handling in fix paths?"
     - Compliance+Technical Debt: "Do compliance violations indicate or worsen technical debt?"
     - Performance+Security: "Do any security fixes introduce performance issues?"
   - WAIT: All 5 must complete
   - OUTPUT: `cross_cutting_insights` list

5. **Validation, Aggregation, Output**
   - Validate all issues, filter invalid, deduplicate, generate report

## Quick Code Review Sequence (7 agent invocations)

1. **Steps 1-3: Input, Context, Content** (same as deep)

2. **Review** (4 agents in parallel)
   - Launch: bug-detection, error-handling, security, test-coverage
   - MODE: `quick` for all agents
   - OUTPUT: Quick review findings

3. **Synthesis** (3 agents in parallel)
   - Pairs and questions:
     - Bugs+Error Handling: "Do identified bugs have proper error handling in fix paths?"
     - Bugs+Security: "Do security issues introduce or relate to bugs?"
     - Bugs+Test Coverage: "Are identified bugs covered by tests?"
   - OUTPUT: `cross_cutting_insights` list

4. **Validation, Aggregation, Output** (same as deep)

## Code Review Model Selection (Authoritative Source)

Default model: **Sonnet** for all agents and modes.

**Opus exceptions (thorough):** architecture, bug-detection, performance, security, technical-debt
**Opus exceptions (quick):** bug-detection, security
**Gaps mode:** Always Sonnet (no exceptions)

## Language-Specific Focus

Load language configs ONLY for detected languages/frameworks:
- `detected_languages.dotnet` → `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`
- `detected_languages.nodejs` → `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`
- `detected_frameworks.react` → also `${CLAUDE_PLUGIN_ROOT}/languages/react.md`

## Synthesis Invocation

Invoke synthesis agents **multiple times in parallel** with different category pairs. See `${CLAUDE_PLUGIN_ROOT}/agents/code/synthesis-code-agent.md` for agent definition.

---

# Code Review Validation Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-code.md` for validation rules and auto-validation patterns. Load at Steps 9-12 only.
