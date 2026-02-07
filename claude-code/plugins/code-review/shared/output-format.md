# Output Format

This document defines the output format for code review results and serves as the **authoritative output schema reference** for all agents. This file is loaded once during output generation phase.

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

### Review Depth Descriptions

| Command | Review Depth Description |
|---------|-------------------------|
| `/deep-code-review` | Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis) |
| `/deep-code-review-staged` | Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis) |
| `/quick-code-review` | Quick (7 invocations: 4 review + 3 synthesis) |
| `/quick-code-review-staged` | Quick (7 invocations: 4 review + 3 synthesis) |

## No Issues Found

When no issues are found, use the standard header followed by "No issues found. All checks passed:" with only the categories that were actually checked, then "Files reviewed:" with the file list.

**Deep review categories (9):** API Contracts, Architecture, Bugs (logical errors, edge cases), Compliance (AI instructions), Error Handling, Performance, Security, Technical Debt, Test Coverage

**Quick review categories (4 + synthesis):** Bug detection, Error handling, Security analysis, Test coverage, Cross-cutting synthesis

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

### Major Issues (Should Fix)

### Minor Issues

### Suggestions

### Cross-Cutting Insights

### Test Recommendations

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

## Complete Output Example

For a complete example showing all format elements together, see:
`${CLAUDE_PLUGIN_ROOT}/shared/references/complete-output-example.md`
