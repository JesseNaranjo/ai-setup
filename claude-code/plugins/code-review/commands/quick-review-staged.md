---
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Quick code review with 7 agent invocations (4 review + 3 synthesis)
argument-hint: "[--output-file <path>] [--language nodejs|dotnet]"
model: opus
---

Perform a fast 4-agent code review for staged git changes, focusing on bugs, security, error handling, and test coverage.

Parse arguments from `$ARGUMENTS`:
- Optional: `--output-file <path>` to specify output location
- Optional: `--language nodejs|dotnet` to force language detection

---

## Step 0: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md` for the settings loading process.

Check for `.claude/code-review.local.md` in the project root:
- If found and `enabled: false`, stop with message
- Apply settings: `output_dir`, `skip_agents`, `min_severity`, `language`, `custom_rules`
- Read project-specific instructions from markdown body

---

## Step 1: Staged Changes Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-staged.md` for the validation process.

**Default output file**: `{output_dir}/.quick-review-staged.md` (default: `.quick-review-staged.md`)

---

## Step 2: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md` for:
- AI Agent Instructions file patterns
- Project type detection rules
- Related test file patterns per language

---

## Step 3: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-staged.md` for the content gathering process.

---

## Step 4: 4-Agent Quick Review

Launch 4 agents in parallel with **quick** mode:

| Agent | Model | MODE | Focus |
|-------|-------|------|-------|
| `${CLAUDE_PLUGIN_ROOT}/agents/bug-detection-agent.md` | Opus | quick | Obvious bugs, null refs, clear logical errors |
| `${CLAUDE_PLUGIN_ROOT}/agents/security-agent.md` | Opus | quick | Critical security issues, injection, hardcoded secrets |
| `${CLAUDE_PLUGIN_ROOT}/agents/error-handling-agent.md` | Sonnet | quick | Missing error handling, swallowed exceptions |
| `${CLAUDE_PLUGIN_ROOT}/agents/test-coverage-agent.md` | Sonnet | quick | Critical paths without tests |

Each agent receives:
- The current branch name
- A summary of the staged changes
- The staged diff
- Full file content for broader context
- The detected project type (for language-specific focus from `${CLAUDE_PLUGIN_ROOT}/languages/*.md`)
- The AI Agent Instructions files relevant to each staged file
- Related test files for context
- MODE parameter: **quick**

**Quick mode agents focus on**:
- Critical and Major severity issues only
- Most obvious and impactful problems
- Issues that would block a merge

---

## Synthesis Phase: Cross-Agent Synthesis

After the 4-agent review, launch 3 synthesis agents in parallel.

See `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` for full agent definition.

| Synthesis Agent | Input Categories | Cross-Cutting Question |
|-----------------|-----------------|------------------------|
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Bugs + Error Handling | "Do identified bugs have proper error handling in fix paths?" |
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Security + Bugs | "Do security issues introduce or relate to bugs?" |
| `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` | Bugs + Test Coverage | "Are identified bugs covered by tests?" |

Launch all 3 synthesis agents in parallel, each with their respective category pairs.

**Output**: `cross_cutting_insights` list added to the issue pool before validation.

---

## Step 5: Validation (Streamlined)

Validate Critical and Major issues using batch validation by file. Minor issues and Suggestions are included without validation.

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for validation process.

---

## Step 6: Aggregation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for aggregation rules:
- Filter invalid issues
- Apply severity downgrades
- Deduplicate by file+line range
- Add consensus badges

---

## Step 7: Generate Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` and `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for formatting.

**Review depth**: "Quick (4-agent + 3 synthesis)"

**Categories to include in summary table**: 4 only (Bugs, Security, Error Handling, Test Coverage)

**Footer note**: *For comprehensive review (compliance, performance, architecture, API), run `/deep-review-staged`*

---

## Step 8: Write Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` for write process.

**Default output file**: `.quick-review-staged.md`

---

## False Positives

See `${CLAUDE_PLUGIN_ROOT}/shared/false-positives.md` for issues that should NOT be flagged.

Quick review should be extra conservative - skip theoretical edge cases.

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Quick review is optimized for speed - use `/deep-review-staged` for thorough analysis.
- Focus on blocking issues that must be fixed before merge.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the working copy (not diff line numbers).
