# Code Review Orchestration

This document defines agent invocation patterns and execution sequences for code review pipelines.

## Agent Invocation

Plugin agents are registered as subagent types with the pattern `code-review:<agent-name>`.

Use the Task tool to launch each agent. Always pass the `model` parameter explicitly (see Code Review Model Selection table below).

```
Task(
  subagent_type: "code-review:<agent-name>",
  model: "<model>",  // See Code Review Model Selection table
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
| `ai_instructions` | list | Conditional | Full content for architecture/compliance agents; summary-only for others (see AI Instructions Distribution) |
| `related_tests` | list | Conditional | Only for bug-detection, technical-debt, test-coverage agents (see Test File Distribution) |
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

### Content Distribution Optimization

To reduce Execution Context usage, not all agents receive all content. The orchestrator should distribute content selectively based on agent requirements.

#### Test File Distribution

Pass `related_tests` content ONLY to agents that analyze test relationships:

| Agent | Receives `related_tests` | Rationale |
|-------|--------------------------|-----------|
| bug-detection-agent | Yes | Uses test files to understand expected behavior |
| technical-debt-agent | Yes | Identifies untested deprecated code |
| test-coverage-agent | Yes | Primary consumer - analyzes test coverage |
| All other agents | No | Can use Grep/Glob if cross-file analysis warrants |

**Estimated savings:** 6 agents x ~300 lines average test content = ~1,800 lines per review

#### AI Instructions Distribution

Pass full `ai_instructions` content ONLY to agents that need project-specific rules:

| Agent | Receives full `ai_instructions` | Rationale |
|-------|--------------------------------|-----------|
| architecture-agent | Yes | Checks documented architectural patterns and conventions |
| compliance-agent | Yes | Primary consumer - verifies adherence to documented standards |
| All other agents | Summary only | Receive: "AI instructions exist at [paths]. Use Grep to check specific rules if needed." |

**Estimated savings:** 7 agents x ~500 lines average AI instructions = ~3,500 lines per review

## Deep Code Review Sequence (19 agent invocations)

1. **Steps 1-3: Input, Context, Content**
   - Validate input, discover context, gather file content
   - OUTPUT: Files to review, diffs, AI instructions, test files

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
   - **CRITICAL: WAIT** - DO NOT proceed to Synthesis until ALL 5 agents complete
   - OUTPUT: Phase 2 findings (subtle issues, edge cases)

4. **Synthesis** (5 agents in parallel)
   - **CRITICAL: DO NOT START until Phase 1 AND Phase 2 are FULLY COMPLETE**
   - Launch: 5 instances of synthesis-code-agent with category pairs
   - INPUT: ALL findings from Phase 1 AND Phase 2
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

| Agent | Model (thorough) | Model (gaps) | Model (quick) |
|-------|------------------|--------------|---------------|
| api-contracts-agent | sonnet | N/A | N/A |
| architecture-agent | opus | N/A | N/A |
| bug-detection-agent | opus | sonnet | opus |
| compliance-agent | sonnet | sonnet | N/A |
| error-handling-agent | sonnet | N/A | sonnet |
| performance-agent | opus | sonnet | N/A |
| security-agent | opus | sonnet | opus |
| synthesis-code-agent | sonnet | N/A | sonnet |
| technical-debt-agent | opus | sonnet | N/A |
| test-coverage-agent | sonnet | N/A | sonnet |

## Content Strategy by Phase

Different phases have different content requirements. This strategy reduces token usage while maintaining review quality.

**Phase 1 (Thorough):**
- Full file content provided
- Full diff content provided
- AI instructions per distribution rules (see Content Distribution Optimization above)
- Test files per distribution rules (see Content Distribution Optimization above)
- Agents perform comprehensive analysis

**Phase 2 (Gaps):**
- Diff content always provided
- Full file content: Not provided by default - agents use Read tool if deeper analysis needed
- Previous findings provided (defines skip zones)
- Focus: Subtle issues, edge cases not caught in Phase 1
- Model: Sonnet (cost-optimized for constrained task with prior context)

**Synthesis:**
- Findings from Phase 1 + Phase 2 provided
- File paths provided (agents can Read if cross-reference needed)
- Focus: Cross-cutting concerns only
- No file content passed by default

**Rationale:** Agents have access to Read, Grep, and Glob tools. Providing file paths instead of full content for later phases allows agents to fetch content on-demand, reducing baseline token usage while preserving capability.

## Language-Specific Focus

Load language configs ONLY for detected languages/frameworks to minimize context usage:

- If `detected_languages.dotnet` has files: Load `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`
- If `detected_languages.nodejs` has files: Load `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md`
- If `detected_frameworks.react` has files: Also load `${CLAUDE_PLUGIN_ROOT}/languages/react.md`
- Skip loading configs for languages/frameworks not present in the review

For mixed codebases (monorepos):
- Each file receives only its relevant language config
- React files receive both nodejs.md AND react.md checks
- Agents receive language-specific checks per file, not all configs
- Cross-language issues (e.g., API contract mismatches) are handled by architecture and API agents

## Synthesis Invocation

The synthesis agents are designed to be invoked **multiple times in parallel** with different category pairs.

See `${CLAUDE_PLUGIN_ROOT}/agents/code/synthesis-code-agent.md` for the full agent definition and analysis logic.

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

**Authoritative source for category pairs:** See Deep Code Review Sequence (5 pairs) and Quick Code Review Sequence (3 pairs) above.

---

# Code Review Validation Rules

This section defines the validation process and domain-specific patterns for code review commands.

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
| API Contracts | Sonnet |
| Architecture | Opus |
| Bugs | Opus |
| Clarity | Sonnet |
| Completeness | Opus |
| Compliance | Sonnet |
| Consistency | Sonnet |
| Error Handling | Sonnet |
| Examples | Opus |
| Performance | Opus |
| Security | Opus |
| Structure | Sonnet |
| Technical Debt | Sonnet |
| Test Coverage | Sonnet |

**Cross-cutting insights** (from synthesis-code-agent or synthesis-docs-agent) always use **Opus** for validation.

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

**Domain-specific patterns:** See Auto-Validation Patterns (Code) section below.

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

## Auto-Validation Patterns (Code)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Architecture patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `circular_dependency` | N/A (detected via import graph analysis) | Circular import/dependency between modules |
| `god_class_500_lines` | N/A (detected via line count) | Class/module exceeds 500 lines |
| `function_10_params` | `function\s+\w+\s*\([^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*,\s*[^)]*\)` | Function has 10+ parameters |
| `direct_new_instantiation` | `(?:=\s*new\s+\w+Service\|=\s*new\s+\w+Repository)\s*\(` | Direct instantiation of service/repository dependency |

**Bug patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `empty_catch_block` | `catch\s*\([^)]*\)\s*\{\s*(?:\/\/[^\n]*)?\s*\}` | Empty catch block (allows single comment) |
| `missing_await` | `(?:const\|let\|var)\s+\w+\s*=\s*(?!await)[^;]*\basync\s+\w+\(` | Async function call without await |
| `null_dereference` | `(?:\?\.\s*\w+\s*\(\)\s*\.)\|(?:\w+\s*&&\s*\w+\.\w+\s*\?\s*\w+\.\w+\.\w+)` | Null access after optional chain or guard |

**Compliance patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `missing_authorize_attribute` | `\[(?:Http(?:Get\|Post\|Put\|Delete\|Patch)\|Route)\][^[]*(?<!\[Authorize\])\s*public\s+(?:async\s+)?(?:Task<)?(?:IActionResult\|ActionResult)` | ASP.NET endpoint without [Authorize] attribute |
| `wrong_case_filename` | N/A (detected via filesystem comparison) | Filename violates project naming convention |
| `explicit_must_violation` | N/A (detected via rule matching) | Code violates a MUST rule from AI instructions |
| `missing_required_jsdoc` | `export\s+(?:async\s+)?function\s+\w+\s*\([^)]*\)\s*(?::\s*\w+)?\s*\{(?!\s*/\*\*)` | Exported function without JSDoc comment |

**Performance patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `nested_loop_includes` | `for\s*\([^)]*\)[\s\S]*?for\s*\([^)]*\)[\s\S]*?\.(?:includes\|indexOf)\s*\(` | O(n²) nested loop with includes/indexOf |
| `select_star_in_loop` | `(?:for\|while\|forEach)[\s\S]*?SELECT\s+\*` | SELECT * inside a loop |
| `n_plus_one_query` | `(?:for\|while\|forEach\|\.map\|\.forEach)[\s\S]{0,200}(?:findOne\|findById\|query\|execute\|fetch)` | Database query inside iteration (N+1) |

**Security patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `hardcoded_password` | `(?:password\|passwd\|pwd)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded password in assignment |
| `hardcoded_api_key` | `(?:api[_-]?key\|apikey)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded API key |
| `hardcoded_token` | `(?:token\|bearer\|auth[_-]?token\|access[_-]?token)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded auth token |
| `hardcoded_secret` | `(?:secret\|client[_-]?secret\|private[_-]?key)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded secret/private key |
| `hardcoded_credentials` | `(?:credentials\|connection[_-]?string)\s*[:=]\s*['"][^'"]+['"]` | Hardcoded credentials object |
| `sql_injection_concat` | `(?:SELECT\|INSERT\|UPDATE\|DELETE\|FROM\|WHERE).*[+]\s*(?:req\|request\|params\|query\|body\|input\|user)` | SQL with string concatenation of user input |
| `sql_injection_template` | `(?:SELECT\|INSERT\|UPDATE\|DELETE\|FROM\|WHERE).*\$\{.*(?:req\|request\|params\|query\|body\|input\|user)` | SQL with template literal interpolation of user input |
| `eval_untrusted` | `eval\s*\(\s*(?:req\|request\|params\|query\|body\|input\|user)` | eval() with untrusted input |
| `new_function_untrusted` | `new\s+Function\s*\(\s*(?:req\|request\|params\|query\|body\|input\|user)` | new Function() with untrusted input |

**Technical Debt patterns (always valid):**

| Pattern Name | Regex | Description |
|-------------|-------|-------------|
| `deprecated_npm_package` | N/A (detected via `npm ls` output) | npm package with deprecation warning |
| `todo_without_issue` | `(?:\/\/\|#)\s*TODO(?:\([^)]*\))?:?\s+(?!.*(?:#\d+\|ISSUE-\|JIRA-\|TICKET-))` | TODO/FIXME without issue reference |
| `commented_out_code` | `^(?:\s*(?:\/\/\|#).*\n){10,}` | 10+ consecutive lines of commented code |
| `hack_comment` | `(?:\/\/\|#\|\/\*)\s*(?:HACK\|WORKAROUND\|XXX)\s*[:\-]?` | Explicit HACK/WORKAROUND/XXX marker |
| `outdated_callback` | `function\s+\w+\s*\([^)]*,\s*(?:callback\|cb\|done)\s*\)` | Callback pattern in async context |

## Category-Specific False Positive Rules (Code)

Each category has specific exclusions in addition to the general false positive rules in `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md`.

### API Contracts

- Internal/private API changes
- Changes to APIs with no external consumers
- Additive changes that don't break existing consumers
- Changes that follow established deprecation process
- Beta/experimental APIs clearly marked as unstable

### Architecture

- Pragmatic compromises with clear justification
- Patterns that are overkill for the scale of the project
- Architecture decisions already documented and justified
- Temporary code with clear TODOs

### Bug Detection

- Code that appears buggy but is correct in context
- Defensive code that handles edge cases (unless it has a bug)
- Code with explicit comments explaining why it's correct

### Compliance

- Code that appears to violate a rule but has an explicit override comment
- Ambiguous rules where the code could reasonably be compliant
- Rules that don't apply to this file type or context
- Style preferences not explicitly stated as rules

### Error Handling

- Code where errors are intentionally ignored with explicit comments
- Errors that are handled at a higher level
- Internal code with documented error handling strategy
- Logging-only catch blocks where that's the intended behavior

### Performance

- Micro-optimizations that won't have measurable impact
- Performance issues in code that runs rarely
- Code that prioritizes readability over minor performance gains

### Security

- Internal-only code with no untrusted input exposure
- Code with explicit security comments explaining the design
- Vulnerabilities already mitigated elsewhere in the code

### Technical Debt

- Dependencies intentionally pinned for compatibility (documented reason)
- Legacy patterns in legacy modules explicitly marked as deprecated
- Dead code that's actually conditionally compiled (build flags)
- TODO comments that reference issue tracking (TODO(#123))
- Workarounds with documented upstream bugs and tracking
- Class components in projects supporting older React versions intentionally

### Test Coverage

- Private/internal implementation details
- Code that's impractical to unit test (better suited for integration tests)
- Code already covered by higher-level tests
- Test files themselves (don't require tests of tests)
- Generated code or boilerplate
- Configuration files or constants
- Dead code that should be removed rather than tested

---

# Output Format

This section defines the output format for code review results and serves as the **authoritative output schema reference** for all agents.

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
- `review-type`: The command type (`deep-code-review`, `quick-code-review`, etc.)

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
