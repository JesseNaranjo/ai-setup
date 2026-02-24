## Test File Patterns

`*.test.ts`, `*.spec.ts`, `*.test.js`, `*.spec.js`, `*-test.js`, `*-spec.js`, `__tests__/`, `tests/`

## Category-Specific Checks

### Architecture {#architecture}

- Barrel file abuse — re-exports causing bundle bloat
- Missing `noUncheckedIndexedAccess` in tsconfig — non-obvious default, allows undefined on index access without error
- `as any` casts in production code (not `.d.ts` stubs or test files) — suppresses type checking
- `@ts-ignore`/`@ts-expect-error` without justification comment on preceding line
- `enum` usage in new code: runtime overhead, poor tree-shaking — prefer `as const` objects
- Node.js 22 `--experimental-strip-types`: no enums, no namespaces, no `const enum` — type stripping silently drops these without error
- [NestJS] Circular module imports — ModuleRef required due to circular DI

### Bugs {#bugs}

- Timezone-naive Date operations in server code
- require() in ESM without createRequire (Node 14+)
- `structuredClone()` silently drops functions, Symbols, WeakMap/WeakRef entries — objects with callback properties lose them without error
- `Date.parse()` format asymmetry: `new Date('2024-01-01')` = UTC, `new Date('01/01/2024')` = local time
- Promise.all without error isolation — single rejection loses all other results (use Promise.allSettled when partial results are acceptable)
- Async route handler without error forwarding — Express doesn't catch async rejections natively; missing next(err) call or express-async-errors wrapper causes unhandled rejection
- [NestJS] Injectable scope — singleton services holding request-scoped data
- [NestJS] Missing validation pipe on DTOs (no class-validator decorators) — unvalidated input reaches service layer
- [Vite] `import.meta.env` variables: only `VITE_` prefixed vars exposed to client. Non-prefixed silently resolve to `undefined`
- [Vite] `import.meta.glob()` default eager:false returns `() => Promise<Module>`, not Module — accessing .default without await returns undefined
- [Vite] CJS dependencies without `optimizeDeps.include` cause runtime loading errors in dev (Vite pre-bundles on first access)
- [MongoDB] .find() without .limit() on user-facing queries — unbounded result sets consume memory
- [Redis] Pub/Sub without reconnection handler — silently stops receiving after connection drop

### Error Handling {#errors}

- [Express] Express route without try/catch wrapping async operations
- [Express] Missing error middleware — no app.use((err, req, res, next)) handler

### Performance {#performance}

- Synchronous crypto (crypto.pbkdf2Sync) blocking event loop
- `fetch()` without `AbortSignal.timeout(ms)` in server code — Node.js undici has no default timeout, waits indefinitely
- Missing AbortController for cancellable fetch/stream operations — abandoned requests hold connections
- [MongoDB] Mongoose .populate() in loops — N+1. Use .populate() on initial query or aggregation $lookup
- [MongoDB] Missing .lean() on read-only Mongoose queries — skips hydration overhead (5-10x faster for large result sets)
- [Redis] KEYS pattern in production — O(N) blocks event loop. Use SCAN iterator
- [Redis] SET without TTL in cache patterns — unbounded memory growth

### Security {#security}

- Prototype pollution — Object.assign/recursive merge with user input
- `dotenv` loads first occurrence of duplicate keys — attacker prepending to `.env` overrides all values
- Lifecycle scripts (preinstall/postinstall): new dependencies with lifecycle scripts execute arbitrary code at install. Flag unfamiliar packages with preinstall or postinstall
- Missing `ignore-scripts=true` in `.npmrc` for CI environments (pnpm: `pnpm.enable-pre-post-scripts=false` in package.json)
- Missing or uncommitted lockfile (package-lock.json/yarn.lock/pnpm-lock.yaml) — allows dependency version drift
- Dependencies using `*`, `latest`, or unpinned git URLs — non-reproducible builds
- [Express] Route param injection — req.params in SQL/shell without validation
- [Express] Trust proxy misconfiguration
- [Express] Session fixation — missing `session.regenerate()` after authentication
- [Express] Missing CSRF protection on state-changing endpoints (no csurf/csrf-csrf)
- [NestJS] Missing guards on controllers
- [Docker] `node:latest` or unpinned Node.js base image — use `node:<version>-slim` with digest pin
- [Docker] Running as root — add `USER node` after COPY
- [Docker] `COPY package*.json` after `COPY . .` — invalidates layer cache, rebuilds dependencies on every code change
- [MongoDB] $where or $regex with user input — JavaScript/expression injection

### Technical Debt {#debt}

- Legacy bundler — Webpack 4 and below, Gulp, Grunt. Flag Webpack 5 projects without Vite/esbuild migration plan
- Monolithic modules — 1000+ lines or 50+ exports
- Bun/Deno runtime differences: flag Node.js-specific APIs (`child_process`, `cluster`, `node:*` prefixed) when project targets multiple runtimes

### Test Coverage {#tests}

- Express routes without supertest integration tests
- Missing test for error middleware error path
- Async operations without rejection test cases
