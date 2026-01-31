# .NET / C# Language Configuration

Language-specific checks and patterns for .NET and C# projects.

## Detection

Detect .NET projects by checking for any of these files:
- `*.csproj` (project file)
- `*.sln` (solution file)
- `*.slnx` (solution file, newer format)

## .NET Version Detection

Detect target framework for version-specific checks:

| Target | Detection | Special Checks |
|--------|-----------|----------------|
| .NET 8+ | `<TargetFramework>net8.0</TargetFramework>` | Primary constructors, collection expressions |
| .NET 6-7 | `<TargetFramework>net6.0</TargetFramework>` | Minimal APIs, file-scoped namespaces |
| .NET Framework | `<TargetFramework>net48</TargetFramework>` | Legacy patterns flagged |
| .NET Standard | `<TargetFramework>netstandard2.0</TargetFramework>` | Compatibility concerns |

## Test File Patterns

- `*.Tests.cs`
- `*Tests/` projects
- `*.UnitTests/` projects
- `*.IntegrationTests/` projects
- `tests/` directories

## Language Server Integration (Optional)

When a C# LSP (OmniSharp or `csharp-lsp` plugin) is available, agents can leverage Roslyn analyzer diagnostics for enhanced accuracy. This supplements (does not replace) pattern-based detection.

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

## Category-Specific Checks

### Bugs {#bugs}

| Issue Type | Description |
|------------|-------------|
| Null reference exceptions | Dereferencing potentially null values without null checks |
| `IDisposable` not disposed | Objects implementing IDisposable not disposed or not in using blocks |
| Async deadlocks | Using `.Result` or `.Wait()` on async tasks (blocks thread pool) |
| LINQ deferred execution issues | LINQ queries executed multiple times unintentionally |
| String comparison issues | Case-sensitive comparisons where case-insensitive is needed |
| Collection modification during iteration | Modifying collections while iterating with foreach |

### Security {#security}

| Issue Type | Description |
|------------|-------------|
| SQL injection via string concatenation | Building SQL queries with string concatenation instead of parameters |
| Insecure deserialization | Deserializing untrusted data without type validation |
| Hardcoded connection strings | Database credentials in source code instead of configuration |
| Missing `[Authorize]` attributes | Controller actions accessible without authentication |
| Path traversal | User input in file paths without sanitization |
| Weak cryptography | MD5/SHA1 for security purposes, weak encryption modes |
| CSRF vulnerabilities | Missing anti-forgery tokens on state-changing operations |
| XXE vulnerabilities | XML parsing without disabling external entities |

### Performance {#performance}

| Issue Type | Description |
|------------|-------------|
| Boxing/unboxing overhead | Value types repeatedly boxed to object and unboxed |
| LINQ in hot loops | LINQ methods in tight loops where manual iteration is faster |
| Excessive allocations | Creating objects in loops, string concatenation in loops |
| Missing `ConfigureAwait(false)` | In library code, not using ConfigureAwait(false) on awaits |
| N+1 EF queries | Entity Framework lazy loading causing multiple database calls |
| Missing async I/O | Synchronous I/O blocking thread pool threads |
| Large object heap allocations | Repeatedly allocating objects > 85KB |

### Architecture {#architecture}

| Issue Type | Description |
|------------|-------------|
| DI anti-patterns | Service locator pattern, captive dependencies, improper scoping |
| Missing interfaces for testability | Concrete dependencies preventing unit testing |
| Controller bloat | Controllers with business logic instead of delegation |
| Improper layering violations | Presentation layer accessing data layer directly |
| Static abuse | Excessive static methods/classes preventing testability |
| Circular dependencies | Projects or classes with circular references |

### Error Handling {#errors}

| Issue Type | Description |
|------------|-------------|
| Missing exception handling | Operations that can throw without try/catch |
| Swallowed exceptions | Empty catch blocks, catch blocks that only log |
| Improper `Task` handling | Tasks not awaited, fire-and-forget without error handling |
| Missing try-finally for cleanup | Resources not cleaned up when exceptions occur |
| Generic exception catching | Catching `Exception` instead of specific types |
| Improper exception wrapping | Losing stack trace when re-throwing |

### Test Coverage {#tests}

| Issue Type | Description |
|------------|-------------|
| Missing unit tests | New classes/methods without corresponding test files |
| Missing integration tests | API endpoints without integration tests |
| Missing edge case tests | Methods without tests for null, empty, boundary values |
| Async test issues | Tests not properly handling async operations |
| Test isolation | Tests sharing state, database state not reset between tests |
| Missing mock verification | Mocks set up but not verified |

### Technical Debt {#debt}

| Issue Type | Description |
|------------|-------------|
| Deprecated NuGet packages | Packages marked obsolete, discontinued libraries |
| Pre-.NET 6 patterns | `WebClient`, sync-over-async, old configuration patterns |
| Obsolete attributes | Code using `[Obsolete]` APIs without migration plan |
| Legacy serialization | `BinaryFormatter`, non-JSON serialization in modern code |
| Task.Result usage | Sync-over-async patterns blocking threads (`.Result`, `.Wait()`) |
| Pre-nullable context | Code not using nullable reference types (`#nullable enable`) |
| TODO/FIXME debt | Comments without issue tracking references |
| Commented code | Large blocks of commented-out code (10+ lines) |
| Static class abuse | Static classes preventing testability and DI |
| Missing async suffix | Async methods without `Async` suffix convention |
