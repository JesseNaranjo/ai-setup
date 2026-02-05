---
name: quick-code-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Quick code review of staged changes with 7 agent invocations
argument-hint: "[--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a fast code review using 4 agents (7 invocations total) for staged git changes, focusing on bugs, security, error handling, and test coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location (default: see `output-format.md` Filename Generation)
- Optional: `--language dotnet|nodejs|react` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Common Steps (Steps 2, 4, 6)

See `${CLAUDE_PLUGIN_ROOT}/shared/command-common-steps.md`.

---

## Steps 3 & 5: Input Validation and Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md` for the validation, content gathering, and tiered context behavior.

---

## Step 7: 4-Agent Quick Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template

### Review Phase (4 agents in parallel)

Launch 4 agents with **quick** mode. See `orchestration-sequence.md` for model assignments.

**Agents**: Bug Detection (Opus), Security (Opus), Error Handling (Sonnet), Test Coverage (Sonnet)

Each agent receives staged diff, full file content, and AI Agent Instructions.

**Quick mode agents focus on**:
- Critical and Major severity issues only
- Most obvious and impactful problems
- Issues that would block a merge

**CRITICAL: WAIT** - All Review phase agents must complete before proceeding to Synthesis.

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

*For comprehensive review (compliance, performance, architecture, API), run `/deep-code-review-staged`*
