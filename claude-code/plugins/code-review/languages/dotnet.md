## Test File Patterns

`*.Tests.cs`, `*Tests/`, `*.UnitTests/`, `*.IntegrationTests/` projects

## Category-Specific Checks

### Architecture {#architecture}

- HttpClient injected directly instead of IHttpClientFactory — prevents named client configuration and mock injection in tests
- Missing interface abstraction for external service calls (testability)

### Bugs {#bugs}

- Lazy loading disconnected — navigation property after context disposed
- IAsyncDisposable for .NET 8+ async resources (not just IDisposable)
- `[AsParameters]` (.NET 8) binds all public properties including unintended ones (Id, CreatedBy, IsAdmin) — same over-binding risk as `[FromBody]`
- async void methods outside event handlers — exceptions crash the process unobserved
- Task.Result or Task.Wait in async context — synchronous blocking causes thread pool starvation/deadlocks

### Error Handling {#errors}

- Missing try/catch around async Task operations in controllers
- AggregateException not unwrapped — InnerException ignored
- Using `catch (Exception)` without filtering (CA1031)

### Performance {#performance}

- Missing AsNoTracking for read-only queries
- ToList() before filter — materializing before Where()
- Unbounded DbContext lifetime: use AddDbContextPool for high-throughput
- `FrozenDictionary<K,V>`/`FrozenSet<T>` (.NET 8): static readonly `Dictionary`/`HashSet` fields should use Frozen variants (3-10x read perf)
- `SearchValues<char>` (.NET 8): `IndexOfAny(new char[]{...})` with static char sets should use `SearchValues.Create()` for SIMD acceleration
- `ConfigureAwaitOptions.SuppressThrowing` (.NET 8): replaces `try { await task; } catch { }` for fire-and-forget
- Missing CancellationToken propagation through async call chains — operations continue after caller cancels, wasting resources

### Security {#security}

- Hardcoded connection strings, missing `[Authorize]`
- XXE — XML parsing without disabling external entities
- Model over-binding — [Bind]/[FromBody] allowing sensitive property binding
- Missing [ValidateAntiForgeryToken] on POST/PUT/DELETE (or RequireAntiforgery() for Minimal API)
- `[FromKeyedServices]` (.NET 8): keyed DI with string keys derived from user input allows service substitution
- NuGet source confusion: `nuget.config` with both internal feed and nuget.org without `<packageSourceMapping>` (.NET 6+). Severity: Critical
- Missing packages.lock.json when using Central Package Management — non-reproducible builds
- Floating NuGet version ranges (`*`, `1.*`) in .csproj — allows unintended upgrades

### Technical Debt {#debt}

- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration
- Legacy serialization — `BinaryFormatter`, non-JSON serialization
- Missing `#nullable enable`
- Primary constructors (.NET 8/C# 12): classes with single constructor + field assignment should use primary constructor syntax
- `TimeProvider` (.NET 8): direct `DateTime.Now`/`DateTimeOffset.Now` in injectable services — use `TimeProvider.System` for testability
- `IExceptionHandler` (.NET 8): replaces `UseExceptionHandler` inline lambda pattern

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
