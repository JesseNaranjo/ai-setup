# Documentation Review Orchestration

This document defines agent invocation patterns and execution sequences for documentation review pipelines.

## Orchestrator Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing AI Agent Instructions rules, quote the exact rule being violated.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not diff line numbers).

## Agent Invocation

Always pass the `model` parameter explicitly (see Documentation Review Model Selection table below).

### Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `MODE` | string | Yes | `thorough`, `gaps`, or `quick` |
| `project_type` | string | Yes | `nodejs`, `dotnet`, or both |
| `files_to_review` | list | Yes | See File Entry Schema below |
| `ai_instructions` | list | Optional | Summary of AI instruction files for context |
| `previous_findings` | list | Gaps only | Prior findings for deduplication. Each entry: `title`, `file`, `line`, `range` (string or null), `category`, `severity` |
| `skill_instructions` | object | Optional | From `--skills` argument. Fields: `focus_areas` (list), `checklist` (list of {category, severity, items}), `auto_validate` (list), `false_positive_rules` (list), `methodology` ({approach, steps, questions}) |
| `additional_instructions` | string | Optional | Combined content from settings file body + `--prompt` argument + orchestrator-injected rules (gaps behavior, pre-existing detection) |

### File Entry Schema

**Critical files** (has_changes: true, tier: "critical"):

| Field | Type | Notes |
|-------|------|-------|
| `path` | string | Relative file path |
| `has_changes` | boolean | `true` |
| `tier` | string | `"critical"` |
| `diff` | string | Unified diff content |
| `full_content` | string | Complete file content |

**Peripheral files** (staged reviews — unchanged context files):

| Field | Type | Notes |
|-------|------|-------|
| `path` | string | Relative file path |
| `has_changes` | boolean | `false` |
| `tier` | string | `"peripheral"` |
| `preview` | string | First ~50 lines |
| `line_count` | integer | Total lines in file |
| `full_content_available` | boolean | `true` (agent can use Read tool) |

End the prompt with: `Return findings as YAML per agent examples in your agent file.`

### Content Distribution Optimization

#### Agent Common Content Distribution

The orchestrator distributes relevant portions of the content below to each agent via `additional_instructions`, eliminating per-agent file reads.

**Distribution per agent** (append to `additional_instructions`):
1. **Output schema** from Agent Common Instructions below
2. **MODE definition** matching the current mode from Agent Common Instructions below
3. **General false positive rules** from Agent Common Instructions below
4. **Category-specific false positive rules** — extract ONLY the agent's category from Category-Specific False Positive Rules below

Synthesis agents are excluded from this distribution.

### Agent Common Instructions (Distributed to All Agents)

#### Standard Agent Input

**Required:** files_to_review (diffs/content), project_type (nodejs/dotnet/both), MODE (thorough/gaps/quick)
**Optional:** skill_instructions (skill-derived focus), previous_findings (gaps mode deduplication)
**Tools:** Read, Grep, Glob

#### MODE Parameter

- **thorough**: Comprehensive review, all issues in agent's domain
- **gaps**: Subtle issues missed by thorough; receives previous_findings to skip duplicates
- **quick**: Critical issues only (highest-impact, merge-blocking)

#### False Positive Rules

**Do NOT flag:**
- **Correct Code**: Non-obvious but valid edge case handling or intentional patterns
- **Linter Territory**: Formatting/import issues handled by linters (do NOT run linters to verify)
- **Pedantic Concerns**: Minor style preferences a senior engineer would not flag
- **Pre-existing Issues**: Issues existing before current changes, not modified
- **Scope Limitations**: General quality unless required in AI instructions; test code unless reviewing tests; theoretical edge cases extremely unlikely in practice
- **Silenced Issues**: Code with lint-disable/suppress comments or documented suppressions

**Deep review** can flag more issues but still skip pre-existing, silenced, and pure style.
**Quick review**: only blocking issues; ignore minor style, skip theoretical edge cases.

#### Output Schema

Each issue requires: `title`, `file`, `line`, `range` (string or null), `category`, `severity` (Critical/Major/Minor/Suggestion), `description`, `fix_type` (diff/prompt), `fix_diff` or `fix_prompt`.

See agent file for category-specific extra fields.

### Category-Specific False Positive Rules (Documentation)

Each category has specific exclusions in addition to the general false positive rules above.

