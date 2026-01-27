---
name: deep-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Deep 19-agent code review of staged changes
argument-hint: "[--output-file <path>] [--language nodejs|dotnet] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a comprehensive code review using all 9 agents (19 invocations total) for staged git changes. Execute agents with both thorough and gaps modes for maximum coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location (default: `.deep-review-staged.md`)
- Optional: `--language nodejs|dotnet` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Common Steps (Steps 1, 3, 5)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

---

## Step 2: Staged Changes Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-staged.md` for the validation process.

---

## Step 4: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-staged.md` for the content gathering process and tiered context behavior.

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

### Phase 1: Thorough Review (9 agents in parallel)

Launch all 9 agents with **thorough** mode. See review-workflow.md for model assignments.

**Agents**: API Contracts, Architecture, Bug Detection, Compliance, Error Handling, Performance, Security, Technical Debt, Test Coverage

Each agent receives staged diff, full file content, and AI Agent Instructions.

### Phase 2: Gaps Review (5 Sonnet agents in parallel)

After Phase 1 completes, launch 5 agents with **gaps** mode, passing Phase 1 findings as `previous_findings`.

**Agents**: Bug Detection, Compliance, Performance, Security, Technical Debt

See each agent's "Gaps Mode Behavior" section for gaps mode rules.

---

## Step 7: Cross-Agent Synthesis (5 agents in parallel)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` "Cross-Agent Synthesis" section.

Launch 5 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`.

---

## Steps 8-11: Validation, Aggregation, Output

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

**Output config:** Review Type: "Deep (19 invocations)", Categories: All 9
