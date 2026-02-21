---
name: docs-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Bash(find:*), Read, Write, Glob
description: Documentation review with configurable depth (deep: up to 13 agent invocations, quick: up to 7)
argument-hint: "[file1...] [--depth deep|quick] [--output-file <path>] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a documentation review. Depth controls the review pipeline: deep uses all 6 documentation agents (up to 13 invocations total) with thorough + gaps modes; quick uses 4 agents (up to 7 invocations) focusing on critical issues (accuracy, clarity, examples, structure). If no files specified, discover and review all documentation files in the project.

Parse arguments from `$ARGUMENTS`:
- Optional: One or more file paths (space-separated) - if omitted, discover all docs
- Optional: `--depth deep|quick` (default: `deep`)
- Optional: `--output-file <path>` to specify output location (default: see Filename Generation in review-orchestration-docs.md)
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills

Invoke superpowers methodology skills; pass to agents via `skill_instructions.methodology`. If unavailable, proceed without.

## Step 2: Load Settings

Load from `.claude/code-review.local.md` in project root. If missing, use defaults.

Fields: `enabled` (true; false → stop with error), `output_dir` ("." ; prepend to output filename unless --output-file), `skip_agents` ([] ; exclude from review), `min_severity` ("suggestion" ; filter to issues at or above), `additional_test_patterns` ([] ; merge with defaults).

Markdown body → "Project-Specific Instructions" for all agents. `--prompt` appends. Precedence: CLI flags > settings file > defaults.

## Step 3: Context Discovery

### AI Instructions

Priority (higher overrides): (1) `CLAUDE.md` in reviewed file directories, (2) `CLAUDE.md` in repo root, (3) `.ai/AI-AGENT-INSTRUCTIONS.md`, (4) `.github/copilot-instructions.md`. Per file, resolve which apply (same directory or parent up to repo root).

**Discovery Output:**

```yaml
ai_instructions: [{path, applies_to}]
```

---

## Step 4: Input Validation and Content Gathering

### Input Validation

**Parse arguments:** File paths (optional, space-separated), `--output-file`, `--prompt`.

**Git repository check:** Verify via `git rev-parse --git-dir`. If not, stop: "Not a git repository."

**Discover documentation files:** If specific files provided, validate those. Otherwise discover:

Standard documentation: Root markdown (README.md, CHANGELOG.md, CLAUDE.md, CODE_OF_CONDUCT.md, CONTRIBUTING.md, LICENSE.md) + `docs/**/*.md`, `documentation/**/*.md`

AI Agent Instruction Files:
- `.ai/AI-AGENT-INSTRUCTIONS.md` (standard location)
- `.github/copilot-instructions.md`
- `.github/ISSUE_TEMPLATE/*.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `AI-AGENT-INSTRUCTIONS.md` (root - should be in `.ai/`)

**AI instruction file standardization tracking:**
- AI-AGENT-INSTRUCTIONS.md: `.ai/` = correct; root only = should move to `.ai/`; missing = should create
- CLAUDE.md header: check references `.ai/AI-AGENT-INSTRUCTIONS.md`
- copilot-instructions.md: `.github/` = exists; missing = should create

**Filter by scope:** Specific files provided → verify existence, review only those. No files → include all discovered; prioritize README > CLAUDE.md > CHANGELOG > docs/

If no files found, stop: "No documentation files found. Ensure your project has README.md, docs/, or other markdown documentation."

**Validation output:** Documentation files to review, AI instruction file standardization status, output file path, additional instructions.

### Content Gathering

Launch Sonnet agent to gather documentation content and code references.

**Current branch:** `git branch --show-current`

**Documentation files:** Read full content, classify type (README, API docs, tutorial, etc.)

**Code references:** Scan docs for verifiable code references (function/class names, file paths, imports, API endpoints, CLI commands). For referenced code files that exist, read relevant sections (signatures, definitions, exports) for accuracy verification.

**Project metadata:** Read `package.json`, `*.csproj`, `pyproject.toml`/`setup.py`, `VERSION` file, git tags for version verification.

**AI instruction file context (if reviewing AI instruction files):** Read `.ai/AI-AGENT-INSTRUCTIONS.md`, `CLAUDE.md`, `.github/copilot-instructions.md` if they exist. Check header consistency. Verify cross-references are valid.

**Output:** Current branch name. Per doc file: full content, type classification. Referenced code snippets for verification. Project metadata (versions, etc.). AI instruction file cross-reference status. Review summary.

---

## Step 5: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 6: Review Execution

**If depth == deep:**
Execute the **Deep Docs Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md`. Follow all CRITICAL WAIT barriers between phases.

**If depth == quick:**
Execute the **Quick Docs Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md`. Follow all CRITICAL WAIT barriers.

---

## Step 7: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md`.

---

## Steps 8-11: Validation, Aggregation, Output

Validate, aggregate, and generate output per `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-docs.md`. Write to file.

**Output config (deep):** Review Type: "Deep Documentation Review (up to 13 invocations)", Categories: All 6
**Output config (quick):** Review Type: "Quick Documentation Review (up to 7 invocations)", Categories: Accuracy, Clarity, Examples, Structure
Include AI instruction file standardization section. Report summary: total issues by severity, issues by category, AI instruction standardization status, path to report.
