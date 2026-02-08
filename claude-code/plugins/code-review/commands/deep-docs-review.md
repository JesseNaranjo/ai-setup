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
- Optional: `--output-file <path>` to specify output location (default: see `output-format.md` Filename Generation)
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills (Recommended)

Before beginning the review workflow, invoke the following skills and pass their methodology to subagents:

1. `superpowers:using-superpowers` - Skill usage methodology (ALWAYS FIRST)
2. `superpowers:brainstorming` - Explore interpretations before concluding
3. `superpowers:verification-before-completion` - Verify findings before reporting

**Fallback:** If the superpowers plugin is not installed, proceed with standard review workflow.

---

## Step 2: Settings

Load project-specific settings if `.claude/code-review.local.md` exists.

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md`.

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

See:
- `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` "Documentation Review Orchestration" for phase definitions and **Documentation Review Model Selection** table
- `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` for Task invocation template

### Phase 1: Thorough Review (6 agents in parallel)

Launch all 6 documentation agents with **thorough** mode. See `review-orchestration-docs.md` "Documentation Review Model Selection" for model assignments.

**Agents**: Accuracy, Clarity, Completeness, Consistency, Examples, Structure

Pass to all agents:
- Documentation file contents
- Related code snippets for verification
- Project metadata
- AI instruction file status
- Additional prompt instructions (if provided)

### Phase 2: Gaps Review (3 Sonnet agents in parallel)

**CRITICAL: WAIT** - All Phase 1 agents must complete before starting Phase 2.

After Phase 1 completes, launch 3 agents with **gaps** mode, passing Phase 1 findings as `previous_findings`.

**Agents**: Accuracy, Completeness, Consistency

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-common-instructions.md` "Gaps Mode Behavior Template" for gaps mode rules (duplicate detection, constraints). See each agent's MODE Parameter section for category-specific focus areas.

**CRITICAL: WAIT** - All Phase 2 agents must complete before proceeding to Synthesis.

---

## Step 8: Cross-Agent Synthesis (4 agents in parallel)

**CRITICAL: DO NOT START Synthesis until Phase 1 AND Phase 2 (Step 7) are FULLY COMPLETE.**

See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` "Documentation Review Orchestration" for synthesis pairs.

Launch 4 synthesis agents with category pairs:
- Accuracy+Examples: "Do code examples match the documented behavior they claim to demonstrate?"
- Clarity+Structure: "Does poor structure contribute to clarity issues, or vice versa?"
- Completeness+Consistency: "Are missing sections causing terminology inconsistencies elsewhere?"
- Consistency+Structure: "Do formatting inconsistencies reflect structural organization problems?"

---

## Step 9: Validation

Validate all findings per `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-docs.md`:
- Filter false positives
- Verify issue locations exist
- Remove duplicates across agents

---

## Step 10: Aggregation

Aggregate validated findings:
- Group by category (Accuracy, Clarity, Completeness, Consistency, Examples, Structure)
- Sort by severity within categories
- Include cross-cutting insights from synthesis

---

## Step 11: Output

Generate the review report using `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md`.

**Output config:**
- Review Type: "Deep Documentation Review (13 invocations)"
- Categories: All 6 documentation categories
- Include AI instruction file standardization section

Write to the output file path (see `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for filename generation).

Report completion to user with summary:
- Total issues found by severity
- Issues by category
- AI instruction standardization status
- Path to full report
