# Agent Invocation Pattern

This document defines how to invoke review agents via the Task tool.

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` - Phase definitions and model selection table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` - Common agent instructions (MODE, false positives, gaps, pre-existing issue detection)
- `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Authoritative output schema reference
- `${CLAUDE_PLUGIN_ROOT}/agents/code/*.md` - Code review agent definitions (including synthesis-code-agent)
- `${CLAUDE_PLUGIN_ROOT}/agents/docs/*.md` - Documentation review agent definitions (including synthesis-docs-agent)

## Subagent Types

Plugin agents are registered as subagent types with the pattern `code-review:[agent-name]`:

| Agent | Subagent Type |
|-------|---------------|
| API Contracts | `code-review:api-contracts-agent` |
| Architecture | `code-review:architecture-agent` |
| Bug Detection | `code-review:bug-detection-agent` |
| Compliance | `code-review:compliance-agent` |
| Error Handling | `code-review:error-handling-agent` |
| Performance | `code-review:performance-agent` |
| Security | `code-review:security-agent` |
| Synthesis (Code) | `code-review:synthesis-code-agent` |
| Synthesis (Docs) | `code-review:synthesis-docs-agent` |
| Technical Debt | `code-review:technical-debt-agent` |
| Test Coverage | `code-review:test-coverage-agent` |

## Invocation Template

Use the Task tool to launch each agent. Always pass the `model` parameter explicitly (see `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`).

```
Task(
  subagent_type: "code-review:<agent-name>",
  model: "<model>",  // See orchestration-sequence.md Code Review Model Selection table
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

## Model Selection

**Important**: Always pass the `model` parameter explicitly when invoking Task. Refer to `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for the correct model per agent and mode.

**Rationale**: Gaps mode uses Sonnet (not Opus) because it receives prior findings context and follows explicit checklists, reducing the complexity of the task. This is a cost optimization, not a quality tradeoff.

## Content Distribution Optimization

To reduce Execution Context usage, not all agents receive all content. The orchestrator should distribute content selectively based on agent requirements.

### Test File Distribution

Pass `related_tests` content ONLY to agents that analyze test relationships:

| Agent | Receives `related_tests` | Rationale |
|-------|--------------------------|-----------|
| bug-detection-agent | Yes | Uses test files to understand expected behavior |
| technical-debt-agent | Yes | Identifies untested deprecated code |
| test-coverage-agent | Yes | Primary consumer - analyzes test coverage |
| All other agents | No | Can use Grep/Glob if cross-file analysis warrants |

**Estimated savings:** 6 agents x ~300 lines average test content = ~1,800 lines per review

### AI Instructions Distribution

Pass full `ai_instructions` content ONLY to agents that need project-specific rules:

| Agent | Receives full `ai_instructions` | Rationale |
|-------|--------------------------------|-----------|
| architecture-agent | Yes | Checks documented architectural patterns and conventions |
| compliance-agent | Yes | Primary consumer - verifies adherence to documented standards |
| All other agents | Summary only | Receive: "AI instructions exist at [paths]. Use Grep to check specific rules if needed." |

**Estimated savings:** 7 agents x ~500 lines average AI instructions = ~3,500 lines per review

## Synthesis Invocation

This section defines the invocation pattern for the synthesis agents. The synthesis agents are designed to be invoked **multiple times in parallel** with different category pairs.

See `${CLAUDE_PLUGIN_ROOT}/agents/code/synthesis-code-agent.md` (code reviews) or `${CLAUDE_PLUGIN_ROOT}/agents/docs/synthesis-docs-agent.md` (docs reviews) for the full agent definition and analysis logic.

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

Commands launch 5 instances of the synthesis agent simultaneously, each with different category pairs.

**Authoritative source for category pairs:** See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` "Synthesis" sections for the definitive list of pairs and cross-cutting questions for both deep review (5 pairs) and quick review (3 pairs).

Each instance operates independently and returns its own `cross_cutting_insights` list. The orchestrating command merges all results.
