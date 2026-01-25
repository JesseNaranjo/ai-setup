---
name: deep-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Deep 16-agent review of staged changes
argument-hint: "[--output-file <path>] [--language nodejs|dotnet] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a comprehensive code review using all 9 agents (16 invocations total) for staged git changes. Execute agents with both thorough and gaps modes for maximum coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location (default: `.deep-review-staged.md`)
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

## Step 6: Two-Phase Deep Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template
- `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` for workflow steps and settings application

### Usage Tracking

Initialize per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
- Record `review_started_at` timestamp
- Initialize 3 phases: "Phase 1: Thorough Review", "Phase 2: Gaps Review", "Synthesis"

### Phase 1: Thorough Review (8 agents in parallel)

Launch all 8 agents with **thorough** mode. See review-workflow.md for model assignments.

**Agents**: Compliance, Bug Detection, Security, Performance, Architecture, API Contracts, Error Handling, Test Coverage

Each agent receives staged diff, full file content, and AI Agent Instructions.

### Phase 2: Gaps Review (4 Sonnet agents in parallel)

After Phase 1 completes, launch 4 agents with **gaps** mode, passing Phase 1 findings as `previous_findings`.

**Agents**: Compliance, Bug Detection, Security, Performance

See `${CLAUDE_PLUGIN_ROOT}/shared/gaps-mode-rules.md` for gaps mode operation.

---

## Step 7: Cross-Agent Synthesis (4 agents in parallel)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` "Cross-Agent Synthesis" section.

Launch 4 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`.

---

## Steps 8-11: Validation, Aggregation, Output

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` for:
- **Validation Phase** (Step 8)
- **Aggregation Phase** (Step 9)
- **Output Generation** (Step 10): Review Type: "Deep (16 invocations)", Categories: All 8
- **Write Output** (Step 11)
- **False Positives**
- **Notes** (line numbers reference working copy, not diff)
