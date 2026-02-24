# Node.js/TypeScript LSP Reference

## Quick Reference

LSP diagnostics supplement (not replace) pattern-based detection with compiler-level precision.

**Availability:** Node.js/TypeScript → `typescript-lsp` plugin. Unavailable → skip this file; all agents function without LSP.

**Usage:** Prioritize LSP for type-related and null-reference issues; combine LSP + patterns for security and cross-cutting concerns.

## Diagnostic Code Mapping

| TS Code | Category | Description |
|---------|----------|-------------|
| TS2304 | Architecture | Cannot find name (missing import) |
| TS2307 | Architecture | Cannot find module |
| TS2322 | Bugs | Type mismatch in assignment |
| TS2339 | Bugs | Property does not exist on type |
| TS2345 | Bugs | Argument type mismatch |
| TS2531 | Bugs | Object is possibly null |
| TS2532 | Bugs | Object is possibly undefined |
| TS2551 | Bugs | Property does not exist (did you mean?) |
| TS2571 | Bugs | Object is of type 'unknown' |
| TS2614 | Architecture | Module has no exported member |
| TS2741 | Bugs | Property missing in type assignment |
| TS2769 | Bugs | No overload matches this call |
| TS6133 | Technical Debt | Unused variable/parameter |
| TS6196 | Technical Debt | Unused declaration |
| TS6198 | Technical Debt | All imports unused |
| TS7006 | Architecture | Implicit any type |
| TS7031 | Architecture | Implicit any in binding element |
| TS7053 | Bugs | Element implicitly has 'any' type (index signature) |
| TS18046 | Bugs | Value is 'unknown' |
| TS18048 | Bugs | Value is possibly 'undefined' |

## Agent Guidelines

1. **Prioritize LSP** for type bugs (TS2322, TS2345, TS2531, TS2532)
2. **LSP + patterns** for security (LSP: types, patterns: dangerous sinks)
3. **LSP for cross-file** imports, exports, circular dependencies
4. **Patterns only** for runtime-specific (Promise handling, event loop)
