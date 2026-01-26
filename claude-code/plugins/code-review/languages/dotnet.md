# .NET / C# Language Configuration

Language-specific checks and patterns for .NET and C# projects.

## Detection

Detect .NET projects by checking for any of these files:
- `*.csproj` (project file)
- `*.sln` (solution file)
- `*.slnx` (solution file, newer format)

## Test File Patterns

- `*.Tests.cs`
- `*Tests/` projects
- `*.UnitTests/` projects
- `*.IntegrationTests/` projects
- `tests/` directories

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
