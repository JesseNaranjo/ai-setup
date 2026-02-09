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

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` for the common validation process (grouping, cross-cutting insight validation, batch validator prompt, false positives, verdicts).

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

### Auto-Validation (Skip Validation)

Some high-confidence patterns skip validation entirely and are marked `auto_validated: true`:

**Domain-specific patterns:** See Auto-Validation Patterns (Code) section below.

## Aggregation Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` for aggregation rules (remove invalid, apply downgrades, deduplicate, consensus detection).

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

See `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for the complete output format specification.
