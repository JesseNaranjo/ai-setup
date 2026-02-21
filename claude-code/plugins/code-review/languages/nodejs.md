## Test File Patterns

`*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `*-test.js`, `*-spec.js`, `__tests__/`, `tests/`

## Category-Specific Checks

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle bloat
- Missing noImplicitAny, noUncheckedIndexedAccess, strictNullChecks in tsconfig

### Bugs {#bugs}

- Timezone-naive Date operations in server code
- require() in ESM without createRequire (Node 14+)
- `structuredClone()` silently drops functions, Symbols, WeakMap/WeakRef entries — objects with callback properties lose them without error
- `Date.parse()` format asymmetry: `new Date('2024-01-01')` = UTC, `new Date('01/01/2024')` = local time

### Error Handling {#errors}

- Missing .catch() on Promise chains in request handlers
- Express route without try/catch wrapping async operations
- Swallowed errors — catch block with no logging or re-throw

### Performance {#performance}

- Synchronous crypto (crypto.pbkdf2Sync) blocking event loop
- `fetch()` without `AbortSignal.timeout(ms)` in server code — Node.js undici has no default timeout, waits indefinitely

### Security {#security}

- Prototype pollution — Object.assign/recursive merge with user input
- Missing helmet, missing rate limiting
- eval()/Function()/vm.runInContext() with user input
- `dotenv` loads first occurrence of duplicate keys — attacker prepending to `.env` overrides all values
- postinstall scripts: new dependencies with `scripts.postinstall` execute arbitrary code at install. Flag unfamiliar packages with postinstall
- Missing `ignore-scripts=true` in `.npmrc` for CI environments

### Technical Debt {#debt}

- Legacy bundler — Webpack 4, Gulp/Grunt in modern projects
- Monolithic modules — 1000+ lines or 50+ exports

### Test Coverage {#tests}

- Express routes without supertest integration tests
- Missing test for error middleware error path
- Async operations without rejection test cases

## Framework-Specific Checks

Apply when framework detected by context-discovery.

### Express

- Missing error middleware — no app.use((err, req, res, next)) handler
- Route param injection — req.params in SQL/shell without validation
- Trust proxy misconfiguration
- Session fixation — missing `session.regenerate()` after authentication
- Missing CSRF protection on state-changing endpoints (no csurf/csrf-csrf)

### NestJS

- Injectable scope — singleton services holding request-scoped data
- Circular module imports — ModuleRef required due to circular DI
- Missing guards on controllers
- Missing validation pipe on DTOs (no class-validator decorators)

### Vite

- `import.meta.env` variables: only `VITE_` prefixed vars exposed to client. Non-prefixed silently resolve to `undefined`
- `import.meta.glob()` default eager:false returns `() => Promise<Module>`, not Module — accessing .default without await returns undefined
- CJS dependencies without `optimizeDeps.include` cause runtime loading errors in dev (Vite pre-bundles on first access)
