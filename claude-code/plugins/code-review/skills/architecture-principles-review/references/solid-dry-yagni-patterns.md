# SOLID/DRY/YAGNI Detection Patterns

Detailed patterns and heuristics for detecting design principle violations.

## SOLID Principle Patterns

### Single Responsibility Principle (SRP)

**Detection heuristics:**

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| File line count | >500 lines | Major |
| Class method count | >15 methods | Major |
| Function parameter count | >10 parameters | Major |
| Import count from different domains | >5 unrelated domains | Minor |
| Mixed keywords (UI + DB + HTTP) | Present in same file | Major |

**Code patterns to flag:**

```typescript
// BAD: Class doing authentication AND user profile AND notifications
class UserService {
  login() { /* auth logic */ }
  logout() { /* auth logic */ }
  updateProfile() { /* profile logic */ }
  getProfile() { /* profile logic */ }
  sendEmail() { /* notification logic */ }
  sendPush() { /* notification logic */ }
}
```

**Grep patterns:**
- `class.*Service.*{` followed by methods from multiple domains
- Files importing both `@nestjs/common` and direct database drivers
- Files with both `fetch`/`axios` and DOM manipulation

---

### Open/Closed Principle (OCP)

**Detection heuristics:**

| Indicator | Severity |
|-----------|----------|
| Switch on type/kind string | Minor |
| If-else chains checking instanceof | Minor |
| Adding new case requires modifying existing code | Minor |

**Code patterns to flag:**

```typescript
// BAD: Must modify to add new payment type
function processPayment(type: string) {
  switch (type) {
    case 'credit': return processCreditCard();
    case 'debit': return processDebitCard();
    case 'paypal': return processPayPal();
    // Must add case here for new payment type
  }
}
```

**Grep patterns:**
- `switch.*type|kind|variant` followed by string literals
- `if.*instanceof.*else if.*instanceof`

---

### Liskov Substitution Principle (LSP)

**Detection heuristics:**

| Indicator | Severity |
|-----------|----------|
| `throw new NotImplementedError` in overridden methods | Major |
| `instanceof` checks on function parameters | Major |
| Type assertions after calling interface methods | Major |

**Code patterns to flag:**

```typescript
// BAD: Square violates Rectangle's setWidth/setHeight contract
class Square extends Rectangle {
  setWidth(w: number) {
    this.width = w;
    this.height = w; // Violates base class contract
  }
}

// BAD: Runtime type check on interface parameter
function processShape(shape: IShape) {
  if (shape instanceof Circle) {
    // Special handling for Circle
  }
}
```

**Grep patterns:**
- `throw new NotImplemented`
- `override.*{[^}]*throw`
- `instanceof.*\?.*:`

---

### Interface Segregation Principle (ISP)

**Detection heuristics:**

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Interface method count | >10 methods | Minor |
| Empty method implementations | Any | Minor |
| Methods returning `undefined` or throwing | Any | Minor |

**Code patterns to flag:**

```typescript
// BAD: Fat interface
interface IRepository<T> {
  findById(id: string): T;
  findAll(): T[];
  create(item: T): T;
  update(id: string, item: T): T;
  delete(id: string): void;
  softDelete(id: string): void;
  restore(id: string): void;
  count(): number;
  exists(id: string): boolean;
  findByName(name: string): T[];
  findByDate(date: Date): T[];
  // ... more methods
}

// BAD: Partial implementation
class ReadOnlyRepo implements IRepository<User> {
  create(item: User) { throw new Error('Not supported'); }
  update(id: string, item: User) { throw new Error('Not supported'); }
  delete(id: string) { throw new Error('Not supported'); }
  // ...
}
```

**Grep patterns:**
- `interface.*{` followed by >10 method signatures
- `implements.*{[^}]*throw new Error\(['"]Not`

---

### Dependency Inversion Principle (DIP)

**Detection heuristics:**

| Indicator | Severity |
|-----------|----------|
| `new ConcreteClass()` in business logic | Major |
| Import of concrete implementations (not interfaces) | Minor |
| Circular imports between modules | Major |

**Code patterns to flag:**

```typescript
// BAD: Direct instantiation of dependency
class OrderService {
  private emailService = new EmailService(); // Should be injected

  processOrder(order: Order) {
    this.emailService.send(order.customer.email);
  }
}

// BAD: Depending on concrete class
import { PostgresUserRepository } from './PostgresUserRepository';
// Should import IUserRepository interface
```

**Grep patterns:**
- `private.*=.*new [A-Z].*Service`
- `import.*Repository|Service.*from.*(?!interface)`

---

## DRY Violation Patterns

### Code Duplication

**Detection heuristics:**

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Similar code blocks | >10 lines, >80% similar | Minor |
| Identical function bodies | >5 lines | Minor |
| Copy-pasted validation | >3 occurrences | Minor |

**Code patterns to flag:**

```typescript
// BAD: Duplicated validation in multiple handlers
function createUser(data: UserDTO) {
  if (!data.email) throw new Error('Email required');
  if (!data.email.includes('@')) throw new Error('Invalid email');
  if (!data.name) throw new Error('Name required');
  // ... same validation in updateUser, importUser
}
```

---

### Repeated Configuration

**Detection heuristics:**

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Same literal value | >3 occurrences | Minor |
| Magic numbers | Any unexplained number | Suggestion |
| Repeated strings | >2 occurrences | Minor |

**Code patterns to flag:**

```typescript
// BAD: Magic numbers
setTimeout(callback, 30000); // What does 30000 mean?
if (retries > 3) { /* ... */ }
const PAGE_SIZE = 20; // Good
if (results.length > 20) { /* Bad - should use PAGE_SIZE */ }
```

---

## YAGNI Violation Patterns

### Unused Abstractions

**Detection heuristics:**

| Indicator | Severity |
|-----------|----------|
| Interface with exactly 1 implementation | Suggestion |
| Abstract class with exactly 1 subclass | Suggestion |
| Factory creating exactly 1 product type | Suggestion |
| Generic type parameter always same type | Suggestion |

**Grep patterns:**
- `implements I[A-Z].*` appearing exactly once in codebase
- `extends Abstract` appearing exactly once
- `Factory` classes with single `create` return type

---

### Over-Engineering

**Detection heuristics:**

| Indicator | Severity |
|-----------|----------|
| Options parameter always empty/default | Suggestion |
| Plugin/extension system with 0 plugins | Suggestion |
| Event emitter with 0 listeners | Suggestion |

**Code patterns to flag:**

```typescript
// BAD: Options never used
function fetchData(url: string, options: FetchOptions = {}) {
  // options is never anything but {}
  return fetch(url);
}

// All callers:
fetchData('/api/users');
fetchData('/api/orders');
// No caller ever passes options
```

---

### Speculative Generality

**Detection heuristics:**

| Indicator | Severity |
|-----------|----------|
| Parameters always same value | Suggestion |
| Switch/if branches never executed | Suggestion |
| Exported functions never imported | Suggestion |

**Grep patterns:**
- Function parameter always passed same literal
- `case 'x':` where 'x' never appears as input value
- `export function` with 0 imports elsewhere
