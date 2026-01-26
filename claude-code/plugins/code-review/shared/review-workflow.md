# Code Review Workflow Orchestration

This document defines the orchestration logic for the code review workflow. Agent definitions, language configs, validation rules, and output formats are in separate files.

## Related Files

- `shared/orchestration-sequence.md` - Phase definitions and model selection table
- `shared/agent-invocation-pattern.md` - How to invoke agents via Task tool
- `shared/agent-common-instructions.md` - Common agent instructions (MODE, false positives, gaps)
- `agents/*.md` - Individual agent definitions with MODE parameter
- `languages/nodejs.md` - Node.js/TypeScript specific checks
- `languages/dotnet.md` - .NET/C# specific checks
- `shared/validation-rules.md` - Validation process and rules
- `shared/output-format.md` - Output formatting templates
- `shared/severity-definitions.md` - Canonical severity definitions

## Severity Classification

See `shared/severity-definitions.md` for canonical severity definitions and examples.

## Agent Configuration

The plugin includes 8 review agents plus a synthesis agent. Each agent supports specific modes (thorough, gaps, quick) with mode-specific model selection.

**Agent Files**: See `agents/*.md` for individual agent definitions
**Model Selection**: See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`
**Common Instructions**: See `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md`

## Orchestration Sequence

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for authoritative execution sequences including:
- Deep review phase definitions (16 agent invocations)
- Quick review phase definitions (7 agent invocations)
- Model selection table per agent and mode

## Usage Tracking Protocol

Throughout the review workflow, maintain usage tracking to record agent invocations and timing. See `shared/usage-tracking.md` for the complete schema.

### Initialization (Before Step 4)

Before launching the first agent:

1. Record `review_started_at` timestamp for the review
2. Initialize the phases array based on review type:
   - **Deep review**: 3 phases (Phase 1: 8 agents, Phase 2: 4 agents, Synthesis: 4 agents)
   - **Quick review**: 2 phases (Review: 4 agents, Synthesis: 3 agents)
3. Initialize each phase with an empty agents array

### Recording Each Agent Invocation

For every agent launch:

1. **Before Task tool call**: Record `agent_started_at` timestamp
2. **After Task tool returns**:
   - Record `agent_ended_at` timestamp
   - Calculate `agent_duration_seconds` = agent_ended_at - agent_started_at
   - Record `task_id` from Task tool return (confirms actual invocation)
   - Set `status` to "completed" (or "failed" if error)
3. **For skipped agents** (via skip_agents setting):
   - Set `status` to "skipped"
   - Set `agent_duration_seconds` to 0
   - No timestamps or task_id

### Confirming Actual Invocation

The Task tool returns a `task_id` when an agent completes. Record this ID as proof of actual invocation:
- Present: Agent was actually invoked
- Absent with duration: Agent may have failed

### Parallel Execution Timing

When agents run in parallel within a phase:
- **phase_started_at**: Timestamp when first agent launched
- **phase_ended_at**: Timestamp when last agent completed
- **phase_duration_seconds**: phase_ended_at - phase_started_at
- Individual agent durations are independent
- Sum of individual durations may exceed phase duration (expected with parallelism)

### Timing Anomaly Detection

Flag potential issues based on timing:

| Condition | Indicator | Concern |
|-----------|-----------|---------|
| Opus < 15s | `[!]` too fast | May have errored early |
| Sonnet < 10s | `[!]` too fast | May have errored early |
| Synthesis < 5s | `[!]` too fast | May have errored early |
| Opus > 180s | `[*]` too slow | May be stuck |
| Sonnet > 120s | `[*]` too slow | May be stuck |
| Synthesis > 90s | `[*]` too slow | May be stuck |

### Generating Usage Summary

At the end of the workflow (before generating review output):

1. Read from the usage_tracking structure maintained during workflow
2. Format according to `shared/output-format.md` Usage Summary section
3. Calculate phase totals and detect timing anomalies
4. Include agent details in expandable section

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

## Workflow Steps

### Step 1: Input Validation

Verify:
- Current directory is a git repository
- Files/changes exist to review
- Parse command arguments

### Step 2: Context Discovery

1. Find all AI Agent Instructions files:
   - `CLAUDE.md` files (root and directories)
   - `.ai/AI-AGENT-INSTRUCTIONS.md` files
   - `.github/copilot-instructions.md` files

2. Detect project type and language per file:

   **Per-File Language Detection:**
   For each file being reviewed, detect its language:

   - **By extension**:
     - `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs` → Node.js/TypeScript
     - `.cs` → .NET/C#
     - `.py` → Python (future support)

   - **By nearest project file**:
     - Walk up directories to find `package.json` or `*.csproj`
     - Use that project's language config

   - **For monorepos/mixed codebases**:
     - Apply language-specific checks PER FILE
     - Group files by language when reporting to agents
     - Example context: "Node.js files: [a.ts, b.ts], .NET files: [c.cs]"

   **Language Override:**
   Commands accept `--language` parameter to force detection:
   - `--language nodejs` - Force Node.js/TypeScript checks for all files
   - `--language dotnet` - Force .NET/C# checks for all files

3. Find related test files based on detected project type(s)

### Step 3: Content Gathering

1. Get current branch name
2. For files with changes: get diff AND full file content
3. For files without changes: get full file content
4. Read related test files for context

### Step 4: Review Execution

Review execution varies by review type:

#### Deep Review: Two-Phase Sequential Approach

**Phase 1: Launch 8 agents with thorough mode in parallel**
- 3 Opus agents (bug-detection, security, performance) in thorough mode
- 5 Sonnet agents (compliance, architecture, api-contracts, error-handling, test-coverage) in thorough mode

Wait for all Phase 1 agents to complete and collect their findings.

**Phase 2: Launch 4 Sonnet agents with gaps mode in parallel**

Pass Phase 1 findings as context to gaps mode agents. Each agent receives findings from its own category:

```yaml
previous_findings:
  - title: "SQL injection in getUser"
    file: "src/db/users.ts"
    line: 23
    range: null  # null for single-line issues, "start-end" for multi-line
    category: "Security"
    severity: "Critical"
    # Gaps agent should skip this - already flagged
