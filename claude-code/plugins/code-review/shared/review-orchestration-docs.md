# Documentation Review Orchestration

This document defines agent invocation patterns and execution sequences for documentation review pipelines.

## Agent Invocation

Plugin agents are registered as subagent types with the pattern `code-review:<agent-name>`.

Use the Task tool to launch each agent. Always pass the `model` parameter explicitly (see Documentation Review Model Selection table below).

```
Task(
  subagent_type: "code-review:<agent-name>",
  model: "<model>",  // See Documentation Review Model Selection table
  description: "[Agent name] review for [scope]",
  prompt: "<prompt fields below>"
)
```

### Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `MODE` | string | Yes | `thorough`, `gaps`, or `quick` |
| `project_type` | string | Yes | `nodejs`, `dotnet`, or both |
| `files_to_review` | list | Yes | See File Entry Schema below |
| `ai_instructions` | list | Optional | Summary of AI instruction files for context |
| `previous_findings` | list | Gaps only | Prior findings for deduplication. Each entry: `title`, `file`, `line`, `range` (string or null), `category`, `severity` |
| `skill_instructions` | object | Optional | From `--skills` argument. Fields: `focus_areas` (list), `checklist` (list of {category, severity, items}), `auto_validate` (list), `false_positive_rules` (list), `methodology` ({approach, steps, questions}) |
| `additional_instructions` | string | Optional | Combined content from settings file body + `--prompt` argument |

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

## Deep Docs Review Sequence (13 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Discover and validate documentation files
   - Gather doc content and related code references
   - OUTPUT: Documentation files, related code snippets, AI instruction files

2. **Phase 1: Thorough Review** (6 agents in parallel)
   - Launch: accuracy, clarity, completeness, consistency, examples, structure
   - Models: accuracy, completeness, examples (Opus); clarity, consistency, structure (Sonnet)
   - MODE: `thorough` for all agents
   - **CRITICAL: WAIT** - DO NOT proceed to Phase 2 until ALL 6 agents complete
   - OUTPUT: Phase 1 findings (grouped by category)

3. **Phase 2: Gaps Review** (3 Sonnet agents in parallel)
   - Launch: accuracy, completeness, consistency
   - MODE: `gaps`
   - Model: Sonnet (cost-optimized for constrained task)
   - INPUT: Phase 1 findings passed as `previous_findings`
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
   - MODE: `quick` for all agents
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

### Synthesis Prompt Schema

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `synthesis_input.category_a.name` | string | Yes | First category name |
| `synthesis_input.category_a.findings` | list | Yes | All findings from category_a (Phase 1 + Phase 2) |
| `synthesis_input.category_b.name` | string | Yes | Second category name |
| `synthesis_input.category_b.findings` | list | Yes | All findings from category_b (Phase 1 + Phase 2) |
| `synthesis_input.cross_cutting_question` | string | Yes | The cross-cutting analysis question |
| `synthesis_input.files_content` | list | Yes | File diffs and full content for context |

### Parallel Synthesis Pattern

Commands launch multiple instances of the synthesis agent simultaneously, each with different category pairs. Each instance operates independently and returns its own `cross_cutting_insights` list. The orchestrating command merges all results.

**Authoritative source for category pairs:** See Deep Docs Review Sequence (4 pairs) and Quick Docs Review Sequence (3 pairs) above.

---

# Documentation Review Validation Rules

This section defines the validation process and domain-specific patterns for documentation review commands.

## Batch Validation Process

To optimize cost and latency, issues are validated in batches grouped by file rather than individually.

### Grouping Strategy

1. **Group by file**: All issues for the same file are validated in a single agent call
2. **Group by validator model**: Issues requiring same model (Opus vs Sonnet) are grouped together
3. **Benefit**: Reduces agent invocations from N issues to M files (typically 60-80% reduction)

### Validator Model Assignment

| Issue Category | Validator Model |
|----------------|-----------------|
| Accuracy | Opus |
| Clarity | Sonnet |
| Completeness | Opus |
| Consistency | Sonnet |
| Examples | Opus |
| Structure | Sonnet |

**Cross-cutting insights** (from synthesis-docs-agent) always use **Opus** for validation.

**Note for Quick Reviews:** Despite the quick review philosophy of using Sonnet where possible, cross-cutting insights still use Opus for validation because they represent novel connections between categories that require more nuanced judgment to validate. The time savings of Sonnet validation does not justify the risk of missing subtle cross-category interactions.

