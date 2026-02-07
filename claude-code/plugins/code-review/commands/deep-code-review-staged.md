---
name: deep-code-review-staged
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Deep code review of staged changes with 19 agent invocations
argument-hint: "[--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a comprehensive code review using all 9 agents (19 invocations total) for staged git changes. Execute agents with both thorough and gaps modes for maximum coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location (default: see `output-format.md` Filename Generation)
- Optional: `--language dotnet|nodejs|react` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills

Invoke superpowers skills (using-superpowers, brainstorming, systematic-debugging, verification-before-completion, writing-plans, executing-plans, requesting-code-review, dispatching-parallel-agents, subagent-driven-development) and pass methodology to all subagents via `skill_instructions.methodology`. If superpowers plugin unavailable, proceed without.

## Step 2: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md`. If `.claude/code-review.local.md` has `enabled: false`, stop. Apply: `output_dir`, `skip_agents`, `min_severity`, `language`.

## Step 4: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md`.

---

## Steps 3 & 5: Input Validation and Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/staged-processing.md` for the validation, content gathering, and tiered context behavior.

---

## Step 6: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 7: Two-Phase Deep Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/invocation-patterns.md` for Task invocation template

### Phase 1: Thorough Review (9 agents in parallel)

Launch all 9 agents with **thorough** mode. See `orchestration-sequence.md` for model assignments.

**Agents**: API Contracts, Architecture, Bug Detection, Compliance, Error Handling, Performance, Security, Technical Debt, Test Coverage

Each agent receives staged diff, full file content, and AI Agent Instructions.

### Phase 2: Gaps Review (5 Sonnet agents in parallel)

**CRITICAL: WAIT** - All Phase 1 agents must complete before starting Phase 2.

After Phase 1 completes, launch 5 agents with **gaps** mode, passing Phase 1 findings as `previous_findings`.

**Agents**: Bug Detection, Compliance, Performance, Security, Technical Debt

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` "Gaps Mode Behavior Template" for gaps mode rules (duplicate detection, constraints). See each agent's MODE Parameter section for category-specific focus areas.

**CRITICAL: WAIT** - All Phase 2 agents must complete before proceeding to Synthesis.

---

## Step 8: Cross-Agent Synthesis (5 agents in parallel)

**CRITICAL: Synthesis receives ALL findings from prior phases. Do NOT launch until prior phases are FULLY COMPLETE.**

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for category pairs.

Launch 5 synthesis agents with category pairs from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate per `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` and `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-code.md`. Aggregate: filter invalid, apply severity downgrades, deduplicate by file+line range, add consensus badges. Generate output per `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`. Write to file.

**Output config:** Review Type: "Deep (19 invocations)", Categories: All 9

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing AI Agent Instructions rules, quote the exact rule being violated.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not diff line numbers for file reviews, working copy lines for staged reviews).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
