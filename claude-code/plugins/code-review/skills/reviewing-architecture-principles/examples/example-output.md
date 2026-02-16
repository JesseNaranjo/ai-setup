# Architecture Principles Review - Example Output

Sample output demonstrating SOLID, DRY, and YAGNI violation findings.

## Example Review Output

```yaml
issues:
  - title: "God class violates Single Responsibility Principle"
    file: "src/services/UserService.ts"
    line: 1
    range: "1-380"
    category: "Architecture"
    severity: "Major"
    description: "UserService handles authentication, profile management, email notifications, and billing - four distinct responsibilities."
    principle: "Single Responsibility Principle (SRP)"
    impact: "Changes to billing logic could break authentication; testing requires mocking unrelated dependencies"
    fix_type: "prompt"
    fix_prompt: "Split UserService into focused services: 1) AuthService for login/logout/session, 2) ProfileService for user data, 3) NotificationService for emails, 4) BillingService for payments. Each service should have single responsibility and own tests."

  - title: "Duplicated validation logic across order handlers"
    file: "src/handlers/orders/createOrder.ts"
    line: 23
    range: "23-45"
    category: "Architecture"
    severity: "Minor"
    description: "Order validation (lines 23-45) is copy-pasted in createOrder, updateOrder, and submitOrder with minor variations."
    principle: "DRY (Don't Repeat Yourself)"
    impact: "Bug fixes must be applied in 3 places; validation rules may drift out of sync"
    fix_type: "diff"
    fix_diff: |
      + import { validateOrder } from '../validators/orderValidator';
      +
        export async function createOrder(data: OrderDTO) {
      -   if (!data.items || data.items.length === 0) {
      -     throw new ValidationError('Order must have items');
      -   }
      -   if (!data.customerId) {
      -     throw new ValidationError('Customer ID required');
      -   }
      -   if (data.items.some(i => i.quantity <= 0)) {
      -     throw new ValidationError('Invalid quantity');
      -   }
      +   validateOrder(data);
          // ... rest of handler

  - title: "Direct database instantiation violates Dependency Inversion"
    file: "src/services/OrderService.ts"
    line: 8
    category: "Architecture"
    severity: "Major"
    description: "OrderService creates its own PostgresOrderRepository instance instead of receiving it via dependency injection."
    principle: "Dependency Inversion Principle (DIP)"
    impact: "Cannot swap repository for testing; tightly coupled to PostgreSQL; difficult to mock in unit tests"
    fix_type: "diff"
    fix_diff: |
      - import { PostgresOrderRepository } from '../repositories/PostgresOrderRepository';
      + import { IOrderRepository } from '../interfaces/IOrderRepository';

        export class OrderService {
      -   private repository = new PostgresOrderRepository();
      +   constructor(private repository: IOrderRepository) {}

          async getOrder(id: string) {
            return this.repository.findById(id);
          }
        }
```

## Summary Statistics

```
Architecture Principles Review Summary
======================================

SOLID Violations:
  - SRP: 1 Major
  - DIP: 1 Major

DRY Violations:
  - Code duplication: 1 Minor

Total Issues: 3
  - Major: 2 (must fix)
  - Minor: 1 (should fix)
```
