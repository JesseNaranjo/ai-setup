---
name: code-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Code review for files or staged changes with configurable depth (deep: 19 agent invocations, quick: 7)
argument-hint: "[<file1> [file2...] | --staged] [--depth deep|quick] [--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a code review for specified files or staged git changes. Depth controls the review pipeline: deep uses all 9 agents (19 invocations total) with thorough + gaps modes; quick uses 4 agents (7 invocations) focusing on bugs, security, error handling, and test coverage. For file reviews with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

Parse arguments from `$ARGUMENTS`:
- Mutually exclusive: One or more file paths (space-separated) OR `--staged` flag
- Optional: `--depth deep|quick` (default: `deep`)
- Optional: `--output-file <path>` to specify output location (default: see Filename Generation in review-orchestration-code.md)
- Optional: `--language dotnet|nodejs|react` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

Error if both file paths and `--staged` are provided.

---

## Step 1: Invoke Methodology Skills

Invoke superpowers methodology skills; pass to agents via `skill_instructions.methodology`. If unavailable, proceed without.

## Step 2: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/pre-review-setup.md` Section 1. Stop if `enabled: false`. Apply: `output_dir`, `skip_agents`, `min_severity`, `language`.

## Step 4: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/pre-review-setup.md` Section 2.

---

## Steps 3 & 5: Input Validation and Content Gathering

**If `--staged`:**
See `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md` for the validation, content gathering, and tiered context behavior. Include the Pre-Existing Issue Detection rules from `staged-processing.md` in each agent's `additional_instructions` prompt field.

**If file paths provided:**
See `${CLAUDE_PLUGIN_ROOT}/shared/file-processing.md` for the validation and content gathering process.

---

## Step 6: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 7: Review Execution

**If depth == deep:**
Execute the **Deep Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. Follow all CRITICAL WAIT barriers between phases.

**If depth == quick:**
Execute the **Quick Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. Follow all CRITICAL WAIT barriers.

**If `--staged`:** Agents receive staged diff and file content per tier classification in `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md`.

---

## Step 8: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate, aggregate, and generate output per `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`. Write to file.

**Output config (deep):** Review Type: "Deep (19 invocations)", Categories: All 9
**Output config (quick):** Review Type: "Quick (7 invocations)", Categories: 4 only
**Note (quick only):** Quick review should be extra conservative - skip theoretical edge cases.
