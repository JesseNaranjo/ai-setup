---
name: deep-docs-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Bash(find:*), Read, Write, Glob
description: Deep 13-agent documentation review with synthesis
argument-hint: "[file1...] [--output-file <path>] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a comprehensive documentation review using all 6 documentation agents (13 invocations total). If no files specified, discover and review all documentation files in the project.

Parse arguments from `$ARGUMENTS`:
- Optional: One or more file paths (space-separated) - if omitted, discover all docs
- Optional: `--output-file <path>` to specify output location (default: `.deep-docs-review.md`)
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
- Standard docs: README.md, CLAUDE.md, CHANGELOG.md, CONTRIBUTING.md
- Documentation directories: docs/**/*.md
- AI instruction files: .ai/AI-AGENT-INSTRUCTIONS.md, .github/copilot-instructions.md

Track AI instruction file standardization status for reporting.

---

## Step 3: Context Discovery

Detect project type and gather context:
- Read package.json, *.csproj, pyproject.toml for version info
- Identify the primary programming language(s)
- Note any style guides or contribution guidelines

---

## Step 4: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-docs.md` for the content gathering process.

Gather:
- Full content of all documentation files
- Code snippets referenced by documentation (for accuracy verification)
- Project metadata (versions, dependencies)
- AI instruction file cross-reference status

---

## Step 5: Two-Phase Deep Review

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/docs-orchestration-sequence.md` for phase definitions and **Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` for Task invocation template

### Usage Tracking

Initialize per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
- Record `review_started_at` timestamp
- Initialize 3 phases: "Phase 1: Thorough Review", "Phase 2: Gaps Review", "Synthesis"

### Phase 1: Thorough Review (6 agents in parallel)

Launch all 6 documentation agents with **thorough** mode. See `docs-orchestration-sequence.md` for model assignments.

**Agents**: Accuracy, Clarity, Completeness, Consistency, Examples, Structure

Pass to all agents:
- Documentation file contents
- Related code snippets for verification
- Project metadata
- AI instruction file status
- Additional prompt instructions (if provided)

### Phase 2: Gaps Review (3 Sonnet agents in parallel)

After Phase 1 completes, launch 3 agents with **gaps** mode, passing Phase 1 findings as `previous_findings`.

**Agents**: Accuracy, Completeness, Consistency

See each agent's "Gaps Mode Behavior" section for gaps mode rules.

---

## Step 6: Cross-Agent Synthesis (4 agents in parallel)

See `${CLAUDE_PLUGIN_ROOT}/shared/docs-orchestration-sequence.md` for synthesis pairs.

Launch 4 synthesis agents with category pairs:
- Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
- Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
- Completeness+Consistency: "Are missing sections causing terminology inconsistencies elsewhere?"
- Consistency+Structure: "Do formatting inconsistencies reflect structural organization problems?"

---

## Step 7: Validation

Validate all findings per `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md`:
- Filter false positives
- Verify issue locations exist
- Remove duplicates across agents

---

## Step 8: Aggregation

Aggregate validated findings:
- Group by category (Accuracy, Clarity, Completeness, Consistency, Examples, Structure)
- Sort by severity within categories
- Include cross-cutting insights from synthesis

---

## Step 9: Output

Generate the review report using `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`.

**Output config:**
- Review Type: "Deep Documentation Review (13 invocations)"
- Categories: All 6 documentation categories
- Include AI instruction file standardization section

Write to the output file path (default: `.deep-docs-review.md`).

Report completion to user with summary:
- Total issues found by severity
- Issues by category
- AI instruction standardization status
- Path to full report