- **Accuracy**: Intentionally simplified examples (marked "simplified"/"basic example"); pseudocode marked as illustrative; documentation for planned features marked as such
- **Clarity**: Jargon appropriate for stated expert audience; acronyms defined earlier in the same document; intentionally terse reference docs (vs tutorials)
- **Completeness**: Internal/private APIs not for external use; features marked experimental/unstable; sections that would duplicate linked content
- **Consistency**: Code/API names that must match implementation (even if inconsistent with prose); quoted text preserving original formatting; version-specific sections that intentionally differ
- **Examples**: Pseudocode marked as illustrative (not runnable); partial examples with "..." for omitted code; examples showing error cases (intentionally incorrect); shell examples with placeholder values like `<your-token>`
- **Structure**: Intentionally orphaned archive/historical documents; heading hierarchy violations in code-generated documentation; AI instruction files in projects not using AI assistants (if explicitly stated)

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

**Documentation review:** accuracy, completeness, consistency.

See each agent file for category-specific focus areas (what subtle issues thorough mode misses).

## Deep Docs Review Sequence (13 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Discover and validate documentation files
   - Gather doc content and related code references
   - OUTPUT: Documentation files, related code snippets, AI instruction files

2. **Phase 1: Thorough Review** (6 agents in parallel)
   - Launch: accuracy, clarity, completeness, consistency, examples, structure
   - Models: accuracy, completeness, examples (Opus); clarity, consistency, structure (Sonnet)
   - MODE: `thorough` for all agents
   - Pass to all agents: documentation file contents, related code snippets for verification, project metadata, AI instruction file status, additional prompt instructions (if provided)
   - **CRITICAL: WAIT** - DO NOT proceed to Phase 2 until ALL 6 agents complete
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (3 Sonnet agents in parallel)
   - Launch: accuracy, completeness, consistency
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
   - Distribute Gaps Mode Behavior rules from this document (duplicate detection skip zones, severity constraints, 5-finding cap) to each gaps agent's `additional_instructions`
   - **CRITICAL: WAIT** - DO NOT proceed to Synthesis until ALL 3 agents complete
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (4 agents in parallel)
   - **CRITICAL: DO NOT START until Phase 1 AND Phase 2 are FULLY COMPLETE**
   - Launch: 4 instances of synthesis-docs-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
   - Pairs and questions:
     - Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
     - Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
     - Completeness+Consistency: "Are missing sections causing terminology inconsistencies elsewhere?"
     - Consistency+Structure: "Do formatting inconsistencies reflect structural organization problems?"
   - WAIT: All 4 must complete
   - OUTPUT: `cross_cutting_insights` list

5. **Validation, Aggregation, Output**
   - Validate all issues, filter invalid, deduplicate, generate report

## Quick Docs Review Sequence (7 agent invocations)

1. **Steps 1-3: Input, Context, Content** (same as deep)

2. **Review** (4 agents in parallel)
   - Launch: accuracy, clarity, examples, structure
   - Models: accuracy (Opus), clarity (Sonnet), examples (Opus), structure (Sonnet)
   - MODE: `quick` for all agents
   - Accuracy: Critical mismatches, wrong function names, broken examples
   - Clarity: Incomprehensible sections, undefined critical acronyms
   - Examples: Syntax errors, missing critical imports, wrong API calls
   - Structure: Broken links, major navigation issues, AI instruction file errors
   - Pass to all agents: documentation file contents, related code snippets for verification, AI instruction file status, additional prompt instructions (if provided)
   - OUTPUT: Quick review findings

3. **Synthesis** (3 agents in parallel)
   - Pairs and questions:
     - Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
     - Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
     - Examples+Structure: "Are example placements and references structurally sound?"
   - OUTPUT: `cross_cutting_insights` list

4. **Validation, Aggregation, Output** (same as deep)

## Documentation Review Model Selection (Authoritative Source)

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| accuracy-agent | opus | sonnet | opus |
| clarity-agent | sonnet | N/A | sonnet |
| completeness-agent | opus | sonnet | N/A |
| consistency-agent | sonnet | sonnet | N/A |
| examples-agent | opus | N/A | opus |
| structure-agent | sonnet | N/A | sonnet |
| synthesis-docs-agent | sonnet | N/A | sonnet |

## Synthesis Invocation

The synthesis agents are designed to be invoked **multiple times in parallel** with different category pairs.

See `${CLAUDE_PLUGIN_ROOT}/agents/docs/synthesis-docs-agent.md` for the full agent definition and analysis logic.

---

# Documentation Review Validation Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-docs.md` for validation rules and auto-validation patterns. Load at Steps 9-12 only.
