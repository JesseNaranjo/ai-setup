# Compliance Review Example Output

This is an example of the expected output format for a compliance review.

---

## Compliance Review

**Scope:** src/api/orders.ts, src/services/OrderService.ts
**Project Type:** Node.js/TypeScript
**Instructions Source:** CLAUDE.md

### Rules Checked

From `CLAUDE.md`:
- All API endpoints MUST have authentication
- Controllers MUST NOT contain business logic
- All public functions MUST have JSDoc comments

### Findings

| Severity | Count |
|----------|-------|
| Major | 2 |
| Minor | 1 |

### Major Issues (Rule Violations)

**1. Missing Authentication on Endpoint** `Major` `Compliance`
`src/api/orders.ts:45`

**Rule violated:** "All API endpoints MUST have authentication" (CLAUDE.md:23)

The `/api/orders/export` endpoint has no authentication middleware.

```typescript
// Current code
router.get('/export', exportOrders);
```

**Fix**:
```diff
- router.get('/export', exportOrders);
+ router.get('/export', requireAuth, exportOrders);
```

---

**2. Business Logic in Controller** `Major` `Compliance`
`src/api/orders.ts:67-85`

**Rule violated:** "Controllers MUST NOT contain business logic" (CLAUDE.md:31)

The controller calculates order totals and applies discounts directly instead of delegating to a service.

```typescript
router.post('/create', async (req, res) => {
  // Business logic in controller
  const subtotal = items.reduce((sum, i) => sum + i.price * i.qty, 0);
  const discount = user.isPremium ? subtotal * 0.1 : 0;
  const total = subtotal - discount + tax;
  // ...
});
```

**Fix prompt** (copy to Claude Code):
> Move the order calculation logic from src/api/orders.ts:67-85 to OrderService:
> 1. Create OrderService.calculateTotal(items, user) method
> 2. Move subtotal, discount, and total calculations to the service
> 3. Update the controller to call orderService.calculateTotal()

---

### Minor Issues

**3. Missing JSDoc on Public Function** `Minor` `Compliance`
`src/services/OrderService.ts:34`

**Rule violated:** "All public functions MUST have JSDoc comments" (CLAUDE.md:47)

The `processRefund` function has no JSDoc documentation.

```typescript
async processRefund(orderId: string, amount: number): Promise<RefundResult> {
```

**Fix**:
```diff
+ /**
+  * Process a refund for an order
+  * @param orderId - The ID of the order to refund
+  * @param amount - The refund amount in cents
+  * @returns The refund result with transaction ID
+  * @throws {OrderNotFoundError} If order doesn't exist
+  */
  async processRefund(orderId: string, amount: number): Promise<RefundResult> {
```

---

### Summary

- **2 Major violations**: Missing auth and business logic in controller
- **1 Minor violation**: Missing JSDoc documentation

### Compliance Score

- Rules checked: 3
- Rules passed: 0
- Rules violated: 3
- Compliance rate: 0%

---
Review saved to: .reviewing-compliance.md
