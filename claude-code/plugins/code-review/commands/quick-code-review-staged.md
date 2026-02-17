---
name: quick-code-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Quick code review of staged changes with 7 agent invocations
argument-hint: "[--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a fast code review using 4 agents (7 invocations total) for staged git changes, focusing on bugs, security, error handling, and test coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location (default: see Filename Generation in review-orchestration-code.md)
- Optional: `--language dotnet|nodejs|react` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills

Invoke superpowers skills (using-superpowers, brainstorming, systematic-debugging, verification-before-completion, writing-plans, executing-plans, requesting-code-review, dispatching-parallel-agents, subagent-driven-development) and pass methodology to all subagents via `skill_instructions.methodology`. If superpowers plugin unavailable, proceed without.

## Step 2: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/pre-review-setup.md` Section 1. If `.claude/code-review.local.md` has `enabled: false`, stop. Apply: `output_dir`, `skip_agents`, `min_severity`, `language`.

## Step 4: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/pre-review-setup.md` Section 2.

---

## Steps 3 & 5: Input Validation and Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md` for the validation, content gathering, and tiered context behavior. Include the Pre-Existing Issue Detection rules from `staged-processing.md` in each agent's `additional_instructions` prompt field.

---

## Step 6: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 7: Quick Review

Execute the **Quick Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`:
- Use the **Code Review Model Selection** table for model assignments
- Use the **Agent Common Content Distribution** rules to build each agent's `additional_instructions`
- Follow all CRITICAL WAIT barriers

Each agent receives staged diff and full file content per the tier classification in `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md`.

---

## Step 8: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate per `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`. Aggregate: filter invalid, apply severity downgrades, deduplicate by file+line range, add consensus badges. Generate output per `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`. Write to file.

**Output config:** Review Type: "Quick (7 invocations)", Categories: 4 only
**Note:** Quick review should be extra conservative - skip theoretical edge cases.

*For comprehensive review (compliance, performance, architecture, API), run `/deep-code-review-staged`*