### Quick Review Validation Scope

Quick reviews validate **Critical and Major severity issues only**. Minor issues and Suggestions skip the validation phase entirely to optimize for speed. This means:

- Critical/Major issues: Full validation with appropriate model
- Minor issues: Auto-accepted without validation
- Suggestions: Auto-accepted without validation

This differs from deep reviews, which validate all severity levels.

### Cross-Cutting Insight Validation

Synthesis agents produce `cross_cutting_insights` that require special validation:

1. **Integration Point**: Cross-cutting insights are added to the issue pool AFTER synthesis completes, BEFORE validation begins
2. **Category Assignment**: Each insight specifies a `category` field - use this for validator model assignment
3. **Validator Model**: Cross-cutting insights always use **Opus** for validation
4. **Related Findings Check**: Validator must verify that BOTH `related_findings` references exist in the original findings

**Cross-Cutting Validation Checks:**
1. Verify both related findings are real issues (not false positives)
2. Verify the insight describes a genuine interaction between the two
3. Verify the insight is NOT a duplicate of either original finding
4. Verify the insight adds value beyond what each category found independently

**Cross-Cutting Deduplication**: If a cross-cutting insight duplicates an issue from the original category findings:
- Mark as INVALID with reason: "Duplicates existing finding from [category]"
- The original finding takes precedence

### Batch Validator Prompt

**Skip validation for:** Issues in test files (unless explicitly testing production code paths), style suggestions without functional impact, and documentation suggestions.

For each file with issues, launch ONE validator. Prompt schema:

| Field | Type | Notes |
|-------|------|-------|
| `file_path` | string | File being validated |
| `issues_to_validate[].id` | integer | Sequential issue ID |
| `issues_to_validate[].title` | string | Issue title |
| `issues_to_validate[].line` | integer | Line number |
| `issues_to_validate[].category` | string | Issue category |
| `issues_to_validate[].severity` | string | Issue severity |
| `issues_to_validate[].description` | string | Issue description |

**Required return format:**

```yaml
validations:
  - id: 1
    verdict: VALID|INVALID|DOWNGRADE
    new_severity: # only if DOWNGRADE
    reason: "brief explanation"
```

### Auto-Validation (Skip Validation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Domain-specific patterns:** See Auto-Validation Patterns (Documentation) section below.

**Notes on pattern matching:**
- Patterns are case-insensitive for SQL keywords
- Patterns require at least one character (`[^'"]+`) to avoid matching empty string placeholders like `password = ""`
- Patterns check for common variable names indicating user input: `req`, `request`, `params`, `query`, `body`, `input`, `user`
- Empty catch pattern allows a single comment line to avoid flagging intentional empty catches with explanation

### Auto-Validation Output

```yaml
issues:
  - title: "Hardcoded database password"
    auto_validated: true
    confidence_pattern: "hardcoded_credential"
    ...
```

### Common False Positives to Check

Validators should check for these common false positive patterns:

1. **Pre-existing issues**: The issue existed before the changes being reviewed
2. **Context makes it correct**: The code appears wrong but has valid context
3. **Handled elsewhere**: The issue is addressed in another part of the codebase
4. **Explicit ignore comments**: The issue is intentionally silenced (lint-ignore, etc.)
5. **Theoretical only**: The issue requires unrealistic conditions to manifest
6. **Test/dev code**: The issue is in test or development-only code paths
7. **Internal code**: The issue is in internal code with no external exposure

### Validation Output

Each validator returns:
- **VALID**: Issue is confirmed, keep original severity
- **INVALID**: Issue is a false positive, remove from results
- **DOWNGRADE**: Issue is real but severity should be lower

## Aggregation Rules

After validation, perform these aggregation steps:

### 1. Remove Invalid Issues

Filter out all issues marked INVALID by their validator.

### 2. Apply Severity Downgrades

For issues marked DOWNGRADE:
- Update the severity to the new level specified by the validator
- Keep the original description and details

### 3. Deduplicate Issues

When multiple agents flag the same issue:
- Merge issues for the same file and overlapping line ranges
- Keep the most detailed description
- Use the highest severity among the merged issues
- Add a consensus badge indicating multiple agents flagged it

### 4. Consensus Detection Algorithm

