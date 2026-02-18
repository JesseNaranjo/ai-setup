## Test File Patterns

`*.Tests.cs`, `*Tests/`, `*.UnitTests/`, `*.IntegrationTests/` projects

## Category-Specific Checks

### Bugs {#bugs}

- Lazy loading disconnected — navigation property after context disposed

### Security {#security}

- Insecure deserialization — untrusted data without type validation
- Hardcoded connection strings, missing `[Authorize]`
- XXE — XML parsing without disabling external entities
- Model over-binding — [Bind]/[FromBody] allowing sensitive property binding
- JWT without validation parameters

### Performance {#performance}

- Missing AsNoTracking for read-only queries
- ToList() before filter — materializing before Where()

### Architecture {#architecture}

- DI anti-patterns — service locator, captive dependencies, improper scoping

### Error Handling {#errors}

- Unawaited tasks, fire-and-forget without error handling

### Technical Debt {#debt}

- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration
- Legacy serialization — `BinaryFormatter`, non-JSON serialization
- Missing `#nullable enable`

## Entity Framework Core Patterns

Apply when EF Core detected (Microsoft.EntityFrameworkCore in csproj).

### Query Issues

- N+1 via lazy loading — navigation properties in loops
- Client-side evaluation — LINQ in memory instead of SQL
- FromSqlRaw with string interpolation — use FromSqlInterpolated
- Missing AsSplitQuery — large Include graphs, Cartesian explosion

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
