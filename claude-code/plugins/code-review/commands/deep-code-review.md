---
name: deep-code-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Deep code review with 19 agent invocations and synthesis
argument-hint: "<file1> [file2...] [--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a comprehensive code review using all 9 agents (19 invocations total) for the specified files. For files with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

Parse arguments from `$ARGUMENTS`:
- Required: One or more file paths (space-separated)
- Optional: `--output-file <path>` to specify output location (default: `.deep-code-review.md`)
- Optional: `--language dotnet|nodejs|react` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Common Steps (Steps 2, 4, 6)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

---

## Step 3: Input Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-files.md` for the validation process.

---

## Step 5: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-files.md` for the content gathering process.

---

## Step 7: Two-Phase Deep Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template

### Usage Tracking

Initialize per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
- Record `review_started_at` timestamp
- Initialize 3 phases: "Phase 1: Thorough Review", "Phase 2: Gaps Review", "Synthesis"

### Phase 1: Thorough Review (9 agents in parallel)

Launch all 9 agents with **thorough** mode. See `orchestration-sequence.md` for model assignments.

**Agents**: API Contracts, Architecture, Bug Detection, Compliance, Error Handling, Performance, Security, Technical Debt, Test Coverage

### Phase 2: Gaps Review (5 Sonnet agents in parallel)

**CRITICAL: WAIT and RECORD** - All Phase 1 agents must complete. Record timing/task_id per `usage-tracking.md` before starting Phase 2.

After Phase 1 completes, launch 5 agents with **gaps** mode, passing Phase 1 findings as `previous_findings`.

**Agents**: Bug Detection, Compliance, Performance, Security, Technical Debt

See each agent's "Gaps Mode Behavior" section for gaps mode rules.

**CRITICAL: WAIT and RECORD** - All Phase 2 agents must complete. Record timing/task_id per `usage-tracking.md` before proceeding to Synthesis.

---

## Step 8: Cross-Agent Synthesis (5 agents in parallel)

**CRITICAL: DO NOT START Synthesis until Phase 1 AND Phase 2 (Step 7) are FULLY COMPLETE.**

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` "Cross-Agent Synthesis" section.

Launch 5 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`.

---

## Steps 9-12: Validation, Aggregation, Output

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

**Output config:** Review Type: "Deep (19 invocations)", Categories: All 9
