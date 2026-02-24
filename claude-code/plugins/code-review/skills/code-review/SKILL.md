---
name: code-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob, Grep
description: Code review with configurable depth (deep: up to 19 agent invocations, quick: up to 7). Describe what to review in the prompt.
argument-hint: "\"<review prompt>\" [--depth deep|quick] [--output-file <path>] [--language dotnet|nodejs|react] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a code review. Depth controls the review pipeline: deep uses all 9 agents (up to 19 invocations total) with thorough + gaps modes; quick uses 4 agents (up to 7 invocations) focusing on bugs, security, error handling, and test coverage. For file reviews with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

Parse `$ARGUMENTS`: extract structural flags (--depth, --output-file, --language, --skills); remaining text = review prompt.
- Optional: `--depth deep|quick` (default: `deep`)
- Optional: `--output-file <path>` to specify output location (default: see Filename Generation in review-orchestration-code.md)
- Optional: `--language dotnet|nodejs|react` to force language detection
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts

---

## Step 1: Invoke Methodology Skills

Invoke superpowers methodology skills; pass to agents via `skill_instructions.methodology`. If unavailable, proceed without.

## Step 2: Load Settings

Load from `.claude/code-review.local.md` in project root. If missing, use defaults.

Fields: `enabled` (true; false → stop with error), `output_dir` ("." ; prepend to output filename unless --output-file), `skip_agents` ([] ; exclude from review), `min_severity` ("suggestion" ; filter to issues at or above), `language` ("" ; empty = auto-detect), `additional_test_patterns` ([] ; merge with defaults).

Markdown body → "Project-Specific Instructions" for all agents. Review prompt appended. Precedence: CLI flags > settings file > defaults.

## Step 3: Context Discovery

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

## Step 4: Scope Inference and Content Gathering

### Scope Inference

Analyze the review prompt to determine review scope:

1. **Staged scope**: Prompt mentions "staged" or "staged changes" → staged review flow
2. **Commit scope**: Prompt contains git references (SHAs, ranges like `abc..def` or `HEAD~3..HEAD`,
   or keywords like "commit", "last N commits", "changes since/between <ref>") → commit review flow.
   Resolve references: single SHA = review that commit (`SHA~1..SHA`); range = use as-is;
   "last N commits" = `HEAD~N..HEAD`; branch name like "since main" = `main..HEAD`.
3. **File path scope**: Prompt contains file paths (paths ending in known extensions like `.ts`, `.js`, `.tsx`, `.jsx`, `.cs`, `.py`, `.md`, `.json`, or matching existing repo files via Glob) → specific file review
4. **Descriptive scope**: Prompt describes a module, feature, or code area → use Glob/Grep to discover relevant files. Validate discovered files exist and are non-empty. Fail if no files match the description.
5. **No scope**: Empty prompt or no inferrable scope → error: "Cannot infer what to review. Specify files, 'staged changes', commits, or describe the code area to review."

### Content Gathering

**If staged or commit scope (diff-based):**

Verify git repository (stop: "Not a git repository."). For staged: verify staged changes exist (stop: "No staged changes to review."). For commit: resolve commit references — validate SHAs exist via `git rev-parse`, resolve branch names, compute range. Stop if invalid: "Invalid commit reference: <ref>."

Launch Sonnet agent to gather:

- **Diff:** Full diff with line markers. Staged: `git diff --staged`. Commit: `git diff <base>..<head>`
- **Current branch:** Branch name
- **File content (tiered):** Changed files (has_changes=true) → full content (critical). Unchanged import/context files (has_changes=false) → first 50 lines preview (peripheral)
- **Related test files:** From Context Discovery (see `languages/nodejs.md`, `languages/dotnet.md`). Always critical tier
- **Summary:** Files modified count, change description, project types, test files, tier classification. Commit scope also includes: commit range

**Output:** Branch name. Per file: path, has_changes, tier. Critical: diff + full_content. Peripheral: preview (50 lines), line_count, full_content_available: true. Related tests (critical). Review summary. Commit scope also includes: commit range.

**Pre-Existing Issue Detection** — include in each agent's `additional_instructions`:

**CRITICAL**: Only flag issues in CHANGED lines.

**Context to agents:** (1) Diff with line markers (`+` = additions), (2) surrounding unchanged lines (context only), (3) full file content (reference only).

**Flag:** Lines starting with `+` in diff. Changes that INTRODUCE issues (e.g., removes null check). Changes that WORSEN existing issues (e.g., increases vulnerability scope).

**Do NOT flag:** Unchanged code (no `+` prefix). Pre-existing problems not worsened. Style in untouched code. Issues in "full file" context but not in diff.

**Tiered Context** — inject into `additional_instructions` for staged and commit scope reviews:

**critical:** Full content provided — analyze thoroughly.
**peripheral:** Preview only (50 lines). If cross-file analysis discovers relevance, use Read tool for full content.

**If file path or descriptive scope:**

For descriptive scope: use Glob and Grep to discover files matching the described module, feature, or code area.

Validate: git repo presence (fail if not), file existence (exclude missing), change status (uncommitted vs none). Fail if no valid files remain. Split files by has_changes / no changes.

Launch Sonnet agent to gather content:
- Files with uncommitted changes: diff + full content. Files without changes: full content only.
- Related test files from Context Discovery step (patterns per `languages/nodejs.md` and `languages/dotnet.md`).
- Output: current branch name, per-file diff/content/has_changes, related test contents, review summary (files with changes vs entirety, project type(s), test files found).

---

## Step 5: Skill Loading

Skip if `--skills` not provided. Otherwise see `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md`.

---

## Step 6: Review Execution

**If depth == deep:**
Execute the **Deep Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. Follow all CRITICAL WAIT barriers between phases.

**If depth == quick:**
Execute the **Quick Code Review Sequence** from `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`. Follow all CRITICAL WAIT barriers.

**If staged or commit scope detected:** Agents receive diff and file content per tier classification above (Step 4).

---

## Step 7: Cross-Agent Synthesis

Execute the **Synthesis** step from the applicable Review Sequence in `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md`.

---

## Steps 8-11: Validation, Aggregation, Output

Validate, aggregate, and generate output per `${CLAUDE_PLUGIN_ROOT}/shared/review-validation-code.md`. Write to file.

**Output config (deep):** Review Type: "Deep (up to 19 invocations)", Categories: All 9
**Output config (quick):** Review Type: "Quick (up to 7 invocations)", Categories: 4 only
**Note (quick only):** Quick review should be extra conservative - skip theoretical edge cases.
