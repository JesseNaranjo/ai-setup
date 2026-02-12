---
name: deep-docs-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Bash(find:*), Read, Write, Glob
description: Deep documentation review with 13 agent invocations and synthesis
argument-hint: "[file1...] [--output-file <path>] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a comprehensive documentation review using all 6 documentation agents (13 invocations total). If no files specified, discover and review all documentation files in the project.

Parse arguments from `$ARGUMENTS`:
- Optional: One or more file paths (space-separated) - if omitted, discover all docs
- Optional: `--output-file <path>` to specify output location (default: see Filename Generation in review-orchestration-docs.md)
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills

Invoke superpowers skills (using-superpowers, brainstorming, systematic-debugging, verification-before-completion, writing-plans, executing-plans, requesting-code-review, dispatching-parallel-agents, subagent-driven-development) and pass methodology to all subagents via `skill_instructions.methodology`. If superpowers plugin unavailable, proceed without.

---

## Step 2: Settings

Load project-specific settings if `.claude/code-review.local.md` exists.

See `${CLAUDE_PLUGIN_ROOT}/shared/pre-review-setup.md` Section 1.

---

## Step 3: Input Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/docs-processing.md` for the validation process.

Discover documentation files including:
- Standard docs: README.md, CLAUDE.md, CHANGELOG.md, CONTRIBUTING.md
- Documentation directories: docs/**/*.md
- AI instruction files: .ai/AI-AGENT-INSTRUCTIONS.md, .github/copilot-instructions.md

Track AI instruction file standardization status for reporting.

---

## Step 4: Context Discovery

Detect project type and gather context:
- Read package.json, *.csproj, pyproject.toml for version info
- Identify the primary programming language(s)
- Note any style guides or contribution guidelines

---

## Step 5: Skill Loading

**Skip this step entirely if `--skills` argument was not provided.**

If `--skills` is provided:
See `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md` for complete skill resolution and orchestration.

---

## Step 6: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/docs-processing.md` for the content gathering process.

Gather:
- Full content of all documentation files
- Code snippets referenced by documentation (for accuracy verification)
- Project metadata (versions, dependencies)
- AI instruction file cross-reference status

---

## Step 7: Two-Phase Deep Review

Execute the **Deep Docs Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md`:
- Use the **Documentation Review Model Selection** table for model assignments
- Use the **Agent Common Content Distribution** rules
- Follow all CRITICAL WAIT barriers between phases

---

## Step 8: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate per `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-docs.md`. Aggregate: filter invalid, apply severity downgrades, deduplicate by file+line range, add consensus badges. Generate output per `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`. Write to file.

**Output config:** Review Type: "Deep Documentation Review (13 invocations)", Categories: All 6
Include AI instruction file standardization section. Report summary: total issues by severity, issues by category, AI instruction standardization status, path to report.
