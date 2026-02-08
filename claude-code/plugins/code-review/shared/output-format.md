# Output Format

This document defines the output format for code review results and serves as the **authoritative output schema reference** for all agents. This file is loaded once during output generation phase.

## Related Files

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

### Rules

- Use lowercase with hyphens. Keep summary to 2-4 words.
- Truncate to keep total filename under 80 characters.
- For staged commands, always use "staged-changes" as the summary.

---

## Review Header

All reviews start with this header:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]
```

## No Issues Found

When no issues are found, use the standard header followed by "No issues found. All checks passed:" with only the categories that were actually checked, then "Files reviewed:" with the file list.

## Issues Found

When issues are found, output the standard header followed by:

1. **Summary table**: Categories as rows, severity levels as columns (`Critical | Major | Minor | Suggestions`), with a `**Total**` row. Include only reviewed categories (9 for deep, 4 for quick).
2. **Severity-grouped sections**: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only include sections with issues.
3. **Cross-Cutting Insights**: After severity sections, before Test Recommendations (see below).
4. **Test Recommendations**: If applicable.
5. Footer: `Review saved to: [filepath]`

## Cross-Cutting Insights Section

After the regular severity-grouped issues, include cross-cutting insights from the synthesis phase. Format each insight as:

**[N]. [Insight title]** `[Severity]` `[Primary Category] + [Secondary Category]`
`path/to/file.ts:line`
[Description of the cross-cutting concern. Reference which findings from each category are related.]
[Fix suggestion using diff or prompt format]

**When to include:** Only if synthesis agents produced `cross_cutting_insights`. Appears AFTER severity-grouped issues, BEFORE Test Recommendations. If none, omit section entirely.

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

Use inline code format: `` `Critical` ``, `` `Major` ``, `` `Minor` ``, `` `Suggestion` ``

### Consensus Badge

When multiple agents flag the same issue:
- `[2 agents]` - Flagged by 2 agents
- `[3+ agents]` - Flagged by 3 or more agents

### File Location

Use inline code format with path and line range: `` `src/utils/helper.ts:42-48` ``

## Actionable Fix Formats

Fixes should be actionable - users can apply them directly without interpretation. Choose the format based on fix complexity.

**Only report ONE entry per unique issue. Do not duplicate issues.**

### Fix Type Classification

| Fix Type | When to Use | Format |
|----------|-------------|--------|
| `diff` | Single location, ≤10 lines, exact replacement known | Inline diff block |
| `prompt` | Multi-location, structural changes, or requires context decisions | Claude Code prompt |

### Inline Diffs (fix_type: diff)

For simple, single-location fixes. Must be complete drop-in replacements (no "..." or partial code), ≤10 lines, requiring no changes elsewhere.

````markdown
**Fix**:
```diff
- const user = db.query(`SELECT * FROM users WHERE id = ${id}`);
+ const user = db.query('SELECT * FROM users WHERE id = ?', [id]);
```
````

### Fix Prompts (fix_type: prompt)

For complex fixes requiring multiple locations, structural changes, or context decisions. Use blockquote format (`>`) for easy copying. Start with action verb, list specific files and changes.

````markdown
**Fix prompt** (copy to Claude Code):
> Refactor UserService to separate authentication logic into AuthService:
> 1. Create src/services/AuthService.ts
> 2. Move login(), logout(), validateToken() from UserService
> 3. Update imports in src/api/auth.ts and src/middleware/auth.ts
````

## Complete Output Example

For a complete example showing all format elements together, see:
`${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md`

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
