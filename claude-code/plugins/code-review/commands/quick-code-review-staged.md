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

See `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md` for the validation, content gathering, and tiered context behavior.

---

## Step 6: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 7: 4-Agent Quick Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` for phase definitions and **Code Review Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` for Task invocation template

### Review Phase (4 agents in parallel)

Launch 4 agents with **quick** mode. See `review-orchestration-code.md` for model assignments.

**Agents**: Bug Detection (Opus), Security (Opus), Error Handling (Sonnet), Test Coverage (Sonnet)

Each agent receives staged diff, full file content, and AI Agent Instructions.

**Quick mode agents focus on**:
- Critical and Major severity issues only
- Most obvious and impactful problems
- Issues that would block a merge

**CRITICAL: WAIT** - All Review phase agents must complete before proceeding to Synthesis.

---

## Step 8: Cross-Agent Synthesis (3 agents in parallel)

**CRITICAL: Synthesis receives ALL findings from prior phases. Do NOT launch until prior phases are FULLY COMPLETE.**

See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` for category pairs.

Launch 3 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate per `${CLAUDE_PLUGIN_ROOT}/shared/validation-aggregation.md` and `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "Code Review Validation Rules" (domain-specific patterns). Aggregate: filter invalid, apply severity downgrades, deduplicate by file+line range, add consensus badges. Generate output per `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`. Write to file.

**Output config:** Review Type: "Quick (7 invocations)", Categories: 4 only
**Note:** Quick review should be extra conservative - skip theoretical edge cases.

*For comprehensive review (compliance, performance, architecture, API), run `/deep-code-review-staged`*

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing AI Agent Instructions rules, quote the exact rule being violated.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not diff line numbers for file reviews, working copy lines for staged reviews).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
