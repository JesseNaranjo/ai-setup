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

> **LSP Integration (agent-only):** See `${CLAUDE_PLUGIN_ROOT}/shared/references/lsp-integration.md` for .NET LSP integration details. Not loaded during orchestration.

## Category-Specific Checks

### Bugs {#bugs}

- Null reference exceptions
- `IDisposable` not disposed — objects not disposed or not in using blocks
- Async deadlocks — using `.Result` or `.Wait()` on async tasks (blocks thread pool)
- LINQ deferred execution issues — queries executed multiple times unintentionally
- String comparison issues — case-sensitive comparisons where case-insensitive is needed
- Collection modification during iteration
- CancellationToken ignored
- DbContext disposed early
- Lazy loading disconnected — navigation property accessed after context disposed

### Security {#security}

- SQL injection via string concatenation
- Insecure deserialization — deserializing untrusted data without type validation
- Hardcoded connection strings
- Missing `[Authorize]` attributes
- Path traversal
- Weak cryptography — MD5/SHA1 for security purposes, weak encryption modes
- CSRF vulnerabilities
- XXE vulnerabilities — XML parsing without disabling external entities
- Open redirect in Redirect()
- Model over-binding — [Bind] or [FromBody] allowing sensitive property binding
- JWT without validation — JwtSecurityTokenHandler without validation parameters

### Performance {#performance}

- Boxing/unboxing overhead
- LINQ in hot loops
- Excessive allocations — creating objects in loops, string concatenation in loops
- Missing `ConfigureAwait(false)` — in library code
- N+1 EF queries
- Missing async I/O
- Large object heap allocations — repeatedly allocating objects > 85KB
- Sync over async in middleware
- Missing response compression
- Missing AsNoTracking
- Include without filter
- ToList() before filter — materializing query before applying Where()

### Architecture {#architecture}

- DI anti-patterns — service locator pattern, captive dependencies, improper scoping
- Missing interfaces for testability
- Controller bloat — controllers with business logic instead of delegation
- Improper layering violations
- Static abuse
- Circular dependencies

### Error Handling {#errors}

- Missing exception handling
- Swallowed exceptions — empty catch blocks, catch blocks that only log
- Improper `Task` handling — tasks not awaited, fire-and-forget without error handling
- Missing try-finally for cleanup
- Generic exception catching — catching `Exception` instead of specific types
- Improper exception wrapping — losing stack trace when re-throwing

### Test Coverage {#tests}

- Missing unit tests
- Missing integration tests
- Missing edge case tests
- Async test issues
- Test isolation — tests sharing state, database state not reset between tests
- Missing mock verification

### Technical Debt {#debt}

- Deprecated NuGet packages
- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration patterns
- Obsolete attributes — code using `[Obsolete]` APIs without migration plan
- Legacy serialization — `BinaryFormatter`, non-JSON serialization in modern code
- Task.Result usage — sync-over-async patterns blocking threads (`.Result`, `.Wait()`)
- Pre-nullable context — code not using nullable reference types (`#nullable enable`)
- TODO/FIXME debt
- Commented code — large blocks of commented-out code (10+ lines)
- Static class abuse
- Missing async suffix

## Entity Framework Core Patterns

Apply when EF Core is detected (Microsoft.EntityFrameworkCore in csproj).

### Query Issues

- N+1 via lazy loading — navigation properties accessed in loops
- Client-side evaluation — LINQ expressions evaluated in memory instead of SQL
- Raw SQL injection — FromSqlRaw with string interpolation
- Missing AsSplitQuery — large Include graphs causing Cartesian explosion

### Context Issues

- Long-lived DbContext
- DbContext in Singleton — scoped DbContext injected into Singleton service
- Missing transactions — multiple SaveChanges without TransactionScope
- Concurrent DbContext access

## ASP.NET Core Middleware Checks

Apply when ASP.NET Core is detected (Microsoft.AspNetCore in csproj).

### Middleware Order Issues

- Auth after endpoints — UseAuthentication/Authorization after MapControllers
- Exception handler position — UseExceptionHandler not at start of pipeline
- CORS misconfigured — UseCors after UseRouting or before policy applied

### Minimal API Security

- Missing RequireAuthorization
- Missing validation
- Improper Results usage — returning raw data instead of Results.Ok/BadRequest
