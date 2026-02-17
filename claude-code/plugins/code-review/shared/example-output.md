# Example Output

Consolidated example demonstrating canonical output format across severity levels, fix types, and finding categories.

## Code Review

**Reviewed:** 3 file(s) | **Branch:** feature/user-auth
**Review Depth:** Deep review — Phase 1 (9 agents) → Phase 2 (5 gaps agents) → Synthesis

| Category | Critical | Major | Minor | Suggestions | Total |
|----------|----------|-------|-------|-------------|-------|
| Architecture | 0 | 1 | 0 | 0 | 1 |
| Documentation | 1 | 0 | 0 | 0 | 1 |
| Security | 1 | 0 | 0 | 0 | 1 |
| Technical Debt | 0 | 1 | 0 | 0 | 1 |
| **Total** | **2** | **2** | **0** | **0** | **4** |

### Critical Issues (Must Fix)

**1. SQL injection in user query** `Critical` `Security` `[2 agents]`
`src/data/UserRepository.ts:45-48`
User input interpolated directly into SQL query without parameterization, enabling injection attacks.
**Fix**:
```diff
- const query = `SELECT * FROM users WHERE email = '${email}'`;
- const result = await db.query(query);
+ const query = 'SELECT * FROM users WHERE email = ?';
+ const result = await db.query(query, [email]);
```

**2. Incorrect function signature in API docs** `Critical` `Documentation`
`docs/api/users.md:45`
Documentation shows `createUser(name)` but actual signature is `createUser(name, email)`.
- `documented_value`: `createUser(name: string): User`
- `actual_value`: `createUser(name: string, email: string): User`
- `code_location`: `src/api/users.ts:23`
**Fix**:
```diff
- ### createUser(name)
+ ### createUser(name, email)
- Creates a new user with the given name.
+ Creates a new user with the given name and email.
  | name | string | User's display name |
+ | email | string | User's email address |
```

### Major Issues (Should Fix)

**3. God class violates Single Responsibility Principle** `Major` `Architecture`
`src/services/UserService.ts:1-380`
UserService handles authentication, profile management, email notifications, and billing — four distinct responsibilities.
- `principle`: Single Responsibility Principle (SRP)
**Fix prompt** (copy to Claude Code):
> Split UserService into focused services:
> 1. Create src/services/AuthService.ts for login/logout/session management
> 2. Create src/services/ProfileService.ts for user profile data
> 3. Create src/services/NotificationService.ts for email sending
> 4. Move billing logic to existing BillingService if present, or create it
> 5. Update all import sites and inject new services via constructor

**4. Deprecated 'request' package** `Major` `Technical Debt`
`package.json:15`
The `request` package has been deprecated since February 2020 and receives no security updates.
- `debt_type`: deprecated_dependency
- `urgency`: soon
- `effort_estimate`: medium
**Fix prompt** (copy to Claude Code):
> Replace 'request' with 'node-fetch' or 'axios':
> 1. Install replacement: npm install node-fetch
> 2. Find all usages: grep -r "require('request')" src/
> 3. Update each file to use fetch API or axios equivalent
> 4. Remove request from package.json dependencies

### Minor Issues

### Suggestions

Review saved to: 2026-02-05_user-auth_deep-code-review.md
