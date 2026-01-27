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

## Gaps Mode Behavior

When MODE=gaps, agents that support gaps mode have inline rules in their respective files. Only these 5 agents support gaps mode: bug-detection, compliance, performance, security, technical-debt.

See each agent file for category-specific gaps mode rules and focus areas.

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
