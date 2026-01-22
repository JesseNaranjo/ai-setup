---
name: deep-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Deep code review with 16 agent invocations (8 thorough + 4 gaps + 4 synthesis)
argument-hint: "<file1> [file2...] [--output-file <path>] [--language nodejs|dotnet]"
model: opus
---

Perform a comprehensive code review using all 9 agents (16 invocations total) for the specified files. For files with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

Parse arguments from `$ARGUMENTS`:
- Required: One or more file paths (space-separated)
- Optional: `--output-file <path>` to specify output location
- Optional: `--language nodejs|dotnet` to force language detection

---

## Step 0: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md` for the settings loading process.

Check for `.claude/code-review.local.md` in the project root:
- If found and `enabled: false`, stop with message
- Apply settings: `output_dir`, `skip_agents`, `min_severity`, `language`, `custom_rules`
- Read project-specific instructions from markdown body

---

## Step 1: Input Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-files.md` for the validation process.

**Default output file**: `{output_dir}/.deep-review.md` (default: `.deep-review.md`)

---

## Step 2: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md` for:
- AI Agent Instructions file patterns
- Project type detection rules
- Related test file patterns per language

---

## Step 3: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-files.md` for the content gathering process.

---

## Step 4: Two-Phase Deep Review

See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` for orchestration logic and the **Agent Invocation Pattern** section in `${CLAUDE_PLUGIN_ROOT}/shared/skill-common-workflow.md` for the exact Task tool invocation format.

**Agent invocation uses Task tool with subagent_type** (e.g., `code-review:compliance-agent`), not file paths directly.

Deep review uses a sequential two-phase approach to reduce duplicates and improve gaps analysis.

### Usage Tracking Initialization

Before launching agents, initialize usage tracking per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
1. Record `review_started_at` timestamp
2. Initialize 3 phases: "Phase 1: Thorough Review", "Phase 2: Gaps Review", "Synthesis"

### Phase 1: Thorough Review (8 agents in parallel)

Launch all 8 agents with **thorough** mode. See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` "Model Selection per Agent" table for model assignments.

**Agents**: Compliance, Bug Detection, Security, Performance, Architecture, API Contracts, Error Handling, Test Coverage

Each agent receives:
- The current branch name
- A summary of what is being reviewed
- Which files have changes vs which are being reviewed in full
- The detected project type (for language-specific focus from `${CLAUDE_PLUGIN_ROOT}/languages/*.md`)
- The AI Agent Instructions files relevant to each file being reviewed
- For files with changes: both the diff AND full file content
- For files without changes: the full file content
- Related test files for context
- MODE parameter: **thorough**

**Usage Tracking - Phase 1:**
1. Record `phase_started_at` before launching agents
2. For each agent: record `agent_started_at` before Task call, `agent_ended_at` and `task_id` after Task returns
3. Record `phase_ended_at` when all agents complete

**Wait for all Phase 1 agents to complete before proceeding.**

### Phase 2: Gaps Review with Context (4 Sonnet agents in parallel)

After Phase 1 completes, launch 4 agents with **gaps** mode (all use Sonnet), passing Phase 1 findings. See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` for model assignments.

**Agents**: Compliance, Bug Detection, Security, Performance

Each gaps agent receives all Phase 1 inputs PLUS:
- **previous_findings**: List of issues already flagged in this category from Phase 1

See `${CLAUDE_PLUGIN_ROOT}/shared/gaps-mode-rules.md` for gaps mode operation.

**Note**: Sonnet is used for gaps mode because this phase is constrained by prior findings context and explicit checklists, making it suitable for a more cost-efficient model while retaining quality.

**Usage Tracking - Phase 2:**
1. Record `phase_started_at` before launching gaps agents
2. For each agent: record `agent_started_at` before Task call, `agent_ended_at` and `task_id` after Task returns
3. Record `phase_ended_at` when all agents complete

---

## Step 5: Cross-Agent Synthesis

After Phase 1 and Phase 2 complete, launch 4 synthesis agents in parallel.

See `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` for full agent definition.

| Synthesis Agent | Subagent Type | Input Categories | Cross-Cutting Question |
|-----------------|---------------|-----------------|------------------------|
| Synthesis | `code-review:synthesis-agent` | Security + Performance | "Do any security fixes introduce performance issues?" |
| Synthesis | `code-review:synthesis-agent` | Architecture + Test Coverage | "Are architectural changes covered by tests?" |
| Synthesis | `code-review:synthesis-agent` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| Synthesis | `code-review:synthesis-agent` | Compliance + Bugs | "Do compliance violations introduce or mask bugs?" |

Launch all 4 synthesis agents in parallel, each with their respective category pairs and findings from Phase 1 and Phase 2.

**Usage Tracking - Synthesis:**
1. Record `phase_started_at` before launching synthesis agents
2. For each synthesis agent: record `agent_started_at` before Task call, `agent_ended_at` and `task_id` after Task returns
3. Record `phase_ended_at` when all agents complete
4. Record `review_ended_at` timestamp

**Output**: `cross_cutting_insights` list added to the issue pool before validation.

---

## Step 6: Universal Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for complete validation process including:
- Auto-validation patterns
- Batch grouping by file
- Validator model assignment

---

## Step 7: Aggregation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for aggregation rules:
- Filter invalid issues
- Apply severity downgrades
- Deduplicate by file+line range
- Add consensus badges

---

## Step 8: Generate Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` and `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for formatting.

**Generate Usage Summary first:**
1. Generate Usage Summary from tracking data per `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` Usage Summary Section
2. Include Phase Breakdown: Phase 1 (8 agents), Phase 2 (4 agents), Synthesis (4 agents)
3. Include Agent Timing Details in expandable section
4. Flag timing anomalies with `[!]` (too fast) or `[*]` (too slow) indicators

**Then generate Code Review output:**

**Review depth**: "Deep (16 invocations: 8 thorough + 4 gaps + 4 synthesis)"

**Categories to include in summary table**: All 8 (Compliance, Bugs, Security, Performance, Architecture, API Contracts, Error Handling, Test Coverage)

---

## Step 9: Write Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` for write process.

**Default output file**: `.deep-review.md`

---

## False Positives

See `${CLAUDE_PLUGIN_ROOT}/shared/false-positives.md` for issues that should NOT be flagged.

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing AI Agent Instructions rules, quote the exact rule being violated.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not diff line numbers).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
