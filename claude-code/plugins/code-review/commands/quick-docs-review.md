---
name: quick-docs-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Bash(find:*), Read, Write, Glob
description: Quick 7-agent documentation review with synthesis
argument-hint: "[file1...] [--output-file <path>] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a quick documentation review using 4 documentation agents (7 invocations total). Focuses on critical issues: accuracy, clarity, examples, and structure.

Parse arguments from `$ARGUMENTS`:
- Optional: One or more file paths (space-separated) - if omitted, discover all docs
- Optional: `--output-file <path>` to specify output location (default: `.quick-docs-review.md`)
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Settings

Load project-specific settings if `.claude/code-review.local.md` exists.

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md`.

---

## Step 2: Input Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-docs.md` for the validation process.

Discover documentation files including:
- Standard docs: README.md, CLAUDE.md, CHANGELOG.md
- Documentation directories: docs/**/*.md
- AI instruction files: .ai/AI-AGENT-INSTRUCTIONS.md, .github/copilot-instructions.md

Track AI instruction file standardization status for reporting.

---

## Step 3: Context Discovery

Detect project type and gather context:
- Read package.json, *.csproj, pyproject.toml for version info
- Identify the primary programming language(s)

---

## Step 4: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-docs.md` for the content gathering process.

Gather:
- Full content of documentation files
- Code snippets referenced by documentation (for accuracy verification)
- Project metadata (versions)

---

## Step 5: Quick Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/docs-orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template

### Usage Tracking

Initialize per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
- Record `review_started_at` timestamp
- Initialize 2 phases: "Review", "Synthesis"

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

---

## Step 6: Cross-Agent Synthesis (3 agents in parallel)

See `${CLAUDE_PLUGIN_ROOT}/shared/docs-orchestration-sequence.md` for synthesis pairs.

Launch 3 synthesis agents with category pairs:
- Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
- Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
- Examples+Structure: "Are example placements and references structurally sound?"

---

## Step 7: Validation

Validate all findings per `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`:
- Filter false positives
- Verify issue locations exist
- Remove duplicates

---

## Step 8: Aggregation

Aggregate validated findings:
- Group by category (Accuracy, Clarity, Examples, Structure)
- Sort by severity within categories
- Include cross-cutting insights from synthesis

---

## Step 9: Output

Generate the review report using `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`.

**Output config:**
- Review Type: "Quick Documentation Review (7 invocations)"
- Categories: Accuracy, Clarity, Examples, Structure

Write to the output file path (default: `.quick-docs-review.md`).

Report completion to user with summary:
- Total issues found by severity
- Issues by category
- AI instruction standardization status (if issues found)
- Path to full report
