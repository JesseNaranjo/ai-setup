---
name: docs-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Bash(find:*), Read, Write, Glob, Grep
description: Use when reviewing project documentation for quality issues. Supports deep (up to 14 agents) and quick (up to 7 agents) depth. Omit prompt for auto-discovery.
argument-hint: "\"<review prompt>\" [--depth deep|quick] [--output-file <path>] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a documentation review. Depth controls the review pipeline: deep uses all 6 documentation agents (up to 14 invocations total) with thorough + gaps modes; quick uses 4 agents (up to 7 invocations) focusing on critical issues (accuracy, clarity, examples, structure). If no reviewable scope inferred from prompt, discover and review all documentation files in the project.

Parse `$ARGUMENTS`: extract structural flags (--depth, --output-file, --skills); remaining text = review prompt.
- Optional: `--depth deep|quick` (default: `deep`)
- Optional: `--output-file <path>` to specify output location (default: see Filename Generation in review-orchestration-docs.md)
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills

Invoke superpowers methodology skills; pass to agents via `skill_instructions.methodology`. If unavailable, proceed without.

## Step 2: Load Settings

Load from `.claude/code-review.local.md` in project root. If missing, use defaults.

Fields: `enabled` (true; false → stop with error), `output_dir` ("." ; prepend to output filename unless --output-file), `skip_agents` ([] ; exclude from review), `min_severity` ("suggestion" ; filter to issues at or above).

Markdown body → "Project-Specific Instructions" for all agents. Review prompt appended. Precedence: CLI flags > settings file > defaults.

## Step 3: Context Discovery

### AI Instructions

Priority (higher overrides): (1) `CLAUDE.md` in reviewed file directories, (2) `CLAUDE.md` in repo root, (3) `.ai/AI-AGENT-INSTRUCTIONS.md`, (4) `.github/copilot-instructions.md`. Per file, resolve which apply (same directory or parent up to repo root).

**Discovery Output:**

```yaml
ai_instructions: [{path, applies_to}]
```

---

## Step 4: Scope Inference and Content Gathering

### Scope Inference

Analyze the review prompt to determine review scope:

1. **File path scope**: Prompt contains file paths (paths ending in `.md`, `.txt`, `.rst`, or matching existing repo files via Glob) → specific file review
2. **Commit scope**: Prompt contains git references (SHAs, ranges, or keywords like "commit",
   "last N commits", "changes since <ref>") → review documentation files changed in those commits.
   Resolve references same as code-review. Filter changed files to documentation files only
   (*.md, *.txt, *.rst). If no documentation files changed, fall through to auto-discovery with
   warning: "No documentation files changed in specified commits. Reviewing all project documentation."
3. **Descriptive scope**: Prompt describes specific documentation (e.g., "check README accuracy", "review the API docs") → discover those files using Glob/Grep
4. **Auto-discovery scope**: Prompt provides general instructions or is empty → auto-discover all project documentation
5. **No docs found**: Auto-discovery finds zero docs → error: "No documentation files found. Ensure your project has README.md, docs/, or other markdown documentation."

### Input Validation

**Git repository check:** Verify via `git rev-parse --git-dir`. If not, stop: "Not a git repository."

**Discover documentation files:** If specific files determined from scope inference (file path or commit scope), validate those. Otherwise discover:

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

**Filter by scope:** Specific files determined → verify existence, review only those. Auto-discovery → include all discovered; prioritize README > CLAUDE.md > CHANGELOG > docs/

**Validation output:** Documentation files to review, AI instruction file standardization status, output file path.

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

**Output config (deep):** Review Type: "Deep Documentation Review (up to 14 invocations)", Categories: All 6
**Output config (quick):** Review Type: "Quick Documentation Review (up to 7 invocations)", Categories: Accuracy, Clarity, Examples, Structure
Include AI instruction file standardization section. Report summary: total issues by severity, issues by category, AI instruction standardization status, path to report.
