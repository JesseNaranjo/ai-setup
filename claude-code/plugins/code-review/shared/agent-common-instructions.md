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
Code with non-obvious but valid edge case handling or intentional patterns.

#### Linter Territory
Formatting/import issues handled by linters/formatters (do NOT run linters to verify).

#### Pedantic Concerns
Minor style preferences a senior engineer would not flag.

#### Pre-existing Issues
Issues in the codebase that existed before the current changes and weren't modified.

#### Scope Limitations
General code quality concerns unless required in AI Agent Instructions. Issues in test code unless specifically reviewing tests. Theoretical edge cases extremely unlikely in practice.

#### Silenced Issues
Code with lint-disable/suppress comments or documented intentional suppressions.

### Deep vs Quick Review Differences

**Deep review** can flag more issues but should still avoid pre-existing issues, silenced issues, and pure style preferences.

**Quick review** is optimized for speed: focus only on blocking issues, ignore minor style concerns, skip theoretical edge cases.

### Category-Specific False Positive Rules

- **Code reviews:** See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` "Category-Specific False Positive Rules (Code) > [Your Category]"
- **Docs reviews:** See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-docs.md` "Category-Specific False Positive Rules (Documentation) > [Your Category]"

## Gaps Mode Behavior Template

When MODE=gaps, agents receive `previous_findings` from thorough mode to avoid duplicates.

### Duplicate Detection (Common to All Gaps Agents)

- Skip issues in same file within +/-5 lines of prior findings
- Skip same issue type on same function/method
- For range findings (lines A-B): skip zone = [A-5, B+5]

See `${CLAUDE_PLUGIN_ROOT}/shared/review-orchestration-code.md` Prompt Schema table (`previous_findings` field) for the schema.

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
- Issue is in a line starting with `+` in the diff (newly added code)
- Change INTRODUCES the issue (e.g., removes null check that protected existing code)
- Change WORSENS an existing issue (e.g., increases scope of vulnerability)

**Do NOT flag:**
- Issues in unchanged code (lines without `+` prefix)
- Pre-existing problems not made worse by the change
- Style issues in untouched code nearby
- Issues visible in "full file" context but not in the diff

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

## Severity Definitions

### Critical

Issues that must be fixed before merge. These represent:

- **Exploitable security vulnerabilities**: SQL injection, XSS, authentication bypass, command injection
- **Data loss or corruption risks**: Unhandled writes, race conditions affecting data integrity
- **System crashes or service unavailability**: Null pointer dereferences in critical paths, infinite loops
- **Broken core functionality**: Logic errors that prevent primary features from working

**Action**: Block merge until resolved.

### Major

Issues that should be fixed but may not block merge in time-sensitive situations:

- **Security issues requiring specific conditions**: CSRF without sensitive actions, information disclosure requiring authentication
- **Significant performance degradation**: O(n²) in hot paths, memory leaks over time, N+1 queries
- **Logic errors affecting key workflows**: Incorrect calculations, wrong branch conditions
- **Breaking API changes**: Removed endpoints, changed response shapes, incompatible parameter changes

**Action**: Fix before merge unless time-critical with documented exception.

### Minor

Issues that improve code quality but don't affect correctness:

- **Code quality issues**: Duplicated code, overly complex functions, poor naming
- **Non-critical performance inefficiencies**: Suboptimal but not problematic performance
- **Style/pattern inconsistencies**: Deviations from project conventions
- **Missing edge case handling**: Uncommon scenarios without explicit handling

**Action**: Fix when convenient or in follow-up PR.

### Suggestion

Opportunities for improvement, not problems:

- **Improvement opportunities**: Better algorithms, cleaner patterns
- **Better alternatives exist**: More idiomatic approaches available
- **Documentation gaps**: Missing comments on complex logic
- **Test coverage recommendations**: Additional test cases that would improve confidence

**Action**: Consider for future improvement.

### Severity by Category Examples

| Category | Critical | Major | Minor | Suggestion |
|----------|----------|-------|-------|------------|
| **Security** | SQL injection | CSRF on sensitive action | Missing CSP header | Add rate limiting |
| **Bug** | Null deref in payment flow | Wrong calculation | Off-by-one in pagination | Simplify conditional |
| **Performance** | Infinite loop | N+1 query | Unnecessary re-render | Use memo() |
| **Architecture** | Circular dependency breaking build | Tight coupling | Missing abstraction | Consider pattern X |
| **API** | Removed required endpoint | Changed response type | Inconsistent naming | Add OpenAPI docs |
| **Error Handling** | Crash on invalid input | Missing retry on transient error | Generic error message | Add error codes |
| **Test Coverage** | No tests for auth flow | Missing edge case test | Low branch coverage | Add property tests |
| **Compliance** | Violates MUST rule | Violates SHOULD rule | Inconsistent with guideline | Could follow suggestion |
| **Technical Debt** | Deprecated dependency with CVE | Major version 2+ behind | TODO without tracking | Modernization opportunity |

### Technical Debt Severity Guidelines

- **Critical**: Deprecated dependency with known vulnerabilities (CVE), removed API usage requiring immediate migration, blocking modernization path
- **Major**: Major version 2+ behind with breaking changes pending, scalability blocker in production, extensive workaround code affecting multiple files
- **Minor**: Outdated patterns that still work correctly, TODO/FIXME without urgency or tracking, minor documentation gaps
- **Suggestion**: Code modernization opportunity, style improvements, optional refactoring for consistency
