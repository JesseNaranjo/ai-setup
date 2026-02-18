---
name: code-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Code review for files or staged changes with configurable depth (deep: up to 19 agent invocations, quick: up to 7)
argument-hint: "[<file1> [file2...] | --staged] [--depth deep|quick] [--output-file <path>] [--language dotnet|nodejs|react] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a code review for specified files or staged git changes. Depth controls the review pipeline: deep uses all 9 agents (up to 19 invocations total) with thorough + gaps modes; quick uses 4 agents (up to 7 invocations) focusing on bugs, security, error handling, and test coverage. For file reviews with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

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

Load from `.claude/code-review.local.md` in project root. If missing, use defaults.

Fields: `enabled` (true; false → stop with error), `output_dir` ("." ; prepend to output filename unless --output-file), `skip_agents` ([] ; exclude from review), `min_severity` ("suggestion" ; filter to issues at or above), `language` ("" ; empty = auto-detect), `additional_test_patterns` ([] ; merge with defaults).

Markdown body → "Project-Specific Instructions" for all agents. `--prompt` appends. Precedence: CLI flags > settings file > defaults.

## Step 4: Context Discovery

### AI Instructions

Priority (higher overrides): (1) `CLAUDE.md` in reviewed file directories, (2) `CLAUDE.md` in repo root, (3) `.ai/AI-AGENT-INSTRUCTIONS.md`, (4) `.github/copilot-instructions.md`. Per file, resolve which apply (same directory or parent up to repo root).

### Language Detection

Detect: Node.js/TypeScript (`package.json`), .NET/C# (`*.csproj`/`*.sln`/`*.slnx`). Override: `--language nodejs|dotnet|react` for ALL files. Monorepo: per-file detection, group by language.

**Framework:** Node.js files: check nearest `package.json` for `react`/`react-dom` in dependencies. `--language react` = Node.js + React checks.

**LSP:** Check for `typescript-lsp` (Node.js), `csharp-lsp`/OmniSharp (.NET). If available: read `${CLAUDE_PLUGIN_ROOT}/shared/references/lsp-integration.md`, add `lsp_available` to discovery. If none: skip.

**Test Files:** Per detected language from `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md` and `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md`. Merge `additional_test_patterns` from settings.

**Discovery Output:**

```yaml
ai_instructions: [{path, applies_to}]
detected_languages: {language: [files]}
detected_frameworks: {framework: [files]}
test_files: {source: [tests]}
language_override: string | null
lsp_available: [types] | null
```

Only load `languages/*.md` for detected languages. Unknown: warn, skip.

---

## Steps 3 & 5: Input Validation and Content Gathering

**If `--staged`:**

Verify git repository (stop: "Not a git repository.") and staged changes exist (stop: "No staged changes to review."). Parse `--output-file` (command provides default) and `--language` (default: auto-detect per file).

Launch Sonnet agent to gather:

- **Staged diff:** Full diff with line markers
- **Current branch:** Branch name
- **File content (tiered):** Changed files (has_changes=true) → full content (critical). Unchanged import/context files (has_changes=false) → first 50 lines preview (peripheral)
- **Related test files:** From Context Discovery (see `languages/nodejs.md`, `languages/dotnet.md`). Always critical tier
- **Summary:** Files modified count, change description, project types, test files, tier classification

**Output:** Branch name. Per file: path, has_changes, tier. Critical: diff + full_content. Peripheral: preview (50 lines), line_count, full_content_available: true. Related tests (critical). Review summary.

**Pre-Existing Issue Detection** — include in each agent's `additional_instructions`:

**CRITICAL**: Only flag issues in CHANGED lines.

**Context to agents:** (1) Diff with line markers (`+` = additions), (2) surrounding unchanged lines (context only), (3) full file content (reference only).

**Flag:** Lines starting with `+` in diff. Changes that INTRODUCE issues (e.g., removes null check). Changes that WORSEN existing issues (e.g., increases vulnerability scope).

**Do NOT flag:** Unchanged code (no `+` prefix). Pre-existing problems not worsened. Style in untouched code. Issues in "full file" context but not in diff.

**Tiered Context** — inject into `additional_instructions` for staged reviews:

**critical:** Full content provided — analyze thoroughly.
**peripheral:** Preview only (50 lines). If cross-file analysis discovers relevance, use Read tool for full content.

**If file paths provided:**

Parse arguments: file paths (all non-flag arguments, space-separated), `--output-file` path, `--language` override (`nodejs`, `react`, `dotnet`).

Validate: git repo presence (fail if not), file existence (exclude missing), change status (uncommitted vs none). Fail if no valid files remain. Split files by has_changes / no changes.

Launch Sonnet agent to gather content:
- Files with uncommitted changes: diff + full content. Files without changes: full content only.
- Related test files from Context Discovery step (patterns per `languages/nodejs.md` and `languages/dotnet.md`).
- Output: current branch name, per-file diff/content/has_changes, related test contents, review summary (files with changes vs entirety, project type(s), test files found).

---

## Step 6: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 7: Review Execution

**If depth == deep:**
Execute the **Deep Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. Follow all CRITICAL WAIT barriers between phases.

**If depth == quick:**
Execute the **Quick Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. Follow all CRITICAL WAIT barriers.

**If `--staged`:** Agents receive staged diff and file content per tier classification above (Steps 3 & 5).

---

## Step 8: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## Steps 9-12: Validation, Aggregation, Output

Validate, aggregate, and generate output per `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`. Write to file.

**Output config (deep):** Review Type: "Deep (up to 19 invocations)", Categories: All 9
**Output config (quick):** Review Type: "Quick (up to 7 invocations)", Categories: 4 only
**Note (quick only):** Quick review should be extra conservative - skip theoretical edge cases.
