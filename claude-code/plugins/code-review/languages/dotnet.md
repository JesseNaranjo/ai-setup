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
- [MongoDB] BsonDocument without typed model in production queries — loses compile-time type safety
- [Redis] ConnectionMultiplexer without retry policy — transient failures cause cascading request failures
- [Blazor] Interactive elements (`@onclick`, `@onchange`) without accessible name — screen readers announce generic "button" or "element"
- [Blazor] Form inputs without associated label (`<label for="...">` or `aria-label`) — fails WCAG 1.3.1
- [Blazor] Dynamic content updates without `aria-live` region — screen readers miss asynchronous content changes (loading states, validation messages, notifications)
- [Blazor] `<NavLink>` navigation without skip-to-content link or focus management — keyboard users must tab through entire nav on every page transition

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
- [MongoDB] FindSync instead of FindAsync — blocks thread pool. Use Find().ToListAsync()
- [Redis] IDistributedCache without AbsoluteExpirationRelativeToNow — entries never expire, unbounded memory
- [Redis] StringGet/StringSet for complex objects — use HashSet/HashGet for structured data to avoid full serialization
- [ASP.NET Core] `ILogger.LogInformation($"User {userId}")` — string interpolation allocates even when log level is disabled; use structured parameters `ILogger.LogInformation("User {UserId}", userId)`
- [ASP.NET Core] Missing `IHealthCheck` registration for external dependencies (database, cache, message broker) — orchestrators cannot distinguish between app failure and dependency failure

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
- [MongoDB] Filter with user input in BsonRegularExpression — regex injection
- `HttpWebRequest` with user-controlled URL — SSRF risk; use `HttpClient` with `SocketsHttpHandler` and URL allowlist

### Technical Debt {#debt}

- Pre-.NET 6 patterns — `WebClient`, `BinaryFormatter`, non-JSON serialization, sync-over-async, old configuration
- Missing `#nullable enable`
- Primary constructors (.NET 8/C# 12): classes with single constructor + field assignment should use primary constructor syntax
- `TimeProvider` (.NET 8): direct `DateTime.Now`/`DateTimeOffset.Now` in injectable services — use `TimeProvider.System` for testability
- `IExceptionHandler` (.NET 8): replaces `UseExceptionHandler` inline lambda pattern
- `System.Threading.Lock` (.NET 9): replaces `lock(object)` — dedicated lock type with scoped `EnterScope()`, avoids accidental lock on non-lock objects
- `params` collections (.NET 9/C# 13): `params` now accepts `Span<T>`, `ReadOnlySpan<T>`, `IEnumerable<T>` — not just arrays. Flag `params T[]` when span variant reduces allocations
- LINQ `CountBy`/`AggregateBy` (.NET 9): replaces `GroupBy(k).Select(g => ...)` pattern — single-pass, no intermediate grouping allocations
- `Newtonsoft.Json` in .NET 6+ without documented compatibility requirement — `System.Text.Json` is default; flag unless codebase has explicit reason (polymorphic serialization, LINQ-to-JSON, custom converters not yet migrated)

### Test Coverage {#tests}

- EF Core queries without integration tests (in-memory provider or testcontainers)
- Missing tests for authorization policies
- Controller actions without response status code assertions