1. **Group by file** and detect overlapping issues (same file, intersecting line ranges: NOT (L2 < M1 OR M2 < L1))
2. **Build clusters** transitively (if A overlaps B and B overlaps C, cluster = {A, B, C})
3. **Merge each cluster**: use highest severity, longest description, combined categories, and add badge (`[2 agents]` or `[3+ agents]`) prepended to title

## Auto-Validation Patterns (Documentation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Accuracy patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `api_signature_mismatch` | N/A (detected via code comparison) | Documented function signature differs from implementation |
| `missing_parameter_doc` | N/A (detected via param comparison) | Parameter exists in code but not documented |
| `outdated_version_reference` | N/A (detected via version comparison) | Documentation references older version than package.json/csproj |

**Clarity patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `undefined_acronym` | `\b[A-Z]{2,}\b(?!.*\([^)]+\))` | Acronym used without expansion (first occurrence) |
| `passive_voice_instruction` | N/A (detected via NLP analysis) | Instructions written in passive voice |

**Completeness patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_api_doc` | N/A (detected via export comparison) | Exported function/class has no documentation |
| `empty_section` | `^#+\s+.+\n\s*\n(?=^#+)` | Section header with no content before next header |
| `missing_error_handling_doc` | N/A (detected via throw analysis) | Function throws but no error documentation |

**Consistency patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `inconsistent_terminology` | N/A (detected via term frequency analysis) | Same concept named differently across docs |
| `inconsistent_heading_style` | N/A (detected via heading pattern analysis) | Mixed title case and sentence case headings |

**Examples patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `example_syntax_error` | N/A (detected via parser) | Code example has syntax error |
| `example_undefined_import` | N/A (detected via import analysis) | Example uses undefined import |

**Structure patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `broken_internal_link` | `\[([^\]]+)\]\((?!https?://)([^)]+)\)` | Internal markdown link (check target exists) |
| `missing_ai_instruction_header` | N/A (detected via header check) | CLAUDE.md missing required header comment |
| `ai_instruction_wrong_location` | N/A (detected via path check) | AI instruction file in wrong directory |

## Category-Specific False Positive Rules (Documentation)

Each category has specific exclusions in addition to the general false positive rules in `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md`.

### Accuracy

- Intentionally simplified examples (marked as "simplified" or "basic example")
- Pseudocode clearly marked as illustrative
- Version-specific documentation with version clearly noted
- Documentation for planned/upcoming features marked as such

### Clarity

- Jargon appropriate for stated expert audience
- Acronyms defined earlier in the same document
- Industry-standard terms in domain-specific docs (e.g., "REST" in API docs)
- Intentionally terse reference documentation (vs tutorials)
- Code comments within code blocks (different standards apply)

### Completeness

- Internal/private APIs not intended for external use
- Features clearly marked as experimental/unstable
- Configuration options with sensible defaults that rarely need changing
- Platform-specific docs when project only targets one platform
- Sections that would duplicate content available elsewhere (with link)

### Consistency

- Intentional variations for emphasis or clarity
- Code/API names that must match implementation (even if inconsistent with prose style)
- Quoted text that preserves original formatting
- Version-specific sections that intentionally differ
- External content (quotes, references) with different style

### Examples

- Pseudocode clearly marked as illustrative (not runnable)
- Intentionally simplified examples with explicit notes about what's omitted
- Partial examples with "..." indicating omitted code
- Examples for older versions in clearly versioned documentation
- Examples showing error cases (intentionally incorrect code to demonstrate what not to do)
- Shell examples with placeholder values like `<your-token>`

### Structure

- Intentionally orphaned archive/historical documents
- External links to known-stable resources (official docs, RFCs)
- Heading hierarchy violations in code-generated documentation
- Alternative navigation paths that are intentional (multiple entry points)
- AI instruction files in projects that don't use AI assistants (if explicitly stated)

---

# Output Format

