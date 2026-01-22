# Output Format

This document defines the output format for code review results.

## Usage Summary Section

The Usage Summary appears BEFORE the Code Review header to provide visibility into which agents were invoked and their timing.

### Format

```markdown
## Usage Summary

| Metric | Value |
|--------|-------|
| Review Type | Deep (16 invocations) |
| Total Duration | 3m 5s |
| Agents Invoked | 16 of 16 planned |

### Phase Breakdown

| Phase | Duration | Agents | Status |
|-------|----------|--------|--------|
| Phase 1: Thorough | 1m 44s | 8/8 | ✓ |
| Phase 2: Gaps | 44s | 4/4 | ✓ |
| Synthesis | 29s | 4/4 | ✓ |

<details>
<summary>Agent Timing Details</summary>

**Phase 1: Thorough Review** (1m 44s)
| Agent | Model | Duration | Findings | Status |
|-------|-------|----------|----------|--------|
| compliance-agent | sonnet | 27s | 2 | ✓ |
| bug-detection-agent | opus | 1m 44s | 5 | ✓ |
| security-agent | opus | 1m 12s | 3 | ✓ |
| performance-agent | opus | 58s | 1 | ✓ |
| architecture-agent | sonnet | 35s | 0 | ✓ |
| api-contracts-agent | sonnet | 31s | 0 | ✓ |
| error-handling-agent | sonnet | 28s | 2 | ✓ |
| test-coverage-agent | sonnet | 33s | 4 | ✓ |

**Phase 2: Gaps Review** (44s)
| Agent | Model | Duration | Findings | Status |
|-------|-------|----------|----------|--------|
| compliance-agent | sonnet | 22s | 1 | ✓ |
| bug-detection-agent | sonnet | 44s | 2 | ✓ |
| security-agent | sonnet | 38s | 0 | ✓ |
| performance-agent | sonnet | 41s | 1 | ✓ |

**Synthesis** (29s)
| Agent | Model | Duration | Findings | Status |
|-------|-------|----------|----------|--------|
| synthesis (Security+Performance) | sonnet | 25s | 1 | ✓ |
| synthesis (Architecture+Test Coverage) | sonnet | 29s | 1 | ✓ |
| synthesis (Bugs+Error Handling) | sonnet | 22s | 0 | ✓ |
| synthesis (Compliance+Bugs) | sonnet | 27s | 0 | ✓ |

</details>

---
```

### Review Type Values

| Command | Review Type Value |
|---------|-------------------|
| `/deep-review` | Deep (16 invocations) |
| `/deep-review-staged` | Deep (16 invocations) |
| `/quick-review` | Quick (7 invocations) |
| `/quick-review-staged` | Quick (7 invocations) |

### Status Indicators

| Indicator | Meaning |
|-----------|---------|
| `✓` | Completed successfully |
| `✗` | Failed |
| `-` | Skipped (via skip_agents setting) |
| `[!]` | Too fast (< 5s for Synthesis, < 10s for Sonnet, < 15s for Opus) |
| `[*]` | Too slow (> 90s for Synthesis, > 120s for Sonnet, > 180s for Opus) |

### Duration Formatting

Format durations for readability:
- Under 60 seconds: `27s`
- 60+ seconds: `1m 44s`
- 60+ minutes: `1h 5m 30s`

### Quick Review Phase Names

For quick reviews, use these phase names:
- "Review" (instead of "Phase 1: Thorough")
- "Synthesis"

### Agents Invoked Calculation

`Agents Invoked` = count of agents with status "completed" or "failed"
`planned` = total expected agents for the review type (excluding skipped)

Example with skip_agents:
- Deep review with `skip_agents: ["architecture"]`
- Expected: 15 planned (16 - 1 skipped)
- Display: "15 of 15 planned"

### Handling Anomalies

When timing anomalies are detected, append the indicator to the status:

```markdown
| bug-detection-agent | opus | 8s | 0 | ✓ [!] |
| security-agent | opus | 3m 45s | 7 | ✓ [*] |
```

### Partial/Interrupted Reviews

If a review is interrupted before completion:
- Show available data for completed agents
- Use `(incomplete)` for phases that didn't finish
- Example: "Agents Invoked: 10 of 16 planned (incomplete)"

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
| `/deep-review` | Deep (16 invocations: 8 thorough + 4 gaps + 4 synthesis) |
| `/deep-review-staged` | Deep (16 invocations: 8 thorough + 4 gaps + 4 synthesis) |
| `/quick-review` | Quick (7 invocations: 4 review + 3 synthesis) |
| `/quick-review-staged` | Quick (7 invocations: 4 review + 3 synthesis) |

## No Issues Found

When no issues are found after validation:

```markdown
## Code Review

**Reviewed:** [N] file(s) | **Branch:** [branch-name]
**Review Depth:** [review-depth-description]

No issues found. All checks passed:
- Compliance (AI instructions)
- Bugs (logical errors, edge cases)
- Security
- Performance
- Architecture
- API Contracts
- Error Handling
- Test Coverage

Files reviewed:
- [file1.ts]
- [file2.ts]
```

For quick reviews (4 agents), only list the categories that were checked:

