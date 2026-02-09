---
name: quick-docs-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Bash(find:*), Read, Write, Glob
description: Quick documentation review with 7 agent invocations and synthesis
argument-hint: "[file1...] [--output-file <path>] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a quick documentation review using 4 documentation agents (7 invocations total). Focuses on critical issues: accuracy, clarity, examples, and structure.

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
- Standard docs: README.md, CLAUDE.md, CHANGELOG.md
- Documentation directories: docs/**/*.md
- AI instruction files: .ai/AI-AGENT-INSTRUCTIONS.md, .github/copilot-instructions.md

Track AI instruction file standardization status for reporting.

---

## Step 4: Skill Loading

**Skip this step entirely if `--skills` argument was not provided.**

If `--skills` is provided:
See `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md` for complete skill resolution and orchestration.

---

## Step 5: Context Discovery

Detect project type and gather context:
- Read package.json, *.csproj, pyproject.toml for version info
- Identify the primary programming language(s)

---

## Step 6: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/docs-processing.md` for the content gathering process.

Gather:
- Full content of documentation files
- Code snippets referenced by documentation (for accuracy verification)
- Project metadata (versions)

---

## Step 7: Quick Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` "Documentation Review Orchestration" for phase definitions and **Documentation Review Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` for Task invocation template

### Review (4 agents in parallel)

Launch 4 documentation agents with **quick** mode:

**Agents**: Accuracy, Clarity, Examples, Structure

- Accuracy (Opus): Critical mismatches, wrong function names, broken examples
- Clarity (Sonnet): Incomprehensible sections, undefined critical acronyms
- Examples (Opus): Syntax errors, missing critical imports, wrong API calls
- Structure (Sonnet): Broken links, major navigation issues, AI instruction file errors

Pass to all agents:
- Documentation file contents
- Related code snippets for verification
- AI instruction file status
- Additional prompt instructions (if provided)

**CRITICAL: WAIT** - All 4 Review phase agents must complete before proceeding to Synthesis.

---

## Step 8: Cross-Agent Synthesis (3 agents in parallel)

**CRITICAL: DO NOT START Synthesis until the Review phase (Step 7) is FULLY COMPLETE.**

See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` "Documentation Review Orchestration" for synthesis pairs.

Launch 3 synthesis agents with category pairs:
- Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
- Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
- Examples+Structure: "Are example placements and references structurally sound?"

---

## Step 9: Validation

Validate all findings per `${CLAUDE_PLUGIN_ROOT}/shared/references/validation-rules-docs.md`:
- Filter false positives
- Verify issue locations exist
- Remove duplicates

---

## Step 10: Aggregation

Aggregate validated findings:
- Group by category (Accuracy, Clarity, Examples, Structure)
- Sort by severity within categories
- Include cross-cutting insights from synthesis

---

## Step 11: Output

Generate the review report using `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`.

**Output config:**
- Review Type: "Quick Documentation Review (7 invocations)"
- Categories: Accuracy, Clarity, Examples, Structure

Write to the output file path (see `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` "Filename Generation").

Report completion to user with summary:
- Total issues found by severity
- Issues by category
- AI instruction standardization status (if issues found)
- Path to full report
