# Output Format

Authoritative output schema reference for all agents.

## Output Process

Generate: Header → Summary Table (reviewed categories only) → Issues (by severity) → Cross-Cutting Insights (if any) → Test Recommendations (if applicable). Display in terminal. Write to file (generated filename or `--output-file`). Print: "Review saved to: [filepath]"

## Filename

Pattern: `yyyy-mm-dd_few-words-summary_review-type.md` — 2-4 lowercase-hyphenated words from file names or "staged-changes". Review-type matches command name. Under 80 chars. Example: `2026-02-05_auth-middleware_deep-code-review.md`

## Review Header

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]
```

**No issues:** Header → "No issues found. All checks passed:" → checked categories → "Files reviewed:" → file list.

## Issues Found

Header followed by:
1. **Summary table**: Categories × severity (`Critical | Major | Minor | Suggestions`) + `**Total**` row. Only reviewed categories (9 deep, 4 quick).
2. **Severity sections**: `### Critical Issues (Must Fix)`, `### Major Issues (Should Fix)`, `### Minor Issues`, `### Suggestions` — only sections with issues.
3. **Cross-Cutting Insights** (after severity, before Test Recommendations)
4. **Test Recommendations** (if applicable)
5. Footer: `Review saved to: [filepath]`

## Cross-Cutting Insights

Only if synthesis agents produced `cross_cutting_insights`; omit section entirely if none.

```
**[N]. [Insight title]** `[Severity]` `[Primary Category] + [Secondary Category]`
`path/to/file.ts:line`
[Description referencing related findings from each category.]
[Fix using diff or prompt format]
```

## Issue Entry Format

```markdown
**1. SQL injection in user query** `Critical` `Security` `[2 agents]`
`src/api/users.ts:42-48`

User input interpolated directly into SQL query, enabling injection attacks.

**Fix**:
```diff
- const user = db.query(`SELECT * FROM users WHERE id = ${id}`);
+ const user = db.query('SELECT * FROM users WHERE id = ?', [id]);
```
```

Consensus badge: `[2 agents]` or `[3+ agents]` after category. Omit if single agent.

## Actionable Fix Formats

**One entry per unique issue.** Fixes must be directly applicable.

**Inline diffs** (fix_type: diff): Single-location, ≤10 lines, complete drop-in replacement (no `...` or partial code), no changes elsewhere. Format shown in Issue Entry above.

**Fix prompts** (fix_type: prompt): Multi-location/structural fixes. Blockquote format (`>`), action verb, specific files and changes.

````markdown
**Fix prompt** (copy to Claude Code):
> Refactor UserService to separate authentication logic into AuthService:
> 1. Create src/services/AuthService.ts
> 2. Move login(), logout(), validateToken() from UserService
> 3. Update imports in src/api/auth.ts and src/middleware/auth.ts
````
