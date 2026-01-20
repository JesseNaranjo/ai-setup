# Common Skill Workflow Steps

Reusable workflow steps shared across all review skills. Skills reference these procedures instead of duplicating them.

## Step 1: Determine Scope

**Goal:** Establish exactly what code needs review.

**Note:** This workflow is executed from the main conversation (not via subagent). Use AskUserQuestion to clarify the review scope when the user's request is ambiguous.

| Option | Description | When to Use |
|--------|-------------|-------------|
| **Staged changes** | Review `git diff --cached` | Pre-commit review |
| **Specific files** | User provides file paths | Targeted review |
| **Directory** | All files in a directory | Module-level audit |

For detailed decision points and edge case handling, see `${CLAUDE_PLUGIN_ROOT}/shared/references/scope-determination.md`.

---

## Step 2: Gather Context

**Goal:** Collect all information needed for accurate analysis.

### Project Type Detection

| Project Type | Detection Method |
|--------------|------------------|
| Node.js/TypeScript | `package.json` exists in project root |
| .NET/C# | `*.csproj`, `*.sln`, or `*.slnx` exists |

### Find AI Agent Instructions

Search in priority order:

| Priority | Location |
|----------|----------|
| 1 | `CLAUDE.md` (project root) |
| 2 | `[directory]/CLAUDE.md` (subdirectory overrides) |
| 3 | `.github/copilot-instructions.md` |
| 4 | `.ai/AI-AGENT-INSTRUCTIONS.md` |
| 5 | `docs/AI-INSTRUCTIONS.md` |

### Find Related Test Files

| Project Type | Test File Patterns |
|--------------|-------------------|
| Node.js | `*.test.ts`, `*.spec.ts`, `__tests__/*.ts` |
| .NET | `*Tests.cs`, `*Test.cs`, `*.Tests/*.cs` |

For each file being reviewed, search for corresponding test file:
- `src/services/OrderService.ts` → `src/services/OrderService.test.ts`
- `src/Services/OrderService.cs` → `tests/Services/OrderServiceTests.cs`

---

## Step 3: Launch Review Agent

**Goal:** Execute the review via a specialized agent.

### Agent Invocation Pattern

See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` section "Agent Invocation Pattern" for:
- Complete Task invocation format with YAML prompt structure
- Model selection per agent and mode
- Two-pass pattern (thorough + gaps)

**Quick Reference - Subagent Types:**

| Agent | Subagent Type |
|-------|---------------|
| Security | `code-review:security-agent` |
| Bug Detection | `code-review:bug-detection-agent` |
| Performance | `code-review:performance-agent` |
| Compliance | `code-review:compliance-agent` |
| Architecture | `code-review:architecture-agent` |
| API Contracts | `code-review:api-contracts-agent` |
| Error Handling | `code-review:error-handling-agent` |
| Test Coverage | `code-review:test-coverage-agent` |
| Synthesis | `code-review:synthesis-agent` |

**Example invocation:**

```
Task(
  subagent_type: "code-review:security-agent",
  model: "opus",  // See review-workflow.md for model selection table
  description: "[Agent name] review for [scope]",
  prompt: """
MODE: [thorough|gaps|quick]
project_type: [nodejs|dotnet]

files_to_review:
  - path: "src/file.ts"
    has_changes: [true|false]
    diff: |
      [diff content if has_changes]
    full_content: |
      [full file content]

ai_instructions:
  - source: "CLAUDE.md"
    content: |
      [relevant instructions]

Return findings as YAML per shared/output-schema-base.md.
"""
)
```

**Important**: Always pass the `model` parameter explicitly. See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` for the model selection table (Opus vs Sonnet per agent and mode).

---

## Step 4: Validate Findings

**Goal:** Filter false positives and confirm real issues.

### Quick Reference

- **Auto-validated patterns**: High-confidence issues skip validation (defined per skill)
- **Batch validation**: Group issues by file, one validator per file
- **Verdict options**: VALID, INVALID, or DOWNGRADE

For detailed validation process, batch procedures, and false positive indicators, see `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-details.md`.

---

## Step 5: Report Results

**Goal:** Present findings in actionable, severity-prioritized format.

### Standard Output Structure

```markdown
## [Review Type] Review

**Scope:** [files or staged changes reviewed]
**Project Type:** [Node.js/TypeScript or .NET/C#]

### Findings

| Severity | Count |
|----------|-------|
| Critical | X |
| Major | X |

### Critical Issues (Must Fix)

**1. [Issue Title]** `Critical` `[Category]`
`file/path.ts:line-range`

[Description]

**Fix**:
```diff
- old line
+ new line
```
```

### Fix Types

| Type | When to Use |
|------|-------------|
| **diff** | Single-location fixes ≤10 lines |
| **prompt** | Multi-location or structural fixes |

### Severity Classification

| Severity | Criteria |
|----------|----------|
| **Critical** | Immediate fix required, high impact |
| **Major** | Should fix before merge |
| **Minor** | Can merge, fix soon |
| **Suggestion** | Optional improvement |

---

## Additional Resources

### Reference Files

For detailed procedures:
- **`references/scope-determination.md`** - Detailed scope options and edge cases
- **`references/validation-details.md`** - Batch validation and false positive handling
- **`references/skill-troubleshooting.md`** - Common issues and solutions

### Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` - Full orchestration logic
- `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` - Detailed validation process
- `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Complete output templates
- `${CLAUDE_PLUGIN_ROOT}/shared/output-schema-base.md` - YAML schema for findings
- `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` - Severity classification
