# Common Command Steps

This document contains shared workflow steps referenced by all review commands (deep-code-review, deep-code-review-staged, quick-code-review, quick-code-review-staged).

## Step Numbering Scheme

Commands use a consistent workflow:
- **Step 0**: Defined here (optional methodology skills)
- **Steps 1, 3, 5**: Defined here (shared across all commands)
- **Steps 2, 4, 6-7**: Defined inline in each command (command-specific)
- **Steps 8-11**: Defined here (shared across all commands)

**Related File**: `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` provides phase definitions and model selection for review execution.

---

## Step 0: Invoke Methodology Skills (Optional)

Before beginning the review workflow, the orchestrator MAY invoke external methodology skills if available. These are optional enhancements - the review workflow functions correctly without them.

**Optional External Skills (from `superpowers` plugin):**
- `superpowers:using-superpowers` - Skill usage methodology
- `superpowers:brainstorming` - Explore interpretations before concluding
- `superpowers:systematic-debugging` - Systematic issue detection
- `superpowers:verification-before-completion` - Verify findings before reporting

**Usage:** If loaded successfully, apply methodologies from these skills throughout the review process. These inform HOW to analyze, not WHAT to look for. If skills are not found, proceed with the standard review workflow.

**Note:** These skills are part of the optional `superpowers` plugin. Additional superpowers skills (writing-plans, executing-plans, requesting-code-review, dispatching-parallel-agents, subagent-driven-development) apply when planning or coordinating work, not during review execution.

---

## Step 1: Load Settings

See `${CLAUDE_PLUGIN_ROOT}/shared/settings-loader.md` for the settings loading process.

Check for `.claude/code-review.local.md` in the project root:
- If found and `enabled: false`, stop with message
- Apply settings: `output_dir`, `skip_agents`, `min_severity`, `language`
- Read project-specific instructions from markdown body

---

## Step 3: Context Discovery

See `${CLAUDE_PLUGIN_ROOT}/shared/context-discovery.md` for:
- AI Agent Instructions file patterns
- Project type detection rules
- Related test file patterns per language

---

## Step 5: Skill Loading and Interpretation

**Skip this step entirely if `--skills` argument was not provided.**

If `--skills` is provided:
1. See `${CLAUDE_PLUGIN_ROOT}/shared/skill-resolver.md` for skill resolution
2. See `${CLAUDE_PLUGIN_ROOT}/shared/skill-orchestration.md` for orchestration adjustments

---

## Step 8: Validation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for complete validation process including:
- Auto-validation patterns
- Batch grouping by file
- Validator model assignment

---

## Step 9: Aggregation

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` for aggregation rules:
- Filter invalid issues
- Apply severity downgrades
- Deduplicate by file+line range
- Add consensus badges

---

## Step 10: Output Generation

See `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for formatting and generation process.

**REQUIRED: Generate Usage Summary FIRST (before any other output):**

The Usage Summary MUST appear at the very beginning of the output file, before the Code Review header. This section is MANDATORY - outputs missing this section are INCOMPLETE.

Generate the Usage Summary following the format in `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` "Usage Summary Section":
- Use model assignments from `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md`
- Flag timing anomalies with `[!]` (too fast) or `[*]` (too slow) indicators per `${CLAUDE_PLUGIN_ROOT}/shared/usage-tracking.md`

**IMPORTANT:** If the Usage Summary section is missing from the output, the review is INCOMPLETE and must be regenerated.

---

## Step 11: Write Output

See `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for write process.

---

## Cross-Agent Synthesis

**CRITICAL: For Deep Review, Synthesis is Phase 3 and requires strict sequential execution:**

1. Phase 1 (Thorough Review) - agents run in parallel
2. **WAIT** - All Phase 1 agents must complete
3. Phase 2 (Gaps Review) - agents run in parallel
4. **WAIT** - All Phase 2 agents must complete
5. Phase 3 (Synthesis) - agents run in parallel
6. Continue to validation

**Do NOT launch Synthesis until Phase 2 is FULLY COMPLETE.**

Synthesis receives findings from ALL prior phases (Phase 1 + Phase 2) to detect cross-cutting concerns.

**For Quick Review:** Launch synthesis agents after the Review phase completes. Do NOT launch Synthesis until Review phase is FULLY COMPLETE.

See `${CLAUDE_PLUGIN_ROOT}/shared/synthesis-invocation-pattern.md` for:
- Invocation format (required YAML structure)
- Parallel invocation pattern diagram
- Example Task tool invocation

See `${CLAUDE_PLUGIN_ROOT}/agents/synthesis-agent.md` for:
- Full agent definition and analysis logic
- Cross-cutting pairs and questions
- Output schema and guidelines

See `${CLAUDE_PLUGIN_ROOT}/shared/orchestration-sequence.md` for:
- Category pairs and cross-cutting questions (Deep Review vs Quick Review)
- Model selection (Sonnet)

**Output**: `cross_cutting_insights` list added to the issue pool before validation.

---

## False Positives

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section for issues that should NOT be flagged.

---

## Notes

- Use git CLI to interact with the repository. Do not use GitHub CLI.
- Create a todo list before starting.
- Cite each issue with file path and line numbers (e.g., `src/utils.ts:42-48`).
- When referencing AI Agent Instructions rules, quote the exact rule being violated.
- File paths should be relative to the repository root.
- Line numbers should reference the lines in the actual file (not diff line numbers for file reviews, working copy lines for staged reviews).
- When reviewing full files (no changes), be more lenient - focus on clear bugs, not style issues.
