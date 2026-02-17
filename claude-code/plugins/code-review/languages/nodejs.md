## Test File Patterns

- `*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `*-test.js`, `*-spec.js`
- `__tests__/`, `tests/` directories

## Modern JavaScript/TypeScript Patterns

| Modern Pattern | Legacy Pattern | Category |
|----------------|----------------|----------|
| `?.` optional chaining | `x && x.y && x.y.z` | Architecture |
| `??` nullish coalescing | `x \|\| defaultValue` (falsy trap) | Bugs |
| `Object.hasOwn()` | `hasOwnProperty.call()` | Security |
| Private class fields `#` | WeakMap or naming convention | Architecture |
| Top-level await | Wrapper async IIFE | Architecture |

### ESM vs CommonJS

- __dirname/__filename in ESM — CommonJS globals in ESM context
- Default export interop — CJS default imports from ESM modules
- Dynamic import() for CommonJS — require() where import() needed
- Package.json "type" mismatch — .mjs/.cjs extensions, wrong module syntax

## Category-Specific Checks

### Bugs {#bugs}

- Unhandled promise rejections — missing `.catch()` or try/catch around await
- Async/await pitfalls — missing `await`, parallel vs sequential execution
- Promise.all partial failure — use Promise.allSettled where appropriate
- Timezone-naive Date operations in server code

### Security {#security}

- Prototype pollution — Object.assign/recursive merge with user input
- ReDoS — regexes vulnerable to catastrophic backtracking
- JWT validation — missing signature verification, weak algorithms, improper storage
- SSRF — user input in fetch/axios URLs without allowlist
- Express helmet missing, rate limiting absent

### Performance {#performance}

- Event loop blocking — CPU-intensive ops without worker threads, sync I/O in async context
- Memory leaks — unclosed listeners, closures capturing large objects, unbounded caches
- Missing stream usage for large data, N+1 queries

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle size issues
- `any` abuse, incorrect type assertions, missing generics
- Missing noImplicitAny, noUncheckedIndexedAccess, strictNullChecks in tsconfig

### Error Handling {#errors}

- Swallowed errors — empty catch blocks, catch-only-log

### Test Coverage {#tests}

- Async test issues — missing done callback, unhandled rejections in tests
- Test isolation — shared state, order-dependent tests

### Technical Debt {#debt}

- Deprecated dependencies — npm deprecation warnings, major version 2+ behind
- Legacy bundler config — Webpack 4, Gulp/Grunt in modern projects
- Monolithic modules — 1000+ lines or 50+ exports

## Framework-Specific Checks

Apply when corresponding framework detected by context-discovery.

### Express Checks

- Missing error middleware — no app.use((err, req, res, next)) handler
- Route parameter injection — req.params in SQL/shell without validation
- Trust proxy misconfiguration
- Body parser limits — express.json() without size limits

### NestJS Checks

- Injectable scope issues — singleton services holding request-scoped data
- Circular module imports — ModuleRef required due to circular DI
- Missing guards on controllers
