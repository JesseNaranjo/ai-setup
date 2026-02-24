## Test File Patterns

`*.Tests.cs`, `*Tests/`, `*.UnitTests/`, `*.IntegrationTests/` projects

## Category-Specific Checks

### Architecture {#architecture}

- HttpClient injected directly instead of IHttpClientFactory — prevents named client configuration and mock injection in tests
- [ASP.NET Core] Middleware ordering: UseAuthentication/Authorization after MapControllers, UseExceptionHandler not at start, CORS UseCors after UseRouting or before policy

### Bugs {#bugs}

- Lazy loading disconnected — navigation property after context disposed
- IAsyncDisposable for .NET 8+ async resources (not just IDisposable)
- `[AsParameters]` (.NET 8) binds all public properties including unintended ones (Id, CreatedBy, IsAdmin) — same over-binding risk as `[FromBody]`
- async void methods outside event handlers — exceptions crash the process unobserved
- Task.Result or Task.Wait in async context — synchronous blocking causes thread pool starvation/deadlocks
- [EF Core] DbContext in Singleton — scoped DbContext injected into Singleton
- [EF Core] Missing transactions — multiple SaveChanges without TransactionScope

### Error Handling {#errors}

- AggregateException not unwrapped — InnerException ignored
- [ASP.NET Core] Raw data instead of Results.Ok/BadRequest in Minimal API

### Performance {#performance}

- ToList() before filter — materializing before Where()
- Unbounded DbContext lifetime: use AddDbContextPool for high-throughput
- `FrozenDictionary<K,V>`/`FrozenSet<T>` (.NET 8): static readonly `Dictionary`/`HashSet` fields should use Frozen variants (3-10x read perf)
- `SearchValues<char>` (.NET 8): `IndexOfAny(new char[]{...})` with static char sets should use `SearchValues.Create()` for SIMD acceleration
- `ConfigureAwaitOptions.SuppressThrowing` (.NET 8): replaces `try { await task; } catch { }` for fire-and-forget
- Missing CancellationToken propagation through async call chains — operations continue after caller cancels, wasting resources
- `HybridCache` (.NET 9): replaces manual IDistributedCache + IMemoryCache layering — single API with stampede protection and tag-based invalidation
- [EF Core] N+1 via lazy loading — navigation properties in loops
- [EF Core] Client-side evaluation — LINQ in memory instead of SQL
- [EF Core] Missing AsSplitQuery — large Include graphs, Cartesian explosion
- [EF Core] Missing index hint — frequent WHERE on non-indexed columns
- [EF Core] Unbounded Include depth — navigation properties without explicit depth limit

### Security {#security}

- XXE — XML parsing without disabling external entities
- Model over-binding — [Bind]/[FromBody] allowing sensitive property binding
- Missing [ValidateAntiForgeryToken] on POST/PUT/DELETE (or RequireAntiforgery() for Minimal API)
- `[FromKeyedServices]` (.NET 8): keyed DI with string keys derived from user input allows service substitution
- NuGet source confusion: `nuget.config` with both internal feed and nuget.org without `<packageSourceMapping>` (.NET 6+). Severity: Critical
- Missing packages.lock.json when using Central Package Management — non-reproducible builds
- Floating NuGet version ranges (`*`, `1.*`) in .csproj — allows unintended upgrades
- [EF Core] FromSqlRaw with string interpolation — use FromSqlInterpolated
- [ASP.NET Core] Missing RequireAuthorization and input validation on Minimal API endpoints
- [ASP.NET Core] Missing model validation — `[ApiController]` without `ModelState.IsValid` check or FluentValidation
- [Docker] `mcr.microsoft.com/dotnet/aspnet:latest` or unpinned base image — pin to specific version with digest
- [Docker] Running as root — add `USER app` (built-in non-root user in .NET 8+ images)
- [Docker] Publishing with `--no-restore` in multi-stage build without prior restore layer — breaks layer caching

### Technical Debt {#debt}

- Pre-.NET 6 patterns — `WebClient`, sync-over-async, old configuration
- Legacy serialization — `BinaryFormatter`, non-JSON serialization
- Missing `#nullable enable`
- Primary constructors (.NET 8/C# 12): classes with single constructor + field assignment should use primary constructor syntax
- `TimeProvider` (.NET 8): direct `DateTime.Now`/`DateTimeOffset.Now` in injectable services — use `TimeProvider.System` for testability
- `IExceptionHandler` (.NET 8): replaces `UseExceptionHandler` inline lambda pattern
- `System.Threading.Lock` (.NET 9): replaces `lock(object)` — dedicated lock type with scoped `EnterScope()`, avoids accidental lock on non-lock objects
- `params` collections (.NET 9/C# 13): `params` now accepts `Span<T>`, `ReadOnlySpan<T>`, `IEnumerable<T>` — not just arrays. Flag `params T[]` when span variant reduces allocations
- LINQ `CountBy`/`AggregateBy` (.NET 9): replaces `GroupBy(k).Select(g => ...)` pattern — single-pass, no intermediate grouping allocations

### Test Coverage {#tests}

- EF Core queries without integration tests (in-memory provider or testcontainers)
- Missing tests for authorization policies
- Controller actions without response status code assertions
