# Common Agent Instructions

This document contains shared instructions for all review agents. Agents reference this file instead of duplicating content.

## Contents

- [Standard Agent Input](#standard-agent-input)
- [Using skill_instructions (Full Version)](#using-skill_instructions-full-version)
- [Using skill_instructions (Methodology Only)](#using-skill_instructions-methodology-only)
- [Using Tiered Context](#using-tiered-context)
- [MODE Parameter (Common)](#mode-parameter-common)
- [False Positive Rules](#false-positive-rules)
- [Gaps Mode Behavior Template](#gaps-mode-behavior-template)
  - [Duplicate Detection](#duplicate-detection-common-to-all-gaps-agents)
  - [Constraints](#constraints-common-to-all-gaps-agents)
  - [Gaps-Supporting Agents](#gaps-supporting-agents)
- [Skill Instructions Apply to All Modes](#skill-instructions-apply-to-all-modes)
- [Pre-Existing Issue Detection](#pre-existing-issue-detection-for-stageddiff-reviews)
  - [Automatic Cross-File Analysis](#automatic-cross-file-analysis)
- [Output Schema](#output-schema)

## Standard Agent Input

All review agents receive the following inputs. Agents should NOT repeat this in their own documentation.

**Required inputs:**
- **Files to review**: Diffs and/or full content (see Using Tiered Context below)
- **Detected project type**: Node.js, .NET, or both
- **MODE parameter**: thorough, gaps, or quick (see MODE Parameter below)

**Optional inputs:**
- **skill_instructions**: Skill-derived focus areas and methodology (see below)
- **previous_findings**: Prior findings for gaps mode deduplication

**Available tools:** `["Read", "Grep", "Glob"]`

## Using skill_instructions (Full Version)

For agents that receive review-focused skills (architecture-agent, bug-detection-agent, compliance-agent, performance-agent, security-agent, technical-debt-agent):

When `skill_instructions` is present in the prompt, apply it as follows:

1. **focus_areas**: Prioritize checking these categories FIRST before standard checks. Structure findings around these areas where applicable.
2. **checklist**: For each checklist category, explicitly verify EVERY item. If an item is clean (no issues found), acknowledge it was checked.
3. **auto_validate**: Issues matching these pattern IDs should include `auto_validated: true` in output.
4. **false_positive_rules**: Apply these as ADDITIONAL false positive filters beyond this agent's standard rules.

For methodology skills (like `superpowers:brainstorming`):
1. **methodology.approach**: Adopt this mindset throughout analysis
2. **methodology.steps**: Follow these steps as part of your review process
3. **methodology.questions**: Consider these questions when evaluating each potential finding

When `skill_instructions` is absent, proceed with standard review process.

## Using skill_instructions (Methodology Only)

For agents that receive only methodology skills (api-contracts-agent, error-handling-agent, test-coverage-agent, synthesis-agent):

When `skill_instructions` is present, apply methodology skills as follows:

1. **methodology.approach**: Adopt this mindset throughout analysis
2. **methodology.steps**: Follow these steps as part of your review process
3. **methodology.questions**: Consider these questions when evaluating each potential finding

When `skill_instructions` is absent, proceed with standard review process.

## Using Tiered Context

When files include tier information (staged reviews):

**For `tier: "critical"` files:**
- Full content is provided - analyze thoroughly
- This is the primary review focus

**For `tier: "peripheral"` files:**
- Only a preview (first 50 lines) is provided
- Use the preview to understand file purpose
- If cross-file analysis discovers relevance, use Read tool to get full content

**Cross-File Discovery Pattern:**
```
Grep(pattern: "[relevant pattern]", path: "src/")
Read(file_path: "[discovered file]")  # Read if relevant to analysis
```

## MODE Parameter (Common)

All review agents accept a MODE parameter that controls review depth:

- **thorough**: Comprehensive review checking all issues in the agent's domain
- **gaps**: Focus on subtle issues that might be missed; receives prior findings context to skip duplicates
- **quick**: Fast pass on critical issues only (highest-impact findings)

See each agent file for mode-specific focus areas.

## False Positive Rules

See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` "False Positive Rules" section for the complete list of issues that should NOT be flagged.

See each agent file for category-specific false positive exclusions.

## Gaps Mode Behavior Template

When MODE=gaps, agents receive `previous_findings` from thorough mode to avoid duplicates.

### Duplicate Detection (Common to All Gaps Agents)

- Skip issues in same file within ±5 lines of prior findings
- Skip same issue type on same function/method
- For range findings (lines A-B): skip zone = [A-5, B+5]

See `${CLAUDE_PLUGIN_ROOT}/shared/agent-invocation-pattern.md` lines 80-86 for the complete `previous_findings` schema.

### Constraints (Common to All Gaps Agents)

- Only report Major or Critical severity (skip Minor/Suggestion)
- Maximum 5 new findings per agent
- Model: Always Sonnet (cost optimization)

### Gaps-Supporting Agents

Only these agents support gaps mode: bug-detection, compliance, performance, security, technical-debt.

See each agent file for category-specific focus areas (what subtle issues thorough mode misses).

## Skill Instructions Apply to All Modes

When `--skills` is provided to a review command, **ALL agents receive `skill_instructions` regardless of MODE**:

- **thorough**: Receives full skill focus areas, checklists, and methodology guidance
- **gaps**: Receives the **same** skill_instructions as thorough mode - skills inform gap detection priorities
- **quick**: Receives the same skill_instructions - skills inform the quick pass focus

This means gaps-mode agents should apply skill-specific focus areas and validation rules just as thoroughly as thorough-mode agents, but with their standard gaps-mode behavior (focusing on subtle issues, avoiding duplicates of previous_findings).

See `${CLAUDE_PLUGIN_ROOT}/shared/skill-handling.md` for how skill_instructions are generated and passed to agents.

## Pre-Existing Issue Detection (For Staged/Diff Reviews)

**CRITICAL**: When reviewing staged changes or diffs, agents must only flag issues in CHANGED lines.

**Context provided to agents:**
1. Diff content with line markers (lines starting with `+` are additions)
2. Surrounding unchanged lines (for understanding context only)
3. Full file content (for reference only, not for flagging issues)

**Rules for what to flag:**
- ✅ Issue is in a line starting with `+` in the diff (newly added code)
- ✅ Change INTRODUCES the issue (e.g., removes null check that protected existing code)
- ✅ Change WORSENS an existing issue (e.g., increases scope of vulnerability)

**Do NOT flag:**
- ❌ Issues in unchanged code (lines without `+` prefix)
- ❌ Pre-existing problems not made worse by the change
- ❌ Style issues in untouched code nearby
- ❌ Issues visible in "full file" context but not in the diff

**Example:**
```diff
  function getUser(id) {
+   const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);  // FLAG: SQL injection
    if (existingBuggyCode) {  // DO NOT FLAG: pre-existing, not in diff
      return null;
    }
+   return user;
  }
```

### Automatic Cross-File Analysis

Agents automatically perform cross-file analysis when the reviewed code suggests cross-cutting concerns. This is triggered by:

- **Import/export statements** → Check for consumers, circular dependencies
- **Class/interface definitions** → Check implementations, inheritance chains
- **API contracts** → Check for breaking changes affecting consumers
- **Shared types/utilities** → Check all usages for consistency

When triggered, agents use Grep and Glob tools to discover related files, then Read to analyze relationships. This enables detection of:
- Unused exports
- Circular dependencies
- Broken call chains (signature changed, callers not updated)
- Interface/implementation mismatches

See `${CLAUDE_PLUGIN_ROOT}/agents/architecture-agent.md` for detailed cross-file analysis triggers and process.

## Output Schema

Use the YAML schema shown in your agent's examples. Each issue requires these base fields:
- `title`: Brief description
- `file`: File path relative to repo root
- `line`: Primary line number
- `range`: "start-end" for multi-line, null for single-line
- `category`: Agent's category name
- `severity`: Critical, Major, Minor, or Suggestion
- `description`: Detailed explanation
- `fix_type`: "diff" or "prompt"
- `fix_diff` or `fix_prompt`: The suggested fix

See `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` for severity classification rules.

See `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for the authoritative output schema reference used during output generation.
