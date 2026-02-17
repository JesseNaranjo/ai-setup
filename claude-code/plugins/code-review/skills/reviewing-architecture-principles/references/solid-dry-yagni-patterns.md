# SOLID/DRY/YAGNI Detection Patterns

## Contents
- SRP Detection
- OCP Detection
- LSP Detection
- ISP Detection
- DIP Detection
- DRY Violations
- YAGNI Violations

## SRP Detection

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| File line count | >500 lines | Major |
| Class method count | >15 methods | Major |
| Function parameter count | >10 parameters | Major |
| Import count from different domains | >5 unrelated domains | Minor |
| Mixed keywords (UI + DB + HTTP) | Present in same file | Major |

```typescript
// BAD: Class doing authentication AND user profile AND notifications
class UserService {
  login() { ... } logout() { ... }      // auth
  updateProfile() { ... }               // profile
  sendEmail() { ... } sendPush() { ... } // notifications
}
```

**Grep:** `class.*Service.*{` with methods from multiple domains; files importing both `@nestjs/common` and direct DB drivers; files with both `fetch`/`axios` and DOM manipulation

## OCP Detection

| Indicator | Severity |
|-----------|----------|
| Switch on type/kind string | Minor |
| If-else chains checking instanceof | Minor |
| Adding new case requires modifying existing code | Minor |

```typescript
// BAD: Must modify to add new payment type
function processPayment(type: string) {
  switch (type) { case 'credit': ... case 'debit': ... case 'paypal': ... }
}
```

**Grep:** `switch.*type|kind|variant` with string literals; `if.*instanceof.*else if.*instanceof`

## LSP Detection

| Indicator | Severity |
|-----------|----------|
| `throw new NotImplementedError` in overridden methods | Major |
| `instanceof` checks on function parameters | Major |
| Type assertions after calling interface methods | Major |

```typescript
// BAD: Square violates Rectangle's setWidth/setHeight contract
class Square extends Rectangle {
  setWidth(w: number) { this.width = w; this.height = w; }
}
// BAD: Runtime type check on interface parameter
function processShape(shape: IShape) { if (shape instanceof Circle) { ... } }
```

**Grep:** `throw new NotImplemented`; `override.*{[^}]*throw`; `instanceof.*\?.*:`

## ISP Detection

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Interface method count | >10 methods | Minor |
| Empty method implementations | Any | Minor |
| Methods returning `undefined` or throwing | Any | Minor |

```typescript
// BAD: Partial implementation of fat interface
class ReadOnlyRepo implements IRepository<User> {
  create(item: User) { throw new Error('Not supported'); }
  update(id: string, item: User) { throw new Error('Not supported'); }
}
```

**Grep:** `interface.*{` with >10 method signatures; `implements.*{[^}]*throw new Error\(['"]Not`

## DIP Detection

| Indicator | Severity |
|-----------|----------|
| `new ConcreteClass()` in business logic | Major |
| Import of concrete implementations (not interfaces) | Minor |
| Circular imports between modules | Major |

```typescript
// BAD: Direct instantiation of dependency
class OrderService {
  private emailService = new EmailService(); // Should be injected
}
```

**Grep:** `private.*=.*new [A-Z].*Service`; `import.*Repository|Service.*from.*(?!interface)`

## DRY Violations

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Similar code blocks | >10 lines, >80% similar | Minor |
| Identical function bodies | >5 lines | Minor |
| Copy-pasted validation | >3 occurrences | Minor |
| Same literal value | >3 occurrences | Minor |
| Magic numbers | Any unexplained number | Suggestion |

```typescript
// BAD: Duplicated validation in multiple handlers
function createUser(data: UserDTO) {
  if (!data.email) throw new Error('Email required');
  if (!data.email.includes('@')) throw new Error('Invalid email');
  // ... same validation repeated in updateUser, importUser
}
```

## YAGNI Violations

| Indicator | Severity |
|-----------|----------|
| Interface with exactly 1 implementation | Suggestion |
| Abstract class with exactly 1 subclass | Suggestion |
| Factory creating exactly 1 product type | Suggestion |
| Generic type parameter always same type | Suggestion |
| Options parameter always empty/default | Suggestion |
| Plugin/extension system with 0 plugins | Suggestion |
| Parameters always same value | Suggestion |
| Switch/if branches never executed | Suggestion |
| Exported functions never imported | Suggestion |

**Grep:** `implements I[A-Z].*` appearing exactly once; `extends Abstract` appearing exactly once; `export function` with 0 imports elsewhere
