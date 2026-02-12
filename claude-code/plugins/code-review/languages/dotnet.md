## Test File Patterns

- `*.Tests.cs`
- `*Tests/` projects
- `*.UnitTests/` projects
- `*.IntegrationTests/` projects
- `tests/` directories

## Category-Specific Checks

### Bugs {#bugs}

- Null reference exceptions
- `IDisposable` not disposed — missing using blocks
- Async deadlocks — `.Result` or `.Wait()` on async tasks (blocks thread pool)
- LINQ deferred execution — queries executed multiple times unintentionally
- String comparison issues — case-sensitive where case-insensitive needed
- Collection modification during iteration
- CancellationToken ignored
- DbContext disposed early
- Lazy loading disconnected — navigation property accessed after context disposed

### Security {#security}

- SQL injection via string concatenation
- Insecure deserialization — untrusted data without type validation
- Hardcoded connection strings
- Missing `[Authorize]` attributes
- Path traversal
- Weak cryptography — MD5/SHA1 for security, weak encryption modes
- CSRF vulnerabilities
- XXE — XML parsing without disabling external entities
- Open redirect in Redirect()
- Model over-binding — [Bind]/[FromBody] allowing sensitive property binding
- JWT without validation parameters

### Performance {#performance}

- Boxing/unboxing overhead
- LINQ in hot loops
- Excessive allocations — object creation in loops, string concat in loops
- Missing `ConfigureAwait(false)` in library code
- N+1 EF queries
- Missing async I/O
- Large object heap — repeatedly allocating >85KB objects
- Sync over async in middleware
- Missing response compression
- Missing AsNoTracking
- Include without filter
- ToList() before filter — materializing query before Where()

### Architecture {#architecture}

- DI anti-patterns — service locator, captive dependencies, improper scoping
- Missing interfaces for testability
- Controller bloat — business logic in controllers
- Improper layering violations
- Static abuse
- Circular dependencies

### Error Handling {#errors}

- Swallowed exceptions — empty catch blocks, catch-only-log
- Improper `Task` handling — unawaited tasks, fire-and-forget without error handling
- Missing try-finally for cleanup
- Generic exception catching — catching `Exception` instead of specific types
- Improper exception wrapping — losing stack trace on re-throw

### Test Coverage {#tests}

- Missing unit/integration/edge case tests
- Async test issues
- Test isolation — shared state, database state not reset
- Missing mock verification

### Technical Debt {#debt}

- Deprecated NuGet packages
- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration
- Obsolete attributes — `[Obsolete]` APIs without migration plan
- Legacy serialization — `BinaryFormatter`, non-JSON serialization
- Task.Result usage — sync-over-async blocking threads
- Pre-nullable context — missing `#nullable enable`
- TODO/FIXME debt
- Commented code (10+ lines)
- Static class abuse
- Missing async suffix

## Entity Framework Core Patterns

Apply when EF Core is detected (Microsoft.EntityFrameworkCore in csproj).

### Query Issues

- N+1 via lazy loading — navigation properties in loops
- Client-side evaluation — LINQ evaluated in memory instead of SQL
- Raw SQL injection — FromSqlRaw with string interpolation
- Missing AsSplitQuery — large Include graphs causing Cartesian explosion

### Context Issues

- Long-lived DbContext
- DbContext in Singleton — scoped DbContext injected into Singleton
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
