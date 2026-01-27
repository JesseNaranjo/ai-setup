# Code Review Workflow Orchestration

This document defines the orchestration logic for the code review workflow. Agent definitions, language configs, validation rules, and output formats are in separate files.

**Note:** Commands use `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` for step definitions (Steps 1-11). This file provides detailed orchestration logic referenced during Step 6 (Review Execution).

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` - Phase definitions and model selection table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` - How to invoke agents via Task tool
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` - Common agent instructions (MODE, false positives, gaps)
- `${CLAUDE_PLUGIN_ROOT}/agents/*.md` - Individual agent definitions with MODE parameter
- `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` - Node.js/TypeScript specific checks
- `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md` - .NET/C# specific checks
- `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` - Validation process and rules
- `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` - Output formatting templates
- `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` - Canonical severity definitions

## Severity Classification

See `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` for canonical severity definitions and examples.

## Agent Configuration

The plugin includes 9 review agents plus a synthesis agent. Each agent supports specific modes (thorough, gaps, quick) with mode-specific model selection.

**Agent Files**: See `agents/*.md` for individual agent definitions
**Model Selection**: See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`
**Common Instructions**: See `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md`

## Orchestration Sequence

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for authoritative execution sequences including:
- Deep review phase definitions (19 agent invocations)
- Quick review phase definitions (7 agent invocations)
- Model selection table per agent and mode

## Usage Tracking Protocol

See `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md` for the complete tracking schema and recording protocol including:
- Initialization before launching agents
- Recording each agent invocation (timestamps, task_id, status)
- Phase timing for parallel execution
- Timing anomaly detection thresholds

## Review Configurations

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for:
- Deep review phase definitions and model selection
- Quick review phase definitions and model selection
- Synthesis phase category pairs

## Settings Application

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md` for complete setting definitions, defaults, and implementation details.

**Quick Reference:**

| Setting | Applied In | Purpose |
|---------|-----------|---------|
| `skip_agents` | Step 4 | Filter agents from review |
| `min_severity` | Step 6 | Filter output severity |
| `custom_rules` | Step 4 | Add to compliance agent |
| `additional_test_patterns` | Step 2 | Extend test file discovery |
| Project instructions (markdown body) | Step 4 | Add context to all agents |

## Skill-Informed Orchestration

When `--skills` is provided, load skill orchestration details from:
`${CLAUDE_PLUGIN_ROOT}/shared/skill-orchestration.md`

This file contains:
- Skill loading via Skill() tool (mandatory first, file read as fallback)
- Agent selection adjustments
- Agent prompt generation with skill_instructions
- Validation phase adjustments (auto-validate patterns, false positive rules)
- Synthesis phase adjustments
- skill_instructions generation algorithm

---

## Orchestration Details

This section provides orchestration-specific details that supplement the workflow steps defined in `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

### Review Execution

Review execution varies by review type. See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for authoritative phase definitions including:
- Deep review phases (Phase 1: 9 agents thorough, Phase 2: 5 agents gaps)
- Quick review phases (4 agents quick)
- Model selection per agent and mode

For gaps mode behavior, see the inline gaps rules in each supporting agent file (bug-detection, compliance, performance, security, technical-debt).

### Agent Invocation Pattern

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for:
- Subagent type mapping
- Task invocation template with full prompt structure
- Common agent input fields

**Model Selection**: See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for authoritative model selection table.

### Pre-Existing Issue Detection (For Staged/Diff Reviews)

**CRITICAL**: When reviewing staged changes or diffs, agents must only flag issues in CHANGED lines.

**Context provided to agents:**
1. Diff content with line markers (lines starting with `+` are additions)
2. Surrounding unchanged lines (for understanding context only)
3. Full file content (for reference only, not for flagging issues)

**Rules for what to flag:**
- ✅ Issue is in a line starting with `+` in the diff (newly added code)
- ✅ Change INTRODUCES the issue (e.g., removes null check that protected existing code)
- ✅ Change WORSENS an existing issue (e.g., increases scope of vulnerability)

**Do NOT flag:**
- ❌ Issues in unchanged code (lines without `+` prefix)
- ❌ Pre-existing problems not made worse by the change
- ❌ Style issues in untouched code nearby
- ❌ Issues visible in "full file" context but not in the diff

**Example:**
```diff
  function getUser(id) {
+   const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);  // FLAG: SQL injection
    if (existingBuggyCode) {  // DO NOT FLAG: pre-existing, not in diff
      return null;
    }
+   return user;
  }
```

#### Automatic Cross-File Analysis

Agents automatically perform cross-file analysis when the reviewed code suggests cross-cutting concerns. This is triggered by:

- **Import/export statements** → Check for consumers, circular dependencies
- **Class/interface definitions** → Check implementations, inheritance chains
- **API contracts** → Check for breaking changes affecting consumers
- **Shared types/utilities** → Check all usages for consistency

When triggered, agents use Grep and Glob tools to discover related files, then Read to analyze relationships. This enables detection of:
- Unused exports
- Circular dependencies
- Broken call chains (signature changed, callers not updated)
- Interface/implementation mismatches

See `agents/architecture-agent.md` for detailed cross-file analysis triggers and process.

### Synthesis Phase: Cross-Agent Synthesis

For synthesis phase orchestration, category pairs, and invocation format:
- See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for category pair definitions
- See `${CLAUDE_PLUGIN_ROOT}/shared/synthesis-invocation-pattern.md` for invocation parameters and output format

Synthesis insights are added to the issue pool before validation.

## Language-Specific Focus

Load language configs ONLY for detected languages to minimize context usage:

- If `detected_languages.nodejs` has files: Load `languages/nodejs.md`
- If `detected_languages.dotnet` has files: Load `languages/dotnet.md`
- Skip loading configs for languages not present in the review

For mixed codebases (monorepos):
- Each file receives only its relevant language config
- Agents receive language-specific checks per file, not all configs
- Cross-language issues (e.g., API contract mismatches) are handled by architecture and API agents

## False Positive Guidelines

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section for the complete list of issues that should NOT be flagged.

## Notes

- Use git CLI for repository interactions (not GitHub CLI)
- Always cite file paths and line numbers for each issue
- Quote exact AI Agent Instructions rules when citing violations
- File paths should be relative to repository root
- Line numbers reference lines in working copy (not diff line numbers)
- Maintain a todo list to track review progress
- Each issue should appear only once in the output (deduplicate)
