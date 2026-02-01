# Complete Output Example

This reference file contains a complete example showing all format elements together for code review output. This example is loaded on-demand when formatting final review results.

See `${CLAUDE_PLUGIN_ROOT}/shared/output-format.md` for format specifications and rules.

**Note:** This file contains a complete example inside a code block. The example demonstrates all format elements: Summary Table, Critical/Major/Minor Issues, Suggestions, Test Recommendations, and Cross-Cutting Insights.

```markdown
## Code Review

**Reviewed:** 3 file(s) | **Branch:** feature/user-auth
**Review Depth:** Deep (19 invocations: 9 thorough + 5 gaps + 5 synthesis)

### Summary

| Category | Critical | Major | Minor | Suggestions |
|----------|----------|-------|-------|-------------|
| API Contracts | 0 | 0 | 0 | 0 |
| Architecture | 0 | 0 | 0 | 0 |
| Bugs | 0 | 1 | 1 | 0 |
| Compliance | 0 | 1 | 0 | 0 |
| Error Handling | 0 | 1 | 0 | 0 |
| Performance | 0 | 0 | 1 | 1 |
| Security | 1 | 0 | 0 | 0 |
| Technical Debt | 0 | 0 | 0 | 0 |
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
