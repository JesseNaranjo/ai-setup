# Output Format

Authoritative output schema reference for all agents.

## Output Generation Process

1. **Generate**: Header (review depth from command) → Summary Table (reviewed categories only) → Issues (by severity) → Cross-Cutting Insights (if any) → Test Recommendations (if applicable)
2. **Display** in terminal
3. **Write** to file using generated filename (or `--output-file <path>` if provided)
4. Print: "Review saved to: [filepath]"

---

## Filename Generation

Pattern: `yyyy-mm-dd_few-words-summary_review-type.md`

Summary: 2-4 lowercase-hyphenated words from file names (file-based) or "staged-changes" (staged). Review-type matches the command name. Keep total under 80 chars.

Example: `2026-02-05_auth-middleware_deep-code-review.md`

---

## Review Header

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]
```

**No issues:** Standard header, then "No issues found. All checks passed:" with checked categories, then "Files reviewed:" with file list.

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

Consensus badge: `[2 agents]` or `[3+ agents]`, appended after category. Omit if single agent.

## Actionable Fix Formats

Fixes must be directly applicable. **Only report ONE entry per unique issue.**

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
