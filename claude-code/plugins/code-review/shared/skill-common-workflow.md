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

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md` for:
- Project type detection rules
- AI Agent Instructions file patterns
- Test file patterns per language

**Quick Reference:**

| Detection | File/Pattern |
|-----------|-------------|
| Node.js | `package.json` |
| .NET | `*.csproj`, `*.sln` |
| Test files | See `languages/*.md` |

---

## Step 3: Launch Review Agent

**Goal:** Execute the review via a specialized agent.

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for:
- Complete Task invocation format
- Model selection per agent
- Subagent type mapping

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for:
- Two-pass pattern (thorough + gaps)
- Model selection table

---

## Step 4: Validate Findings

**Goal:** Filter false positives and confirm real issues.

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for:
- Auto-validated patterns
- Batch validation process
- Verdict options (VALID, INVALID, DOWNGRADE)

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
- **`${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`** - Batch validation, auto-validation patterns, and false positive handling
- **`references/skill-troubleshooting.md`** - Common issues and solutions

### Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` - Full orchestration logic
- `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` - Detailed validation process
- `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Complete output templates
- `${CLAUDE_PLUGIN_ROOT}/shared/output-schema-base.md` - YAML schema for findings
- `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` - Severity classification
