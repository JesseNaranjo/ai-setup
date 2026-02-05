# Output Format

This document defines the output format for code review results and serves as the **authoritative output schema reference** for all agents. This file is loaded once during output generation phase.

## Contents

- [Related Files](#related-files)
- [Output Generation Process](#output-generation-process)
  - [Generate Review Output](#0-generate-review-output)
  - [Display Output](#1-display-output)
  - [Write to File](#2-write-to-file)
  - [Confirm Output](#3-confirm-output)
  - [Fix Formatting Rules](#fix-formatting-rules)
- [Filename Generation](#filename-generation)
- [Review Header](#review-header)
  - [Review Depth Descriptions](#review-depth-descriptions)
- [No Issues Found](#no-issues-found)
  - [Deep Review (9 categories)](#deep-review-9-categories)
  - [Quick Review (4 categories + synthesis)](#quick-review-4-categories--synthesis)
- [Issues Found](#issues-found)
- [Cross-Cutting Insights Section](#cross-cutting-insights-section)
- [Issue Entry Format](#issue-entry-format)
  - [Severity Badge](#severity-badge)
  - [Category Badge](#category-badge)
  - [Consensus Badge](#consensus-badge)
  - [File Location](#file-location)
- [Actionable Fix Formats](#actionable-fix-formats)
  - [Fix Type Classification](#fix-type-classification)
  - [Inline Diffs](#inline-diffs-fix_type-diff)
  - [Fix Prompts](#fix-prompts-fix_type-prompt)
  - [Legacy Format Support](#legacy-format-support)
- [File Output](#file-output)
- [Complete Output Example](#complete-output-example)

## Related Files

- `${CLAUDE_PLUGIN_ROOT}/shared/severity-definitions.md` - Canonical severity definitions
- `${CLAUDE_PLUGIN_ROOT}/shared/validation-rules.md` - Validation and aggregation rules
- `${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md` - Complete output example (reference)

## Output Generation Process

### 0. Generate Review Output

1. **Header**: Use the review depth description provided by the command
2. **Summary Table**: Include only the categories that were reviewed
3. **Issues**: Group by severity (Critical, Major, Minor, Suggestions)
4. **Cross-Cutting Insights**: Include if synthesis agents produced insights
5. **Test Recommendations**: Include if applicable

### 1. Display Output

Display the formatted review in the terminal.

### 2. Write to File

Write the same content to a file using the generated filename (see [Filename Generation](#filename-generation)).

If `--output-file <path>` argument was provided, use that path instead.

### 3. Confirm Output

At the end, print: "Review saved to: [filepath]"

### Fix Formatting Rules

- **fix_type: diff** for single-location fixes ≤10 lines
  - Show inline diff block with `-` and `+` lines
  - Must be complete drop-in replacements (no "..." or partial code)

- **fix_type: prompt** for multi-location, structural, or complex fixes
  - Show copyable Claude Code prompt in blockquote format
  - Must be specific and actionable

**Only report ONE entry per unique issue. Do not duplicate issues.**

---

## Filename Generation

Generate output filenames following this pattern:
`yyyy-mm-dd_few-words-summary_review-type.md`

### Components

- `yyyy-mm-dd`: Current date (e.g., `2026-02-05`)
- `few-words-summary`: 2-4 words describing the review scope, derived from:
  - File names being reviewed (for file-based commands)
  - "staged-changes" (for staged commands)
  - Main feature/component name when identifiable
- `review-type`: The command type (`deep-code-review`, `quick-code-review`, etc.)

### Examples

| Command | Example Filename |
|---------|------------------|
| `/deep-code-review` | `2026-02-05_auth-service-refactor_deep-code-review.md` |
| `/deep-code-review-staged` | `2026-02-05_staged-changes_deep-code-review-staged.md` |
| `/deep-docs-review` | `2026-02-05_api-docs_deep-docs-review.md` |
| `/quick-code-review` | `2026-02-05_utils-helpers_quick-code-review.md` |
| `/quick-code-review-staged` | `2026-02-05_staged-changes_quick-code-review-staged.md` |
| `/quick-docs-review` | `2026-02-05_readme-updates_quick-docs-review.md` |

### Rules

- Use lowercase with hyphens for word separation
- Keep summary to 2-4 words maximum
- Replace spaces and special characters with hyphens
- Truncate long names to keep total filename under 80 characters
- For staged commands, always use "staged-changes" as the summary

---

## Review Header

All reviews start with this header:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]
```

### Review Depth Descriptions

| Command | Review Depth Description |
|---------|-------------------------|
| `/deep-code-review` | Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis) |
| `/deep-code-review-staged` | Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis) |
| `/quick-code-review` | Quick (7 invocations: 4 review + 3 synthesis) |
| `/quick-code-review-staged` | Quick (7 invocations: 4 review + 3 synthesis) |

## No Issues Found

When no issues are found after validation, list only the categories that were actually checked.

### Deep Review (9 categories)

For deep reviews (`/deep-code-review`, `/deep-code-review-staged`), list all 9 categories:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis)

No issues found. All checks passed:
- API Contracts
- Architecture
- Bugs (logical errors, edge cases)
- Compliance (AI instructions)
- Error Handling
- Performance
- Security
- Technical Debt
- Test Coverage

Files reviewed:
- [file1.ts]
- [file2.ts]
```

### Quick Review (4 categories + synthesis)

For quick reviews (`/quick-code-review`, `/quick-code-review-staged`), list only the 4 categories that were checked:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** Quick (7 invocations: 4 review + 3 synthesis)

No issues found. All checks passed:
- Bug detection
- Error handling
- Security analysis
- Test coverage
- Cross-cutting synthesis

Files reviewed:
- [file1.ts]
- [file2.ts]
```

## Issues Found

When issues are found:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| API Contracts | 0 | 0 | 0 | 0 |
| Architecture | 0 | 0 | 0 | 0 |
| Bugs | 0 | 0 | 0 | 0 |
| Compliance | 0 | 0 | 0 | 0 |
| Error Handling | 0 | 0 | 0 | 0 |
| Performance | 0 | 0 | 0 | 0 |
| Security | 0 | 0 | 0 | 0 |
| Technical Debt | 0 | 0 | 0 | 0 |
| Test Coverage | 0 | 0 | 0 | 0 |
| **Total** | **0** | **0** | **0** | **0** |

### Critical Issues (Must Fix)

[List critical issues or "None"]

### Major Issues (Should Fix)

[List major issues or "None"]

### Minor Issues

[List minor issues or "None"]

### Suggestions

[List suggestions or "None"]

### Cross-Cutting Insights

[Issues identified by synthesis agents that span multiple categories. Show only if synthesis phase produced insights.]

### Test Recommendations

[Aggregated test case suggestions]

---
Review saved to: [filepath]
```

## Cross-Cutting Insights Section

After the regular severity-grouped issues, include cross-cutting insights from the synthesis phase. These are issues that span multiple review categories and represent ripple effects or hidden interactions.

### Format

```markdown
### Cross-Cutting Insights

Issues spanning multiple categories:

**[N]. [Insight title]** `[Severity]` `[Primary Category] + [Secondary Category]`
`path/to/file.ts:line`

[Description of the cross-cutting concern - what it is and why it matters. Reference which findings from each category are related.]

[Fix suggestion using diff or prompt format]
```

### Example Cross-Cutting Insight

```markdown
### Cross-Cutting Insights

Issues spanning multiple categories:

**10. Security Fix Creates Performance Regression** `Major` `Security + Performance`
`src/services/AuthService.cs:45-60`

The SQL parameterization fix (Issue #1) adds query preparation overhead that will be called on every request. The auth endpoint handles 1000+ requests/sec.

**Fix prompt** (copy to Claude Code):
> Cache the parameterized query preparation in AuthService.cs to avoid repeated compilation overhead. Use a static prepared statement or query cache with appropriate TTL.

**11. Architectural Change Missing Test Coverage** `Major` `Architecture + Test Coverage`
`src/interfaces/IUserService.ts:1-25`

The new IUserService interface (from architectural refactoring) has no corresponding tests. Existing tests still use the concrete class.

**Fix prompt** (copy to Claude Code):
> Add interface contract tests for IUserService in tests/interfaces/IUserService.test.ts. Ensure all implementations satisfy the interface contract.
```

### When to Include

- Only include if synthesis agents produced `cross_cutting_insights`
- Cross-cutting insights appear AFTER the regular severity-grouped issues
- Cross-cutting insights appear BEFORE Test Recommendations
- If no cross-cutting insights, omit this section entirely

## Issue Entry Format

Each issue entry follows this format:

```markdown
**[N]. [Issue title]** `[Severity]` `[Category]` [Consensus badge if applicable]
`path/to/file.ts:line-range`

[Description of the issue - what it is and why it matters]

[For small fixes - suggestion block]
[For larger fixes - prompt block]
```

### Severity Badge

Use inline code format:
- `` `Critical` ``
- `` `Major` ``
- `` `Minor` ``
- `` `Suggestion` ``

### Category Badge

Use inline code format:
- `` `API Contracts` ``
- `` `Architecture` ``
- `` `Bugs` ``
- `` `Compliance` ``
- `` `Error Handling` ``
- `` `Performance` ``
- `` `Security` ``
- `` `Test Coverage` ``

### Consensus Badge

When multiple agents flag the same issue:
- `[2 agents]` - Flagged by 2 agents
- `[3+ agents]` - Flagged by 3 or more agents

### File Location

Use inline code format with path and line range:
`` `src/utils/helper.ts:42-48` ``

## Actionable Fix Formats

Fixes should be actionable - users can apply them directly without interpretation. Choose the format based on fix complexity.

### Fix Type Classification

| Fix Type | When to Use | Format |
|----------|-------------|--------|
| `diff` | Single location, ≤10 lines, exact replacement known | Inline diff block |
| `prompt` | Multi-location, structural changes, or requires context decisions | Claude Code prompt |

### Inline Diffs (fix_type: diff)

For simple, single-location fixes where the exact code change is known:

````markdown
**Fix**:
```diff
- const user = db.query(`SELECT * FROM users WHERE id = ${id}`);
+ const user = db.query('SELECT * FROM users WHERE id = ?', [id]);
```
````

**Requirements for diff blocks:**
- Lines starting with `-` show code to remove
- Lines starting with `+` show code to add
- Must be complete (no "..." or partial code)
- Must be a drop-in replacement
- Must not require changes elsewhere
- Should be ≤10 lines changed

**Multi-line diff example:**
````markdown
**Fix**:
```diff
- async function getUser(id) {
-   const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);
-   return user;
- }
+ async function getUser(id: string): Promise<User | null> {
+   const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
+   if (!user) return null;
+   return user;
+ }
```
````

### Fix Prompts (fix_type: prompt)

For complex fixes requiring multiple locations, structural changes, or context decisions:

````markdown
**Fix prompt** (copy to Claude Code):
> Refactor UserService to separate authentication logic into AuthService:
> 1. Create src/services/AuthService.ts
> 2. Move login(), logout(), validateToken() from UserService
> 3. Update imports in src/api/auth.ts and src/middleware/auth.ts
````

**Requirements for prompt blocks:**
- Use blockquote format (`>`) for easy copying
- Start with the action verb (Refactor, Extract, Add, Fix, etc.)
- List specific files and changes when multi-location
- Be specific enough that Claude Code can execute without clarification

**Single-location prompt example (when diff is impractical):**
````markdown
**Fix prompt** (copy to Claude Code):
> Add comprehensive input validation to processOrder in src/services/orders.ts:72-95. Validate orderId is a valid UUID, items array is non-empty with valid product IDs and quantities > 0, and customerId exists in the database. Throw descriptive errors for each validation failure.
````

### Legacy Format Support

For backward compatibility, `suggestion` blocks are still supported and equivalent to diffs:

````markdown
```suggestion
const result = await fetchData();
if (!result) {
  throw new Error('Failed to fetch data');
}
```
````

This is treated as `fix_type: diff` internally.

## File Output

Write the review to a file using the generated filename (see [Filename Generation](#filename-generation)).

If `--output-file <path>` argument was provided, use that path instead of the generated filename.

End with:
```markdown
---
Review saved to: [filepath]
```

## Complete Output Example

For a complete example showing all format elements together, see:
`${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md`