This section defines the output format for documentation review results and serves as the **authoritative output schema reference** for all agents.

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md` - Complete output example (reference)

## Output Generation Process

### 0. Generate Review Output

1. **Header**: Use the review depth description provided by the command
2. **Summary Table**: Include only the categories that were reviewed
3. **Issues**: Group by severity (Critical, Major, Minor, Suggestions)
4. **Cross-Cutting Insights**: Include if synthesis agents produced insights
5. **Test Recommendations**: Include if applicable

### 1. Display Output

Display the formatted review in the terminal.

### 2. Write to File

Write the same content to a file using the generated filename (see [Filename Generation](#filename-generation)).

If `--output-file <path>` argument was provided, use that path instead.

### 3. Confirm Output

At the end, print: "Review saved to: [filepath]"

---

## Filename Generation

Generate output filenames following this pattern:
`yyyy-mm-dd_few-words-summary_review-type.md`

### Components

- `yyyy-mm-dd`: Current date (e.g., `2026-02-05`)
- `few-words-summary`: 2-4 words describing the review scope, derived from:
  - File names being reviewed (for file-based commands)
  - "staged-changes" (for staged commands)
  - Main feature/component name when identifiable
- `review-type`: The command type (`deep-docs-review`, `quick-docs-review`, etc.)

### Rules

- Use lowercase with hyphens. Keep summary to 2-4 words.
- Truncate to keep total filename under 80 characters.
- For staged commands, always use "staged-changes" as the summary.

---

## Review Header

All reviews start with this header:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]
```

## No Issues Found

When no issues are found, use the standard header followed by "No issues found. All checks passed:" with only the categories that were actually checked, then "Files reviewed:" with the file list.

## Issues Found

When issues are found, output the standard header followed by:

1. **Summary table**: Categories as rows, severity levels as columns (`Critical | Major | Minor | Suggestions`), with a `**Total**` row. Include only reviewed categories (9 for deep, 4 for quick).
2. **Severity-grouped sections**: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only include sections with issues.
3. **Cross-Cutting Insights**: After severity sections, before Test Recommendations (see below).
4. **Test Recommendations**: If applicable.
5. Footer: `Review saved to: [filepath]`

## Cross-Cutting Insights Section

After the regular severity-grouped issues, include cross-cutting insights from the synthesis phase. Format each insight as:

**[N]. [Insight title]** `[Severity]` `[Primary Category] + [Secondary Category]`
`path/to/file.ts:line`
[Description of the cross-cutting concern. Reference which findings from each category are related.]
[Fix suggestion using diff or prompt format]

**When to include:** Only if synthesis agents produced `cross_cutting_insights`. Appears AFTER severity-grouped issues, BEFORE Test Recommendations. If none, omit section entirely.

## Issue Entry Format

Each issue entry follows this format:

```markdown
**[N]. [Issue title]** `[Severity]` `[Category]` [Consensus badge if applicable]
`path/to/file.ts:line-range`

[Description of the issue - what it is and why it matters]

[For small fixes - suggestion block]
[For larger fixes - prompt block]
```

### Severity Badge

Use inline code format: `` `Critical` ``, `` `Major` ``, `` `Minor` ``, `` `Suggestion` ``

### Consensus Badge

When multiple agents flag the same issue:
- `[2 agents]` - Flagged by 2 agents
- `[3+ agents]` - Flagged by 3 or more agents

### File Location

Use inline code format with path and line range: `` `src/utils/helper.ts:42-48` ``

## Actionable Fix Formats

Fixes should be actionable - users can apply them directly without interpretation. Choose the format based on fix complexity.

**Only report ONE entry per unique issue. Do not duplicate issues.**

### Fix Type Classification

| Fix Type | When to Use | Format |
|----------|-------------|--------|
| `diff` | Single location, ≤10 lines, exact replacement known | Inline diff block |
| `prompt` | Multi-location, structural changes, or requires context decisions | Claude Code prompt |

### Inline Diffs (fix_type: diff)

For simple, single-location fixes. Must be complete drop-in replacements (no "..." or partial code), ≤10 lines, requiring no changes elsewhere.

````markdown
**Fix**:
```diff
- const user = db.query(`SELECT * FROM users WHERE id = ${id}`);
+ const user = db.query('SELECT * FROM users WHERE id = ?', [id]);
```
````

### Fix Prompts (fix_type: prompt)

For complex fixes requiring multiple locations, structural changes, or context decisions. Use blockquote format (`>`) for easy copying. Start with action verb, list specific files and changes.

````markdown
**Fix prompt** (copy to Claude Code):
> Refactor UserService to separate authentication logic into AuthService:
> 1. Create src/services/AuthService.ts
> 2. Move login(), logout(), validateToken() from UserService
> 3. Update imports in src/api/auth.ts and src/middleware/auth.ts
````

## Complete Output Example

For a complete example showing all format elements together, see:
`${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md`
