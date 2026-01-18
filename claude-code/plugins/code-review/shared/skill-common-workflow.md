# Common Skill Workflow Steps

This document contains reusable workflow steps shared across all review skills. Skills reference these procedures instead of duplicating them.

## Step 1: Determine Scope

**Goal:** Establish exactly what code needs review.

**Note:** This workflow is executed from the main conversation (not via subagent). Use AskUserQuestion to clarify the review scope when the user's request is ambiguous.

Present these options:

| Option | Description | When to Use |
|--------|-------------|-------------|
| **Staged changes** | Review `git diff --cached` | Pre-commit review |
| **Specific files** | User provides file paths | Targeted review |
| **Directory** | All files in a directory | Module-level audit |

### Decision Points

**Staged changes:**
1. Run `git diff --cached --name-only` to get file list
2. If no staged changes exist, inform user and offer alternatives
3. Get full diff with `git diff --cached`

**Specific files:**
1. Verify each file exists using Glob
2. If file not found, ask for correction
3. Read each file's content

**Directory:**
1. Use Glob to find all code files (`**/*.ts`, `**/*.js`, `**/*.cs`, etc.)
2. Warn if more than 20 files (suggest narrowing scope)
3. Prioritize files relevant to the review type

### Edge Cases

- **Empty file list:** "No files found to review. Please check the path or stage your changes."
- **Binary files:** Skip and note in output
- **Very large files (>5000 lines):** Warn user, suggest focusing on specific functions

---

## Step 2: Gather Context

**Goal:** Collect all information needed for accurate analysis.

### Project Type Detection

Detect the project type for each file being reviewed:

| Project Type | Detection Method |
|--------------|------------------|
| Node.js/TypeScript | `package.json` exists in project root |
| .NET/C# | `*.csproj`, `*.sln`, or `*.slnx` exists |

### Find AI Agent Instructions

Search for instruction files that may contain project-specific rules:

1. `CLAUDE.md` in project root
2. `.github/copilot-instructions.md`
3. `.ai/AI-AGENT-INSTRUCTIONS.md`
4. Subdirectory `CLAUDE.md` files relevant to reviewed files

### Find Related Test Files

| Project Type | Test File Patterns |
|--------------|-------------------|
| Node.js | `*.test.ts`, `*.spec.ts`, `__tests__/*.ts` |
| .NET | `*Tests.cs`, `*Test.cs`, `*.Tests/*.cs` |

For each file being reviewed, search for corresponding test file:
- `src/services/OrderService.ts` -> `src/services/OrderService.test.ts`
- `src/Services/OrderService.cs` -> `tests/Services/OrderServiceTests.cs`

---

## Step 3: Launch Review Agent

**Goal:** Execute the review via a specialized agent.

### Agent Invocation Pattern

Use the Task tool to launch agents. Note: Plugin agents are NOT registered as subagent types. Use "Explore" as the subagent_type and reference the agent file via `${CLAUDE_PLUGIN_ROOT}` in the prompt:

```
Task(
  subagent_type: "Explore",
  model: "opus",  // or "sonnet" per agent config
  description: "[Agent name] review for [scope]",
  prompt: """
MODE: [thorough|gaps|quick]

project_type: [nodejs|dotnet]

files_to_review:
  - path: "src/file.ts"
    has_changes: [true|false]
    diff: |
      [diff content if has_changes is true]
    full_content: |
      [full file content]

ai_instructions:
  - source: "CLAUDE.md"
    content: |
      [relevant instructions]

[Additional context specific to agent type]

Follow ${CLAUDE_PLUGIN_ROOT}/agents/[agent-name].md instructions.
Return findings as YAML per shared/output-schema-base.md.
"""
)
```

### Two-Pass Pattern (Thorough + Gaps)

For comprehensive reviews, use a two-pass approach:

**Pass 1 - Thorough Mode:**
```yaml
MODE: thorough
# Comprehensive review of all issues
```

**Pass 2 - Gaps Mode (after Pass 1 completes):**
```yaml
MODE: gaps

prior_findings:
  - title: "Issue from Pass 1"
    file: "src/file.ts"
    line: 42
    severity: "Major"
    # Agent skips this issue and focuses on subtle issues

# Focus on edge cases, subtle issues, and gaps
```

---

## Step 4: Validate Findings

**Goal:** Filter false positives and confirm real issues.

### Auto-Validated Patterns

High-confidence patterns skip validation and are marked `auto_validated: true`. Each skill defines its own auto-validation patterns (e.g., hardcoded secrets, SQL injection concatenation).

### Batch Validation Process

For issues not matching auto-validation patterns:

1. Group all issues for the same file
2. Launch one validator per file (not per issue)
3. Validator receives file content + all issues + full context
4. Validator returns: VALID, INVALID, or DOWNGRADE for each

### Common False Positive Indicators

Instruct validators to check for:

- **Test code:** Issues in test files may be intentional
- **Comments/docs:** Code in documentation is not executable
- **Disabled code:** Commented-out code is not active
- **Placeholder values:** `"changeme"` or `"TODO"` are not real values
- **Framework guarantees:** Framework may provide protection elsewhere

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
| Minor | X |
| Suggestion | X |

### Critical Issues (Must Fix)

**1. [Issue Title]** `Critical` `[Category]`
`file/path.ts:line-range`

[Description of the issue and its impact]

```[language]
// Problematic code
```

**Fix**:
```diff
- old line
+ new line
```

---

[Continue for each issue...]

### Summary

- **X Critical issues**: [summary]
- **X Major issues**: [summary]
- **X Minor issues**: [summary]
```

### Fix Types

For each issue, provide one of:

**Inline diff** (`fix_type: diff`): For single-location fixes ≤10 lines
```diff
- vulnerable line
+ fixed line
```

**Claude Code prompt** (`fix_type: prompt`): For multi-location or structural fixes
> Description of fix with numbered steps:
> 1. First change
> 2. Second change
> 3. Update related code

### Severity Classification

| Severity | Criteria |
|----------|----------|
| **Critical** | Immediate fix required, high impact |
| **Major** | Should fix before merge, significant impact |
| **Minor** | Can merge, fix soon, low impact |
| **Suggestion** | Optional improvement |

---

## Troubleshooting

**Agent returns no issues but problems exist:**
- Verify file content is included in the prompt
- Check that diffs show the problematic lines
- Ensure project_type is correctly detected
- Try expanding scope to include related files

**Too many false positives:**
- Increase min_severity to "Major" in settings
- Exclude test files if not relevant
- Add context about intentional exceptions

**Agent times out:**
- Reduce scope to fewer files
- Split large files into focused reviews
- Use quick mode for initial scan

---

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` - Full orchestration logic
- `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` - Detailed validation process
- `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Complete output templates
- `${CLAUDE_PLUGIN_ROOT}/shared/output-schema-base.md` - YAML schema for findings
- `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` - Severity classification
