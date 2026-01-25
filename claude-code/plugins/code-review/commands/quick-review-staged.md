---
name: quick-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Quick 7-agent review of staged changes
argument-hint: "[--output-file <path>] [--language nodejs|dotnet] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a fast 4-agent code review for staged git changes, focusing on bugs, security, error handling, and test coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location (default: `.quick-review-staged.md`)
- Optional: `--language nodejs|dotnet` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Common Steps (Steps 1, 3, 5)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` for:
- **Step 1**: Load Settings
- **Step 3**: Context Discovery
- **Step 5**: Skill Loading and Interpretation

---

## Step 2: Staged Changes Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-staged.md` for the validation process.

---

## Step 4: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-staged.md` for the content gathering process.

**Tiered Context:** Staged reviews use tiered context:
- Changed files receive full content (critical tier)
- Unchanged files receive preview + metadata (peripheral tier)
- Agents can Read peripheral files on-demand if cross-file analysis discovers relevance

---

## Step 6: 4-Agent Quick Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template
- `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` for workflow steps and settings application

### Usage Tracking

Initialize per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
- Record `review_started_at` timestamp
- Initialize 2 phases: "Review", "Synthesis"

### Review Phase (4 agents in parallel)

Launch 4 agents with **quick** mode. See review-workflow.md for model assignments.

**Agents**: Bug Detection (Opus), Security (Opus), Error Handling (Sonnet), Test Coverage (Sonnet)

Each agent receives staged diff, full file content, and AI Agent Instructions.

**Quick mode agents focus on**:
- Critical and Major severity issues only
- Most obvious and impactful problems
- Issues that would block a merge

---

## Step 7: Cross-Agent Synthesis (3 agents in parallel)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` "Cross-Agent Synthesis" section.

Launch 3 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`.

---

## Steps 8-11: Validation, Aggregation, Output

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` for:
- **Validation Phase** (Step 8): Critical/Major only; Minor and Suggestions skip validation
- **Aggregation Phase** (Step 9)
- **Output Generation** (Step 10): Review Type: "Quick (7 invocations)", Categories: 4 only
- **Write Output** (Step 11)
- **False Positives**: Quick review should be extra conservative - skip theoretical edge cases
- **Notes** (line numbers reference working copy, not diff)

**Footer note**: *For comprehensive review (compliance, performance, architecture, API), run `/deep-review-staged`*
