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

## Skill Instructions Agent Guidance

When `--skills` is active, inject the following into each agent's `additional_instructions`:

When `skill_instructions` is present in the prompt, apply it as follows:

1. **focus_areas**: Prioritize checking these categories FIRST before standard checks. Structure findings around these areas where applicable.
2. **checklist**: For each checklist category, explicitly verify EVERY item. If an item is clean (no issues found), acknowledge it was checked.
3. **auto_validate**: Issues matching these pattern IDs should include `auto_validated: true` in output.
4. **false_positive_rules**: Apply these as ADDITIONAL false positive filters beyond this agent's standard rules.

For methodology skills (like `superpowers:brainstorming`):
1. **methodology.approach**: Adopt this mindset throughout analysis
2. **methodology.steps**: Follow these steps as part of your review process
3. **methodology.questions**: Consider these questions when evaluating each potential finding

**Note:** Agents without a primary review skill (api-contracts-agent, error-handling-agent, synthesis-code-agent, synthesis-docs-agent, test-coverage-agent) receive only the methodology section.

When `skill_instructions` is absent, proceed with standard review process.

## Tiered Context Agent Guidance

For staged reviews, inject the following into each agent's `additional_instructions`:

When files include tier information (staged reviews):

**For `tier: "critical"` files:**
- Full content is provided - analyze thoroughly
- This is the primary review focus

**For `tier: "peripheral"` files:**
- Only a preview (first 50 lines) is provided
- Use the preview to understand file purpose
- If cross-file analysis discovers relevance, use Read tool to get full content

---

# Code Review Validation Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-code.md` for validation rules, auto-validation patterns, and category-specific false positive rules. Load at Steps 9-12 only.
