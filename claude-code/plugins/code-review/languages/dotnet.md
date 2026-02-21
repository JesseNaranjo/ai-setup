## Test File Patterns

`*.Tests.cs`, `*Tests/`, `*.UnitTests/`, `*.IntegrationTests/` projects

## Category-Specific Checks

### Architecture {#architecture}

- Service registration order — dependent services registered before dependencies
- Missing interface abstraction for external service calls (testability)

### Bugs {#bugs}

- Lazy loading disconnected — navigation property after context disposed
- IAsyncDisposable for .NET 8+ async resources (not just IDisposable)

### Error Handling {#errors}

- Missing try/catch around async Task operations in controllers
- AggregateException not unwrapped — InnerException ignored
- Using `catch (Exception)` without filtering (CA1031)

### Performance {#performance}

- Missing AsNoTracking for read-only queries
- ToList() before filter — materializing before Where()
- Unbounded DbContext lifetime: use AddDbContextPool for high-throughput

### Security {#security}

- Hardcoded connection strings, missing `[Authorize]`
- XXE — XML parsing without disabling external entities
- Model over-binding — [Bind]/[FromBody] allowing sensitive property binding
- Missing [ValidateAntiForgeryToken] on POST/PUT/DELETE (or RequireAntiforgery() for Minimal API)

### Technical Debt {#debt}

- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration
- Legacy serialization — `BinaryFormatter`, non-JSON serialization
- Missing `#nullable enable`

### Test Coverage {#tests}

- EF Core queries without integration tests (in-memory provider or testcontainers)
- Missing tests for authorization policies
- Controller actions without response status code assertions

## Entity Framework Core Patterns

Apply when EF Core detected (Microsoft.EntityFrameworkCore in csproj).

### Query Issues

- N+1 via lazy loading — navigation properties in loops
- Client-side evaluation — LINQ in memory instead of SQL
- FromSqlRaw with string interpolation — use FromSqlInterpolated
- Missing AsSplitQuery — large Include graphs, Cartesian explosion
- Missing index hint — frequent WHERE on non-indexed columns
- Unbounded Include depth — navigation properties without explicit depth limit

### Context Issues

- DbContext in Singleton — scoped DbContext injected into Singleton
- Missing transactions — multiple SaveChanges without TransactionScope

## ASP.NET Core Middleware Checks

Apply when ASP.NET Core detected (Microsoft.AspNetCore in csproj).

### Middleware Order

- Auth after endpoints — UseAuthentication/Authorization after MapControllers
- UseExceptionHandler not at start of pipeline
- CORS misconfigured — UseCors after UseRouting or before policy

### Minimal API Security

- Missing RequireAuthorization, validation
- Raw data instead of Results.Ok/BadRequest

### Model Validation

- Missing model validation — `[ApiController]` without `ModelState.IsValid` check or FluentValidation
