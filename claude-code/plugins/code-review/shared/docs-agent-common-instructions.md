# Common Documentation Agent Instructions

This document contains shared instructions for all documentation review agents. Agents reference this file instead of duplicating content.

## Standard Agent Input

All documentation review agents receive the following inputs. Agents should NOT repeat this in their own documentation.

**Required inputs:**
- **Documentation files**: Full content of markdown files being reviewed
- **Related code files**: Code snippets referenced by documentation (for accuracy verification)
- **MODE parameter**: thorough, gaps, or quick (see MODE Parameter below)

**Optional inputs:**
- **skill_instructions**: Skill-derived focus areas and methodology
- **previous_findings**: Prior findings for gaps mode deduplication

**Available tools:** `["Read", "Grep", "Glob"]`

## Using skill_instructions

When `skill_instructions` is present in the prompt, apply it as follows:

1. **focus_areas**: Prioritize checking these documentation aspects FIRST before standard checks
2. **checklist**: For each checklist item, explicitly verify. If clean, acknowledge it was checked
3. **auto_validate**: Issues matching these pattern IDs include `auto_validated: true` in output
4. **false_positive_rules**: Apply as ADDITIONAL false positive filters

When `skill_instructions` is absent, proceed with standard review process.

## MODE Parameter (Common)

All documentation review agents accept a MODE parameter that controls review depth:

- **thorough**: Comprehensive review checking all issues in the agent's domain
- **gaps**: Focus on subtle issues that might be missed; receives prior findings context to skip duplicates
- **quick**: Fast pass on critical issues only (highest-impact findings)

See each agent file for mode-specific focus areas.

## Documentation-Specific Context

### Code Reference Verification

When documentation references code (functions, classes, APIs, commands):
1. Use Grep/Glob to locate the referenced code
2. Verify the documented behavior matches implementation
3. Check parameter names, types, return values
4. Verify code examples are syntactically correct and runnable

### Cross-Document Consistency

Documentation often spans multiple files. Check for:
- Consistent terminology across README, API docs, tutorials
- Version numbers match across all documentation
- Links between documents are valid and bidirectional where appropriate

## False Positive Guidelines

**Universal documentation false positives - do NOT flag:**
- Intentionally simplified examples (marked as "simplified" or "basic")
- Placeholder values in examples (`your-api-key`, `example.com`)
- Documentation for deprecated features that includes deprecation notices
- TODOs in draft documentation clearly marked as draft
- Style preferences that don't affect comprehension

See each agent file for category-specific false positive exclusions.

## Gaps Mode Behavior

When MODE=gaps, agents that support gaps mode have inline rules in their respective files. Only these 3 agents support gaps mode: accuracy, completeness, consistency.

**Common gaps mode rules:**
- Skip issues in same file within ±3 lines of prior findings
- Skip same issue type on same section/heading
- Focus on subtle issues thorough mode typically misses
- Only report Major or Critical severity
- Maximum 5 new findings per agent

## Output Schema

Use the YAML schema shown in your agent's examples. Each issue requires these base fields:
- `title`: Brief description
- `file`: File path relative to repo root
- `line`: Primary line number (or heading/section reference)
- `range`: "start-end" for multi-line, null for single-line
- `category`: Agent's category name
- `severity`: Critical, Major, Minor, or Suggestion
- `description`: Detailed explanation
- `fix_type`: "diff" or "prompt"
- `fix_diff` or `fix_prompt`: The suggested fix

### Severity Guidelines for Documentation

- **Critical**: Factually incorrect information that could cause errors, security issues, or significant user confusion
- **Major**: Missing essential information, broken examples, significant clarity issues
- **Minor**: Style inconsistencies, minor omissions, suboptimal organization
- **Suggestion**: Improvements that would enhance but aren't strictly necessary

See `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` for general severity classification rules.