```markdown
No issues found. All checks passed:
- Bug detection
- Security analysis
- Error handling
- Test coverage
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
| Compliance | 0 | 0 | 0 | 0 |
| Bugs | 0 | 0 | 0 | 0 |
| Security | 0 | 0 | 0 | 0 |
| Performance | 0 | 0 | 0 | 0 |
| Architecture | 0 | 0 | 0 | 0 |
| API Contracts | 0 | 0 | 0 | 0 |
| Error Handling | 0 | 0 | 0 | 0 |
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
- `` `Compliance` ``
- `` `Bugs` ``
- `` `Security` ``
- `` `Performance` ``
- `` `Architecture` ``
- `` `API Contracts` ``
- `` `Error Handling` ``
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

Write the review to a file:
- Default for `/deep-review`: `.deep-review.md`
- Default for `/deep-review-staged`: `.deep-review-staged.md`
- Default for `/quick-review`: `.quick-review.md`
- Default for `/quick-review-staged`: `.quick-review-staged.md`
- Custom path if `--output-file` argument provided

End with:
```markdown
---
Review saved to: [filepath]
```

## Example Complete Output

```markdown
## Code Review

**Reviewed:** 3 file(s) | **Branch:** feature/user-auth
**Review Depth:** Deep (16 invocations: 8 thorough + 4 gaps + 4 synthesis)

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| Compliance | 0 | 1 | 0 | 0 |
| Bugs | 0 | 1 | 1 | 0 |
| Security | 1 | 0 | 0 | 0 |
| Performance | 0 | 0 | 1 | 1 |
| Architecture | 0 | 0 | 0 | 0 |
| API Contracts | 0 | 0 | 0 | 0 |
| Error Handling | 0 | 1 | 0 | 0 |
| Test Coverage | 0 | 0 | 0 | 2 |
| **Total** | **1** | **3** | **2** | **3** |

### Critical Issues (Must Fix)

**1. SQL Injection in User Query** `Critical` `Security` [2 agents]
`src/data/UserRepository.cs:45-48`

User input is directly concatenated into SQL query without parameterization.

**Fix**:
```diff
- var query = "SELECT * FROM Users WHERE Id = " + userId;
- var result = cmd.ExecuteQuery(query);
+ var query = "SELECT * FROM Users WHERE Id = @id";
+ cmd.Parameters.AddWithValue("@id", userId);
+ var result = cmd.ExecuteQuery(query);
```

### Major Issues (Should Fix)

**2. Missing CLAUDE.md Required Header** `Major` `Compliance`
`src/api/UserController.cs:1`

CLAUDE.md requires all API controllers to have a header comment describing the controller's purpose.

**Fix prompt** (copy to Claude Code):
> Add header comment to src/api/UserController.cs describing the controller's purpose per CLAUDE.md requirements. Include a brief description of the endpoints and their responsibilities.

**3. Null Reference Exception** `Major` `Bugs`
`src/services/AuthService.cs:78`

User object may be null when accessed on line 78.

**Fix**:
```diff
+ if (user == null)
+ {
+     throw new InvalidOperationException("User not found");
+ }
  return user.Email;
```

**4. Missing Error Handling on External Call** `Major` `Error Handling`
`src/services/AuthService.cs:92-95`

External API call has no try-catch block. Network failures will crash the service.

**Fix prompt** (copy to Claude Code):
> Wrap the external API call in src/services/AuthService.cs:92-95 in a try-catch block. Handle network failures gracefully by logging the error and returning an appropriate error response or throwing a custom ServiceUnavailableException.

### Minor Issues

**5. Potential Off-by-One Error** `Minor` `Bugs`
`src/utils/Pagination.cs:34`

Loop condition uses `<=` which may process one extra item.

**Fix**:
```diff
- for (int i = 0; i <= items.Count; i++)
+ for (int i = 0; i < items.Count; i++)
```

**6. Inefficient String Concatenation** `Minor` `Performance`
`src/utils/Logger.cs:56-62`

String concatenation in loop. Consider using StringBuilder.

**Fix prompt** (copy to Claude Code):
> Refactor the string concatenation loop in src/utils/Logger.cs:56-62 to use StringBuilder for better performance. Initialize StringBuilder before the loop and append to it, then call ToString() at the end.

### Suggestions

**7. Consider Caching User Lookup** `Suggestion` `Performance`
`src/services/AuthService.cs:45`

User lookup is called multiple times per request. Consider caching the result.

### Test Recommendations

**8. Missing Test for AuthService.ValidateToken** `Suggestion` `Test Coverage`
`src/services/AuthService.cs:100-120`

New validation logic has no corresponding tests.

- What: Test ValidateToken with expired tokens
- Behavior: Should throw TokenExpiredException
- Location: tests/services/AuthServiceTests.cs

**9. Missing Edge Case Test** `Suggestion` `Test Coverage`
`src/utils/Pagination.cs:30-40`

No tests for empty input or single-item lists.

- What: Test Paginate with empty list and single item
- Behavior: Should return empty page / single page respectively
- Location: tests/utils/PaginationTests.cs

### Cross-Cutting Insights

Issues spanning multiple categories:

**10. SQL Fix Adds Per-Request Overhead** `Major` `Security + Performance`
`src/data/UserRepository.cs:45-48`

The SQL parameterization fix (Issue #1) will add query preparation overhead on each request. The user lookup is called in authentication which handles 500+ requests/sec.

**Fix prompt** (copy to Claude Code):
> Cache the parameterized query preparation in UserRepository.cs using a static SqlCommand or prepared statement pattern to avoid repeated compilation overhead.

---
Review saved to: .code-review-staged.md
```
