# Common Agent Instructions

This document contains shared instructions for all review agents. Agents reference this file instead of duplicating content.

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

For agents that receive review-focused skills (security-agent, bug-detection-agent, performance-agent, compliance-agent):

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

For agents that receive only methodology skills (architecture-agent, api-contracts-agent, error-handling-agent, test-coverage-agent, synthesis-agent):

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

## Universal False Positive Rules

ALL agents MUST NOT flag:
- Pre-existing issues not introduced in the changes being reviewed
- Code with explicit ignore comments (lint-disable, eslint-ignore, etc.)
- Theoretical issues that require very specific or unrealistic conditions to manifest
- Issues that the type system, linter, or compiler will catch
- Issues in test code or development-only paths unless specifically reviewing tests
- Issues already in previous_findings (gaps mode only)

See each agent file for category-specific false positive rules.

## Gaps Mode Behavior (Common)

When MODE=gaps, all agents follow this pattern:

1. **Check previous_findings**: For each potential issue, check if it matches a prior finding (same file + overlapping line range). Skip if already flagged.
2. **Apply skip zone**: See `${CLAUDE_PLUGIN_ROOT}/shared/gaps-mode-rules.md` for skip zone calculation (±5 lines)
3. **Focus on subtle issues**: Prioritize edge cases, second-order effects, and issues that complement (don't duplicate) thorough mode findings
4. **Complement, don't repeat**: Find issues in code paths not covered by prior findings

See each agent file for category-specific gaps mode focus areas.

## Output Schema

All agents output YAML following `${CLAUDE_PLUGIN_ROOT}/shared/output-schema-base.md`.

Each issue requires these base fields:
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
