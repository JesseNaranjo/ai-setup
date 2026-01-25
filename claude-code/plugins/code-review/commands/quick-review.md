---
name: quick-review
allowed-tools: Task, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(ls:*), Read, Write, Glob
description: Quick 7-agent code review with synthesis
argument-hint: "<file1> [file2...] [--output-file <path>] [--language nodejs|dotnet] [--prompt \"<instructions>\"] [--skills <skill1,skill2,...>]"
model: opus
---

Perform a fast 4-agent code review for the specified files, focusing on bugs, security, error handling, and test coverage. For files with uncommitted changes, review those changes. For files without uncommitted changes, review the entire file.

Parse arguments from `$ARGUMENTS`:
- Required: One or more file paths (space-separated)
- Optional: `--output-file <path>` to specify output location
- Optional: `--language nodejs|dotnet` to force language detection
- Optional: `--prompt "<instructions>"` to add instructions passed to all agents
- Optional: `--skills <skill1,skill2,...>` to embed skill methodologies in agent prompts (e.g., `--skills security-review,superpowers:brainstorming`)

---

## Step 1: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md` for the settings loading process.

Check for `.claude/code-review.local.md` in the project root:
- If found and `enabled: false`, stop with message
- Apply settings: `output_dir`, `skip_agents`, `min_severity`, `language`, `custom_rules`
- Read project-specific instructions from markdown body

---

## Step 2: Input Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/input-validation-files.md` for the validation process.

**Default output file**: `{output_dir}/.quick-review.md` (default: `.quick-review.md`)

---

## Step 3: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md` for:
- AI Agent Instructions file patterns
- Project type detection rules
- Related test file patterns per language

---

## Step 4: Content Gathering

See `${CLAUDE_PLUGIN_ROOT}/shared/content-gathering-files.md` for the content gathering process.

---

## Step 5: Skill Loading and Interpretation (if --skills provided)

See `${CLAUDE_PLUGIN_ROOT}/shared/skill-loading.md` for the complete skill loading process.

---

## Step 6: 4-Agent Quick Review

See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` for orchestration logic, the **Model Selection per Agent** table, and the **Agent Invocation Pattern** for the exact Task tool invocation format.

**Agent invocation uses Task tool with subagent_type** (e.g., `code-review:bug-detection-agent`), not file paths directly.

### Usage Tracking Initialization

Before launching agents, initialize usage tracking per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`:
1. Record `review_started_at` timestamp
2. Initialize 2 phases: "Review", "Synthesis"

Launch 4 agents in parallel with **quick** mode. See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` "Model Selection per Agent" table for model assignments.

**Critical**: When invoking each agent via Task tool, explicitly pass the `model` parameter per the authoritative table in `review-workflow.md`. The command's `model: opus` frontmatter applies to the orchestrator, not to individual agents.

**Agents**: Bug Detection (Opus), Security (Opus), Error Handling (Sonnet), Test Coverage (Sonnet)

Each agent receives:
- The current branch name
- A summary of what is being reviewed
- Which files have changes vs which are being reviewed in full
- The detected project type (for language-specific focus from `${CLAUDE_PLUGIN_ROOT}/languages/*.md`)
- The AI Agent Instructions files relevant to each file being reviewed
- For files with changes: both the diff AND full file content
- For files without changes: the full file content
- Related test files for context
- MODE parameter: **quick**
- Skill-derived instructions from Step 5 (tailored per agent from `--skills` argument)
- Additional instructions from `--prompt` argument (combined with project instructions from settings)

**Quick mode agents focus on**:
- Critical and Major severity issues only
- Most obvious and impactful problems
- Issues that would block a merge

**Usage Tracking - Review Phase:**
1. Record `phase_started_at` before launching agents
2. For each agent: record `agent_started_at` before Task call, `agent_ended_at` and `task_id` after Task returns
3. Record `phase_ended_at` when all agents complete

---

## Step 7: Cross-Agent Synthesis

After the 4-agent review, launch 3 synthesis agents in parallel.

See `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` for full agent definition.

See `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md` "Quick Review Synthesis (3 agents in parallel)" section for:
- The 3 category pairs and cross-cutting questions
- Synthesis agent input/output format

Launch all 3 synthesis agents in parallel, each with their respective category pairs.

**Example Synthesis Invocation:**

```
Task(
  subagent_type: "code-review:synthesis-agent",
  model: "sonnet",
  description: "Synthesis: Bugs + Error Handling",
  prompt: """
Analyze cross-cutting concerns between Bugs and Error Handling findings.

synthesis_input:
  category_a:
    name: "Bugs"
    findings:
      # Include ALL bug detection findings
      - title: "[Title from bug findings]"
        file: "[path]"
        line: [line]
        severity: "[severity]"
        description: "[description]"
        fix_type: "[diff|prompt]"
        fix_diff: |  # or fix_prompt
          [fix content]

  category_b:
    name: "Error Handling"
    findings:
      # Include ALL error handling findings
      - title: "[Title from error handling findings]"
        ...

  cross_cutting_question: "Do identified bugs have proper error handling in fix paths?"

  files_content:
    - path: "[file being reviewed]"
      diff: |
        [diff if has changes]
      full_content: |
        [full file content]

Return findings as cross_cutting_insights YAML list per synthesis-agent.md schema.
"""
)
```

**Usage Tracking - Synthesis:**
1. Record `phase_started_at` before launching synthesis agents
2. For each synthesis agent: record `agent_started_at` before Task call, `agent_ended_at` and `task_id` after Task returns
3. Record `phase_ended_at` when all agents complete
4. Record `review_ended_at` timestamp

**Output**: `cross_cutting_insights` list added to the issue pool before validation.

---

## Step 8: Validation (Streamlined)

Validate Critical and Major issues using batch validation by file. Minor issues and Suggestions are included without validation.

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for validation process.

---

## Step 9: Aggregation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for aggregation rules:
- Filter invalid issues
- Apply severity downgrades
- Deduplicate by file+line range
- Add consensus badges

---

## Step 10: Generate Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` and `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for formatting.

**REQUIRED: Generate Usage Summary FIRST (before any other output):**

The Usage Summary MUST appear at the very beginning of the output file, before the Code Review header. This section is MANDATORY - outputs missing this section are INCOMPLETE.

Generate the Usage Summary following the format in `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` "Usage Summary Section":
- Review Type: "Quick (7 invocations)"
- Phases: "Review", "Synthesis" (use quick review phase names per output-format.md)
- Include 4 review agents and 3 synthesis agents
- Use model assignments from `${CLAUDE_PLUGIN_ROOT}/shared/review-workflow.md`

Flag timing anomalies with `[!]` (too fast) or `[*]` (too slow) indicators per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`.

**IMPORTANT:** If the Usage Summary section is missing from the output, the review is INCOMPLETE and must be regenerated.

**Then generate Code Review output:**

**Review depth**: "Quick (4-agent + 3 synthesis)"

**Categories to include in summary table**: 4 only (Bugs, Security, Error Handling, Test Coverage)

**Footer note**: *For comprehensive review, run `/deep-review <files>` or `/deep-review-staged` for staged changes.*

---

## Step 11: Write Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-generation.md` for write process.

**Default output file**: `.quick-review.md`

---

## False Positives

See `${CLAUDE_PLUGIN_ROOT}/shared/false-positives.md` for issues that should NOT be flagged.

Quick review should be extra conservative - skip theoretical edge cases.

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Quick review is optimized for speed - use `/deep-review` for thorough analysis.
- Focus on blocking issues that must be fixed before merge.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not diff line numbers).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
- Create a todo list before starting.
