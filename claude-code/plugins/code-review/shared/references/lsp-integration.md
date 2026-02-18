# Language Server Integration Reference

## Quick Reference

LSP diagnostics supplement (not replace) pattern-based detection with compiler-level precision.

**Availability:** Node.js/TypeScript → `typescript-lsp` plugin. .NET/C# → `csharp-lsp` or OmniSharp plugin. Unavailable → skip this file; all agents function without LSP.

**Usage:** Prioritize LSP for type-related and null-reference issues; combine LSP + patterns for security and cross-cutting concerns.

## Node.js/TypeScript LSP

### Diagnostic Code Mapping

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

### Agent Guidelines

1. **Prioritize LSP** for type bugs (TS2322, TS2345, TS2531, TS2532)
2. **LSP + patterns** for security (LSP: types, patterns: dangerous sinks)
3. **LSP for cross-file** imports, exports, circular dependencies
4. **Patterns only** for runtime-specific (Promise handling, event loop)

## .NET/C# LSP

### Diagnostic Code Mapping

| Code | Category | Description |
|------|----------|-------------|
| CA1031 | Error Handling | Do not catch general exception types |
| CA1032 | Architecture | Implement standard exception constructors |
| CA1054 | Architecture | URI parameters should not be strings |
| CA1062 | Security | Validate parameter is not null |
| CA1063 | Architecture | Implement IDisposable correctly |
| CA1303 | Architecture | Do not pass literals as localized parameters |
| CA1304 | Bugs | Specify CultureInfo |
| CA1305 | Bugs | Specify IFormatProvider |
| CA1307 | Bugs | Specify StringComparison |
| CA1310 | Bugs | Specify StringComparison for correctness |
| CA1822 | Performance | Member can be static |
| CA1825 | Performance | Avoid zero-length array allocations |
| CA1829 | Performance | Use Length/Count property instead of Count() |
| CA1836 | Performance | Prefer IsEmpty over Count when available |
| CA1848 | Performance | Use LoggerMessage delegates |
| CA1852 | Performance | Type can be sealed |
| CA1860 | Performance | Avoid using 'Enumerable.Any()' extension method |
| CA2000 | Bugs | Dispose objects before losing scope |
| CA2007 | Performance | Missing ConfigureAwait |
| CA2100 | Security | SQL queries for security vulnerabilities |
| CA2211 | Architecture | Non-constant fields should not be visible |
| CA2227 | Architecture | Collection properties should be read only |
| CA2234 | Architecture | Pass System.Uri instead of strings |
| CS0168 | Technical Debt | Variable declared but never used |
| CS0219 | Technical Debt | Variable assigned but never used |
| CS1998 | Performance | Async method lacks await operators |
| CS4014 | Bugs | Async method not awaited |
| CS8600 | Bugs | Converting null literal to non-nullable type |
| CS8601 | Bugs | Possible null reference assignment |
| CS8602 | Bugs | Dereference of possibly null reference |
| CS8603 | Bugs | Possible null reference return |
| CS8604 | Bugs | Possible null reference argument |
| CS8618 | Bugs | Non-nullable property not initialized |
| CS8625 | Bugs | Cannot convert null literal to non-nullable |
| CS8629 | Bugs | Nullable value type may be null |

### Agent Guidelines

1. **Prioritize LSP** for nullable references (CS8600-CS8618)
2. **LSP for async** — CS4014 more accurate than patterns for missing awaits
3. **LSP + patterns** for security — LSP: types, patterns: SQL concatenation
4. **LSP for IDisposable** — CA2000 tracks lifetimes better than patterns
5. **Patterns only** for framework-specific (EF queries, DI patterns)
