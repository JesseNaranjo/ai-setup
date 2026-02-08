# Language Server Integration Reference

LSP integration details extracted from language configuration files. Agents can leverage LSP diagnostics for enhanced accuracy when available. LSP supplements (does not replace) pattern-based detection.

## Node.js/TypeScript LSP Integration

When the `typescript-lsp` plugin is available, agents can leverage real TypeScript compiler diagnostics for enhanced accuracy.

### LSP Plugin Detection

Check for TypeScript LSP availability:
- Plugin installed: `typescript-lsp` in enabled plugins
- Server running: TypeScript Language Server process active

### LSP-Enhanced Capabilities

| Capability | LSP Method | Review Enhancement |
|------------|------------|-------------------|
| Type errors | `textDocument/diagnostic` | Precise null/undefined detection, missing properties |
| Unused symbols | `textDocument/diagnostic` | Accurate dead code detection |
| Go to definition | `textDocument/definition` | Track cross-file references, circular imports |
| Find references | `textDocument/references` | Identify all usages, unused exports |
| Type information | `textDocument/hover` | Get exact types for API contract validation |

### Diagnostic Code Mapping

Map TypeScript diagnostics to review categories:

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

### Agent Usage Guidelines

When TypeScript LSP is available:

1. **Prioritize LSP diagnostics** for type-related bugs (TS2322, TS2345, TS2531, TS2532)
2. **Combine LSP + patterns** for security issues (LSP finds types, patterns find dangerous sinks)
3. **Use LSP for cross-file analysis** when tracking imports, exports, and circular dependencies
4. **Fall back to patterns** when LSP unavailable or for runtime-specific issues (Promise handling, event loop)

### Example: Enhanced Null Detection

**Pattern-only approach:**
```typescript
// Pattern: (?<![\?!])\.(\w+) after nullable
user.profile.name  // May miss context
```

**LSP-enhanced approach:**
```
Diagnostic: TS2532 at line 15, column 5
Message: Object is possibly 'undefined'
Context: user.profile is Optional<Profile>
```

LSP provides precise location and type context unavailable to pattern matching.

## .NET/C# LSP Integration

When a C# LSP (OmniSharp or `csharp-lsp` plugin) is available, agents can leverage Roslyn analyzer diagnostics for enhanced accuracy.

### LSP Plugin Detection

Check for C# LSP availability:
- Plugin installed: `csharp-lsp` or OmniSharp in enabled plugins
- Server running: OmniSharp or Roslyn LSP server process active

### LSP-Enhanced Capabilities

| Capability | LSP Method | Review Enhancement |
|------------|------------|-------------------|
| Nullable analysis | `textDocument/diagnostic` | Precise null reference detection |
| Async issues | `textDocument/diagnostic` | Detect missing awaits, async deadlocks |
| IDisposable tracking | `textDocument/diagnostic` | Accurate disposal pattern violations |
| Go to definition | `textDocument/definition` | Track inheritance chains, interface implementations |
| Find references | `textDocument/references` | Identify all usages, missing attributes |

### Diagnostic Code Mapping

Map Roslyn/analyzer diagnostics to review categories:

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

### Agent Usage Guidelines

When C# LSP is available:

1. **Prioritize LSP diagnostics** for nullable reference issues (CS8600-CS8618)
2. **Use LSP for async analysis** - CS4014 is more accurate than pattern matching for missing awaits
3. **Combine LSP + patterns** for security - LSP validates types, patterns find SQL concatenation
4. **Use LSP for IDisposable** - CA2000 tracks object lifetimes better than patterns
5. **Fall back to patterns** when LSP unavailable or for framework-specific issues (EF queries, DI patterns)

### Example: Enhanced Async Detection

**Pattern-only approach:**
```csharp
// Pattern: async method call without await
GetUserAsync(id);  // May miss context - could be intentional fire-and-forget
```

**LSP-enhanced approach:**
```
Diagnostic: CS4014 at line 25, column 9
Message: Because this call is not awaited, execution continues before the call completes
Context: Method returns Task<User>, not void
```

LSP distinguishes intentional fire-and-forget (Task-returning in void context) from bugs.
