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

See `${CLAUDE_PLUGIN_ROOT}/shared/file-processing.md` for the validation and content gathering process.

---

## Step 6: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 7: Quick Review

Execute the **Quick Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`:
- Use the **Code Review Model Selection** table for model assignments
- Use the **Agent Common Content Distribution** rules to build each agent's `additional_instructions`
- Follow all CRITICAL WAIT barriers

---

## Step 8: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate per `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-code.md`. Aggregate: filter invalid, apply severity downgrades, deduplicate by file+line range, add consensus badges. Generate output per `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`. Write to file.

**Output config:** Review Type: "Quick (7 invocations)", Categories: 4 only
**Note:** Quick review should be extra conservative - skip theoretical edge cases.

*For comprehensive review, run `/deep-code-review <files>` or `/deep-code-review-staged` for staged changes.*