```

Gaps mode agents use this context to:
- Skip issues that match prior findings (same file + overlapping line ranges)
- Focus analysis on areas not yet covered
- Find subtle issues that complement thorough findings

#### Quick Review: Single-Phase Parallel

Launch 4 agents with quick mode in parallel. No gaps phase.

#### Agent Invocation Pattern

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for:
- Subagent type mapping
- Task invocation template with full prompt structure
- Common agent input fields

**Model Selection**: See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for authoritative model selection table.

#### Pre-Existing Issue Detection (For Staged/Diff Reviews)

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

After the review phase and before validation, launch **synthesis agents** that analyze findings across categories to identify cross-cutting concerns.

**Purpose**: Catch issues that span multiple review categories, such as:
- Security fixes that introduce performance regressions
- Architectural changes lacking test coverage
- Bug fixes that break API contracts or lack error handling

#### Deep Review Synthesis (4 agents in parallel)

| Agent | Input Categories | Cross-Cutting Question |
|-------|-----------------|------------------------|
| `agents/synthesis-agent.md` | Security + Performance | "Do any security fixes introduce performance issues?" |
| `agents/synthesis-agent.md` | Architecture + Test Coverage | "Are architectural changes covered by tests?" |
| `agents/synthesis-agent.md` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| `agents/synthesis-agent.md` | Compliance + Bugs | "Do compliance violations introduce or mask bugs?" |

#### Quick Review Synthesis (3 agents in parallel)

| Agent | Input Categories | Cross-Cutting Question |
|-------|-----------------|------------------------|
| `agents/synthesis-agent.md` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| `agents/synthesis-agent.md` | Security + Bugs | "Do security issues introduce or relate to bugs?" |
| `agents/synthesis-agent.md` | Bugs + Test Coverage | "Are identified bugs covered by tests?" |

Each synthesis agent receives:
- All findings from both input categories
- The file diffs/content being reviewed
- Context about what each category found

**Synthesis Agent Output**:
```yaml
cross_cutting_insights:
  - title: "SQL injection fix uses unbounded query"
    related_findings:
      security: "SQL injection in getUser"
      performance: "Unbounded query in user lookup"
    # Both categories required - if only one has a finding, don't flag as cross-cutting
    insight: "The parameterized query fix doesn't limit result set size, potential DoS"
    category: "Performance"
    severity: "Major"
    file: "src/db/users.ts"
    line: 25
```

Synthesis insights are added to the issue pool before validation.

### Step 5: Validation

For each issue from the review phases, launch a validation subagent.

See `shared/validation-rules.md` for:
- Validator model assignment
- Validation prompt template
- Common false positive checks

### Step 6: Aggregation

1. Remove issues marked INVALID
2. Apply severity downgrades
3. Deduplicate by file+line range
4. Add consensus badges for multi-agent issues

See `shared/validation-rules.md` for full aggregation rules.

### Step 7: Generate Output

Format results according to `shared/output-format.md`.

### Step 8: Write Output

1. Display formatted review in terminal
2. Write to output file (default or specified via --output-file)
3. Print "Review saved to: [filepath]"

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

See `${CLAUDE_PLUGIN_ROOT}/shared/false-positives.md` for the complete list of issues that should NOT be flagged.

## Notes

- Use git CLI for repository interactions (not GitHub CLI)
- Always cite file paths and line numbers for each issue
- Quote exact AI Agent Instructions rules when citing violations
- File paths should be relative to repository root
- Line numbers reference lines in working copy (not diff line numbers)
- Maintain a todo list to track review progress
- Each issue should appear only once in the output (deduplicate)
