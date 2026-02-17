---
name: code-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Code review of staged git changes with configurable depth (deep: 19 agent invocations, quick: 7)
argument-hint: "[--depth deep|quick] [--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a code review for staged git changes. Depth controls the review pipeline: deep uses all 9 agents (19 invocations total) with thorough + gaps modes for maximum coverage; quick uses 4 agents (7 invocations) focusing on bugs, security, error handling, and test coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--depth deep|quick` (default: `deep`)
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

## Step 7: Review Execution

**If depth == deep:**
Execute the **Deep Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`:
- Use the **Code Review Model Selection** table for model assignments
- Use the **Agent Common Content Distribution** rules to build each agent's `additional_instructions`
- Follow all CRITICAL WAIT barriers between phases

Each agent receives staged diff and full file content per the tier classification in `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md`.

**If depth == quick:**
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

Validate, aggregate, and generate output per `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`. Write to file.

**Output config (deep):** Review Type: "Deep (19 invocations)", Categories: All 9
**Output config (quick):** Review Type: "Quick (7 invocations)", Categories: 4 only
**Note (quick only):** Quick review should be extra conservative - skip theoretical edge cases.
