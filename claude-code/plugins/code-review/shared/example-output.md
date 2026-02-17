# Example Output

Consolidated example demonstrating canonical output format across all severity levels, fix types, and finding categories.

## Code Review

**Reviewed:** 3 file(s) | **Branch:** feature/user-auth
**Review Depth:** Deep review — Phase 1 (9 agents) → Phase 2 (5 gaps agents) → Synthesis

| Category | Critical | Major | Minor | Suggestions | Total |
|----------|----------|-------|-------|-------------|-------|
| Architecture | 0 | 1 | 1 | 0 | 2 |
| Bugs | 1 | 1 | 0 | 0 | 2 |
| Documentation | 1 | 1 | 0 | 0 | 2 |
| Security | 1 | 0 | 0 | 0 | 1 |
| Technical Debt | 0 | 1 | 0 | 1 | 2 |
| **Total** | **3** | **4** | **1** | **1** | **9** |

### Critical Issues (Must Fix)

**1. Race condition in balance update** `Critical` `Bugs`
`src/services/PaymentService.ts:67-75`

Balance is read, modified, and saved without atomic operation. Concurrent requests can cause incorrect balances or double-spend.

**Fix prompt** (copy to Claude Code):
> Fix the race condition in deductBalance in src/services/PaymentService.ts:67-75:
> 1. Wrap in a database transaction with row-level locking (SELECT ... FOR UPDATE)
> 2. Or use atomic update: UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?
> 3. Check affected rows count to verify the deduction succeeded before committing

---

**2. SQL injection in user query** `Critical` `Security` `[2 agents]`
`src/data/UserRepository.ts:45-48`

User input interpolated directly into SQL query without parameterization, enabling injection attacks.

**Fix**:
```diff
- const query = `SELECT * FROM users WHERE email = '${email}'`;
- const result = await db.query(query);
+ const query = 'SELECT * FROM users WHERE email = ?';
+ const result = await db.query(query, [email]);
```

---

**3. Incorrect function signature in API docs** `Critical` `Documentation`
`docs/api/users.md:45`

Documentation shows `createUser(name)` but actual signature is `createUser(name, email)`. Callers following the docs will get runtime errors.

- `documented_value`: `createUser(name: string): User`
- `actual_value`: `createUser(name: string, email: string): User`
- `code_location`: `src/api/users.ts:23`

**Fix**:
```diff
- ### createUser(name)
+ ### createUser(name, email)

- Creates a new user with the given name.
+ Creates a new user with the given name and email.

  **Parameters:**
  | Name | Type | Description |
  |------|------|-------------|
  | name | string | User's display name |
+ | email | string | User's email address |
```

---

### Major Issues (Should Fix)

**4. God class violates Single Responsibility Principle** `Major` `Architecture`
`src/services/UserService.ts:1-380`

UserService handles authentication, profile management, email notifications, and billing — four distinct responsibilities. Changes to billing logic can break authentication; testing requires mocking unrelated dependencies.

- `principle`: Single Responsibility Principle (SRP)

**Fix prompt** (copy to Claude Code):
> Split UserService into focused services:
> 1. Create src/services/AuthService.ts for login/logout/session management
> 2. Create src/services/ProfileService.ts for user profile data
> 3. Create src/services/NotificationService.ts for email sending
> 4. Move billing logic to existing BillingService if present, or create it
> 5. Update all import sites and inject new services via constructor

---

**5. Null reference on optional payment method** `Major` `Bugs`
`src/services/PaymentService.ts:34`

Accessing `user.paymentMethod.id` without checking if `paymentMethod` exists. Crashes when user has no payment method configured.

**Fix**:
```diff
- const methodId = user.paymentMethod.id;
+ const methodId = user.paymentMethod?.id;
+ if (!methodId) {
+   throw new Error('User has no payment method configured');
+ }
```

---

**6. No troubleshooting section in getting started guide** `Major` `Documentation`
`docs/getting-started.md:200`

Common setup failures (port conflicts, database connection, missing env vars) are undocumented, causing support requests.

- `missing_type`: section

**Fix prompt** (copy to Claude Code):
> Add a Troubleshooting section at the end of docs/getting-started.md covering:
> 1. Port already in use (EADDRINUSE) — change port in .env or kill conflicting process
> 2. Database connection failed (ECONNREFUSED) — verify PostgreSQL is running and DATABASE_URL is correct
> 3. Missing environment variables — copy .env.example to .env and fill required values

---

**7. Deprecated 'request' package** `Major` `Technical Debt`
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
> 4. Update tests that mock the request module
> 5. Remove request from package.json dependencies

---

### Minor Issues

**8. Duplicated validation logic across order handlers** `Minor` `Architecture`
`src/handlers/orders/createOrder.ts:23-45`

Order validation is copy-pasted in createOrder, updateOrder, and submitOrder with minor variations. Bug fixes must be applied in 3 places; validation rules may drift out of sync.

- `principle`: DRY (Don't Repeat Yourself)

**Fix**:
```diff
+ import { validateOrder } from '../validators/orderValidator';
+
  export async function createOrder(data: OrderDTO) {
-   if (!data.items || data.items.length === 0) {
-     throw new ValidationError('Order must have items');
-   }
-   if (!data.customerId) {
-     throw new ValidationError('Customer ID required');
-   }
+   validateOrder(data);
    // ... rest of handler
```

---

### Suggestions

**9. React class component in React 18+ project** `Suggestion` `Technical Debt`
`src/components/UserProfile.tsx:1-95`

Class component pattern where functional components with hooks are the modern standard.

- `debt_type`: outdated_pattern
- `urgency`: low
- `effort_estimate`: small

**Fix prompt** (copy to Claude Code):
> Convert UserProfile from class to functional component:
> 1. Replace class declaration with function component
> 2. Convert this.state to useState hooks
> 3. Convert componentDidMount/componentDidUpdate to useEffect hooks
> 4. Remove all 'this.' references

---

Review saved to: 2026-02-05_user-auth_deep-code-review.md
