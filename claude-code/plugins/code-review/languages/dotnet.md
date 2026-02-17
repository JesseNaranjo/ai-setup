## Test File Patterns

- `*.Tests.cs`, `*Tests/`, `*.UnitTests/`, `*.IntegrationTests/` projects

## Category-Specific Checks

### Bugs {#bugs}

- `IDisposable` not disposed — missing using blocks
- Async deadlocks — `.Result` or `.Wait()` on async tasks (blocks thread pool)
- LINQ deferred execution — queries executed multiple times unintentionally
- Lazy loading disconnected — navigation property accessed after context disposed

### Security {#security}

- Insecure deserialization — untrusted data without type validation
- Hardcoded connection strings, missing `[Authorize]` attributes
- XXE — XML parsing without disabling external entities
- Model over-binding — [Bind]/[FromBody] allowing sensitive property binding
- JWT without validation parameters

### Performance {#performance}

- LINQ in hot loops, excessive allocations in hot paths
- Missing `ConfigureAwait(false)` in library code
- Large object heap — repeatedly allocating >85KB objects
- Missing AsNoTracking for read-only queries
- ToList() before filter — materializing query before Where()

### Architecture {#architecture}

- DI anti-patterns — service locator, captive dependencies, improper scoping
- Controller bloat — business logic in controllers
- Improper layering violations, static abuse, circular dependencies

### Error Handling {#errors}

- Swallowed exceptions — empty catch blocks, catch-only-log
- Improper `Task` handling — unawaited tasks, fire-and-forget without error handling

### Test Coverage {#tests}

- Async test issues — async void test methods, missing await in assertions
- Test isolation — shared state, database state not reset

### Technical Debt {#debt}

- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration
- Legacy serialization — `BinaryFormatter`, non-JSON serialization
- Pre-nullable context — missing `#nullable enable`

## Entity Framework Core Patterns

Apply when EF Core detected (Microsoft.EntityFrameworkCore in csproj).

### Query Issues

- N+1 via lazy loading — navigation properties in loops
- Client-side evaluation — LINQ evaluated in memory instead of SQL
- FromSqlRaw with string interpolation — use FromSqlInterpolated
- Missing AsSplitQuery — large Include graphs causing Cartesian explosion

### Context Issues

- DbContext in Singleton — scoped DbContext injected into Singleton
- Missing transactions — multiple SaveChanges without TransactionScope

## ASP.NET Core Middleware Checks

Apply when ASP.NET Core detected (Microsoft.AspNetCore in csproj).

### Middleware Order Issues

- Auth after endpoints — UseAuthentication/Authorization after MapControllers
- Exception handler position — UseExceptionHandler not at start of pipeline
- CORS misconfigured — UseCors after UseRouting or before policy applied

### Minimal API Security

- Missing RequireAuthorization, validation
- Improper Results usage — returning raw data instead of Results.Ok/BadRequest
