# Common Agent Instructions

This document contains shared instructions for all review agents. Agents reference this file instead of duplicating content.

## Contents

- [Standard Agent Input](#standard-agent-input)
- [Using skill_instructions](#using-skill_instructions)
- [Using Tiered Context](#using-tiered-context)
- [MODE Parameter (Common)](#mode-parameter-common)
- [Language-Specific Checks](#language-specific-checks)
- [False Positive Rules](#false-positive-rules)
  - [Do NOT Flag](#do-not-flag)
  - [Deep vs Quick Review Differences](#deep-vs-quick-review-differences)
  - [Category-Specific False Positive Rules](#category-specific-false-positive-rules)
- [Gaps Mode Behavior Template](#gaps-mode-behavior-template)
- [Pre-Existing Issue Detection](#pre-existing-issue-detection-for-stageddiff-reviews)
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

## Using skill_instructions

When `skill_instructions` is present in the prompt, apply it as follows:

1. **focus_areas**: Prioritize checking these categories FIRST before standard checks. Structure findings around these areas where applicable.
2. **checklist**: For each checklist category, explicitly verify EVERY item. If an item is clean (no issues found), acknowledge it was checked.
3. **auto_validate**: Issues matching these pattern IDs should include `auto_validated: true` in output.
4. **false_positive_rules**: Apply these as ADDITIONAL false positive filters beyond this agent's standard rules.

For methodology skills (like `superpowers:brainstorming`):
1. **methodology.approach**: Adopt this mindset throughout analysis
2. **methodology.steps**: Follow these steps as part of your review process
3. **methodology.questions**: Consider these questions when evaluating each potential finding

**Note:** Agents without a primary review skill (api-contracts-agent, error-handling-agent, synthesis-code-agent, synthesis-docs-agent, test-coverage-agent) receive only the methodology section.

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

## MODE Parameter (Common)

All review agents accept a MODE parameter that controls review depth:

- **thorough**: Comprehensive review checking all issues in the agent's domain
- **gaps**: Focus on subtle issues that might be missed; receives prior findings context to skip duplicates
- **quick**: Fast pass on critical issues only (highest-impact findings)

See each agent file for mode-specific focus areas.

## Language-Specific Checks

When the detected project type includes a specific language, check the corresponding language file for category-specific patterns:

- **Node.js/TypeScript:** `${CLAUDE_PLUGIN_ROOT}/languages/nodejs.md#[your-category]`
- **.NET/C#:** `${CLAUDE_PLUGIN_ROOT}/languages/dotnet.md#[your-category]`
- **React:** `${CLAUDE_PLUGIN_ROOT}/languages/react.md#[your-category]` (extends Node.js)

## False Positive Rules

Issues that should NOT be flagged by review agents.

### Do NOT Flag

#### Correct Code

- Code that handles edge cases in non-obvious but valid ways
- Intentional patterns that look unusual but serve a purpose
- Something that appears to be a bug but is actually correct behavior

#### Linter Territory

- Formatting issues covered by prettier/eslint/other formatters
- Import ordering or unused import warnings
- Issues that a linter will catch (do not run the linter to verify)

#### Pedantic Concerns

- Minor formatting inconsistencies
- Nitpicks that a senior engineer would not flag
- Style preferences that don't affect functionality

#### Pre-existing Issues

- Issues in files with changes that existed before these changes
- Issues in the codebase that are not related to the current changes
- Pre-existing code patterns that weren't modified

#### Scope Limitations

- General code quality concerns unless explicitly required in AI Agent Instructions
- Issues in test code or development-only paths unless specifically reviewing tests
- Suggestions for features or patterns not in scope
- Theoretical edge cases that are extremely unlikely in practice

#### Silenced Issues

- Code with lint-disable comments for specific rules
- Intentionally suppressed warnings with documented reasons
- Issues mentioned in AI Agent Instructions but explicitly silenced in code

### Deep vs Quick Review Differences

**Deep review** can flag more issues but should still avoid pre-existing issues, silenced issues, and pure style preferences.

**Quick review** is optimized for speed: focus only on blocking issues, ignore minor style concerns, skip theoretical edge cases.

### Category-Specific False Positive Rules

- **Code reviews:** See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-code.md` "Category-Specific False Positive Rules > [Your Category]"
- **Docs reviews:** See `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules-docs.md` "Category-Specific False Positive Rules > [Your Category]"

## Gaps Mode Behavior Template

When MODE=gaps, agents receive `previous_findings` from thorough mode to avoid duplicates.

### Duplicate Detection (Common to All Gaps Agents)

- Skip issues in same file within ±5 lines of prior findings
- Skip same issue type on same function/method
- For range findings (lines A-B): skip zone = [A-5, B+5]

See `${CLAUDE_PLUGIN_ROOT}/shared/invocation-patterns.md` lines 80-86 for the complete `previous_findings` schema.

### Constraints (Common to All Gaps Agents)

- Only report Major or Critical severity (skip Minor/Suggestion)
- Maximum 5 new findings per agent
- Model: Always Sonnet (cost optimization)

### Gaps-Supporting Agents

**Code review:** bug-detection, compliance, performance, security, technical-debt.
**Documentation review:** accuracy, completeness, consistency.

See each agent file for category-specific focus areas (what subtle issues thorough mode misses).

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

Agents perform cross-file analysis automatically using tools when code suggests cross-cutting concerns (imports/exports, class/interface definitions, API contracts, shared types). See `${CLAUDE_PLUGIN_ROOT}/agents/code/architecture-agent.md` for detailed triggers.

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

**Canonical example (both fix types):**

```yaml
issues:
  - title: "Brief descriptive title"
    file: "src/path/to/file.ts"
    line: 23
    range: "23-25"  # null for single-line
    category: "Category Name"
    severity: "Critical|Major|Minor|Suggestion"
    description: "Detailed explanation of the issue"
    # Add agent-specific extra fields here (see your agent file)
    fix_type: "diff"
    fix_diff: |
      - const old = badCode();
      + const fixed = goodCode();

  - title: "Multi-location structural issue"
    file: "src/path/to/file.ts"
    line: 45
    range: "45-89"
    category: "Category Name"
    severity: "Major"
    description: "Issue requiring structural changes across multiple locations"
    fix_type: "prompt"
    fix_prompt: "Description of what to change and where. Include specific files, line ranges, and the approach."
```
