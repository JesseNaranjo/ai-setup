# SOLID/DRY/YAGNI Detection Patterns

## SRP Detection

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| File lines | >500 | Major |
| Class methods | >15 | Major |
| Function params | >10 | Major |
| Imports from different domains | >5 unrelated | Minor |
| Mixed keywords (UI + DB + HTTP) | Same file | Major |

**Grep:** `class.*Service.*{` with methods from multiple domains; files importing both `@nestjs/common` and direct DB drivers; files with both `fetch`/`axios` and DOM manipulation

## OCP Detection

| Indicator | Severity |
|-----------|----------|
| Switch on type/kind string | Minor |
| If-else chains checking instanceof | Minor |
| Adding new case requires modifying existing | Minor |

**Grep:** `switch.*type|kind|variant` with string literals; `if.*instanceof.*else if.*instanceof`

## LSP Detection

| Indicator | Severity |
|-----------|----------|
| `throw new NotImplementedError` in overrides | Major |
| `instanceof` checks on function params | Major |
| Type assertions after interface method calls | Major |

**Grep:** `throw new NotImplemented`; `override.*{[^}]*throw`; `instanceof.*\?.*:`

## ISP Detection

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Interface methods | >10 | Minor |
| Empty method implementations | Any | Minor |
| Methods returning `undefined`/throwing | Any | Minor |

**Grep:** `interface.*{` with >10 method signatures; `implements.*{[^}]*throw new Error\['"]Not`

## DIP Detection

| Indicator | Severity |
|-----------|----------|
| `new ConcreteClass()` in business logic | Major |
| Import of concrete implementations (not interfaces) | Minor |
| Circular imports | Major |

**Grep:** `private.*=.*new [A-Z].*Service`; `import.*Repository|Service.*from.*(?!interface)`

## DRY Violations

| Indicator | Threshold | Severity |
|-----------|-----------|----------|
| Similar code blocks | >10 lines, >80% similar | Minor |
| Identical function bodies | >5 lines | Minor |
| Copy-pasted validation | >3 occurrences | Minor |
| Same literal value | >3 occurrences | Minor |
| Magic numbers | Any unexplained | Suggestion |

## YAGNI Violations

| Indicator | Severity |
|-----------|----------|
| Interface with exactly 1 implementation | Suggestion |
| Abstract class with exactly 1 subclass | Suggestion |
| Factory creating exactly 1 product | Suggestion |
| Generic type always same concrete type | Suggestion |
| Options param always empty/default | Suggestion |
| Plugin/extension system with 0 plugins | Suggestion |
| Parameters always same value | Suggestion |
| Switch/if branches never executed | Suggestion |
| Exported functions never imported | Suggestion |

**Grep:** `implements I[A-Z].*` appearing exactly once; `extends Abstract` appearing exactly once; `export function` with 0 imports elsewhere

## Additional Detection Heuristics

Configuration sprawl: same config value in 3+ places instead of centralized constant. Severity: Minor.
God function: single function >50 lines with >3 nesting levels. Severity: Major.
