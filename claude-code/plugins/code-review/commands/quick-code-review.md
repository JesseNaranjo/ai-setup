---
name: quick-code-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Quick code review with 7 agent invocations and synthesis
argument-hint: "<file1> [file2...] [--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a fast code review using 4 agents (7 invocations total) for the specified files, focusing on bugs, security, error handling, and test coverage. For files with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

Parse arguments from `$ARGUMENTS`:
- Required: One or more file paths (space-separated)
- Optional: `--output-file <path>` to specify output location (default: `.quick-code-review.md`)
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

## Step 7: 4-Agent Quick Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template

### Usage Tracking

Initialize per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
- Record `review_started_at` timestamp
- Initialize 2 phases: "Review", "Synthesis"

### Review Phase (4 agents in parallel)

Launch 4 agents with **quick** mode. See `orchestration-sequence.md` for model assignments.

**Agents**: Bug Detection (Opus), Security (Opus), Error Handling (Sonnet), Test Coverage (Sonnet)

**Quick mode agents focus on**:
- Critical and Major severity issues only
- Most obvious and impactful problems
- Issues that would block a merge

**CRITICAL: WAIT and RECORD** - All Review phase agents must complete. Record timing/task_id per `usage-tracking.md` before proceeding to Synthesis.

---

## Step 8: Cross-Agent Synthesis (3 agents in parallel)

**CRITICAL: DO NOT START Synthesis until the Review phase (Step 7) is FULLY COMPLETE.**

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md` "Cross-Agent Synthesis" section.

Launch 3 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`.

---

## Steps 9-12: Validation, Aggregation, Output

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

**Output config:** Review Type: "Quick (7 invocations)", Categories: 4 only
**Note:** Quick review should be extra conservative - skip theoretical edge cases.

*For comprehensive review, run `/deep-code-review <files>` or `/deep-code-review-staged` for staged changes.*
